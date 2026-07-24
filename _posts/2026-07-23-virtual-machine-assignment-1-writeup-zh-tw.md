---
layout: post
title: "114-2 虛擬機器作業一報告"
date: 2026-07-23 09:00:00 +0800
permalink: /zh/projects/114-2-vm-assignment-1-writeup/
lang: zh-TW
translation_key: 114-2-vm-assignment-1-writeup
categories: [Coursework]
tags: [Project, Virtual Machines, AI Assisted]
published: true
---

[English](/projects/114-2-vm-assignment-1-writeup/)

這是個人課程作業，實作過程接受 Claude Code 逐步引導。我跟著操作並理解 guest 到 HVC 再到 KVM 的控制路徑，之後在 host 上交叉檢查實際 affinity。以下技術報告保留繳交內容，不主張實作由我獨立完成。

2026 Spring · Virtual Machine (VM)<br>
Ming-Lung Tsai · CSIE, B12902078

## 1 第一部分：設計與實作

### 1.1 環境設定

- Host：x86 上的 Ubuntu 22.04 LTS
- QEMU：v7.0.0
- Linux/KVM：v5.15
- 磁碟映像：兩個 Ubuntu 20.04 cloud image：`cloud.img`（25GB），作為 KVM host filesystem；`cloud_inner.img`（2GB，大小設定為可放進 `cloud.img`），供 nested VM 使用。

### 1.2 整體架構

這份作業的目標，是讓 guest VM 能把自己的 vCPU 固定到 KVM host 上的特定 physical CPU。要做到這件事，除了修改 KVM 本身，還需要一個在 guest 內執行的 kernel module（`hw1_module`），作為 user space 與 hypervisor 之間的介面。Guest kernel 看不到 host 上代表 vCPU 的 thread，因此不能只在 guest 內處理 affinity。Affinity 操作必須跨越 guest/host 邊界，也就需要一次 VM exit。

讀寫兩個方向的理由相同。Guest 寫入 cpumask 時，module 必須觸發 VM exit，讓 KVM 在 host 端對所有 vCPU kernel thread 呼叫 `set_cpus_allowed_ptr()`。Guest 讀取目前 cpumask 時，也需要另一次 VM exit，才能從 host 取回數值。

端到端流程（寫入）：user-space 程式開啟 `/dev/hw1_module`，以 8-bit cpumask 呼叫 `write()`。Kernel module 在 `device_write()` 收到資料後發出 HVC hypercall，觸發 VM exit 並 trap 進 EL2。KVM 的 hypercall handler 把請求分派到 `kvm_assign_affinity()`；該函式透過 `kvm_for_each_vcpu` 走訪所有 vCPU，對每個 vCPU kernel thread 以指定 cpumask 呼叫 `set_cpus_allowed_ptr()`，把整台 VM 固定在指定 physical CPU 上。

端到端流程（讀取）：user-space 程式對 `/dev/hw1_module` 呼叫 `read()`。Kernel module 在 `device_read()` 發出 HVC hypercall，trap 進 EL2。KVM 將請求分派到 `kvm_read_affinity()`，讀取 `current->cpus_ptr`；此處的 `current` 是處理這次 VM exit 的 vCPU kernel thread。因為 `kvm_assign_affinity()` 對所有 vCPU 套用相同 mask，每個 vCPU thread 都有相同 cpumask，所以無論哪個 vCPU 處理 hypercall，回傳值都一致。

### 1.3 KVM 端實作

我們先追蹤 VM exit handling path，找出適合插入實作的位置。

![KVM handle_exit 實作](/assets/img/vm-assignment-1/hw1-2_1.png)

VM exit 發生時，KVM 進入 `handle_exit()`，再分派到 `handle_trap_exception()`。該函式呼叫 `kvm_get_exit_handler()`，依 ESR exception class 從 `arm_exit_handlers[]` 取出 function pointer。HVC call 會對應到 `handle_hvc()`，最後呼叫 `kvm_hvc_call_handler()`。

![KVM HVC call handler](/assets/img/vm-assignment-1/hw1-2_2.png)

```text
handle_exit() => handle_trap_exception() => kvm_get_exit_handler() =>
arm_exit_handlers[esr_ec] => handle_hvc() => kvm_hvc_call_handler()
```

`kvm_hvc_call_handler()` 依 `func_id` 選擇目標函式。從 `arm_hypercalls.h` 可見，function ID 取自 vCPU register `x0`；guest 在發出 hypercall 前也必須把 ID 放在這裡。

![SMCCC function ID 從 vCPU register x0 讀取](/assets/img/vm-assignment-1/hw1-3_1.png)

理解 call chain 後，我們做了以下修改。在 `include/linux/arm-smccc.h` 定義 hypercall `func_id`：

```c
#define ARM_SMCCC_ASSIGN_AFFINITY \
    ARM_SMCCC_CALL_VAL(ARM_SMCCC_FAST_CALL, ARM_SMCCC_SMC_64, \
                       ARM_SMCCC_OWNER_STANDARD, 0x54)
#define ARM_SMCCC_READ_AFFINITY \
    ARM_SMCCC_CALL_VAL(ARM_SMCCC_FAST_CALL, ARM_SMCCC_SMC_64, \
                       ARM_SMCCC_OWNER_STANDARD, 0x55)
```

在 `arch/arm64/kvm/hypercalls.c` 註冊 handler：

```c
case ARM_SMCCC_ASSIGN_AFFINITY:
    return kvm_assign_affinity(vcpu);
case ARM_SMCCC_READ_AFFINITY:
    return kvm_read_affinity(vcpu);
```

在新檔案 `arch/arm64/kvm/affinity.c` 實作 handler，並把它加入 `arch/arm64/kvm/Makefile`：

```c
int kvm_read_affinity(struct kvm_vcpu *vcpu)
{
    u32 val;
    const cpumask_t *mask = current->cpus_ptr;
    bitmap_to_arr32(&val, cpumask_bits(mask), 32);
    printk("kvm_read_affinity: current->pid=%d affinity_mask=%*pbl val=%u\n",
           vcpu->pid, cpumask_pr_args(mask), val);
    unsigned long i;
    struct kvm_vcpu *v;
    kvm_for_each_vcpu (i, v, vcpu->kvm) {
        int pid = pid_vnr(v->pid);
        printk("kvm_read_affinity: vcpu->pid=%d affinity_mask=%*pbl val=%u\n",
               pid, cpumask_pr_args(mask), val);
    }

    smccc_set_retval(vcpu, val, 0, 0, 0);
    return 1;
}

int kvm_assign_affinity(struct kvm_vcpu *vcpu)
{
    u32 val = smccc_get_arg1(vcpu);
    cpumask_t mask;
    cpumask_clear(&mask);
    bitmap_from_arr32(cpumask_bits(&mask), &val, 32);

    /*
     * NOTE: By the ordering of spec, it should apply affinity to all vcpu
     * not just current(pthread)
     *
     * Old approach: only pins the vCPU thread handling this HVC.
     * If the guest process migrates to another vCPU between ASSIGN and READ,
     * the READ will see a different current->cpus_ptr.
     *
     * set_cpus_allowed_ptr(current, &mask);
     */

    struct kvm_vcpu *v;
    unsigned long i;
    kvm_for_each_vcpu (i, v, vcpu->kvm) {
        struct task_struct *t;
        rcu_read_lock();
        t = pid_task(rcu_dereference(v->pid), PIDTYPE_PID);
        if (t)
            get_task_struct(t);
        rcu_read_unlock();
        if (t) {
            set_cpus_allowed_ptr(t, &mask);
            put_task_struct(t);
            int pid = pid_vnr(v->pid);
            printk("kvm_assign_affinity: vcpu->pid=%d, affinity_mask=%*pbl val=%u\n",
                   pid, cpumask_pr_args(&mask), val);
        }
    }

    smccc_set_retval(vcpu, val, 0, 0, 0);
    return 1;
}
```

`kvm_assign_affinity()` 透過 `kvm_for_each_vcpu`，把 cpumask 套用到所有 vCPU kernel thread，而不只處理當次 hypercall 的 thread。如此一來，即使 guest process 在 ASSIGN 與 READ 之間移到另一個 vCPU，從任一 vCPU thread 的 `current->cpus_ptr` 讀值，`kvm_read_affinity()` 都會得到一致結果。

### 1.4 Guest kernel module

`hw1_module.c` 是 guest user space 與 KVM hypercall 介面之間的橋接。寫入時，module 從 user space 讀取 8-bit cpumask，再透過 inline HVC instruction 發出 `ARM_SMCCC_ASSIGN_AFFINITY`，依 SMCCC calling convention 把 mask 放在 `x1`。讀取時，它發出 `ARM_SMCCC_READ_AFFINITY`，再把 `x0` 的回傳值複製回 user space。

```c
static ssize_t hw1_module_read(struct file *file, char __user *buffer,
                               size_t count, loff_t *offset) {
  pr_info("hw1_module: Do the Read Affinity Operations\n");
  uint8_t cpumask;
  register uint64_t x0 asm("x0") = ARM_SMCCC_READ_AFFINITY; // func_id
  asm volatile("hvc #0\n" : "+r"(x0)::"memory");
  uint8_t result = (uint8_t)x0;
  if (copy_to_user(buffer, &result, 1))
    return -EFAULT;
  return count;
}

static ssize_t hw1_module_write(struct file *file, const char __user *buffer,
                                size_t count, loff_t *offset) {
  pr_info("hw1_module: Do the Assign Affinity Operations\n");
  uint8_t cpumask;
  if (copy_from_user(&cpumask, buffer, 1))
    return -EFAULT;
  register uint64_t x0 asm("x0") = ARM_SMCCC_ASSIGN_AFFINITY; // func_id
  register uint64_t x1 asm("x1") = cpumask;
  asm volatile("hvc #0\n" : "+r"(x0) : "r"(x1) : "memory");
  return count;
}
```

### 1.5 建置與執行方式

#### 1.5.1 修改的檔案

| 檔案 | 修改 |
|---|---|
| `include/linux/arm-smccc.h` | 定義 `ARM_SMCCC_ASSIGN_AFFINITY` 和 `ARM_SMCCC_READ_AFFINITY` |
| `arch/arm64/kvm/hypercalls.c` | 加入兩個新 function ID 的 dispatch case |
| `arch/arm64/kvm/affinity.c` | 新檔案，實作 `kvm_assign_affinity()` 和 `kvm_read_affinity()` |
| `arch/arm64/kvm/Makefile` | 把 `affinity.o` 加入 build |

#### 1.5.2 套用 patch，重新建置 kernel 與 kernel module

```bash
cd linux/
git apply b12902078_hw1_kernel.patch
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
make KDIR=/path/to/linux ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

#### 1.5.3 啟動 KVM host，並把 `hw1_module.ko` 傳到 KVM host

```bash
./run-kvm.sh -k arch/arm64/boot/Image -i cloud.img
scp -P 2222 hw1_module.ko root@localhost:/root
scp -P 2222 /path/to/Image root@localhost:/root
scp -P 2222 /path/to/cloud_inner.img root@localhost:/root
scp -P 2222 /path/to/test.c root@localhost:/root
```

#### 1.5.4 把測試程式和 `hw1_module.ko` 放進 `cloud_inner.img`（在 KVM host 中）

```bash
gcc -static -o test test.c
sudo mount -o loop cloud_inner.img /mnt
sudo cp test /mnt/root/test
sudo cp hw1_module.ko /mnt/root/hw1_module.ko
sudo umount /mnt
```

#### 1.5.5 執行 guest VM

```bash
./run-guest.sh -k Image -i cloud_inner.img
```

#### 1.5.6 安裝 module 並執行測試

```bash
insmod hw1_module.ko
./test
```

## 2 第二部分：實驗與評估

### 2.1 實驗目標

這項實驗要驗證 KVM host 上的 affinity 機制是否正確：從 guest VM 內寫入 cpumask 後，KVM host 上對應的 vCPU kernel thread 應固定到指定 physical CPU。

### 2.2 設定

KVM host 透過 QEMU（`-smp 6`）以 6 個 vCPU 啟動。Nested VM 使用 2 個 vCPU。Kernel module `hw1_module.ko` 編譯後，以 `insmod` 安裝在 nested VM 內。

我們在 KVM host 上使用以下工具觀察 vCPU thread scheduling：

- `ps -eLo pid,psr,comm`：顯示每個 thread 目前執行在哪個 physical CPU 上（affinity）
- `taskset -p <pid>`：顯示 thread 設定的 `Cpus_allowed` mask（affinity）
- `journalctl -k -n 20`：確認 hypercall 已到達 EL2，且收到正確 cpumask

### 2.3 步驟

#### 2.3.1 安裝 module 並執行測試

```bash
insmod hw1_module.ko
./test
```

#### 2.3.2 找出 QEMU vCPU

```bash
ps -eLo pid,psr,comm | grep qemu
htop
```

#### 2.3.3 記錄 pinning 前的 baseline affinity

```bash
taskset -p <vcpu thread_pid>
cat /proc/<vcpu thread_pid>/status | grep Cpus_allowed
```

#### 2.3.4 在 guest VM 執行測試與 while loop

```bash
./test
while true; do :; done &
```

#### 2.3.5 確認 hypercall 已執行

```bash
journalctl -k -n 20
```

#### 2.3.6 在 KVM host CPU 上驗證 affinity 已生效

```bash
taskset -p <vCPU_thread_pid>
cat /proc/<vCPU_thread_pid>/status | grep Cpus_allowed
```

### 2.4 結果

#### 2.4.1 找出 vCPU

![QEMU vCPU thread](/assets/img/vm-assignment-1/hw1-8_1.png)

![htop 中的 QEMU vCPU thread](/assets/img/vm-assignment-1/hw1-8_2.png)

- Guest VM 有兩個 vCPU（pthread），因此可以從 htop 找出 PID。

#### 2.4.2 記錄 pinning 前的 baseline affinity

![vCPU thread 991 的 baseline affinity](/assets/img/vm-assignment-1/hw1-8_3.png)

![vCPU thread 992 的 baseline affinity](/assets/img/vm-assignment-1/hw1-8_4.png)

#### 2.4.3 在 guest VM 執行測試與 while loop

![Guest 測試把 CPU mask 從 3f 改成 05](/assets/img/vm-assignment-1/hw1-8_5.png)

#### 2.4.4 確認 hypercall 已執行

![Kernel journal 顯示 KVM affinity hypercall](/assets/img/vm-assignment-1/hw1-9_1.png)

#### 2.4.5 驗證 affinity 已套用到 KVM host CPU

![QEMU vCPU thread 已固定到 host CPU](/assets/img/vm-assignment-1/hw1-9_2.png)

---

[下載原始 PDF](/files/114-2-vm-assignment-1-writeup.pdf)

---
layout: post
title: "114-2 虛擬機器作業二報告"
date: 2026-07-23 10:00:00 +0800
permalink: /zh/projects/114-2-vm-assignment-2-writeup/
lang: zh-TW
translation_key: 114-2-vm-assignment-2-writeup
categories: [Coursework]
tags: [Project, Virtual Machines, AI Assisted]
published: true
---

[English](/projects/114-2-vm-assignment-2-writeup/)

這是個人課程作業，實作過程接受 Claude Code 實際操作引導。我跟著操作、學習並驗證跨層控制路徑；以下技術報告保留繳交內容，不主張 patch 由我獨立完成。

2026 Spring · Virtual Machine (VM)<br>
Ming-Lung Tsai · CSIE, B12902078

## Cloud Hypervisor 移植

## Q1. Realm KVM 介面

### (a) 如何找到正確的 patch 位置？

請列出用來辨識問題的來源，例如 Linux kernel code、RFC patch 等。

```text
enable_cap -> ioctl(KVM_ENABLE_CAP())
```

![Cloud Hypervisor 建立 Realm 時回傳 EINVAL](/assets/img/vm-assignment-2/hw2-1_1.png)

我從錯誤訊息開始，尋找 source 中可能出錯的位置，首先找到 `cloud-hypervisor/hypervisor/src/kvm/mod.rs` 的 `arm_rme_realm_create`。從錯誤訊息也能判斷失敗的 branch：一定是 `kvm_enable_cap`，因為只有這裡會單獨得到 os error 22（也就是 `EINVAL`），沒有印出額外訊息。

接著，我從規格與 source code 理解 Rust wrapper 的分層方式：`kvm-ioctls` 定義行為，也就是要對 KVM 發出哪個 ioctl；`kvm-bindings` 則定義 ioctl 的內容，並映射到 Linux 的 `kvm.h`。

![Cloud Hypervisor 中的 arm_rme_realm_create](/assets/img/vm-assignment-2/hw2-2_1.png)

綜合這些線索後，我先檢查 binding 中的 capability 定義 `KVM_CAP_ARM_RME` 是否和 `kvm.h` 相同：

- `kvm.h`

![kvm.h 中的 KVM_CAP_ARM_RME](/assets/img/vm-assignment-2/hw2-2_2.png)

- `bindings.rs`

![bindings.rs 中的 KVM_CAP_ARM_RME](/assets/img/vm-assignment-2/hw2-2_3.png)

`kvm.h` 是 240，`bindings.rs` 卻是 300，因而找到了 root cause。把兩處都改成 240 後，Normal VM 可以正常開機。

![修正第一個常數後 Normal VM 開機](/assets/img/vm-assignment-2/hw2-3_1.png)

![Normal VM 到達 Buildroot login prompt](/assets/img/vm-assignment-2/hw2-3_2.png)

但只做上述修改時，Realm VM 仍無法啟動。

![Realm VM 仍在 vCPU finalize 階段失敗](/assets/img/vm-assignment-2/hw2-3_3.png)

這次問題出在 `KVM_ARM_VCPU_FINALIZE`。沿用 Normal VM 的推理方式，我找到 root cause 並修改常數，Realm VM 之後就能正確啟動。

- `bindings.rs`

![bindings.rs 中的 KVM_ARM_VCPU_REC](/assets/img/vm-assignment-2/hw2-3_4.png)

- `kvm.h`

![kvm.h 中的 KVM_ARM_VCPU_REC](/assets/img/vm-assignment-2/hw2-3_5.png)

兩個 bug 本質上是同一類問題：cloud-hypervisor 把 kvm fork 固定在 `cca/v7`，但 `cca-3world` 中的 Linux 已經是 `cca-host/v8`，兩個版本之間 kernel UAPI 的編號發生變化。v8 在 `KVM_CAP_ARM_RME` 和 `KVM_ARM_VCPU_REC` 之前插入新的 enum，造成 binding 裡的常數和 kernel `kvm.h` 不一致。因此，辨識兩個問題的方法都是比較 `bindings.rs` 與 `~/.shrinkwrap/build/source/cca-3world/linux` 下 `kvm.h` 的常數：

- `KVM_CAP_ARM_RME`：bindings 300 -> `kvm.h`（`uapi/linux/kvm.h:944`）240
- `KVM_ARM_VCPU_REC`：bindings 8 -> `kvm.h` 9（v8 插入 `KVM_ARM_VCPU_HAS_EL2_E2H0 = 8`）

### (b) CCA 用哪個 RMI 執行 vCPU？

在 `vmm/src/cpu.rs` 中，`start_vcpu` 會建立 thread、設定 vCPU attribute，再透過 KVM 執行 vCPU。

繼續追蹤程式碼，可以到 `linux/arch/arm64/kvm/arm.c` 的 `kvm_arch_vcpu_ioctl_run`。

其中一段程式碼會跳到 `kvm_rec_enter`：

```c
if (vcpu_is_rec(vcpu))
    ret = kvm_rec_enter(vcpu);
else
    ret = kvm_arm_vcpu_enter_exit(vcpu);
```

補充：RMM 中的 vCPU 稱為 Realm Execution Context（REC）。

`kvm_rec_enter` 位於 `linux/arch/arm64/kvm/rme.c`（約第 1477 行），它會呼叫 `rmi_rec_enter`；後者是在 `linux/arch/arm64/include/asm/rmi_cmds.h` 定義的 SMC wrapper：

- `kvm_rec_enter`

```c
int kvm_rec_enter(struct kvm_vcpu *vcpu)
{
    struct realm_rec *rec = &vcpu->arch.rec;

    switch (rec->run->exit.exit_reason) {
    case RMI_EXIT_HOST_CALL:
    case RMI_EXIT_PSCI:
        for (int i = 0; i < REC_RUN_GPRS; i++)
            rec->run->enter.gprs[i] = vcpu_get_reg(vcpu, i);
        break;
    case RMI_EXIT_RIPAS_CHANGE:
        kvm_complete_ripas_change(vcpu);
        break;
    }

    if (kvm_realm_state(vcpu->kvm) != REALM_STATE_ACTIVE)
        return -EINVAL;

    return rmi_rec_enter(virt_to_phys(rec->rec_page),
                         virt_to_phys(rec->run));
}
```

- `rmi_rec_enter`

```c
static inline int rmi_rec_enter(unsigned long rec, unsigned long run_ptr)
{
    struct arm_smccc_res res;

    arm_smccc_1_1_invoke(SMC_RMI_REC_ENTER, rec, run_ptr, &res);

    return res.a0;
}
```

因此使用的 RMI 是 `RMI_REC_ENTER`，由 `rmi_rec_enter` 透過 `SMC_RMI_REC_ENTER` SMC call 發出。其 FID 為 `0x015c`，定義在 `linux/arch/arm64/include/asm/rmi_smc.h`。RMM 執行 REC 後，會 ERET 回 Realm 的 EL1，繼續執行 guest code。

### (c) 建立 Realm guest memory 的 Stage-2 mapping 時使用哪些 RMI？

在 `linux/arch/arm64/kvm/rme.c` 中，受保護 guest memory 的 Stage-2 mapping 使用以下 RMI：

- `rmi_data_create`（`SMC_RMI_DATA_CREATE`，FID `0x0153`）：在 `realm_create_protected_data_granule` 中，以已知內容（從 host 複製）填入 protected leaf entry。Realm 開機載入 kernel image 時會使用這個介面。
- `rmi_data_create_unknown`（`SMC_RMI_DATA_CREATE_UNKNOWN`，FID `0x0154`）：在 `realm_map_protected` 中，替 guest 執行時才 fault-in 的 page 建立 protected leaf entry；host 看不到其內容。
- `rmi_rtt_create`（`SMC_RMI_RTT_CREATE`，FID `0x015d`）：`realm_create_rtt_levels` 用它建立中間 RTT level。RTT 是 Realm Translation Table，也就是 Stage-2 page table 的 non-leaf structure。如果上述任一 `rmi_data_create*` 回傳 `RMI_ERROR_RTT`，代表對應層級缺少 RTT，因此會先呼叫此函式建立，再重新嘗試。

簡單來說，`rmi_rtt_create` 建立 page table 的「結構」（中間層），`rmi_data_create` 與 `rmi_data_create_unknown` 填入「leaf」。（對應的 unprotected/shared 半部使用 `rmi_rtt_map_unprotected`，FID `0x015f` 進行 mapping。）

## Q2. 錯誤分析

以下 RMM log 會重複出現：

```text
[ rmm ] Inject Sync EA into current REC. FAR_EL2: 0xffffffffff5fd018, ELR_EL2: 0xffffc23d55f15a54
```

### (a) FAR_EL2 和 ELR_EL2 是什麼？

- **ELR_EL2，Exception Link Register**

  當 interrupt、hypervisor call（HVC）或 abort 的目標是 EL2 時，處理器會自動把造成例外，或原本即將執行之 instruction 的 address 存入 `ELR_EL2`。Hypervisor 處理完例外後執行 ERET，跳回 `ELR_EL2` 保存的 address，繼續正常執行。

- **FAR_EL2，Fault Address Register**

  如果 CPU 因為存取無效或受限的 memory location 而發生 exception，`FAR_EL2` 會記錄確切的 faulting virtual address。作業系統或 hypervisor 會讀取 `FAR_EL2` 進行診斷，再採取修正動作，例如建立 memory mapping 或終止行為異常的 VM。

### (b) FAR_EL2 最低 12 bits 為 0x018。這個 offset 對應哪個 PL011 register？

從 `include/linux/amba/serial.h` 可見：

![UART01x_FR 定義](/assets/img/vm-assignment-2/hw2-5_1.png)

也就是 `#define UART01x_FR 0x18`，對應 UART Flag register（UARTFR）。Earlycon 傳送字元前會輪詢該 register 的 TXFF/BUSY bit，因此 SEA 中 `FAR_EL2` 的最低 12 bits 是 `0x018`。

### (c) RMM 中實際注入此錯誤的函式是哪一個？

查看 `/.shrinkwrap/build/source/cca-3world/rmm` 的 RMM source code。

以下程式來自 `/cca-3world/rmm/runtime/core/inject_exp.c`：

```c
void inject_sync_idabort(unsigned long fsc)
{
    unsigned long esr_el2 = read_esr_el2();
    unsigned long far_el2 = read_far_el2();
    unsigned long elr_el2 = read_elr_el2();
    unsigned long spsr_el2 = read_spsr_el2();
    unsigned long vbar_el1 = read_vbar_el12();
    unsigned long sctlr_el1 = read_sctlr_el12();

    unsigned long esr_el1 = calc_esr_idabort(esr_el2, spsr_el2, fsc);
    bool sctlr2_ease_bit = ((read_sctlr2_el12_if_present() &
                             SCTLR2_ELx_EASE_BIT) != 0UL);
    unsigned long pc = calc_vector_entry(vbar_el1, spsr_el2,
                                         sctlr2_ease_bit);
    unsigned long spsr = calc_spsr(spsr_el2, sctlr_el1);

    write_far_el12(far_el2);
    write_elr_el12(elr_el2);
    write_spsr_el12(spsr_el2);
    write_esr_el12(esr_el1);
    write_elr_el2(pc);
    write_spsr_el2(spsr);

    INFO("Inject Sync EA into current REC. FAR_EL2: 0x%lx, ELR_EL2: 0x%lx\n",
         far_el2, elr_el2);
}
```

`Inject Sync EA into current REC. FAR_EL2: ...` 這行 log 是由函式結尾的 `INFO()` 印出，因此注入函式是 `inject_sync_idabort`。它的 caller 是 `runtime/core/exit.c` 的 `handle_data_abort`：REC 因 data abort 離開並進入 RMM，而 IPA 被判斷為 protected 且 RIPAS = EMPTY 時，就會呼叫 `inject_sync_idabort(ESR_EL2_ABORT_FSC_SEA)`（`exit.c:233`）。這與 Q3(b) 的規格說明一致。

## Q3. RMM 規格

請參考 RMM specification：

### (a) 引用規格，說明 unprotected IPA 和 protected IPA 的差異。

根據 D2.1 Realm shared memory protocol description：

Realm software 把 IPA 的最高有效 bit 當成「protection attribute」bit。每個最高有效 bit 為 `0` 的 Protected IPA 都有一個對應的 Unprotected IPA alias；把最高有效 bit 設為 `1` 就能產生該 alias。

### (b) 引用規格，說明 RMM 何時會向 REC 注入 SEA。

根據 D2.1 Realm shared memory protocol flow：

![RMM shared memory protocol flow](/assets/img/vm-assignment-2/hw2-7_1.png)

從這個流程可以歸納 RMM 向 REC 注入 SEA 的時機：Realm 存取 RIPAS 為 EMPTY 的 Protected IPA 時，代表該 IPA 尚未設為 RAM，也沒有實際的 protected page backing，RMM 就會把 SEA 注入該 REC。相反地，存取 RIPAS 為 RAM 的 protected page 不會產生 SEA。圖中的第三階段（存取 Unprotected footprint -> SEA），以及建立 shared buffer 時在設為 RIPAS_EMPTY 後存取 protected alias（-> SEA），都符合相同規則。

這也與 RMM source 一致：在 `runtime/core/exit.c::handle_data_abort` 中，當 `empty_ipa = ipa_is_empty(fipa, rec)` 為 true，就會呼叫 `inject_sync_idabort(ESR_EL2_ABORT_FSC_SEA)`；超過 Realm IPA size 的 IPA 也會注入 SEA。這份作業中的 earlycon failure，正是因為它透過 protected alias（最高 IPA bit = 0）存取 PL011，而該 Protected IPA 為 EMPTY，所以才會反覆出現 Inject Sync EA。

## Q4. 驗證修正

套用修正後，在相同 kernel image 上執行以下實驗：

### (a) pl011 的 physical address 是多少？

Kernel 透過 `/proc/iomem` 公開目前系統的 physical memory 與 device MMIO region map。在 Realm guest 中執行 `grep "pl011" /proc/iomem`。

![proc iomem 中的 PL011 address](/assets/img/vm-assignment-2/hw2-7_2.png)

pl011 的 physical address 是 `0x09000000`，範圍為 `09000000-09000fff`。這是 device tree 的 `reg` 提供的 bare IPA，沒有設定 shared bit，因此 `/proc/iomem` 顯示的是 protected half 中的 address。

### (b) 在 dmesg 中找出 earlycon 的 physical MMIO address。兩者相同嗎？

![dmesg 中的 earlycon MMIO address](/assets/img/vm-assignment-2/hw2-8_1.png)

Earlycon MMIO address 是 `0x0000800009000000`（`earlycon: pl11 at MMIO 0x0000800009000000`）。

不相同。兩者正好相差 bit 47，也就是 shared/unprotected bit：`0x800009000000 = 0x09000000 | (1 << 47)`。在第二部分中，我們針對 cloud-hypervisor 的 `arm_rme` 情況，把 shared bit OR 進 `earlycon=` address，讓 earlycon 直接指向 unprotected alias，因而能成功輸出訊息。

我們沒有修改 device tree 中裝置的 `reg`，因此裝置回報 IPA 的方式不變，仍是 bare `0x09000000`，也就是 (a) 中看到的數值。PL011 driver 也能運作，因為 Realm-aware `ioremap()` 會在 page table 中替它加入 shared bit。

簡單來說，差異在於 shared bit 套用的階段不同。Earlycon 是在 command line 的 address 上預先加入 shared bit，也就是在 Realm ioremap 之前，並繞過它。一般 PL011 driver 則是在建立 mapping 時，由 ioremap 加入 shared bit。

### (c) 找出兩行只有啟用 earlycon 時才會出現的 boot message。

請排除 earlycon initialization 本身。

![Legacy bootconsole message](/assets/img/vm-assignment-2/hw2-8_2.png)

排除 earlycon 自己的 initialization（`earlycon: pl11 at MMIO 0x...`）後，只有啟用 earlycon 時才會出現的兩行是：

- `printk: legacy bootconsole [pl11] enabled`
- `printk: legacy bootconsole [pl11] disabled`

前者出現在註冊 bootconsole 時；後者出現在真實 console（`hvc0`/`ttyAMA0`）接手並 handoff 或停用 bootconsole 時。沒有 earlycon 就完全不會有 bootconsole，所以兩行都不會出現。

### (d) 修正後的 cloud-hypervisor 是否仍能讓 Normal VM 使用 earlycon？

可以，畫面底部能看到 log。

![Normal VM 中正常運作的 earlycon](/assets/img/vm-assignment-2/hw2-9_1.png)

## Q5. Realm guest kernel：ioremap 路徑

### (a) 為什麼使用 MMIO 時需要 ioremap()？

既然已經在 kernel 中，為什麼不能直接存取裝置的 physical address？

MMU 啟用後，CPU 只會發出 virtual address。每次 load/store 都會把 virtual address 轉成 physical address，因此 kernel 不能直接 dereference physical address。Kernel 的 linear（direct）mapping 也只涵蓋一般 RAM；裝置 MMIO region 刻意不包含在內，所以一開始就沒有 kernel virtual address 指向裝置。

MMIO 還必須使用 device memory attribute，例如 non-cacheable，且不能 speculation、reordering 或 write-merging，不能使用一般 cacheable memory attribute。否則存取可能被 cache 或重新排序，破壞裝置 register semantics。`ioremap()` 同時解決兩個問題：它配置一段 kernel virtual address，並以正確 device attribute 建立指向裝置 physical region 的 page-table mapping，讓 kernel 得到可用的 pointer。

### (b) Realm guest kernel 的 ioremap 流程和 Normal VM 有何不同？

Normal VM 的 `ioremap()` 只會用 device attribute 把裝置映射到其 physical address。Realm guest 的 `ioremap()` 多做一步：對一般 shared MMIO，它會在 page-table mapping 中設定 shared（non-secure）bit，讓存取導向 guest IPA space 的 unprotected half，而不是 protected half。裝置位於 unprotected IPA 後，RMM 可以把存取轉送給 host 模擬，不會注入 fault。簡單來說，Realm flow 會在 mapping 加入 shared bit，Normal flow 不會；這就是 MMIO access 不會觸發 SEA 的原因。

### (c) Realm guest kernel 已處理 ioremap，為什麼 earlycon 仍會失敗？

Earlycon 完全不走 Realm-aware `ioremap()` 路徑。它很早就會替 UART 建立自己的 mapping，並照原樣映射 kernel command line 提供的 physical address，沒有執行 Realm `ioremap()` 原本會做的 shared-bit 步驟。因此，即使 Realm kernel 能替一般 driver 處理 shared bit，earlycon 的 mapping 仍指向 IPA space 的 protected half。

Earlycon 寫入 PL011 時，存取會落在 protected IPA，RMM 隨即向 Realm 注入 Synchronous External Abort，也就是反覆出現的 "Inject Sync EA"，所以 earlycon 失敗。修正必須放在 Cloud Hypervisor 的原因也在這裡：我們預先在 command line 的 `earlycon=` address 設定 shared bit，讓 earlycon 原樣建立的早期 mapping 也會落在 unprotected half。

## Q6. Host kernel：MMIO fault forwarding

### (a) 引用 host KVM 中取出 fault IPA 並分派給 MMIO emulation 的函式。

該函式是 `arch/arm64/kvm/mmu.c` 的 `kvm_handle_guest_abort`。它先用 `kvm_vcpu_get_fault_ipa()` 取得 fault IPA，判斷 IPA 位於沒有 memslot 的 device region 後，補回最低 12 bits，再交給 `io_mem_abort()` 進行 MMIO emulation：

```c
/*
 * The IPA is reported as [MAX:12], so we need to
 * complement it with the bottom 12 bits from the
 * faulting VA. This is always 12 bits, irrespective
 * of the page size.
 */
ipa |= kvm_vcpu_get_hfar(vcpu) & GENMASK(11, 0);
ret = io_mem_abort(vcpu, kvm_gpa_from_fault(vcpu->kvm, ipa));
goto out_unlock;
```

### (b) Host 使用哪個 register 取得 Stage-2 data abort 的 faulting IPA？

查看 `arch/arm64/include/asm/kvm_emulate.h` 的 `kvm_vcpu_get_fault_ipa()`：

![kvm_vcpu_get_fault_ipa 讀取 HPFAR_EL2](/assets/img/vm-assignment-2/hw2-10_1.png)

該 register 是 `HPFAR_EL2`（Hypervisor IPA Fault Address Register）。如圖所示，它取出 `hpfar_el2` 的 FIPA field，再向左移 `8` bits 以重建 IPA。

### (c) 哪個 KVM 函式把 PL011 access 轉送到 userspace Cloud Hypervisor？

該函式是 `arch/arm64/kvm/mmio.c` 的 `io_mem_abort`。它先解碼 syndrome，嘗試交給 in-kernel device（`kvm_io_bus_read/write`）處理。如果沒有裝置接手，也就是回傳非零值，就填入 `run->mmio`、設定 `exit_reason`，並返回 userspace 由 VMM 模擬：

```c
/*
 * Prepare MMIO operation. First decode the syndrome data we get
 * from the CPU. Then try if some in-kernel emulation feels
 * responsible, otherwise let user space do its magic.
 */
is_write = kvm_vcpu_dabt_iswrite(vcpu);
len = kvm_vcpu_dabt_get_as(vcpu);
rt = kvm_vcpu_dabt_get_rd(vcpu);

if (is_write) {
    data = vcpu_data_guest_to_host(vcpu, vcpu_get_reg(vcpu, rt),
                                   len);

    trace_kvm_mmio(KVM_TRACE_MMIO_WRITE, len, fault_ipa, &data);
    kvm_mmio_write_buf(data_buf, len, data);

    ret = kvm_io_bus_write(vcpu, KVM_MMIO_BUS, fault_ipa, len,
                           data_buf);
} else {
    trace_kvm_mmio(KVM_TRACE_MMIO_READ_UNSATISFIED, len,
                   fault_ipa, NULL);

    ret = kvm_io_bus_read(vcpu, KVM_MMIO_BUS, fault_ipa, len,
                          data_buf);
}
```

### (d) 延續 (c)，ioctl 的 `exit_reason` 是什麼？

`exit_reason` 是 `KVM_EXIT_MMIO`。從 `arch/arm64/kvm/mmio.c` 可見：

```c
if (!ret) {
    /* We handled the access successfully in the kernel. */
    if (!is_write)
        memcpy(run->mmio.data, data_buf, len);
    vcpu->stat.mmio_exit_kernel++;
    kvm_handle_mmio_return(vcpu);
    return 1;
}

if (is_write)
    memcpy(run->mmio.data, data_buf, len);
vcpu->stat.mmio_exit_user++;
run->exit_reason = KVM_EXIT_MMIO;
return 0;
```

### (e) Userspace VMM 收到的 faulting IPA，和 (b) 中 register 保存的值相同嗎？

如果不同，原因是什麼？

從 `arch/arm64/kvm/mmu.c` 可見：

- IPA：`ipa = fault_ipa = kvm_vcpu_get_fault_ipa(vcpu);`
- VMM 收到的值：`ipa |= kvm_vcpu_get_hfar(vcpu) & GENMASK(11, 0);`

HPFAR register 只回報對齊 page size 的 faulting address，因此最低 12 bits，也就是 bits `[11:0]`，都為 0。KVM 在把 IPA 交給 `io_mem_abort` 或 userspace 前，會從 faulting VA 取回最低 12 bits，再 OR 回去：`kvm_vcpu_get_hfar(vcpu) & GENMASK(11, 0)`。

`mmu.c` 的註解明確寫道："The IPA is reported as [MAX:12], so we need to complement it with the bottom 12 bits from the faulting VA. This is always 12 bits, irrespective of the page size." 因此，VMM 收到的 faulting IPA 和 (b) register 的 raw value 不同；差別就是補回的最低 12-bit offset。在這個案例中，earlycon 存取的 UARTFR offset 正好是 `0x018`。

---

[下載原始 PDF](/files/114-2-vm-assignment-2-writeup.pdf)

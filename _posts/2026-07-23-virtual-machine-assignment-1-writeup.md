---
layout: post
title: "Letting the guest control host vCPU affinity"
date: 2026-07-23 09:00:00 +0800
permalink: /projects/114-2-vm-assignment-1-writeup/
lang: en
translation_key: 114-2-vm-assignment-1-writeup
categories: [Coursework]
tags: [Project, Virtual Machines, AI Assisted]
published: true
---

[繁體中文版](/zh/projects/114-2-vm-assignment-1-writeup/)

This was an individual course assignment completed with step-by-step implementation guidance from Claude Code. I followed and learned the guest-to-HVC-to-KVM control path, then cross-checked the runtime affinity on the host. The technical writeup below preserves the submitted work and does not claim independently authored implementation.

2026 Spring · Virtual Machine (VM)<br>
Ming-Lung Tsai · CSIE, B12902078

## 1 Part 1 - Design & Implementation

### 1.1 Environment Setup

- Host: Ubuntu 22.04 LTS on x86
- QEMU: v7.0.0
- Linux/KVM: v5.15
- Disk images: Two Ubuntu 20.04 cloud images: `cloud.img` (25GB) for the KVM host filesystem, and `cloud_inner.img` (2GB, sized to fit inside `cloud.img`) for the nested VM.

### 1.2 Overall Architecture

The goal of this assignment is to allow a guest VM to pin its vCPU to a specific physical CPU on the KVM host. Achieving this requires modifications to KVM itself, along with a kernel module (`hw1_module`) running inside the guest that serves as the interface between user space and the hypervisor. To understand why this cannot be handled purely within the guest, the guest kernel has no visibility into these threads. Therefore, any affinity operation must cross the guest/host boundary, which means a VM exit is required.

The same reasoning applies to both directions. When the guest writes a cpumask, the module must trigger a VM exit so that KVM can apply `set_cpus_allowed_ptr()` to all vCPU kernel threads on the host side. When the guest reads the current cpumask, another VM exit is needed to retrieve the value from the host.

End-to-end flow (write): A user-space program opens `/dev/hw1_module` and issues a `write()` with an 8-bit cpumask. The kernel module receives this in `device_write()` and fires an HVC hypercall, which causes a VM exit and traps into EL2. KVM's hypercall handler dispatches to `kvm_assign_affinity()`, which iterates over all vCPUs via `kvm_for_each_vcpu` and calls `set_cpus_allowed_ptr()` on each vCPU kernel thread with the provided cpumask, pinning the entire VM to the specified physical CPUs.

End-to-end flow (read): A user-space program issues a `read()` on `/dev/hw1_module`. The kernel module fires an HVC hypercall via `device_read()`, trapping into EL2. KVM dispatches to `kvm_read_affinity()`, which reads `current->cpus_ptr` - where `current` is the vCPU kernel thread that handled this VM exit. Since `kvm_assign_affinity()` applies the same mask to all vCPUs, every vCPU thread carries an identical cpumask, so the returned value is always consistent regardless of which vCPU serves the hypercall.

### 1.3 KVM-Side Implementation

We begin by tracing the VM exit handling path to understand where to insert our implementation.

![KVM handle_exit implementation](/assets/img/vm-assignment-1/hw1-2_1.png)

When a VM exit occurs, KVM enters `handle_exit()`, which dispatches to `handle_trap_exception()`. This function calls `kvm_get_exit_handler()`, which returns a function pointer from `arm_exit_handlers[]` indexed by the ESR exception class. For HVC calls, this resolves to `handle_hvc()`, which ultimately calls `kvm_hvc_call_handler()`.

![KVM HVC call handler](/assets/img/vm-assignment-1/hw1-2_2.png)

```text
handle_exit() => handle_trap_exception() => kvm_get_exit_handler() =>
arm_exit_handlers[esr_ec] => handle_hvc() => kvm_hvc_call_handler()
```

`kvm_hvc_call_handler()` selects the target function based on `func_id`. As seen in `arm_hypercalls.h`, the function ID is read from the vCPU's register `x0`, which is also where the guest must place it before issuing the hypercall.

![SMCCC function ID is read from vCPU register x0](/assets/img/vm-assignment-1/hw1-3_1.png)

With this call chain understood, we make the following changes. Define hypercall `func_id` in `include/linux/arm-smccc.h`:

```c
#define ARM_SMCCC_ASSIGN_AFFINITY \
    ARM_SMCCC_CALL_VAL(ARM_SMCCC_FAST_CALL, ARM_SMCCC_SMC_64, \
                       ARM_SMCCC_OWNER_STANDARD, 0x54)
#define ARM_SMCCC_READ_AFFINITY \
    ARM_SMCCC_CALL_VAL(ARM_SMCCC_FAST_CALL, ARM_SMCCC_SMC_64, \
                       ARM_SMCCC_OWNER_STANDARD, 0x55)
```

Register handlers in `arch/arm64/kvm/hypercalls.c`:

```c
case ARM_SMCCC_ASSIGN_AFFINITY:
    return kvm_assign_affinity(vcpu);
case ARM_SMCCC_READ_AFFINITY:
    return kvm_read_affinity(vcpu);
```

Implement the handlers in a new file `arch/arm64/kvm/affinity.c`, also added to `arch/arm64/kvm/Makefile`:

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

`kvm_assign_affinity()` applies the cpumask to all vCPU kernel threads via `kvm_for_each_vcpu`, rather than only the thread currently handling the hypercall. This ensures that `kvm_read_affinity()` - which reads `current->cpus_ptr` from whichever vCPU thread serves the READ hypercall - always returns a consistent value, even if the guest process migrates across vCPUs between the two calls.

### 1.4 Guest Kernel Module

`hw1_module.c` implements the bridge between guest user space and the KVM hypercall interface. On `write()`, the module reads an 8-bit cpumask from user space and issues `ARM_SMCCC_ASSIGN_AFFINITY` via an inline HVC instruction, passing the mask in `x1` as required by the SMCCC calling convention. On `read()`, it issues `ARM_SMCCC_READ_AFFINITY` and copies the returned value from `x0` back to user space.

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

### 1.5 How to Build and Run

#### 1.5.1 Modified Files

| File | Change |
|---|---|
| `include/linux/arm-smccc.h` | Define `ARM_SMCCC_ASSIGN_AFFINITY` and `ARM_SMCCC_READ_AFFINITY` |
| `arch/arm64/kvm/hypercalls.c` | Add dispatch cases for the two new function IDs |
| `arch/arm64/kvm/affinity.c` | New file implementing `kvm_assign_affinity()` and `kvm_read_affinity()` |
| `arch/arm64/kvm/Makefile` | Add `affinity.o` to the build |

#### 1.5.2 Apply patch and rebuild kernel, kernel module

```bash
cd linux/
git apply b12902078_hw1_kernel.patch
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
make KDIR=/path/to/linux ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

#### 1.5.3 Boot KVM host and Transfer `hw1_module.ko` to KVM host

```bash
./run-kvm.sh -k arch/arm64/boot/Image -i cloud.img
scp -P 2222 hw1_module.ko root@localhost:/root
scp -P 2222 /path/to/Image root@localhost:/root
scp -P 2222 /path/to/cloud_inner.img root@localhost:/root
scp -P 2222 /path/to/test.c root@localhost:/root
```

#### 1.5.4 Move test, `hw1_module.ko` into `cloud_inner.img` (In KVM host)

```bash
gcc -static -o test test.c
sudo mount -o loop cloud_inner.img /mnt
sudo cp test /mnt/root/test
sudo cp hw1_module.ko /mnt/root/hw1_module.ko
sudo umount /mnt
```

#### 1.5.5 Run the Guest VM

```bash
./run-guest.sh -k Image -i cloud_inner.img
```

#### 1.5.6 Install Module and Run the test

```bash
insmod hw1_module.ko
./test
```

## 2 Part 2 - Experiment & Evaluation

### 2.1 Experimental Goal

The goal of this experiment is to verify that the affinity mechanism is correctly implemented on the KVM host - specifically, that writing a cpumask from inside the guest VM causes the corresponding vCPU kernel threads on the KVM host to be pinned to the specified physical CPU(s).

### 2.2 Setup

The KVM host is booted with 6 vCPUs via QEMU (`-smp 6`). The nested VM runs with 2 vCPU. The kernel module `hw1_module.ko` is compiled and installed inside the nested VM via `insmod`.

We use the following tools on the KVM host to observe vCPU thread scheduling:

- `ps -eLo pid,psr,comm` - shows which physical CPU each thread is currently running on (affinity)
- `taskset -p <pid>` - shows the configured `Cpus_allowed` mask of a thread (affinity)
- `journalctl -k -n 20` confirms that the hypercall reached EL2 and the correct cpumask was received

### 2.3 Procedure

#### 2.3.1 Install Module and Run the test

```bash
insmod hw1_module.ko
./test
```

#### 2.3.2 Identify the qemu vcpu

```bash
ps -eLo pid,psr,comm | grep qemu
htop
```

#### 2.3.3 Record the baseline affinity before pinning

```bash
taskset -p <vcpu thread_pid>
cat /proc/<vcpu thread_pid>/status | grep Cpus_allowed
```

#### 2.3.4 Run the test and while-loop on Guest VM

```bash
./test
while true; do :; done &
```

#### 2.3.5 Confirm the hypercall has been performed

```bash
journalctl -k -n 20
```

#### 2.3.6 Verify the affinity do apply on KVM host's CPUs

```bash
taskset -p <vCPU_thread_pid>
cat /proc/<vCPU_thread_pid>/status | grep Cpus_allowed
```

### 2.4 Results

#### 2.4.1 Identify the vcpu

![QEMU vCPU threads](/assets/img/vm-assignment-1/hw1-8_1.png)

![QEMU vCPU threads in htop](/assets/img/vm-assignment-1/hw1-8_2.png)

- Since the Guest VM has two vcpus (pthread), we can know the PID from the htop.

#### 2.4.2 Record the baseline affinity before pinning

![Baseline affinity of vCPU thread 991](/assets/img/vm-assignment-1/hw1-8_3.png)

![Baseline affinity of vCPU thread 992](/assets/img/vm-assignment-1/hw1-8_4.png)

#### 2.4.3 Run the test and while-loop on Guest VM

![Guest test changes the CPU mask from 3f to 05](/assets/img/vm-assignment-1/hw1-8_5.png)

#### 2.4.4 Confirm the hypercall has been performed

![Kernel journal showing KVM affinity hypercalls](/assets/img/vm-assignment-1/hw1-9_1.png)

#### 2.4.5 Verify the affinity do apply on KVM host's CPUs

![QEMU vCPU threads pinned to host CPUs](/assets/img/vm-assignment-1/hw1-9_2.png)

---

[Download the original PDF](/files/114-2-vm-assignment-1-writeup.pdf)

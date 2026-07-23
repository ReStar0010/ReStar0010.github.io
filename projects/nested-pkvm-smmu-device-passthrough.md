---
layout: page
title: "Device Passthrough Investigation & Implementation on SMMU-Supported pKVM"
permalink: /projects/nested-pkvm-smmu-device-passthrough/
published: true
---

```text
unassigned stream -> host identity map (VMID 0)
assigned stream   -> guest VMID + guest Stage-2
```

That was the change at the center of our Virtual Machines final project.

We started from the `pkvm-smmu-v5` development branch. Its hypervisor-owned SMMUv3 driver protected the host from device DMA by keeping the real stream table under EL2 control. Our project asked a different question: could the same foundation route an assigned device through a protected guest's Stage-2?

The absence of a guest-assignment interface was therefore not necessarily an omission in the original work. Host DMA isolation and guest device passthrough are different scopes. We treated the branch as a base and implemented the additional ownership, attachment, and teardown path required by the second.

The resulting prototype let EL2 assign a device stream to a protected guest's Stage-2, let the host drive that assignment through the standard passthrough stack, and revoked DMA before guest memory was reclaimed.

## The stack we were trying to cross

```text
L0: Arm server
└── QEMU 11.0.1 in pure TCG mode
    ├── emulated EL2
    ├── emulated GICv3
    └── emulated SMMUv3
        └── L1: Linux 6.18-rc5 pKVM host
            ├── VFIO / IOMMU
            └── L2 guest
                ├── Proof A: minimal protected pVM
                └── Path B: full Ubuntu guest + e1000e
```

The L0 machine did not support FEAT_NV2, so KVM could not accelerate a guest that itself needed EL2. The L0-to-L1 boundary therefore ran under QEMU TCG. It was slow, but it gave us the nested EL2 and SMMUv3 model needed to exercise the path.

## Why CPU isolation was not enough

pKVM places the hypervisor at EL2 and treats the host kernel as untrusted. CPU memory accesses are constrained by Stage-2 page tables owned by the hypervisor.

A device doing DMA does not pass through the CPU MMU. Without an IOMMU enforcing the same boundary, a passed-through device could write outside the guest, including into another guest or the hypervisor.

On Arm, SMMUv3 provides that translation layer. Each DMA-capable device has a StreamID. Its Stream Table Entry (STE) determines which translation context and page tables the SMMU uses. The core idea of the project was simple:

> When a device belongs to a protected guest, point its STE at that guest's Stage-2 instead of the host identity map.

The difficulty was making attach, detach, locking, TLB invalidation, and VM teardown preserve that statement at every moment.

## What the project produced

The implementation covered four connected pieces:

1. **Guest Stage-2 routing at EL2.** We extended the hypervisor-owned SMMUv3 driver so an assigned StreamID could use a protected guest's VMID, page-table root, and VTCR instead of the host identity map.
2. **An attach/detach interface.** We added SMCCC hypercalls and connected them to the standard Linux VFIO/IOMMUFD path, allowing ordinary device assignment to install or remove the EL2-owned mapping without changing the guest-visible device model.
3. **Lifecycle and isolation fixes.** We validated host-supplied StreamIDs, reconciled CPU and SMMU address-size limits, restricted guest-VMID invalidations, handled live reassignment, and detached streams before VM memory was reclaimed.
4. **A newer base and a reproducible environment.** We ported the mechanism onto the Linux 6.18-based pKVM SMMUv3 v5 branch and brought the nested stack up under QEMU with an emulated SMMUv3 and VFIO endpoint.

## My role: turning the design into a runnable experiment

My role was closest to a project manager and systems integrator. I kept the research question, implementation milestones, and validation plan pointed at the same target: showing where host DMA isolation ended and what protected guest passthrough still required.

I helped shape the experiment into two complementary paths instead of treating a successful boot or ping as sufficient evidence. Proof A isolated the security property by checking DMA through a protected guest's Stage-2. Path B exercised the systems path with a full Ubuntu guest, a real e1000e driver, and bidirectional traffic. Keeping those claims separate also made the unfinished production criterion explicit.

I personally brought up the nested environment: QEMU's emulated EL2, GICv3, and SMMUv3 at L0; the Linux pKVM host at L1; and the protected test VM and full Ubuntu guest at L2. Much of the work was integration debugging across boundaries — kernel configuration, VFIO modes, SMMU register access, nested BAR handling, and the difference between a control-plane trace and actual device I/O.

That role was less about owning one isolated patch and more about making the complete experiment executable, deciding what the next failure could prove, and keeping the final report honest about the boundary of the result.

## Three implementation decisions

### 1. Snapshot the guest translation context

Each device assignment records the guest's:

- VMID;
- Stage-2 page-table root;
- VTCR value describing the shape of that page table.

The reshadow path runs later while holding the SMMU lock. Looking the guest up again at that point would require reacquiring the VM-table lock in the opposite order. Snapshotting the three fields during attach keeps reshadow self-contained and preserves one lock order:

```text
VM table lock -> SMMU lock
```

### 2. Reuse the existing STE reshadow path

The hypervisor already rebuilt hardware STEs from the host's descriptors. We added one decision to that path:

```text
assignment exists  -> guest VMID + guest PGD + guest VTCR
no assignment      -> host identity map
```

Attach and detach therefore share one STE construction mechanism instead of maintaining two implementations that could drift apart.

### 3. Tear the DMA path down before freeing memory

Removing a software assignment record is not sufficient. The hardware STE and cached SMMU translations can still point at the old guest page tables.

Detach has to:

1. remove the assignment;
2. reshadow the STE back to the host mapping;
3. issue `TLBI_S12_VMALL` for the guest VMID.

VM teardown uses the same primitive in a strict order:

```text
block new attaches
-> detach every assigned device
-> reclaim the guest Stage-2 pages
```

Reclaiming the pages first would leave a stale-DMA window in which the device could write into memory that had already been reused.

## From host isolation to guest assignment

The original SMMUv3 path kept the host's DMA identity map under hypervisor control. Guest passthrough required a new ownership relationship: EL2 had to know which protected VM owned each StreamID and which Stage-2 configuration the SMMU should walk.

We added a small attach/detach hypercall interface for that purpose. A host-side bridge carries the protected VM handle and the device's streams from the ordinary VFIO/IOMMUFD attach path into EL2. The hypervisor records the assignment in private state and consults it whenever it reshadows the hardware STE:

```text
VFIO / IOMMUFD attach
-> protected VM handle + StreamID
-> EL2 assignment table
-> STE built from guest VMID, PGD, and VTCR
```

The guest still sees an ordinary assigned device. The host-visible IOMMU domain is programmed as usual for bookkeeping, but the untrusted host never gains direct control of the real SMMU stream table.

## Closing the path under QEMU

### Proof A: protected pVM DMA

The first experiment used an emulated `iommu-testdev`, bound it to `vfio-pci` in L1, and asked it to DMA `0x12345678` into a protected pVM page.

The pVM's own vCPU read the same value back:

```text
bench:iommu_testdev_dma n=10000 bytes=4096
                        avg_us=29.957 mib_s=130.4
pvm-passthrough: PASS device DMA reached the pVM
```

An L0 SMMU trace recorded:

- `CFGI_STE_RANGE` and `CFGI_CD` commands;
- a four-level Stage-2 page-table walk;
- VMID 1 attached to the translation;
- 10,000 successful translations;
- 9,999 IOTLB hits after the first miss.

This proved that pKVM programmed the emulated SMMUv3 and that the device's DMA was resolved through the protected guest's Stage-2.

### Path B: full-OS L2 device I/O

The second experiment passed an emulated e1000e NIC into a full Ubuntu L2 guest:

```text
PCI 0000:00:03.0 v=0x8086 cls=0x020000 drv=e1000e
e1000e 0000:00:03.0 eth1: NIC Link is Up 1000 Mbps
eth1 UP 10.0.3.15/24
4 packets transmitted, 4 received, 0% packet loss
```

Bringing up the full path exposed six independent integration problems:

1. enlarge the pKVM hypervisor IOMMU pool;
2. use `neoverse-n2` to avoid an incompatible S1PIE path;
3. enable the legacy VFIO container/type1 path after iommufd-cdev returned `EINVAL`;
4. expose read-only SMMU ID registers to prevent `IOMMU_GET_HW_INFO` from panicking the hypervisor;
5. retain `CONFIG_VFIO_DEVICE_CDEV`;
6. set `x-no-mmap=on` so BAR MMIO trapped through L1 QEMU instead of taking a nested direct-mmap path that could not be forwarded.

The final failure was especially useful. A plain L2 booted successfully, but the guest crashed at `__ew32 -> writel`, the first MMIO write to the passed-through BAR. Disabling direct mmap made that access trap through QEMU; the driver then completed its probe and the NIC worked.

The L0 trace showed an STE and Context Descriptor installed for the NIC's StreamID `0x20`. The ping demonstrated working RX and TX through the full guest driver stack.

## The boundary of the result

Proof A had the security property: a protected pVM and SMMU Stage-2 DMA enforcement. It did not run a full operating system and production device driver.

Path B had the system property: a full Ubuntu guest, PCI enumeration, a bound e1000e driver, and bidirectional network I/O. Its QEMU invocation did not request the protected private-memory VM type.

Together, the experiments covered the two sides of the project: protected DMA enforcement and a real full-OS driver path. They did not combine both properties in one guest.

The distinction is easier to see as a matrix:

| Experiment | Protected memory | Full OS | Real driver | SMMU trace | Bidirectional I/O |
|---|---:|---:|---:|---:|---:|
| Proof A | Yes | No | No | Yes | DMA write + vCPU readback |
| Path B | No | Yes | Yes | Control-plane evidence | Yes |
| Target experiment | Yes | Yes | Yes | Yes | Yes |

The missing third row is the production completion criterion. Our prototype supplied the mechanism, the host integration, the safety invariants, and evidence for both halves; it did not turn QEMU 11's full Ubuntu L2 into a protected private-memory pVM.

## Where this led

The technically interesting part was not that we eventually got a ping. It was extending a host-isolation mechanism into guest assignment while carrying one security invariant across every layer:

```text
assigned device DMA
-> EL2-owned STE
-> guest VMID and Stage-2
-> detach and TLBI
-> only then reclaim guest memory
```

That end-to-end way of reading a system now carries into my kvTZ-Trusty research. Reusing a security mechanism for a new scope means tracing the property through integration and teardown, not only showing that each component exists.

---

**Context:** CSIE 5310 Virtual Machines, Spring 2026 · Group Final Project

**Report:** [Download the final report](/files/vm-final-pkvm-smmu-report.pdf)

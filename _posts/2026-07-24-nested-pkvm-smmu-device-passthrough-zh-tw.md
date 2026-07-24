---
layout: post
title: "在支援 SMMU 的 pKVM 上研究與實作裝置直通"
date: 2026-07-24 00:01:00 +0800
permalink: /zh/projects/nested-pkvm-smmu-device-passthrough/
lang: zh-TW
translation_key: nested-pkvm-smmu-device-passthrough
categories: [Project]
tags: [Project, pKVM, SMMUv3, AI Assisted]
published: true
---

[English](/projects/nested-pkvm-smmu-device-passthrough/)

我們從 `pkvm-smmu-v5` 開發分支開始。該分支由 hypervisor 擁有的 SMMUv3 驅動把真實 stream table 留在 EL2 控制下，藉此防止裝置 DMA 侵犯 host。我們的專案問的是另一個問題：能否以相同基礎，讓指定裝置透過受保護 guest 的 Stage-2 進行轉譯？

因此，原始工作沒有 guest assignment 介面，不一定是疏漏。Host DMA 隔離和 guest 裝置直通的範圍不同。我們把該分支當作基礎，實作後者所需的額外所有權、attach 和 teardown 路徑。

最後的原型讓 EL2 能把裝置 stream 指派給受保護 guest 的 Stage-2，讓 host 透過標準 passthrough stack 驅動這項指派，並在回收 guest 記憶體前撤銷 DMA。

## 要跨越的系統堆疊

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

L0 機器不支援 FEAT_NV2，因此 KVM 無法加速本身也需要 EL2 的 guest。L0 到 L1 的邊界只能在 QEMU TCG 下執行。速度很慢，但它提供了測試這條路徑所需的 nested EL2 與 SMMUv3 模型。

## 為什麼只有 CPU 隔離還不夠

pKVM 把 hypervisor 放在 EL2，並把 host kernel 視為不可信任。CPU 記憶體存取受 hypervisor 擁有的 Stage-2 page table 限制。

裝置執行 DMA 時不會經過 CPU MMU。如果沒有 IOMMU 執行相同邊界，直通裝置就可能寫到 guest 外部，包括其他 guest 或 hypervisor。

在 Arm 上，SMMUv3 提供這層轉譯。每個能執行 DMA 的裝置都有 StreamID，其 Stream Table Entry（STE）決定 SMMU 要使用哪個轉譯 context 和 page table。專案的核心概念很直接：

> 當裝置屬於受保護 guest 時，讓它的 STE 指向該 guest 的 Stage-2，而不是 host identity map。

困難之處，是讓 attach、detach、locking、TLB invalidation 和 VM teardown 在每一刻都維持這項條件。

## 專案產出

實作涵蓋四個相連的部分：

1. **在 EL2 路由 guest Stage-2。** 我們擴充由 hypervisor 擁有的 SMMUv3 驅動，讓指定 StreamID 能使用受保護 guest 的 VMID、page-table root 和 VTCR，而不是 host identity map。
2. **Attach/detach 介面。** 我們加入 SMCCC hypercall，並連接到標準 Linux VFIO/IOMMUFD 路徑，讓一般裝置指派能安裝或移除 EL2 擁有的 mapping，而不必改變 guest 看到的裝置模型。
3. **生命週期與隔離修正。** 我們驗證 host 提供的 StreamID、協調 CPU 和 SMMU 的 address-size 限制、限制 guest VMID invalidation、處理執行中的重新指派，並在回收 VM 記憶體前 detach stream。
4. **較新的基礎與可重現環境。** 我們把機制移植到以 Linux 6.18 為基礎的 pKVM SMMUv3 v5 分支，並在 QEMU 下以模擬 SMMUv3 和 VFIO endpoint 啟動整個 nested stack。

## 我的角色：把設計變成可執行實驗

我在既有協作 kernel tree 上負責 guest Stage-2 passthrough 的移植與 self-test harness。這些實作放在更大的 prototype 裡，由我們一起設計、整合與除錯。

在團隊中，我的角色最接近專案管理與系統整合。我協助讓研究問題、實作里程碑和驗證計畫維持同一個目標：釐清 host DMA 隔離到哪裡為止，以及受保護 guest passthrough 還需要補上什麼。

我協助把實驗拆成兩條互補路徑，而不是把成功開機或 ping 當成充分證據。Proof A 透過受保護 guest 的 Stage-2 檢查 DMA，單獨驗證安全性質。Path B 使用完整 Ubuntu guest、真正的 e1000e 驅動和雙向流量，測試系統路徑。把兩種主張分開，也能清楚指出尚未完成的 production criterion。

我們一起啟動了 L0 的 QEMU 模擬 EL2、GICv3 與 SMMUv3，L1 的 Linux pKVM host，以及 L2 的受保護測試 VM 與完整 Ubuntu guest。這項工作需要跨越 kernel configuration、VFIO mode、SMMU register access、nested BAR handling，以及 control-plane trace 和真實裝置 I/O 的差異來除錯。

因此，我的貢獻是協作系統實驗的一部分：讓相關路徑和測試進入可執行狀態、判斷下一個失敗能證明什麼，並讓期末報告如實呈現成果邊界。

## 三項實作決策

### 1. 保存 guest translation context 的快照

每次裝置指派會記錄 guest 的：

- VMID；
- Stage-2 page-table root；
- 描述該 page table 結構的 VTCR 值。

Reshadow 路徑之後會在持有 SMMU lock 時執行。如果當下再次查找 guest，就必須以相反順序重新取得 VM-table lock。在 attach 時先保存三個欄位，能讓 reshadow 自給自足，並維持單一 lock order：

```text
VM table lock -> SMMU lock
```

### 2. 重用既有 STE reshadow 路徑

Hypervisor 原本就會依照 host descriptor 重建硬體 STE。我們在該路徑加入一個判斷：

```text
assignment exists  -> guest VMID + guest PGD + guest VTCR
no assignment      -> host identity map
```

Attach 和 detach 因此共用同一套 STE 建構機制，不必維護兩份可能逐漸分歧的實作。

### 3. 釋放記憶體前先拆除 DMA 路徑

只刪除軟體 assignment record 並不夠。硬體 STE 和快取中的 SMMU translation 仍可能指向舊的 guest page table。

Detach 必須：

1. 移除 assignment；
2. 把 STE reshadow 回 host mapping；
3. 對 guest VMID 發出 `TLBI_S12_VMALL`。

VM teardown 以嚴格順序使用相同 primitive：

```text
block new attaches
-> detach every assigned device
-> reclaim the guest Stage-2 pages
```

如果先回收 page，就會留下 stale-DMA window，裝置可能寫入已被重新使用的記憶體。

## 從 host isolation 到 guest assignment

原始 SMMUv3 路徑把 host 的 DMA identity map 留在 hypervisor 控制下。Guest passthrough 需要新的所有權關係：EL2 必須知道哪個受保護 VM 擁有各個 StreamID，以及 SMMU 應走訪哪組 Stage-2 configuration。

為此，我們加入一個小型 attach/detach hypercall 介面。Host-side bridge 從一般 VFIO/IOMMUFD attach 路徑，把受保護 VM handle 和裝置 stream 傳入 EL2。Hypervisor 在私有狀態中記錄指派，並在每次 reshadow 硬體 STE 時查詢：

```text
VFIO / IOMMUFD attach
-> protected VM handle + StreamID
-> EL2 assignment table
-> STE built from guest VMID, PGD, and VTCR
```

Guest 看到的仍是一般 assigned device。Host 可見的 IOMMU domain 會照常設定以供 bookkeeping，但不可信任的 host 不會取得真實 SMMU stream table 的直接控制權。

## 在 QEMU 下打通路徑

### Proof A：受保護 pVM DMA

第一個實驗使用模擬的 `iommu-testdev`，在 L1 把它綁定到 `vfio-pci`，並要求它把 `0x12345678` DMA 寫入受保護 pVM page。

pVM 自己的 vCPU 讀回了相同數值：

```text
bench:iommu_testdev_dma n=10000 bytes=4096
                        avg_us=29.957 mib_s=130.4
pvm-passthrough: PASS device DMA reached the pVM
```

L0 SMMU trace 記錄了：

- `CFGI_STE_RANGE` 與 `CFGI_CD` command；
- 四層 Stage-2 page-table walk；
- translation 使用 VMID 1；
- 10,000 次成功 translation；
- 第一次 miss 後有 9,999 次 IOTLB hit。

這證明 pKVM 已設定模擬 SMMUv3，而且裝置 DMA 確實透過受保護 guest 的 Stage-2 解析。

### Path B：完整作業系統的 L2 裝置 I/O

第二個實驗把模擬 e1000e NIC 直通到完整 Ubuntu L2 guest：

```text
PCI 0000:00:03.0 v=0x8086 cls=0x020000 drv=e1000e
e1000e 0000:00:03.0 eth1: NIC Link is Up 1000 Mbps
eth1 UP 10.0.3.15/24
4 packets transmitted, 4 received, 0% packet loss
```

打通完整路徑時，出現六個彼此獨立的整合問題：

1. 擴大 pKVM hypervisor IOMMU pool；
2. 使用 `neoverse-n2`，避開不相容的 S1PIE 路徑；
3. 在 iommufd-cdev 回傳 `EINVAL` 後，啟用舊版 VFIO container/type1 路徑；
4. 開放唯讀 SMMU ID register，避免 `IOMMU_GET_HW_INFO` 讓 hypervisor panic；
5. 保留 `CONFIG_VFIO_DEVICE_CDEV`；
6. 設定 `x-no-mmap=on`，讓 BAR MMIO 透過 L1 QEMU trap，而不是走無法轉送的 nested direct-mmap 路徑。

最後一個失敗特別有用。一般 L2 能順利開機，但 guest 在 `__ew32 -> writel`，也就是第一次寫入直通 BAR 的 MMIO 時崩潰。停用 direct mmap 後，該存取改為透過 QEMU trap；驅動隨後完成 probe，NIC 也能正常運作。

L0 trace 顯示 NIC 的 StreamID `0x20` 已安裝 STE 與 Context Descriptor。Ping 證明完整 guest driver stack 能正常收送封包。

## 成果邊界

Proof A 具備安全性質：受保護 pVM 和 SMMU Stage-2 DMA enforcement。但它沒有執行完整作業系統與 production device driver。

Path B 具備系統性質：完整 Ubuntu guest、PCI enumeration、已綁定的 e1000e driver 和雙向網路 I/O。但其 QEMU invocation 沒有要求 protected private-memory VM type。

兩個實驗共同涵蓋專案的兩面：protected DMA enforcement 和真實 full-OS driver path。它們沒有在同一個 guest 中結合兩種性質。

用矩陣看會更清楚：

| 實驗 | Protected memory | Full OS | Real driver | SMMU trace | Bidirectional I/O |
|---|---:|---:|---:|---:|---:|
| Proof A | Yes | No | No | Yes | DMA write + vCPU readback |
| Path B | No | Yes | Yes | Control-plane evidence | Yes |
| Target experiment | Yes | Yes | Yes | Yes | Yes |

缺少的第三列就是 production completion criterion。我們的原型提供了機制、host integration、safety invariant，以及兩個面向各自的證據；它沒有把 QEMU 11 的完整 Ubuntu L2 變成受保護 private-memory pVM。

## 後續影響

技術上真正有意思的，不是最後終於 ping 成功，而是把 host-isolation 機制擴充到 guest assignment，同時讓一項 security invariant 穿過每一層：

```text
assigned device DMA
-> EL2-owned STE
-> guest VMID and Stage-2
-> detach and TLBI
-> only then reclaim guest memory
```

這種端到端閱讀系統的方式，現在也延續到我的 kvTZ-Trusty 研究。把安全機制用到新的範圍，必須沿著 integration 和 teardown 追蹤其性質，不能只證明各元件存在。

---

**背景：** CSIE 5310 Virtual Machines，2026 春季 · 期末專案

**報告：** [下載期末報告](/files/vm-final-pkvm-smmu-report.pdf)

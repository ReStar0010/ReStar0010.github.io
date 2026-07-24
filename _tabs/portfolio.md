---
icon: fas fa-diagram-project
order: 1
title: Portfolio
---

<style>
  .pf .pf-r { position:absolute; width:1px; height:1px; opacity:0; pointer-events:none; }
  .pf-toggle { display:flex; gap:.4rem; margin:.2rem 0 1.6rem; }
  .pf-toggle label { cursor:pointer; font:600 .78rem/1.2 ui-monospace,"SF Mono",Menlo,monospace; padding:.4rem .85rem; border-radius:999px; border:1px solid var(--card-border-color,rgba(0,0,0,.15)); color:var(--text-muted-color,#7a7a85); user-select:none; transition:background .12s,color .12s; }
  #pf-en:checked ~ .pf-toggle label[for=pf-en],
  #pf-zh:checked ~ .pf-toggle label[for=pf-zh] { background:var(--link-color,#2a5db0); color:#fff; border-color:transparent; }
  #pf-en:focus-visible ~ .pf-toggle label[for=pf-en],
  #pf-zh:focus-visible ~ .pf-toggle label[for=pf-zh] { outline:2px solid var(--link-color,#2a5db0); outline-offset:2px; }
  .pf-item { margin:0 0 1.4rem; }
  .pf-item h3 { margin:0 0 .3rem; }
  .pf-item p { margin:0; color:var(--text-color); }
  .pf .zh { display:none; }
  #pf-zh:checked ~ .pf-item .en { display:none; }
  #pf-zh:checked ~ .pf-item .zh { display:block; }
</style>

<div class="pf">
  <input type="radio" name="pflang" id="pf-en" class="pf-r">
  <input type="radio" name="pflang" id="pf-zh" class="pf-r" checked>
  <div class="pf-toggle"><label for="pf-zh">中文</label><label for="pf-en">EN</label></div>
  <div class="pf-item">
    <h3><span class="en">Current Thesis Research: Confidential Computing / TrustZone Virtualization</span><span class="zh">目前的論文研究：機密運算／TrustZone 虛擬化</span></h3>
    <p class="en">At SSLab, I am investigating how AVF could virtualize Arm TrustZone on top of Google pKVM. The thesis is still taking shape, so my current work is narrowing the systems boundary and finding a testable question.</p>
    <p class="zh">我目前在 SSLab 研究如何透過 AVF，在 Google pKVM 上虛擬化 Arm TrustZone。題目還在收斂中；現階段的工作是釐清系統邊界，找出能實際驗證的研究問題。</p>
  </div>
  <div class="pf-item">
    <h3><span class="en">pKVM / SMMUv3 Device Passthrough</span><span class="zh">pKVM / SMMUv3 裝置直通</span></h3>
    <p class="en">Host DMA isolation stops short of assigning a device safely to a protected guest. We brought up the nested environment, and I helped turn that gap into a prototype that routes and revokes device DMA through the guest's Stage-2. <a href="/projects/nested-pkvm-smmu-device-passthrough/">Read the case study →</a></p>
    <p class="zh">主機端 DMA 隔離並不等於能把裝置安全交給受保護 VM。我們一起架起 nested 環境，而我協助把這個缺口做成 prototype：讓裝置 DMA 經過 guest Stage-2，並在 teardown 前撤銷。<a href="/zh/projects/nested-pkvm-smmu-device-passthrough/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Virtual Machine Assignments</span><span class="zh">虛擬機器作業</span></h3>
    <p class="en">I worked across the guest/host boundary twice: first by <a href="/projects/114-2-vm-assignment-1-writeup/">letting a guest control host vCPU affinity</a>, then by <a href="/projects/114-2-vm-assignment-2-writeup/">booting a Realm VM far enough to trace early console MMIO</a>.</p>
    <p class="zh">我做了兩次 guest／host 邊界上的實作：先<a href="/zh/projects/114-2-vm-assignment-1-writeup/">讓 guest 控制 host vCPU affinity</a>，再<a href="/zh/projects/114-2-vm-assignment-2-writeup/">啟動 Realm VM，追到 early console 的 MMIO 路徑</a>。</p>
  </div>
  <div class="pf-item">
    <h3>CouPro</h3>
    <p class="en">CouPro had to connect a campus coupon product to purchases that merchants could verify, without pretending every redemption was incremental. As CTO and product lead, my implementation work covered claim, sharing, redemption, and analytics paths; our field experiments recorded 239 users and 236 physical redemptions. <a href="/projects/coupro/">Read the case study →</a></p>
    <p class="zh">CouPro 要把校園優惠產品接到店家能核對的實體消費，同時不能把每次核銷都說成新增營收。我擔任 CTO 與產品負責人，實作範圍涵蓋領券、分享、核銷與分析流程；兩次場域實驗共記錄 239 名使用者與 236 次實體核銷。<a href="/zh/projects/coupro/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">N-Day CVE-2025-59366 Reproduction</span><span class="zh">N-Day CVE-2025-59366 復現</span></h3>
    <p class="en">We had a firmware patch and a disclosed AiCloud CVE, but not the data flow connecting path traversal to command injection. At TeamT5 Camp, I worked with two teammates across binary diffing, WebDAV path analysis, dynamic tracing, and isolated reproduction; the public record omits a usable payload. <a href="/en/projects/asus-aicloud-cve-2025-59366/">Read the security record →</a></p>
    <p class="zh">我們手上有 firmware patch 與已公開的 AiCloud CVE，卻還缺少把 path traversal 接到 command injection 的資料流。在 TeamT5 Camp，我和兩位隊友一起完成 binary diff、WebDAV 路徑分析、動態追蹤與隔離環境重現；公開版本不放可直接使用的 payload。<a href="/projects/asus-aicloud-cve-2025-59366/">閱讀資安紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">LLM Multi-Agent Bystander Study</span><span class="zh">LLM 多代理人旁觀者效應研究</span></h3>
    <p class="en">We expected larger LLM panels to diffuse responsibility, but 82 clean trials hit an intervention ceiling instead. I helped design the controlled group-size experiment; the study kept that null result separate from ten corrupted runs. <a href="/projects/llm-multi-agent-bystander-study/">Read the research record →</a></p>
    <p class="zh">我們原本預期 LLM panel 變大後會出現責任分散，但 82 次有效試驗反而全部碰到介入率天花板。我參與設計控制群體大小的實驗；研究結果則把 null result 與 10 次損壞的執行紀錄分開處理。<a href="/zh/projects/llm-multi-agent-bystander-study/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">YouBike Dispatch-Information Simulation</span><span class="zh">YouBike 調度資訊模擬</span></h3>
    <p class="en">Could riders save time if existing rebalancing locations were visible, with no extra bikes added? I proposed the question and helped formalize the comparison; our 76-station simulation estimated a 1.8–2.4 minute saving under its measured shortage level. <a href="/projects/youbike-dispatch-information-simulation/">Read the research record →</a></p>
    <p class="zh">如果不增加任何車，只公開既有調度位置，使用者能省多少時間？我提出題目並協助團隊形式化比較；76 站模擬在量得的缺車程度下，估計可省 1.8–2.4 分鐘。<a href="/zh/projects/youbike-dispatch-information-simulation/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Lore</h3>
    <p class="en">LINE groups lose decisions and promises inside chat history, so my implementation work focused on a private bot that turns messages into retrievable facts, digests, and reminders. Replay tests cover the workflow, but no real group has used it long enough to validate the product. <a href="/projects/lore-group-memory/">Read the project record →</a></p>
    <p class="zh">LINE 群組裡的決定與承諾很容易沉進聊天紀錄，所以我的實作集中在一個私人 bot，把訊息整理成可查詢的事實、摘要與提醒。流程已有 replay tests，但還沒有真實群組長期使用的驗證。<a href="/zh/projects/lore-group-memory/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Legacy WordPress Operations</span><span class="zh">老 WordPress 行政流程自動化</span></h3>
    <p class="en">In a de-identified public-sector job, one old WordPress site split updates across REST, browser-only forms, and FTP. I mapped those boundaries with AI assistance and turned a roughly 90-minute first attempt into a repeatable, round-trip-verified workflow that later ran in about three minutes per item. <a href="/en/projects/legacy-wordpress-ai-operations/">Read the project record →</a></p>
    <p class="zh">一份去識別化的公部門小打工，把同一個老 WordPress 網站的更新拆散在 REST、瀏覽器表單與 FTP。我用 AI 摸清介面邊界，把第一次約 90 分鐘的操作整理成可重跑、能 round-trip 驗證的流程，後來單筆約三分鐘。<a href="/projects/legacy-wordpress-ai-operations/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>FurnitureStyle</h3>
    <p class="en">One prompt was being asked to recognize furniture and rewrite its color at the same time, which hid the intermediate result. My implementation work split the path into two model calls and also covered profiles, favorites, and data-model changes; the prototype was not evaluated for recommendation quality. <a href="/projects/furniturestyle/">Read the project record →</a></p>
    <p class="zh">原本一個 prompt 同時負責辨識家具與改寫色彩，兩個問題糾纏在一起。我的實作把流程拆成兩次模型呼叫，也涵蓋 profile、收藏與資料模型；這個 prototype 尚未驗證推薦品質。<a href="/zh/projects/furniturestyle/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Adventure Cube</h3>
    <p class="en">The live children's-story app never shipped, while its batch story and TTS pipeline kept working. I designed the overall architecture and contributed testing, prompt material, TTS experiments, audio integration, and the handover. <a href="/projects/adventure-cube/">Read the project record →</a></p>
    <p class="zh">兒童故事 app 沒有正式上線，但批次故事與 TTS pipeline 留了下來。我設計整體架構，也參與測試、prompt 素材、TTS 實驗、音訊整合與交接。<a href="/zh/projects/adventure-cube/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">US Stock-Market NLP</span><span class="zh">美國股市 NLP 預測</span></h3>
    <p class="en">Can MD&amp;A language from 342 annual reports predict the next year's financial direction? I worked on HAC and RNN experiments, the proposal, and the shared ensemble; the reported “non-fail rate” was not a trading return. <a href="/projects/us-stock-market-nlp/">Read the research record →</a></p>
    <p class="zh">342 份年報的 MD&amp;A，能不能預測隔年財務指標方向？我負責 HAC、RNN 實驗與 proposal，也共同處理 ensemble；報告裡的「non-fail rate」不是投資報酬。<a href="/zh/projects/us-stock-market-nlp/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">NTU Homeless Service Club</span><span class="zh">台大無家者服務社</span></h3>
    <p class="en">As vice president for the fifth and sixth terms, I worked with NGO partners and helped produce the club's first journal. I also handled curation and outreach for local markets.</p>
    <p class="zh">我在第 5、6 屆擔任副社長，與 NGO 夥伴協作，也參與完成社團首刊。我另外負責市集策展與對外推廣。</p>
  </div>
</div>

<!--
  新增一個專案：在 .pf 那個 </div> 收尾「之前」，複製一段：
    <div class="pf-item">
      <h3>Project Name</h3>
      <p class="en">English one-line description.</p>
      <p class="zh">中文一行描述。</p>
    </div>
  ▸ EN / 中文 由上方按鈕切換；預設中文。
  ▸ 專案名稱不需翻譯時可直接寫 <h3>；需要翻譯時在標題內分別使用 .en / .zh。
  ▸ 想寫完整長文時，把描述換成連結：<a href="/projects/xxx/">Read more →</a>
    （單一 project 頁模板：projects/example-project.md）
-->

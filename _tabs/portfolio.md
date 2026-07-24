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
    <h3><span class="en">Current Thesis Research: Confidential Computing / TrustZone Virtualization</span><span class="zh">目前的論文研究：Confidential Computing / TrustZone Virtualization</span></h3>
    <p class="en">At SSLab, I am researching how to virtualize Arm TrustZone on Google pKVM through AVF, with the goal of successfully deploying it on an Android device.</p>
    <p class="zh">我目前在 SSLab 研究如何透過 AVF，在 Google pKVM 上虛擬化 Arm TrustZone，並希望能夠成功部署到 Android 裝置上。</p>
  </div>
  <div class="pf-item">
    <h3>Device Passthrough</h3>
    <p class="en">We brought up nested pKVM with an emulated SMMUv3 and built a device-passthrough prototype that routes DMA through the protected guest's Stage-2, then revokes those mappings before teardown. <a href="/projects/nested-pkvm-smmu-device-passthrough/">Read the case study →</a></p>
    <p class="zh">我們架起含模擬 SMMUv3 的 nested pKVM 環境，做出 Device Passthrough prototype，讓裝置 DMA 經過 protected guest 的 Stage-2，並在 teardown 前撤銷相關 mapping。<a href="/zh/projects/nested-pkvm-smmu-device-passthrough/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Virtual Machines</span><span class="zh">虛擬機器</span></h3>
    <p class="en"><a href="/projects/114-2-vm-assignment-1-writeup/">Let a guest control host vCPU affinity</a>, then <a href="/projects/114-2-vm-assignment-2-writeup/">booted a Realm VM and traced the early console MMIO path</a>.</p>
    <p class="zh"><a href="/zh/projects/114-2-vm-assignment-1-writeup/">讓 guest 控制 host vCPU affinity</a>、<a href="/zh/projects/114-2-vm-assignment-2-writeup/">啟動 Realm VM，追到 early console 的 MMIO 路徑</a>。</p>
  </div>
  <div class="pf-item">
    <h3>CouPro</h3>
    <p class="en">CouPro is a two-sided campus deals platform where unused coupons can circulate and become transaction data that merchants can track. As CTO and product lead, I built the claim, sharing, redemption, and analytics flows; our team visited merchants and tested the product with students to see whether both sides would actually use it. <a href="/projects/coupro/">Read the case study →</a></p>
    <p class="zh">CouPro 是校園優惠雙邊平台，讓用不到的優惠券繼續流通，也把領取、分享與核銷變成商家能追蹤的數據。我擔任 CTO 與產品負責人，實作領券、分享、核銷與分析流程；團隊則拜訪商家並找學生測試，看雙方是否真的願意使用。<a href="/zh/projects/coupro/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">N-Day CVE-2025-59366 Reproduction</span><span class="zh">N-Day CVE-2025-59366 復現</span></h3>
    <p class="en">We reconstructed the attack chain of a disclosed AiCloud CVE, tracing how path traversal reached command injection. I worked on firmware diffing, WebDAV path analysis, dynamic tracing, and isolated reproduction; the public record leaves out a usable payload. <a href="/en/projects/asus-aicloud-cve-2025-59366/">Read the security record →</a></p>
    <p class="zh">我們重建了已公開 AiCloud CVE 的攻擊鏈，追出 path traversal 如何一路進入 command injection。我參與 firmware diff、WebDAV 路徑分析、動態追蹤與隔離環境重現；公開版本不放可直接使用的 payload。<a href="/projects/asus-aicloud-cve-2025-59366/">閱讀資安紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">LLM Multi-Agent Bystander Study</span><span class="zh">LLM 多代理人旁觀者效應研究</span></h3>
    <p class="en">We tested whether larger LLM panels would diffuse responsibility, but all 82 valid trials hit the same intervention ceiling. I helped design the controlled group-size experiment; the study kept that null result separate from ten corrupted runs. <a href="/projects/llm-multi-agent-bystander-study/">Read the research record →</a></p>
    <p class="zh">我們測試 LLM panel 變大後是否會出現責任分散，但 82 次有效試驗全都碰到相同的介入率天花板。我參與設計控制群體大小的實驗；研究則把這個 null result 與 10 次損壞的執行紀錄分開。<a href="/zh/projects/llm-multi-agent-bystander-study/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">YouBike Dispatch-Information Simulation</span><span class="zh">YouBike 調度資訊模擬</span></h3>
    <p class="en">We simulated whether showing riders existing rebalancing locations could reduce travel without adding bikes. I proposed the question and helped formalize the comparison; across 76 stations, the model estimated a 1.8–2.4 minute saving at the measured shortage level. <a href="/projects/youbike-dispatch-information-simulation/">Read the research record →</a></p>
    <p class="zh">我們模擬在不增加車輛的情況下，公開既有調度位置能否縮短使用者的移動時間。我提出題目並協助團隊形式化比較；76 站模型在量得的缺車程度下，估計可省 1.8–2.4 分鐘。<a href="/zh/projects/youbike-dispatch-information-simulation/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Lore</h3>
    <p class="en">Lore turns LINE messages into searchable group memory, digests, and reminders. I implemented the private bot and replay-tested its workflow; it has not yet been validated through long-term use by a real group. <a href="/projects/lore-group-memory/">Read the project record →</a></p>
    <p class="zh">Lore 把 LINE 訊息整理成可查詢的群組記憶、摘要與提醒。我實作這個私人 bot，並用 replay tests 驗證流程；目前還沒有真實群組長期使用。<a href="/zh/projects/lore-group-memory/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Legacy WordPress Operations</span><span class="zh">老 WordPress 行政流程自動化</span></h3>
    <p class="en">With AI assistance, I mapped a de-identified legacy WordPress workflow split across REST, browser-only forms, and FTP, then automated it with round-trip checks. A roughly 90-minute first attempt became a repeatable process that later ran in about three minutes per item. <a href="/en/projects/legacy-wordpress-ai-operations/">Read the project record →</a></p>
    <p class="zh">我用 AI 協助拆解一個去識別化的老 WordPress 更新流程，串起 REST、瀏覽器表單與 FTP，再加入 round-trip 檢查。第一次約 90 分鐘的操作，後來整理成單筆約三分鐘、可重跑的流程。<a href="/projects/legacy-wordpress-ai-operations/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>FurnitureStyle</h3>
    <p class="en">FurnitureStyle turns a furniture image into a structured description that can drive product search. I split recognition and color rewriting into separate model calls, then implemented profiles, favorites, and the supporting data-model changes. <a href="/projects/furniturestyle/">Read the project record →</a></p>
    <p class="zh">FurnitureStyle 把家具圖片轉成結構化描述，再用它搜尋商品。我把辨識與色彩改寫拆成兩次模型呼叫，也完成 profile、收藏與相關資料模型。<a href="/zh/projects/furniturestyle/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Adventure Cube</h3>
    <p class="en">Adventure Cube is a batch pipeline that generates children's stories, normalizes the text for Taiwanese usage, and produces narrated audio. I designed the architecture, experimented with prompts and TTS, integrated audio, and prepared the handover. <a href="/projects/adventure-cube/">Read the project record →</a></p>
    <p class="zh">Adventure Cube 是一條批次流程，產生兒童故事、統一臺灣用語，再輸出旁白音檔。我設計整體架構，測試 prompt 與 TTS、整合音訊並準備交接。<a href="/zh/projects/adventure-cube/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">US Stock-Market NLP</span><span class="zh">美國股市 NLP 預測</span></h3>
    <p class="en">We used MD&amp;A text from 342 annual reports to predict the next year's financial direction. I worked on the HAC and RNN experiments, wrote the proposal, and helped combine the models into an ensemble. <a href="/projects/us-stock-market-nlp/">Read the research record →</a></p>
    <p class="zh">我們用 342 份年報的 MD&amp;A 預測隔年財務指標方向。我負責 HAC、RNN 實驗與 proposal，也參與把模型整合成 ensemble。<a href="/zh/projects/us-stock-market-nlp/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">NTU Homeless Service Club</span><span class="zh">台大無家者服務社</span></h3>
    <p class="en">As vice president for the fifth and sixth terms, I worked with NGO partners and helped publish the club's first journal. I also curated local markets and handled outreach.</p>
    <p class="zh">我在第 5、6 屆擔任副社長，與 NGO 夥伴合作並參與社團首刊的出版，也負責市集策展與對外推廣。</p>
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

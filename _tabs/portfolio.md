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
    <p class="en">This thesis at SSLab explores virtualizing Arm TrustZone on Google pKVM through AVF, with the goal of running it on a physical Android device.</p>
    <p class="zh">這項論文研究在 SSLab 探索如何透過 AVF，在 Google pKVM 上虛擬化 Arm TrustZone，目標是部署到實體 Android 裝置。</p>
  </div>
  <div class="pf-item">
    <h3>Device Passthrough</h3>
    <p class="en">The Device Passthrough prototype brings an emulated SMMUv3 into nested pKVM, routing DMA through the protected guest's Stage-2 and revoking those mappings before teardown. <a href="/projects/nested-pkvm-smmu-device-passthrough/">Read the case study →</a></p>
    <p class="zh">Device Passthrough prototype 把模擬 SMMUv3 帶進 nested pKVM，讓裝置 DMA 經過 protected guest 的 Stage-2，並在 teardown 前撤銷相關 mapping。<a href="/zh/projects/nested-pkvm-smmu-device-passthrough/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Virtual Machines</span><span class="zh">虛擬機器</span></h3>
    <p class="en">This virtualization work <a href="/projects/114-2-vm-assignment-2-writeup/">boots a Realm VM far enough to trace early-console MMIO across the guest/host boundary</a>. A second experiment <a href="/projects/114-2-vm-assignment-1-writeup/">lets the guest control host vCPU affinity</a>.</p>
    <p class="zh">這組虛擬化實作<a href="/zh/projects/114-2-vm-assignment-2-writeup/">啟動 Realm VM，追蹤 early-console MMIO 如何跨越 guest／host 邊界</a>；另一項實作則<a href="/zh/projects/114-2-vm-assignment-1-writeup/">讓 guest 控制 host vCPU affinity</a>。</p>
  </div>
  <div class="pf-item">
    <h3>CouPro</h3>
    <p class="en">CouPro keeps unused campus coupons moving and turns every claim, share, and redemption into data merchants can track. Across two campus tests, the team's June 2026 pitch recorded 239 registered users and 236 physical redemptions. <a href="/projects/coupro/">Read the case study →</a></p>
    <p class="zh">CouPro 讓用不到的校園優惠券繼續流通，也把每次領取、分享與核銷變成商家能追蹤的數據。團隊 2026 年 6 月的簡報記錄了兩次校園測試，共 239 名註冊使用者與 236 次實體核銷。<a href="/zh/projects/coupro/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">N-Day CVE-2025-59366 Reproduction</span><span class="zh">N-Day CVE-2025-59366 復現</span></h3>
    <p class="en">This reproduction uses firmware diffing, WebDAV analysis, and dynamic tracing to reconstruct how path traversal reaches command injection in a disclosed AiCloud CVE. <a href="/en/projects/asus-aicloud-cve-2025-59366/">Read the security record →</a></p>
    <p class="zh">這次復現重建了已公開 AiCloud CVE 的攻擊鏈，透過 firmware diff、WebDAV 分析與動態追蹤，找出 path traversal 如何一路進入 command injection。<a href="/projects/asus-aicloud-cve-2025-59366/">閱讀資安紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">LLM Multi-Agent Bystander Study</span><span class="zh">LLM 多代理人旁觀者效應研究</span></h3>
    <p class="en">The study tested whether larger LLM panels diffuse responsibility. All 82 valid trials hit the same intervention ceiling; the expected bystander effect did not appear. <a href="/projects/llm-multi-agent-bystander-study/">Read the research record →</a></p>
    <p class="zh">這項研究測試 LLM panel 變大後是否會出現責任分散。82 次有效試驗全都碰到相同的介入率天花板，原先預期的旁觀者效應並未出現。<a href="/zh/projects/llm-multi-agent-bystander-study/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">YouBike Dispatch-Information Simulation</span><span class="zh">YouBike 調度資訊模擬</span></h3>
    <p class="en">The simulation asks whether better information can save time without adding a single bike. Across 76 stations, showing existing rebalancing locations reduced estimated mean arrival time by 1.8–2.4 minutes at the measured shortage level. <a href="/projects/youbike-dispatch-information-simulation/">Read the research record →</a></p>
    <p class="zh">這項模擬測試能否只靠更好的資訊，在不增加任何車輛的情況下節省時間。76 站模型顯示，公開既有調度位置可讓估計移動時間減少 1.8–2.4 分鐘。<a href="/zh/projects/youbike-dispatch-information-simulation/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Lore</h3>
    <p class="en">Lore is a private, replay-tested LINE bot prototype that turns messages into searchable facts, scheduled digests, and reminders. Real-group validation is still ahead. <a href="/projects/lore-group-memory/">Read the project record →</a></p>
    <p class="zh">Lore 是一個經過 replay tests 的私人 LINE bot prototype，把訊息整理成可查詢的事實、定期摘要與提醒；真實群組驗證仍是下一步。<a href="/zh/projects/lore-group-memory/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Legacy WordPress Operations</span><span class="zh">老 WordPress 行政流程自動化</span></h3>
    <p class="en">This AI-assisted workflow connects a legacy WordPress site's REST API, browser-only forms, and FTP updates with round-trip checks. A roughly 90-minute first attempt became a repeatable process that runs in about three minutes per item. <a href="/en/projects/legacy-wordpress-ai-operations/">Read the project record →</a></p>
    <p class="zh">這套 AI 輔助流程串起老 WordPress 的 REST API、瀏覽器表單與 FTP 更新，再用 round-trip 檢查結果。第一次約 90 分鐘的操作，後來縮成單筆約三分鐘、可重跑的流程。<a href="/projects/legacy-wordpress-ai-operations/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>FurnitureStyle</h3>
    <p class="en">FurnitureStyle turns one furniture photo into a structured description that can drive product search. Separating recognition from color rewriting keeps each model step visible and easier to control. <a href="/projects/furniturestyle/">Read the project record →</a></p>
    <p class="zh">FurnitureStyle 把一張家具照片轉成能直接搜尋商品的結構化描述。辨識與色彩改寫分成兩次模型呼叫，讓每一步的結果都看得見，也更容易控制。<a href="/zh/projects/furniturestyle/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Adventure Cube</h3>
    <p class="en">Adventure Cube turns batches of generated children's stories into narration ready for Taiwanese listeners. The workflow cleans the text, normalizes local wording, then produces TTS audio. <a href="/projects/adventure-cube/">Read the project record →</a></p>
    <p class="zh">Adventure Cube 把批次生成的兒童故事做成適合臺灣聽眾的旁白。流程先清理文字、統一臺灣用語，再輸出 TTS 音檔。<a href="/zh/projects/adventure-cube/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">US Stock-Market NLP</span><span class="zh">美國股市 NLP 預測</span></h3>
    <p class="en">This NLP project tests whether the MD&amp;A sections of 342 annual reports can predict the next year's financial direction, comparing HAC, RNN, SVM, and Naive Bayes experiments. <a href="/projects/us-stock-market-nlp/">Read the research record →</a></p>
    <p class="zh">這個 NLP 專案測試 342 份年報的 MD&amp;A，能否預測隔年財務指標方向，並比較 HAC、RNN、SVM 與 Naive Bayes 實驗。<a href="/zh/projects/us-stock-market-nlp/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">NTU Homeless Service Club</span><span class="zh">台大無家者服務社</span></h3>
    <p class="en">During Stanley's fifth and sixth terms as vice president, the NTU Homeless Service Club worked with NGO partners and published its first journal. The club paired on-campus curation with outreach at local markets.</p>
    <p class="zh">Stanley 在第 5、6 屆擔任副社長期間，台大無家者服務社與 NGO 夥伴合作，並出版社團首刊；社團也結合校內策展與在地市集推廣。</p>
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

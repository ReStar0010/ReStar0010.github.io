---
icon: fas fa-diagram-project
order: 1
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
  <input type="radio" name="pflang" id="pf-en" class="pf-r" checked>
  <input type="radio" name="pflang" id="pf-zh" class="pf-r">
  <div class="pf-toggle"><label for="pf-en">EN</label><label for="pf-zh">中文</label></div>
  <div class="pf-item">
    <p class="en">Selected projects, course research, and work in progress. Each record states what the current evidence can support.</p>
    <p class="zh">這裡整理已完成、仍在進行，以及課程中的專案；每一筆都只寫目前證據能支持的內容。</p>
  </div>
  <div class="pf-item">
    <h3>CouPro</h3>
    <p class="en">A localized, two-sided deals platform centered on campus life (NTU CEP, 18th cohort). <a href="/projects/coupro/">Read the case study →</a></p>
    <p class="zh">以校園為中心的在地化優惠雙邊平台（台大創創學程 CEP 第 18 屆）。<a href="/projects/coupro/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3>LLM Multi-Agent Bystander Study</h3>
    <p class="en">A pre-registered panel experiment that retained a ceiling-limited null result and its corrupted-run boundary. <a href="/projects/llm-multi-agent-bystander-study/">Read the research record →</a></p>
    <p class="zh">預先註冊的多代理人 panel 實驗；保留 ceiling-limited null result 與 corrupted runs 的分析邊界。<a href="/projects/llm-multi-agent-bystander-study/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>YouBike Dispatch-Information Simulation</h3>
    <p class="en">A 76-station simulation isolating the time value of disclosing existing rebalancing locations. <a href="/projects/youbike-dispatch-information-simulation/">Read the research record →</a></p>
    <p class="zh">以 76 個站點的模擬，估計公開既有調度位置資訊能減少多少抵達時間。<a href="/projects/youbike-dispatch-information-simulation/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>US Stock-Market NLP</h3>
    <p class="en">A five-person study comparing text models over MD&amp;A sections from 342 annual reports. <a href="/projects/us-stock-market-nlp/">Read the research record →</a></p>
    <p class="zh">五人團隊以 342 份年報的 MD&amp;A 文字比較多種預測模型。<a href="/projects/us-stock-market-nlp/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>ASUS AiCloud Patch-Diff Analysis</h3>
    <p class="en">A TeamT5 Camp reproduction of a disclosed AiCloud vulnerability using firmware diffing and dynamic tracing. <a href="/projects/asus-aicloud-cve-2025-59366/">Read the security record →</a></p>
    <p class="zh">TeamT5 Camp 小組以 firmware diff 與動態追蹤重現已公開的 AiCloud 漏洞。<a href="/projects/asus-aicloud-cve-2025-59366/">閱讀資安紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>FurnitureStyle</h3>
    <p class="en">A Django prototype that turns an image or description into structured furniture attributes and shopping queries. <a href="/projects/furniturestyle/">Read the project record →</a></p>
    <p class="zh">把圖片或描述轉成家具屬性與購物搜尋條件的 Django prototype。<a href="/projects/furniturestyle/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Lore</h3>
    <p class="en">A private LINE bot prototype for group memory, Q&amp;A, digests, and reminders. <a href="/projects/lore-group-memory/">Read the project record →</a></p>
    <p class="zh">處理群組記憶、問答、摘要與提醒的私人 LINE bot prototype。<a href="/projects/lore-group-memory/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Adventure Cube</h3>
    <p class="en">A team children's-story project whose active surface is now a batch story and TTS pipeline. <a href="/projects/adventure-cube/">Read the project record →</a></p>
    <p class="zh">兒童故事團隊專案；目前可運作的部分是批次故事與 TTS pipeline。<a href="/projects/adventure-cube/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Confidential Computing / TrustZone Virtualization</h3>
    <p class="en">Research at SSLab: virtualizing Arm TrustZone with AVF (related to Google pKVM).</p>
    <p class="zh">SSLab 研究：用 AVF 把 Arm TrustZone 虛擬化（與 Google pKVM 相關）。</p>
  </div>
  <div class="pf-item">
    <h3>Virtual Machine</h3>
    <p class="en"><a href="/projects/114-2-vm-assignment-1-writeup/">Let a guest control host vCPU affinity</a> · <a href="/projects/114-2-vm-assignment-2-writeup/">Boot a Realm VM with early console output</a></p>
    <p class="zh"><a href="/projects/114-2-vm-assignment-1-writeup/">讓 guest 控制 host vCPU affinity</a> · <a href="/projects/114-2-vm-assignment-2-writeup/">啟動 Realm VM 並輸出 early console</a></p>
  </div>
  <div class="pf-item">
    <h3>pKVM / SMMUv3 Device Passthrough</h3>
    <p class="en">Extending host DMA isolation into protected guest assignment. <a href="/projects/nested-pkvm-smmu-device-passthrough/">Read the case study →</a></p>
    <p class="zh">把 host DMA isolation 延伸到 protected guest device assignment。<a href="/projects/nested-pkvm-smmu-device-passthrough/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3>NTU Homeless Service Club</h3>
    <p class="en">Vice president (5th &amp; 6th terms): NGO partnerships, the club's first journal, and curation &amp; outreach at local markets.</p>
    <p class="zh">第 5、6 屆副社長：NGO 培力串聯、社團首刊、市集策展與行銷。</p>
  </div>
</div>

<!--
  新增一個專案：在 .pf 那個 </div> 收尾「之前」，複製一段：
    <div class="pf-item">
      <h3>Project Name</h3>
      <p class="en">English one-line description.</p>
      <p class="zh">中文一行描述。</p>
    </div>
  ▸ EN / 中文 由上方按鈕切換；預設 EN（想預設中文：把 checked 從 #pf-en 移到 #pf-zh）。
  ▸ 標題 <h3> 中英共用，只有描述會隨語言切換。
  ▸ 想寫完整長文時，把描述換成連結：<a href="/projects/xxx/">Read more →</a>
    （單一 project 頁模板：projects/example-project.md）
-->

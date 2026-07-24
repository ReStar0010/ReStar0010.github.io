---
icon: fas fa-diagram-project
order: 1
title: 作品集
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
    <p class="en">Selected projects, course research, and work in progress. Each record states what the current evidence can support.</p>
    <p class="zh">這裡收錄已完成、正在進行，以及課堂上的專案。每一筆只寫目前有證據支持的內容。</p>
  </div>
  <div class="pf-item">
    <h3>CouPro</h3>
    <p class="en">A localized, two-sided deals platform centered on campus life (NTU CEP, 18th cohort). <a href="/projects/coupro/">Read the case study →</a></p>
    <p class="zh">以校園為中心的在地化優惠雙邊平台（台大創創學程 CEP 第 18 屆）。<a href="/projects/coupro/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">LLM Multi-Agent Bystander Study</span><span class="zh">LLM 多代理人旁觀者效應研究</span></h3>
    <p class="en">A pre-registered panel experiment that retained a ceiling-limited null result and its corrupted-run boundary. <a href="/projects/llm-multi-agent-bystander-study/">Read the research record →</a></p>
    <p class="zh">我參與設計多代理人 panel 的實驗結構，測試群體變大時，LLM 是否更少出聲反對。結果碰到天花板效應，沒有觀察到原先預期的差異。<a href="/projects/llm-multi-agent-bystander-study/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">YouBike Dispatch-Information Simulation</span><span class="zh">YouBike 調度資訊模擬</span></h3>
    <p class="en">A 76-station simulation isolating the time value of disclosing existing rebalancing locations. <a href="/projects/youbike-dispatch-information-simulation/">Read the research record →</a></p>
    <p class="zh">我提出研究題目，並用類似 PM 的角色協助團隊把問題形式化：在不增加調度車輛的前提下，用 76 站模擬估計公開調度位置能省下多少時間。<a href="/projects/youbike-dispatch-information-simulation/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">US Stock-Market NLP</span><span class="zh">美國股市 NLP 預測</span></h3>
    <p class="en">A five-person study comparing text models over MD&amp;A sections from 342 annual reports. <a href="/projects/us-stock-market-nlp/">Read the research record →</a></p>
    <p class="zh">五人團隊從 342 份美國上市公司年報擷取 MD&amp;A，測試文字模型能否預測隔年財務指標方向。文章以公開 repo 與報告為準。<a href="/projects/us-stock-market-nlp/">閱讀研究紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">N-Day CVE-2025-59366 Reproduction</span><span class="zh">N-Day CVE-2025-59366 復現</span></h3>
    <p class="en">A TeamT5 Camp reproduction of a disclosed AiCloud vulnerability using firmware diffing and dynamic tracing. <a href="/projects/asus-aicloud-cve-2025-59366/">Read the security record →</a></p>
    <p class="zh">TeamT5 Camp 三人小組從 ASUS AiCloud 的 firmware patch 反推資料流，再以動態追蹤重現已公開的 path traversal 與 command injection 漏洞鏈。<a href="/projects/asus-aicloud-cve-2025-59366/">閱讀資安紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>FurnitureStyle</h3>
    <p class="en">A Django prototype that turns an image or description into structured furniture attributes and shopping queries. <a href="/projects/furniturestyle/">Read the project record →</a></p>
    <p class="zh">把家具辨識與色彩轉換拆成兩次模型呼叫，再將結構化結果送進購物搜尋的 Django prototype。<a href="/projects/furniturestyle/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Lore</h3>
    <p class="en">A private LINE bot prototype for group memory, Q&amp;A, digests, and reminders. <a href="/projects/lore-group-memory/">Read the project record →</a></p>
    <p class="zh">把 LINE 群組裡散落的訊息整理成可查詢的記憶、摘要與提醒。目前仍是私人 prototype，還沒有真實群組的長期使用證據。<a href="/projects/lore-group-memory/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3>Adventure Cube</h3>
    <p class="en">A team children's-story project whose active surface is now a batch story and TTS pipeline. <a href="/projects/adventure-cube/">Read the project record →</a></p>
    <p class="zh">我設計整體架構，也參與實際測試與回饋迭代。專案未正式上線，目前持續運作的是批次故事與 TTS pipeline。<a href="/projects/adventure-cube/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Legacy WordPress Operations</span><span class="zh">老 WordPress 行政流程自動化</span></h3>
    <p class="en">A de-identified public-sector part-time workflow that turns REST, browser, and FTP operations into repeatable runbooks with round-trip verification. <a href="/projects/legacy-wordpress-ai-operations/">Read the project record →</a></p>
    <p class="zh">一份去識別化的公部門小打工：我用 AI 拆開 REST、瀏覽器與 FTP 的操作邊界，再把一次性的摸索整理成可重跑、可驗證的流程。<a href="/projects/legacy-wordpress-ai-operations/">閱讀專案紀錄 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Confidential Computing / TrustZone Virtualization</span><span class="zh">機密運算／TrustZone 虛擬化</span></h3>
    <p class="en">Research at SSLab: virtualizing Arm TrustZone with AVF (related to Google pKVM).</p>
    <p class="zh">SSLab 研究：用 AVF 把 Arm TrustZone 虛擬化（與 Google pKVM 相關）。</p>
  </div>
  <div class="pf-item">
    <h3><span class="en">Virtual Machine</span><span class="zh">虛擬機器</span></h3>
    <p class="en"><a href="/projects/114-2-vm-assignment-1-writeup/">Let a guest control host vCPU affinity</a> · <a href="/projects/114-2-vm-assignment-2-writeup/">Boot a Realm VM with early console output</a></p>
    <p class="zh"><a href="/projects/114-2-vm-assignment-1-writeup/">讓 guest 控制 host vCPU affinity</a> · <a href="/projects/114-2-vm-assignment-2-writeup/">啟動 Realm VM 並輸出 early console</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">pKVM / SMMUv3 Device Passthrough</span><span class="zh">pKVM / SMMUv3 裝置直通</span></h3>
    <p class="en">Extending host DMA isolation into protected guest assignment. <a href="/projects/nested-pkvm-smmu-device-passthrough/">Read the case study →</a></p>
    <p class="zh">把主機端的 DMA 隔離延伸到受保護 VM 的裝置指派。<a href="/projects/nested-pkvm-smmu-device-passthrough/">閱讀案例 →</a></p>
  </div>
  <div class="pf-item">
    <h3><span class="en">NTU Homeless Service Club</span><span class="zh">台大無家者服務社</span></h3>
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
  ▸ EN / 中文 由上方按鈕切換；預設中文。
  ▸ 專案名稱不需翻譯時可直接寫 <h3>；需要翻譯時在標題內分別使用 .en / .zh。
  ▸ 想寫完整長文時，把描述換成連結：<a href="/projects/xxx/">Read more →</a>
    （單一 project 頁模板：projects/example-project.md）
-->

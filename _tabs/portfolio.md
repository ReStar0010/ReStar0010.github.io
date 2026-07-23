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
    <p class="en">Projects I've built or am building. Names first — details to come.</p>
    <p class="zh">我做過、正在做的一些專案。先列項目，內容之後再補。</p>
  </div>
  <div class="pf-item">
    <h3>CouPro</h3>
    <p class="en">A localized, two-sided deals platform centered on campus life (NTU CEP, 18th cohort).</p>
    <p class="zh">以校園為中心的在地化優惠雙邊平台（台大創創學程 CEP 第 18 屆）。</p>
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

---
icon: fas fa-diagram-project
order: 1
---

A selection of things I'm building and taking apart. The writing lives in the
[posts]({{ '/' | relative_url }}); this page is the "what I've made" side.

<!--
  TO ADD A PROJECT: copy one <article class="proj-card"> … </article> block below
  and edit the title, description, tech tags, and links. Order = importance,
  top-left first. External links (github.com etc.) are safe; avoid linking to
  internal pages that don't exist yet, or the build's link check will fail.
-->

<style>
  .proj-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    gap: 1rem;
    margin: 1.5rem 0;
  }
  .proj-card {
    display: flex;
    flex-direction: column;
    background: var(--card-bg, #fff);
    border: 1px solid var(--card-border-color, rgba(0, 0, 0, 0.075));
    border-radius: 12px;
    padding: 1.1rem 1.2rem 1.15rem;
    box-shadow: var(--card-shadow, 0 1px 3px rgba(0, 0, 0, 0.06));
    transition: transform 0.15s ease, box-shadow 0.15s ease;
  }
  .proj-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 10px 26px -12px rgba(0, 0, 0, 0.28);
  }
  .proj-card h3 {
    margin: 0.1rem 0 0.5rem !important;
    font-size: 1.12rem;
  }
  .proj-card p {
    margin: 0 0 0.95rem;
    font-size: 0.9rem;
    line-height: 1.55;
    color: var(--text-muted-color, #6b6b76);
    flex: 1;
  }
  .proj-tech {
    display: flex;
    flex-wrap: wrap;
    gap: 0.35rem;
    margin-bottom: 0.95rem;
  }
  .proj-tech span {
    font-size: 0.72rem;
    font-family: ui-monospace, "SF Mono", Menlo, monospace;
    padding: 0.15rem 0.55rem;
    border-radius: 999px;
    background: var(--tag-bg, rgba(0, 0, 0, 0.05));
    color: var(--text-muted-color, #6b6b76);
  }
  .proj-links {
    display: flex;
    align-items: center;
    gap: 1rem;
    font-size: 0.83rem;
  }
  .proj-links a { text-decoration: none; font-weight: 600; }
  .proj-status {
    font-size: 0.75rem;
    font-family: ui-monospace, "SF Mono", Menlo, monospace;
    color: var(--text-muted-color, #6b6b76);
  }
  html[data-mode="dark"] .proj-card {
    background: var(--card-bg, #1e1f24);
    border-color: rgba(255, 255, 255, 0.08);
  }
  html[data-mode="dark"] .proj-tech span {
    background: rgba(255, 255, 255, 0.08);
  }
</style>

<div class="proj-grid">

  <article class="proj-card">
    <h3>CouPro</h3>
    <p>An incubating startup building a smoother coupon-sharing platform — turning
       scattered coupons into something people actually reach for.</p>
    <div class="proj-tech"><span>Startup</span><span>Product</span></div>
    <div class="proj-links"><span class="proj-status">● Building</span></div>
  </article>

  <article class="proj-card">
    <h3>Kernel / Isolation Research <em>(placeholder)</em></h3>
    <p>Replace me: a project on isolation, compartmentalization, or kernel security.
       Describe the problem, your approach, and the result in two sentences.</p>
    <div class="proj-tech"><span>C</span><span>Linux</span><span>Kernel</span></div>
    <div class="proj-links"><a href="https://github.com/ReStar0010">Code →</a></div>
  </article>

  <article class="proj-card">
    <h3>Reverse-Engineering Write-ups <em>(placeholder)</em></h3>
    <p>Replace me: link a RE challenge or write-up. This card is a template — copy
       it to add more.</p>
    <div class="proj-tech"><span>Ghidra</span><span>C</span><span>RE</span></div>
    <div class="proj-links"><a href="https://github.com/ReStar0010">Code →</a></div>
  </article>

</div>

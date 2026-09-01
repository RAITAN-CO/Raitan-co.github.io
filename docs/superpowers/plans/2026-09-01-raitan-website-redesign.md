# RAITAN Website Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Port raitan.co from Squarespace to a static Jekyll site in `RAITAN-CO/Raitan-co.github.io`, with a visual redesign (hybrid "Dark Technical × Clean Industrial" direction) and three content additions: a HELIONODE product page, an IPC-A-610/IPC-620 Certifications section on About, and a working News blog.

**Architecture:** Jekyll, built natively by GitHub Pages (no custom build step). One shared `_layouts/default.html` holds the `<head>`, nav, and footer; every page is a content file wrapped by that layout. `_posts/` is a real Jekyll collection so News is an actual blog, not a hand-coded page.

**Tech Stack:** Jekyll (Ruby gem, pinned `~> 4.3`), plain HTML/CSS (no JS framework, no build tooling beyond Jekyll itself), Formspree for the contact form (no backend).

**Spec:** `docs/superpowers/specs/2026-09-01-raitan-website-redesign-design.md`

## Global Constraints

- Colors (exact hex, from spec): page background `#0c1015`, raised panel `#0f141a` / `#12171e`, primary text `#eef1f5`, muted text `#8b93a1`, accent (orange) `#e0632a`, secondary/structural (steel blue-grey) `#3d5a6c`, hairline dividers `#1c232c`.
- Fonts (Google Fonts, from spec): headlines — Archivo, weight 700–800. Body — IBM Plex Sans. Nav/labels/eyebrows/data — IBM Plex Mono.
- Cert wording is exact and non-negotiable: **"ISO 9001:2015 CERTIFIED"** and **"IPC-A-610 & IPC/WHMA-A-620 CLASS 3 CERTIFIED"**. Never write "IPC-610"/"IPC-620" (wrong standard names) or "Certified Staff" (wrong — credential is per-person "Certified IPC Specialist").
- Do NOT create a `CNAME` file or otherwise reference the raitan.co custom domain anywhere in this build — DNS cutover is explicitly deferred to a later, separate step.
- Do NOT fabricate HELIONODE technical specs (voltage, power output, dimensions). Only the reference image and one-line description ("standalone solar power system for remote sensors") are confirmed.
- Nav structure is fixed: Home / About / Batteries ▾ (SEN, TAKI, MUGEN) / Solar ▾ (Solar Pavement, HELIONODE) / News / Contact.

---

## File Structure

```
repo/
├── _config.yml
├── Gemfile
├── .gitignore
├── _layouts/
│   └── default.html
├── _includes/
│   ├── nav.html
│   └── footer.html
├── assets/
│   └── css/
│       └── main.css
├── _posts/
│   └── 2026-09-01-ipc-a-610-620-class-3-certification.md
├── index.html
├── about.html
├── sen.html
├── taki.html
├── mugen.html
├── solar-pavement.html
├── helionode.html
├── contact.html
└── news.html
```

---

### Task 1: Jekyll scaffold + verify local build

**Files:**
- Create: `Gemfile`
- Create: `_config.yml`
- Create: `.gitignore`
- Create: `index.html` (temporary placeholder, replaced fully in Task 4)

**Interfaces:**
- Produces: a working `bundle exec jekyll build` / `bundle exec jekyll serve` toolchain that every later task builds on. Later tasks assume `bundle exec jekyll build` succeeds and writes to `_site/`.

- [ ] **Step 1: Write the Gemfile**

```ruby
# Gemfile
source "https://rubygems.org"

gem "jekyll", "~> 4.3"

group :jekyll_plugins do
  gem "jekyll-feed"
end

# Windows/JRuby compatibility (harmless on other platforms)
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
```

- [ ] **Step 2: Write `_config.yml`**

```yaml
title: RAITAN
description: "Custom battery systems, solar energy solutions, and subsea engineering — engineered and assembled to specification in Singapore."
url: "https://raitan-co.github.io"
baseurl: ""

markdown: kramdown
plugins:
  - jekyll-feed

collections:
  posts:
    output: true
    permalink: /news/:year-:month-:day-:title/

defaults:
  - scope:
      path: ""
      type: "posts"
    values:
      layout: "default"
```

- [ ] **Step 3: Write `.gitignore`**

```
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata
.bundle/
vendor/
```

- [ ] **Step 4: Write a temporary placeholder `index.html`**

```html
---
layout: none
title: RAITAN
---
<!doctype html>
<html>
<head><title>RAITAN — scaffold check</title></head>
<body><p>Scaffold OK</p></body>
</html>
```

- [ ] **Step 5: Install gems**

Run: `cd "/Volumes/G-DRIVE ArmorATD/Cluade Agent/raitan_website/repo" && bundle install`
Expected: completes without error, creates `Gemfile.lock`. If it fails because system Ruby is too old for the resolved jekyll version, retry with `bundle install --path vendor/bundle` and if still failing, pin `gem "jekyll", "~> 4.2.0"` in the Gemfile (broader Ruby compatibility) and re-run.

- [ ] **Step 6: Build and verify**

Run: `bundle exec jekyll build`
Expected: `_site/index.html` exists and contains "Scaffold OK":
```bash
grep -q "Scaffold OK" _site/index.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 7: Commit**

```bash
git add Gemfile Gemfile.lock _config.yml .gitignore index.html
git commit -m "Scaffold Jekyll site"
```

---

### Task 2: Design system CSS

**Files:**
- Create: `assets/css/main.css`

**Interfaces:**
- Produces: CSS custom properties (`--bg`, `--bg-raised`, `--ink`, `--ink-soft`, `--accent`, `--accent-soft`, `--line`) and component classes (`.nav`, `.nav-links`, `.hero`, `.cert-row`, `.cert`, `.product-grid`, `.prod`, `.btn`, `.cert-cards`, `.cert-card`, `.post-list`, `.post-item`, `.footer`) that every later page/include task uses by name.

- [ ] **Step 1: Write the stylesheet**

```css
/* assets/css/main.css */

:root {
  --bg: #0c1015;
  --bg-raised: #12171e;
  --bg-raised-alt: #0f141a;
  --ink: #eef1f5;
  --ink-soft: #8b93a1;
  --accent: #e0632a;
  --accent-soft: #3d5a6c;
  --line: #1c232c;
}

* { box-sizing: border-box; }

html, body {
  margin: 0;
  background: var(--bg);
  color: var(--ink);
  font-family: "IBM Plex Sans", -apple-system, "Segoe UI", sans-serif;
  line-height: 1.6;
}

a { color: inherit; text-decoration: none; }

.mono {
  font-family: "IBM Plex Mono", monospace;
}

.wrap {
  max-width: 1100px;
  margin: 0 auto;
  padding: 0 36px;
}

/* ---- nav ---- */

.nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 36px;
  border-bottom: 1px solid var(--line);
}

.nav-logo {
  font-family: "Archivo", sans-serif;
  font-weight: 800;
  font-size: 15px;
  letter-spacing: 0.1em;
  color: var(--ink);
}

.nav-links {
  display: flex;
  gap: 26px;
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  letter-spacing: 0.08em;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav-links > li {
  position: relative;
}

.nav-links a, .nav-links > li > span {
  color: var(--ink-soft);
  cursor: pointer;
}

.nav-links a.on, .nav-links a:hover, .nav-links > li:hover > span {
  color: var(--ink);
}

.nav-links a.on {
  border-bottom: 1.5px solid var(--accent);
  padding-bottom: 3px;
}

.nav-dropdown {
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  background: var(--bg-raised);
  border: 1px solid var(--line);
  padding: 10px 0;
  min-width: 160px;
  z-index: 10;
}

.nav-links > li:hover .nav-dropdown,
.nav-links > li:focus-within .nav-dropdown {
  display: block;
}

.nav-dropdown a {
  display: block;
  padding: 8px 16px;
  white-space: nowrap;
}

.nav-dropdown a:hover {
  color: var(--accent);
}

/* ---- hero ---- */

.hero {
  padding: 64px 36px 48px;
}

.hero-eyebrow {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  letter-spacing: 0.16em;
  color: var(--accent);
  text-transform: uppercase;
  margin: 0 0 18px;
}

.hero h1 {
  font-family: "Archivo", sans-serif;
  font-weight: 800;
  font-size: clamp(30px, 4vw, 44px);
  line-height: 1.08;
  color: var(--ink);
  margin: 0 0 18px;
  max-width: 14ch;
}

.hero-sub {
  font-size: 15px;
  color: var(--ink-soft);
  max-width: 50ch;
  line-height: 1.6;
  margin: 0 0 28px;
}

.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: "IBM Plex Mono", monospace;
  font-size: 12px;
  letter-spacing: 0.05em;
  background: var(--accent);
  color: var(--bg);
  font-weight: 500;
  padding: 12px 22px;
  border-radius: 2px;
  border: none;
  cursor: pointer;
}

.btn-outline {
  background: transparent;
  border: 1px solid var(--accent);
  color: var(--accent);
}

/* ---- cert row ---- */

.cert-row {
  display: flex;
  gap: 22px;
  margin-top: 40px;
  padding-top: 24px;
  border-top: 1px solid var(--line);
  flex-wrap: wrap;
}

.cert {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10.5px;
  color: var(--ink-soft);
  letter-spacing: 0.04em;
  display: flex;
  align-items: center;
  gap: 7px;
}

.cert .dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: var(--accent-soft);
}

/* ---- product grid (home page strip) ---- */

.product-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 1px;
  background: var(--line);
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
}

.prod {
  background: var(--bg-raised-alt);
  padding: 22px 18px;
  min-height: 108px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.prod .tag {
  font-family: "IBM Plex Mono", monospace;
  font-size: 9px;
  letter-spacing: 0.1em;
  color: var(--ink-soft);
  text-transform: uppercase;
}

.prod .name {
  font-family: "Archivo", sans-serif;
  font-weight: 700;
  font-size: 14px;
  color: var(--ink);
  margin-top: 14px;
}

.prod .desc {
  font-size: 10.5px;
  color: var(--ink-soft);
  margin-top: 4px;
  line-height: 1.4;
}

@media (max-width: 760px) {
  .nav-links { display: none; }
  .product-grid { grid-template-columns: repeat(2, 1fr); }
}

/* ---- content pages ---- */

.section {
  padding: 48px 36px;
  border-top: 1px solid var(--line);
}

.section h2 {
  font-family: "Archivo", sans-serif;
  font-weight: 700;
  font-size: 24px;
  color: var(--ink);
  margin: 0 0 20px;
}

.section p {
  color: var(--ink-soft);
  max-width: 68ch;
  margin: 0 0 16px;
}

.eyebrow {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  letter-spacing: 0.14em;
  color: var(--accent);
  text-transform: uppercase;
  margin: 0 0 14px;
}

.spec-table {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 20px;
  margin: 24px 0;
}

.spec-table .spec {
  background: var(--bg-raised);
  border: 1px solid var(--line);
  padding: 14px 16px;
}

.spec-table .spec .label {
  font-family: "IBM Plex Mono", monospace;
  font-size: 10px;
  letter-spacing: 0.08em;
  color: var(--ink-soft);
  text-transform: uppercase;
}

.spec-table .spec .value {
  font-family: "Archivo", sans-serif;
  font-weight: 700;
  font-size: 18px;
  color: var(--ink);
  margin-top: 6px;
}

/* ---- certification cards (about page) ---- */

.cert-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 20px;
  margin-top: 24px;
}

.cert-card {
  background: var(--bg-raised);
  border: 1px solid var(--line);
  padding: 22px;
}

.cert-card h3 {
  font-family: "Archivo", sans-serif;
  font-weight: 700;
  font-size: 16px;
  color: var(--ink);
  margin: 0 0 10px;
}

.cert-card p {
  font-size: 13px;
  color: var(--ink-soft);
  margin: 0 0 10px;
}

.cert-card .holder {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--accent-soft);
}

/* ---- news / blog list ---- */

.post-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.post-item {
  padding: 24px 0;
  border-bottom: 1px solid var(--line);
}

.post-item .post-date {
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ink-soft);
  letter-spacing: 0.06em;
}

.post-item h3 {
  font-family: "Archivo", sans-serif;
  font-weight: 700;
  font-size: 19px;
  color: var(--ink);
  margin: 8px 0;
}

.post-body p {
  color: var(--ink-soft);
  max-width: 68ch;
}

.post-body ul {
  color: var(--ink-soft);
  max-width: 68ch;
}

/* ---- contact form ---- */

.form-field {
  margin-bottom: 18px;
}

.form-field label {
  display: block;
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  letter-spacing: 0.06em;
  color: var(--ink-soft);
  text-transform: uppercase;
  margin-bottom: 6px;
}

.form-field input, .form-field textarea {
  width: 100%;
  background: var(--bg-raised);
  border: 1px solid var(--line);
  color: var(--ink);
  padding: 10px 12px;
  font-family: "IBM Plex Sans", sans-serif;
  font-size: 14px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
}

@media (max-width: 600px) {
  .form-row { grid-template-columns: 1fr; }
}

/* ---- footer ---- */

.footer {
  padding: 32px 36px;
  border-top: 1px solid var(--line);
  font-family: "IBM Plex Mono", monospace;
  font-size: 11px;
  color: var(--ink-soft);
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
}
```

- [ ] **Step 2: Verify the file is syntactically valid CSS**

Run: `cd "/Volumes/G-DRIVE ArmorATD/Cluade Agent/raitan_website/repo" && python3 -c "
content = open('assets/css/main.css').read()
assert content.count('{') == content.count('}'), 'unbalanced braces'
print('PASS')
"`
Expected: `PASS`

- [ ] **Step 3: Commit**

```bash
git add assets/css/main.css
git commit -m "Add design system stylesheet"
```

---

### Task 3: Shared layout + nav + footer includes

**Files:**
- Create: `_layouts/default.html`
- Create: `_includes/nav.html`
- Create: `_includes/footer.html`
- Modify: `index.html:1-10` (switch from `layout: none` scaffold to `layout: default`)

**Interfaces:**
- Consumes: `assets/css/main.css` classes from Task 2 (`.nav`, `.nav-links`, `.nav-dropdown`, `.footer`).
- Produces: `{{ content }}` wrapping used by every page task from here on. Every page file needs `layout: default` and `title:` in its front matter — later tasks assume this.

- [ ] **Step 1: Write `_includes/nav.html`**

```html
<nav class="nav">
  <a href="{{ '/' | relative_url }}" class="nav-logo">RAITAN</a>
  <ul class="nav-links">
    <li><a href="{{ '/' | relative_url }}" class="{% if page.url == '/' %}on{% endif %}">HOME</a></li>
    <li><a href="{{ '/about.html' | relative_url }}" class="{% if page.url == '/about.html' %}on{% endif %}">ABOUT</a></li>
    <li tabindex="0">
      <span class="{% if page.url == '/sen.html' or page.url == '/taki.html' or page.url == '/mugen.html' %}on{% endif %}">BATTERIES ▾</span>
      <div class="nav-dropdown">
        <a href="{{ '/sen.html' | relative_url }}">SEN</a>
        <a href="{{ '/taki.html' | relative_url }}">TAKI</a>
        <a href="{{ '/mugen.html' | relative_url }}">MUGEN</a>
      </div>
    </li>
    <li tabindex="0">
      <span class="{% if page.url == '/solar-pavement.html' or page.url == '/helionode.html' %}on{% endif %}">SOLAR ▾</span>
      <div class="nav-dropdown">
        <a href="{{ '/solar-pavement.html' | relative_url }}">SOLAR PAVEMENT</a>
        <a href="{{ '/helionode.html' | relative_url }}">HELIONODE</a>
      </div>
    </li>
    <li><a href="{{ '/news.html' | relative_url }}" class="{% if page.url == '/news.html' %}on{% endif %}">NEWS</a></li>
    <li><a href="{{ '/contact.html' | relative_url }}" class="{% if page.url == '/contact.html' %}on{% endif %}">CONTACT</a></li>
  </ul>
</nav>
```

- [ ] **Step 2: Write `_includes/footer.html`**

```html
<footer class="footer">
  <div>RAITAN PTE LTD &middot; UEN 202138267Z &middot; 20 Woodlands Link #05-18, Singapore 738733</div>
  <div>&copy; {{ 'now' | date: "%Y" }} Raitan Pte Ltd. All rights reserved.</div>
</footer>
```

- [ ] **Step 3: Write `_layouts/default.html`**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ page.title }} — RAITAN</title>
  <meta name="description" content="{{ page.description | default: site.description }}">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Archivo:wght@600;700;800&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap">
  <link rel="stylesheet" href="{{ '/assets/css/main.css' | relative_url }}">
</head>
<body>
  {% include nav.html %}
  {{ content }}
  {% include footer.html %}
</body>
</html>
```

- [ ] **Step 4: Update `index.html` to use the real layout**

```html
---
layout: default
title: Home
---
<div class="hero">
  <p class="hero-eyebrow">Scaffold check</p>
  <h1>Layout wiring OK</h1>
</div>
```

- [ ] **Step 5: Build and verify nav/footer render on every page**

Run: `bundle exec jekyll build`
Expected:
```bash
grep -q "RAITAN" _site/index.html && \
grep -q "nav-dropdown" _site/index.html && \
grep -q "UEN 202138267Z" _site/index.html && \
echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 6: Commit**

```bash
git add _layouts _includes index.html
git commit -m "Add shared layout, nav, and footer"
```

---

### Task 4: Home page

**Files:**
- Modify: `index.html` (full rewrite, replacing the Task 3 placeholder)

**Interfaces:**
- Consumes: `_layouts/default.html`, `.hero`, `.cert-row`, `.product-grid` classes.

**Source copy** (from the live Squarespace site, lightly tightened): headline "Custom Battery Systems for Demanding Applications", subhead "RAITAN engineers and assembles battery systems tailored to your voltage, chemistry, depth, and current requirements", ISO 9001 badge (Cert No. Q-047/25, issued 24 Apr 2025), and the five-product strip (MUGEN, TAKI, SEN, Solar Pavement, HELIONODE).

- [ ] **Step 1: Write the full home page**

```html
---
layout: default
title: Home
description: "Custom battery systems, solar energy solutions, and subsea engineering — engineered and assembled to specification in Singapore."
---
<div class="hero">
  <p class="hero-eyebrow">Custom battery &amp; energy systems</p>
  <h1>Custom Battery Systems for Demanding Applications</h1>
  <p class="hero-sub">RAITAN engineers and assembles battery systems tailored to your voltage, chemistry, depth, and current requirements.</p>
  <a href="{{ '/contact.html' | relative_url }}" class="btn">GET IN TOUCH &rarr;</a>

  <div class="cert-row">
    <div class="cert"><span class="dot"></span>ISO 9001:2015 CERTIFIED — Cert No. Q-047/25, issued 24 Apr 2025</div>
    <div class="cert"><span class="dot"></span>IPC-A-610 &amp; IPC/WHMA-A-620 CLASS 3 CERTIFIED</div>
  </div>
</div>

<div class="product-grid">
  <div class="prod">
    <div class="tag">Battery</div>
    <div>
      <div class="name">MUGEN</div>
      <div class="desc">Distributed ESS, up to 500VDC / 20kWh, unlimited parallel</div>
    </div>
  </div>
  <div class="prod">
    <div class="tag">Battery</div>
    <div>
      <div class="name">TAKI</div>
      <div class="desc">High-current series for electric motor vehicles</div>
    </div>
  </div>
  <div class="prod">
    <div class="tag">Battery</div>
    <div>
      <div class="name">SEN</div>
      <div class="desc">Subsea battery for AUV/ROV/sensors, up to 300m depth</div>
    </div>
  </div>
  <div class="prod">
    <div class="tag">Solar</div>
    <div>
      <div class="name">Solar Pavement</div>
      <div class="desc">Walkable PV tiles, proven at Sentosa Skywalk</div>
    </div>
  </div>
  <div class="prod">
    <div class="tag">Solar</div>
    <div>
      <div class="name">HELIONODE</div>
      <div class="desc">Standalone solar power for remote sensors</div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "Custom Battery Systems for Demanding Applications" _site/index.html && \
grep -q "Q-047/25" _site/index.html && \
grep -q "HELIONODE" _site/index.html && \
echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Build home page"
```

---

### Task 5: About page + Certifications section

**Files:**
- Create: `about.html`

**Interfaces:**
- Consumes: `.section`, `.cert-cards`, `.cert-card` classes.

**Source copy**: from the live Squarespace `/about` page (company description, philosophy) plus the three verified certifications (ISO 9001:2015, IPC-A-610J, IPC/WHMA-A-620E) from the spec.

- [ ] **Step 0: Certificate PDFs are already in place**

Raymond copied the four real certificate PDFs into `assets/certifications/` ahead of this task:
- `assets/certifications/ISO-9001-2015-Certificate.pdf`
- `assets/certifications/IPC-A-610J-Raymond-Tan.pdf`
- `assets/certifications/IPC-WHMA-A-620E-Raymond-Tan.pdf`
- `assets/certifications/IPC-WHMA-A-620E-Toh-Yue-Khing.pdf`

Verify they're present before starting:
```bash
ls assets/certifications/*.pdf | wc -l
```
Expected output: `4`

- [ ] **Step 1: Write the About page**

```html
---
layout: default
title: About
description: "Raitan Pte Ltd — Singapore-based engineering firm specialising in custom battery systems, solar energy solutions, and subsea engineering."
---
<div class="hero">
  <p class="hero-eyebrow">About Raitan</p>
  <h1>Singapore's Specialist in Custom Battery Engineering</h1>
  <p class="hero-sub">Raitan is a Singapore-based engineering firm specialising in custom battery systems, solar energy solutions, and subsea engineering. We design and assemble every system to customer specification — no off-the-shelf products, no compromises.</p>
</div>

<div class="section">
  <h2>Who we are</h2>
  <p>Raitan Pte Ltd is a Singapore-incorporated engineering firm founded in 2022, specialising in the design and assembly of custom battery systems across multiple chemistries — Lithium Iron Phosphate (LFP), Nickel Manganese Cobalt (NMC), and others — alongside solar energy solutions including BIPV, and subsea engineering support.</p>
  <p>We do not sell catalogue products. Every Raitan battery system — whether a 300m-rated subsea pack for an AUV, a distributed energy storage system for a carpark rooftop, or a high-current LFP pack for an electric vehicle — is engineered and assembled to the customer's exact specification at our facility at 20 Woodlands Link, Singapore.</p>
  <p>Raitan is an active member of Singapore's battery standards workgroups — SS725, TR77, and TR136.</p>
</div>

<div class="section">
  <h2>Company philosophy</h2>
  <p><strong>Longevity is sustainability.</strong> A battery that lasts 3,000 cycles is a battery that does not reach landfill. We engineer systems to last — reducing replacement frequency, material waste, and total cost of ownership.</p>
  <p><strong>Value for money, precision-engineered.</strong> Every Raitan system is specified exactly for the customer's application — no over-engineering, no under-engineering. Customers pay for the performance their application demands, nothing more.</p>
  <p><strong>Closing the green energy loop.</strong> In land-scarce Singapore, conventional solar demands space that simply does not exist. Raitan's BIPV solutions generate power from surfaces that already exist — floors, walkways, terraces — paired with the MUGEN distributed ESS fitted into dead spaces.</p>
</div>

<div class="section">
  <h2>Certifications</h2>
  <p>Raitan's quality management system and production personnel hold the following certifications.</p>
  <div class="cert-cards">
    <div class="cert-card">
      <h3>ISO 9001:2015</h3>
      <p>Quality Management System — Design and Assembly of Battery Systems.</p>
      <div class="holder">Cert No. Q-047/25 &middot; Issued 24 Apr 2025 &middot; QCERT Singapore</div>
      <p><a href="{{ '/assets/certifications/ISO-9001-2015-Certificate.pdf' | relative_url }}" class="btn btn-outline" style="margin-top:12px;">VIEW CERTIFICATE &rarr;</a></p>
    </div>
    <div class="cert-card">
      <h3>IPC-A-610J</h3>
      <p>Acceptability of Electronic Assemblies — Class 3 (High Performance/High Reliability Electronic Products).</p>
      <div class="holder">Certified IPC Specialist: <a href="{{ '/assets/certifications/IPC-A-610J-Raymond-Tan.pdf' | relative_url }}">Raymond Tan</a></div>
    </div>
    <div class="cert-card">
      <h3>IPC/WHMA-A-620E</h3>
      <p>Requirements and Acceptance for Cable and Wire Harness Assemblies — Class 3.</p>
      <div class="holder">Certified IPC Specialists: <a href="{{ '/assets/certifications/IPC-WHMA-A-620E-Raymond-Tan.pdf' | relative_url }}">Raymond Tan</a>, <a href="{{ '/assets/certifications/IPC-WHMA-A-620E-Toh-Yue-Khing.pdf' | relative_url }}">Toh Yue Khing</a></div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "IPC-A-610J" _site/about.html && \
grep -q "IPC/WHMA-A-620E" _site/about.html && \
grep -q "Toh Yue Khing" _site/about.html && \
grep -q "Q-047/25" _site/about.html && \
grep -q "certifications/ISO-9001-2015-Certificate.pdf" _site/about.html && \
grep -q "certifications/IPC-A-610J-Raymond-Tan.pdf" _site/about.html && \
grep -q "certifications/IPC-WHMA-A-620E-Raymond-Tan.pdf" _site/about.html && \
grep -q "certifications/IPC-WHMA-A-620E-Toh-Yue-Khing.pdf" _site/about.html && \
echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add about.html assets/certifications
git commit -m "Build About page with Certifications section"
```

---

### Task 6: SEN product page

**Files:**
- Create: `sen.html`

**Interfaces:**
- Consumes: `.hero`, `.section`, `.spec-table` classes.

**Source copy**: from the live Squarespace `/sen` page.

- [ ] **Step 1: Write the SEN page**

```html
---
layout: default
title: SEN Series
description: "Depth-rated lithium battery systems for AUV, ROV, and subsea robotic platforms. Rated to 300m."
---
<div class="hero">
  <p class="hero-eyebrow">Underwater battery pack</p>
  <h1>SEN Series</h1>
  <p class="hero-sub">Depth-rated lithium battery systems for AUV, ROV, and subsea robotic platforms. Available in three energy configurations. Rated to 300m.</p>
</div>

<div class="section">
  <h2>Product overview</h2>
  <p>The SEN Series is Raitan's underwater battery platform — a family of pressure-rated, cylindrical lithium battery packs designed for integration into AUVs, ROVs, and subsea robotic systems operating at depths to 300 metres.</p>
  <p>All SEN batteries use UN 38.3 compliant cells and are assembled under Raitan's ISO 9001:2015 certified QMS. Each unit includes an integrated BMS providing cell-level protection, RS485 telemetry, and SOC reporting to topside systems.</p>

  <div class="spec-table">
    <div class="spec"><div class="label">Small</div><div class="value">0.6 kWh</div></div>
    <div class="spec"><div class="label">Medium</div><div class="value">1 kWh</div></div>
    <div class="spec"><div class="label">Large</div><div class="value">2 kWh</div></div>
    <div class="spec"><div class="label">Depth rating</div><div class="value">300 m</div></div>
  </div>
</div>

<div class="section">
  <h2>Applications</h2>
  <p><strong>Underwater vehicles.</strong> AUV propulsion and hotel load power for survey, inspection, and defence missions. Multiple SEN batteries may be wired in parallel for extended mission endurance beyond single-battery range. Battery voltage, housing configuration, depth rating, and connector interface are customisable.</p>
  <p><strong>Subsea instruments &amp; sensors.</strong> Long-duration power for seabed monitoring nodes, acoustic modems, and environmental sensors.</p>
  <a href="{{ '/contact.html' | relative_url }}" class="btn btn-outline">DISCUSS YOUR APPLICATION &rarr;</a>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "SEN Series" _site/sen.html && grep -q "300 m" _site/sen.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add sen.html
git commit -m "Build SEN product page"
```

---

### Task 7: TAKI product page

**Files:**
- Create: `taki.html`

**Interfaces:**
- Consumes: `.hero`, `.section` classes.

**Source copy**: from the live Squarespace `/taki` page.

- [ ] **Step 1: Write the TAKI page**

```html
---
layout: default
title: TAKI Series
description: "High-current battery packs for electric motor vehicles, AGVs, and industrial drives. 24V and 48V variants."
---
<div class="hero">
  <p class="hero-eyebrow">High current battery pack</p>
  <h1>TAKI Series</h1>
  <p class="hero-sub">Purpose-built for electric motor vehicles, AGVs, and high-demand industrial drives. Available in 24V and 48V variants with NMC / LFP / NA chemistry and integrated high-current BMS.</p>
</div>

<div class="section">
  <h2>Design for high current</h2>
  <p>The TAKI Series is Raitan's high-current battery pack, designed specifically for applications that demand sustained high discharge rates.</p>
  <p>Where conventional battery packs throttle output under load to protect cells, the TAKI's high-current BMS and cell configuration are engineered from first principles around peak discharge performance. As with all Raitan battery systems, the TAKI is built-to-order — voltage, capacity, form factor, cell count, BMS thresholds, and connector type are all specifiable at enquiry.</p>
</div>

<div class="section">
  <h2>Applications</h2>
  <p><strong>Electric motor vehicles.</strong> Golf carts, electric utility vehicles, last-mile delivery EVs, and light electric transport platforms.</p>
  <p><strong>Industrial drives &amp; equipment.</strong> Electric forklifts, scissor lifts, and powered industrial tools. The 48V variant provides extended run-time for full-shift industrial operations without interim charging.</p>
  <p><strong>Defence &amp; specialist platforms.</strong> Custom voltage, form factor, connector, and BMS configurations for specialist ground vehicle, UAV ground support, and portable power applications. Defence enquiries handled in confidence — contact Raitan's sales team directly.</p>
  <a href="{{ '/contact.html' | relative_url }}" class="btn btn-outline">CUSTOMISE YOUR BATTERY &rarr;</a>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "TAKI Series" _site/taki.html && grep -q "48V" _site/taki.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add taki.html
git commit -m "Build TAKI product page"
```

---

### Task 8: MUGEN product page

**Files:**
- Create: `mugen.html`

**Interfaces:**
- Consumes: `.hero`, `.section`, `.spec-table` classes.

**Source copy**: from the live Squarespace `/mugen` page.

- [ ] **Step 1: Write the MUGEN page**

```html
---
layout: default
title: MUGEN Series
description: "Modular, stackable distributed energy storage for space-constrained urban environments. Scalable to 500VDC."
---
<div class="hero">
  <p class="hero-eyebrow">Distributed energy storage system</p>
  <h1>MUGEN Series</h1>
  <p class="hero-sub">Modular, stackable battery energy storage designed for space-constrained urban environments. Scalable to 500VDC, with unlimited parallel capacity and PV input up to 1000VDC.</p>

  <div class="spec-table">
    <div class="spec"><div class="label">Max stack voltage</div><div class="value">500 V</div></div>
    <div class="spec"><div class="label">Per DESS</div><div class="value">20 kWh</div></div>
    <div class="spec"><div class="label">Parallel connections</div><div class="value">Unlimited</div></div>
    <div class="spec"><div class="label">Build</div><div class="value">Built-to-order</div></div>
  </div>
</div>

<div class="section">
  <h2>Design for dead spaces</h2>
  <p>The MUGEN Series is a versatile, modular battery pack engineered to be stacked and distributed across a building's underutilised areas — awkward carpark corners, rooftop plant spaces, and other dead zones that conventional large-format BESS cannot use.</p>
  <p>Individual battery packs connect in series to reach up to 500VDC nominal, and in parallel without limit — allowing the system to scale with demand over time, adding capacity without replacing existing hardware. A dedicated PV input accepts up to 1000VDC with a maximum power of 15kW, making MUGEN the natural storage partner for Raitan's solar pavement and solar deck installations.</p>
</div>

<div class="section">
  <h2>Key benefits</h2>
  <p><strong>Cost saving.</strong> Curbs contracted capacity costs, reduces energy curtailment, reduces land opportunity cost by deploying into otherwise unusable spaces.</p>
  <p><strong>Safety by design.</strong> Distributed topology reduces risk concentration, prevents fire propagation between cells, and supports individual pack isolation.</p>
  <p><strong>Space maximisation.</strong> Eliminates the need for a dedicated plant room — packs install into carpark pillars, ceiling voids, and rooftop corners.</p>
  <p><strong>Peak shaving.</strong> Discharge during peak demand periods to reduce maximum contracted grid capacity, directly reducing monthly utility bills.</p>
  <a href="{{ '/contact.html' | relative_url }}" class="btn btn-outline">SPEC A SYSTEM &rarr;</a>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "MUGEN Series" _site/mugen.html && grep -q "500 V" _site/mugen.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add mugen.html
git commit -m "Build MUGEN product page"
```

---

### Task 9: Solar Pavement product page

**Files:**
- Create: `solar-pavement.html`

**Interfaces:**
- Consumes: `.hero`, `.section`, `.spec-table` classes.

**Source copy**: from the live Squarespace `/solar-pavement` page.

- [ ] **Step 1: Write the Solar Pavement page**

```html
---
layout: default
title: Solar Pavement
description: "Building-integrated PV — solar paving and decking tiles supplied in partnership with Platio, proven at Sentosa Skywalk."
---
<div class="hero">
  <p class="hero-eyebrow">Solar solutions</p>
  <h1>Building-Integrated PV</h1>
  <p class="hero-sub">Singapore faces a fundamental constraint that limits conventional solar deployment: there is simply not enough open land or dedicated roof space. Raitan's solar pavement and solar deck systems, supplied in partnership with Platio, integrate photovoltaic generation into surfaces that already form part of your built environment.</p>
</div>

<div class="section">
  <h2>Pilot deployment: Sentosa Skywalk</h2>
  <p>Raitan successfully completed a pilot installation of 612Wp of solar pavement at Sentosa Skywalk — a real-world public-space deployment demonstrating the integration of photovoltaic paving with micro-inverter technology. Unlike string inverter systems where a single shaded panel degrades the output of the entire array, each micro-inverter operates independently, critical where walkway railings cast moving shadows across the pavement through the day.</p>
  <p>This project was recognised at the 2022 Sustainability Open Innovation Challenge, where Raitan won the Sentosa Challenge category.</p>
</div>

<div class="section">
  <h2>Solar Paver</h2>
  <p>Structural PV paving tiles rated for pedestrian load. Anti-slip, weather-resistant. For public plazas, pathways, and urban open spaces. Proven in Singapore tropical conditions at Sentosa.</p>
  <div class="spec-table">
    <div class="spec"><div class="label">Size</div><div class="value">353×353×41mm</div></div>
    <div class="spec"><div class="label">Weight</div><div class="value">6.5kg/pcs</div></div>
    <div class="spec"><div class="label">Max loading</div><div class="value">2000kg/pcs</div></div>
    <div class="spec"><div class="label">Voltage (OCV)</div><div class="value">2.72V</div></div>
    <div class="spec"><div class="label">Current (SCC)</div><div class="value">8.89A</div></div>
    <div class="spec"><div class="label">Nominal power</div><div class="value">17Wp/pcs</div></div>
  </div>
</div>

<div class="section">
  <h2>Solar Deck</h2>
  <p>Solar decking panels for elevated walkways, rooftop terraces, and landscape structures. Integrates generation into built form without dedicated PV roof arrays.</p>
  <div class="spec-table">
    <div class="spec"><div class="label">Size</div><div class="value">447×1000×40mm</div></div>
    <div class="spec"><div class="label">Weight</div><div class="value">11.5kg/pcs</div></div>
    <div class="spec"><div class="label">Max loading</div><div class="value">300kg/pcs</div></div>
    <div class="spec"><div class="label">Voltage (OCV)</div><div class="value">6.93V</div></div>
    <div class="spec"><div class="label">Current (SCC)</div><div class="value">11.7A</div></div>
    <div class="spec"><div class="label">Nominal power</div><div class="value">59Wp/pcs</div></div>
  </div>
  <a href="{{ '/contact.html' | relative_url }}" class="btn btn-outline">DISCUSS A DEPLOYMENT &rarr;</a>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "Sentosa Skywalk" _site/solar-pavement.html && grep -q "353×353×41mm" _site/solar-pavement.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add solar-pavement.html
git commit -m "Build Solar Pavement product page"
```

---

### Task 10: HELIONODE product page

**Files:**
- Create: `helionode.html`

**Interfaces:**
- Consumes: `.hero`, `.section` classes.

**Content status**: per Global Constraints, only the reference image and a one-line description are confirmed. This task writes real, honest copy using only confirmed facts — it does not invent voltage, power, or dimension numbers.

- [ ] **Step 1: Write the HELIONODE page**

```html
---
layout: default
title: HELIONODE
description: "Standalone solar power system for remote sensors — off-grid generation and storage in a single deployable unit."
---
<div class="hero">
  <p class="hero-eyebrow">Standalone solar power system</p>
  <h1>HELIONODE</h1>
  <p class="hero-sub">A self-contained solar power system for remote sensors — twin solar panels, integrated electronics enclosure, and battery storage in a single pole-mounted unit, built for locations without grid access.</p>
</div>

<div class="section">
  <h2>Off-grid power, wherever the sensor is</h2>
  <p>HELIONODE pairs solar generation with onboard battery storage to keep remote sensors, monitoring equipment, and communications gear running without a grid connection or scheduled site visits for battery swaps.</p>
  <p>Full technical specifications (power output, battery capacity, mounting options, and enclosure ratings) are available on request — get in touch to discuss your deployment.</p>
  <a href="{{ '/contact.html' | relative_url }}" class="btn btn-outline">ASK ABOUT HELIONODE &rarr;</a>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "HELIONODE" _site/helionode.html && grep -q "available on request" _site/helionode.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add helionode.html
git commit -m "Build HELIONODE product page"
```

**Note for the executor:** flag to Raymond that this page is intentionally thin until he supplies real specs — don't let a later task quietly backfill invented numbers.

---

### Task 11: Contact page + Formspree

**Files:**
- Create: `contact.html`

**Interfaces:**
- Consumes: `.hero`, `.section`, `.form-field`, `.form-row` classes.

- [ ] **Step 1: Write the Contact page**

```html
---
layout: default
title: Contact
description: "Get in touch with Raitan to discuss your battery, solar, or subsea engineering project."
---
<div class="hero">
  <p class="hero-eyebrow">Let's work together</p>
  <h1>Get in touch</h1>
  <p class="hero-sub">Further case studies available upon request. Please provide some information on your project or goals and we'll move the conversation on from there.</p>
</div>

<div class="section">
  <form action="https://formspree.io/f/YOUR_FORM_ID" method="POST">
    <div class="form-row">
      <div class="form-field">
        <label for="first-name">First name</label>
        <input type="text" id="first-name" name="firstName" required>
      </div>
      <div class="form-field">
        <label for="last-name">Last name</label>
        <input type="text" id="last-name" name="lastName" required>
      </div>
    </div>
    <div class="form-field">
      <label for="email">Email</label>
      <input type="email" id="email" name="email" required>
    </div>
    <div class="form-field">
      <label for="subject">Subject</label>
      <input type="text" id="subject" name="subject" required>
    </div>
    <div class="form-field">
      <label for="message">Message</label>
      <textarea id="message" name="message" rows="6" required></textarea>
    </div>
    <button type="submit" class="btn">SUBMIT</button>
  </form>
</div>
```

- [ ] **Step 2: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "formspree.io" _site/contact.html && grep -q "firstName" _site/contact.html && echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 3: Commit**

```bash
git add contact.html
git commit -m "Build Contact page with Formspree form"
```

**Note for the executor:** `action="https://formspree.io/f/YOUR_FORM_ID"` is a literal placeholder in the form's `action` attribute — Raymond needs to create a Formspree account, create a form, and give you the real form ID before this goes live. Flag this explicitly; don't ship the site with the placeholder ID live.

---

### Task 12: News collection + first post

**Files:**
- Create: `news.html`
- Create: `_posts/2026-09-01-ipc-a-610-620-class-3-certification.md`

**Interfaces:**
- Consumes: Jekyll's `site.posts` collection (configured in Task 1's `_config.yml`), `.post-list`, `.post-item`, `.post-body` classes.

- [ ] **Step 1: Write the News index page**

```html
---
layout: default
title: News
description: "Updates from Raitan Pte Ltd."
---
<div class="hero">
  <p class="hero-eyebrow">Updates</p>
  <h1>News</h1>
</div>

<div class="section">
  <ul class="post-list">
    {% for post in site.posts %}
    <li class="post-item">
      <div class="post-date">{{ post.date | date: "%d %b %Y" }}</div>
      <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p>{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
    </li>
    {% endfor %}
  </ul>
</div>
```

- [ ] **Step 2: Write the first post, using Raymond's supplied copy verbatim**

```markdown
---
layout: default
title: "Raitan Achieves IPC-A-610 & IPC-620 Class 3 Workmanship Certification"
description: "Raitan's production and quality personnel are now certified to IPC-A-610 and IPC/WHMA-A-620 at Class 3."
---
<div class="hero">
  <p class="hero-eyebrow">{{ page.date | date: "%d %b %Y" }}</p>
  <h1>{{ page.title }}</h1>
</div>

<div class="section post-body">

Raitan Pte Ltd's production and quality personnel are now certified to IPC-A-610, Acceptability of Electronic Assemblies, and IPC/WHMA-A-620, Requirements and Acceptance for Cable and Wire Harness Assemblies, at Class 3 (High Performance/High Reliability Electronic Products).

This certification is embedded within our ISO 9001:2015 Quality Management System, supporting Clause 7.2 (Competence) and Clause 8.5 (Production and Service Provision), and gives our customers:

- Consistent Class 3 workmanship across soldered electronic assemblies and wire/cable harness builds
- Objective, auditable evidence of operator competency, retained as controlled quality records
- Lower first-pass inspection failure rates and reduced non-conformance risk on delivered assemblies
- A workforce trained to standards recognised across defence, aerospace, medical, and other high-reliability sectors

This capability supports Raitan's continual improvement objectives under ISO 9001:2015 Clause 10.3, and reinforces our position as a supplier able to meet the most stringent electronics workmanship requirements for Singapore's defence and precision engineering industry.

</div>
```

- [ ] **Step 3: Build and verify**

Run: `bundle exec jekyll build`
```bash
grep -q "News" _site/news.html && \
grep -rq "IPC-A-610" _site/news/2026-09-01-ipc-a-610-620-class-3-certification/index.html && \
grep -q "IPC-A-610 & IPC-620 Class 3 Workmanship Certification" _site/news.html && \
echo PASS || echo FAIL
```
Expected output: `PASS`

- [ ] **Step 4: Commit**

```bash
git add news.html _posts
git commit -m "Add News blog with IPC certification announcement"
```

---

### Task 13: Full-site build verification and local preview

**Files:**
- None created — this is a verification-only task across everything built so far.

- [ ] **Step 1: Clean build**

```bash
cd "/Volumes/G-DRIVE ArmorATD/Cluade Agent/raitan_website/repo"
rm -rf _site .jekyll-cache
bundle exec jekyll build
```
Expected: no errors, `_site/` regenerated.

- [ ] **Step 2: Verify every page exists and every internal nav link resolves to a real file**

```bash
python3 -c "
import os
pages = ['index.html','about.html','sen.html','taki.html','mugen.html',
         'solar-pavement.html','helionode.html','contact.html','news.html']
missing = [p for p in pages if not os.path.exists(os.path.join('_site', p))]
assert not missing, f'missing: {missing}'
print('PASS')
"
```
Expected output: `PASS`

- [ ] **Step 3: Serve locally and spot-check in the browser**

Run: `bundle exec jekyll serve --port 4001` (background this or run in a separate terminal)
Then load `http://127.0.0.1:4001/` and click through Home → About → each Batteries dropdown item → each Solar dropdown item → News → the IPC post → Contact. Confirm: nav dropdown opens on hover, active-page underline shows on the current nav item, cert badges read exactly "ISO 9001:2015 CERTIFIED" and "IPC-A-610 & IPC/WHMA-A-620 CLASS 3 CERTIFIED", and the Contact form renders (submission itself won't work until the real Formspree ID is in place from Task 11).

- [ ] **Step 4: Stop the local server**

```bash
kill %1 2>/dev/null || true
```

No commit for this task — it's verification only.

---

### Task 14: Push to GitHub and verify Pages deploy

**Files:**
- None created.

- [ ] **Step 1: Push to the remote**

```bash
cd "/Volumes/G-DRIVE ArmorATD/Cluade Agent/raitan_website/repo"
git push origin main
```
Expected: push succeeds (this repo was cloned fresh in Task 1's prerequisite step, so `main` should be a simple fast-forward — if it isn't, stop and report rather than force-pushing).

- [ ] **Step 2: Confirm GitHub Pages build succeeded**

Open `https://github.com/RAITAN-CO/Raitan-co.github.io/actions` in the browser and confirm the Pages build workflow completed successfully (green check), not failed (red X).

- [ ] **Step 3: Confirm the live default URL**

Open `https://raitan-co.github.io` in the browser and confirm the home page loads with the new design (dark ground, orange accent, Archivo headline) — this is Raymond's review checkpoint before any DNS change happens.

No commit for this task.

---

## Explicitly out of scope for this plan

- Pointing raitan.co's DNS at this site (CrazyDomains change) — Raymond does this himself after reviewing the `raitan-co.github.io` deploy.
- Sourcing/replacing product photography — this plan ships with the new text/layout system but no new product photos; if Raymond wants imagery added, that's a follow-up task.
- Formspree account creation — Raymond needs to do this himself and hand over the real form ID (Task 11 ships with a placeholder that must not go live as-is).

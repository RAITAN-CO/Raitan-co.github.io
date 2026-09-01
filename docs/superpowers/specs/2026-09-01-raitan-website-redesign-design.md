# RAITAN website: Squarespace → static HTML on GitHub Pages

## Context

raitan.co currently runs on Squarespace. Raymond wants Claude to own ongoing
maintenance of the site going forward, which isn't practical on Squarespace's
closed editor — so the site is being ported to plain HTML/CSS served from
GitHub Pages (`RAITAN-CO/Raitan-co.github.io`), with a visual refresh along
the way rather than a pixel-for-pixel clone.

## Current site (as of 2026-09-01)

Squarespace site at raitan.co, dark-themed, kanji-brush logo treatment.

Pages: Home, About, Batteries (dropdown: SEN, TAKI, MUGEN), Solar (dropdown:
Solar Pavement), News (currently empty, no posts), Contact (working
Squarespace-backed form: First Name, Last Name, Email, Subject, Message).

ISO 9001:2015 badge shown on the homepage (Cert No. Q-047/25, issued 24 Apr
2025, QCERT Singapore).

## Decisions

### Visual direction

A hybrid of two explored directions ("Dark Technical" and "Clean
Industrial"), approved by Raymond via mockup review:

- Dark charcoal-navy ground (not pure black) — `#0c1015` page background,
  `#0f141a` / `#12171e` for raised panels
- Off-white primary text `#eef1f5`, cool-grey muted text `#8b93a1`
- Accent: safety-orange `#e0632a` (CTAs, eyebrows, highlights) — pulled from
  the industrial direction, replacing a cooler blue
- Secondary/structural color: steel blue-grey `#3d5a6c` (dividers, badge
  dots, secondary UI)
- Hairline dividers `#1c232c`
- Typography:
  - Display/headlines: **Archivo**, weight 700–800 — bold, industrial
  - Body: **IBM Plex Sans**
  - Labels/nav/eyebrows/data: **IBM Plex Mono** — carries the technical
    read throughout (nav items, badges, cert row, product tags)
- Reference mockup: published as a Claude Artifact during brainstorming
  (nav bar, hero, cert row, 5-tile product strip). Not committed to the
  repo — rebuild from this spec's tokens during implementation.

### Site structure

Same top-level IA as today, plus one new product and one restructured page:

- Home
- About (content + new Certifications section, see below)
- Batteries ▾ — SEN, TAKI, MUGEN (unchanged)
- Solar ▾ — Solar Pavement, **HELIONODE** (new)
- News (was empty static page → becomes a working blog, see below)
- Contact (form via Formspree, see below)

### HELIONODE (new product)

Standalone solar power system for powering remote sensors — twin tilted
solar panels on a pole, electronics enclosure, base cabinet (battery
storage). Reference image already provided by Raymond (pole-mounted render).
Placed under the **Solar** nav dropdown alongside Solar Pavement, since it's
a solar-first product rather than primarily a battery product.

Content status: reference image in hand; full spec copy (power output,
dimensions, sensor compatibility, etc.) still needs to be gathered from
Raymond during page-writing — do not fabricate specs, ask him directly when
drafting this page.

### About page — Certifications section

Confirmed structure: three cert cards — ISO 9001:2015, IPC-A-610J, and
IPC/WHMA-A-620E — each showing the standard name, what it covers, the cert
holder(s)/number where applicable, and a link to view the certificate PDF.

Verified via SharePoint (QualityControl site, `04_Records/4.8_Training_Records/`):

- **ISO 9001:2015** — RAITAN PTE LTD, issued by QCERT Singapore, Cert No.
  Q-047/25, issued 24 Apr 2025. PDF: `05_Certification/ISO_9001_Certificates/20250424 ISO 9001 Cert.pdf`
- **IPC-A-610J** (Certified IPC Specialist) — Raymond Tan. PDF:
  `4.8_Training_Records/RAYMOND TAN/610/CIS_IPC-A-610J_EN_Certificate of Completion IPC-A-610J Certification.pdf`
- **IPC/WHMA-A-620E** (Certified IPC Specialist) — Raymond Tan and Toh Yue
  Khing. PDFs:
  `4.8_Training_Records/RAYMOND TAN/CIS_IPCWHMA-620E_EN_Certificate of Completion IPCWHMA-A-620E Certification.pdf`
  and `4.8_Training_Records/Toh YK/YK Toh - CIS_IPCWHMA-620E_EN_Certificate of Completion IPCWHMA-A-620E Certification.pdf`

Site-wide badge (nav/footer/hero, wherever ISO 9001 currently appears alone)
becomes: **"ISO 9001:2015 CERTIFIED"** + **"IPC-A-610 & IPC/WHMA-A-620 CLASS 3
CERTIFIED"**. Do not use "IPC-610"/"IPC-620" (wrong — official names are
IPC-A-610 and IPC/WHMA-A-620) or "Certified Staff" (wrong — the credential is
per-person "Certified IPC Specialist").

### News → working blog

Becomes a real Jekyll collection (`_posts/`), newest-first, not a hand-coded
page. First post is the IPC-A-610/620 Class 3 certification announcement,
using Raymond's supplied copy verbatim (see below) — added the same way any
future post will be (a new file in `_posts/`), not a one-off special case.

Full first-post copy (Raymond's own text, to use as-is):

> **Raitan Achieves IPC-A-610 & IPC-620 Class 3 Workmanship Certification**
>
> Raitan Pte Ltd's production and quality personnel are now certified to
> IPC-A-610, Acceptability of Electronic Assemblies, and IPC/WHMA-A-620,
> Requirements and Acceptance for Cable and Wire Harness Assemblies, at
> Class 3 (High Performance/High Reliability Electronic Products).
>
> This certification is embedded within our ISO 9001:2015 Quality
> Management System, supporting Clause 7.2 (Competence) and Clause 8.5
> (Production and Service Provision), and gives our customers:
>
> - Consistent Class 3 workmanship across soldered electronic assemblies
>   and wire/cable harness builds
> - Objective, auditable evidence of operator competency, retained as
>   controlled quality records
> - Lower first-pass inspection failure rates and reduced non-conformance
>   risk on delivered assemblies
> - A workforce trained to standards recognised across defence, aerospace,
>   medical, and other high-reliability sectors
>
> This capability supports Raitan's continual improvement objectives under
> ISO 9001:2015 Clause 10.3, and reinforces our position as a supplier able
> to meet the most stringent electronics workmanship requirements for
> Singapore's defence and precision engineering industry.

### Contact form

No backend on GitHub Pages, so the existing Squarespace-hosted form is
replaced with **Formspree** (free-tier form-to-email service) — form stays
on-page, submissions land in Raymond's inbox, no server code needed. Same
fields as today: First Name, Last Name, Email, Subject, Message.

### Technical architecture

- **Jekyll**, built in natively by GitHub Pages — no local build step, no
  Node/npm dependency, no GitHub Actions workflow needed for the build
  itself.
- Shared `_layouts/default.html` (or similar) holds the nav, footer, and
  `<head>` — every page is just its content, wrapped by the layout. This is
  the reason Jekyll was chosen over hand-authored plain HTML: with 9 pages
  sharing the same chrome, a shared layout is the only way to avoid editing
  the nav in 9 places every time it changes.
- `_posts/` collection for News.
- Repo: `RAITAN-CO/Raitan-co.github.io` (already exists, currently just a
  README — confirmed correctly named for GitHub Pages auto-publish).
- **Deploy target for this round: the default GitHub Pages URL**
  (`https://raitan-co.github.io`), not the custom domain. Raymond reviews
  there first.
- **Custom domain (raitan.co) DNS change is explicitly out of scope for
  this build** — done later, separately, at Raymond's domain registrar
  (CrazyDomains), only after he's approved the site on the default URL.
  Do not touch DNS or add a `CNAME` file pointing at raitan.co until he
  asks for it.

## Out of scope / explicitly deferred

- DNS / custom domain cutover (see above)
- Final copy for Home/About beyond the Certifications section (existing
  Squarespace copy is the source to work from, but wording will be
  tightened as part of the redesign, not carried over verbatim — confirm
  specific page copy with Raymond as each page is built)
- HELIONODE full spec sheet (image in hand, technical details pending)
- Any analytics/SEO tooling beyond basic meta tags — not discussed, don't
  add without asking

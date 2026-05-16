# Handover — Accrued Carbon Website
*Date: 16 May 2026 (session 2 update)*

---

## Project overview

**Accrued Carbon** (accruedcarbon.com) — a soil carbon science and MRV consultancy website. Built in plain HTML/CSS/JS. Hosted on GitHub Pages (repo: `adibandla/accruedcarbon.com`), custom domain via Spaceship registrar. DNS fully propagated. HTTPS enforced.

Files live at: `/Users/abandla/Desktop/3_career/1_portfolio/` (git repo now initialised here, remote: `adibandla/accruedcarbon.com`)
- `index.html` — homepage (updated to list the sampling economics tool)
- `about.html` — about page (Aditya Bandla, PhD)
- `soc-permanence-tool.html` — Carbon Permanence Tool (SOC pool dynamics, tonne-year)
- `cue_soc_dashboard.html` — CUE × Soil Carbon Projects dashboard
- `soc-sampling-roi.html` — **NEW: SOC Sampling Economics tool** (see below)
- `favicon.svg` — hourglass icon, white strokes on transparent bg
- `og-image.png` — 1200×630 social sharing card (generated via Python Pillow, 2x LANCZOS downscale)
- `robots.txt` — allows all crawlers including facebookexternalhit

Git workflow: PRs via `gh`. CLI installed at `/Users/abandla/.local/bin/gh` (v2.92.0), PATH added to `~/.zshrc`, **authenticated** (account: adibandla, keyring). `gh auth setup-git` run — pushes work automatically. Use `gh pr create` for all changes going forward.

**Open PR #13**: [Add SOC Sampling Economics tool](https://github.com/adibandla/accruedcarbon.com/pull/13) on branch `claude/adoring-proskuriakova-656fd9`. Needs merge to main.

**Open PR #14**: [Add custom-input tab and anonymise dataset names in permanence tool](https://github.com/adibandla/accruedcarbon.com/pull/14) on branch `add-permanence-custom-tab`. Needs merge to main.

**Uncommitted local changes** (separate PRs needed):
- `about.html` — section label changed from "Where I can add value" to "Problems I can help you solve"
- `soc-sampling-roi.html` — EONS formula display fix (δ added, matches PR #13 code fix)
- `HANDOVER.md` — this file

---

## SOC Sampling Economics tool (`soc-sampling-roi.html`) — NEW

Interactive decision-support tool exploring how field variability (CV) drives sampling cost and project viability in soil carbon projects. Two modes:

### Carbon markets mode (VM0042)
- Sweeps spatial CV 1–75% and shows profit at EONS across the full landscape
- **Break-even CV**: maximum stock CV at which the project is still profitable at EONS
- Combined CV = √(CV²_spatial + CV²_technical) per Even et al. 2025 (SOIL journal)
- Sensitivity table: credit price × sample cost → break-even CV

### Corporate removals mode (GHG Protocol LSR)
- **Time-to-credibility framing**: how many monitoring years before cumulative SOC signal exceeds measurement uncertainty to a given confidence threshold
- Three charts: claim uncertainty vs. years (n=10–100), years-to-credibility vs. n, years-to-credibility vs. spatial CV
- Sensitivity table: sequestration rate × n → years to credibility (green ≤5yr, amber 5–10yr, red >10yr)
- Key insight: σ/annual_rate is the fundamental driver; three levers are more samples, lower-CV field selection, longer monitoring horizon

### Key math (verified, stress-tested)
VM0042 Eq. 65 uncertainty deduction:
```
unc% = (σ·f / √n / δ) × 100 × M    where M=0.4307, f=√2 (ind) or 1 (paired)
```

EONS (economic optimum — **note: δ cancels from uncertainty cost term**):
```
EONS = [G·p·M·σ·f / (2c·δ)]^(2/3)  =  [area×(44/12)×p×M×σ×f / (2c)]^(2/3)
```

Break-even CV (analytical, setting profit-at-EONS = 0):
```
σ_be = 2δ / (M·f·3√3) × √(G·p/c)
CV_be = σ_be / mean_SOC × 100%
```

Profit at EONS (analytical):
```
Profit = G·p - 3·c·EONS
```

**Bug history**: An earlier version omitted δ from EONS (used G·p instead of G·p/δ) and from break-even (used 1 instead of δ). This caused EONS to be overestimated by δ^(2/3) ≈ 1.84× at defaults (δ=2.5), and break-even CV to be underestimated by factor δ. Both fixed in PR #13.

**Formula verification status**: Seqana's EONS calculator (seqana.com/resources/eons-calculator) is a live tool but does not publicly disclose its formula — verification against their output is pending. The math derivation from first principles (cancellation of δ from the uncertainty cost term) is solid.

Removals math (`relUnc`, `yearsToCred`, `nToDetect`) verified correct — no bugs found.

### Implementation details
- Chart.js 4.4.1 with SRI hash, dark/light theme via `data-theme` on `<html>`
- Beasley-Springer-Moro inverse normal approximation for CI calculations
- Fully self-contained single HTML file (~1220 lines)
- Three Chart.js instances reused across modes (profitCVChart, beChart, eonsCVChart)

---

## Design system (apply consistently to all pages)

- **Fonts**: Inter (body), IBM Plex Mono (labels/mono), EB Garamond (logo only)
- **Google Fonts import**: `IBM+Plex+Mono:wght@400;500&family=Inter:ital,wght@0,300;0,400;0,500;1,400&family=EB+Garamond:wght@400`
- **CSS variables**:
  - Light: `--bg:#ffffff --text:#1b1e23 --muted:#808080 --border:#e0e0e0`
  - Dark: `--bg:#1b1e23 --text:#ebebec --muted:#808080 --border:#393a3d`
  - Accent palette: `--green:#7eb36a --teal:#64b9c4 --blue:#85a2f7 --purple:#bc85d9 --orange:#ea9755 --red:#f07071 --amber:#e5c07b`
- **Dark theme default**: `<html data-theme="dark">`, theme toggle button reads "Light"
- **No border-radius anywhere** — hairline 1px borders throughout
- **Nav** (identical on all pages):
  ```html
  <nav>
    <div class="nav-logo">
      <a href="index.html">[accrued] carbon</a>
      <span class="nav-tagline">Don't let your soil carbon gains turn into a liability.</span>
    </div>
    <div class="nav-links">
      <a href="about.html" class="nav-link">About</a>
      <a href="#" class="nav-link">Work</a>
      <a href="soc-permanence-tool.html" class="nav-link">Tools</a>
      <a href="#" class="nav-link">Writing</a>
    </div>
    <button class="nav-theme-btn" onclick="toggleTheme()" id="themeNavBtn">Light</button>
  </nav>
  ```
- **Page wrapper**: `<div class="page">` max-width 1080px, centered
- **Footer**: 4-column grid including Credits column — "Designed by Aditya Bandla with Claude Code"

---

## OG / social sharing

All pages have full Open Graph and Twitter Card meta tags. `og:image` points to:
```
https://raw.githubusercontent.com/adibandla/accruedcarbon.com/main/og-image.png
```
Using the raw GitHub CDN instead of the accruedcarbon.com domain because GitHub Pages blocks Facebook/WhatsApp crawler IPs at the infrastructure level. The raw CDN returns 200 to all crawlers.

Tags include `og:image:type`, `og:image:width` (1200), `og:image:height` (630) so crawlers don't need to fetch the image to determine dimensions.

**WhatsApp preview confirmed working** (tested 15 May 2026). Plain URL cache (without `?v=2`) will clear within 24–72 hours of first scrape.

---

## Security

All pages have:
- `rel="noopener noreferrer"` on all `target="_blank"` links
- SRI hashes on Chart.js CDN (`integrity="sha384-..."`)
- No geolocation JS (removed — was leaking user coordinates)
- Contact email: `aditya@accruedcarbon.com`

---

## The CUE dashboard — current state and problem

`cue_soc_dashboard.html` was the last major science task. It uses Chart.js for a scatter chart (CUE_ST vs Rh) and bar chart (biome CUE estimates), plus a project table and filter system. All JS/data is in the file.

### What the dashboard was trying to do
Overlay Cui et al. (2026) *Sci. Adv.* eadz5319's finding — that microbial CUE decouples from heterotrophic respiration (Rh) above a threshold of 340 gC m⁻² yr⁻¹ — against 10 illustrative soil carbon registry projects. The idea: projects in low-Rh (high CUE) biomes have a stronger biological basis for SOC sequestration via the microbial pathway.

### Why the premise breaks down

**The Cui et al. paper explicitly excluded managed systems** — croplands, managed pastures, fertilised plantations, agroforests — from its dataset. Four of the ten projects in the dashboard (Saskatchewan, Iowa ESMC, Indigo Ag, India Punjab) are directly excluded from the model's training domain. Several others are borderline.

This isn't a minor caveat. The literature search confirmed:

1. **Kallenbach et al. (2019)** *Front. Microbiol.* — The foundational gap paper. States explicitly: "we remain unable to predict [CUE's] response to different combinations of agricultural practices." Still largely true in 2026.

2. **Holm et al. (2025)** *J. Sustainable Ag. Environ.* — Most current meta-analysis on tillage and CUE. Starting from 4,184 papers, only **11 eligible studies** existed. Low/no-till raises CUE ~12%, but: **no evidence this translates to SOC increases**.

3. **Sokol et al. (2025)** *Sci. Adv.* eadv9482 — **Most important paper**. Using 118 soils from 15 US agricultural sites: CUE does NOT predict MAOC formation in agricultural soils. Saturation deficit and existing MAOC drive formation instead. The CUE → necromass → MAOC → SOC chain breaks at the mineral interface in managed systems.

4. **Anon. (2025)** *Soil Use & Management* — Conversion from natural to anthropogenic land use reduces CUE in all cases. Natural mixed forest → farmland: CUE falls 70.7%.

5. **Tao et al. (2023)** *Nature* 618:981 — CUE is 4× more important than other factors in global SOC storage (confirmed, citation accurate).

### Science note on CUE itself
Higher CUE = more C retained in microbial biomass per unit substrate = more necromass = potentially more MAOC. But total SOC accumulation = CUE × C input rate × necromass stabilisation efficiency × mineral capacity. High CUE in arid systems means little if C inputs are tiny. CUE is a necessary but not sufficient condition — and in managed systems (Sokol 2025), it may not be the binding constraint at all.

### Model errors in the current dashboard code
The `cueModel()` function in `cue_soc_dashboard.html` has wrong coefficients. The paper (Fig 2D) gives:
- Global below-threshold fit: **y = 0.615 − 0.001×Rh** (not 0.54 − 0.0008×Rh)
- Above threshold: CUE is **flat at 0.27**, no significant trend (R²<0.01, P=0.53) — not an exponential decay

Current code creates a non-physical discontinuity at Rh=340 (jumps from 0.27 to 0.285). Fix:
```js
function cueModel(Rh) {
  if (Rh <= 340) return Math.max(0.27, 0.615 - 0.001 * Rh);
  return 0.27;
}
```
CUE_pred values in PROJECTS are hardcoded and don't match even the wrong model. They should be computed from `cueModel(p.Rh_est)` at runtime.

### Biome misclassifications in current data
- Brazil Cerrado → currently "Tropical Dry Forest" → should be "Tropical Savanna (Cerrado)"
- India Punjab → currently "Subtropical Savanna" → should be "Subtropical Cropland"
- Kenya → currently "Hot Desert" with Rh=80 → should be "Arid Shrubland", Rh ~120–140

---

## Agreed next step: Reframe the dashboard (Option C)

The user agreed the dashboard should become a **demonstration of the white space**, not a prediction tool. The framing:

> Carbon market methodologies implicitly assume the microbial CUE pathway works similarly in managed and natural systems. The literature says it doesn't — and we don't have the data to know by how much, where, or under which practices. The dashboard maps exactly that ignorance.

**Concretely this means:**
- The scatter chart background (Cui et al. model curve) = what we know from natural ecosystems
- The project dots = where the projects sit in that landscape, but explicitly labelled as **outside the model's domain** for managed systems
- The white space between the natural-ecosystem model and the managed-system reality = the research gap
- Key callouts reference Sokol 2025 (CUE doesn't predict MAOC in agricultural soils), Holm 2025 (only 11 studies globally), Kallenbach 2019 (still unable to predict CUE from agricultural practices)
- The stat cards / header reframe from "predictions" to "situating projects in the CUE landscape"
- A prominent caveat panel explaining which projects are in/out of model scope
- The tool positions Accrued Carbon as understanding the frontier — this is the science that should inform MRV, and it currently doesn't

**What to preserve:** All JS data, Chart.js charts, filter system, project table, design system. Only the framing, text, callouts, model function, and CUE_pred derivation need changing.

---

## Pending items

- **Merge PR #13 and #14** — both need merge to main and GitHub Pages redeploy
- **CUE dashboard reframe** — highest priority science/content task (see above)
- **Writing section** — critical for career positioning (Science Lead / CSO / Director of Research targets). At least one 600–800 word opinionated analytical piece. Suggested topics: why the CUE → MAOC pathway breaks down in managed systems and what it means for VM0042, or the permanence overstatement story. Tools provide all the material; writing makes the argument explicit.
- **Cloudflare Analytics** — script not yet inserted. Get tag from Cloudflare dashboard, add before `</body>` on all pages.
- **Work and Writing pages** — not yet built, nav links are `href="#"`
- **GitHub and Terms/Privacy footer links** — still `href="#"`
- **No 404 page** — GitHub Pages serves a default; a custom `404.html` would be on-brand
- **Copyright year** — footer shows "© 2025", should be 2026
- **Calendly link** — `https://calendly.com/accruedcarbon/lets-talk-carbon` (on homepage)
- **Commit remaining local changes** — `about.html` label + `soc-sampling-roi.html` formula display (see above)

---

## Science context from session 2

### RothC / permanence tool discussion
RothC's BIO pool is a model artefact, not independently validated against fumigation-extraction MBC. Pool-level (BIO, HUM) predictions are validated mainly at Rothamsted long-term plots; validation in managed agricultural systems is weak. Carbon age from RothC is inferred from pool turnover rates, not ¹⁴C — systematically underestimates old C proportion. RothC is used for forward permanence projections in VM0042 but not for pool-stability permanence arguments.

The permanence tool (`soc-permanence-tool.html`) is unaffected by RothC issues at the model level — it takes claimed sequestration as input and applies POC/MAOC MRT discounting. The blind spots are: (1) the claimed seq inputs from LCAs often *are* derived from RothC-class models, carrying unacknowledged pool-level uncertainty; (2) the POC/MAOC split from Prairie et al. 2023 is an average — actual split depends on CUE dynamics and mineral capacity in ways the tool doesn't represent. The Breure 2025 saturation slider captures part of this. The tool is honest within its framework; the gap is upstream (input quality) and in the split imputation.

**First Milk data note**: the Dairy Producer seq figure (1.08 kg CO₂e/litre) is from measure-remeasure, not a model. This removes RothC uncertainty from the total seq input. The POC/MAOC split is still imputed from Prairie et al., not measured via fractionation.

### Portfolio / career positioning
Aditya is targeting Science Lead, CSO, Director of Research/Translation roles — private sector (integrity orgs, MRV companies, ag-carbon startups), not academic. Peer review publication record is weak; this is not a hard filter for these roles. Key differentiator is: reads frontier literature critically, translates science to implications, builds tools non-scientists can use. Writing section is the most important gap — opinionated analytical pieces (not tech docs) are the primary credibility signal for senior science leadership roles. The tools are strong but signal "doing" more than "leading"; writing fills the "can they argue a position?" gap.

---

## Key people
- **Kim Ten Wolde** — BD lead at AgriCarbon. User chatted with him. Dashboard shared as proof of concept. Keep tone rigorous but accessible.

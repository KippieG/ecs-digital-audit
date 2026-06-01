<div align="center">

# ECS European Containers — Digital Presence Audit

**Scope:** ecs.be · LinkedIn · Customer Portals · Legal & Compliance  
**Method:** Manual review · curl/HTTP inspection · page source analysis  
**Date:** June 2026 · **Author:** Philippe Godfroy

</div>

---

## Summary

| # | Finding | Area | Severity | Fix Effort |
|---|---------|------|----------|------------|
| 1 | Supply Chain Portal completely dead (DNS NXDOMAIN) | Broken functionality | 🔴 Critical | Low |
| 2 | Both customer portals use HTTP, not HTTPS | Security | 🟠 High | Low |
| 3 | T&C: no click-wrap, UK carrier doc 14 months old | Legal | 🟠 High | Medium |
| 4 | GDPR: advertising cookies fire without consent category | Compliance | 🟠 High | Low |
| 5 | CEO LinkedIn: default grey header, zero brand presence | Brand | 🟡 Medium | Trivial |
| 6 | ECS active on 2 social channels vs. 4–5 industry standard | Marketing | 🟡 Medium | Medium |
| 7 | All T&C PDFs publicly indexable via guessable paths | Info disclosure | 🟢 Low | Low |

---

## 1. 🔴 Critical — Broken Customer Portal

**Page:** [ecs.be/nl/mijn-portaalsites](https://www.ecs.be/nl/mijn-portaalsites)

The portal page lists two customer-facing links:

| Portal | URL | Status |
|--------|-----|--------|
| Intermodal Transport Portal | `http://customerportal-intermodal.ecs.be` | ✅ Reachable |
| Supply Chain Portal | `http://customerportal-supplychain.ecs.be` | 🔴 **Dead** |

```bash
$ curl -o /dev/null -s -w "%{http_code}" http://customerportal-supplychain.ecs.be
000   ← no connection at all
```

**Root cause:** `customerportal-supplychain.ecs.be` has no DNS record — the subdomain was likely decommissioned but the link on ecs.be was never updated.

**Impact:** A customer navigating to their supply chain portal hits a blank browser error page (`DNS_PROBE_FINISHED_NXDOMAIN`). No fallback, no redirect, no error message from ECS. For a company positioning itself as a supply chain partner, this is the first thing a new customer or prospect might click.

**Fix:** Update the link to the current portal URL, or add a DNS redirect.

---

## 2. 🟠 High — Customer Portals Served Over HTTP

Both portal links on the ECS website use `http://` — not `https://`:

```
http://customerportal-intermodal.ecs.be
http://customerportal-supplychain.ecs.be
```

Customer portal sessions (login, shipment data, track & trace) transmitted over unencrypted HTTP are vulnerable to interception. Modern browsers also flag HTTP pages with a "Not secure" warning, which undermines trust at login.

**Fix:** Force HTTPS on both subdomains and update the links on ecs.be.

---

## 3. 🟠 High — Terms & Conditions: No Click-Wrap, Outdated UK Document

**Page:** [ecs.be/nl/algemene-voorwaarden](https://www.ecs.be/nl/algemene-voorwaarden)

### 3a. No click-wrap acceptance

All six T&C documents are offered as plain PDF downloads with no acceptance mechanism:

```
❌ No checkbox ("I have read and accept the terms")
❌ No confirmation button before download
❌ No logged acceptance timestamp
✅ PDF is accessible to anyone, accepted or not
```

Without click-wrap, proving a customer accepted the current version of the T&C in a dispute is difficult. Belgian and UK courts both require demonstrable consent for standard contract terms.

### 3b. Document version matrix

| Stakeholder | Entity | Last Updated | Flag |
|-------------|--------|--------------|------|
| Customer | ECS NV / 2XL NV | Sep 2025 | ✅ |
| Customer | ECS Trucking BV | Aug 2025 | ✅ |
| Customer | 2XL France SAS | Aug 2025 | ✅ |
| Supplier & Contractor | ECS NV / 2XL NV | Sep 2025 | ✅ |
| Transport Carrier (non-UK) | ECS NV / 2XL NV | Aug 2025 | ✅ |
| Transport Carrier (UK) | ECS NV / 2XL NV | **Jul 2024** | ⚠️ 14 months old |

The UK carrier T&C has not been updated since July 2024. The UK's Road Haulage Association issued updated guidance on post-Brexit cabotage rules and HMRC import procedures in late 2024 and early 2025. A T&C document that predates these changes may no longer reflect current ECS obligations to UK carriers.

**Fix:** Add click-wrap acceptance per document type. Review UK carrier T&C against current HMRC/RHA guidelines.

---

## 4. 🟠 High — GDPR Cookie Consent Gap

**Source:** Page source of ecs.be — `eu_cookie_compliance` config block

### What the cookie banner shows

The cookie consent popup on ecs.be presents **two** consent categories:

- ✅ Functionele en statistische cookies *(required, pre-ticked)*
- ☐ Analytische cookies *(optional)*

### What the cookie config actually loads

Extracted from the page source (`allowed_cookies` field):

```
functional:   BIGipServer, lang, _Ifa, Iissc
analytics:    _ga, _gid, _gat, hubspotutk, nQ_cookieID, Leadfeeder
advertisement: _fbp (Facebook pixel), fr (Facebook), bscookie (LinkedIn), mc
social_media:  lidc (LinkedIn Insight Tag), tr
```

**The gap:** `advertisement` and `social_media` cookies — including the **Facebook pixel** (`_fbp`) and **LinkedIn Insight Tag** (`lidc`) — are listed in the config but have **no corresponding consent category in the UI**. Users who click "Functionele cookies aanvaarden" or manage only functional/analytics cookies are not presented with an advertising opt-in.

Under GDPR Art. 6(1)(a) and the ePrivacy Directive, advertising and social tracking cookies require explicit opt-in consent. Loading them without a visible, dedicated consent toggle is a compliance gap.

Additionally: the cookie policy version in the page source reads `"cookie_policy_version":"1.0.0"` — suggesting the policy has never been formally versioned or updated since launch.

**Fix:** Add an "Advertentie & Social Media" consent category to the banner. Gate `_fbp`, `lidc`, `bscookie`, `fr` behind that category. Increment the policy version on each update.

---

## 5. 🟡 Medium — CEO LinkedIn: No Brand Presence

**Profile:** [linkedin.com/in/richarddehaas](https://www.linkedin.com/in/richarddehaas/)  
**Role:** CEO, ECS European Containers

### Before

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│              [ default LinkedIn grey ]                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
  ◉ Richard de Haas
    CEO at ECS European Containers
```

*→ See [`screenshots/ceo-linkedin-before.png`](screenshots/ceo-linkedin-before.png)*

The CEO's profile shows ECS as employer but carries LinkedIn's default grey banner — no logo, no brand colour, no tagline, no imagery from Zeebrugge.

Every recruiter, customer, investor, or partner who clicks through from "ECS European Containers" on LinkedIn lands on an unbranded executive page. The company page itself is correctly branded — but the CEO profile, typically the highest-traffic individual page for any company, is not.

### After (proposed)

```
┌─────────────────────────────────────────────────────────┐
│  ████████████████████████████████████████████████████  │
│  █  [ECS LOGO]    Together We Excel         [PORT]   █  │
│  █  European Containers · Zeebrugge · Since 1994     █  │
│  ████████████████████████████████████████████████████  │
└─────────────────────────────────────────────────────────┘
  ◉ Richard de Haas
    CEO at ECS European Containers
```

*Colours: `#8D1D45` (ECS red) · `#F8CE3E` (ECS yellow) — matches existing brand assets*

**Fix:** Upload a branded LinkedIn banner (1584 × 396px). Effort: 15 minutes.

---

## 6. 🟡 Medium — Social Media: Two Channels

ECS's footer links two channels:

| Platform | Handle | Status |
|----------|--------|--------|
| LinkedIn | [ECS European Containers](https://www.linkedin.com/company/ecs-european-containers) | ✅ Active |
| Facebook | [ECS.TogetherWeExcel](https://www.facebook.com/ECS.TogetherWeExcel) | ✅ Active |
| Instagram | — | ❌ Not present |
| YouTube | — | ❌ Not present |
| Twitter / X | — | ❌ Not present |

### Missed opportunity

Port and logistics content performs exceptionally on Instagram and YouTube: crane operations, truck loading time-lapses, Super Mega Trailer visuals, Zeebrugge harbour footage. These are inherently visual formats that require no scripted production — ECS already has the location and the equipment.

Competitor and partner benchmarks show 4–5 active channels is standard among comparable European logistics operators. Two-channel presence limits organic reach and employer branding for recruitment (a stated priority on the ECS careers page).

**Fix:** Low-effort: repurpose existing photography/video to Instagram. Medium-effort: a quarterly YouTube format ("A day at ECS", "Super Mega Trailer explained", "Brexit customs in 90 seconds").

---

## 7. 🟢 Low — T&C PDFs Fully Indexable

**robots.txt** on ecs.be does not disallow `/sites/default/files/`:

```robotstxt
# From https://www.ecs.be/robots.txt
User-agent: *
Disallow: /admin/
Disallow: /core/
Disallow: /user/login
# ← no Disallow for /sites/default/files/
```

All T&C documents are served from paths with an embedded date:

```
/sites/default/files/2025-09/general_conditions_ecs_nv_2xl_nv_customer_nl_2.pdf
/sites/default/files/2025-08/general_conditions_ecs_trucking_bv_nl.pdf
/sites/default/files/2024-07/general_conditions_ecs_nv_2xl_nv_carrier_uk_eng.pdf
```

The naming convention makes all historical document versions guessable and indexable. Combined with the absence of version numbers in the filenames (only dates), it's unclear from the URL whether a given document is current or superseded.

**Fix:** Add `Disallow: /sites/default/files/` to robots.txt if these files should not be indexed. Alternatively, serve T&C documents through a versioned, stable URL (`/terms/customer/current`) and redirect old URLs.

---

## Methodology

```bash
# Verify broken portal
curl -o /dev/null -s -w "%{http_code}" http://customerportal-supplychain.ecs.be
# → 000 (no DNS record)

# Inspect portal URLs on site
curl -s https://www.ecs.be/nl/mijn-portaalsites | grep -i "customerportal"

# Check HTTP headers on main site
curl -IL https://www.ecs.be/nl | grep -i "strict-transport\|content-security\|x-frame"

# Retrieve T&C page with links
curl -s https://www.ecs.be/nl/algemene-voorwaarden | grep -i "pdf\|download"

# Read robots.txt
curl https://www.ecs.be/robots.txt

# GDPR config extracted from page source
# eu_cookie_compliance JSON block → allowed_cookies field
```

---

## Related: What I Built While Auditing

While mapping ECS's digital ecosystem, I built tools designed to run **directly on ECS's existing infrastructure** (TAS terminal system, Microsoft Business Central):

| Project | What it does | Stack |
|---------|-------------|-------|
| [**eco-match-engine**](https://github.com/KippieG/eco-match-engine) | Eliminates empty return mileage by AI-matching open trips. Integrates with TAS + Business Central. Estimated 15–25% reduction in empty km. | Python · FastAPI · Power Platform |
| [**delay-dna**](https://github.com/KippieG/delay-dna) | Predicts shipment delays hours or a full day before they happen. Combines weather, ferry schedules, customs flags, and historical delay patterns per route. | React · Node.js · ML |
| [**ecs-ecoload**](https://github.com/KippieG/ecs-ecoload) | Super Mega Trailer load optimizer, live reefer container monitoring via SignalR, Brexit customs document validator. | .NET 10 · Angular 17 · DDD · Docker |

---

<div align="center">

**Philippe Godfroy** · [philgodf@gmail.com](mailto:philgodf@gmail.com)

</div>

# ECS European Containers — Digital Presence Audit

**Scope:** ecs.be · LinkedIn company page · CEO LinkedIn profile  
**Method:** Manual review + automated link-checking  
**Author:** Philippe Godfroy

---

## Findings

### 1. Broken Links — ecs.be

| Location | Broken URL | HTTP Status | Impact |
|----------|------------|-------------|--------|
| [Page / section] | `https://www.ecs.be/[path]` | 404 | Dead navigation |
| [Page / section] | `https://www.ecs.be/[path]` | 404 | Dead navigation |
| [Page / section] | `https://www.ecs.be/[path]` | 404 | Dead navigation |

> All broken URLs confirmed with `curl -o /dev/null -s -w "%{http_code}" <url>`.

---

### 2. Terms & Conditions — Publicly Accessible Without Gate

**URL:** `https://www.ecs.be/[terms-path]`  
**Status:** 200 OK — no authentication, no click-wrap, no version header  

The T&C document is directly accessible via a guessable path and is not versioned. No `Last-Modified` header is returned by the server, meaning there is no audit trail of when terms were last updated — a compliance risk for GDPR Article 13 obligations.

```
HTTP/2 200
content-type: application/pdf
cache-control: public, max-age=86400
last-modified: [not present]
```

---

### 3. Open / Unprotected Pages

| URL | What is exposed | Risk |
|-----|-----------------|------|
| `https://www.ecs.be/[path]` | [Description of exposed content] | Medium |
| `https://www.ecs.be/[path]` | [Description of exposed content] | Low |

Pages respond with `200 OK` and are indexed by Google (`site:ecs.be` search returns them). No robots.txt disallow rule is present for these paths.

```
# https://www.ecs.be/robots.txt — relevant excerpt
User-agent: *
Disallow:          ← no rules covering the above paths
```

---

### 4. Brand Inconsistency — LinkedIn

#### 4a. Company Page

| Check | Status |
|-------|--------|
| Logo present | ✅ |
| Banner image | ⚠️ [note any issue] |
| About section complete | ⚠️ [note any issue] |
| Contact info | ⚠️ [note any issue] |

#### 4b. CEO Profile — No Company Banner

The CEO's LinkedIn profile lists ECS European Containers as current employer but carries a **default LinkedIn grey banner** — no ECS branding, no Zeebrugge imagery, no company colours.

For a company that markets itself as market leader in UK consolidation, the most visible executive profile on LinkedIn communicates no brand identity. Every recruiter, customer, or partner who visits the profile lands on an empty grey header.

**Screenshot evidence:** *(attach screenshot)*

---

### 5. Methodology

```bash
# Broken link scan
wget --spider -r -nd --delete-after -o crawl.log https://www.ecs.be 2>&1
grep -i "broken\|404\|error" crawl.log

# HTTP header inspection
curl -I https://www.ecs.be/[path]

# robots.txt
curl https://www.ecs.be/robots.txt

# Google index check
# Searched: site:ecs.be [path keywords]
```

---

### 6. Summary

| Finding | Severity | Effort to fix |
|---------|----------|---------------|
| Broken links (navigation) | Medium | Low — 404 redirects or content restore |
| Unversioned T&C document | Medium | Low — add version + Last-Modified header |
| Open pages indexed by Google | Low–Medium | Low — robots.txt disallow or auth gate |
| CEO LinkedIn — no banner | Low | Trivial — upload a banner image |

---

*Philippe Godfroy · [philgodf@gmail.com](mailto:philgodf@gmail.com)*

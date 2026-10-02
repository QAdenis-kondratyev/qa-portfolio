# Web QA Audit & UX Analysis — Yuzu Asian Kitchen (Live SPA)

[![Testing Type](https://img.shields.io/badge/Testing-Exploratory%20%7C%20Functional%20%7C%20BVA-blue.svg)](#test-methodology--execution-rounds)
[![Platform](https://img.shields.io/badge/Target-Live%20Production%20SPA-green.svg)](#)
[![Artifacts](https://img.shields.io/badge/Deliverables-STR%20%7C%20Jira%20%7C%20Checklist%20%7C%20Presentation-orange.svg)](#deliverables)
[![Defects](https://img.shields.io/badge/Defects-16%20(1%20Critical%2C%203%20High%2C%2012%20Medium)-red.svg)](#defect-distribution--impact-analysis)

A comprehensive quality audit and end-to-end exploratory assessment of **Yuzu Asian Kitchen** (`yuzuasiankitchen.orderss.co.il`) — a live production food-ordering single-page application (SPA) serving multiple branches (Haifa, Rishon LeZion) in 6 languages (Hebrew, English, French, Spanish, Russian, Arabic).

> Capstone project — Software QA & Automation course (TCQA46), Technion · July–August 2026

---

## 📌 Executive Summary

| Metric | Value |
|---|---|
| Checklist items executed | ~75 (across 3 rounds) |
| Raw findings logged | 31 |
| Unique confirmed defects | **16** (1 Critical · 3 High · 12 Medium) |
| Languages verified | 6 |
| Branches cross-checked | 2 |
| Release verdict | **No-Go** until BUG-012 is fixed |

**Key highlights**

- **Critical revenue blocker isolated (BUG-012):** dismissing the drink-upsell modal wipes the entire cart (stored in `localStorage`) — orders are lost at the moment of highest purchase intent.
- **Defect triage & deduplication:** 31 raw findings consolidated into 16 unique defects; repeated localization findings across pages traced to a shared, template-level i18n root cause instead of being reported as separate one-offs.
- **Boundary Value Analysis:** checkout fields have no length limits or input sanitization — the order comment field accepted **34,012 characters**, and the name field accepted digits and raw HTML tags.
- **Stakeholder reporting:** 16-slide executive presentation with competitive UX benchmarking (Nini Hachi, The Sushi Co) and a clear Go/No-Go release recommendation.

---

## 🛠 Test Methodology & Execution Rounds

**Environment:** Google Chrome (desktop) · Windows 11 · Chrome DevTools (Network, Application/Storage, Console, Elements)
**Techniques:** Exploratory testing · Checklist-based testing · Boundary Value Analysis · Equivalence Partitioning · State-transition testing (cart / browser history)

| Round | Focus Areas | Checklist Items | Raw Findings | Unique Defects | Severity |
|---|---|---|---|---|---|
| Round 1 | Initial walkthrough: usability, navigation, GUI, links, localization (6 languages) | ~45 | 25 | 10 | 2 High · 8 Medium |
| Round 2 | Deep dive: cart state, search, coupon codes, Back/Forward history, storage | ~20 | 3 | 3 | 1 Critical · 1 High · 1 Medium |
| Round 3 | Boundary testing: form fields, character sanitization, cross-branch verification | ~10 | 3 | 3 | 3 Medium |
| **Total** | **Full application scope** | **~75** | **31** | **16** | **1 Critical · 3 High · 12 Medium** |

---

## 🐞 Defect Distribution & Impact Analysis

### Defects by Area

| Area | Defects | Share | Typical issues |
|---|---|---|---|
| Cart & Checkout | 8 | 50% | State persistence, upsell modal, duplicate lines, coupon logic |
| Localization / i18n | 4 | 25% | Untranslated legal pages, hard-coded strings, placeholder text |
| Input Validation | 2 | 12.5% | No `maxlength`, no client-side sanitization |
| Search & Menu | 2 | 12.5% | Empty search state, broken product images in edit modal |

### 🚨 BUG-012 [CRITICAL] — Declining the Drink Upsell Modal Purges the Shopping Cart

**Preconditions:** Cart is empty; user is on the menu page of any branch.

**Steps to Reproduce:**
1. Open a customizable dish (e.g., Tom Yam Soup) and select modifiers/sauces.
2. Confirm the dish — it appears in the cart (qty 1).
3. The system opens an automatic drink-upsell modal.
4. Close the modal with the **✕** button (no "No thanks" option exists).

**Expected Result:** The modal closes; the cart keeps all previously added items; the user can proceed to checkout.

**Actual Result:** The cart is emptied (0 items, ₪0.00). No warning, no undo, no recovery.

**Investigation:** In DevTools → Application → Local Storage, the cart object (`__mobx_sync__`) is cleared at the moment the modal is dismissed — the close action resets cart state instead of only closing the modal component.

**Business impact:** Direct order loss at peak purchase intent; reproducible 100%.

### ⚠️ BUG-014 / BUG-016 [MEDIUM] — Missing Length Limits & Input Sanitization at Checkout

- **BUG-014:** The order comment `<textarea>` has no `maxlength`. It accepted a 34,012-character payload, which was sent to the backend without truncation.
- **BUG-016:** The customer name field accepts purely numeric values and raw HTML tags, which are then rendered back in the UI.

**Risk:** Data-quality issues in order records, UI breakage, and a potential injection surface if output encoding is also missing server-side.

---

## 📦 Deliverables

| Artifact | Format | Description |
|---|---|---|
| Software Test Report (STR) | PDF / DOCX (EN + HE) | Scope, environment, execution summary, defect statistics, release verdict |
| Bug Report | Excel + Jira | 16 defects with steps, expected/actual, severity, priority, screenshots |
| Web Testing Checklist | Excel | ~75 checklist items with Pass/Fail status per round |
| Executive Presentation | PPTX (16 slides) | Findings, UX benchmarking vs. competitors, Go/No-Go recommendation |

---

## 🧰 Tools

`Jira` · `Chrome DevTools` · `Excel` · `Word` · `PowerPoint`

---

## 💡 Key Takeaways

- In SPAs, cart and session bugs often live in client-side storage — DevTools inspection turns "the cart disappeared" into a precise, developer-ready root cause.
- Deduplicating findings by root cause produces a shorter, more actionable report than listing every symptom.
- Severity should be judged by business impact: a single cart-wipe defect outweighs a dozen cosmetic issues and justifies a No-Go.

---

**Author:** Denis Kondratyev — QA Engineer (Manual & Automation)
[GitHub](https://github.com/QAdenis-kondratyev)

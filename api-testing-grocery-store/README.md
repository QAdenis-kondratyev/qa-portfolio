# Simple Grocery Store API — Manual & Data-Driven API Testing

![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)
![Newman](https://img.shields.io/badge/Newman-CLI-orange)

**Type:** REST API testing — course project, Technion *Software QA & Automation* (TCQA46), Sep 2026
**Target:** [simple-grocery-store-api.click](https://simple-grocery-store-api.click) — public REST API (products, cart, orders, auth)
**Tools:** Postman, Collection Runner, JavaScript (`pm.*` API), CSV data files, Newman
**Techniques:** Equivalence Partitioning, Boundary Value Analysis, negative testing (incl. invalid/missing token, SQL-injection input), data-driven testing

---

## Summary

| Metric | Value |
|---|---|
| Areas covered | Products · Cart · Auth · Orders |
| Test cases | **53** — 50 manual + 3 data-driven automated suites |
| Automated runs | **18** CSV iterations → **54** assertions |
| Defects found | **4** — all input-validation gaps |

## Defects

| ID | Defect | Severity | Endpoint |
|---|---|---|---|
| DEF-01 | `results` accepts `0`, non-numeric and out-of-range values — no `400`, silently returns default list | Medium | `GET /products` |
| DEF-02 | `available` accepts non-boolean values — no error, filter silently ignored | Low-Medium | `GET /products` |
| DEF-03 | `clientName` accepts a purely numeric value (e.g. `12345678`) | Low | `POST /api-clients` |
| DEF-04 | `customerName` accepts an empty string on update, although creation rejects it | Medium | `PATCH /orders/{orderId}` |

**Root cause pattern.** The API validates some constraints correctly (upper bound on `results`, duplicate email, auth) but skips others (lower bound, type, format, non-empty on update). The 4 defects point to one systemic gap — inconsistent server-side validation — rather than 4 isolated bugs.

## Automation

| Suite | Data file | Iterations | Assertions | Passed | Failed |
|---|---|---|---|---|---|
| Category filter | `data/category.csv` | 7 | 21 | 21 | 0 |
| `results` boundary | `data/results_boundary.csv` | 5 | 15 | 13 | 2 → reproduces DEF-01 (`results=0`) |
| Invalid registration | `data/auth_invalid_registration.csv` | 6 | 18 | 16 | 2 → reproduces DEF-03 (`clientName=12345678`) |

Each iteration checks status code, response time (< 1000 ms) and response content (category match / item count / error message).

How it works:
- Each CSV row = one test input **plus its expected outcome**; assertions branch on the expected column.
- A pre-request script removes a JSON key entirely for "missing field" cases (`OMIT` marker) — `{{variable}}` substitution alone can't drop a key.
- The one row with a valid email gets a `Date.now()` suffix, so repeated runs against the live API don't fail on duplicate email.

## How to run

```bash
npm install -g newman

newman run postman/grocery-store-api-tests.postman_collection.json \
  -e postman/grocery-store.postman_environment.json \
  --folder "Product results filter (data-driven)" \
  -d data/results_boundary.csv
```

## Repository contents

| Path | Content |
|---|---|
| `postman/grocery-store-api-tests.postman_collection.json` | Full collection: Products, Cart, Auth, Orders, Automation |
| `postman/grocery-store.postman_environment.json` | Environment (`base_url`) |
| `data/*.csv` | Data files for the 3 automated suites |
| `docs/Test_Case_Report.pdf` | All 53 test cases with steps, expected/actual results, status |
| `docs/AI_Usage_Note.pdf` | Transparency note on AI assistance in this project |

## AI usage

Test design, execution and defect discovery are my own work. AI assistance was used to learn Postman scripting mechanics and for document formatting — details in `docs/AI_Usage_Note.pdf`.

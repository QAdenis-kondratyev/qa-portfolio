# REST API Testing — Simple Grocery Store API (Manual + Data-Driven Automation)

[![Testing Type](https://img.shields.io/badge/Testing-API%20%7C%20Functional%20%7C%20Negative%20%7C%20BVA-blue.svg)](#test-scope--coverage)
[![Tool](https://img.shields.io/badge/Tool-Postman%20%7C%20JavaScript-orange.svg)](#automation-approach)
[![Test Cases](https://img.shields.io/badge/Test%20Cases-53%20(50%20manual%20%2B%203%20automated)-green.svg)](#test-scope--coverage)
[![Defects](https://img.shields.io/badge/Defects-4%20confirmed-red.svg)](#defects-found)

Manual and data-driven automated testing of the **Simple Grocery Store API** (`https://simple-grocery-store-api.click`) — a public REST API covering product catalog, carts, client authentication and orders.

> Course project — Software QA & Automation course (TCQA46), Technion · September 2026

---

## 📌 Executive Summary

| Metric | Value |
|---|---|
| Endpoint areas covered | 5 (Products, Cart, Auth, Orders, Automation) |
| Test cases designed & executed | **53** (TC-01–50 manual · TC-51–53 automated suites) |
| Automated data-driven iterations | 54 (3 CSV-driven suites) |
| Confirmed defects | **4** (DEF-01–04) |
| Root-cause analysis | 4 defects traced to one systemic issue — inconsistent server-side validation |

**Key highlights**

- **Full CRUD flow coverage:** products → cart → client registration/token → order create/update/delete, including positive, negative and boundary cases.
- **Data-driven automation in Postman:** CSV datasets with an *expected-outcome* column, so one request validates both valid and invalid inputs in a single run.
- **Defects reproduced automatically:** the automated suites re-detect DEF-01 and DEF-03 on every run — ready to act as regression checks once fixed.
- **Systemic finding instead of isolated bugs:** the API correctly enforces some constraints (upper bound on `results`, duplicate email, auth) but skips others (type/lower-bound checks, boolean typing, format and non-empty checks on update).

---

## 🗂 Test Scope & Coverage

| Area | Endpoints | Focus |
|---|---|---|
| Products | `GET /products`, `GET /products/{id}` | Filters (`category`, `results`, `available`), invalid IDs, response schema |
| Cart | `POST /carts`, `GET/POST/PATCH/DELETE /carts/{id}/items` | Add/update/remove items, invalid product IDs, quantity limits |
| Auth | `POST /api-clients` | Registration, duplicate email, invalid name/email, token issuance |
| Orders | `POST/GET/PATCH/DELETE /orders` | Bearer token required, create/update/delete, field validation |
| Automation | TC-51–53 | Data-driven negative & boundary suites |

**Techniques:** Equivalence Partitioning · Boundary Value Analysis · Negative testing · Status-code and response-body validation · State verification across chained requests

---

## 🐞 Defects Found

| ID | Title | Severity | Endpoint |
|---|---|---|---|
| DEF-01 | `results` accepts invalid values (0, non-numeric, out-of-range) without `400` — silently falls back to the default list | Medium | `GET /products` |
| DEF-02 | `available` accepts non-boolean values — no error, filter silently ignored | Low–Medium | `GET /products` |
| DEF-03 | `clientName` accepts a purely numeric value (e.g. `"12345678"`) without format validation | Low | `POST /api-clients` |
| DEF-04 | `customerName` accepts an empty string on update — inconsistent with `POST` validation | Medium | `PATCH /orders/{orderId}` |

### Systemic Observation

All four defects share one root cause: **server-side validation is partial and inconsistent across endpoints and methods.** The same field can be validated on `POST` but not on `PATCH`, and upper bounds are checked while lower bounds and types are not. Recommendation: a single shared validation layer (schema-based) applied to every write and filter parameter.

---

## 🤖 Automation Approach

Built in **Postman Collection Runner** with JavaScript test scripts:

- **Iteration data from CSV** — `pm.iterationData.get(...)` feeds inputs *and* expected outcomes per row.
- **Branching assertions** — `pm.test()` checks for `2xx` on valid rows and `4xx` on invalid rows, based on the expected-outcome column.
- **Conditional request body** — a Pre-request Script removes a JSON key when the CSV value is `OMIT`, enabling "missing field" tests (plain `{{variable}}` substitution can't drop a key).
- **Idempotent runs against a live API** — unique test data generated at runtime (`Date.now()` suffix), so suites can be re-run without collisions.

| Suite | Dataset | Result | Notes |
|---|---|---|---|
| TC-51 · Category filter | `data/category.csv` | 21 / 21 pass | Valid + invalid categories |
| TC-52 · `results` boundary | `data/results_boundary.csv` | 13 / 15 pass | 2 failures reproduce **DEF-01** |
| TC-53 · Invalid registration | `data/auth_invalid_registration.csv` | 16 / 18 pass | 2 failures reproduce **DEF-03** |

---

## 📦 Repository Contents

```
api-testing-grocery-store/
├── Grocery Store API Tests.postman_collection.json   # Products / Cart / Auth / Orders / Automation
├── Grocery Store.postman_environment.json            # base_url and runtime variables
├── data/
│   ├── category.csv
│   ├── results_boundary.csv
│   └── auth_invalid_registration.csv
├── docs/
│   ├── Test Case Document.docx                       # 53 test cases, 5 sections
│   └── Defect Summary.docx                           # DEF-01–04 + Systemic Observation
└── README.md
```

## ▶️ How to Run

1. Import the collection and environment into Postman.
2. Select the **Grocery Store** environment.
3. Run the *Auth* folder first to register a client and store the access token.
4. To run a data-driven suite: **Collection Runner** → select the request from the *Automation* folder → **Data** → choose the matching CSV from `data/` → **Run**.

Optional CLI run with Newman:

```bash
npm install -g newman
newman run "Grocery Store API Tests.postman_collection.json" \
  -e "Grocery Store.postman_environment.json" \
  --folder "Automation" \
  -d data/results_boundary.csv
```

---

## 🧰 Tools

`Postman` · `JavaScript (pm API)` · `CSV data files` · `Word` · `Excel`

---

## 💡 Key Takeaways

- A `200 OK` is not a pass — silently ignoring an invalid parameter is a defect because the client never learns its request was wrong.
- Comparing validation between `POST` and `PATCH` for the same field is a cheap, high-yield check.
- Putting the expected outcome into the dataset turns one request into a full positive + negative suite.

---

**Author:** Denis Kondratyev — QA Engineer (Manual & Automation)
[GitHub](https://github.com/QAdenis-kondratyev)

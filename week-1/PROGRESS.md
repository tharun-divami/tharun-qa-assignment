# QA Excellence — Week 1 Assessment Progress Tracker

**Assignment:** Hands-On QA Assessment — B2B Vendor Invoice Management Portal  
**Source Material:** `01_QA_Excellence.pptx`  
**Folder:** `week-1/`  
**Started:** 2026-06-06

---

## Activity Log

| # | Date | Activity | Status | Output File |
|---|------|----------|--------|-------------|
| 1 | 2026-06-06 | Extracted and reviewed all 39 slides from PPT | ✅ Done | — |
| 2 | 2026-06-06 | Created assessment plan (in conversation) | ✅ Done | — |
| 3 | 2026-06-06 | Requirements Clarification Document v1 (pre-PRD, archived) | ✅ Archived | `02_requirements_clarification.md` |
| 4 | 2026-06-06 | PRD — Vendor Invoice Management Portal (US-VIMP-001 to 019, BR-VIMP-001 to 011, 14 Open Questions) | ✅ Done | `02_prd.md` |
| 5 | 2026-06-06 | Requirements Clarification Document v2 — references PRD IDs (US/BR/OQ/AC) | ✅ Done | `03_requirements_clarification.md` |
| 6 | 2026-06-06 | PRD v2 — AC section fully expanded: exact test data, BVA boundaries, security ACs, negative paths (AC-VIMP-001 to 009, ~70 individual ACs) | ✅ Done | `02_prd.md` (updated) |
| 7 | 2026-06-06 | Comprehensive Test Plan — all 8 QA phases, ~280 test cases, 11 modules, Decision Tables, BVA summary, E2E scenarios, Smoke set, OWASP, WCAG, Load tests | ✅ Done | `04_test_plan.md` |
| 8 | 2026-06-06 | Merged clarification docs — v1 deprecated, v2 upgraded to v3 with 4 additional unique questions from v1 | ✅ Done | `03_requirements_clarification.md` (v3) |
| 9 | 2026-06-06 | Extreme & Edge Case Test Cases — 78 detailed TCs across 12 modules: race conditions, injection, Unicode, boundary, OWASP, HSTS, CSP, soak, DST, BOLA, JWT alg:none, etc. | ✅ Done | `05_test_cases.md` |
| 10 | — | RTM / Bug Reports — deferred (no application yet) | ⏸ Deferred | — |
| 8 | — | Requirements Traceability Matrix (RTM) | 🔲 Pending | `06_rtm.md` |
| 9 | — | Bug Report Template + Samples | 🔲 Pending | `07_bug_reports.md` |
| 10 | — | Non-Functional & Integration Testing Notes | 🔲 Pending | `08_nft_integration.md` |
| 11 | — | Deployment Verification Checklist | 🔲 Pending | `09_deployment_checklist.md` |

---

## File Index

| File | Type | Description |
|------|------|-------------|
| `01_QA_Excellence.pptx` | Source | Training PPT — all 39 slides |
| `02_prd.md` | PRD | Product Requirements Document — 19 user stories, 11 business rules, 9 AC groups, 14 open questions |
| `02_requirements_clarification.md` | Archived | v1 clarification doc (pre-PRD, superseded) |
| `03_requirements_clarification.md` | Active | v2 clarification doc — every question anchored to PRD US/BR/OQ/AC IDs |
| `04_test_plan.md` | ✅ Active | Comprehensive Test Plan — ~280 standard TCs, all 8 phases, all types |
| `05_test_cases.md` | ✅ Active | 78 extreme/edge case TCs — full detail, all layers, 12 modules |

---

## PRD Quick Reference

**Module:** VIMP (Vendor Invoice Management Portal)  
**Roles:** R-01 Vendor | R-02 AP User | R-03 AP Admin | R-04 Finance User | R-05 System Admin

| ID Range | Type | Count |
|----------|------|-------|
| US-VIMP-001 to 019 | User Stories | 19 |
| BR-VIMP-001 to 011 | Business Rules | 11 |
| AC-VIMP-001 to 009 | Acceptance Criteria Groups | 9 (22 individual ACs) |
| OQ-01 to OQ-14 | Open Questions | 14 |

---

## Prompts & Decisions Log

### 2026-06-06 — Session Start
**User prompt:** Reviewed `01_QA_Excellence.pptx`, noted all training content, asked to create a plan and proceed with assessment.  
**Decision:** Extracted 39 slides, identified assessment requirements on slides 31–32. Created plan covering deliverable files.

### 2026-06-06 — Requirements Clarification v1
**User prompt:** Start with generating the requirement document. Track all activity in a markdown file within week-1.  
**Decision:** Created `PROGRESS.md` as tracker. Generated `02_requirements_clarification.md` (v1) with 100+ questions directly from PPT requirements.

### 2026-06-06 — PRD + Requirements Clarification v2
**User prompt:** `/daksh` — Create a PRD document first, then have the requirements clarification document refer to PRD requirements.  
**Decision:** Created `02_prd.md` following Daksh PRD format (US-VIMP-NNN, BR-VIMP-NNN, AC-VIMP-NNN with Given/When/Then, data contract, open questions, approval block). Rewrote requirements clarification as `03_requirements_clarification.md` (v2) — every question now anchored to a specific PRD ID. Old v1 file archived in place.

### 2026-06-06 — Merge Clarification Docs + Extreme Test Cases
**User prompt:** `/daksh` — Merge two clarification docs into one. Skip RTM and bug reports. Create detailed test cases and scenarios covering all possible extreme and edge cases.  
**Decision:**
1. Deprecated `02_requirements_clarification.md` (v1, pre-PRD). Added 4 unique questions from v1 into `03_requirements_clarification.md` (upgraded to v3 — single canonical file): "Remember Me" option, password self-reset, vendor profile update rules, per-day/month submission limits, withdrawal before AP review, AP team invoice assignment.
2. Created `05_test_cases.md` — 78 extreme/edge case test cases in full detail across 12 modules. Each case has: exact Precondition, Test Data (specific values), numbered Steps, and Expected Result spanning UI + API + DB layers. Techniques used: race condition simulation, OWASP attack vectors (IDOR/BOLA, JWT alg:none, Host Header Injection, Path Traversal, Open Redirect, Clickjacking, CSP), concurrency conflicts, boundary values at system limits, Unicode/emoji in all fields, malformed API payloads, integration failure modes (timeout, duplicate callbacks, out-of-order events), DST/timezone boundary, and business logic exploits (over-invoicing, resubmit rejected invoice, PO concurrent closure).

### 2026-06-06 — Expanded ACs + Comprehensive Test Plan
**User prompt:** `/daksh` — More detailed ACs (especially acceptance criteria). Test plan must be precise and detailed with all possible test cases and scenarios. Refer to PPT for QA excellence, don't miss any aspects.  
**Decision:**  
1. Replaced PRD Section 6 (ACs) entirely — expanded from 22 high-level ACs to ~70 detailed ACs with: exact test data values, BVA boundaries (★), EP representatives (⊕), negative paths, security scenarios (XSS, SQLi, IDOR, CSRF, JWT tampering), concurrency/race conditions, and state transition edge cases.  
2. Created `04_test_plan.md` — the primary QA artifact. Covers all 8 QA lifecycle phases from PPT, all testing types from Slide 9 (Functional + Non-Functional), all test design techniques from Slide 6 (EP, BVA, Decision Tables, State Transition), ~280 test cases across 11 modules (Auth, Invoice, Approval, Payment, Notifications, Reports, RBAC, API, DB, NFT, Integration), 3 Decision Tables, BVA summary table, 3 E2E scenarios, 15-case smoke set, OWASP Top 10 security suite, WCAG 2.1 AA accessibility tests, performance/load/stress/soak tests, documentation tests, shift-left checklist.

---

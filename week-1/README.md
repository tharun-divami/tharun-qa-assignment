# Week 1 — QA Hands-On Assessment
## B2B Vendor Invoice Management Portal

**Submitted by:** Tharun Pola  
**Training:** QA Excellence Upskilling Program  
**Date:** 2026-06-06  
**Source Material:** `01_QA_Excellence.pptx` (39 slides)

---

## Assignment Brief

The source training session (`01_QA_Excellence.pptx`) covered the full QA lifecycle across 8 phases — from requirements analysis through deployment verification — including test design techniques, testing levels, types, artifacts, and the QA mindset.

The hands-on assessment (Slides 31–32) presented a B2B client scenario:

> **"B2B Use Case: Vendor Invoice Management Portal"**
>
> Client Requirements:
> 1. Vendors can register and log in to the portal
> 2. Vendors can submit invoices against purchase orders
> 3. The AP team can view, approve, or reject invoices
> 4. Approved invoices are forwarded for payment processing
> 5. Both parties receive email notifications on status changes
> 6. The system generates monthly invoice activity reports
> 7. Only authorised users may access the system

The mission: identify ambiguous requirements, raise clarifying questions, define edge cases, design a test approach, and call out hidden risks.

---

## Deliverable Documents

Documents are numbered in the order they were produced, following the QA lifecycle phases taught in the session.

| # | File | Type | What It Contains |
|---|------|------|-----------------|
| — | [`01_QA_Excellence.pptx`](01_QA_Excellence.pptx) | Source | Training slide deck — 39 slides |
| 1 | [`02_prd.md`](02_prd.md) | PRD | Product Requirements Document |
| 2 | [`03_requirements_clarification.md`](03_requirements_clarification.md) | Clarification | Requirements ambiguity analysis & client questions |
| 3 | [`04_test_plan.md`](04_test_plan.md) | Test Plan | Comprehensive test plan with ~280 test cases |
| 4 | [`05_test_cases.md`](05_test_cases.md) | Test Cases | 78 extreme & edge case test cases in full detail |
| — | [`PROGRESS.md`](PROGRESS.md) | Tracker | Session-by-session activity log & decisions |

---

## Document Descriptions

### [`02_prd.md`](02_prd.md) — Product Requirements Document

Translates the 7 client requirements into a structured PRD following product management conventions.

**Contains:**
- 5 user roles (Vendor, AP User, AP Admin, Finance User, System Admin)
- 19 user stories (`US-VIMP-001` to `US-VIMP-019`) with role, source traceability, and rationale
- 11 business rules (`BR-VIMP-001` to `BR-VIMP-011`) governing system behaviour
- 70+ acceptance criteria (`AC-VIMP-001` to `AC-VIMP-009`) in **Given / When / Then** format with:
  - Exact test data values
  - Boundary Value Analysis markers (★)
  - Equivalence Partition markers (⊕)
  - Negative paths, security scenarios, concurrency edge cases
- Data contract (inbound and outbound system boundaries)
- 14 open questions (`OQ-01` to `OQ-14`) requiring client sign-off

---

### [`03_requirements_clarification.md`](03_requirements_clarification.md) — Requirements Clarification

Phase 1 QA artifact: identifies every ambiguity in the client requirements and raises targeted questions before any development begins.

**Contains:**
- Ambiguity analysis for each of the 7 client requirements
- 100+ clarifying questions anchored to specific PRD IDs (`US-VIMP-NNN`, `BR-VIMP-NNN`, `AC-VIMP-NNN`, `OQ-NNN`)
- Status-to-recipient notification matrix (template for client to complete)
- Full RBAC permission matrix (template for client to confirm)
- OQ consolidation table mapping all open questions to their sections

Every question follows the 5-question framework from the QA Excellence session (Slide 3):
*What does success look like? What happens when it fails? Who is affected? What data is involved? What are the boundaries?*

---

### [`04_test_plan.md`](04_test_plan.md) — Comprehensive Test Plan

The primary QA strategy document. Covers all 8 phases of the QA lifecycle and all test types from the training material.

**Contains:**
- Test objectives (all 6 from Slide 37)
- Risk matrix (12 risks with likelihood, impact, and mitigation)
- Test strategy: testing pyramid, shift-left activities, test design techniques
- All functional test types: System, Regression, Smoke, UAT, E2E, API (Slide 9)
- All non-functional test types: Performance, Load, Stress, Security (OWASP), Usability, Compatibility, Accessibility (WCAG 2.1 AA), Reliability, Documentation (Slide 9)
- Entry and exit criteria
- Test environment and test data strategy
- ~280 test cases across 11 modules (Auth, Invoice, Approval, Payment, Notifications, Reports, RBAC, API, DB, NFT, Integration)
- 3 decision tables (Approval workflow, Invoice submission, Notification triggers)
- Boundary value analysis summary table
- 3 end-to-end scenarios (golden paths)
- 15-case post-deploy smoke test set
- Regression strategy and defect management guidelines
- Test metrics (coverage, defect density, escape rate)
- Tools matrix

---

### [`05_test_cases.md`](05_test_cases.md) — Extreme & Edge Case Test Cases

Detailed test cases for scenarios that stress the system beyond normal usage. Each case includes exact preconditions, test data, numbered steps, and expected results across UI, API, and DB layers.

**Contains 78 test cases across 12 modules:**

| Module | Examples of Extreme Cases |
|--------|--------------------------|
| Authentication | Unicode in name fields, 254-char email, simultaneous same-email registration, token replay after logout, password reset invalidates all sessions |
| Invoice Submission | Zero-byte / corrupt / password-protected / embedded-JS PDFs, double-extension files, PO closed mid-submission, 50 concurrent submissions, JSON injection in line items |
| AP Approval | 10,000-invoice queue load, stale browser tab race condition, network drop mid-approval, concurrent approve+reject same invoice |
| Payment | Malformed HTTP 200, duplicate payment reference, 35-second timeout, concurrent bulk approvals |
| RBAC & Security | BOLA/IDOR sequential ID enumeration, JWT `alg:none` attack, HTTP verb tampering, path traversal, clickjacking, open redirect, host header injection |
| Concurrency | Approve+reject simultaneously, vendor deactivated mid-submission, PO closure during form fill |
| Data Extremes | Whitespace-only inputs, null API fields, 64KB strings, 100-level deep JSON, invalid UTF-8, 100 MB request body |
| Integration Failures | Unknown HTTP status codes, permanent email bounce (550), expired SAML assertion, out-of-order event delivery, duplicate payment callbacks |
| Non-Functional | 72-hour soak test, HSTS enforcement, Content Security Policy, keyboard-only form recovery (WCAG), PII in localStorage check |
| Business Logic | Invoice amount = exact PO value, over-invoicing by ₹0.01, second invoice against fully-invoiced PO, resubmit rejected with same invoice number |

---

## Reviewer's Reading Order

For a complete picture of the QA thinking, review in this order:

```
01_QA_Excellence.pptx          → Understand the training context
        ↓
02_prd.md                      → What the system must do (requirements → stories → ACs)
        ↓
03_requirements_clarification.md → What questions must be answered before testing
        ↓
04_test_plan.md                → How the system will be tested (strategy + standard cases)
        ↓
05_test_cases.md               → Where the system is likely to break (extreme cases)
```

---

## QA Lifecycle Coverage

This assessment covers all 8 phases from the training material:

| Phase | Covered In |
|-------|-----------|
| Phase 1 — Requirements Analysis | `02_prd.md`, `03_requirements_clarification.md` |
| Phase 2 — Test Planning | `04_test_plan.md` (Sections 2–9) |
| Phase 3 — Test Design | `04_test_plan.md` (Section 9 + Decision Tables), `05_test_cases.md` |
| Phase 4 — Shift-Left | `04_test_plan.md` (Section 4.2 + Section 18) |
| Phase 5 — Functional Testing | `04_test_plan.md` (Modules 1–8), `05_test_cases.md` |
| Phase 6 — Non-Functional Testing | `04_test_plan.md` (Module 10), `05_test_cases.md` (TC-EX-130–136) |
| Phase 7 — Integration Testing | `04_test_plan.md` (Module 11), `05_test_cases.md` (TC-EX-120–125) |
| Phase 8 — Deployment & Docs | `04_test_plan.md` (Section 13 — Smoke Test Set) |

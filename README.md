# QA Upskilling — Weekly Assignments

**Trainee:** Tharun Pola  
**Program:** QA Excellence Upskilling  
**Repository:** Weekly hands-on assessment submissions

---

## How This Repository Is Structured

Each week's assignment lives in its own folder. Every folder contains a `README.md` that explains that week's brief, lists all deliverables, and guides the reviewer through the documents in reading order.

```
tharun-qa-assignment/
|-- README.md                          (you are here - navigation index)
|-- week-1/
|   |-- README.md                      (week 1 brief + document guide)
|   |-- 01_QA_Excellence.pptx
|   |-- 02_prd.md
|   |-- 03_requirements_clarification.md
|   |-- 04_test_plan.md
|   |-- 05_test_cases.md
|   \-- PROGRESS.md
|-- week-2/                            (upcoming)
|-- week-3/                            (upcoming)
\-- ...
```

---

## Weekly Assignments

### [Week 1 — QA Fundamentals: Vendor Invoice Management Portal](week-1/README.md)

**Topic:** QA Excellence — Lifecycle, Artifacts, Testing Strategy  
**Source:** `01_QA_Excellence.pptx` (39 slides)  
**Use Case:** B2B Vendor Invoice Management Portal

A client presented 7 requirements for a vendor invoice portal. The task was to apply the full QA lifecycle — from requirements analysis through deployment — producing professional QA artifacts.

| Deliverable | File |
|-------------|------|
| Product Requirements Document | [02_prd.md](week-1/02_prd.md) |
| Requirements Clarification | [03_requirements_clarification.md](week-1/03_requirements_clarification.md) |
| Comprehensive Test Plan (~280 test cases) | [04_test_plan.md](week-1/04_test_plan.md) |
| Extreme & Edge Case Test Cases (78 cases) | [05_test_cases.md](week-1/05_test_cases.md) |

**Skills demonstrated:** Requirements analysis, risk-based testing, EP/BVA/Decision Tables, OWASP security testing, WCAG accessibility, performance/load testing, API & DB testing, shift-left QA, test design across all 8 QA lifecycle phases.

---

<!-- Add new weeks below as they are completed -->

---

## Conventions Followed Across All Weeks

| Convention | Detail |
|-----------|--------|
| **Folder naming** | `week-N/` — one folder per assignment week |
| **File numbering** | Files prefixed `01_`, `02_`, etc. — reflects the order artifacts were produced |
| **IDs** | All requirements, stories, rules, and test cases carry traceable IDs (e.g., `US-VIMP-001`, `TC-EX-042`) |
| **Test case format** | Precondition → Test Data → Steps → Expected Result (UI + API + DB) |
| **Activity log** | Each week contains a `PROGRESS.md` recording every prompt, decision, and change made during the session |
| **README per week** | Every week folder has its own `README.md` with the assignment brief and reviewer reading order |

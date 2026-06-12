# Test Plan
## Vendor Invoice Management Portal (VIMP)

**Document Type:** Test Plan — Phase 2 of the QA Lifecycle  
**Module:** VIMP — Vendor Invoice Management Portal  
**Prepared by:** QA Engineer  
**Date:** 2026-06-06  
**Version:** 1.0  
**PRD Reference:** `02_prd.md` (US-VIMP-001–019, BR-VIMP-001–011, AC-VIMP-001–009)  
**Clarification Reference:** `03_requirements_clarification.md`

---

## 1. Objectives

Testing the Vendor Invoice Management Portal serves six core purposes, derived from the QA Excellence framework:

| # | Objective | What It Means for VIMP |
|---|-----------|------------------------|
| 1 | Build Confidence | Demonstrate the portal correctly handles all invoice lifecycle states |
| 2 | Detect Defects Early | Identify issues in design and API contracts before UI is built (shift-left) |
| 3 | Verify Requirements | Confirm every US-VIMP-NNN is implemented per its AC-VIMP-NNN |
| 4 | Validate User Needs | Ensure vendors can submit invoices and AP team can approve without friction |
| 5 | Measure Quality | Track defect density, test coverage %, and escape rate per sprint |
| 6 | Reduce Risk | Surface payment-forwarding errors, RBAC gaps, and data integrity failures before go-live |

---

## 2. Scope

### 2.1 In Scope

| Area | Test Layer | Notes |
|------|-----------|-------|
| Vendor Registration & Login | UI, API, DB | All auth flows including edge cases |
| Invoice Submission | UI, API, DB | File upload, PO matching, duplicate detection |
| AP Approval & Rejection Workflow | UI, API, DB | State machine, audit trail, segregation of duties |
| Payment Forwarding | API, Integration | Auto-trigger, retry, failure handling |
| Email Notifications | Integration | Per-status triggers, content, delivery SLA |
| Monthly Reports | UI, API | Generation, download, access control |
| Role-Based Access Control | UI, API | All 5 roles × all features |
| Non-Functional Testing | Performance, Security, Usability | Load, OWASP, WCAG |
| Integration Testing | External Systems | Payment gateway, email service, SSO/auth provider |
| API Testing | API contracts | Request/response, error codes, auth tokens, data integrity |
| Database Testing | DB | Data correctness, constraints, no orphaned records |
| Deployment Verification | Config, Smoke | Post-deploy smoke test, rollback readiness |

### 2.2 Out of Scope

- Unit tests (developer responsibility; monitored via coverage report)
- Payment execution (bank transfers, bank APIs) — VIMP only forwards to payment system
- Vendor onboarding (pre-portal procurement workflow)
- Mobile application testing
- Multi-language / i18n
- Specific ERP integration internals (tested at the integration boundary only)

---

## 3. Risk Analysis

A test plan without risk analysis is just a wish list. The following risks drive test prioritisation.

### 3.1 Risk Matrix

| Risk ID | Risk | Likelihood | Impact | Priority | Mitigation |
|---------|------|-----------|--------|----------|-----------|
| R-01 | Duplicate invoice processed and paid twice | Medium | Critical | P1 | Idempotency tests, DB constraint tests |
| R-02 | Unauthorised user approves or views invoices | Medium | Critical | P1 | Full RBAC matrix tests, API auth tests |
| R-03 | Payment forwarded to wrong bank account | Low | Critical | P1 | Data integrity tests, bank detail validation |
| R-04 | Invoice status stuck in transition (partial state) | Medium | High | P1 | Concurrency tests, network interruption tests |
| R-05 | Vendor sees another vendor's invoices | Low | Critical | P1 | Row-level security tests (BR-VIMP-007) |
| R-06 | Notification not sent on status change | Medium | High | P2 | Notification trigger tests, SLA verification |
| R-07 | Session not invalidated on account deactivation | Low | High | P1 | BR-VIMP-010 tests (AC-VIMP-009i) |
| R-08 | SQL injection / XSS via input fields | Medium | Critical | P1 | Security tests — all input surfaces |
| R-09 | Performance degradation under load | Medium | High | P2 | Load test: 1000 concurrent users |
| R-10 | Monthly report covers wrong date range | Low | Medium | P2 | Date boundary tests (Dec 31 / Jan 1) |
| R-11 | File upload bypass (malicious file stored) | Low | Critical | P1 | Format + MIME type validation tests |
| R-12 | Approval without valid PO reference | Low | High | P1 | BR-VIMP-001 enforcement tests |

### 3.2 Testing Priority by Risk

- **P1 (Test First, Block Release):** R-01, R-02, R-03, R-04, R-05, R-07, R-08, R-11, R-12
- **P2 (Test Before Release):** R-06, R-09, R-10
- **P3 (Test in Regression):** All remaining functional scenarios

---

## 4. Test Strategy

### 4.1 The Testing Pyramid

Following the QA Excellence framework (Slide 21), test investment is distributed as:

```
         /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
        /  E2E / UI Tests   \    ← Few: 15–20 scenarios (golden paths + critical edge cases)
       /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
      /   API Integration     \  ← Some: 80–100 test cases (contracts, auth, data flow)
     /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
    /   Unit Tests (Dev-owned)  \ ← Many: >200 (developer-owned, CI-enforced)
   /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
```

QA owns the API and E2E layers. Unit test coverage is monitored, not authored by QA.

### 4.2 Shift-Left Activities (Phase 4)

QA involvement during development:

| Activity | When | QA Action |
|----------|------|-----------|
| Story review | Before sprint | Review US-VIMP-NNN for ambiguity; raise if AC is unclear |
| API contract review | Before backend coding | Validate request/response shapes; flag missing error codes |
| Design review | Wireframe stage | Identify UX flows that break BR-VIMP-NNN rules |
| Three Amigos | Per story | Dev + QA + PO align on AC before coding begins |
| Code review participation | PR stage | QA flags logic that would fail AC-VIMP-NNN |
| Acceptance testing | Immediately after dev | QA runs AC-VIMP-NNN before story is moved to Done |

### 4.3 Test Design Techniques Used

| Technique | Applied To |
|-----------|-----------|
| **Equivalence Partitioning (EP)** | Email format, file format, user roles, invoice status |
| **Boundary Value Analysis (BVA)** | File size (4.9 MB / 5.0 MB / 5.1 MB), date range (±1 day at boundary), character counts, amount = 0 |
| **Decision Table Testing** | Login workflow, invoice approval routing, notification triggers |
| **State Transition Testing** | Invoice status machine (SUBMITTED → APPROVED/REJECTED → PAYMENT_INITIATED) |
| **Exploratory Testing** | Unscripted sessions focused on integration points and edge case discovery |
| **Error Guessing** | Based on common defect patterns: concurrent updates, session replay, empty states |

---

## 5. Test Types Coverage

### 5.1 Functional Testing

| Type | Coverage in VIMP |
|------|-----------------|
| **System Testing** | Full end-to-end portal workflow; all modules integrated |
| **Regression Testing** | Full suite re-run on every release; smoke subset on hotfixes |
| **Smoke / Sanity Testing** | 15-case subset run post-deploy (see Section 14) |
| **User Acceptance Testing (UAT)** | Client team verifies portal against their actual AP process |
| **End-to-End Testing** | 3 golden-path scenarios: Vendor Submit → AP Approve → Payment Forwarded → Vendor Notified |
| **API Testing** | All endpoints: contracts, auth tokens, error codes, data integrity |

### 5.2 Non-Functional Testing

| Type | Approach | Tool |
|------|---------|------|
| **Performance** | Response time < 2s for all portal pages under normal load (50 concurrent users) | JMeter / k6 |
| **Load** | 1000 concurrent users for 30 minutes; no error rate > 0.1% | JMeter |
| **Stress** | Ramp to 200% of expected load; identify break point and graceful degradation behaviour | JMeter |
| **Security** | OWASP Top 10: SQLi, XSS, IDOR, auth bypass, sensitive data exposure, CSRF | OWASP ZAP / Burp Suite |
| **Usability** | Task completion rate for core vendor workflow without guidance; error message clarity | Manual + user sessions |
| **Compatibility** | Chrome (latest), Firefox (latest), Safari (latest), Edge (latest); Windows + macOS | BrowserStack |
| **Accessibility** | WCAG 2.1 Level AA: keyboard navigation, screen reader compatibility, colour contrast | axe DevTools |
| **Reliability** | Portal runs 72 hours without restart under moderate load; no memory leaks | JMeter soak test |
| **Documentation** | User guide matches actual UI; error messages are user-friendly and accurate | Manual review |

---

## 6. Entry and Exit Criteria

### 6.1 Entry Criteria (Before Testing Begins)

- [ ] PRD (`02_prd.md`) has at least one client approval signature
- [ ] All P1 open questions (OQ-01, OQ-03–08, OQ-11, OQ-13, OQ-14) answered
- [ ] Test environment provisioned and accessible
- [ ] Test data (vendor accounts, POs, AP users) seeded
- [ ] API documentation available (endpoints, auth, payloads)
- [ ] Build deployed to staging; smoke test passes

### 6.2 Exit Criteria (Before Release Approval)

- [ ] 100% of P1 test cases executed
- [ ] 0 open P1 (Critical) defects
- [ ] 0 open P2 (High) defects blocking core workflows
- [ ] Test coverage ≥ 90% of US-VIMP-NNN stories
- [ ] All AC-VIMP-NNN acceptance criteria verified as PASS
- [ ] RTM shows no requirement without a linked, executed test case
- [ ] Performance baseline met: p95 response time < 2s under 50 concurrent users
- [ ] Security scan shows no Critical or High OWASP findings unresolved
- [ ] UAT sign-off obtained from client AP team

---

## 7. Test Environment

| Environment | Purpose | Data |
|-------------|---------|------|
| **DEV** | Developer unit + integration tests | Synthetic, auto-reset |
| **STAGING** | QA test execution, regression, performance | Masked production-like data |
| **UAT** | Client acceptance testing | Client-approved test data |
| **PRODUCTION** | Post-deploy smoke tests only | Real data — read-only smoke |

**Staging Configuration Requirements:**
- Payment system: mock/stub responding to all test payloads
- Email service: test mailbox (no real emails sent); capture service (e.g., Mailhog)
- Auth/SSO: test IdP or local auth mode
- DB: PostgreSQL instance with schema identical to production
- Storage: S3-compatible bucket for attachment uploads

---

## 8. Test Data Strategy

Good test data is as important as good test cases. The following categories must be prepared before test execution.

| Category | Examples | Coverage |
|----------|---------|---------|
| **Vendor Accounts** | Active, Pending Verification, Deactivated, Locked | AC-VIMP-002b, 002d, 002e |
| **AP Users** | AP User, AP Admin, Finance User, System Admin | AC-VIMP-009 |
| **Purchase Orders** | Open PO (vendor A), Closed PO, PO assigned to vendor B, PO with partial remaining value | AC-VIMP-003b–003c |
| **Invoices (Staged)** | SUBMITTED, APPROVED, REJECTED, PAYMENT_INITIATED, PAYMENT_COMPLETED | State transition tests |
| **File Attachments** | PDF 4.9 MB, PDF 5.0 MB, PDF 5.1 MB, JPEG 1 MB, XLSX 2 MB, .exe (malicious), zero-byte file | AC-VIMP-003e–003h |
| **Boundary Dates** | Today, today−180, today−181, today+1, Dec 31, Jan 1 | AC-VIMP-003k–003m, 008b |
| **Invoice Amounts** | 0.00, −1000, 1.00, 50000.00, 9999999.99 | AC-VIMP-003i–003j |
| **Rejection Reasons** | Empty, 1 char, 500 chars, 501 chars, XSS payload | AC-VIMP-005b–005f |
| **Malicious Inputs** | SQLi strings, XSS payloads, CSRF tokens, tampered JWTs | AC-VIMP-001j, 002j, 009f–009h |

---

## 9. Test Scenarios & Test Cases

Test cases follow the qualities defined in QA Excellence (Slide 16): clear preconditions, single expected result, atomic, reproducible, specific test data, traced to requirement, covering negative paths.

**Column Key:**
- **Ref:** PRD story / AC reference
- **Type:** P = Positive | N = Negative | B = Boundary | S = Security | E = Edge Case
- **Layer:** UI = browser | API = HTTP endpoint | DB = database assertion | INT = integration
- **Pri:** P1 = Critical | P2 = High | P3 = Medium

---

### Module 1 — Authentication & Registration

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-001 | Valid vendor registration | US-VIMP-001 / AC-VIMP-001a | P | UI + API | P1 | Email "v_new@testco.com" not registered | 1. Open /register 2. Fill all fields 3. Submit | Company: TestCo, Email: v_new@testco.com, Pwd: Secure@123 | 201 response, account PENDING_VERIFICATION, verification email sent, no auto-login |
| TC-VIMP-002 | Email verification within 24h | AC-VIMP-001b | P | UI | P1 | Account in PENDING state, link received <1h ago | Click verification link in email | Valid link | Account status → ACTIVE, redirected to login, link invalidated |
| TC-VIMP-003 | Expired verification link (25h) | AC-VIMP-001c | B | UI | P2 | Account in PENDING state, link 25h old | Click verification link | 25h-old link | "Link has expired" error, account stays PENDING, resend option shown |
| TC-VIMP-004 | Duplicate email registration | AC-VIMP-001d | N | UI + API | P1 | "existing@co.com" already registered | Submit registration with email "existing@co.com" | existing@co.com | Error: "Account with this email already exists." No new record in DB |
| TC-VIMP-005 | Missing mandatory fields | AC-VIMP-001e | N | UI | P2 | Registration page open | Submit with Company Name = blank (repeat for each field) | Blank fields | Field highlighted red, "This field is required", form not submitted |
| TC-VIMP-006 | Invalid email formats | AC-VIMP-001f | N | UI | P2 | Registration page open | Submit with (a)userwithoutat.com (b)@nodomain.com (c)user@ | Invalid formats | "Please enter a valid email address" for each |
| TC-VIMP-007 | Weak password — too short | AC-VIMP-001g | B | UI | P2 | Registration page open | Enter password = "Abc1!" (5 chars) | 5-char password | "Password must be at least 8 characters" error |
| TC-VIMP-008 | Password at minimum length | AC-VIMP-001g | B | UI | P2 | Registration page open | Enter password = "Abcde1!!" (8 chars) | 8-char password | Password accepted |
| TC-VIMP-009 | Password confirmation mismatch | AC-VIMP-001h | N | UI | P2 | Registration page open | Pwd: Secure@123, Confirm: Secure@124 | Mismatched | "Passwords do not match" on confirm field |
| TC-VIMP-010 | XSS in company name field | AC-VIMP-001i | S | UI + DB | P1 | Registration page open | Company Name = `<script>alert('xss')</script>` | XSS payload | Stored as HTML-encoded text; no script executes |
| TC-VIMP-011 | SQL injection in email field | AC-VIMP-001j | S | API | P1 | Registration page open | Email = `' OR '1'='1` | SQLi string | Email format validation error; no SQL error; no unintended DB change |
| TC-VIMP-012 | Resend verification email rate limit | AC-VIMP-001k | B | UI | P2 | Account in PENDING_VERIFICATION | Request resend → immediately request again within 60s | Rapid resend | First resend OK; second within 60s returns: "Please wait before requesting again" |
| TC-VIMP-013 | Vendor login — valid credentials | AC-VIMP-002a | P | UI + API | P1 | Active account: vendor@testco.com / Secure@123 | Enter valid email + password, click Login | vendor@testco.com, Secure@123 | 200 + session token, redirect to /dashboard, token in HttpOnly cookie |
| TC-VIMP-014 | Email case-insensitive login | AC-VIMP-002b | E | UI | P2 | Active account: vendor@testco.com | Login with "VENDOR@TESTCO.COM" | Mixed-case email | Login succeeds |
| TC-VIMP-015 | Wrong password (4 attempts) | AC-VIMP-002c | N | UI | P1 | Active account, 0 prior failures | Enter wrong password 4× | WrongPass1! | Each returns "Invalid email or password"; account stays unlocked after 4th |
| TC-VIMP-016 | Account lockout on 5th failure | AC-VIMP-002d | B | UI + DB | P1 | Account with 4 existing failed attempts | Enter wrong password (5th attempt) | WrongPass1! | Account LOCKED, lockout message shown, unlock email sent, DB: locked_at set |
| TC-VIMP-017 | Login with PENDING_VERIFICATION account | AC-VIMP-002e | N | UI | P1 | Account status = PENDING_VERIFICATION | Enter valid credentials | Correct creds | "Please verify your email before logging in" — access denied |
| TC-VIMP-018 | Login with DEACTIVATED account | AC-VIMP-002f | N | UI | P1 | Account status = DEACTIVATED | Enter valid credentials | Correct creds | "Account deactivated. Contact support." — access denied |
| TC-VIMP-019 | Session timeout | AC-VIMP-002g | B | UI | P1 | Logged in, idle for [timeout+1min] | Attempt any navigation after timeout | Idle beyond timeout | Redirect to /login: "Session expired" |
| TC-VIMP-020 | Concurrent session invalidation | AC-VIMP-002h | E | UI | P2 | Logged in on Browser A | Login with same account on Browser B | Same account | Browser A's next request → "Logged out — accessed from another location" |
| TC-VIMP-021 | Empty credentials form submission | AC-VIMP-002i | N | UI | P2 | Login page open | Submit with both fields empty | Blank | "This field is required" on both; no HTTP request made |
| TC-VIMP-022 | SQL injection at login | AC-VIMP-002j | S | API | P1 | Login endpoint accessible | POST login with email=`admin'--`, password=anything | SQLi | "Invalid email or password"; no SQL error; no unauth session |

---

### Module 2 — Invoice Submission

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-030 | Valid invoice submission | AC-VIMP-003a | P | UI + API + DB | P1 | Active vendor logged in, PO-2024-1001 open and assigned | Complete invoice form with valid fields; upload 2MB PDF; Submit | INV-2024-0501, PO-2024-1001, 50000 INR, 3 line items | Status=SUBMITTED, confirmation email, appears in AP queue, DB record correct |
| TC-VIMP-031 | PO not found | AC-VIMP-003b | N | UI | P1 | Active vendor | Submit invoice with PO = "PO-INVALID-9999" | Invalid PO | "PO not found or not assigned to your account" |
| TC-VIMP-032 | PO belongs to different vendor | AC-VIMP-003c | S | API | P1 | Vendor A logged in; PO-2024-2000 belongs to Vendor B | Submit invoice referencing PO-2024-2000 | Vendor B's PO | Same error as TC-031 (no disclosure that PO exists) |
| TC-VIMP-033 | Duplicate invoice detection | AC-VIMP-003d | N | API | P1 | INV-2024-0501 already SUBMITTED against PO-2024-1001 | Submit same INV-2024-0501 against PO-2024-1001 again | Duplicate combo | "Duplicate invoice detected." No DB record created |
| TC-VIMP-034 | File upload: 4.9 MB PDF (below limit) | AC-VIMP-003e | B | UI | P2 | Invoice form open | Attach PDF of exactly 4.9 MB | 4.9 MB PDF | File accepted, no error |
| TC-VIMP-035 | File upload: exactly 5.0 MB (at limit) | AC-VIMP-003f | B | UI | P2 | Invoice form open | Attach PDF of exactly 5.0 MB | 5.0 MB PDF | File accepted (boundary is inclusive) |
| TC-VIMP-036 | File upload: 5.1 MB (over limit) | AC-VIMP-003g | B | UI | P1 | Invoice form open | Attach PDF of 5.1 MB | 5.1 MB PDF | "File exceeds 5 MB limit" — file rejected, not stored |
| TC-VIMP-037 | Unsupported file type (.exe) | AC-VIMP-003h | S | UI | P1 | Invoice form open | Select malware.exe for upload | .exe file | Client-side rejection: "Unsupported file format" — file not sent to server |
| TC-VIMP-038 | MIME type spoofing (rename .exe to .pdf) | — | S | API | P1 | Invoice form open | Upload file renamed malware.pdf (actually .exe) | Renamed .exe | Server-side MIME validation rejects: "File content does not match declared format" |
| TC-VIMP-039 | Zero invoice amount | AC-VIMP-003i | B | UI | P1 | Invoice form open | Enter Total Amount = 0.00 | 0.00 | "Invoice amount must be greater than zero" |
| TC-VIMP-040 | Negative invoice amount | AC-VIMP-003j | B | API | P1 | Logged in | POST invoice with amount = -1000.00 | -1000.00 | HTTP 422: "Amount must be a positive value" |
| TC-VIMP-041 | Future-dated invoice | AC-VIMP-003k | B | UI | P2 | Invoice form open | Set date = tomorrow | Date+1 | "Invoice date cannot be in the future" |
| TC-VIMP-042 | Invoice date at 180-day boundary (valid) | AC-VIMP-003m | B | UI | P2 | Invoice form open | Set date = today − 180 | today-180 | Accepted (boundary inclusive) |
| TC-VIMP-043 | Invoice date at 181-day boundary (invalid) | AC-VIMP-003l | B | UI | P2 | Invoice form open | Set date = today − 181 | today-181 | "Invoice date is too old. Max 180 days." |
| TC-VIMP-044 | Invoice with 0 line items | AC-VIMP-003n | B | UI | P1 | Invoice header filled, no line items added | Submit | No line items | "At least one line item is required" |
| TC-VIMP-045 | Amount mismatches line items sum | AC-VIMP-003o | N | UI | P2 | Line items total = 48000, header amount = 50000 | Submit | Mismatch | "Total amount does not match sum of line items" |
| TC-VIMP-046 | Deactivated vendor submits invoice (API) | AC-VIMP-003p | S | API | P1 | Vendor DEACTIVATED, token still in session | POST /api/invoices with valid payload | Deactivated account + token | HTTP 403: "Account not authorised to submit invoices" |
| TC-VIMP-047 | Double-submit prevention (back button) | — | E | UI | P2 | Invoice submitted successfully | Click browser Back → click Submit again | Repeat submit | Duplicate blocked; "This invoice has already been submitted" or no-op |
| TC-VIMP-048 | Invoice submission by AP User (wrong role) | — | S | API | P1 | AP User logged in | POST /api/invoices | AP User token | HTTP 403: "Only vendor accounts can submit invoices" |

---

### Module 3 — AP Approval Workflow

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-060 | AP User views pending invoices | US-VIMP-009 | P | UI + API | P1 | 5 invoices in SUBMITTED status | Login as AP User → navigate to Approval Queue | AP credentials | Queue shows 5 invoices; each shows vendor, amount, date, PO |
| TC-VIMP-061 | Approve a submitted invoice | AC-VIMP-004a | P | UI + API + DB | P1 | Invoice INV-0501 in SUBMITTED status | View invoice → Click Approve → Confirm | AP User: ap_user1@co.com | Status=APPROVED, audit trail entry (actor + timestamp), payment trigger fired, vendor email sent |
| TC-VIMP-062 | Approve already-approved invoice | AC-VIMP-004b | N | API | P1 | Invoice INV-0501 in APPROVED status | POST /api/invoices/INV-0501/approve | Repeat call | HTTP 409: "Invoice already approved." No duplicate audit entry. No duplicate payment. |
| TC-VIMP-063 | Approve a REJECTED invoice | AC-VIMP-004c | N | UI + API | P1 | Invoice in REJECTED status | Attempt to approve it | REJECTED invoice | UI: Approve button disabled. API: HTTP 409. |
| TC-VIMP-064 | Segregation of duties enforcement | AC-VIMP-004d | S | UI + API | P1 | ap_user2 is also contact for invoice (test scenario) | ap_user2 attempts to approve invoice they submitted | Same user | UI: Approve button disabled. API: HTTP 403. |
| TC-VIMP-065 | Concurrent approval race condition | AC-VIMP-004e | E | API | P1 | Two AP Users see same invoice in SUBMITTED | ap_user1 and ap_user2 simultaneously call approve | Same invoice, 2 tokens | Exactly one approval succeeds; second gets "already actioned"; single audit entry |
| TC-VIMP-066 | AP User deactivated mid-session | AC-VIMP-004f | S | API | P1 | AP User logged in; admin deactivates account | AP User approves invoice after deactivation | Mid-session deactivation | HTTP 401; approval not persisted |
| TC-VIMP-067 | Approval with optional comment | AC-VIMP-004g | P | UI + DB | P2 | Invoice in SUBMITTED status | Approve + add comment "Verified delivery receipt" | With comment | Comment stored in audit trail; visible to AP Admin; NOT visible to vendor |
| TC-VIMP-068 | Reject with valid reason | AC-VIMP-005a | P | UI + API + DB | P1 | Invoice INV-0510 in SUBMITTED status | Click Reject → enter reason → Confirm | Reason: "PO expired, resubmit with new PO" | Status=REJECTED, reason stored, audit entry, vendor email with reason and deeplink |
| TC-VIMP-069 | Reject with empty reason | AC-VIMP-005b | N | UI | P1 | Invoice in SUBMITTED status | Click Reject → leave reason blank → Confirm | Empty reason | "A rejection reason is required." Modal stays open. No status change. |
| TC-VIMP-070 | Reject with 1-character reason (min) | AC-VIMP-005c | B | UI | P2 | Invoice in SUBMITTED status | Enter reason = "X" → Confirm | 1 char | Rejection accepted |
| TC-VIMP-071 | Reject with 500-character reason (max) | AC-VIMP-005d | B | UI | P2 | Invoice in SUBMITTED status | Enter 500-char reason | 500 chars exactly | Rejection accepted, full text stored |
| TC-VIMP-072 | Reject with 501-character reason (over max) | AC-VIMP-005e | B | UI | P2 | Invoice in SUBMITTED status | Enter 501-char reason | 501 chars | 501st char blocked; counter shows 500/500 |
| TC-VIMP-073 | XSS in rejection reason | AC-VIMP-005f | S | UI + DB | P1 | Invoice in SUBMITTED status | Rejection reason = `<img src=x onerror=alert(1)>` | XSS payload | Stored as escaped text; no script fires on vendor view |
| TC-VIMP-074 | Reject already-rejected invoice | AC-VIMP-005g | N | API | P2 | Invoice in REJECTED status | Attempt to reject again | Already rejected | HTTP 409: "Invoice already rejected." No state change |
| TC-VIMP-075 | Reject a PAYMENT_INITIATED invoice | AC-VIMP-005h | N | UI + API | P1 | Invoice in PAYMENT_INITIATED status | Attempt to reject | In-payment invoice | Reject button disabled in UI; API: HTTP 409 |

---

### Module 4 — Payment Forwarding

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-090 | Auto-forwarding on approval | AC-VIMP-006a | P | API + INT | P1 | Invoice in SUBMITTED, payment system mock running | Approve invoice | INV-0501 | Within 60s: payment API called, status=PAYMENT_INITIATED, pay ref stored |
| TC-VIMP-091 | Payment system unavailable — retry queue | AC-VIMP-006b | N | API + INT | P1 | Payment system mock returns 503 | Approve invoice | INV-0521, mock=503 | Status=PENDING_PAYMENT_FORWARDING; retries at T+5, T+15, T+60 min; after 3 failures: AP Admin email alert |
| TC-VIMP-092 | Payment system returns success | AC-VIMP-006c | P | API + INT | P1 | Payment system mock returns 200 + pay ref | Approve invoice | INV-0522, mock returns {PAY-88001} | Status=PAYMENT_INITIATED; PAY-88001 stored; visible to AP Admin and Finance User |
| TC-VIMP-093 | Duplicate forwarding prevention | AC-VIMP-006d | E | API | P1 | Invoice already in PAYMENT_INITIATED | Trigger forwarding event again | Repeat event | Second call blocked; no duplicate payment API call; idempotency key enforced |
| TC-VIMP-094 | Missing vendor bank details | AC-VIMP-006e | N | API + DB | P1 | Vendor has no bank details in profile | Approve invoice | Vendor without bank data | Forwarding blocked; status=PENDING_PAYMENT_FORWARDING; AP Admin alert sent |
| TC-VIMP-095 | Finance User views payment status | US-VIMP-014 | P | UI | P2 | Invoice in PAYMENT_INITIATED | Login as Finance User → find invoice | Finance credentials | Payment ref, initiated date visible; no approve/reject buttons available |
| TC-VIMP-096 | Vendor views payment status | — | P/N | UI | P2 | Invoice in PAYMENT_INITIATED | Login as Vendor → view invoice | Vendor credentials | Vendor sees status PAYMENT_INITIATED (or PAYMENT_COMPLETED); no internal payment refs leaked |

---

### Module 5 — Email Notifications

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-110 | Notification: Invoice submitted → AP team | AC-VIMP-007a | P | INT | P2 | Email capture service running | Vendor submits invoice | INV-0601 | Email to AP team inbox within 5 min: subject contains "INV-0601", includes vendor name, PO, amount, portal link |
| TC-VIMP-111 | Notification: Invoice approved → vendor | AC-VIMP-007b | P | INT | P1 | Invoice approved | AP User approves invoice | INV-0601 | Email to vendor within 5 min: subject "Approved: INV-0601", correct invoice number and amount |
| TC-VIMP-112 | Notification: Invoice rejected → vendor with reason | AC-VIMP-007c | P | INT | P1 | Invoice rejected with reason | AP User rejects with "PO expired" | INV-0602, reason: "PO expired" | Email to vendor: reason text present, deeplink to INV-0602 |
| TC-VIMP-113 | Notification: Payment initiated → vendor | AC-VIMP-007d | P | INT | P2 | Invoice forwarded to payment system | Payment forwarding succeeds | INV-0601 | Email to vendor: "Payment initiated for INV-0601" with payment reference if available |
| TC-VIMP-114 | No duplicate notification | AC-VIMP-007f | E | INT | P2 | Invoice status changes once | Single status transition | INV-0601 | Exactly 1 email per event. Check email capture: no duplicates. |
| TC-VIMP-115 | Invalid vendor email — graceful failure | AC-VIMP-007e | N | INT | P2 | Vendor email = "bad_email@" (data integrity issue) | Status change triggers notification | Malformed email | Email dispatch fails; error logged; AP Admin alerted; no system crash |
| TC-VIMP-116 | Notification deeplink is correct | AC-VIMP-007g | P | UI | P2 | Rejection email sent | Click portal link from rejection email | Rejection email link | Link navigates to invoice INV-0602 after login; not to generic dashboard; no 404 |

---

### Module 6 — Reporting

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-130 | Monthly report generated correctly | AC-VIMP-008a | P | API + DB | P2 | May 2026: 120 submitted, 95 approved, 18 rejected | Run report job for May 2026 | May 2026 data | Report: submitted=120, approved=95, rejected=18, pending=7, total_value=₹12,450,000 |
| TC-VIMP-131 | Report date boundary: Dec 31 invoice in Dec report | AC-VIMP-008b | B | DB | P2 | Invoice submitted 2026-12-31 23:59:59 | Generate December report | Dec 31 23:59:59 invoice | Invoice appears in December report, not January |
| TC-VIMP-132 | Report date boundary: Jan 1 invoice in Jan report | AC-VIMP-008b | B | DB | P2 | Invoice submitted 2027-01-01 00:00:00 | Generate January report | Jan 1 00:00:00 invoice | Invoice appears in January report, not December |
| TC-VIMP-133 | Report for month with zero invoices | AC-VIMP-008c | E | API | P2 | No activity in Feb 2026 | Generate February 2026 report | Empty month | Report generated with all counts = 0; no error; downloadable |
| TC-VIMP-134 | Vendor cannot access reports | AC-VIMP-008d | S | UI + API | P1 | Vendor logged in | Navigate to /reports or GET /api/reports | Vendor token | HTTP 403; no report data returned |
| TC-VIMP-135 | PDF download validity | AC-VIMP-008e | P | UI | P2 | May 2026 report available | AP Admin downloads as PDF | Valid month | Valid PDF file, readable, contains correct metrics |
| TC-VIMP-136 | XLSX download validity | AC-VIMP-008f | P | UI | P2 | May 2026 report available | AP Admin downloads as XLSX | Valid month | Valid Excel file, numeric fields are numbers (not text), data matches portal view |

---

### Module 7 — Role-Based Access Control (RBAC)

| TC-ID | Title | Ref | Type | Layer | Pri | Precondition | Steps | Test Data | Expected Result |
|-------|-------|-----|------|-------|-----|-------------|-------|-----------|----------------|
| TC-VIMP-150 | Vendor blocked from AP approval queue | AC-VIMP-009a | S | API | P1 | Vendor token | GET /api/invoices/approval-queue | Vendor JWT | HTTP 403, no data |
| TC-VIMP-151 | AP User blocked from admin panel | AC-VIMP-009b | S | API | P1 | AP User token | GET /admin/users | AP User JWT | HTTP 403, no data |
| TC-VIMP-152 | Finance User cannot approve | AC-VIMP-009c | S | API | P1 | Finance User token | POST /api/invoices/INV-0501/approve | Finance JWT | HTTP 403; invoice status unchanged |
| TC-VIMP-153 | Vendor cannot view other vendor's invoice | AC-VIMP-009d | S | API | P1 | Vendor A token; INV-0700 = Vendor B's | GET /api/invoices/INV-0700 | Vendor A JWT | HTTP 403 or 404 (no info disclosure) |
| TC-VIMP-154 | Unauthenticated request to protected endpoint | AC-VIMP-009e | S | API | P1 | No token | GET /api/invoices | No auth header | HTTP 401; redirect to /login |
| TC-VIMP-155 | Tampered JWT (role escalation) | AC-VIMP-009f | S | API | P1 | Valid vendor JWT | Modify payload: role → AP_ADMIN; send request | Tampered token | HTTP 401: "Invalid authentication token" |
| TC-VIMP-156 | Expired JWT | AC-VIMP-009g | S | API | P1 | JWT expired 1 min ago | Use expired token in request | Expired token | HTTP 401; redirect to /login |
| TC-VIMP-157 | CSRF on approval endpoint | AC-VIMP-009h | S | API | P1 | Cross-origin request crafted | POST approval from external origin | No CSRF token | HTTP 403; approval not executed |
| TC-VIMP-158 | Account deactivated → session invalidated | AC-VIMP-009i | S | API | P1 | AP User logged in; admin deactivates account | AP User's next request after deactivation | Mid-session | HTTP 401; any in-flight action fails |
| TC-VIMP-159 | Role downgrade takes effect immediately | AC-VIMP-009j | S | API | P2 | AP Admin logged in; admin changes role to AP User | AP Admin's next request to admin-only endpoint | Role downgraded | HTTP 403 on admin endpoints |
| TC-VIMP-160 | Direct URL manipulation by vendor | — | S | UI | P1 | Vendor logged in | Manually type /admin/invoices/all in browser | Vendor session | 403 page shown; no AP data rendered |
| TC-VIMP-161 | IDOR: Vendor manipulates invoice ID in URL | — | S | API | P1 | Vendor A, knows Vendor B's invoice ID | GET /api/invoices/INV-B-0001 | Vendor A token | HTTP 403 or 404 — no data from Vendor B |

---

### Module 8 — API Layer Tests

API tests verify the contract between client and server, independent of the UI.

| TC-ID | Title | Type | Pri | Endpoint | Test Input | Expected Response |
|-------|-------|------|-----|----------|-----------|------------------|
| TC-VIMP-180 | Login — correct payload | P | P1 | POST /api/auth/login | `{email, password}` valid | 200, JWT token, user_id, role |
| TC-VIMP-181 | Login — missing password field | N | P2 | POST /api/auth/login | `{email}` only | 422: "password is required" |
| TC-VIMP-182 | Login — extra unknown fields | E | P3 | POST /api/auth/login | `{email, password, hack: "x"}` | 200 (unknown fields ignored) or 422 |
| TC-VIMP-183 | Invoice create — valid payload | P | P1 | POST /api/invoices | Full valid invoice payload | 201, invoice_id returned |
| TC-VIMP-184 | Invoice create — missing required field | N | P1 | POST /api/invoices | Payload missing `po_number` | 422: "po_number is required" |
| TC-VIMP-185 | Invoice create — invalid amount type | N | P1 | POST /api/invoices | `amount: "fifty"` (string) | 422: "amount must be a number" |
| TC-VIMP-186 | Approval — invoice not found | N | P1 | POST /api/invoices/NOTEXIST/approve | Non-existent invoice ID | 404: "Invoice not found" |
| TC-VIMP-187 | Approval — wrong Content-Type | N | P2 | POST /api/invoices/INV-0501/approve | Content-Type: text/plain | 415: "Unsupported Media Type" |
| TC-VIMP-188 | Rate limiting on login endpoint | S | P1 | POST /api/auth/login | 20 requests in 10 seconds | 429: "Too Many Requests" after threshold |
| TC-VIMP-189 | Large payload injection | S | P1 | POST /api/invoices | Payload > 10 MB body | 413: "Request Entity Too Large" |
| TC-VIMP-190 | Response contains no sensitive data | S | P1 | GET /api/invoices/INV-0501 | Valid vendor request for own invoice | Response MUST NOT contain: other vendor data, internal payment IDs, system paths, stack traces |
| TC-VIMP-191 | Error messages reveal no stack traces | S | P1 | Any endpoint with invalid input | Trigger 500 error | Error response: generic message only. No stack trace, no DB schema, no file paths. |

---

### Module 9 — Database Tests

Database tests verify the data layer independently, ensuring correctness beyond what the API surface exposes.

| TC-ID | Title | Type | Pri | What to Verify |
|-------|-------|------|-----|----------------|
| TC-VIMP-200 | Invoice saved correctly on submission | P | P1 | DB: invoice record matches submitted payload (invoice_no, po_id, vendor_id, amount, status=SUBMITTED, submitted_at) |
| TC-VIMP-201 | Status transitions are atomic | E | P1 | DB: no invoice exists in an undefined intermediate state (e.g., null status) after concurrent approval |
| TC-VIMP-202 | Audit trail entries are immutable | S | P1 | DB: `audit_log` table rows cannot be updated or deleted by any application role; only INSERT is permitted |
| TC-VIMP-203 | No orphaned invoice records | E | P2 | DB: every invoice record has a valid `vendor_id` FK; every `vendor_id` has an existing user record |
| TC-VIMP-204 | File attachment stored with invoice ID | P | P1 | DB + Storage: attachment record linked to invoice_id; deleting invoice hard-deletes or tombstones attachment |
| TC-VIMP-205 | Duplicate invoice blocked at DB level | S | P1 | DB: UNIQUE constraint on (invoice_number, po_id, vendor_id) prevents duplicates even if API validation bypassed |
| TC-VIMP-206 | Payment forwarding logged | P | P1 | DB: `payment_forward_log` record created on each forwarding attempt: invoice_id, attempt_time, status (SUCCESS/FAILED), response_payload |
| TC-VIMP-207 | User deactivation cascades to sessions | S | P1 | DB: user set to DEACTIVATED → all rows in `active_sessions` for that user are deleted or invalidated |
| TC-VIMP-208 | Report data matches live invoice data | P | P2 | DB: aggregate query over invoices for May 2026 matches stored report figures (no stale caching) |

---

### Module 10 — Non-Functional Testing

#### 10.1 Performance & Load Tests

| TC-ID | Title | Type | Pri | Scenario | Success Criteria |
|-------|-------|------|-----|---------|-----------------|
| TC-VIMP-220 | Page load time — vendor dashboard | Perf | P2 | Single user, no load | p95 < 2 seconds |
| TC-VIMP-221 | Invoice submission under 50 concurrent users | Load | P2 | 50 users simultaneously submit invoices | p95 response < 3s; error rate < 0.1% |
| TC-VIMP-222 | AP approval queue with 1000 concurrent users | Load | P2 | 1000 concurrent users accessing portal | p95 < 5s; no 5xx errors |
| TC-VIMP-223 | File upload: 5 MB PDF under load | Load | P2 | 20 concurrent uploads at max size | All uploads complete; no 413/500; storage persists |
| TC-VIMP-224 | Monthly report generation (large dataset) | Perf | P2 | Report for month with 10,000 invoices | Report generated within 60 seconds |
| TC-VIMP-225 | Stress test: ramp to 2× expected load | Stress | P2 | Ramp from 50 → 2000 users | Graceful degradation (queuing) rather than crash; system recovers after load drops |
| TC-VIMP-226 | Soak test: 72-hour moderate load | Reliability | P2 | 50 concurrent users for 72 hours | No memory leak; no performance degradation over time; uptime ≥ 99.5% |

#### 10.2 Security Tests (OWASP Top 10)

| TC-ID | Title | OWASP Category | Pri | Test Action | Pass Condition |
|-------|-------|---------------|-----|------------|----------------|
| TC-VIMP-230 | SQL Injection — all input fields | A03 Injection | P1 | Automated SQLi scan + manual: `' OR 1=1--` in all fields | No SQL errors, no unintended data returned |
| TC-VIMP-231 | XSS — stored XSS in invoice fields | A03 Injection | P1 | Submit `<script>alert('xss')</script>` in all text fields | Script not executed when data is displayed |
| TC-VIMP-232 | Broken Authentication — brute force | A07 Auth Failures | P1 | Automated rapid login attempts | Rate limiting and lockout enforced |
| TC-VIMP-233 | Sensitive Data Exposure — passwords in API response | A02 Crypto Failures | P1 | Inspect all API responses for credential fields | No passwords, tokens, or keys in response bodies |
| TC-VIMP-234 | Sensitive Data Exposure — HTTPS enforcement | A02 Crypto Failures | P1 | Access portal over HTTP | Automatic redirect to HTTPS; no content served over HTTP |
| TC-VIMP-235 | IDOR — access other users' data | A01 Broken Access Control | P1 | Enumerate invoice IDs and vendor IDs | 403/404 for any resource not owned by the requestor |
| TC-VIMP-236 | Security Misconfiguration — verbose errors | A05 Security Misconfig | P1 | Trigger 500 errors | No stack traces, DB schemas, or file paths in responses |
| TC-VIMP-237 | Security Misconfiguration — directory listing | A05 Security Misconfig | P1 | Attempt /uploads/, /files/, /assets/ as directory | 403 or 404; no file listing |
| TC-VIMP-238 | CSRF — state-changing operations | A01 Broken Access Control | P1 | Craft cross-origin POST to approve/reject endpoints | CSRF token validation blocks request |
| TC-VIMP-239 | Session Fixation | A07 Auth Failures | P1 | Obtain session token before login; check if same token used post-login | New session token issued on successful login |
| TC-VIMP-240 | File Upload Bypass — malicious file | A04 Insecure Design | P1 | Upload .exe, .php, .js renamed as .pdf | Server-side MIME validation rejects the file |
| TC-VIMP-241 | Insecure Direct Object Reference — payment records | A01 Broken Access Control | P1 | Vendor requests another vendor's payment record | 403 or 404; no payment data exposed |

#### 10.3 Usability Tests

| TC-ID | Title | Pri | Test Scenario | Pass Condition |
|-------|-------|-----|--------------|----------------|
| TC-VIMP-250 | First-time vendor completes invoice submission | P2 | New vendor, no training — can they submit an invoice? | Task completed without seeking help within 10 minutes |
| TC-VIMP-251 | Error messages are clear and actionable | P2 | Trigger all validation errors in invoice form | Every error message tells the user what went wrong AND what to do next |
| TC-VIMP-252 | Rejection reason visible to vendor | P2 | AP rejects with reason; vendor logs in to check | Rejection reason is prominently displayed on invoice detail page |
| TC-VIMP-253 | Mobile responsiveness (viewport 375px) | P2 | Access portal on 375px viewport (iPhone SE) | Core workflows (submit invoice, view status) are usable without horizontal scroll |

#### 10.4 Accessibility Tests (WCAG 2.1 AA)

| TC-ID | Title | Pri | Criterion | Test Method |
|-------|-------|-----|-----------|------------|
| TC-VIMP-255 | Keyboard navigation — full form | P2 | WCAG 2.1.1 | Navigate entire registration and invoice submission form using Tab/Shift-Tab/Enter only |
| TC-VIMP-256 | Screen reader compatibility | P2 | WCAG 4.1.2 | NVDA or VoiceOver reads all form labels, buttons, and error messages correctly |
| TC-VIMP-257 | Colour contrast — text on backgrounds | P2 | WCAG 1.4.3 | Automated: axe DevTools; min contrast ratio 4.5:1 for normal text |
| TC-VIMP-258 | Error not conveyed by colour alone | P2 | WCAG 1.4.1 | Validation errors include text + icon, not just red colour |

#### 10.5 Compatibility Tests

| TC-ID | Browser / OS | Pri | Scope |
|-------|-------------|-----|-------|
| TC-VIMP-260 | Chrome latest / Windows 11 | P2 | Full functional smoke suite |
| TC-VIMP-261 | Firefox latest / Windows 11 | P2 | Full functional smoke suite |
| TC-VIMP-262 | Safari latest / macOS 15 | P2 | Full functional smoke suite |
| TC-VIMP-263 | Edge latest / Windows 11 | P2 | Full functional smoke suite |

#### 10.6 Documentation Tests

| TC-ID | Title | Pri | What to Verify |
|-------|-------|-----|----------------|
| TC-VIMP-265 | User guide matches actual UI labels | P2 | Every button, field, and menu item in user guide matches current portal UI |
| TC-VIMP-266 | Error messages in guide are current | P2 | Error message text in documentation matches actual error messages displayed |
| TC-VIMP-267 | API docs reflect latest endpoint changes | P2 | All documented endpoints exist; request/response examples match actual API behaviour |

---

### Module 11 — Integration Tests

Integration tests verify what happens at system boundaries — when VIMP talks to external services.

| TC-ID | Title | Ref | Type | Pri | Scenario | Expected Behaviour |
|-------|-------|-----|------|-----|---------|-------------------|
| TC-VIMP-270 | Payment gateway: successful forwarding | US-VIMP-013 | P | P1 | Mock returns 200 + payment ref | Invoice status = PAYMENT_INITIATED; ref stored |
| TC-VIMP-271 | Payment gateway: 503 unavailable | AC-VIMP-006b | N | P1 | Mock returns 503 | Retry queue triggered; AP Admin alert after 3 failures |
| TC-VIMP-272 | Payment gateway: 400 bad request (invalid bank details) | — | N | P1 | Mock returns 400 | Status = PAYMENT_FAILED; AP Admin alerted; vendor NOT notified of internal error |
| TC-VIMP-273 | Payment gateway: response delay >30s | — | E | P2 | Mock delays 35s before 200 | VIMP handles timeout gracefully; no duplicate request; invoice status updated when response received |
| TC-VIMP-274 | Email service: successful dispatch | US-VIMP-015 | P | P1 | Email service mock accepts message | Email delivered to capture mailbox; correct subject/body |
| TC-VIMP-275 | Email service: temporarily unavailable | — | N | P2 | Email service mock returns 503 | Email queued; retry within 5 min; notification sent eventually |
| TC-VIMP-276 | Email service: permanent failure (invalid domain) | — | N | P2 | Email service rejects (550 no such user) | Error logged; AP Admin alerted; no crash |
| TC-VIMP-277 | Auth/SSO: valid SSO token for internal user | OQ-13 | P | P1 | Valid SAML assertion from IdP | Internal user logged in with correct role; portal session created |
| TC-VIMP-278 | Auth/SSO: tampered SAML assertion | OQ-13 | S | P1 | Modified assertion | SAML signature validation fails; login denied |
| TC-VIMP-279 | Auth/SSO: SSO provider unavailable | OQ-13 | N | P2 | IdP returns 503 | Login fails gracefully: "Unable to connect to authentication provider. Please try again." |

---

## 10. Decision Tables

Decision tables map combinations of conditions to expected system behaviour. This technique (QA Excellence, Slide 6) is applied to the three most complex flows.

### 10.1 Invoice Approval Decision Table

| Condition | Case 1 | Case 2 | Case 3 | Case 4 | Case 5 | Case 6 |
|-----------|--------|--------|--------|--------|--------|--------|
| User Role | AP User | AP User | Vendor | Finance | AP Admin | AP User |
| Invoice Status | SUBMITTED | APPROVED | SUBMITTED | SUBMITTED | SUBMITTED | REJECTED |
| Same-user submitted? | No | No | N/A | N/A | No | No |
| **Action: Approve** | | | | | | |
| Expected Result | APPROVED ✓ | Error: Already Approved | HTTP 403 | HTTP 403 | APPROVED ✓ | Error: Wrong Status |

### 10.2 Invoice Submission Decision Table

| Condition | Case 1 | Case 2 | Case 3 | Case 4 | Case 5 | Case 6 |
|-----------|--------|--------|--------|--------|--------|--------|
| Vendor Account Status | Active | Pending | Deactivated | Active | Active | Active |
| PO Assigned to Vendor | Yes | Yes | Yes | No | Yes | Yes |
| Invoice is Duplicate | No | No | No | No | Yes | No |
| File Size | ≤5 MB | ≤5 MB | ≤5 MB | ≤5 MB | ≤5 MB | >5 MB |
| **Expected Result** | SUBMITTED ✓ | HTTP 403 | HTTP 403 | PO Error | Duplicate Error | File Size Error |

### 10.3 Notification Trigger Decision Table

| Status Change | Vendor Notified? | AP User Notified? | AP Admin Notified? |
|--------------|-----------------|------------------|-------------------|
| SUBMITTED | ✓ Confirmation | ✓ New invoice in queue | — |
| APPROVED | ✓ Approval email | — | — |
| REJECTED | ✓ Rejection + reason + deeplink | — | — |
| PAYMENT_INITIATED | ✓ Payment on the way | — | — |
| PAYMENT_FAILED | — | — | ✓ Alert |
| PENDING_PAYMENT_FORWARDING (3 retries) | — | — | ✓ Alert |

---

## 11. Boundary Value Analysis Summary

All BVA test cases from the QA Excellence framework (Slide 6) applied to VIMP:

| Field | Valid Range | Boundary Values Tested | TC Reference |
|-------|------------|----------------------|-------------|
| File Size | 0 < size ≤ 5 MB | 4.9 MB ✓, 5.0 MB ✓, 5.1 MB ✗ | TC-VIMP-034/035/036 |
| Invoice Amount | > 0 | -1.00 ✗, 0.00 ✗, 0.01 ✓, 9,999,999.99 ✓ | TC-VIMP-039/040 |
| Invoice Date | today−180 to today | today−181 ✗, today−180 ✓, today ✓, today+1 ✗ | TC-VIMP-041/042/043 |
| Password Length | min 8 chars | 5 chars ✗, 7 chars ✗, 8 chars ✓ | TC-VIMP-007/008 |
| Rejection Reason | 1–500 chars | 0 ✗, 1 ✓, 500 ✓, 501 ✗ | TC-VIMP-069/070/071/072 |
| Failed Login Attempts | Lockout at 5 | 4 attempts (no lock) ✓, 5th (lock) ✗ | TC-VIMP-015/016 |
| Session Timeout | idle > N min | At timeout ✗, at timeout−1 ✓ | TC-VIMP-019 |
| Report Date Range | Calendar month 00:00–23:59:59 | Dec 31 23:59:59 (in Dec) ✓, Jan 1 00:00:00 (in Jan) ✓ | TC-VIMP-131/132 |

---

## 12. End-to-End Test Scenarios

Three golden-path E2E scenarios covering the full portal lifecycle.

### E2E-01: Complete Invoice Lifecycle (Happy Path)

**Scenario:** Vendor submits invoice → AP approves → Payment forwarded → All parties notified

| Step | Actor | Action | Expected State |
|------|-------|--------|---------------|
| 1 | Vendor | Registers and verifies email | Account = ACTIVE |
| 2 | Vendor | Logs in, submits INV-E2E-001 against PO-E2E-100 with 2MB PDF | Invoice = SUBMITTED |
| 3 | System | Notifies AP team of new submission | AP email received within 5 min |
| 4 | AP User | Reviews and approves INV-E2E-001 | Invoice = APPROVED, audit trail created |
| 5 | System | Auto-forwards to payment system | Invoice = PAYMENT_INITIATED, pay ref stored |
| 6 | System | Notifies vendor of approval | Vendor email received with correct invoice details |
| 7 | Vendor | Logs in to check status | Sees PAYMENT_INITIATED status |
| 8 | Finance User | Logs in to view report | Invoice appears in current month's activity |

### E2E-02: Rejection and Resubmission Flow

| Step | Actor | Action | Expected State |
|------|-------|--------|---------------|
| 1 | Vendor | Submits INV-E2E-002 with incorrect PO amount | Invoice = SUBMITTED |
| 2 | AP User | Rejects with reason "Amount mismatch — PO value is ₹40,000" | Invoice = REJECTED, vendor email with reason |
| 3 | Vendor | Views rejection reason, corrects invoice | — |
| 4 | Vendor | Resubmits corrected INV-E2E-002-R1 | Invoice = SUBMITTED (new record or revision) |
| 5 | AP User | Approves revised invoice | Invoice = APPROVED, payment triggered |

### E2E-03: Security Enforcement — Unauthorised Access Attempt

| Step | Actor | Action | Expected Result |
|------|-------|--------|----------------|
| 1 | Vendor A | Logs in | Active session |
| 2 | Vendor A | Attempts to access Vendor B's invoice via URL manipulation | HTTP 403, no data returned |
| 3 | Vendor A | Attempts to access /admin/users | HTTP 403 |
| 4 | Attacker | Attempts to approve invoice with no auth token | HTTP 401 |
| 5 | Attacker | Uses tampered JWT with escalated role | HTTP 401 — signature invalid |

---

## 13. Smoke / Sanity Test Set

Executed immediately after every deployment. Must all pass before any further testing.

| TC-ID | Title | Layer | Expected |
|-------|-------|-------|---------|
| SMK-01 | Portal loads at root URL | UI | HTTP 200, login page renders |
| SMK-02 | Vendor login | UI + API | Valid credentials → dashboard |
| SMK-03 | AP User login | UI + API | Valid credentials → AP dashboard |
| SMK-04 | Invoice submission (minimal valid) | UI + API | Invoice created with SUBMITTED status |
| SMK-05 | Invoice appears in AP queue | UI | Submitted invoice visible to AP User |
| SMK-06 | Approve invoice | UI + API | Invoice transitions to APPROVED |
| SMK-07 | Rejection creates rejection reason | API + DB | Rejection reason stored, status = REJECTED |
| SMK-08 | Notification email triggered on approval | INT | Email captured in test mailbox within 5 min |
| SMK-09 | Vendor cannot access AP queue | API | HTTP 403 |
| SMK-10 | Unauthenticated request blocked | API | HTTP 401 |
| SMK-11 | File upload (valid PDF) | UI | File accepted without error |
| SMK-12 | File upload (over limit) | UI | File rejected with size error |
| SMK-13 | Monthly report accessible to AP Admin | UI | Report renders or download available |
| SMK-14 | Logout clears session | UI + API | Post-logout request returns 401 |
| SMK-15 | Health check endpoint responds | API | GET /health → 200 OK |

---

## 14. Regression Strategy

| Trigger | Scope |
|---------|-------|
| Every sprint release | Full regression suite (all TCs) |
| Hotfix / patch | Smoke set (SMK-01 to SMK-15) + all TCs in affected module |
| Infrastructure change | Performance baseline rerun + full smoke |
| Security patch | Full OWASP security scan |
| Dependency upgrade | API contract tests + auth flow tests |

**Regression Automation Priority:**
1. Smoke set — 100% automated
2. RBAC tests (TC-VIMP-150–161) — 100% automated (API layer)
3. Invoice submission validations — 80% automated
4. AP approval workflow — 80% automated
5. Notification tests — 70% automated (email capture)
6. Performance — scripted (JMeter)
7. Exploratory / usability — manual only

---

## 15. Defect Management

All defects raised in the bug tracking system with the following fields (QA Excellence, Slide 17):

| Field | Description |
|-------|-------------|
| Title | Descriptive, one-line: `[Module] Vendor can submit invoice with amount = 0` |
| Steps to Reproduce | Exact numbered steps; any tester can reproduce |
| Expected vs Actual | Side-by-side; no ambiguity |
| Environment | Browser, OS, env (Staging/UAT), app version |
| Screenshots / Logs | Mandatory for every defect |
| Severity | Critical / High / Medium / Low |
| Priority | P1 (fix now) → P4 (fix eventually) |
| Reproducibility | Always / Intermittent / Once |

**Severity Definitions:**

| Severity | Criteria |
|----------|----------|
| Critical | Data loss, security breach, payment processing failure, system crash |
| High | Core workflow blocked (cannot submit, approve, or reject); RBAC bypass |
| Medium | Feature partially broken; workaround exists |
| Low | UI cosmetic, minor copy error, non-critical edge case |

---

## 16. Test Metrics

How we measure coverage and quality (QA Excellence, Slide 5 — METRICS):

| Metric | Formula | Target |
|--------|---------|--------|
| Test Coverage | Test cases executed / Total test cases planned × 100 | ≥ 95% before release |
| Requirement Coverage | US-VIMP covered by ≥1 executed TC / Total US-VIMP × 100 | 100% |
| Defect Detection Rate | Defects found in QA / Total defects found × 100 | ≥ 90% (minimise escapes to prod) |
| Defect Escape Rate | Defects in production / Total defects × 100 | < 5% |
| Defect Density | Defects per story | < 2 per story at release |
| P1 Defect Resolution Time | Time from report to close | < 24 hours |
| Blocked Test Cases | TCs unable to execute due to environment/build issues | 0 at release |

---

## 17. Tools

| Category | Tool | Purpose |
|----------|------|---------|
| API Testing | Postman / REST Assured | Contract tests, auth tests, data integrity |
| Performance | Apache JMeter / k6 | Load, stress, soak tests |
| Security | OWASP ZAP / Burp Suite | Automated scan + manual pen test |
| Accessibility | axe DevTools | WCAG 2.1 AA automated checks |
| Browser Compatibility | BrowserStack | Cross-browser test execution |
| Email Testing | Mailhog / Mailtrap | Email capture in staging |
| Test Management | (To be confirmed: TestRail / Jira Xray) | TC management, execution, reporting |
| Bug Tracking | Jira | Defect lifecycle management |
| CI/CD Integration | Jenkins / GitHub Actions | Automated smoke + regression on every build |
| DB Testing | DBeaver + custom SQL scripts | Data integrity, constraint, audit trail checks |

---

## 18. Shift-Left Activity Checklist

Actions QA must take at each phase of development, before formal test execution begins.

| Phase | QA Activity |
|-------|------------|
| Requirements | Review US-VIMP-NNN for testability; raise AC ambiguities via `03_requirements_clarification.md` |
| Design | Review wireframes against BR-VIMP-NNN; flag flows that would make test impossible |
| API Contract | Review OpenAPI/Swagger spec; verify error codes, auth headers, required fields |
| Three Amigos | Attend session for each story; confirm AC is shared understanding |
| Sprint start | Prepare test data for stories in sprint |
| Dev in progress | Run unit-test coverage checks; flag if < 80% |
| Story complete | Run AC-VIMP-NNN for that story before Demo |
| Demo | Verify demo covers at least one positive and one negative path |

---

*Document version: 1.0 | Status: Draft — Pending PRD Approval and OQ Resolution*  
*Total planned test cases: ~280 (across all modules, types, and layers)*

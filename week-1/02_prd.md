# Product Requirements Document
## Vendor Invoice Management Portal (VIMP)

**Document Type:** Product Requirements Document (PRD)  
**Module:** VIMP — Vendor Invoice Management Portal  
**Prepared by:** Product Analyst  
**Date:** 2026-06-06  
**Version:** 1.0 — Draft  
**Source:** Client Requirements, `01_QA_Excellence.pptx` (Slides 31–32)

---

## 1. Overview

The Vendor Invoice Management Portal (VIMP) is a B2B web platform that digitises and governs the end-to-end invoice lifecycle between external vendors and an internal Accounts Payable (AP) team. The portal replaces manual, email-driven invoice workflows with a structured submission, approval, and payment-forwarding pipeline — giving both vendors and the AP team a single source of truth for invoice status.

This document defines what VIMP must do, for whom, and the criteria by which completion is judged. It is the primary contract between the product team and the client, and the direct input for test planning and QA artifact generation.

---

## 2. Scope

### 2.1 In Scope

The following functional areas are covered by this PRD:

| Area | Description |
|------|-------------|
| Vendor Registration & Authentication | Self-service registration, login, session management |
| Invoice Submission | Upload and submission of invoices against Purchase Orders |
| AP Approval Workflow | View, approve, and reject invoices with audit trail |
| Payment Forwarding | Automatic forwarding of approved invoices to the payment system |
| Notifications | Email notifications to vendors and AP team on status changes |
| Reporting | Monthly invoice activity report generation |
| Access Control | Role-based access control (RBAC) for all portal users |

### 2.2 Out of Scope

The following are explicitly excluded from this module and deferred:

- Integration with specific ERP or accounting systems (SAP, Oracle) — interface contract only
- Vendor onboarding by the procurement team (pre-portal workflow)
- Payment execution (bank transfers, cheque issuance) — VIMP only forwards to the payment system
- Mobile application — web portal only
- Multi-language / i18n support
- Self-service report customisation beyond monthly standard report

---

## 3. User Roles

VIMP serves three primary user types. Every requirement and story below is written in terms of these roles.

| Role ID | Role | Description |
|---------|------|-------------|
| R-01 | Vendor | External company representative who submits invoices |
| R-02 | AP User | Internal accounts payable team member who reviews and approves invoices |
| R-03 | AP Admin | Senior AP team member with user management and reporting authority |
| R-04 | Finance User | Internal finance team member with read-only access to payment status and reports |
| R-05 | System Admin | Technical administrator responsible for system configuration and user provisioning |

---

## 4. User Stories

Each story is identified as `US-VIMP-NNN` and traces to a client requirement from the source PPT.

### Module: Authentication & Registration

**US-VIMP-001 — Vendor Self-Registration**  
*Source: REQ-01*  
As a **Vendor**, I want to register an account on the portal so that I can submit invoices to the AP team.

**US-VIMP-002 — Vendor Login**  
*Source: REQ-01*  
As a **Vendor**, I want to log in to the portal securely so that I can access my invoices and submission history.

**US-VIMP-003 — AP / Internal User Login**  
*Source: REQ-01, REQ-07*  
As an **AP User / AP Admin / Finance User**, I want to log in to the portal using my corporate credentials so that I can access functions permitted by my role.

**US-VIMP-004 — Session Timeout & Re-authentication**  
*Source: REQ-01, REQ-07*  
As any authenticated user, I want my session to expire after a period of inactivity so that unauthorised parties cannot access the portal if I leave my device unattended.

---

### Module: Invoice Submission

**US-VIMP-005 — Submit Invoice Against a PO**  
*Source: REQ-02*  
As a **Vendor**, I want to submit an invoice referencing a Purchase Order so that the AP team can match and process my payment request.

**US-VIMP-006 — Attach Supporting Documents**  
*Source: REQ-02*  
As a **Vendor**, I want to attach supporting documents (delivery notes, receipts) to my invoice submission so that the AP team has the evidence needed to approve without back-and-forth.

**US-VIMP-007 — View Submission History**  
*Source: REQ-02, REQ-03*  
As a **Vendor**, I want to view all my submitted invoices and their current status so that I know what is pending, approved, rejected, or paid.

**US-VIMP-008 — Resubmit a Rejected Invoice**  
*Source: REQ-02, REQ-03*  
As a **Vendor**, I want to resubmit an invoice that was rejected so that I can correct the issue and still receive payment.

---

### Module: AP Approval Workflow

**US-VIMP-009 — View All Pending Invoices**  
*Source: REQ-03*  
As an **AP User**, I want to see a list of all invoices pending review so that I can prioritise and work through the approval queue.

**US-VIMP-010 — Approve an Invoice**  
*Source: REQ-03*  
As an **AP User**, I want to approve an invoice so that it is forwarded for payment processing.

**US-VIMP-011 — Reject an Invoice**  
*Source: REQ-03*  
As an **AP User**, I want to reject an invoice with a documented reason so that the vendor can understand and correct the issue.

**US-VIMP-012 — View Invoice Audit Trail**  
*Source: REQ-03, REQ-07*  
As an **AP Admin**, I want to view the full history of actions on any invoice (submitted, reviewed by whom, approved/rejected, timestamps) so that I can support audits and dispute resolution.

---

### Module: Payment Forwarding

**US-VIMP-013 — Automatic Payment Forwarding**  
*Source: REQ-04*  
As an **AP Admin**, I want approved invoices to be automatically forwarded to the payment processing system so that payment is initiated without manual intervention.

**US-VIMP-014 — View Payment Status**  
*Source: REQ-04*  
As a **Finance User** or **AP Admin**, I want to see the payment status of approved invoices so that I can confirm payment has been initiated and completed.

---

### Module: Notifications

**US-VIMP-015 — Vendor Notification on Status Change**  
*Source: REQ-05*  
As a **Vendor**, I want to receive an email notification whenever my invoice status changes so that I am informed without having to log in and check manually.

**US-VIMP-016 — AP Team Notification on New Submission**  
*Source: REQ-05*  
As an **AP User**, I want to receive an email notification when a vendor submits a new invoice so that I can process it within the agreed SLA.

---

### Module: Reporting

**US-VIMP-017 — Monthly Invoice Activity Report**  
*Source: REQ-06*  
As an **AP Admin** or **Finance User**, I want to access a monthly invoice activity report so that I can review submission volumes, approval rates, and outstanding items.

---

### Module: Access Control

**US-VIMP-018 — Role-Based Access Enforcement**  
*Source: REQ-07*  
As a **System Admin**, I want all portal features to be restricted by user role so that vendors cannot access AP functions and AP users cannot access system configuration.

**US-VIMP-019 — User Provisioning and Deprovisioning**  
*Source: REQ-07*  
As a **System Admin**, I want to create, modify, and deactivate user accounts and role assignments so that access is always current and reflects organisational changes.

---

## 5. Business Rules

Business rules govern system behaviour and cannot be overridden by individual users.

| Rule ID | Rule | Applies To |
|---------|------|-----------|
| BR-VIMP-001 | An invoice can only be submitted if it references a valid, open Purchase Order registered in the system. | US-VIMP-005 |
| BR-VIMP-002 | Only a vendor whose account is active and verified may submit invoices. Pending or deactivated vendors are blocked. | US-VIMP-005 |
| BR-VIMP-003 | An invoice cannot be approved or rejected by the same user who submitted it (segregation of duties). | US-VIMP-010, US-VIMP-011 |
| BR-VIMP-004 | A rejection must always include a reason. The reason field cannot be empty. | US-VIMP-011 |
| BR-VIMP-005 | Only invoices in "Approved" status are forwarded to the payment system. Rejected or pending invoices are never forwarded. | US-VIMP-013 |
| BR-VIMP-006 | Payment forwarding is triggered automatically upon approval. No manual trigger is required. | US-VIMP-013 |
| BR-VIMP-007 | A vendor may only view their own invoices. They cannot view invoices submitted by other vendors. | US-VIMP-007 |
| BR-VIMP-008 | The monthly report covers all invoices with activity within the calendar month (00:00 on the 1st to 23:59 on the last day). | US-VIMP-017 |
| BR-VIMP-009 | A user's role determines which portal sections they can access. Attempting to access an unauthorised section must return an access denied response, not an error. | US-VIMP-018 |
| BR-VIMP-010 | When a user account is deactivated, all active sessions for that account are immediately invalidated. | US-VIMP-019 |
| BR-VIMP-011 | Duplicate invoice detection: if a vendor submits an invoice with the same invoice number against the same PO, the system must block the submission and display an error. | US-VIMP-005 |

---

## 6. Acceptance Criteria

Acceptance criteria follow **Given / When / Then** format. Each `AC-VIMP-NNN` traces to a user story and includes exact test data, negative paths, boundary conditions, and security scenarios. All criteria must be independently verifiable and reproducible by any tester.

> Test data marked with `★` are specific boundary values derived from Boundary Value Analysis (BVA). Test data marked with `⊕` are equivalence partition representatives.

---

### AC-VIMP-001 — Vendor Registration (→ US-VIMP-001)

**AC-VIMP-001a — Happy Path: Valid Registration**  
**Given** a new vendor visits `/register` and no account exists for `vendor_new@testco.com`  
**When** they submit: Company Name = "TestCo Pvt Ltd", Contact Name = "John Smith", Email = "vendor_new@testco.com" `⊕`, Password = "Secure@123", Tax ID = "22AAAAA0000A1Z5"  
**Then** (1) HTTP 201 response; (2) account created with status = `PENDING_VERIFICATION`; (3) verification email dispatched to "vendor_new@testco.com" within 2 minutes; (4) user is NOT auto-logged in; (5) DB record confirms created_at timestamp within ±5 seconds of submission.

**AC-VIMP-001b — Email Verification: Within Window**  
**Given** a vendor submitted registration 1 hour ago and received a verification link  
**When** they click the verification link (valid, within 24-hour window)  
**Then** account status changes to `ACTIVE`; vendor is redirected to login page with message: "Your email has been verified. Please log in."; verification link is invalidated and cannot be used again.

**AC-VIMP-001c — Email Verification: Expired Link**  
**Given** a vendor received a verification email 25 hours ago `★` (beyond the 24-hour window)  
**When** they click the verification link  
**Then** system displays: "This verification link has expired. Please request a new one." and the account remains in `PENDING_VERIFICATION` state. A "Resend verification email" option is visible.

**AC-VIMP-001d — Duplicate Email Registration**  
**Given** an account already exists for "existing_vendor@co.com"  
**When** a new registration is submitted with email = "existing_vendor@co.com"  
**Then** the form is rejected with field-level error: "An account with this email address already exists." No new account is created. No duplicate DB record exists.

**AC-VIMP-001e — Missing Mandatory Fields (field by field)**  
**Given** the registration form is open  
**When** submitted with: (a) Company Name blank; (b) Email blank; (c) Password blank; (d) Contact Name blank  
**Then** for each case: the specific field is highlighted in red with inline error "This field is required". Form does not submit. Other fields retain their entered values.

**AC-VIMP-001f — Invalid Email Format**  
**Given** the registration form is open  
**When** submitted with email = (a) "userwithoutat.com" `⊕`; (b) "@nodomain.com" `⊕`; (c) "spaces in@email.com" `⊕`; (d) "user@" `⊕`  
**Then** for each: field-level error "Please enter a valid email address." No account created.

**AC-VIMP-001g — Weak Password**  
**Given** the registration form is open  
**When** submitted with password = (a) "abc" `★` (below minimum length); (b) "alllowercase1" `⊕` (no uppercase/special char); (c) "SHORT1!" `★` (7 chars, 1 below minimum 8)  
**Then** for each: inline error "Password must be at least 8 characters and include one uppercase letter, one number, and one special character." Form does not submit.

**AC-VIMP-001h — Password Confirmation Mismatch**  
**Given** vendor enters Password = "Secure@123" and Confirm Password = "Secure@124"  
**When** the form is submitted  
**Then** inline error on confirm field: "Passwords do not match." Form does not submit.

**AC-VIMP-001i — XSS Attempt in Company Name**  
**Given** vendor enters Company Name = `<script>alert('xss')</script>`  
**When** the form is submitted  
**Then** the input is sanitised; the stored company name is the literal string `<script>alert('xss')</script>` (HTML-encoded) or the submission is rejected with "Invalid characters in Company Name." No script executes.

**AC-VIMP-001j — SQL Injection in Email Field**  
**Given** vendor enters Email = `' OR '1'='1`  
**When** the form is submitted  
**Then** the system validates email format and rejects with "Please enter a valid email address." No SQL error surfaces. No unintended DB operation executes.

**AC-VIMP-001k — Resend Verification Email**  
**Given** a vendor's account is in `PENDING_VERIFICATION` state  
**When** they request a resend of the verification email  
**Then** a new verification email is dispatched within 2 minutes. The previous verification link is invalidated. Resend is rate-limited: a second resend request within 60 seconds `★` returns: "Please wait before requesting another verification email."

---

### AC-VIMP-002 — Vendor Login (→ US-VIMP-002)

**AC-VIMP-002a — Happy Path: Valid Login**  
**Given** vendor account "vendor_active@testco.com" has status `ACTIVE` and password "Secure@123"  
**When** they enter email = "vendor_active@testco.com" and password = "Secure@123"  
**Then** (1) HTTP 200 + session token issued; (2) redirected to vendor dashboard `/dashboard`; (3) dashboard displays vendor's company name; (4) session token stored as HttpOnly cookie (not accessible via JS).

**AC-VIMP-002b — Case-Insensitive Email**  
**Given** account exists for "vendor@testco.com"  
**When** login attempted with "VENDOR@TESTCO.COM" `⊕`  
**Then** login succeeds — email matching is case-insensitive.

**AC-VIMP-002c — Wrong Password (Failed Attempt Tracking)**  
**Given** account "vendor_active@testco.com" has 0 failed attempts  
**When** login attempted with password = "WrongPass1!" (attempts 1–4 individually)  
**Then** each attempt returns: "Invalid email or password." — no indication of which field is wrong. Failed attempt counter increments by 1 per attempt. Account remains unlocked through attempt 4.

**AC-VIMP-002d — Account Lockout on 5th Failed Attempt**  
**Given** "vendor_active@testco.com" has 4 failed attempts already recorded  
**When** a 5th incorrect password is submitted `★`  
**Then** (1) account status transitions to `LOCKED`; (2) response: "Your account has been locked after 5 failed attempts. A reset link has been sent to your email."; (3) lockout email dispatched within 2 minutes; (4) even correct password on 6th attempt returns locked message; (5) DB: `failed_attempts = 5`, `locked_at` timestamp set.

**AC-VIMP-002e — Login with Pending Verification Account**  
**Given** account status = `PENDING_VERIFICATION`  
**When** login attempted with correct credentials  
**Then** response: "Please verify your email address before logging in. Check your inbox or request a new verification email." Access denied.

**AC-VIMP-002f — Login with Deactivated Account**  
**Given** account status = `DEACTIVATED`  
**When** login attempted with correct credentials  
**Then** response: "Your account has been deactivated. Please contact support." Access denied. No session token issued.

**AC-VIMP-002g — Session Timeout**  
**Given** a vendor is logged in and has been idle for [session_timeout + 1 minute] `★`  
**When** they attempt any authenticated action  
**Then** session is invalidated; user is redirected to login with message: "Your session has expired. Please log in again."

**AC-VIMP-002h — Concurrent Session (Same Account, Different Browser)**  
**Given** vendor is logged in on Browser A  
**When** they log in again on Browser B (per policy: single session enforced)  
**Then** the Browser A session is invalidated; Browser A's next request redirects to login with: "You have been logged out because your account was accessed from another location."

**AC-VIMP-002i — Empty Credentials Submission**  
**Given** login form is open  
**When** submitted with (a) empty email and valid password; (b) valid email and empty password; (c) both empty  
**Then** for each case: relevant field highlighted with "This field is required." No HTTP call made (client-side validation prevents submission).

**AC-VIMP-002j — SQL Injection in Login Fields**  
**Given** login form is open  
**When** Email = `admin'--` and Password = `anything`  
**Then** login fails with "Invalid email or password." No SQL error exposed. No unauthorised session created.

---

### AC-VIMP-003 — Invoice Submission Against PO (→ US-VIMP-005, US-VIMP-006)

**AC-VIMP-003a — Happy Path: Valid Invoice Submission**  
**Given** vendor "vendor_active@testco.com" is logged in, PO "PO-2024-1001" exists and is assigned to this vendor with remaining value ≥ INV amount  
**When** they submit: Invoice No = "INV-2024-0501", PO = "PO-2024-1001", Date = [today], Amount = 50,000.00 INR, 3 line items, attachment = "delivery_note.pdf" (2 MB `⊕`)  
**Then** (1) invoice created with status = `SUBMITTED`; (2) confirmation email dispatched within 5 min; (3) invoice appears in vendor's submission history; (4) invoice appears in AP pending queue; (5) DB: `invoice_id` generated, `submitted_at` timestamp, `vendor_id`, `po_id`, `status = SUBMITTED`.

**AC-VIMP-003b — PO Does Not Exist**  
**Given** vendor submits invoice referencing PO = "PO-INVALID-9999"  
**When** submission is attempted  
**Then** error: "PO number PO-INVALID-9999 was not found or is not assigned to your account." Invoice not created. DB unchanged.

**AC-VIMP-003c — PO Assigned to a Different Vendor**  
**Given** PO "PO-2024-2000" is assigned to vendor B, and vendor A is logged in  
**When** vendor A submits an invoice referencing PO "PO-2024-2000"  
**Then** error: "PO number PO-2024-2000 was not found or is not assigned to your account." (Same message as 003b — no information disclosure about PO existence for other vendors.)

**AC-VIMP-003d — Duplicate Invoice Detection (→ BR-VIMP-011)**  
**Given** invoice "INV-2024-0501" against PO "PO-2024-1001" is already `SUBMITTED` or `APPROVED`  
**When** same vendor submits invoice number "INV-2024-0501" against PO "PO-2024-1001" again  
**Then** error: "Duplicate invoice detected. Invoice INV-2024-0501 has already been submitted against PO-2024-1001." No duplicate DB record created.

**AC-VIMP-003e — Supported File Format Upload (PDF)**  
**Given** attachment is "invoice.pdf" of size 4.9 MB `★` (at boundary — 0.1 MB below 5 MB limit)  
**When** vendor uploads the file  
**Then** file accepted, preview/icon shown, no error. File stored on submission.

**AC-VIMP-003f — File at Exact Maximum Size**  
**Given** attachment is "invoice.pdf" of exactly 5.0 MB `★` (at the limit)  
**When** vendor uploads the file  
**Then** file accepted (limit is inclusive). No error.

**AC-VIMP-003g — File Exceeds Maximum Size**  
**Given** attachment is "invoice.pdf" of size 5.1 MB `★` (1 step over limit)  
**When** vendor uploads the file  
**Then** upload rejected immediately: "File size exceeds the 5 MB limit. Please upload a smaller file." No file stored.

**AC-VIMP-003h — Unsupported File Format**  
**Given** vendor selects attachment "malware.exe" `⊕`  
**When** file is selected in the upload control  
**Then** client-side rejection: "Unsupported file format. Accepted formats: PDF, JPEG, PNG, XLSX." File not uploaded to server.

**AC-VIMP-003i — Zero Invoice Amount**  
**Given** vendor enters invoice Total Amount = 0.00 `★`  
**When** submission attempted  
**Then** error: "Invoice amount must be greater than zero." Submission blocked.

**AC-VIMP-003j — Negative Invoice Amount**  
**Given** vendor enters invoice Total Amount = -1,000.00 `★`  
**When** submission attempted  
**Then** error: "Invoice amount must be a positive value." Submission blocked.

**AC-VIMP-003k — Future-Dated Invoice**  
**Given** vendor sets Invoice Date = [today + 1 day] `★`  
**When** submission attempted  
**Then** error: "Invoice date cannot be in the future." Submission blocked.

**AC-VIMP-003l — Invoice Date Beyond Acceptable Past Window**  
**Given** vendor sets Invoice Date = [today − 181 days] `★` (beyond 180-day lookback)  
**When** submission attempted  
**Then** error: "Invoice date is too old. Invoices cannot be backdated more than 180 days." Submission blocked.

**AC-VIMP-003m — Invoice Date at Edge of Acceptable Past Window**  
**Given** vendor sets Invoice Date = [today − 180 days] `★` (exactly at limit)  
**When** submission attempted  
**Then** submission accepted (180-day limit is inclusive).

**AC-VIMP-003n — Invoice with No Line Items**  
**Given** vendor completes all invoice header fields but adds 0 line items `★`  
**When** submission attempted  
**Then** error: "At least one line item is required." Submission blocked.

**AC-VIMP-003o — Invoice Amount Mismatches Sum of Line Items**  
**Given** line items sum to 48,000.00 but Total Amount entered = 50,000.00  
**When** submission attempted  
**Then** error: "Total amount (50,000.00) does not match the sum of line items (48,000.00). Please correct before submitting."

**AC-VIMP-003p — Deactivated Vendor Attempts Submission (→ BR-VIMP-002)**  
**Given** vendor account status = `DEACTIVATED` (API test: token still present in-flight)  
**When** invoice submission API called with valid token  
**Then** HTTP 403: "Your account is not authorised to submit invoices." No invoice created.

---

### AC-VIMP-004 — AP Invoice Approval (→ US-VIMP-010)

**AC-VIMP-004a — Happy Path: Single Approval**  
**Given** AP User "ap_user1@company.com" is viewing invoice "INV-2024-0501" in `SUBMITTED` status  
**When** they click Approve, confirm in the modal  
**Then** (1) invoice status = `APPROVED`; (2) audit trail entry: `{action: APPROVED, actor: ap_user1@company.com, timestamp: [now]}`; (3) payment forwarding triggered within 60 seconds; (4) vendor receives approval email; (5) invoice removed from AP pending queue; (6) AP User cannot re-approve the same invoice.

**AC-VIMP-004b — Approval of Already-Approved Invoice**  
**Given** invoice "INV-2024-0501" is in `APPROVED` status  
**When** AP User attempts to approve it again (e.g., via back-button replay or direct API call)  
**Then** error: "This invoice has already been approved." Status unchanged. No duplicate audit entry. No duplicate payment forwarding.

**AC-VIMP-004c — Approval of Rejected Invoice (Wrong Status)**  
**Given** invoice "INV-2024-0502" is in `REJECTED` status  
**When** AP User attempts to approve it  
**Then** error: "Invoice INV-2024-0502 cannot be approved — current status is REJECTED. Vendor must resubmit." Approval button is disabled in UI.

**AC-VIMP-004d — Segregation of Duties (→ BR-VIMP-003)**  
**Given** AP User "ap_user2@company.com" is also listed as contact for vendor "TestCo" and submitted an invoice via a test vendor account  
**When** "ap_user2@company.com" views that invoice in the AP queue  
**Then** the Approve and Reject buttons are disabled. Tooltip: "You cannot approve or reject an invoice you submitted." Attempting the action via API returns HTTP 403.

**AC-VIMP-004e — Concurrent Approval by Two AP Users (Race Condition)**  
**Given** two AP Users "ap_user1" and "ap_user2" simultaneously view invoice "INV-2024-0503" in `SUBMITTED` status  
**When** both click Approve within milliseconds of each other  
**Then** exactly one approval succeeds (first write wins); the second receives: "This invoice has already been actioned by another user. Please refresh." Invoice transitions to `APPROVED` exactly once. Audit trail has exactly one approval entry.

**AC-VIMP-004f — AP User Account Deactivated Mid-Session**  
**Given** AP User "ap_user3@company.com" is logged in and viewing invoice "INV-2024-0504"  
**When** a System Admin deactivates "ap_user3@company.com" mid-session  
**Then** "ap_user3@company.com"'s next action (including approval attempt) returns HTTP 401: "Your account has been deactivated." The approval does not persist even if the request was in-flight at deactivation time.

**AC-VIMP-004g — Approval with Optional Comment**  
**Given** AP User approves invoice and adds comment "Verified against PO-2024-1001 delivery receipt."  
**When** approval is confirmed  
**Then** comment is stored in the audit trail alongside approver ID and timestamp. Comment is visible to AP Admin in invoice history. Vendor does not see the internal comment.

**AC-VIMP-004h — Network Interruption During Approval**  
**Given** AP User clicks Approve and network drops immediately after click  
**When** the request does not complete  
**Then** invoice status remains `SUBMITTED` (no partial state). On network restore and page refresh, invoice still shows as `SUBMITTED`. AP User can re-attempt approval cleanly.

---

### AC-VIMP-005 — AP Invoice Rejection (→ US-VIMP-011)

**AC-VIMP-005a — Happy Path: Rejection with Reason**  
**Given** AP User views invoice "INV-2024-0510" in `SUBMITTED` status  
**When** they click Reject, enter reason = "PO has expired. Please raise a new PO and resubmit." and confirm  
**Then** (1) invoice status = `REJECTED`; (2) rejection reason stored verbatim; (3) audit trail entry with actor, timestamp, reason; (4) vendor email sent within 5 min containing invoice number, reason, and portal deeplink; (5) vendor's portal view shows the rejection reason.

**AC-VIMP-005b — Rejection Reason Is Mandatory (→ BR-VIMP-004)**  
**Given** AP User clicks Reject and leaves the reason field empty `★`  
**When** they attempt to confirm  
**Then** modal does not close. Inline error: "A rejection reason is required." No status change. No DB write.

**AC-VIMP-005c — Rejection Reason at Minimum Length**  
**Given** AP User enters rejection reason = "X" `★` (1 character)  
**When** they confirm  
**Then** rejection accepted. Reason stored as "X". (1-character minimum is valid.)

**AC-VIMP-005d — Rejection Reason at Maximum Length**  
**Given** AP User enters rejection reason = 500 characters `★` (exact limit)  
**When** they confirm  
**Then** rejection accepted. All 500 characters stored and displayed.

**AC-VIMP-005e — Rejection Reason Exceeds Maximum Length**  
**Given** AP User enters rejection reason = 501 characters `★`  
**When** they type the 501st character  
**Then** character input is blocked at 500 (field enforces `maxlength = 500`). Character counter shows "500/500". No truncation on save (what is entered is the max allowed).

**AC-VIMP-005f — XSS in Rejection Reason**  
**Given** AP User enters rejection reason = `<img src=x onerror=alert(1)>`  
**When** saved and displayed to vendor  
**Then** the string is HTML-encoded and displayed as literal text — no script executes on vendor's browser or AP team view.

**AC-VIMP-005g — Reject Already-Rejected Invoice**  
**Given** invoice "INV-2024-0510" is already in `REJECTED` status  
**When** AP User attempts to reject it again  
**Then** error: "This invoice has already been rejected." Status unchanged. No duplicate audit entry.

**AC-VIMP-005h — Reject a Paid Invoice**  
**Given** invoice "INV-2024-0511" is in `PAYMENT_INITIATED` or `PAYMENT_COMPLETED` status  
**When** AP User attempts to reject it  
**Then** error: "Invoices with payment in progress or completed cannot be rejected." Reject button is disabled in UI for these statuses.

---

### AC-VIMP-006 — Payment Forwarding (→ US-VIMP-013)

**AC-VIMP-006a — Automatic Forwarding on Approval**  
**Given** invoice "INV-2024-0501" is approved  
**When** the approval is committed to DB  
**Then** within 60 seconds: (1) payment system API called with payload `{invoice_id, vendor_bank_account, amount, currency, po_ref}`; (2) invoice status = `PAYMENT_INITIATED`; (3) AP Admin dashboard reflects new status; (4) outbound API call logged with request ID and timestamp.

**AC-VIMP-006b — Payment System Unavailable (Retry Queue)**  
**Given** payment system endpoint returns HTTP 503  
**When** forwarding is first attempted for approved invoice "INV-2024-0521"  
**Then** (1) invoice status = `PENDING_PAYMENT_FORWARDING`; (2) retry scheduled at T+5 min, T+15 min, T+60 min `★`; (3) after 3 failed retries, AP Admin receives email: "Invoice INV-2024-0521 could not be forwarded for payment after 3 attempts. Manual action required."; (4) invoice flagged in AP Admin dashboard under "Payment Failures."

**AC-VIMP-006c — Payment System Returns Success**  
**Given** payment system returns HTTP 200 with `{payment_ref: "PAY-88001", scheduled_date: "2026-06-10"}`  
**When** response received  
**Then** invoice status = `PAYMENT_INITIATED`; payment reference "PAY-88001" stored against the invoice; AP Admin and Finance User can see payment ref; vendor sees status update.

**AC-VIMP-006d — Duplicate Forwarding Prevention**  
**Given** invoice "INV-2024-0501" is already in `PAYMENT_INITIATED` status  
**When** a forwarding event is accidentally triggered again (e.g., retry after transient error on a call that actually succeeded)  
**Then** system detects `PAYMENT_INITIATED` status and aborts the second forwarding call. No duplicate payment request sent. Idempotency key used per forwarding call.

**AC-VIMP-006e — Missing Vendor Bank Details**  
**Given** vendor's bank account details are absent from their profile  
**When** an invoice from this vendor is approved and forwarding is attempted  
**Then** forwarding is blocked; invoice status = `PENDING_PAYMENT_FORWARDING`; AP Admin receives alert: "Invoice INV-2024-XXXX: vendor bank details missing. Please update vendor profile before payment can proceed."

---

### AC-VIMP-007 — Email Notifications (→ US-VIMP-015, US-VIMP-016)

**AC-VIMP-007a — Notification: Invoice Submitted → AP Team**  
**Given** vendor submits invoice "INV-2024-0601"  
**When** status transitions to `SUBMITTED`  
**Then** email sent to AP team distribution address within 5 minutes containing: subject "New Invoice Submitted: INV-2024-0601 | TestCo", body includes vendor name, invoice number, PO number, amount, and direct link to the invoice in the AP portal.

**AC-VIMP-007b — Notification: Invoice Approved → Vendor**  
**Given** AP User approves invoice "INV-2024-0601"  
**When** status transitions to `APPROVED`  
**Then** email sent to vendor's registered email within 5 minutes: subject "Invoice INV-2024-0601 Approved", body: invoice number, approved amount, expected payment timeline (if configured), portal link.

**AC-VIMP-007c — Notification: Invoice Rejected → Vendor with Reason**  
**Given** AP User rejects invoice "INV-2024-0602" with reason "PO has expired"  
**When** status transitions to `REJECTED`  
**Then** email to vendor within 5 minutes: subject "Invoice INV-2024-0602 Rejected", body: invoice number, rejection reason "PO has expired", instructions to resubmit, portal deeplink to the specific invoice.

**AC-VIMP-007d — Notification: Payment Initiated → Vendor**  
**Given** invoice "INV-2024-0601" forwarding succeeds  
**When** status transitions to `PAYMENT_INITIATED`  
**Then** email to vendor: subject "Payment Initiated for Invoice INV-2024-0601", body: invoice number, amount, payment reference, estimated payment date (if available from payment system).

**AC-VIMP-007e — Invalid Vendor Email Address**  
**Given** vendor's registered email "bad_email@" is malformed (data integrity failure)  
**When** a notification is triggered  
**Then** email dispatch fails gracefully; error logged: "Notification delivery failed for invoice INV-XXXX: invalid recipient address"; AP Admin receives alert; no system crash or unhandled exception.

**AC-VIMP-007f — No Duplicate Notifications**  
**Given** invoice "INV-2024-0601" transitions to `SUBMITTED`  
**When** the status change event fires  
**Then** exactly one email is sent to each intended recipient. No duplicate emails for the same event on the same invoice.

**AC-VIMP-007g — Notification Contains Valid Portal Deeplink**  
**Given** a rejection email is sent for invoice "INV-2024-0602"  
**When** vendor clicks the portal link in the email  
**Then** they are taken directly to invoice "INV-2024-0602" after authentication (or to login if unauthenticated, with redirect back to the invoice after login). The link does not lead to a 404 or generic dashboard.

---

### AC-VIMP-008 — Monthly Report (→ US-VIMP-017)

**AC-VIMP-008a — Report Generation: Standard Month**  
**Given** the month of May 2026 has activity: 120 invoices submitted, 95 approved, 18 rejected, 7 pending, total payment value = ₹12,450,000  
**When** the report job runs on the first business day of June 2026  
**Then** report contains: submitted = 120, approved = 95, rejected = 18, pending = 7, total_payment_value = ₹12,450,000. Report stored with label "May 2026". Available to AP Admin and Finance User by 09:00 local time.

**AC-VIMP-008b — Report Boundary: December/January Date Edge**  
**Given** invoices submitted on 2026-12-31 23:59:59 `★` are in scope for the December report  
**And** invoices submitted on 2027-01-01 00:00:00 `★` are in scope for the January report  
**When** December and January reports are generated  
**Then** the Dec 31 invoice appears only in December report; the Jan 1 invoice appears only in January report. No cross-month contamination.

**AC-VIMP-008c — Report for Month with Zero Invoices**  
**Given** no invoices were submitted or actioned in February 2026  
**When** the March report job runs  
**Then** report is generated with all count fields = 0 and total_payment_value = ₹0. Report is available (no "no data" error). This is a valid, exportable report.

**AC-VIMP-008d — Access by Unauthorised Role (Vendor)**  
**Given** a Vendor user is authenticated  
**When** they attempt to access the `/reports` section (UI or API)  
**Then** HTTP 403 returned. Report data not rendered. No report content leaked in error response.

**AC-VIMP-008e — PDF Download: Valid File**  
**Given** AP Admin downloads the May 2026 report as PDF  
**When** download completes  
**Then** (1) file is a valid, readable PDF; (2) file size > 0 bytes; (3) PDF contains the expected metrics (spot-check: submitted count, total value); (4) PDF is not password-protected unless specified.

**AC-VIMP-008f — XLSX Download: Valid File**  
**Given** AP Admin downloads the May 2026 report as XLSX  
**When** download completes  
**Then** (1) file opens in Excel without errors; (2) data in correct columns; (3) numeric values are stored as numbers (not text) enabling SUM formulas to work.

---

### AC-VIMP-009 — Role-Based Access Control (→ US-VIMP-018, US-VIMP-019)

**AC-VIMP-009a — Vendor Cannot Access AP Approval Queue**  
**Given** Vendor "vendor_active@testco.com" is authenticated  
**When** they attempt GET `/api/invoices/approval-queue`  
**Then** HTTP 403. Response body: `{"error": "Access denied"}`. No invoice records returned. No AP data in response.

**AC-VIMP-009b — AP User Cannot Access System Admin Panel**  
**Given** AP User "ap_user1@company.com" is authenticated  
**When** they attempt GET `/admin/users`  
**Then** HTTP 403. No user management data returned.

**AC-VIMP-009c — Finance User Cannot Approve Invoices**  
**Given** Finance User "finance1@company.com" is authenticated  
**When** they attempt POST `/api/invoices/INV-2024-0501/approve`  
**Then** HTTP 403. Invoice status unchanged at DB level.

**AC-VIMP-009d — Vendor Cannot View Another Vendor's Invoice (→ BR-VIMP-007)**  
**Given** Vendor A is authenticated; invoice "INV-2024-0700" belongs to Vendor B  
**When** Vendor A requests GET `/api/invoices/INV-2024-0700`  
**Then** HTTP 403 (or 404 — no information disclosure). Invoice data not returned. Vendor A cannot infer that invoice INV-2024-0700 exists.

**AC-VIMP-009e — Unauthenticated Request to Protected Endpoint**  
**Given** no session token present  
**When** any protected endpoint is called (e.g., GET `/api/invoices`)  
**Then** HTTP 401. Redirect to `/login`. No protected data returned.

**AC-VIMP-009f — Tampered JWT Token**  
**Given** user has a valid JWT and manually modifies the payload to escalate role from "VENDOR" to "AP_ADMIN"  
**When** they send the tampered token in Authorization header  
**Then** HTTP 401: "Invalid authentication token." Token signature validation fails. No AP Admin privileges granted.

**AC-VIMP-009g — Expired JWT Used**  
**Given** a JWT token that expired 1 minute ago `★`  
**When** used in an API request  
**Then** HTTP 401: "Session expired. Please log in again." User redirected to `/login`.

**AC-VIMP-009h — CSRF Attack on Approval Endpoint**  
**Given** an attacker crafts a cross-site POST to `/api/invoices/INV-2024-0501/approve` from a different origin  
**When** the request is made  
**Then** CSRF token validation fails; HTTP 403 returned. Invoice not approved. Anti-CSRF protection enforced on all state-changing endpoints.

**AC-VIMP-009i — Account Deactivated → Session Invalidated Immediately (→ BR-VIMP-010)**  
**Given** AP User "ap_user3@company.com" is actively logged in  
**When** System Admin deactivates "ap_user3@company.com"  
**Then** all existing sessions for "ap_user3@company.com" are invalidated immediately (within 1 request cycle). "ap_user3"'s next request returns HTTP 401. They cannot complete any in-flight actions post-deactivation.

**AC-VIMP-009j — Role Downgrade Takes Immediate Effect**  
**Given** user "ap_admin2@company.com" has role AP_ADMIN and is logged in  
**When** System Admin changes role to AP_USER  
**Then** on the user's next request, AP_ADMIN-only features (user management, full report access) return HTTP 403. User's existing dashboard may show a permission error for Admin sections.

---

## 7. Data Contract

This section defines what data VIMP consumes and produces at its system boundaries. Implementation details (schemas, endpoints) are deferred to the TRD.

### 7.1 Inbound

| Source | Data | Used By |
|--------|------|---------|
| Vendor Registration Form | Company name, contact name, email, password, tax ID | US-VIMP-001 |
| Invoice Submission Form | Invoice number, PO number, invoice date, line items, total amount, currency, attachments | US-VIMP-005 |
| AP Approval Action | Approver user ID, timestamp, approval status | US-VIMP-010, US-VIMP-011 |
| Payment System | Payment reference number, payment date, payment status | US-VIMP-014 |

### 7.2 Outbound

| Destination | Data | Triggered By |
|-------------|------|-------------|
| Payment Processing System | Invoice ID, vendor bank details, approved amount, PO reference | US-VIMP-013 (on approval) |
| Email Service | Recipient address, status, invoice number, action URL | US-VIMP-015, US-VIMP-016 |
| Report Storage | Monthly invoice summary (counts, values, statuses) | US-VIMP-017 |

---

## 8. Open Questions

The following items are unresolved and must be answered by the client before test case design is finalised.

| OQ ID | Question | Affects |
|-------|----------|---------|
| OQ-01 | Is vendor registration self-service or invitation-only? | US-VIMP-001, AC-VIMP-001a |
| OQ-02 | What is the session timeout duration for inactive sessions? | US-VIMP-004 |
| OQ-03 | What file formats are accepted for invoice attachments? What is the size limit? | US-VIMP-006, AC-VIMP-003d/e |
| OQ-04 | Can one invoice reference multiple POs? | US-VIMP-005, BR-VIMP-001 |
| OQ-05 | Is multi-level approval required (reviewer → manager), or is single AP User approval sufficient? | US-VIMP-010 |
| OQ-06 | What is the SLA for invoice review? What happens on breach — escalation, reminder, or auto-escalation? | US-VIMP-009 |
| OQ-07 | Which payment system does VIMP integrate with? (SAP, Oracle, internal, bank API?) | US-VIMP-013 |
| OQ-08 | Is payment forwarding always automatic, or does Finance need to confirm batch? | US-VIMP-013, BR-VIMP-006 |
| OQ-09 | Are there predefined rejection reasons or is free-text the only option? | US-VIMP-011, AC-VIMP-005a |
| OQ-10 | Can a vendor resubmit a rejected invoice, and is there a limit on resubmission attempts? | US-VIMP-008 |
| OQ-11 | Which statuses trigger email notifications and to which role(s)? | US-VIMP-015, US-VIMP-016 |
| OQ-12 | What is the exact content and schedule for the monthly report? Who receives it automatically? | US-VIMP-017 |
| OQ-13 | Should the system support SSO/SAML for internal users? | US-VIMP-003 |
| OQ-14 | Is there a regulatory or compliance requirement (GDPR, SOC 2, ISO 27001) that governs data retention? | US-VIMP-012, BR-VIMP-009 |

---

## 9. Approval

*This document requires two sign-offs before QA test planning may proceed.*

```
Approved by:  ___________________________
Role:         ___________________________
Date:         ___________________________

Approved by:  ___________________________
Role:         ___________________________
Date:         ___________________________
```

---

*Document version: 1.0 | Status: Draft — Pending Client Review*

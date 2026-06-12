# Requirements Clarification Document
## Vendor Invoice Management Portal (VIMP)

**Document Type:** Requirements Analysis — Ambiguity Identification & Clarifying Questions  
**Prepared by:** QA Engineer  
**Date:** 2026-06-06  
**Version:** 3.0 — Merged & Consolidated (supersedes v1 `02_requirements_clarification.md`)  
**Source PRD:** `02_prd.md` (User Stories: US-VIMP-001 to US-VIMP-019 | Business Rules: BR-VIMP-001 to BR-VIMP-011)

---

## Purpose

This document identifies ambiguous, incomplete, or conflicting items in the VIMP PRD (`02_prd.md`) that must be resolved before test planning begins. Every question is anchored to a specific **User Story (US-VIMP-NNN)**, **Business Rule (BR-VIMP-NNN)**, or **Open Question (OQ-NNN)** from the PRD so that answers can be traced back to the exact requirement they clarify.

> "The most valuable bug you will ever find is in the requirements." — QA Excellence, Slide 3

The five questions asked for every requirement are drawn from the QA Excellence framework (Slide 3):
1. What does success look like?
2. What happens when it fails?
3. Who are all the users affected?
4. What data is involved?
5. What are the boundaries / limits?

---

## REQ-01 — Vendor Registration & Login
**PRD References:** US-VIMP-001, US-VIMP-002, US-VIMP-003, US-VIMP-004 | OQ-01, OQ-02, OQ-13

### Ambiguities in the PRD

- **US-VIMP-001 / OQ-01:** Registration flow says "Active after email verification" but doesn't state whether AP Admin must also approve before a vendor can submit invoices. Two interpretations exist: (a) email verification alone activates the account, or (b) AP Admin approval is a second gate.
- **US-VIMP-002:** Password policy is undefined — no minimum length, complexity, or expiry rule is specified, making AC-VIMP-002c's lockout threshold (5 attempts) the only security parameter currently pinned.
- **US-VIMP-003 / OQ-13:** Internal users (AP, Finance, Admin) may use SSO. If SSO is implemented, the password-based login flow and lockout logic may not apply to those roles — creating two separate authentication paths with different security postures.
- **US-VIMP-004 / OQ-02:** Session timeout duration is unspecified. This directly affects test design for idle session, concurrent session, and session token expiry test cases.

### Clarifying Questions

**Registration (→ US-VIMP-001, AC-VIMP-001a, OQ-01)**
1. Is vendor registration fully self-service, or must a vendor be invited by an AP Admin before they can register?
2. After email verification, is the vendor account immediately Active and able to submit invoices — or does an AP Admin need to approve the account as a second step?
3. What mandatory fields are required at registration? Is tax ID (GSTIN / VAT number) required upfront, or can it be added later?
4. Can one company register multiple vendor accounts (e.g., different regional offices)?
5. How long is the email verification link valid before it expires? What happens if the vendor does not verify within that window?

**Authentication (→ US-VIMP-002, US-VIMP-003, AC-VIMP-002c, OQ-13)**
6. What is the password policy? (minimum length, complexity requirements, expiry period)
7. Is MFA (Multi-Factor Authentication) mandatory, optional, or not required for vendors? What about internal users?
8. For internal users (AP, Finance, Admin): is SSO/SAML integration required, or will they use portal-native username/password?
9. If SSO is used for internal users, do the lockout rules in AC-VIMP-002c still apply, or does the identity provider handle lockout?
10. Can a vendor account be locked out permanently, or only temporarily? Who can unlock a permanently locked account?

**Session Management (→ US-VIMP-004, OQ-02)**
11. What is the idle session timeout duration? (e.g., 15 min, 30 min, 60 min)
12. Should session timeout differ between vendor users and internal AP/Finance users?
13. Can a vendor be logged in on multiple devices simultaneously? If not, should the new login terminate the existing session?
14. If a session expires while a vendor has an unsaved invoice draft, should the draft be preserved for when they log back in?
15. Is there a "Remember Me" / stay-logged-in option for vendors? If yes, what is the maximum persistent session duration?
16. Can vendors reset their own passwords (self-service via email link), or must they contact support?
17. Can vendors update their registered profile information post-registration (bank details, contact name, tax ID)? If yes, does a profile change require re-verification?
18. Can a vendor deactivate their own account, or only a System Admin can?

---

## REQ-02 — Invoice Submission Against Purchase Orders
**PRD References:** US-VIMP-005, US-VIMP-006, US-VIMP-007, US-VIMP-008 | BR-VIMP-001, BR-VIMP-002, BR-VIMP-011 | OQ-03, OQ-04, OQ-10

### Ambiguities in the PRD

- **US-VIMP-005 / BR-VIMP-001:** The rule states an invoice must reference a "valid, open" PO. "Open" is undefined — does it mean the PO has remaining value (partial invoicing allowed), or simply that it hasn't been closed/cancelled?
- **US-VIMP-005 / OQ-04:** The PRD does not state whether a single invoice can span multiple POs. AC-VIMP-003b tests a single PO mismatch, but the multi-PO case has no acceptance criterion.
- **US-VIMP-006 / OQ-03:** File formats and size limits appear as open questions (OQ-03) and are referenced in AC-VIMP-003d/e as placeholders. These must be defined before test data and boundary tests can be designed.
- **US-VIMP-008:** Resubmission is mentioned but no acceptance criterion defines the resubmission flow — is it a brand-new submission or an edit of the original rejected invoice?
- **BR-VIMP-011:** Duplicate detection is defined on invoice number + PO number. What if the vendor uses the same invoice number against a different PO? Is that a duplicate or a valid new submission?

### Clarifying Questions

**PO Matching & Invoice Rules (→ US-VIMP-005, BR-VIMP-001, OQ-04)**
1. Does the vendor select a PO from a system-provided list, or manually type the PO number? If typed, how is it validated?
2. Can one invoice reference more than one PO? If yes, how is the submission form structured?
3. Can a vendor submit a partial invoice against a PO (e.g., invoice for 60% of the PO value because only partial delivery occurred)?
4. What makes a PO "open" vs. "closed"? Is it based on remaining value, expiry date, or a manual status set by the AP team?
5. Can a PO that has already been fully invoiced accept additional invoices (over-invoicing scenario)?

**Attachments (→ US-VIMP-006, OQ-03)**
6. What file formats are accepted for invoice and attachment uploads? (PDF only? JPEG, PNG, XLSX also?)
7. What is the maximum file size per attachment? Is there a limit on total upload size per submission?
8. What is the maximum number of attachments allowed per invoice?
9. Are attachments mandatory, or can an invoice be submitted without any attachment?

**Invoice Data & Validation (→ US-VIMP-005)**
10. What mandatory fields must every invoice contain? (Invoice number, date, amount, tax, currency, line items?)
11. What currencies are supported? Is invoicing in foreign currency allowed?
12. Is there a maximum number of line items per invoice?
13. What tax formats are expected? (GST, VAT, a flat percentage field?)

**Resubmission (→ US-VIMP-008, OQ-10)**
14. When resubmitting a rejected invoice, does the vendor create a new invoice record or edit the rejected one in place?
15. Is there a limit on how many times a vendor can resubmit the same invoice?
16. Does a resubmission reset the invoice number, or does it carry the same invoice number with a revision marker?

**Duplicate Detection (→ BR-VIMP-011)**
17. Is duplicate detection based on invoice number + PO number + vendor ID? Or just invoice number + PO number?
18. If the same invoice number is used against a different PO, is that treated as a duplicate or a valid new invoice?
19. Is there a limit on how many invoices a vendor can submit per day or per month? If yes, what happens when the limit is reached?
20. Can a vendor edit or withdraw a submitted invoice before the AP team begins reviewing it? If yes, does it return to DRAFT status?

---

## REQ-03 — AP Team Approval & Rejection
**PRD References:** US-VIMP-009, US-VIMP-010, US-VIMP-011, US-VIMP-012 | BR-VIMP-003, BR-VIMP-004 | OQ-05, OQ-06, OQ-09

### Ambiguities in the PRD

- **US-VIMP-010 / OQ-05:** AC-VIMP-004a assumes a single AP User can approve. But multi-level approval (reviewer + manager sign-off) is listed as an open question. If multi-level approval is required, AC-VIMP-004a is incomplete and a new story/AC is needed.
- **US-VIMP-009 / OQ-06:** No SLA for invoice review is defined. Without it, the system cannot generate escalation alerts or mark overdue invoices. This affects notification design (US-VIMP-016) and any AP dashboard priority sorting.
- **US-VIMP-011 / OQ-09:** Rejection reasons may be predefined, free-text, or both. This determines whether test data for AC-VIMP-005a uses a dropdown selection or typed text — and whether the field has a character limit.
- **BR-VIMP-003:** Segregation of duties is enforced when a user is "also the vendor contact." In a real B2B portal, this scenario is unlikely but must be defined clearly — the rule as written is ambiguous about what triggers it.

### Clarifying Questions

**Approval Workflow (→ US-VIMP-009, US-VIMP-010, OQ-05, OQ-06)**
1. Is a single AP User approval sufficient, or is multi-level approval required (e.g., AP Reviewer approves, then AP Manager confirms)?
2. If multi-level: can the same person fulfil both levels, or must they be different users?
3. Is there a defined SLA for invoice review? (e.g., must be approved or rejected within 5 business days)
4. What happens when an invoice exceeds the review SLA — automatic escalation email, in-portal alert, or no action?
5. Can an AP User request additional information from the vendor without formally rejecting the invoice? If yes, what status does the invoice enter?
6. Can an AP User who has approved an invoice reverse that approval? Under what conditions?
7. Can the AP team assign specific invoices to specific team members for review? If yes, can an unassigned AP User still action an invoice assigned to someone else?

**Rejection (→ US-VIMP-011, BR-VIMP-004, OQ-09)**
7. Are rejection reasons predefined (a dropdown list), free-text, or both?
8. If predefined, what are the standard rejection reason categories?
9. Is there a character limit on a free-text rejection reason?
10. Is the rejection reason visible to the vendor in the email notification and in the portal — or only in the portal?

**Audit Trail (→ US-VIMP-012)**
11. What actions must be captured in the invoice audit trail? (View, Submit, Approve, Reject, Resubmit, Payment Initiated?)
12. How long are audit trail records retained?
13. Is the audit trail exportable by the AP Admin? In what format?

**Segregation of Duties (→ BR-VIMP-003)**
14. The rule says a user cannot approve an invoice they submitted. In this portal, vendors submit and AP team approves — so who exactly does this rule target? An internal test vendor account? An AP Admin who also acts as a vendor?
15. Should the system enforce this at the database level, or is a UI-level disable (as in AC-VIMP-004b) sufficient?

---

## REQ-04 — Payment Forwarding
**PRD References:** US-VIMP-013, US-VIMP-014 | BR-VIMP-005, BR-VIMP-006 | OQ-07, OQ-08

### Ambiguities in the PRD

- **US-VIMP-013 / OQ-07:** The payment system destination is unspecified. The data contract in Section 7.2 of the PRD lists "vendor bank details" as outbound data, but it is unclear whether bank details are captured at registration or at invoice submission — and who validates them.
- **US-VIMP-013 / OQ-08:** BR-VIMP-006 says forwarding is automatic. However, many finance teams operate batch payment runs (e.g., every Wednesday). The PRD leaves open whether forwarding means "sent to payment queue immediately" or "included in next batch run."
- **AC-VIMP-006a:** The 60-second SLA for forwarding is assumed. This must be confirmed as a real business requirement, not a placeholder.
- **US-VIMP-014:** Payment status visibility is granted to Finance Users and AP Admins. Vendors are not mentioned — it's unclear if vendors should see "Payment Initiated" and "Payment Completed" on their invoice view.

### Clarifying Questions

**Payment System Integration (→ US-VIMP-013, OQ-07)**
1. Which payment processing system does VIMP integrate with? (SAP, Oracle, internal finance system, bank API?)
2. What data fields are required by the payment system for each invoice? (Invoice ID, vendor bank account number, sort code/IFSC, amount, currency, PO reference?)
3. Where are vendor bank details captured — at registration, on the invoice, or separately managed by the AP team?
4. Who validates vendor bank account details, and when? (At registration? Before first payment?)

**Forwarding Mechanism (→ BR-VIMP-006, OQ-08)**
5. Is payment forwarding real-time (immediately on approval) or batched (e.g., daily at 6 PM)?
6. If batched, when does the batch run? What is the cutoff time for same-day inclusion?
7. Is there a Finance User confirmation step before forwarding, or is it fully automatic as stated in BR-VIMP-006?

**Error Handling (→ AC-VIMP-006b)**
8. If the payment system is unavailable, what is the retry policy? (Retry every X minutes for Y hours?)
9. Who is notified if forwarding fails after all retries are exhausted — AP Admin only, or also Finance?
10. Can a forwarded invoice be recalled from the payment system if an error is discovered post-approval?

**Vendor Visibility (→ US-VIMP-014)**
11. Should vendors be able to see payment status (Payment Initiated, Payment Completed) on their invoice view?
12. Should vendors receive an email notification when payment is completed? (Relates to US-VIMP-015 and OQ-11)

---

## REQ-05 — Email Notifications
**PRD References:** US-VIMP-015, US-VIMP-016 | AC-VIMP-007a, AC-VIMP-007b | OQ-11

### Ambiguities in the PRD

- **AC-VIMP-007a / OQ-11:** The "appropriate party" for each status transition is undefined. Without a status-to-recipient matrix, it is impossible to design notification test cases — the same status change may need to notify vendor only, AP only, or both.
- **US-VIMP-015:** The notification content is partially defined in AC-VIMP-007b for rejection. Other status change emails (Approved, Payment Initiated, Payment Completed) have no content specification.
- No mention of notification failure handling — what happens if the email service is unavailable or the email bounces.

### Clarifying Questions

**Recipients & Triggers (→ US-VIMP-015, US-VIMP-016, OQ-11)**
1. For each status transition below, who receives the notification — Vendor, AP User, AP Admin, Finance User, or a combination?

   | Status Change | Who is notified? |
   |---|---|
   | Invoice Submitted | ? |
   | Invoice Under Review | ? |
   | Invoice Approved | ? |
   | Invoice Rejected | ? |
   | Payment Initiated | ? |
   | Payment Completed | ? |

2. When a vendor has multiple registered contacts (e.g., finance contact + account manager), who receives the notification — all contacts, primary only, or configurable?

**Notification Content (→ AC-VIMP-007b)**
3. What must each notification email include? At minimum: invoice number, status, timestamp, and a portal link?
4. For the rejection email specifically: must the full rejection reason text be included, or only a summary?
5. Should there be a standard email template for all statuses, or custom content per status?

**Delivery & Reliability**
6. What is the maximum acceptable delay between a status change and the corresponding email being sent?
7. If email delivery fails (bounce, service outage), should the system: (a) retry, (b) alert an admin, (c) show an in-portal notification, or a combination?
8. Should sent notification history be stored and viewable by AP Admin for audit purposes?

**Opt-out**
9. Can vendors or AP users unsubscribe from specific notification types? If yes, which ones are mandatory (cannot be opted out)?

---

## REQ-06 — Monthly Invoice Activity Reports
**PRD References:** US-VIMP-017 | BR-VIMP-008 | AC-VIMP-008a, AC-VIMP-008b | OQ-12

### Ambiguities in the PRD

- **US-VIMP-017 / OQ-12:** Report content is listed at a high level (submitted, approved, rejected, pending, payment value). Whether line-item detail, vendor-wise breakdown, or aging analysis is included is unresolved.
- **AC-VIMP-008a:** "First business day of the calendar month" as the generation trigger is assumed — this needs client confirmation. If the report must be available by 9 AM on the 1st, a specific scheduled job time is needed.
- **AC-VIMP-008b:** On-demand report generation for custom date ranges is included, but whether this is a full product requirement or just a nice-to-have is unconfirmed.

### Clarifying Questions

**Report Content (→ US-VIMP-017, OQ-12)**
1. What summary metrics must the monthly report include? (Total submitted, approved, rejected, pending, payment value processed, average approval time?)
2. Should the report include a breakdown by vendor? By PO? By AP approver?
3. Should overdue/aging invoices (pending beyond X days) be flagged in the report?
4. Should rejected invoices appear with their rejection reasons?

**Access & Distribution (→ US-VIMP-017)**
5. Which roles can access the reports section — AP Admin only, Finance User also, or all roles?
6. Is the monthly report automatically emailed to specific stakeholders, or is it only available within the portal?
7. Who configures the distribution list for automatic email delivery?

**Format & Scheduling (→ AC-VIMP-008a, AC-VIMP-008b)**
8. When exactly is the monthly report generated — midnight on the 1st, first business day morning, or manually triggered by AP Admin?
9. What formats must the report be available in — PDF, XLSX, CSV, or all?
10. Can users generate on-demand reports for custom date ranges, or is the report strictly calendar-month only?
11. How long are historical reports retained in the system?

---

## REQ-07 — Authorised User Access Only
**PRD References:** US-VIMP-018, US-VIMP-019 | BR-VIMP-009, BR-VIMP-010 | AC-VIMP-009a, AC-VIMP-009b | OQ-13, OQ-14

### Ambiguities in the PRD

- **US-VIMP-018 / BR-VIMP-009:** Five roles are defined in Section 3, but the permission matrix (which role can do what) is not documented in the PRD. Without it, AC-VIMP-009a/b test only two access-denied cases and do not cover the full RBAC surface.
- **US-VIMP-019 / BR-VIMP-010:** "Immediate session invalidation on deactivation" is stated in BR-VIMP-010. How the system detects and enforces this mid-session (e.g., token revocation, next-request check) needs to be defined for test design.
- **OQ-14:** No compliance framework is named. If GDPR or SOC 2 applies, data retention rules for audit logs, personal data, and invoice records have legal minimums that override whatever the client states.

### Clarifying Questions

**Role Permissions (→ US-VIMP-018, BR-VIMP-009)**
1. Please confirm the full permission matrix for all 5 roles (R-01 through R-05):

   | Action | Vendor | AP User | AP Admin | Finance User | System Admin |
   |--------|--------|---------|----------|--------------|--------------|
   | Register | ✓ | — | — | — | — |
   | Submit Invoice | ? | ? | ? | ? | ? |
   | View Own Invoices | ? | ? | ? | ? | ? |
   | View All Invoices | ? | ? | ? | ? | ? |
   | Approve Invoice | ? | ? | ? | ? | ? |
   | Reject Invoice | ? | ? | ? | ? | ? |
   | View Reports | ? | ? | ? | ? | ? |
   | Manage Users | ? | ? | ? | ? | ? |

2. Can a user hold multiple roles simultaneously? (e.g., AP Admin who is also a Finance User?)
3. Is there a Super Admin role that can impersonate other users for support purposes?

**User Provisioning (→ US-VIMP-019)**
4. Who creates accounts for AP Users, AP Admins, and Finance Users — only the System Admin, or can an AP Admin also create AP User accounts?
5. When a user is deactivated, what happens to invoices that are assigned to them for review and not yet actioned?
6. Is there an account inactivity policy? (e.g., accounts not used in 90 days are auto-deactivated?)

**Access Control (→ US-VIMP-018, OQ-13)**
7. Is portal access restricted to specific IP ranges or VPN, or is it accessible from the open internet?
8. Is SSO/SAML required for internal users? If yes, which identity provider? (Azure AD, Okta, Google Workspace?)
9. Should the system enforce row-level security — i.e., vendors can only see their own invoices? (This is stated in BR-VIMP-007 but needs implementation confirmation.)

**Compliance & Audit (→ US-VIMP-012, OQ-14)**
10. What compliance framework governs this system? (GDPR, SOC 2, ISO 27001, local regulations?)
11. What is the data retention policy for: invoice records, audit trail logs, personal data (vendor contact details)?
12. Must audit logs be tamper-proof (write-once)? Is there a requirement for log integrity verification?

---

## Summary: Open Questions Mapped to PRD

The table below consolidates all Open Questions from the PRD (`02_prd.md`, Section 8) with the clarifying questions raised in this document.

| OQ ID | Question | Clarification Section | Priority |
|-------|----------|-----------------------|----------|
| OQ-01 | Self-service vs. invitation-only registration | REQ-01, Q1–Q2 | High |
| OQ-02 | Session timeout duration | REQ-01, Q11–Q13 | Medium |
| OQ-03 | Accepted file formats and size limits | REQ-02, Q6–Q8 | High |
| OQ-04 | Multi-PO invoicing | REQ-02, Q2 | High |
| OQ-05 | Single vs. multi-level approval | REQ-03, Q1–Q2 | High |
| OQ-06 | Review SLA and escalation policy | REQ-03, Q3–Q4 | High |
| OQ-07 | Payment system identity | REQ-04, Q1–Q3 | High |
| OQ-08 | Real-time vs. batch payment forwarding | REQ-04, Q5–Q6 | High |
| OQ-09 | Predefined vs. free-text rejection reasons | REQ-03, Q7–Q9 | Medium |
| OQ-10 | Resubmission limits and flow | REQ-02, Q14–Q16 | Medium |
| OQ-11 | Status-to-recipient notification matrix | REQ-05, Q1 | High |
| OQ-12 | Monthly report content and schedule | REQ-06, Q1–Q4, Q8 | Medium |
| OQ-13 | SSO/SAML for internal users | REQ-01, Q8 / REQ-07, Q8 | High |
| OQ-14 | Compliance framework and data retention | REQ-07, Q10–Q12 | High |

---

## Next Steps

Once the client provides answers to the questions above:

1. Update PRD (`02_prd.md`) — fill Open Questions section, revise affected ACs
2. Proceed to **Test Plan** (`03_test_plan.md`) — scope, risk, approach, entry/exit criteria
3. Build **Test Scenarios & Test Cases** (`04_test_cases.md`) — mapped to US-VIMP-NNN and AC-VIMP-NNN
4. Establish **RTM** (`05_rtm.md`) — linking PRD stories to test cases

---

*Document version: 2.0 | Status: Draft — Awaiting Client Clarification*  
*Supersedes: `02_requirements_clarification.md` (v1.0 — pre-PRD, now archived)*

# API Reference

[← Back to index](../README.md)

**174 REST endpoints** exposed by the Django backend, grouped by module.

All routes except those marked **public** require an `Authorization: Bearer <JWT>` header.
Public routes are explicitly whitelisted in the authentication middleware; everything else is
rejected before reaching a view.

---

## Authentication

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `api/admin/login/` | Administrator / HR login — **public** |
| POST | `api/send-otp/` | Send employee login OTP — **public** |
| POST | `api/verify-otp/` | Verify OTP, issue token — **public** |
| POST | `api/employee-by-phone/` | Resolve employee by mobile number |
| POST | `api/create-admin/` | Create an administrator account |
| POST | `update-password/` | Change password |
| GET | `get-csrf-token/` | CSRF token for cookie-bearing requests |

---

## Recruitment

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `create-candidate/` | Create a candidate |
| GET | `get-candidates/` | List candidates |
| POST | `submit/` | Submit an application |
| GET | `applications/` | List applications |
| GET/PUT | `applications/<pk>/` | Application detail |
| GET | `interviewstatus/<application_name>/` | Interview status |
| POST | `progressstatus/send-round-email/<application_name>/` | Send round email |
| POST | `send-invitation/` | Interview invitation |
| POST | `interview-email/` | Interview correspondence |
| POST | `final-status/<application_name>/` | Final decision |
| POST | `schedule-interview/` | Schedule an interview |
| POST | `create-teams-meeting/` | Create a Teams meeting |

---

## Onboarding

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `api/onboarding/sign/<sign_token>/` | Open signing session — **public (token)** |
| GET | `api/onboarding/sign/<sign_token>/pages/` | Document pages — **public (token)** |
| POST | `api/onboarding/sign/<sign_token>/submit/` | Submit signature — **public (token)** |
| GET | `api/onboarding/upload/<token>/` | Open upload session — **public (token)** |
| POST | `api/onboarding/upload/<token>/submit/` | Submit documents — **public (token)** |
| GET | `api/onboarding/` | Onboarding dashboard |
| GET | `api/onboarding/<employee_no>/` | Employee onboarding record |
| POST | `api/onboarding/<employee_no>/send/` | Send onboarding link |
| GET | `api/onboarding/candidate/<application_id>/` | Candidate onboarding |
| POST | `api/onboarding/candidate/<application_id>/send/` | Send candidate link |
| GET | `api/onboarding/candidate/<application_id>/preview/` | Preview submission |
| POST | `api/onboarding/candidate/<application_id>/joined/` | Mark joined |
| POST | `api/onboarding/candidate/<application_id>/dropped/` | Mark dropped |
| GET | `employee/form/<token>/` | Joining form — **public (token)** |
| POST | `employee-form/<token>/` | Submit joining form — **public (token)** |
| POST | `upload-document/<employee_no>/` | Upload employee document |

---

## Employee management

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `create-employee/` | Create employee |
| GET | `employees/` | List employees (branch-scoped) |
| GET | `employee/<employee_no>/` | Employee detail |
| GET | `getSingleEmployee/<id>/` | Employee by id |
| GET | `api/employee/<employee_id>/` | Employee record |
| DELETE | `delete-employee/<id>/` | Remove employee |
| GET | `resigned-employees/` | Resigned employees |
| POST | `reactivate-employee/<id>/` | Reactivate a rehire |
| POST | `update-photo/` | Update profile photo |
| GET | `mypage/` | Current user's page |
| POST | `upload_employee_data/` | Bulk employee import |
| POST | `upload_employee_data_ui/` | Bulk import (UI) |
| POST | `api/bulk-employee-master-upload/` | Employee master bulk upload |
| GET | `api/export-employee-master-data/` | Export employee master |
| GET | `employees-without-salary/` | Employees lacking salary structure |
| POST | `api/confirm-employees/` | Confirm employees |
| GET | `template/` | Import template |

---

## Salary & payroll

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `api/calculate-salary/` | Calculate one employee |
| POST | `api/bulk-calculate-salary/` | Calculate in bulk |
| POST | `upload_employee_salary/` | Salary import |
| POST | `upload_employee_salary_ui/` | Salary import (UI) |
| POST | `api/bulk-salary-update/` | Bulk salary update |
| GET | `api/bulk-salary-template/` | Bulk salary template |
| GET | `api/salary-filter-options/` | Filter options |
| GET | `api/export-employee-salary/` | Export salary data |
| GET | `employee/<employee_id>/ctc-revisions/` | Revision history |
| DELETE | `api/delete-ctc-revision/<id>/` | Delete a revision |
| GET | `api/detailed-salary-report/` | Detailed salary report |
| GET | `api/detailed-salary-report/export/` | Export report |
| POST | `updateloans/` | Update loans / advances |

---

## Payslips

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `api/generate-payslips/` | Generate payslips |
| POST | `api/import-salary-payslips/` | Import salary payslips |
| POST | `api/generate-payslip-pdfs/` | Generate PDFs (async) |
| POST | `api/publish-payslips/` | Publish to employees |
| POST | `api/email-payslips/` | Bulk email delivery |
| GET | `generate-payslip-pdf/<payslip_id>/` | Single PDF |
| GET | `generate-payslip-emp/<employee_id>/` | Generate for employee |
| GET | `view-payslip/<payslip_id>/` | View payslip |
| GET | `get-all-payslips/` | All payslips |
| GET | `employee/<employee_no>/payslips/` | Employee payslips |
| GET | `api/my-payslips/<employee_no>/` | Own payslips (employee) |
| DELETE | `delete-pdf-file/<payslip_id>/` | Delete PDF |

---

## Attendance

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `api/attendance/daily-status/` | Daily status |
| POST | `api/attendance/upload/` | Upload biometric data |
| GET | `api/attendance/template/` | Import template |
| GET | `api/attendance/export/` | Export attendance |
| GET | `api/attendance/monthly/` | Monthly grid |
| GET | `api/attendance/monthly-export/` | Export monthly |
| GET | `api/attendance/cycle-summary/` | 26→25 cycle summary |
| GET | `api/attendance/employee/<employee_no>/` | Employee attendance |
| GET | `api/attendance/employee/<employee_no>/cycle/` | Employee cycle |
| GET | `api/attendance/employee/<employee_no>/year-summary/` | Year summary |
| PUT | `api/attendance/<attendance_id>/` | Correct a record |
| POST | `api/update-workdays/` | Update workdays |
| GET | `api/attendance/lop-template/` | LOP template |
| GET/POST | `api/lop-uploads/` | LOP uploads |
| GET | `api/lop-uploads/<pk>/` | LOP upload detail |
| GET | `api/lop-upload-report/` | LOP report |

---

## Leave

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `leaverequest/employee/` | Apply for leave |
| GET | `getleaverequest/employee/<employee_no>/` | Own requests |
| GET | `leaverequest/cc/<employee_no>/` | CC'd requests |
| GET | `getleaverequest/recipient/<recipient_no>/` | Requests to action |
| GET | `adminleaverequest/` | All requests (admin) |
| PUT | `updateleaverequest/<leave_request_id>/` | Update request |
| POST | `respondleaverequest/<leave_request_id>/` | Approve / reject |
| POST | `withdrawleaverequest/<leave_request_id>/` | Withdraw |
| GET | `api/team/members/` | Team members |
| GET | `leave-summary-report/` | Leave summary |
| GET | `api/export-leave-summary/` | Export summary |
| GET | `api/leave-import-template/` | Balance import template |
| POST | `api/bulk-update-leaves/` | Bulk balance update |
| POST | `api/employees/<employee_no>/update-leaves/` | Update one employee |

---

## Tax calculator

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `api/tax-calculator/autofill/<employee_no>/` | Pre-fill from salary master |
| POST | `api/tax-calculator/calculate/` | Compute both regimes |

---

## Tax declarations

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `api/declarations/<employee_no>/<fy_id>/` | Get declaration |
| POST | `api/declarations/<employee_no>/<fy_id>/submit/` | Submit declaration |
| GET/POST | `api/declarations/proofs/` | Proof documents |
| GET | `api/declarations/proofs/<proof_id>/download/` | Download proof |
| DELETE | `api/declarations/proofs/<proof_id>/` | Delete proof |
| GET | `api/declarations/admin/` | Admin review queue |
| POST | `api/declarations/<decl_id>/approve/` | Approve |
| POST | `api/declarations/<decl_id>/reject/` | Reject with reason |

---

## Form 16

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `api/financial-years/` | Financial years |
| GET/POST | `api/employers/` | Employer (deductor) records |
| GET/PUT | `api/employers/<pk>/` | Employer detail |
| GET | `api/form16/admin/` | Admin certificate list |
| GET | `api/form16/admin/<pk>/` | Certificate detail |
| GET | `api/form16/employee/<employee_no>/` | Employee certificates |
| GET | `api/form16/employee/<employee_no>/<pk>/` | Certificate detail |
| POST | `api/form16/bulk-generate/` | Bulk generate |
| POST | `api/form16/<employee_no>/<fy_id>/generate/` | Generate one |
| GET | `api/form16/<form16_id>/part-b/` | Part B data |
| GET | `api/form16/<form16_id>/reconciliation/` | Reconciliation diff |
| POST | `api/form16/<form16_id>/publish/` | Publish |
| POST | `api/form16/<form16_id>/revoke/` | Revoke |
| POST | `api/form16/<form16_id>/refresh/` | Recompute |
| POST | `api/form16/<pk>/<action>/` | Generic action |
| GET | `api/form16/<form16_id>/pdf/part-a/` | Part A PDF |
| GET | `api/form16/<form16_id>/pdf/part-b/` | Part B PDF |
| GET | `api/form16/<form16_id>/pdf/combined/` | Combined PDF |
| POST | `api/form16/salary-entries/import/` | Import salary entries |
| GET/POST | `api/form16/annexure/deposits/` | TDS deposits |
| GET/PUT | `api/form16/annexure/deposits/<pk>/` | Deposit detail |
| GET/POST | `api/form16/annexure/challans/` | TDS challans |
| GET/PUT | `api/form16/annexure/challans/<pk>/` | Challan detail |
| GET/POST | `api/form16/rectifications/` | Rectification requests |
| GET/PUT | `api/form16/rectifications/<pk>/` | Rectification detail |

---

## Master data

| Method | Endpoint | Purpose |
|---|---|---|
| GET/POST | `api/managers/` · `api/managers/<pk>/` | Reporting managers |
| GET/POST | `api/branches/` · `api/branches/<pk>/` | Branches |
| GET/POST | `api/locations/` · `api/locations/<pk>/` | Locations |
| GET/POST | `api/hr-contacts/` · `api/hr-contacts/<pk>/` | HR contacts (accounts) |
| GET/POST | `api/team-heads/` · `api/team-heads/<pk>/` | Team heads |
| GET/POST | `api/assets/` · `api/assets/<pk>/` | Assets |
| GET | `api/employee/<employee_no>/assets/` | Assets by employee |

---

## Support

| Method | Endpoint | Purpose |
|---|---|---|
| GET/POST | `api/support/tickets/` | Employee tickets |
| GET | `api/support/tickets/<ticket_id>/` | Ticket detail |
| GET/POST | `api/support/tickets/<ticket_id>/messages/` | Ticket thread |
| POST | `api/support/tickets/<ticket_id>/mark-read/` | Mark read |
| GET | `api/support/admin/tickets/` | Admin queue |
| PUT | `api/support/admin/tickets/<ticket_id>/status/` | Update status |
| GET | `api/support/hr-options/` | HR routing options |
| GET | `api/support/ai-handoffs/` | AI assistant handoffs |

---

## WhatsApp

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `api/whatsapp/contacts/` | Contacts |
| GET | `api/whatsapp/messages/<phone_number>/` | Conversation |
| POST | `api/whatsapp/send/` | Send message |

---

## Other

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `send-email-user/` | Send user email |
| GET | `api/datafrom/Nueron` | External integration |
| GET | `test/` | Health check |

# IAAD HRMS Portal — Product Requirements Document

## Architecture
- Single static file `/app/index.html` (2,418 lines, mirrored to `/app/frontend/public/hrms.html`)
- Firebase Auth + Firestore real-time + Storage; deployable to GitHub Pages

## Iteration History

### v1.0 — MVP (June 2026)
- 9 modules (Employees, Postings, Offices, Additional Charges, Attachments, Promotion Forgo, Retirements, Death Register, Users), Audit Logs, Settings
- Dashboard with 10 KPIs + 6 charts
- 11 predefined reports + Custom Report Builder
- Bulk import/export Excel/PDF/CSV, dark mode, bilingual labels
- 76 UP Mandal-Jila pairs seeded
- Default seed Super Admin: admin@iaad.gov.in / Admin@12345

### v1.1 — Major Feature Batch (per user request)
- **Employee Master**:
  - New field **Special Category** (None / Physically Handicapped / PH Dependent / Ex-Servicemen)
  - New field **Health Status** (Normal / Critically Ill Self / Critically Ill Dependent)
  - Status options now include **Terminated**
- **Departments Module (new)**: collection `departments`. Office form's Department is now a dropdown sourced from this collection. Auto-calculated columns: Sanctioned/Filled/Vacant/Additional Charges/Attachments/Offices
- **Postings Overhaul** (only-current model):
  - Removed `relievingDate` and the `current` checkbox
  - Added `cadreJoiningDate`, `districtJoiningDate`, `divisionJoiningDate`
  - **Cutoff date** picker at top of Postings page → drives Duration columns (Office/District/Division as `Xy Ym Zd`)
  - **Upsert by employeeId**: only one posting row per employee; new posting auto-replaces old
  - **Conflict guard**: when saving a posting to an office that has active Additional Charge or Attachment → modal pops up with three options: Cancel save / Keep them (save with remark) / Yes, remove & save (cancels them)
- **Additional Charges**:
  - On selecting an employee, `parentOfficeId` + `parentDistrict` are auto-populated from that employee's current posting and saved with the record
- **Status-change Cascade**:
  - Death entry / Retirement (Superannuation or VRS) / direct Employee status change to Terminated → triggers "Vacate the post?" modal → on Yes, deletes the posting record
- **Auto-Retirement**:
  - On every login, employees with `status='Working'` and DOB+60yrs ≤ today are auto-marked `Retired`
  - If any retired/deceased/VRS/terminated employees still have a posting, a popup appears (every 1st of month or once per month) with a per-row "Vacate" button
- **Enhanced Global Search**: searches across Employees (name/EHRMS/mobile/father/district/category/special/health/status/seniority), Offices (name/department/district/division/remarks), Departments, and Postings (order number)
- **Karmchari Search** (Reports → Karmchari Search): instant employee lookup by any substring → click → full multi-module detail card (personal + current posting + AC + attachments + promotion forgo + retirement + death)
- **Karyalay Search** (Reports → Karyalay Search): district picker + office name search → click → full office detail card (sanctioned/filled/vacant + posted employees + AC + attachments TO and FROM)

## Bugs Fixed in v1.1
- Posting save silently failing when only office was picked → fix: removed `req:true` from district/division on postings, added `onChange='syncOfficeDistrict'` on officeId to auto-populate
- Various jQuery / DataTables / Bootstrap collisions resolved in v1.0.1

## Outstanding / Future
- Code split (>2,400 lines monolith → modular files) — recommended by testing agent
- Email notifications for upcoming retirements (would need Cloud Functions)
- Inline photo cropping

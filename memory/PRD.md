# IAAD HRMS Portal — Product Requirements Document

## Original Problem Statement
Build a complete production-ready single-file HRMS portal (index.html) for the **Government Accountant Cadre** of the **Internal Accounts and Audit Directorate**, replacing an existing Excel workbook (accountant sheet.xlsm). Single-file HTML + Bootstrap + Firebase (Auth/Firestore/Storage) + Chart.js + DataTables + SheetJS + jsPDF, deployable directly on GitHub Pages.

## Architecture
- **Single file**: `/app/index.html` (~1900 lines, all HTML/CSS/JS embedded)
- **Frontend libs (CDN)**: Bootstrap 5, Bootstrap Icons, jQuery, DataTables, Chart.js, SheetJS, jsPDF, jsPDF-autotable, html2canvas, Firebase 10.7.1 compat SDK
- **Backend**: Firebase Auth + Firestore (real-time listeners) + Firebase Storage (employee photos)
- **No server-side code** — pure static deploy
- **Preview URL**: `{REACT_APP_BACKEND_URL}/hrms.html` (also at `/app/frontend/public/hrms.html`)

## User Choices Applied
- Bilingual UI (English + Hindi Devanagari)
- Branding: "Internal Accounts and Audit Directorate" / "आन्तरिक लेखा एवं लेखा परीक्षा निदेशालय"
- Firebase Auth + JS role checks; Firestore rules provided to user
- Super Admin creates regular users from Users module
- Default Super Admin seed: `admin@iaad.gov.in` / `Admin@12345` (auto-created on first sign-in attempt)

## What's Been Implemented (June 2026)
### Auth & RBAC
- Firebase Email/Password Auth with auto-seed of Super Admin on first launch
- Two roles: `super_admin` (full access), `user` (CRUD except delete + no user/audit access)
- Active/Inactive user toggle, password reset email link
- Setup Help modal with one-click Firestore rules snippet + direct Firebase console link

### Modules (all with real-time listeners, CRUD, search, bulk import/export Excel/PDF/CSV)
- **Employees** — Manav Sampada ID, name, father, gender, category, mobile, DOB, retirement (auto-calc), home/current district, seniority, status, photo upload
- **Postings** — division, district, department, office, order #, posting/joining/relieving dates, current flag (auto-closes other postings on transfer)
- **Offices** — name, department, district, division, sanctioned (manual), filled/vacant/additional charges (auto-computed)
- **Additional Charges** — employee, office, order, effective/cancellation dates, status
- **Attachments (Sambaddh)** — employee, parent + attached office, order, dates
- **Promotion Forgo** — order, date, reason
- **Retirements** — type (Superannuation/VRS), retirement date, order
- **Death Register** — death date, order

### Business Rules Engine
- Retirement date auto-calc: if DOB day=1 → last day of previous month after 60 yrs; else last day of same month after 60 yrs
- Death entry: marks employee Deceased, closes active posting & additional charges, recalculates office stats
- Retirement entry: marks Retired/VRS, closes posting & charges
- Transfer (new current posting): auto-closes prior current posting
- Filled = active Working employees with current posting in that office; Vacant = Sanctioned − Filled
- Integrity: cannot delete office with active postings or employee with history

### Dashboard
- 10 KPI cards: Total/Working/Retired/Deceased/Attached/Offices/Sanctioned/Filled/Vacant/Additional Charges
- 6 charts: District-wise, Department Sanctioned vs Filled, Top 10 Vacancy, Retirement Forecast (5 yr), Status Distribution, Division-wise Strength
- Recent Activity feed (last 10 audit entries)
- PDF export via html2canvas + jsPDF

### Reports
- 11 predefined reports (Employee List, District/Office/Department-wise, Vacancy, Upcoming Retirements, Death Register, Additional Charge, Attachment, Promotion Forgo, Posting History)
- Custom Report Builder: module/fields/date-range/filters/sort + saved definitions in `savedReports` collection
- Excel + PDF export, DataTables pagination/search

### Global Search
- Top-bar live search across employees + offices with grouped results & quick-jump

### Audit Logging
- All Create/Update/Delete/Import/Export/Login/Logout/Toggle actions logged with timestamp, user, role, module, record id, before/after data
- Audit Logs page (super_admin only) with DataTables

### UX
- Dark/Light theme with localStorage persistence, theme-aware charts
- Sidebar nav with sections (Main / HR Records / Reports / Administration), mobile-responsive collapse
- Bootstrap modals for forms, toast notifications, confirm dialogs
- Govt aesthetic: deep green (#0b3d2e) + accent gold (#b8862a), Sora + Noto Sans Devanagari fonts
- All interactive elements tagged with `data-testid`

## Deployment to GitHub Pages
1. Copy `/app/index.html` to repository root (rename if needed)
2. Push to GitHub, enable Pages from main branch
3. **First-time setup (mandatory)**: Open Firebase Console → Firestore → Rules → paste the rules from the in-app helper → Publish
4. Visit site → sign in with `admin@iaad.gov.in` / `Admin@12345` → change password from user menu
5. Create additional users from the Users module

## Collections (Firestore)
- users, employees, postings, offices, additionalCharges, attachments, promotionForgo, retirements, deceasedEmployees, auditLogs, savedReports, settings (doc: app)

## Mandal-Jila (Division-District) Seed
- 76 pairs from original Excel `Sheet1` embedded in JS as `MANDAL_JILA` constant (UP divisions + New Delhi)
- Cascading dropdowns: District → Division auto-fill, Division → filtered Districts, District → filtered Offices

## Outstanding / Future Enhancements (P1)
- Server-side enforcement of role checks (currently JS-only — Firestore rules provide secondary defense)
- Inline photo cropping
- Email notifications for retirements (would require Firebase Cloud Functions)
- Hindi UI toggle (currently bilingual labels always shown)
- Department master (currently free-text per office)

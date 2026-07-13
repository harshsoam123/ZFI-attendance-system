# Employee Attendance & Payroll Management System

A modern, responsive, mobile-friendly Employee Attendance & Payroll Management web app built with **HTML5, CSS3, vanilla JavaScript (ES6 modules)** and **Firebase (Authentication, Firestore, Storage)**.

Blue & white theme, dark mode support, sidebar navigation, charts, data tables, automatic payroll calculation engine, and full reporting.

---

## 1. Folder Structure

```
attendance-payroll-app/
├── index.html                 # Login page
├── forgot-password.html       # Forgot password page
├── pages/
│   ├── dashboard.html
│   ├── employees.html         # Employee list, add/edit/delete/search/filter
│   ├── employee-profile.html  # Single employee full profile
│   ├── attendance.html        # Mark / view attendance
│   ├── shifts.html            # Shift management
│   ├── leaves.html            # Leave approvals
│   ├── salary-settings.html   # Global & per-employee salary rules
│   ├── reports.html           # Reports + export PDF/Excel/CSV
├── css/
│   ├── style.css              # Core design system (theme, layout, components)
│   └── responsive.css         # Mobile breakpoints
├── js/
│   ├── firebase-config.js     # <-- put your Firebase project keys here
│   ├── auth.js                # Login / logout / forgot password / route guard
│   ├── sidebar.js             # Shared sidebar + navbar + dark mode toggle
│   ├── utils.js                # Helpers + PAYROLL CALCULATION ENGINE
│   ├── dashboard.js
│   ├── employees.js
│   ├── employee-profile.js
│   ├── attendance.js
│   ├── shifts.js
│   ├── leaves.js
│   ├── salary-settings.js
│   └── reports.js
├── assets/icons/               # svg icons used across UI
└── README.md
```

This is a **static, buildless project** — no npm install, no bundler required. Open it directly in VS Code, install the "Live Server" extension, and click "Go Live". It also deploys as-is to Firebase Hosting, Netlify, Vercel, or GitHub Pages.

---

## 2. Firebase Setup (one-time)

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com) → **Add project**.
2. Inside the project:
   - **Authentication** → Sign-in method → enable **Email/Password**. Manually add your admin user (Authentication → Users → Add user).
   - **Firestore Database** → Create database (start in production mode) → pick a region.
   - **Storage** → Get started (for profile photos, Aadhaar/PAN/resume/document uploads).
3. Go to Project Settings → General → "Your apps" → click the `</>` Web icon → register the app → copy the `firebaseConfig` object.
4. Paste it into `js/firebase-config.js` (marked clearly below).

### Firestore Security Rules (starting point — tighten for production)

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null; // admin-only app
    }
  }
}
```

### Storage Rules

```js
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

### Recommended Firestore Indexes
Firestore will prompt you with a direct link to auto-create composite indexes the first time a query needs one (visible in the browser console as an error link). Just click it — no manual setup needed.

---

## 3. Firestore Data Model

| Collection        | Doc ID              | Key Fields |
|--------------------|----------------------|------------|
| `employees`         | auto (`EMP0001`...)  | fullName, fatherName, mobile, email, address, dob, joiningDate, department, designation, shiftId, monthlySalary, aadhaarNo, panNo, bankAccount, ifsc, upiId, photoUrl, aadhaarUrl, panUrl, resumeUrl, otherDocsUrl[], status |
| `shifts`            | auto                  | name, startTime, endTime, graceMinutes, weeklyOffDays[], lateMarkRules |
| `attendance`        | auto (`empId_date`)  | empId, date, status, checkIn, checkOut, workingHours, overtimeHours, lateMinutes, earlyExitMinutes, remarks |
| `leaves`             | auto                  | empId, type, fromDate, toDate, reason, status(pending/approved/rejected), appliedOn |
| `salarySettings`     | `default` + `{empId}` | perDay, perHour, overtimeRate, lateDeduction, halfDayDeduction, partialExitDeduction, leaveDeduction, bonus, incentive, allowances, pf, esi, tax, otherDeductions |
| `salaryRecords`      | auto (`empId_YYYY-MM`)| month, empId, presentDays, absentDays, leaveDays, halfDays, lateCount, partialExitCount, overtimeHours, grossSalary, totalDeductions, netSalary, generatedOn |
| `notifications`      | auto                  | type, message, read, createdAt |

---

## 4. Running Locally in VS Code

1. Open the folder in VS Code: `File → Open Folder → attendance-payroll-app`.
2. Install the **Live Server** extension (ritwickdey.LiveServer).
3. Right-click `index.html` → **Open with Live Server**.
4. Log in with the admin email/password you created in Firebase Authentication.

---

## 5. Pushing to GitHub

```bash
cd attendance-payroll-app
git init
git add .
git commit -m "Initial commit: Employee Attendance & Payroll Management App"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

> `js/firebase-config.js` contains your Firebase keys. Firebase web API keys are not secret in the traditional sense (they identify the project, not authorize access — your Firestore/Storage security rules do that), but if you'd rather not commit them, add the file to `.gitignore` and instead commit `js/firebase-config.example.js`, then have teammates copy it locally.

---

## 6. Deploying (optional)

**Firebase Hosting**
```bash
npm install -g firebase-tools
firebase login
firebase init hosting   # choose this project, public dir = "." , single-page app = No
firebase deploy
```

**GitHub Pages**: Settings → Pages → Deploy from branch `main` / root.

---

## 7. Payroll Calculation Logic (implemented in `js/utils.js`)

```
Per Day Salary   = Monthly Salary / 30  (or configurable days-in-month)
Per Hour Salary  = Per Day Salary / Shift Hours
Gross Salary     = (Present Days × Per Day Salary)
                  + (Half Days × Per Day Salary × 0.5)
                  + Overtime Pay (Overtime Hours × Overtime Rate)
                  + Bonus + Incentive + Allowances

Deductions       = (Absent Days × Per Day Salary)
                  + (Late Marks × Late Deduction)
                  + (Partial Exits × Partial Exit Deduction)
                  + (Unpaid Leave Days × Leave Deduction)
                  + PF + ESI + Tax + Other Deductions

Net Salary       = Gross Salary − Deductions
```
Salary recalculates automatically whenever an attendance record is added/edited (see `recalculateSalaryForEmployee()` in `utils.js`, called from `attendance.js`).

---

## 8. Importing Attendance from a Biometric Machine (any brand)

The **Attendance** page has an **"Import from Biometric Machine"** button that works with **any machine or software's export** — eSSL, ZKTeco, Realtime, Matrix, or anything else. Instead of expecting one fixed file format, the app:

1. Reads your CSV/Excel file and auto-detects the header row (skipping any title/company-name rows above it).
2. Guesses which column is the Employee ID/Name, Date, and In/Out times.
3. Shows you that guess as an editable **Column Mapping** step — you confirm or correct each dropdown so it matches your exact file, then click **Preview Mapping** to see the parsed rows before importing.

**Two punch-format modes are supported:**
- **Separate In Time / Out Time columns** — the common daily-summary export (one row per employee per day).
- **Single Punch Time column** — a raw punch log (many rows per employee per day); the app automatically takes each employee's earliest punch as Check In and latest as Check Out for that date.

Dates and times are parsed robustly whether the file stores them as real Excel date/time cells, Excel serial numbers, or plain text (`09:15`, `9:15 AM`, `04/07/2026`, etc.) — so formatting quirks between different machine software shouldn't cause problems.

**Matching employees:** you choose whether to match by Employee Code, Employee Name, or Mobile Number — pick whichever your export actually contains. Any row that can't be matched to an existing employee is skipped and listed so nothing is silently lost.

**Daily workflow:**
1. Export the day's (or a date range's) attendance report from your biometric software as CSV or Excel.
2. Attendance → Import from Biometric Machine → select the file.
3. Confirm the column mapping (it's usually already correct) → **Preview Mapping** → **Import Attendance**.

**Going further — fully automatic daily sync:** if your machine supports **ADMS/server push mode**, it can push punches directly to a server in real time instead of a manual export/import. That requires a small backend endpoint (a Firebase Cloud Function works well) that speaks your machine's push protocol and writes into this app's `attendance` collection — a separate, one-time integration project.

---

## 10. Monthly Salary Sheet (Paid Leave / Leave Settlement / Advance format)

The **Monthly Salary Sheet** page (separate from the simpler auto-calculated payroll) reproduces a traditional Indian payroll register layout: Paid Leave, Leave Settlement, Basic/HRA/Other Allowance breakup, and Advance Settlement — all in one exportable sheet.

**One-time setup (Salary Settings → Default Rules):**
- Set **Paid Leaves per Month**, **Basic %**, **HRA %**, **Other Allowance %** (must total 100%).
- On each employee's profile (Job & Salary tab) you can optionally override their paid-leave entitlement, set their opening "leaves already used" balance, and set an opening advance/loan balance if they already owe the company money.

**Monthly workflow:**
1. Go to **Monthly Salary Sheet**, pick the month, click **Generate**.
2. Every white-background cell (Used Leave, Leave This Month, Sunday Working, PF, ESI, TDS, Advance Deduction) is editable — adjust anything that doesn't match your records. Everything else recalculates live.
3. Click **Save Sheet** to lock it in — this rolls each employee's leave-used and advance balances forward into next month automatically, and updates their Salary History on their profile page.
4. Click **Export Excel** to download a spreadsheet matching the grouped-header layout (Paid Leave / Leave Settlement / Advance Settlement sections).

**Formulas used:**
```
Balance Leave    = Total Leave − Used Leave (opening)
Remain Leave     = Balance Leave − Leave This Month + Sunday Working
Working Day      = Days in Month − Leave This Month
Leave Deduction  = Remain Leave < 0 ? |Remain Leave| × (Salary Rate ÷ Days in Month) : 0
Salary           = Salary Rate − Leave Deduction
Basic            = Salary × Basic %
HRA              = Salary × HRA %
Other Allowance  = Salary − Basic − HRA
Balance Advance  = Previous Advance − Advance Deduction
Net Pay          = Salary − PF − ESI − TDS − Advance Deduction
```

---

## 11. Notes

- The app is fully **admin-only** (single role) as specified. Extend `employees` role field if you need multi-role logins later.
- Dark mode preference is stored in `localStorage` and toggled from the top navbar on every page.
- Charts use **Chart.js** (loaded via CDN, no install needed).
- PDF export uses **jsPDF + jspdf-autotable**; Excel/CSV export uses **SheetJS (xlsx)** — both via CDN.
- All file uploads (photo, Aadhaar, PAN, resume, other docs) go to Firebase Storage under `documents/{empId}/...`.

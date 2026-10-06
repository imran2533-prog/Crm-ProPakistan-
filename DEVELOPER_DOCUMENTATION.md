# PRO HMS — Pakistan Recovery Oasis Hospital Management System
## Complete Developer Documentation

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Project Structure](#3-project-structure)
4. [Environment Setup](#4-environment-setup)
5. [Database (MongoDB)](#5-database-mongodb)
6. [Backend — Flask API](#6-backend--flask-api)
   - [Authentication & Session](#61-authentication--session)
   - [User Management](#62-user-management)
   - [Patient Management](#63-patient-management)
   - [Canteen Module](#64-canteen-module)
   - [Expenses & Finance](#65-expenses--finance)
   - [Accounts Summary](#66-accounts-summary)
   - [Overheads & Finance Summary](#67-overheads--finance-summary)
   - [Recovery Tracking](#68-recovery-tracking)
   - [Employee Management](#69-employee-management)
   - [Utility Bills](#610-utility-bills)
   - [Call & Meeting Tracker](#611-call--meeting-tracker)
   - [Payment Records](#612-payment-records)
   - [Notifications System](#613-notifications-system)
   - [Export (Excel)](#614-export-excel)
   - [Dashboard Metrics](#615-dashboard-metrics)
   - [Psychologist Sessions](#616-psychologist-sessions)
7. [Frontend — Single Page Application](#7-frontend--single-page-application)
8. [Role-Based Access Control (RBAC)](#8-role-based-access-control-rbac)
9. [Financial Logic Overview](#9-financial-logic-overview)
10. [Real-Time Notifications (Socket.IO)](#10-real-time-notifications-socketio)
11. [PWA & Android TWA](#11-pwa--android-twa)
12. [Deployment](#12-deployment)
13. [Security Features](#13-security-features)
14. [Known Limitations & Notes](#14-known-limitations--notes)

---

## 1. Project Overview

**PRO HMS** (Pakistan Recovery Oasis Hospital Management System) ek full-stack web application hai jo ek rehab/hospital facility ke management ke liye banaya gaya hai. Yeh system patient admission se lekar discharge tak, canteen billing, staff management, aur financial reporting sab kuch handle karta hai.

**Core Features:**
- Patient admission, tracking, aur discharge management
- Role-based login system (Admin, Doctor, Psychologist, Canteen, General Staff)
- Real-time notifications via Socket.IO
- Canteen billing with daily/monthly tracking per patient
- Financial accounts, overheads, aur expense tracking
- Recovery tracking for discharged patients
- Call & Meeting tracker for family follow-ups
- Employee/staff management with salary tracking
- Utility bills management
- Excel export for patient data
- Progressive Web App (PWA) + Android TWA support

---

## 2. Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.x + Flask 3.0 |
| Database | MongoDB Atlas (Cloud) |
| ODM | Flask-PyMongo + pymongo |
| Authentication | Flask Session + Werkzeug password hashing |
| Real-Time | Flask-SocketIO (threading mode) |
| Scheduling | APScheduler (background jobs) |
| Security | Flask-Talisman, Flask-Limiter, Flask-CORS |
| Compression | Flask-Compress (gzip) |
| Email | Gmail SMTP via smtplib (SSL) |
| Data Export | pandas + openpyxl |
| Password Reset | itsdangerous (URLSafeTimedSerializer) |
| Frontend | Vanilla HTML/CSS/JS (Single Page App) |
| CSS Framework | Tailwind CSS (CDN) |
| Icons | Font Awesome 6.4 |
| Fonts | Poppins, Noto Nastaliq Urdu, Manrope, DM Sans |
| Deployment (Primary) | Render.com (gunicorn, gthread workers) |
| Deployment (Alt) | Vercel (serverless, limited features) |
| Mobile | Android TWA (Trusted Web Activity) |

---

## 3. Project Structure

```
PRO-PAKISTAIN-main/
│
├── app.py                  ← Main Flask application (ALL backend logic)
├── requirements.txt        ← Python dependencies
├── runtime.txt             ← Python version for Render
├── render.yaml             ← Render.com deployment config
├── vercel.json             ← Vercel deployment config
├── migrate_data.py         ← One-time data migration script
├── .env                    ← Environment variables (local dev only)
├── .python-version         ← pyenv Python version
│
├── templates/
│   └── index.html          ← Single Page App (entire frontend)
│
├── static/
│   ├── manifest.json       ← PWA manifest
│   ├── sw.js               ← Service Worker (PWA offline support)
│   ├── favicon.svg
│   ├── favicon-96x96.png
│   ├── logo.png
│   └── .well-known/
│       └── assetlinks.json ← Android TWA verification
│
└── android-twa/            ← Android TWA project (separate Android Studio project)
    └── app/src/main/
        └── AndroidManifest.xml
        └── java/site/prohms/app/MainActivity.java
```

---

## 4. Environment Setup

### Local Development

1. Python virtual environment banayein:
```bash
python -m venv venv
venv\Scripts\activate      # Windows
source venv/bin/activate   # Linux/Mac
```

2. Dependencies install karein:
```bash
pip install -r requirements.txt
```

3. `.env` file mein yeh variables set karein:
```env
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>
SECRET_KEY=<random-strong-secret-key>
GMAIL_USER=youremail@gmail.com
GMAIL_APP_PASSWORD=<gmail-app-password>
PASSWORD_RESET_EXPIRY_MINUTES=30
ADMIN_EMAIL=admin@example.com
```

4. Application run karein:
```bash
python app.py
```
Ya production mein:
```bash
gunicorn --worker-class gthread --threads 4 -w 1 app:app
```

### First Run Behavior
- App automatically ek default Admin user **`ImranSaab`** (password: `password123`) create karta hai agar database mein koi user nahi hai.
- Database indices automatically create ho jaate hain.

---

## 5. Database (MongoDB)

### Collections

| Collection | Purpose |
|-----------|---------|
| `users` | Login accounts (Admin, Doctor, etc.) |
| `patients` | Patient records (admission to discharge) |
| `patient_records` | Session notes aur medical records per patient |
| `canteen_sales` | Individual canteen transactions |
| `canteen_balance_overrides` | Manual old-balance overrides for monthly canteen table |
| `expenses` | Income/outgoing expense entries + auto payment records |
| `recovery` | Discharged patient recovery tracking |
| `employees` | Staff/employee records with salary info |
| `utility_bills` | Electricity, gas, etc. bills |
| `call_meeting_tracker` | Family call/meeting log per patient |
| `overheads` | Daily kitchen/other overhead entries |
| `notifications` | Real-time notification documents |
| `psych_sessions` | Psychologist session count tracking |

### Key Patient Document Fields
```json
{
  "_id": "ObjectId",
  "name": "Patient Name",
  "fatherName": "Father Name",
  "age": "35",
  "cnic": "12345-1234567-1",
  "contactNo": "03001234567",
  "address": "City, Pakistan",
  "guardianName": "Guardian",
  "relation": "Brother",
  "admissionDate": "2024-01-15T00:00:00",
  "isDischarged": false,
  "dischargeDate": null,
  "monthlyFee": "15000",
  "monthlyAllowance": "3000",
  "receivedAmount": "10000",
  "laundryStatus": true,
  "laundryAmount": 3500,
  "complaint": "Drug addiction",
  "drugProblem": "Yes",
  "drug": "Heroin",
  "maritalStatus": "Single",
  "prevAdmissions": "0",
  "photo1": "",
  "photo2": "",
  "photo3": "",
  "notes": [],
  "created_at": "ISODate"
}
```

### Database Indices (Auto-Created)
- `users`: unique index on `username`
- `patients`: index on `admissionDate`, `isDischarged`
- `canteen_sales`: index on `patient_id`, `date`
- `expenses`: index on `date`, `category`
- `recovery`: index on `dischargeDate`, `status`
- `notifications`: index on `created_at`, `target_roles`, `target_user_ids`
- `call_meeting_tracker`: index on `year`, `month`

---

## 6. Backend — Flask API

Poora backend **`app.py`** mein hai. Koi separate routes ya blueprints nahi hain.

---

### 6.1 Authentication & Session

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/login` | Public | Login (5 req/min rate limit) |
| POST | `/api/auth/logout` | Logged In | Session clear karna |
| GET | `/api/auth/session` | Public | Current session status check |
| POST | `/api/auth/forgot` | Public | Password reset email bhejta hai |
| POST | `/api/auth/reset` | Public | Token se password reset |

**Login Flow:**
1. Username + password POST hota hai
2. `check_password_hash` se verify hota hai
3. Success pe `session['user_id']`, `session['username']`, `session['role']` set hote hain
4. Frontend ko `username`, `role`, `name`, `user_id` return hota hai

**Password Reset Flow:**
1. User username + registered email deta hai
2. `itsdangerous` se time-bound signed token generate hota hai
3. Gmail SMTP se email bheja jata hai jisme reset link hota hai
4. User link click karta hai → token verify → new password set

**Session Check (`/api/auth/session`):**
- Har page load pe frontend yeh call karta hai
- Agar logged in hai to user info return karta hai
- Agar nahi to `{"is_logged_in": false}` return hota hai

---

### 6.2 User Management

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/users` | Admin | Sab users list karna |
| POST | `/api/users` | Admin | Naya user banana |
| DELETE | `/api/users/<id>` | Admin | User delete karna |
| POST | `/api/users/change_password` | Logged In | Apna password change karna |

**Available Roles:**
- `Admin` — Full access to everything
- `Doctor` — Patient admission, medical records
- `Psychologist` — Session notes, recovery tracking
- `Canteen` — Canteen sales entry
- `General Staff` — Limited read access, employee viewing

**Notes:**
- Default admin `ImranSaab` ko delete nahi kiya ja sakta
- Password hashed store hota hai (Werkzeug pbkdf2)
- Email unique hona chahiye har user ka
- User create/delete pe doosre admins ko notification milti hai

---

### 6.3 Patient Management

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/patients` | Logged In | Patients list (`?archived=false/true/all`) |
| POST | `/api/patients` | Admin, Doctor | Naya patient admit karna |
| PUT | `/api/patients/<id>` | Admin, Doctor | Patient record update/discharge |
| DELETE | `/api/patients/<id>` | Admin | Patient delete karna |
| POST | `/api/patients/<id>/session_note` | Admin, Psychologist | Session note add karna |
| POST | `/api/patients/<id>/medical_record` | Admin, Doctor | Medical record add karna |
| GET | `/api/patients/<id>/records` | Logged In | Patient ke sab records dekhna |

**Patient List Query Params:**
- `?archived=false` (default) — Sirf active patients
- `?archived=true` — Sirf discharged patients (archive)
- `?archived=all` — Sab patients (export ke liye)

**Admission Logic:**
- Admission pe `receivedAmount > 0` ho to automatically `expenses` mein "Initial Advance" record ban jata hai
- `laundryStatus=true` ho to `laundryAmount` default 3500 set hota hai (one-time charge)
- `isDischarged=false`, `dischargeDate=null` se start hota hai

**Discharge Logic:**
- `isDischarged=true` set karne pe `dischargeDate` automatically set hoti hai
- Discharge ke baad billing freeze ho jati hai (days calculation `dischargeDate` pe rok di jati hai)
- `isDischarged=false` karne pe `dischargeDate=null` ho jata hai

**Billing Calculation:**
```
Days Elapsed = dischargeDate - admissionDate (discharged patient)
Days Elapsed = today - admissionDate (active patient)

Daily Rate = monthlyFee / 30
Prorated Fee = Daily Rate × max(days_elapsed, 1)

Balance Due = Prorated Fee + canteenSpent + laundryAmount - receivedAmount
```

---

### 6.4 Canteen Module

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| POST | `/api/canteen/sales` | Admin, Canteen | Simple sale record karna |
| GET | `/api/canteen/sales/breakdown` | Admin, Canteen | Monthly per-patient breakdown |
| GET | `/api/canteen/sales/history` | Admin | Sales history (last 100) |
| GET | `/api/canteen/daily-sheet` | Admin, Canteen | Aaj ki daily sheet |
| GET | `/api/canteen/monthly-table` | Admin, Canteen | Monthly grid table |
| POST | `/api/canteen/daily-entry` | Admin, Canteen | Daily entry save/update |
| POST | `/api/canteen/old-balance` | Admin | Manual old balance override |

**Monthly Table Logic:**
- Har patient ka ek row hota hai
- Columns: Old Balance | Day 1 | Day 2 | ... | Day 31 | Other | Month Total | Total (all-time)
- `Old Balance` = previous months ke sab canteen purchases ka total
- Admin daily entries edit kar sakta hai; Canteen staff sirf naya entry daal sakta hai
- `entry_type: 'daily'` = normal daily purchase
- `entry_type: 'other'` = misc adjustment amount

**Canteen Staff Restriction:**
- Canteen staff kisi existing entry ko edit nahi kar sakta
- Sirf naye entries add kar sakta hai
- Ek din mein ek entry per patient per type

---

### 6.5 Expenses & Finance

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/expenses` | Logged In | Sab expenses + auto entries |
| POST | `/api/expenses` | Admin | Naya expense/income add karna |
| DELETE | `/api/expenses/<id>` | Admin | Expense delete karna |
| GET | `/api/expenses/summary` | Logged In | Monthly incoming/outgoing summary |

**Expense Types:**
- `type: 'incoming'` — Income entry
- `type: 'outgoing'` — Expense entry

**Auto Entries (list mein dikh te hain lekin DB mein store nahi):**
1. `Monthly Fees (auto)` — Sab patients ke monthly fees ka total
2. `Canteen Sales (auto)` — Current month ki canteen sales ka total

**Patient Payment Auto-Recording:**
- Jab patient ki `receivedAmount` update hoti hai to automatically `expenses` mein record banta hai
- `category: 'Patient Fee'`, `auto: True`
- Yeh double-counting se bachne ke liye summary mein alag count hote hain

---

### 6.6 Accounts Summary

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/accounts/summary` | Admin | Har patient ka complete billing summary |

**Response per patient:**
```json
{
  "id": "...",
  "name": "Patient Name",
  "fatherName": "...",
  "age": "...",
  "area": "City",
  "admissionDate": "...",
  "dischargeDate": "...",
  "monthlyFee": "15000",
  "calculatedFee": 25000,
  "daysElapsed": 50,
  "canteenTotal": 3200,
  "laundryStatus": true,
  "laundryAmount": 3500,
  "receivedAmount": "10000",
  "isDischarged": false
}
```

Frontend yahan se balance calculate karta hai:
```
Balance Due = calculatedFee + canteenTotal + laundryAmount - receivedAmount
```

---

### 6.7 Overheads & Finance Summary

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/finance/summary/<month>/<year>` | Admin | Complete monthly financial summary |
| GET | `/api/inventory/stats/<month>/<year>` | Logged In | Monthly admission/discharge/canteen stats |

**Finance Summary Response:**
```json
{
  "month": 10,
  "year": 2026,
  "totalSalaries": 150000,
  "totalUtilityBills": 25000,
  "totalKitchen": 45000,
  "totalCanteenAuto": 18000,
  "totalOthers": 12000,
  "totalPayAdvance": 5000,
  "totalEstimatedOverheads": 255000,
  "totalIncome": 300000,
  "profit": 45000
}
```

**Overheads Daily Tracking:**
- Kitchen expenses, other expenses, pay advances
- Canteen column auto-sync hota hai `canteen_sales` se
- Daily profit/loss = income - overheads

---

### 6.8 Recovery Tracking

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/recovery` | Logged In | Sab recovery entries |
| POST | `/api/recovery` | Admin, General Staff, Doctor, Psychologist | Entry add karna |
| PUT | `/api/recovery/<id>` | Admin, General Staff, Doctor, Psychologist | Entry update karna |
| DELETE | `/api/recovery/<id>` | Admin, General Staff, Doctor, Psychologist | Entry delete karna |

**Recovery Status Options:**
- `Active Recovery` — Patient theek ja raha hai
- `Relapsed` — Dobara lapse ho gaya
- `No Idea` — Contact nahi ho pa raha

**Purpose:** Discharged patients ka follow-up track karna. Yeh patient records se alag collection hai.

---

### 6.9 Employee Management

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/employees` | Admin, General Staff | Sab employees |
| POST | `/api/employees` | Admin | Naya employee add karna |
| PUT | `/api/employees/<id>` | Admin | Employee update karna |
| DELETE | `/api/employees/<id>` | Admin | Employee delete karna |

**Employee Fields:**
- `name`, `designation`, `pay` (monthly salary)
- `advance` (current month advance — auto resets next month)
- `duty_timings`, `date_of_joining`, `cnic`, `phone`

**Advance Logic:**
- Advance month/year ke saath save hota hai (`advance_month`, `advance_year`)
- Agar month/year query param match nahi karta to `advance = 0` return hota hai
- Yeh automatically monthly salary sheets mein advance reset simulate karta hai

---

### 6.10 Utility Bills

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/utility_bills` | Admin | Bills list (optional `?month=&year=`) |
| POST | `/api/utility_bills` | Admin | Naya bill add karna |
| DELETE | `/api/utility_bills/<id>` | Admin | Bill pay/delete karna |

**Bill Pay Flow:**
- Bill delete karne pe automatically `expenses` mein `outgoing` entry add ho jati hai
- Category: `Utility Bill`, Note mein bill type aur ref number

---

### 6.11 Call & Meeting Tracker

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/call_meeting_tracker` | Logged In | Monthly entries |
| POST | `/api/call_meeting_tracker` | Admin | Entry add/update karna |
| DELETE | `/api/call_meeting_tracker/<id>` | Admin | Entry delete karna |
| GET | `/api/call_meeting_tracker/summary/<month>/<year>` | Logged In | Monthly summary |

**Entry Types:**
- `Meeting` — In-person family visit
- `Call` — Phone call
- `VN` — Voice Note

**Use Case:** Track which discharged patients ke family se kab call/meeting hui.

---

### 6.12 Payment Records

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/payment-records` | Admin | Sab patient payment receipts |
| PUT | `/api/payment-records/<id>` | Admin | Payment record update |
| DELETE | `/api/payment-records/<id>` | Admin | Payment record delete (balance reverse bhi hota hai) |

**Important:**
- Payment record delete karne pe patient ki `receivedAmount` bhi automatically adjust hoti hai
- Payment update karne pe delta amount calculate hota hai aur patient balance sync hota hai
- Online payments ke liye `screenshot` field support karta hai

---

### 6.13 Notifications System

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/notifications` | Logged In | Apni notifications dekhna (last 50) |
| POST | `/api/notifications/<id>/read` | Logged In | Notification read mark karna |
| POST | `/api/notifications/read-all` | Logged In | Sab notifications read mark karna |
| GET | `/api/debug/notifications` | Logged In | Debug — sab notifications without filter |

**Notification Targeting:**
- `target_roles` — Role-wise (e.g., `['Admin']`)
- `target_user_ids` — Specific users ko

**Auto-Triggered Notifications (Backend se):**
- New patient admitted
- Patient discharged
- Patient record updated
- Patient deleted
- New user created
- User deleted
- Expense recorded/deleted

**`notify_admins()` Helper:**
- Action karne wale admin ke alaawa sab doosre admins ko notification bhejta hai
- Socket.IO se real-time push hoti hai (Vercel pe polling fallback)

---

### 6.14 Export (Excel)

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| POST | `/api/export` | Admin, Doctor, Psychologist | Patient data Excel file download |

**Request Body:**
```json
{
  "fields": ["name", "fatherName", "admissionDate", "age", "complaint"]
}
```

**Notes:**
- Admin ko sensitive fields (CNIC, contact, address) milte hain
- Non-admin roles ko sensitive fields blank milte hain
- Output: A4 landscape Excel file

---

### 6.15 Dashboard Metrics

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| GET | `/api/dashboard` | Logged In | Main dashboard KPIs |
| GET | `/api/dashboard/admissions` | Logged In | Current month admissions list |
| GET | `/api/db-status` | Public | Database connection health check |

**Dashboard Response:**
```json
{
  "totalPatients": 45,
  "admissionsThisMonth": 5,
  "dischargesThisMonth": 3,
  "totalExpectedBalance": 650000,
  "totalCanteenSalesThisMonth": 28000,
  "totalExpensesThisMonth": 95000,
  "totalPsychSessionsToday": 4
}
```

---

### 6.16 Psychologist Sessions

- `psych_sessions` collection mein psychologist ke daily sessions track hote hain
- Dashboard mein `totalPsychSessionsToday` count dikhta hai
- Aaj ke `psych_sessions` count kiye jaate hain start-of-day se

---

## 7. Frontend — Single Page Application

Poora frontend **ek single file** `templates/index.html` mein hai. Yeh approximately 10,000+ lines ka file hai.

### Architecture
- **No framework** — Pure Vanilla JavaScript
- **SPA routing** — Hash-based ya CSS `display:none/block` se sections toggle
- **Tailwind CSS** — CDN se load hota hai
- **Socket.IO client** — Deferred load (sirf login ke baad chahiye)

### Sections / Pages
| Section | Description |
|---------|-------------|
| Login Screen | Username/password form |
| Dashboard | KPI cards, stats, charts |
| Patient Directory | Active patients list, search, filter |
| Patient Archive | Discharged patients |
| Patient Detail | Full patient profile, billing, records |
| Canteen | Monthly table, daily sheet, breakdown |
| Accounts | Patient billing summary table |
| Finance/Overheads | Monthly overhead tracking |
| Recovery Tracker | Discharged patient follow-up |
| Call/Meeting Log | Family contact calendar |
| Employees | Staff list with salary info |
| Utility Bills | Bills list and payment |
| Expenses | Income/Outgoing ledger |
| Notifications | Bell icon with dropdown |
| User Management | Admin panel for user CRUD |
| Settings | Change password |

### CSS Design System
```css
--shamrock: #2dd4bf      /* Primary teal */
--shamrock-strong: #0d9488
--forest: #042f2e        /* Dark background */
--amber: #f59e0b         /* Warning/highlight */
--sky: #06b6d4           /* Info color */
--surface: #f3faf9       /* Page background */
--brand-primary: #0d9488
--brand-secondary: #ff7a59
```

### Key Frontend Features
- **Urdu text support** — Noto Nastaliq Urdu font, RTL direction
- **Print styles** — Specific sections print-friendly hain
- **Searchable dropdowns** — Custom dropdown component patients ke liye
- **Editable cells** — Canteen monthly table mein inline editing
- **PWA install prompt** — App install banner
- **Blinking dot** — Real-time emergency indicator
- **Animations** — `fadeIn`, `pulse-red-bg`, `blink-pulse`

---

## 8. Role-Based Access Control (RBAC)

### Roles and Permissions

| Feature | Admin | Doctor | Psychologist | General Staff | Canteen |
|---------|-------|--------|--------------|---------------|---------|
| Dashboard | ✅ Full | ✅ | ✅ | ✅ | ✅ |
| Patient List (Active) | ✅ | ✅ | ✅ | ✅ | ❌ |
| Patient Archive | ✅ | ✅ | ✅ | ✅ | ❌ |
| Admit Patient | ✅ | ✅ | ❌ | ❌ | ❌ |
| Edit/Discharge Patient | ✅ | ✅ | ❌ | ❌ | ❌ |
| Delete Patient | ✅ | ❌ | ❌ | ❌ | ❌ |
| Session Notes | ✅ | ❌ | ✅ | ❌ | ❌ |
| Medical Records | ✅ | ✅ | ❌ | ❌ | ❌ |
| Canteen Module | ✅ | ❌ | ❌ | ❌ | ✅ |
| Accounts/Billing | ✅ | ❌ | ❌ | ❌ | ❌ |
| Finance/Overheads | ✅ | ❌ | ❌ | ❌ | ❌ |
| Recovery Tracking | ✅ | ✅ | ✅ | ✅ | ❌ |
| Employees | ✅ Full | ❌ | ❌ | ✅ View | ❌ |
| Utility Bills | ✅ | ❌ | ❌ | ❌ | ❌ |
| Expenses | ✅ | ❌ | ❌ | ❌ | ❌ |
| Export Excel | ✅ | ✅ | ✅ | ❌ | ❌ |
| User Management | ✅ | ❌ | ❌ | ❌ | ❌ |
| Call/Meeting Log | ✅ Full | ✅ View | ✅ View | ✅ View | ❌ |
| Notifications | ✅ | ✅ | ✅ | ✅ | ✅ |

### Backend Decorators

```python
@login_required          # Session mein user_id hona chahiye
@role_required(['Admin'])  # Specific roles allowed
@role_required(['Admin', 'Doctor'])  # Multiple roles
```

---

## 9. Financial Logic Overview

### Patient Billing Formula
```
Step 1: Days Elapsed
  - Active patient:    days = today - admissionDate
  - Discharged:        days = dischargeDate - admissionDate

Step 2: Prorated Fee
  - daily_rate = monthlyFee / 30
  - fee = daily_rate × max(days, 1)

Step 3: Total Charges
  - total = fee + canteenTotal + laundryAmount (if laundryStatus=true)

Step 4: Balance Due
  - balance = total - receivedAmount
  - Dashboard shows sum of all positive balances (money owed to facility)
```

### Canteen Billing
- `canteen_sales` collection mein har transaction alag document hai
- Monthly total per patient aggregation se nikala jata hai
- Patient billing mein sirf `entry_type != 'other'` entries count hoti hain
- `other` entries budgeting ke liye hain, actual billing mein nahi

### Double-Counting Prevention
- Patient payments `expenses` mein `auto:true` flag ke saath save hote hain
- `expenses/summary` endpoint sirf manual entries count karta hai
- Auto entries sirf list display ke liye hai, totals mein nahi

---

## 10. Real-Time Notifications (Socket.IO)

### How It Works
1. User login karta hai → Socket.IO connect hota hai
2. `handle_socket_connect` → user ko `user:<user_id>` aur `role:<role>` rooms mein join karta hai
3. Backend mein `create_notification()` call hoti hai → MongoDB mein save + Socket.IO `emit`
4. Frontend connected clients ko instantly notification milti hai

### Vercel Limitation
- Vercel serverless hai — persistent Socket.IO rooms nahi hote
- Vercel pe Socket.IO push skip ho jata hai (`IS_VERCEL` check)
- Frontend Vercel pe polling karta hai `/api/notifications` endpoint se

### Render (Production)
- Render pe persistent process hoti hai
- Socket.IO real-time push kaam karta hai

---

## 11. PWA & Android TWA

### PWA Features
- `static/manifest.json` — App name, icons, theme color, start URL
- `static/sw.js` — Service Worker (offline caching)
- Mobile-friendly meta tags (`apple-mobile-web-app-capable`)
- Install prompt support

### Android TWA
- `android-twa/` folder mein Android Studio project hai
- TWA (Trusted Web Activity) web app ko native Android app ki tarah wrap karta hai
- `static/.well-known/assetlinks.json` → Digital Asset Links verification
- `/api/assetlinks.json` route yahi file serve karta hai

---

## 12. Deployment

### Render.com (Recommended)
```yaml
# render.yaml
services:
  - type: web
    name: pro-hospital-management
    env: python
    rootDir: PRO-PAKISTAIN-main
    buildCommand: pip install -r requirements.txt
    startCommand: gunicorn --worker-class gthread --threads 4 -w 1 app:app
```

**Required Environment Variables on Render:**
| Variable | Value |
|----------|-------|
| `MONGO_URI` | MongoDB Atlas connection string |
| `SECRET_KEY` | Auto-generated |
| `GMAIL_USER` | Gmail address for password reset |
| `GMAIL_APP_PASSWORD` | Gmail App Password |
| `PASSWORD_RESET_EXPIRY_MINUTES` | 30 |
| `ADMIN_EMAIL` | Default admin email |

### Vercel (Alternative — Limited)
- `vercel.json` present hai
- `IS_VERCEL` environment variable detect hota hai
- **Disabled on Vercel:** Rate limiting, Gzip compression, Socket.IO push
- **Fallback on Vercel:** Notification polling instead of real-time push

---

## 13. Security Features

| Feature | Implementation |
|---------|---------------|
| Password Hashing | Werkzeug `pbkdf2:sha256` |
| Session Management | Flask server-side sessions |
| Rate Limiting | Flask-Limiter (5/min login, 2000/day global) |
| CORS | Flask-CORS (all origins) |
| Input Sanitization | `clean_input_data()` — strips whitespace |
| SQL Injection | N/A — MongoDB (no SQL) |
| CSRF | Session-based (no extra token) |
| Password Reset | Signed time-bound tokens (30 min expiry) |
| HTTPS | Render/Vercel handle SSL |
| Content Security | Flask-Talisman (disabled due to eventlet conflict) |

**`clean_input_data()` function:**
- Sab string values se leading/trailing spaces strip karta hai
- Nested dicts aur lists ko recursively process karta hai
- Injection attacks se basic protection

---

## 14. Known Limitations & Notes

1. **Single `app.py` file** — Poora backend ek file mein hai. Large codebase ke liye blueprints consider karein.

2. **No file upload storage** — Patient photos (`photo1`, `photo2`, `photo3`) base64 strings ke tor par MongoDB mein store hote hain. Large scale pe yeh performance issue ban sakta hai. Recommend: Cloudinary ya S3.

3. **Flask-Talisman disabled** — Security headers nahi hain abhi. Eventlet ke saath conflict ki wajah se disable hai.

4. **Socket.IO threading mode** — `async_mode="threading"` use ho raha hai. High concurrent users ke liye `eventlet` ya `gevent` better hai.

5. **Single worker on Render** — `gunicorn -w 1` — Multiple workers Socket.IO ke saath problematic hain (in-memory rooms). Production pe Redis-based Socket.IO adapter chahiye hogi.

6. **No automated tests** — Koi unit/integration tests nahi hain. New features add karne se pehle test suite banana chahiye.

7. **Frontend single file** — `index.html` bahut bada hai. Component-based framework (Vue/React) aur bundle tool consider karein.

8. **CORS all origins open** — `CORS(app)` sab origins allow karta hai. Production mein specific domains whitelist karein.

9. **Password reset email** — Gmail SMTP use hota hai. High volume ke liye SendGrid ya SES better option hai.

10. **Vercel cold starts** — Serverless pe memory-based rate limiting reset ho jati hai, isliye disabled hai.

---

## Quick Reference — All API Endpoints

```
AUTH
  POST   /api/auth/login
  POST   /api/auth/logout
  GET    /api/auth/session
  POST   /api/auth/forgot
  POST   /api/auth/reset

USERS
  GET    /api/users
  POST   /api/users
  DELETE /api/users/<id>
  POST   /api/users/change_password

PATIENTS
  GET    /api/patients
  POST   /api/patients
  PUT    /api/patients/<id>
  DELETE /api/patients/<id>
  POST   /api/patients/<id>/session_note
  POST   /api/patients/<id>/medical_record
  GET    /api/patients/<id>/records

CANTEEN
  POST   /api/canteen/sales
  GET    /api/canteen/sales/breakdown
  GET    /api/canteen/sales/history
  GET    /api/canteen/daily-sheet
  GET    /api/canteen/monthly-table
  POST   /api/canteen/daily-entry
  POST   /api/canteen/old-balance

EXPENSES
  GET    /api/expenses
  POST   /api/expenses
  DELETE /api/expenses/<id>
  GET    /api/expenses/summary

ACCOUNTS
  GET    /api/accounts/summary

FINANCE
  GET    /api/finance/summary/<month>/<year>
  GET    /api/inventory/stats/<month>/<year>

RECOVERY
  GET    /api/recovery
  POST   /api/recovery
  PUT    /api/recovery/<id>
  DELETE /api/recovery/<id>

EMPLOYEES
  GET    /api/employees
  POST   /api/employees
  PUT    /api/employees/<id>
  DELETE /api/employees/<id>

UTILITY BILLS
  GET    /api/utility_bills
  POST   /api/utility_bills
  DELETE /api/utility_bills/<id>

CALL/MEETING TRACKER
  GET    /api/call_meeting_tracker
  POST   /api/call_meeting_tracker
  DELETE /api/call_meeting_tracker/<id>
  GET    /api/call_meeting_tracker/summary/<month>/<year>

PAYMENT RECORDS
  GET    /api/payment-records
  PUT    /api/payment-records/<id>
  DELETE /api/payment-records/<id>

NOTIFICATIONS
  GET    /api/notifications
  POST   /api/notifications/<id>/read
  POST   /api/notifications/read-all

DASHBOARD
  GET    /api/dashboard
  GET    /api/dashboard/admissions

EXPORT
  POST   /api/export

MISC/DEBUG
  GET    /api/db-status
  GET    /api/debug/dashboard
  GET    /api/debug/notifications
  POST   /api/admin/fix-discharge-dates
  GET    /.well-known/assetlinks.json
```

---

*Documentation prepared for PRO-PAKISTAIN HMS — October 2026*
*For questions, contact the original developer.*

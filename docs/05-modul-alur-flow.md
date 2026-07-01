# Module & Program Flow

## General Scenario — SSO Login

```
┌─────────┐     ┌──────────────┐     ┌─────────────┐
│ Browser │────>│  SSO Portal  │────>│  Role       │
│         │     │  /login      │     │  Detection  │
└─────────┘     └──────────────┘     └──────┬──────┘
                                             │
                     ┌───────────────────────┼───────────────────────┐
                     │                       │                       │
                     ▼                       ▼                       ▼
             ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
             │ Role: Admin  │       │ Role: Student│       │ Role: Teacher│
             │ /admin/*     │       │ /student/*   │       │ /teacher/*   │
             └──────────────┘       └──────────────┘       └──────────────┘

                     ┌──────────────┐
                     │ Role: Guard- │
                     │ ian          │
                     │ /guardian/*  │
                     └──────────────┘
```

---

## 1. Student Attendance Flow

This is the most important and most critical flow in the system. Must be optimized for speed and reliability.

```
[Student opens browser]
        │
        ▼
[Login via SSO] ─── (role detection: student) ───> [Student Dashboard]
        │
        ▼
[Menu: Attendance]
        │
        ▼
[System requests permission:]
  ● Camera Access (WebRTC)
  ● Location Access (Geolocation API)
        │
        ▼
[Student takes selfie]
        │
        ▼
[Client-side image compression]
  ● Resize: 320x240 px
  ● Quality: JPEG 90%
  ● Target: ≤ 20 KB
        │
        ▼
[Send data to server:]
  ● photo (compressed Base64)
  ● latitude
  ● longitude
  ● timestamp
        │
        ▼
[Server performs Triple-Layer Validation]
  1. Check academic calendar (holiday?)
  2. Check active day (Saturday/Sunday?)
  3. Check time range:
     │
     ├── < open_time         → "Not yet time for attendance"
     ├── ≤ late_threshold    → status: "Present"
     ├── ≤ close_time        → status: "Late"
     └── > close_time        → "Attendance closed"
        │
        ▼
[Save to database]
  ● Photo uploaded to Object Storage
  ● Photo URL saved in attendances table
        │
        ▼
[Display confirmation to student]
```

### Attendance Technical Details

| Step | Technology | Notes |
|---|---|---|
| Camera access | `navigator.mediaDevices.getUserMedia()` | HTTPS required |
| Image capture | Canvas API: `canvas.toDataURL('image/jpeg', 0.9)` | Resize to 320x240 |
| Geolocation | `navigator.geolocation.getCurrentPosition()` | `enableHighAccuracy: true` |
| Server validation | Laravel Request + Validation | Triple-layer |
| Photo upload | Laravel Filesystem + S3 driver | File stored, URL saved to DB |

---

## 2. Guardian Flow — Monitor & Leave

```
[Guardian logs in via SSO]
        │
        ▼
[Guardian Dashboard]
        │
        ├── [Select child profile] ─── [View attendance chart]
        │                              ● Daily
        │                              ● Monthly
        │                              ● Semester
        │
        └── [Menu: Submit Leave]
                │
                ▼
        [Leave Form]
          ● Select child (if >1)
          ● Category: Sick / Event / Competition
          ● Start date – end date
          ● Upload evidence (doctor's note, assignment letter)
          ● Submit
                │
                ▼
        [Status: Pending]
        (Waiting for Homeroom Teacher verification)
```

### Leave Business Rules

| Rule | Detail |
|---|---|
| Relationship | 1 Guardian → N Students (siblings) |
| Evidence upload | Format: JPG, PNG, PDF. Max 2 MB per file |
| Approval status | Pending → Approved/Rejected (by Homeroom Teacher) |
| Consecutive leave | Allowed, with valid date range |

---

## 3. Homeroom Teacher Flow — Verification & Monitoring

```
[Homeroom Teacher logs in via SSO]
        │
        ▼
[Homeroom Teacher Dashboard]
        │
        ├── [Notification: Incoming Leave]
        │       │
        │       ▼
        │   [Leave Verification Panel]
        │     ● List of submissions from guardians
        │     ● View details + supporting documents
        │     ● Approve / Reject
        │
        └── [Class Recap]
              ● Filter: Today / This month / This semester
              ● View student list + attendance status
              ● Export PDF/Excel
```

---

## 4. Duty Teacher Flow — Real-Time Monitoring

```
[Duty Teacher logs in via SSO]
        │
        ▼
[Duty Teacher Dashboard]
        │
        ▼
[Daily Monitoring]
        │
        ▼
[Quick Filter — Select Class]
        │
        ▼
[Display:]
  ● Total students in class
  ● ✅ Already attended (Present)
  ● ⚠️ Late
  ● ❌ Not yet attended / Absent
  ● Each student's attendance time
```

---

## 5. Admin Flow — Central Management

```
[Admin logs in via SSO]
        │
        ▼
[Admin Dashboard — School Statistics]
        │
        ├── [Master Data Management]
        │     ├── Students   → CRUD + Import/Export Excel
        │     ├── Teachers   → CRUD + Import/Export Excel
        │     ├── Guardians  → CRUD + Import/Export Excel
        │     └── Classes    → CRUD + Enrolment
        │
        ├── [Attendance & Leave Management]
        │     ├── View daily/monthly/semester recap
        │     ├── Correction / data override (bypass)
        │     └── Export PDF/Excel
        │
        ├── [Settings]
        │     ├── Attendance Hours → Open/Late/Close per day
        │     └── Academic Calendar → Holiday list
        │
        └── [Drill-Down Analytics]
              Level 1 → [Chart Entire School]
                    ↓ click
              Level 2 → [Chart per Class]
                    ↓ click
              Level 3 → [Statistics per Student]
                    ↓ click
              Level 4 → [Attendance Detail by Name]
```

### Import/Export Excel Features

| Feature | Detail |
|---|---|
| Template download | System provides Excel template with correct columns |
| Import | Upload Excel file → validation → batch insert/update |
| Export | Download master data or recap to Excel |
| Recommended library | PhpSpreadsheet (for Laravel) |

---

## 6. Report & Export Flow

```
[User: Admin / Homeroom Teacher / Duty Teacher]
        │
        ▼
[Menu: Reports]
        │
        ├── [Filter]
        │     ● Range: Daily / Monthly / Semester
        │     ● Class: (optional, filter)
        │     ● Status: (optional, filter)
        │
        └── [Export]
              ● PDF → for official printing
              ● Excel → for further data processing
```

---

## Interface Summary

| No | Interface | Role | Priority |
|---|---|---|---|
| 1 | SSO Login | All | P0 |
| 2 | Admin Dashboard | Admin | P0 |
| 3 | Master Data Management | Admin | P0 |
| 4 | Class Management & Enrolment | Admin | P0 |
| 5 | Time & Holiday Settings | Admin | P1 |
| 6 | Student Dashboard | Student | P0 |
| 7 | Live Attendance Form | Student | P0 |
| 8 | Guardian Dashboard | Guardian | P0 |
| 9 | Leave Submission Form | Guardian | P0 |
| 10 | Homeroom Teacher Dashboard | Homeroom Teacher | P0 |
| 11 | Leave Verification Panel | Homeroom Teacher | P1 |
| 12 | Duty Teacher Dashboard | Duty Teacher | P0 |
| 13 | Reports & Export | Admin, Homeroom Teacher, Duty Teacher | P1 |

# System Requirement Analysis

## General Description

**SMART Absen SMA UII** is an integrated student attendance information system based on responsive web designed to automate, secure, and centralize student attendance recording at SMA UII.

This system integrates:
- **Location tracking** (Geolocation API)
- **Biometric image capture** (WebRTC camera)
- **Client-side image compression** for bandwidth efficiency
- **Single Sign-On (SSO)** as the foundation of centralized digital identity

### Load Target

| Metric | Value |
|---|---|
| Peak concurrent users (CCU) | 750+ students |
| Peak time | 06:30 – 07:00 WIB |
| Photo size per student (after compression) | Max 20 KB |
| Total payload for 760 simultaneous students | ~15.2 MB |

---

## Functional Requirements

### 1. Entry Module — Universal SSO Portal

| ID | Requirement |
|---|---|
| F-01 | The system must provide a single authentication (login) portal for all user categories |
| F-02 | The system must automatically detect the role after successful login and redirect to the respective dashboard |
| F-03 | The login architecture must be designed as an **Identity Provider (IdP)** using Laravel Sanctum so it can be used by other foundation applications in the future |

### 2. Admin Module — Central Manager

| ID | Requirement |
|---|---|
| F-04 | **Master Data Management:** Single & bulk CRUD (Excel upload/download) for Students, Teachers, Guardians entities |
| F-05 | **Class Enrolment:** Create classes, assign homeroom teachers, bulk enroll students into classes |
| F-06 | **Attendance & Leave Management:** Full CRUD access rights for attendance data correction/override |
| F-07 | **Analytic Dashboard (4-Level Drill-Down):** Level 1 (school) → Level 2 (per class) → Level 3 (per student) → Level 4 (detail by name) |

### 3. Student Module

| ID | Requirement |
|---|---|
| F-08 | **Live Attendance:** Take selfie (camera required) + lock real-time GPS coordinate point |
| F-09 | **Personal Report:** Attendance charts and recap with Daily/Monthly/Semester filters |

### 4. Guardian Module

| ID | Requirement |
|---|---|
| F-10 | **Multi-Child Profile:** One account can manage >1 student (siblings), with profile-switch feature |
| F-11 | **Digital Leave Submission:** Leave form (Sick/Event/Competition) + select child + duration + upload evidence (image/PDF) |

### 5. Homeroom Teacher Module

| ID | Requirement |
|---|---|
| F-12 | **Leave Verification Panel:** Incoming leave submission notifications from guardians, can Approve/Reject |
| F-13 | **Class Monitoring & Recap:** Attendance report for all students in their class (Daily/Monthly/Semester) |

### 6. Duty Teacher Module

| ID | Requirement |
|---|---|
| F-14 | **Live Monitoring Filtered:** Attendance recap of all students for the current day with quick filter per class |

---

## Non-Functional Requirements

### Performance

| ID | Requirement | Specification |
|---|---|---|
| NF-01 | **Frontend Image Compression** | Resolution 320x240, JPEG 90%, max 20 KB per photo, via JavaScript Canvas API |
| NF-02 | **Database max_connections** | Minimum 1000 concurrent connections |
| NF-03 | **Database Indexing** | Required on columns id_student, date, attendance_status, nisn, name |
| NF-04 | **Storage Offloading** | Prohibited from storing BLOB/Base64 in the database — use URL to external Object Storage |

### Usability

| ID | Requirement |
|---|---|
| NF-05 | **Responsive Web Design** — Students & Guardians optimized for mobile; Admin, Teachers for desktop |
| NF-06 | **No installation** — just a browser, no need to download from Play Store/App Store |
| NF-07 | **Instant updates** — every change is immediately available without manual updates |

### Security

| ID | Requirement |
|---|---|
| NF-08 | **SSO Tokenization** — secure token system to prevent login session manipulation |
| NF-09 | **Role-based middleware** — users cannot access URLs/manipulate data outside their access rights |
| NF-10 | **Triple-Layer Validation** before saving attendance — see security document |

### Reliability

| ID | Requirement |
|---|---|
| NF-11 | **PDF & Excel export** for Daily/Monthly/Semester reports |
| NF-12 | **SSD/NVMe server** to ensure I/O speed |

---

## Role vs Feature Matrix

| Feature | Admin | Student | Guardian | Homeroom Teacher | Duty Teacher |
|---|---|---|---|---|---|
| SSO Login | ✅ | ✅ | ✅ | ✅ | ✅ |
| CRUD Master Data | ✅ | — | — | — | — |
| Upload/Download Excel | ✅ | — | — | — | — |
| Class Enrolment | ✅ | — | — | — | — |
| Configure Attendance Hours | ✅ | — | — | — | — |
| Configure Holiday Calendar | ✅ | — | — | — | — |
| Live Attendance (Selfie+GPS) | — | ✅ | — | — | — |
| View Own Recap | ✅ | ✅ | — | — | — |
| Multi-Child Profile | — | — | ✅ | — | — |
| Submit Leave | — | — | ✅ | — | — |
| Verify Leave | ✅ | — | — | ✅ | — |
| Class Monitoring | ✅ | — | — | ✅ | ✅ |
| Quick Filter per Class | ✅ | — | — | ✅ | ✅ |
| Drill-Down Analytics | ✅ | — | — | — | — |
| Export PDF/Excel | ✅ | — | — | ✅ | ✅ |
| Bypass/Override Data | ✅ | — | — | — | — |

# Budget Plan, Timeline & Roadmap

## Budget Plan

### Total Estimate: **Rp 8,500,000**

| Component | Description | Cost |
|---|---|---|
| **Database Architecture & SSO** | ERD design, multi-user relations, self-managed SSO login foundation | Rp 1,800,000 |
| **Core Attendance Module** | Camera access feature (selfie) + GPS coordinate locking for students | Rp 1,500,000 |
| **Admin & Master Data Module** | CRUD (single & bulk Excel) for Students, Teachers, Classes data | Rp 1,000,000 |
| **Duty Teacher & Homeroom Teacher Module** | Real-time monitoring filter, leave verification system, class recap | Rp 1,000,000 |
| **Student & Guardian UI/UX Module** | Responsive dashboard, attendance charts, leave submission form | Rp 1,150,000 |
| **Object Storage Rental (1 year)** | 50–250 GB, S3-compatible | Rp 850,000 |
| **Server Setup & Deployment** | VPS configuration, database installation, application deployment | Rp 1,200,000 |

### Object Storage Breakdown

| Metric | Value |
|---|---|
| Capacity | 50–250 GB |
| Duration | 1 year |
| Estimated photos per day | 760 students × 20 KB = ~15 MB |
| Estimated per month | ~450 MB |
| Estimated per year | ~5.4 GB (still safe at 50 GB) |
| Recommended providers | Wasabi, Backblaze B2, or MinIO (self-hosted) |

---

## Development Timeline — 8 Weeks

```
Week           | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
               |---|---|---|---|---|---|---|---|
Analysis       |███|   |   |   |   |   |   |   |
Wireframing    |   |███|   |   |   |   |   |   |
ERD Design     |   |███|   |   |   |   |   |   |
SSO Integration|   |   |███|   |   |   |   |   |
Server Setup   |   |   |███|   |   |   |   |   |
Admin CRUD     |   |   |   |███|   |   |   |   |
Enrolment      |   |   |   |███|   |   |   |   |
Attendance     |   |   |   |   |███|   |   |   |
Duty Teacher   |   |   |   |   |   |███|   |   |
Integration    |   |   |   |   |   |███|   |   |
Reports        |   |   |   |   |   |   |███|   |
Testing        |   |   |   |   |   |   |███|   |
Mobile Opt     |   |   |   |   |   |   |███|   |
Bug Fixing     |   |   |   |   |   |   |   |███|
UAT            |   |   |   |   |   |   |   |███|
Deploy         |   |   |   |   |   |   |   |███|
Training       |   |   |   |   |   |   |   |███|
```

### Weekly Details

#### Week 1 — Requirement Analysis ✅ (Completed)
- Stakeholder interviews
- Functional & non-functional requirements documentation
- Business process analysis

#### Week 2 — Wireframing & ERD
- UI/UX wireframe design for 13 interfaces
- Finalize Entity Relationship Diagram
- Design review with team

#### Week 3 — SSO & Server
- SSO login implementation with Laravel Sanctum
- Development server setup
- Object Storage integration setup

#### Week 4 — Admin Dashboard
- Master data CRUD (Students, Teachers, Guardians)
- Excel Import/Export
- Class enrolment

#### Week 5 — Attendance Module
- Camera integration (WebRTC)
- Geolocation integration (Geolocation API)
- Client-side image compression
- Triple-layer validation

#### Week 6 — Duty Teacher & Integration
- Real-time monitoring dashboard
- Class filter
- Leave verification panel
- System integration

#### Week 7 — Reports & Testing
- PDF/Excel export
- Internal testing
- Mobile optimization

#### Week 8 — Finalization
- Bug fixing
- User Acceptance Testing (UAT)
- Production server deployment
- User training

---

## Future Roadmap

### Phase 1: SSO as Foundation Identity Provider

The login system from this application will be used as the main identity portal. In the future, all school applications will use a single SSO account.

```
Target: End of Year 1

┌─────────────────────────────────────────┐
│            SSO Identity Provider         │
│              (SMART Absen)               │
├─────────────────────────────────────────┤
│  ● e-Learning                           │
│  ● Financial / Tuition System           │
│  ● Digital Library                      │
│  ● Other foundation applications        │
└─────────────────────────────────────────┘
```

### Phase 2: "UII Satu Data" — Data Centralization

Realizing data centralization to prevent duplicate data across units. All student and teacher track records are integrated into one ID.

```
Target: End of Year 2

┌──────────────┐
│   One ID     │  ← NISN/NIP as single identity
└──────┬───────┘
       │
       ├── Attendance
       ├── Academic (Grades, Report Cards)
       ├── Administration (Tuition, Scholarships)
       ├── Library
       └── Extracurricular
```

### Phase 3: Automation & Smart Technology

| Feature | Description |
|---|---|
| **Geofencing Radius** | Automatic location radius restriction — students can only attend within the school area |
| **WhatsApp Gateway** | Real-time attendance notifications to guardians via WhatsApp |
| **Face Recognition** | Automatic face verification from attendance photos (AI/ML) |
| **Mobile Native App** | Native Android/iOS application with offline support |

---

## Notes for the Team

1. **P0 Priority** must be completed before the system can be used (SSO, Attendance, Admin)
2. **Budget is flexible** — components can be adjusted if scope changes
3. **8-week timeline** is an ideal target, may be delayed if technical issues arise
4. **Documentation** — after each module is completed, update the documentation in this repository

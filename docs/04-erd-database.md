# Database Design & Entity Relationship Diagram

## Design Philosophy

SMART Absen SMA UII database is designed with the following principles:

1. **Index-first** — all columns used in searches (WHERE, JOIN, ORDER BY) must have an index
2. **No BLOB** — image and document files are not stored in the database, only URL to Object Storage
3. **Strict enum** — using VARCHAR type + CHECK constraint (PostgreSQL does not have native ENUM like MySQL, so use Laravel enum casting + check constraint)
4. **JSONB for semi-structured data** — use JSONB (PostgreSQL) for columns with dynamic structure (e.g., attendance metadata, user preferences)
5. **Indexed relations** — every Foreign Key must have an index for JOIN performance

---

## Entity Relationship Diagram

See `../brief/ERD.png` for the ERD visualization.

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│    users     │     │    guardians     │     │   teachers   │
│──────────────│     │─────────────────│     │──────────────│
│ id (PK)      │────>│ id_guardian      │     │ id_teacher   │
│ username     │     │ id_user (FK)     │     │ id_user (FK) │
│ email        │     │ name             │     │ name         │
│ password     │     │ phone            │     │ teacher_code │
│ role         │     │ address          │     └──────┬───────┘
└──────────────┘                                    │
       │                                            │
       │                                            │
       │    ┌──────────────────┐                    │
       │    │    students      │                    │
       │    │──────────────────│                    │
       └────│ id_student (PK)  │                    │
            │ id_user (FK)     │                    │
            │ id_class (FK) ───│────────────────┐   │
            │ id_guardian (FK) │                │   │
            │ nis (unique)     │                │   │
            │ nisn (INDEX)     │                │   │
            │ name (INDEX)     │                │   │
            │ date_of_birth    │                │   │
            │ phone            │                │   │
            │ address          │                │   │
            │ enrollment_year  │                │   │
            │ status (INDEX)   │                │   │
            └────────┬─────────┘                │   │
                     │                          │   │
                     │ 1:N                      │   │
                     ▼                          │   │
            ┌──────────────────┐                │   │
            │   attendances    │                │   │
            │──────────────────│                │   │
            │ id_attendance(PK)│                │   │
            │ id_student(INDEX)│                │   │
            │ date (INDEX)     │                │   │
            │ time_in          │                │   │
            │ latitude         │                │   │
            │ longitude        │                │   │
            │ photo_url        │                │   │
            │ attendance_status│                │   │
            └──────────────────┘                │   │
                     │                          │   │
                     │ 1:N                      │   │
                     ▼                          │   │
            ┌──────────────────┐                │   │
            │  leave_requests  │                │   │
            │──────────────────│                │   │
            │ id_leave (PK)    │                │   │
            │ id_student(INDEX)│                │   │
            │ id_guardian      │                │   │
            │ category         │                │   │
            │ start_date       │                │   │
            │ end_date         │                │   │
            │ evidence_url     │                │   │
            │ approval_status  │                │   │
            └──────────────────┘                │   │
                                                 │   │
            ┌──────────────────┐                │   │
            │ school_classes   │◄────────────────┘   │
            │──────────────────│                     │
            │ id_class (PK)    │                     │
            │ id_teacher(FK) ──│─────────────────────┘
            │ class_name       │
            └──────────────────┘

            ┌──────────────────────┐
            │   duty_schedules     │
            │──────────────────────│
            │ id_schedule (PK)     │
            │ id_teacher (INDEX)   │
            │ duty_day (INDEX)     │
            └──────────────────────┘

            ┌──────────────────────────────┐
            │ attendance_time_settings     │
            │──────────────────────────────│
            │ id_setting (PK)              │
            │ day                          │
            │ open_time                    │
            │ late_threshold               │
            │ close_time                   │
            └──────────────────────────────┘

            ┌──────────────────────┐
            │ academic_calendars   │
            │──────────────────────│
            │ id_calendar (PK)     │
            │ date                 │
            │ description          │
            │ is_holiday           │
            └──────────────────────┘
```

---

## Table Specifications

### A. Core Tables (Authentication & Master)

#### 1. Table `users` — SSO Portal

| Column | Type | Constraint | Description |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| username | VARCHAR(50) | UNIQUE, INDEXED | Login via NISN/NIP |
| email | VARCHAR(100) | UNIQUE, NULLABLE | — |
| password | VARCHAR(255) | NOT NULL | Bcrypt hashed |
| role | ENUM('admin','student','teacher','guardian') | NOT NULL | — |
| timestamps | — | — | created_at, updated_at |

#### 2. Table `guardians`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_guardian | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_user | BIGINT UNSIGNED | FK → users.id | — |
| name | VARCHAR(100) | NOT NULL | — |
| phone | VARCHAR(20) | NULLABLE | — |
| address | TEXT | NULLABLE | — |

#### 3. Table `teachers`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_teacher | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_user | BIGINT UNSIGNED | FK → users.id | — |
| name | VARCHAR(100) | NOT NULL | — |
| teacher_code | VARCHAR(20) | UNIQUE | Teacher identity code |

#### 4. Table `school_classes`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_class | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_teacher | BIGINT UNSIGNED | FK → teachers.id | Homeroom Teacher |
| class_name | VARCHAR(50) | NOT NULL | Example: "X-A Reguler" |

#### 5. Table `students`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_student | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_user | BIGINT UNSIGNED | FK → users.id | — |
| id_class | BIGINT UNSIGNED | FK → school_classes.id | — |
| id_guardian | BIGINT UNSIGNED | FK → guardians.id | — |
| nis | VARCHAR(30) | UNIQUE | Student Identification Number |
| nisn | VARCHAR(30) | UNIQUE, **INDEXED** | National Student Identification Number |
| name | VARCHAR(100) | **INDEXED** | — |
| date_of_birth | DATE | NOT NULL | — |
| phone | VARCHAR(20) | NULLABLE | — |
| address | TEXT | NULLABLE | — |
| enrollment_year | YEAR | NOT NULL | — |
| status | ENUM('Active','Inactive') | **INDEXED** | — |

### B. Transaction Tables

#### 6. Table `attendances` — Heaviest Table

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_attendance | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_student | BIGINT UNSIGNED | FK → students.id, **INDEXED** | — |
| date | DATE | **INDEXED** | — |
| time_in | TIME | NOT NULL | — |
| latitude | VARCHAR(20) | NOT NULL | — |
| longitude | VARCHAR(20) | NOT NULL | — |
| photo_url | VARCHAR(500) | NOT NULL | URL to Object Storage, example: `attendances/2026/06/absen_123.jpg` |
| attendance_status | ENUM('Present','Late','Absent') | **INDEXED** | — |

> **Warning:** This table will have the most rows. **REQUIRED** to use composite index on `(id_student, date)`.

#### 7. Table `leave_requests`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_leave | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_student | BIGINT UNSIGNED | FK → students.id, **INDEXED** | — |
| id_guardian | BIGINT UNSIGNED | FK → guardians.id | — |
| category | ENUM('Sick','Event','Competition') | NOT NULL | — |
| start_date | DATE | NOT NULL | — |
| end_date | DATE | NOT NULL | — |
| evidence_url | VARCHAR(500) | NULLABLE | URL to Object Storage |
| approval_status | ENUM('Pending','Approved','Rejected') | **INDEXED** | — |

#### 8. Table `duty_schedules`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_schedule | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_teacher | BIGINT UNSIGNED | FK → teachers.id, **INDEXED** | — |
| duty_day | ENUM('Monday','Tuesday','Wednesday','Thursday','Friday') | **INDEXED** | — |

### C. Configuration Tables

#### 9. Table `attendance_time_settings`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_setting | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| day | ENUM('Monday','Tuesday','Wednesday','Thursday','Friday','Saturday') | NOT NULL | — |
| open_time | TIME | NOT NULL | Example: 06:00 — students can start attendance |
| late_threshold | TIME | NOT NULL | Example: 07:00 — past this time status is 'Late' |
| close_time | TIME | NOT NULL | Example: 08:00 — attendance is locked |

#### 10. Table `academic_calendars`

| Column | Type | Constraint | Description |
|---|---|---|---|
| id_calendar | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| date | DATE | NOT NULL | — |
| description | VARCHAR(200) | NULLABLE | Example: "Eid al-Fitr", "Foundation Meeting" |
| is_holiday | BOOLEAN | NOT NULL | Active/Inactive |

---

## Index Strategy

```sql
-- Users
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_role ON users(role);

-- Students
CREATE INDEX idx_students_nisn ON students(nisn);
CREATE INDEX idx_students_name ON students(name);
CREATE INDEX idx_students_status ON students(status);

-- Attendances (critical table)
CREATE INDEX idx_attendances_student_date ON attendances(id_student, date);
CREATE INDEX idx_attendances_date ON attendances(date);
CREATE INDEX idx_attendances_status ON attendances(attendance_status);

-- Leave Requests
CREATE INDEX idx_leave_student ON leave_requests(id_student);
CREATE INDEX idx_leave_status ON leave_requests(approval_status);

-- Duty Schedules
CREATE INDEX idx_schedule_teacher ON duty_schedules(id_teacher);
CREATE INDEX idx_schedule_day ON duty_schedules(duty_day);
```

---

## Important Notes

1. **No BLOB/Base64 in database** — photos and documents are only stored as URLs to Object Storage (S3-compatible)
2. **Composite index** on `attendances(id_student, date)` is crucial for daily/monthly recap query performance
3. **Enum vs Integer** — using ENUM because values are limited and more readable directly from the database
4. **Soft deletes** — consider adding `deleted_at` column for master tables (students, teachers, classes)

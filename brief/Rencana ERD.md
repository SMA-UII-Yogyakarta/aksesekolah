### **A. Core Tables (Authentication & Master)**

**1\. users table (SSO Portal)**

* id (PK)  
* username (VARCHAR, **INDEXED**, unique, for login via NISN/NIP)  
* email (VARCHAR, unique, *Nullable*)  
* password (VARCHAR, hashed)  
* role (ENUM: 'admin', 'siswa', 'guru', 'walimurid')

**2\. guardians table**

* id (PK)  
* user_id (FK to users)  
* name (VARCHAR)  
* phone (VARCHAR, *Nullable*)  
* address (TEXT, *Nullable*)

**3\. teachers table**

* id (PK)  
* user_id (FK to users)  
* name (VARCHAR)  
* teacher_code (VARCHAR, unique)

**4\. school_classes table**

* id (PK)  
* teacher_id (FK to teachers => *As Homeroom Teacher*)  
* class_name (VARCHAR, e.g., "X-A Reguler")

**5\. students table**

* id (PK)  
* user_id (FK to users)  
* class_id (FK to school_classes)  
* guardian_id (FK to guardians)  
* nis (VARCHAR, unique)  
* nisn (VARCHAR, **INDEXED**, unique)  
* name (VARCHAR, **INDEXED**)  
* date_of_birth (DATE)  
* phone (VARCHAR, *Nullable*)  
* address (TEXT, *Nullable*)  
* enrollment_year (YEAR)  
* status (ENUM: 'Active', 'Inactive', **INDEXED**)

  ### 

  ### **B. Transaction Tables (Daily Activity)**

**6\. attendances table (Heaviest Table)**

* id (PK)  
* student_id (FK to students, **INDEXED**)  
* date (DATE, **INDEXED**)  
* time_in (TIME)  
* latitude (VARCHAR)  
* longitude (VARCHAR)  
* photo_url (VARCHAR => *Only stores URL link to file in Object Storage, e.g., 'attendance/2026/06/absen_123.jpg'*)  
* attendance_status (ENUM: 'Present', 'Late', 'Absent', **INDEXED**)

**7\. leave_requests table**

* id (PK)  
* student_id (FK to students, **INDEXED**)  
* guardian_id (FK to guardians)  
* category (ENUM: 'Sick', 'Event', 'Competition')  
* start_date (DATE)  
* end_date (DATE)  
* document_evidence_url (VARCHAR => *URL link to doctor's note/letter file in Object Storage*)  
* approval_status (ENUM: 'Pending', 'Approved', 'Rejected', **INDEXED**)

**8\. duty_schedules table**

* id (PK)  
* teacher_id (FK to teachers, **INDEXED**)  
* duty_day (ENUM: 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', **INDEXED**)

  ### **C. Configuration Tables (Calendar)**

**9\. attendance_time_settings table**

* id (PK)  
* day (Enum: 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday')  
* check_in_open (TIME => e.g., 06:00 AM WIB. Students can only press the attendance button after this time).  
* late_threshold (TIME => e.g., 07:00 AM WIB. If attendance is at 07:01 AM, status is automatically 'Late').  
* check_in_close (TIME => e.g., 08:00 AM WIB. After this time, the attendance check-in button is automatically locked/hidden).

**10\. academic_calendars table**

* id (PK)  
* date (DATE)  
* description (VARCHAR, *Nullable* => e.g., "Eid al-Fitr", "Foundation Meeting")  
* is_holiday (Boolean/Enum => Active/Inactive)

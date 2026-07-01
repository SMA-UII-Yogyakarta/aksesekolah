# **SYSTEM REQUIREMENTS ANALYSIS REPORT**

**Project Name:** SMART ABSEN SMA UII

**Target Architecture:** Web-Based (Mobile Responsive & Desktop)

**Core Technology:** PHP Laravel & RDBMS (PostgreSQL/MariaDB/MySQL)

**Future Projection:** Single Sign-On (SSO) Gateway & Foundation of "UII Satu Data"

1. ## **GENERAL SYSTEM DESCRIPTION**

The Integrated Student Attendance Information System is a responsive web-based application designed to automate, secure, and centralize student attendance recording within the school institution under the foundation.

This system integrates location tracking technology (Geolocation API) and biometric image capture (Webcam) on the client side, optimized to handle high concurrency traffic of 760+ students simultaneously during school entry hours. The system uses Single Sign-On as the initial foundation for centralized digital identity standardization across the entire future foundation application ecosystem.

2. ## **FUNCTIONAL REQUIREMENTS**

Functional requirements are organized by user roles (Role-Based Access Control) and core system modules:

1. ### **Entry Point Module (Universal SSO Portal)**

* The system must provide a single authentication (login) portal for all user categories.  
* The system must automatically detect the account role after successful login, then redirect the user to their respective dashboard (Admin, Teacher, Student, or Guardian).  
* This login module architecture must be designed as an Identity Provider (IdP) using Laravel standards (Passport/Sanctum) so that it can be used by other foundation applications in the future.

2. ### **Access Role Module: Admin (Central Manager)**

* **Master Data Management (Individual & Bulk CRUD):** The system must be able to manage data (Create, Read, Update, Delete) for Student, Teacher, and Guardian entities, both individually and in bulk through Excel template Upload/Download features.  
* **Class Enrolment & Academic Mapping:** The system must provide functions to create Classes, appoint 1 Teacher as Homeroom Teacher (One-to-Many with students), and enroll students into classes in bulk.  
* **Attendance & Leave Management (Bypass System):** Admin has full access rights to perform CRUD on daily, monthly, and semester attendance and leave data as a correction/override function in case of technical issues.  
* **Advanced Drill-Down Analytics Dashboard:** The Admin Dashboard must display real-time interactive attendance statistics with the following hierarchy:  
  * *Level 1:* Overall School Statistics Chart.  
  * *Level 2:* Per-Class Statistics Chart.  
  * *Level 3:* Per-Student Statistics within the class.  
  * *Level 4:* Attendance Details by Name (Daily, Monthly, Semester).

3. ### **Access Role Module: Student**

* **Live Attendance (Camera & GPS):** The system must provide an attendance interface that requires students to take a selfie and lock geolocation coordinates (Latitude & Longitude) in real-time.  
* **Personal Attendance Report:** Provide self-service attendance charts and recaps filterable by Daily, Monthly, and Semester ranges.

4. ### **Access Role Module: Guardian**

* **Business Rule (One-to-Many):** One Guardian account can oversee more than one student (siblings) within the same institution.  
* **Multi-Child Profile:** The Guardian Dashboard must provide a switch-profile feature (child selection) to view each child's attendance recap separately (Daily, Monthly, Semester).  
* **Digital Leave Request:** Provide a leave request form for school absence (Categories: Sick, Event, Competition) with mandatory child profile selection, duration specification, and physical document upload (such as a doctor's letter or assignment letter) in image/PDF format.

5. ### **Access Role Module: Homeroom Teacher**

* **Business Rule (One-to-Many):** One Homeroom Teacher is fully responsible for the data recap of many students within one class scope under their supervision.  
* **Leave Verification Panel:** The system must display an inbox of leave request notifications from Guardians under their supervision, for subsequent approval (Approve) or rejection (Reject).  
* **Class Monitoring & Recap:** Homeroom Teachers can view and explore attendance reports for all students in their class on a Daily, Monthly, or Semester basis.

6. ### **Access Role Module: Duty Teacher**

* **Filtered Live Monitoring:** Duty teachers can view the attendance recap of all school students on the current day, equipped with a quick filter feature by class to facilitate physical field checks.

3. ## **NON-FUNCTIONAL REQUIREMENTS**

1. ### **Performance & Network Concurrency Management**

* **Image Payload Optimization (Frontend Compression):** The student attendance interface must integrate client-side compression scripts (Client-Side Rendering) using JavaScript (HTML5 Canvas/WebcamJS) to limit camera capture resolution to 320x240 pixels with 90% JPEG quality.  
* **File Size Limit:** The resulting photo conversion must be in Base64 text format with a **maximum file size of \< 20 KB** per student. This ensures that total incoming data when 760 students attend simultaneously does not exceed **\~15.2 MB**, thus saving the school's internal bandwidth.  
* **Database Tuning:** The web server and relational database settings on the institution's internal server must raise the maximum connection parameter (`max_connections`) to at least **1000 concurrent connections** to avoid system failure (connection timeout / Too Many Connections) during peak hours (06:30 \- 07:00 AM WIB).

2. ### **Interface Design (Usability)**

* **Responsive Web Design:** The application display must flexibly adapt to device screen sizes. Usage by Students and Guardians is optimized for mobile display (Smartphone), while Admin, Homeroom Teacher, and Duty Teacher usage is optimized for Desktop/Tablet display.

3. ### **Security & Authentication**

* **Secure Access Session:** SSO authentication is supported by a secure tokenization system to prevent login session manipulation.  
* **Access Rights Restriction:** The system must ensure that users cannot access URLs or manipulate data beyond their role access rights through middleware protection at the Laravel code level.

4. ### **Reliability & Data Export**

* **Report Export Compatibility:** All recap reports (Daily, Monthly, Semester) at Admin, Homeroom Teacher, and Duty Teacher levels must be accurately downloadable into universal document formats: **PDF** (for official printed documents) and **Excel** (for further data processing).  
* **Internal Server Readiness:** The system must run stably on the institution's local/standalone server infrastructure with SSD/NVMe-based storage to guarantee data write speed (I/O operations) for attendance image Base64 documents.

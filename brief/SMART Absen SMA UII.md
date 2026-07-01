**1\. Program Flow (User Scenario)**  
This system uses Role-Based Access Control with a single entry point through SSO.

* **Entry Point (SSO Login):** All users (Admin, Teacher, Student, Guardian) log in through a single institutional login portal. The system will automatically detect the user's role and redirect them to their respective dashboard.  
* **Student Flow:** Login => Open Attendance menu => System requests Camera & Location access => Take a selfie => System records time & coordinates => Done.  
* **Guardian Flow:** Login => Open Dashboard to view child's attendance chart => If child is absent, open Leave Request menu => Fill form (Sick/Event/Competition), duration, and upload evidence/letter => Submit request.  
* **Homeroom Teacher Flow:** Login => Open leave notification inbox => Verify and Update student leave status => Monitor daily recap of their assigned class.  
* **Duty Teacher Flow:** Login => Open Daily Monitoring menu => Filter by class being monitored => View list of students who haven't attended/are absent.  
* **Admin Flow:** Login => Manage master data (Students, Teachers, Guardians, Classes) in bulk or individually => Manage attendance and leave records fully => Set digital attendance gate open/close schedule and register academic holiday calendar.

**2\. Total Number of Interfaces**  
The application will have a responsive design (optimized for Mobile & Desktop) with approximately 13 Main Interfaces:

* **Universal Login Page:** Primary portal for SSO integration.  
* **Admin Dashboard:** School statistics summary (today's total attendance, absence chart).  
* **Master Data Management (Admin):** CRUD module for Student, Teacher, and Guardian data (including Excel Upload/Download for bulk data).  
* **Class & Enrolment Management (Admin):** Interface for mapping Homeroom Teachers and enrolling students into their respective classes.  
* **Time & Holiday Settings (Admin):** Dedicated interface for setting opening time, late threshold, daily attendance closing time, and registering red dates/academic holidays.  
* **Student Dashboard:** Personal attendance summary.  
* **Live Attendance Form (Student):** Dedicated page with camera viewfinder and geolocation map.  
* **Guardian Dashboard:** Attendance report specific to their child/children.  
* **Leave Request Form (Guardian):** Leave request input form with file upload feature.  
* **Homeroom Teacher Dashboard:** Attendance report specific to one class.  
* **Leave Verification Panel (Homeroom Teacher):** Interface for approving/rejecting leave requests from Guardians.  
* **Duty Teacher Dashboard:** Real-time attendance monitoring screen with class/day filter.  
* **Reports & Export (Global):** Dedicated interface for printing Daily/Monthly/Semester recaps to PDF/Excel format (accessible by Admin, Homeroom Teacher, and Duty Teacher).

**3\. Tools in Each Interface**

* **Live Attendance Form:** Uses HTML5 Geolocation API to obtain coordinate points (Latitude/Longitude) and WebRTC/MediaDevices API to capture photos directly from the device camera (preventing photo upload from gallery).  
* **Master Data Management:** Uses DataTables library for search, sort, and pagination of large data, along with parser tools for bulk data import/export.  
* **Dashboard & Reports:** Uses data visualization libraries (such as Chart.js or ApexCharts) to display attendance trend charts.  
* **Leave Form:** File Uploader component with document type validation (JPG, PNG, PDF) and file size limits.  
* **Frontend Image Compression:** Implementation of JavaScript-based image compression (Canvas API) on the Live Attendance Form interface to reduce selfie size to a maximum of 200 KB just before sending to the server.  
* **External Object Storage (S3 Protocol):** The system will use separate S3-compatible cloud storage specifically for storing static media files. This includes attendance photo files and uploaded document evidence on the Leave Request Form.

**4\. Infrastructure & Performance Specifications**

* **Concurrent User (CCU) Target:** The server infrastructure (VPS) MUST BE ABLE and optimized to serve queues of up to 750 concurrent users during the critical morning attendance period (06:30 \- 07:00 AM WIB) without experiencing Gateway Timeout.  
* **Index-Based Database Optimization:** The main database engine (MySQL/PostgreSQL) must use Database Indexing on relational and primary search columns (such as student_id, date, and attendance_status).  
* **Database Storage Offloading:** The database is strictly prohibited from storing image files as BLOB or LongText (Base64) data types. The database is purely used to store URLs pointing to file locations in the external Object Storage.

**5\. Logic Security & Time Geofencing**

* **Dynamic Configuration (Anti-Hardcode):** The system is not permitted to use static time logic (hardcoded) in the source code. All opening times, closing times, and late thresholds must be pulled from the attendance_time_settings database table to remain flexible to school schedule changes.  
* **Triple-Layer Validation:** Before saving attendance data, the backend must perform three layers of validation:  
  1. *Academic Calendar Validation:* Reject attendance if the system date matches holiday data in the academic_calendars table.  
  2. *Active Day Validation:* Reject attendance on weekends (Saturday/Sunday), unless set as active by Admin.  
  3. *Time Range Validation:* Reject access if before opening time, record 'Present' if on time, record 'Late' if beyond the tolerance threshold, and lock access (Disabled) if past operational closing time.

**6\. Advantages & Disadvantages (Web-Based vs Native App)**

* **Advantages of Web-Based (Mobile Responsive):**  
  * **No Installation:** Users do not need to download an app from Play Store/App Store that consumes smartphone internal memory. Just open a browser and log in via SSO.  
  * **Instant Updates:** Every fix or new feature is immediately available without needing to update the app.  
* **Disadvantages:**  
  * **Browser & Internet Dependence:** No offline mode as strong as native apps. If the internet signal is lost while capturing geolocation, attendance submission may fail.  
  * **Device Permissions:** Inexperienced users may accidentally block location/camera access permissions in their browser, requiring additional education.

**6\. Budget Plan**

| Component | Description | Estimated Cost |
| :---- | :---- | :---- |
| **Database Architecture & SSO** | ERD design, multi-user relations, and building a standalone SSO login foundation. | Rp 1,800,000 |
| **Main Attendance Module** | Development of camera access (selfie) and GPS coordinate locking features for students. | Rp 1,500,000 |
| **Admin & Master Data Module** | Creation of CRUD interface (including Excel import/export) for Student, Teacher, and Class data. | Rp 1,000,000 |
| **Duty Teacher & Homeroom Teacher Module** | Creation of real-time monitoring filter, leave verification system, and class recap. | Rp 1,000,000 |
| **Student & Guardian UI/UX Module** | Creation of responsive dashboard, attendance charts, and leave request form. | Rp 1,150,000 |
| **External Object Storage Rental** | Annual External Object Storage rental (50 GB \- 250 GB capacity) | Rp 850,000 |
| **Internal Server Setup & Deployment** | Configuration, database installation, and application deployment to the school's own server. | Rp 1,200,000 |
| **TOTAL ESTIMATE** |  | **Rp 8,500,000** |

### **7\. Timeline (Target 8 Weeks / 2 Months)**

| Week | Week 1 | Week 2 | Week 3 | Week 4 | Week 5 | Week 6 | Week 7 | Week 8 |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **Requirements Analysis** |   |   |   |   |   |   |   |   |
| **Wireframing (UI/UX Design)** |   |   |   |   |   |   |   |   |
| **Database ERD Design** |   |   |   |   |   |   |   |   |
| **SSO Integration** |   |   |   |   |   |   |   |   |
| **Server/Environment Setup** |   |   |   |   |   |   |   |   |
| **Admin Dashboard (Master Data CRUD)** |   |   |   |   |   |   |   |   |
| **Class & Enrolment Module (Student, Teacher, Homeroom Relations)** |   |   |   |   |   |   |   |   |
| **Student Attendance (Geolocation & Camera)** |  |  |  |  |  |  |  |  |
| **Duty Teacher & Homeroom Verification Module** |   |   |   |   |   |   |   |   |
| **System Integration** |   |   |   |   |   |   |   |   |
| **Report/Export Features (Daily/Monthly/Semester)** |   |   |   |   |   |   |   |   |
| **Internal Testing** |   |   |   |   |   |   |   |   |
| **Mobile Display Optimization** |   |   |   |   |   |   |   |   |
| **Bug Fixing** |   |   |   |   |   |   |   |   |
| **User Acceptance Testing (UAT)** |   |   |   |   |   |   |   |   |
| **Production Server Deployment** |   |   |   |   |   |   |   |   |
| **User Training/Guide** |   |   |   |   |   |   |   |   |

### 

### **8\. Future Roadmap**

This attendance system is projected as the initial foundation for the foundation's centralized digital transformation, with the following development roadmap:

* **Phase 1: Independent Foundation SSO Establishment** The login system from this application will become the main identity portal (Identity Provider). Going forward, all school applications (e-Learning, Finance, Library) will use a single SSO account from this system.  
* **Phase 2: "UII Satu Data" Milestone** Realizing data centralization to prevent duplicate data across units. All student and teacher records (attendance, academic, administrative) will be integrated under one ID.  
* **Phase 3: Automation & Smart Technology** Tighter geofencing (automatic location radius restrictions), and WhatsApp Gateway for real-time attendance notifications to guardians.

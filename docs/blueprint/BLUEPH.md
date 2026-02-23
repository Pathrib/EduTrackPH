UPDATED BLUEPRINT: Philippine School Attendance System v2.1
With Flexible Gates, ID Generator & Enhanced Installation
📑 TABLE OF CONTENTS
System Overview

Technical Architecture

Database Design (Enhanced)

Application Modules

Installation Wizard (Updated)

Flexible Gate Configuration (NEW)

ID Card Generator System (NEW)

Photo Management (3 Sizes)

Parent Notification System

Reports & Analytics

Promotion Management

Security & Audit Trail

Sync Module (Enhanced)

Backup & Recovery

Mobile App Strategy

Development Timeline

Budget Breakdown

1. SYSTEM OVERVIEW
1.1 Core Philosophy: OFFLINE-FIRST + FLEXIBLE + SCALABLE
Lahat ng PC ay may sariling local database (SQLite) - OFFLINE READY

AUDIT LOGGING para sa lahat ng changes - Walang mawawala

PARENT NOTIFICATIONS via Firebase (FREE for 10K students)

ENCRYPTED SYNC para sa data security

FLEXIBLE GATES - 1, 2, o 3 gates depende sa laki ng school

ID GENERATOR - Built-in ID card designer with PNG export

1.2 School Levels (Mutually Exclusive)
Level	Grade/Year Levels	Sections	Adviser Type
Elementary	Grade 1-6	Per Grade Level	Class Adviser
High School	Grade 7-10 (JHS), Grade 11-12 (SHS)	Per Grade/Strand	Section Adviser
College	1st Year - 5th Year	Per Course/Year	Program Chair/Professor
1.3 Session Schedules with Holiday Support
text
MORNING SESSION
├── Time In: 6:30 AM - 8:00 AM (grace period until 7:30)
├── Late: After 7:30 AM
├── Time Out: 11:00 AM - 12:00 PM
└── Early Out Grace: 10:45 AM (no penalty if after this)

AFTERNOON SESSION
├── Time In: 12:30 PM - 1:30 PM
├── Late: After 1:00 PM
├── Time Out: 4:00 PM - 5:00 PM
└── Early Out Grace: 3:45 PM

EVENING SESSION (College only)
├── Time In: 5:30 PM - 6:30 PM
├── Late: After 6:00 PM
├── Time Out: 8:30 PM - 9:30 PM
└── Early Out Grace: 8:15 PM
2. TECHNICAL ARCHITECTURE
2.1 Complete Application Structure
text
SCHOOL ATTENDANCE SYSTEM v2.1
│
├── [APP 1] EduTrackPH.Core (Shared Library)
│   ├── Models/
│   │   ├── Base/
│   │   │   ├── Person.cs
│   │   │   └── Entity.cs
│   │   ├── Student.cs
│   │   ├── Teacher.cs
│   │   ├── Staff.cs
│   │   ├── Section.cs
│   │   ├── AttendanceLog.cs
│   │   ├── RFIDCard.cs
│   │   ├── AuditLog.cs
│   │   ├── ParentNotification.cs
│   │   ├── SchoolCalendar.cs
│   │   ├── User.cs
│   │   ├── IDTemplate.cs           ← NEW
│   │   ├── IDCardExport.cs         ← NEW
│   │   └── GateConfiguration.cs     ← NEW
│   │
│   ├── Data/
│   │   ├── DatabaseContext.cs
│   │   ├── Repository.cs
│   │   └── Migrations/
│   │
│   ├── Services/
│   │   ├── Attendance/
│   │   ├── Report/
│   │   ├── Promotion/
│   │   ├── Sync/
│   │   ├── Notification/
│   │   ├── Security/
│   │   ├── Backup/
│   │   ├── IDGeneratorService.cs    ← NEW
│   │   ├── PhotoProcessor.cs        ← NEW
│   │   └── GateConfigService.cs     ← NEW
│   │
│   └── Utils/
│       ├── RFIDHelper.cs
│       ├── ImageHelper.cs           ← NEW (for 3 photo sizes)
│       └── FileExporter.cs          ← NEW (for PNG export)
│
├── [APP 2] EnrollmentSystem (PC1)
│   ├── Forms/
│   │   ├── MainDashboard.cs
│   │   ├── EnrollmentForm.cs
│   │   ├── SectionManagement.cs
│   │   ├── RFIDCardManagement.cs
│   │   ├── ReportsViewer.cs
│   │   ├── PromotionWizard.cs
│   │   ├── AuditLogViewer.cs
│   │   ├── UserManagement.cs
│   │   ├── SchoolCalendarForm.cs
│   │   ├── BackupRestoreForm.cs
│   │   ├── IDTemplateDesigner.cs     ← NEW
│   │   ├── IDBatchExport.cs          ← NEW
│   │   ├── PhotoCapture.cs           ← NEW (webcam + upload)
│   │   └── GateSetupWizard.cs        ← NEW
│   │
│   └── Assets/
│       ├── IDTemplates/               ← NEW
│       └── ExportedIDs/               ← NEW
│
├── [APP 3] GateMonitor (PC2, PC3, etc.)
│   ├── Forms/
│   │   ├── FullScreenDisplay.cs
│   │   ├── RFIDReader.cs
│   │   ├── TemporaryCardForm.cs
│   │   ├── Settings.cs
│   │   └── SyncManager.cs
│   │
│   └── Data/
│       ├── LocalAttendance.db
│       └── GateConfig.json            ← NEW (pwedeng 1-3 gates)
│
└── [APP 4] ParentNotificationService
    ├── FirebaseIntegration.cs
    └── SMSService.cs (Optional)
3. DATABASE DESIGN (ENHANCED)
3.1 Core Tables with New Fields
sql
-- ELEMENTARY STUDENTS TABLE (with 3 photo sizes)
CREATE TABLE elem_students (
    student_id INTEGER PRIMARY KEY,
    rfid_uid TEXT UNIQUE,
    lrn TEXT,
    first_name TEXT,
    last_name TEXT,
    middle_name TEXT,
    grade_level INTEGER,
    section_id INTEGER,
    parent_name TEXT,
    parent_contact TEXT,
    parent_email TEXT,
    parent_firebase_token TEXT,
    address TEXT,
    enrollment_date DATE,
    status TEXT DEFAULT 'Active',
    
    -- THREE PHOTO SIZES (NEW)
    photo_high_res BLOB,    -- 450x450 PNG (for ID)
    photo_medium_res BLOB,  -- 200x200 JPEG (for Gate)
    photo_low_res BLOB,     -- 100x100 JPEG (for Form)
    
    photo_updated_at DATETIME,
    photo_source TEXT, -- 'WEBCAM' or 'UPLOAD'
    
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME,
    updated_by INTEGER,
    
    FOREIGN KEY (section_id) REFERENCES elem_sections(section_id)
);

-- GATE CONFIGURATION TABLE (NEW)
CREATE TABLE gate_configuration (
    gate_id INTEGER PRIMARY KEY,
    gate_name TEXT, -- 'Main Gate', 'Back Gate', etc.
    gate_location TEXT,
    pc_name TEXT, -- Para malaman kung saang PC ito
    allow_elementary BOOLEAN DEFAULT 1,
    allow_highschool BOOLEAN DEFAULT 1,
    allow_college BOOLEAN DEFAULT 1,
    allow_teachers BOOLEAN DEFAULT 1,
    allow_staff BOOLEAN DEFAULT 1,
    is_active BOOLEAN DEFAULT 1,
    last_sync DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ID TEMPLATES TABLE (NEW)
CREATE TABLE id_templates (
    template_id INTEGER PRIMARY KEY,
    template_name TEXT, -- 'Student ID Front', 'Student ID Back', etc.
    template_type TEXT, -- 'FRONT' or 'BACK'
    target_level TEXT, -- 'ELEMENTARY', 'HIGH_SCHOOL', 'COLLEGE', 'TEACHER'
    background_image BLOB, -- Template PNG
    is_default BOOLEAN DEFAULT 0,
    orientation TEXT, -- 'PORTRAIT' or 'LANDSCAPE'
    width_mm REAL, -- 85.6mm (CR-80)
    height_mm REAL, -- 54mm (CR-80)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ID TEMPLATE FIELDS (NEW - for drag-drop positions)
CREATE TABLE id_template_fields (
    field_id INTEGER PRIMARY KEY,
    template_id INTEGER,
    field_name TEXT, -- 'STUDENT_NAME', 'ID_NUMBER', 'PHOTO', etc.
    field_type TEXT, -- 'TEXT', 'PHOTO', 'SIGNATURE'
    x_position REAL, -- % or pixels
    y_position REAL,
    width REAL,
    height REAL,
    font_name TEXT,
    font_size INTEGER,
    font_color TEXT,
    is_editable BOOLEAN DEFAULT 1,
    data_source TEXT, -- Which field from students table
    FOREIGN KEY (template_id) REFERENCES id_templates(template_id)
);

-- ID EXPORT LOG (NEW)
CREATE TABLE id_export_log (
    export_id INTEGER PRIMARY KEY,
    student_id INTEGER,
    filename_front TEXT,
    filename_back TEXT,
    export_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    exported_by INTEGER,
    batch_id TEXT, -- Para sa batch exports
    file_size_front INTEGER,
    file_size_back INTEGER,
    file_path TEXT
);

-- PHOTO HISTORY TABLE (NEW)
CREATE TABLE student_photos_history (
    history_id INTEGER PRIMARY KEY,
    student_id INTEGER,
    photo_high_res BLOB,
    photo_medium_res BLOB,
    photo_low_res BLOB,
    archived_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    archived_reason TEXT -- 'NEW_PHOTO', 'CORRECTION'
);
4. APPLICATION MODULES
4.1 ENROLLMENT APP (PC1) - Complete Features
A. Installation Wizard (Updated with Gate Config)
text
SCHOOL SETUP WIZARD v2.1
═══════════════════════════════════

STEP 1: School Information
├── School Name: [________________]
├── School ID: [________________]
├── Address: [________________]
└── Contact #: [________________]

STEP 2: Select School Levels (✓ lahat ng meron)
☑ ELEMENTARY (Grades 1-6)
☑ HIGH SCHOOL (Grades 7-12)
☑ COLLEGE (Tertiary)

STEP 3: Configure Gates
Ilan ang gate ng school? [2 GATES ▼]

GATE 1:
├── Name: [Main Gate]
├── PC Name: [PC2]
└── Pwedeng pumasok dito:
    ☑ Elementary
    ☑ High School
    ☑ College
    ☑ Teachers/Staff

GATE 2:
├── Name: [Back Gate]
├── PC Name: [PC3]
└── Pwedeng pumasok dito:
    ☐ Elementary
    ☑ High School
    ☐ College
    ☑ Teachers (HS only)

STEP 4: Create Admin Account
├── Username: [admin]
├── Password: [********]
└── Email: [admin@school.edu.ph]

STEP 5: Setup Complete
→ Database initialized
→ Gate configurations saved
→ Ready to install on gate PCs!
B. Enhanced Enrollment Form with Photo Capture
text
ENROLLMENT FORM v2.1
═══════════════════════════════════

Student Information
├── LRN: [______________]
├── Last Name: [______________]
├── First Name: [______________]
├── Grade Level: [Grade 1 ▼]
├── Section: [Section A ▼]

PHOTO CAPTURE (NEW)
┌─────────────────────────────────┐
│  [WEBCAM]    [UPLOAD]           │
│                                  │
│     ┌─────────────────┐         │
│     │                 │         │
│     │   PREVIEW       │         │
│     │   1.5x1.5"      │         │
│     │                 │         │
│     └─────────────────┘         │
│                                  │
│  [CROP] [ROTATE] [SAVE]         │
│                                  │
│  ✓ Auto-generate:                │
│    • 450x450 PNG (for ID)        │
│    • 200x200 JPG (for Gate)      │
│    • 100x100 JPG (for Form)      │
└─────────────────────────────────┘

Parent/Guardian Information
├── Name: [______________]
├── Contact #: [______________]
└── Preferred Notification: [Push ▼]

[SAVE & GENERATE ID]  [SAVE LATER]
5. FLEXIBLE GATE CONFIGURATION (NEW SECTION)
5.1 Supported Gate Setups
Setup	Description	Ideal For
1 Gate	Iisang entrance lang	Maliit na private school
2 Gates	Main gate + Back gate	Medium schools
3 Gates	Separate gates per level	Malaking schools
5.2 Configuration Options
javascript
// Sa Gate PC, may ganitong config file
{
  "gateId": "GATE2_BACK",
  "gateName": "Back Gate",
  "pcName": "PC3",
  
  // Allowed levels - pwedeng i-mix
  "allowedLevels": {
    "elementary": false,
    "highSchool": true,
    "college": false,
    "teachers": true,
    "teacherLevels": ["highSchool"] // HS teachers lang
  },
  
  // Sync settings
  "syncServer": "192.168.1.100",
  "syncInterval": 5, // minutes
  "syncFilter": {
    "onlyTheseLevels": ["highSchool"] // HS lang ida-download
  },
  
  // Display settings
  "displayMode": "fullscreen",
  "showRecentTaps": 10
}
5.3 Gate Assignment Matrix
Gate	Elementary	High School	College	Teachers	Staff
Gate 1 (Main)	✓	✓	✓	✓	✓
Gate 2 (Back)	✗	✓	✗	✓ (HS)	✗
Gate 3 (College)	✗	✗	✓	✓ (College)	✗
5.4 Automatic Sync Filtering
csharp
// Sa Gate 2 (Back Gate - HS lang)
public void SyncWithServer()
{
    var config = GetGateConfig();
    
    // Filter: HS lang
    var students = server.GetStudents(new Filter {
        Levels = new[] { "HIGH_SCHOOL" },
        IncludeTeachers = true,
        TeacherLevels = new[] { "HIGH_SCHOOL" }
    });
    
    SaveToLocalDatabase(students);
    
    // Hindi na dina-download ang Elem at College
    // Tipid sa storage at bandwidth!
}
6. ID CARD GENERATOR SYSTEM (NEW SECTION)
6.1 Overview
Ang ID Generator ay integrated sa enrollment form. Pwedeng:

Mag-upload ng template (PNG) para sa front at back

Drag-and-drop ng elements (photo, name, course, etc.)

I-assign kung anong fields ang nasa front/back

I-export as PNG (high quality, 300 DPI)

Batch export - buong section/course nang sabay-sabay

6.2 ID Template Designer UI
text
ID TEMPLATE DESIGNER
═══════════════════════════════════

Template: [Student ID Front ▼]  [LOAD]

┌─────────────────────────────────────────┐
│                                         │
│  ┌─────┐  JUAN DELA CRUZ               │
│  │     │  (draggable)                   │
│  │ PHOT│  ┌─────────────────────────┐   │
│  │(drag │  │ BSIT - 1st Year        │   │
│  │ resize│  │ (draggable)            │   │
│  └─────┘  └─────────────────────────┘   │
│                                         │
│  [SCHOOL LOGO]                          │
│  (fixed)                                │
│                                         │
└─────────────────────────────────────────┘

ELEMENTS PANEL                  PROPERTIES
┌─────────────┐                ┌─────────────┐
│ ☑ Photo     │                │ Field: NAME │
│ ☑ Name      │                │ Font: Arial │
│ ☑ Course    │                │ Size: 14pt  │
│ ☑ ID Number │                │ Color: Black│
│ ☑ Signature │                │ Position    │
│ ☑ QR Code   │                │ X: 120 Y:45 │
└─────────────┘                └─────────────┘

[SAVE TEMPLATE] [PREVIEW] [EXPORT PNG]
6.3 Photo Processing - 3 Sizes
javascript
// Automatic sa pag-upload/capture
function processStudentPhoto(imageFile) {
    return {
        // For ID Card (high quality, PNG)
        highRes: resizeImage(imageFile, 450, 450, 'PNG'),
        
        // For Gate Monitor (mabilis mag-load)
        mediumRes: resizeImage(imageFile, 200, 200, 'JPEG', 0.9),
        
        // For Form Preview (maliit lang)
        lowRes: resizeImage(imageFile, 100, 100, 'JPEG', 0.8)
    };
}

// Save sa database
db.students.update({
    photo_high_res: photos.highRes,  // BLOB
    photo_medium_res: photos.mediumRes,
    photo_low_res: photos.lowRes
});
6.4 ID Export - Manual at Batch
Manual Export (Per Student)
text
After mag-enroll:
┌─────────────────────────────────────┐
│ ID CARD PREVIEW                      │
├─────────────────────────────────────┤
│  FRONT                    BACK       │
│  ┌─────┐  Name           ┌─────┐    │
│  │photo│  Course         │sig  │    │
│  │     │  ID#            │     │    │
│  └─────┘                 └─────┘    │
│                                      │
│ Filename: juan_delacruz              │
│ [EXPORT PNG]                         │
│                                      │
│ ✓ Saved to:                          │
│ C:\EduTrackPH\ExportedIDs\           │
│   juan_delacruz_front.png            │
│   juan_delacruz_back.png             │
└─────────────────────────────────────┘
Batch Export
text
BATCH ID EXPORT
═══════════════════════════════════

Select Group:
○ Section: [BSIT-1A ▼] (35 students)
○ Course: [BSIT ▼] (180 students)
○ Year Level: [1st Year ▼] (95 students)

Export Options:
├── Folder: [BSIT-1A_IDs_2026-03-15]
├── Include: [FRONT & BACK]
└── Format: [PNG - 300 DPI]

[EXPORT ALL]

Progress: [████████░░] 80%

Result:
✓ 35 students processed
✓ 70 PNG files generated
📍 C:\EduTrackPH\ExportedIDs\BSIT-1A_IDs\
6.5 File Naming Convention
text
Format: [firstname]_[lastname]_[front/back].png

Examples:
├── juan_delacruz_front.png
├── juan_delacruz_back.png
├── maria_santos_front.png
└── maria_santos_back.png

For batch:
C:\EduTrackPH\ExportedIDs\BSIT-1A_IDs\
├── juan_delacruz_front.png
├── juan_delacruz_back.png
├── maria_santos_front.png
├── maria_santos_back.png
└── ... (68 more files)
6.6 Signature Integration
html
<!-- Signature Pad sa enrollment -->
<div class="signature-section">
  <h4>Signatures</h4>
  
  <div class="sig-row">
    <label>Student Signature:</label>
    <canvas id="sig-student" width="300" height="100"></canvas>
  </div>
  
  <div class="sig-row">
    <label>Registrar Signature:</label>
    <canvas id="sig-registrar" width="300" height="100"></canvas>
    <button onclick="loadDefault('registrar')">Use Default</button>
  </div>
  
  <div class="sig-row">
    <label>Principal Signature:</label>
    <canvas id="sig-principal" width="300" height="100"></canvas>
    <button onclick="loadDefault('principal')">Use Default</button>
  </div>
</div>
7. PHOTO MANAGEMENT (3 SIZES)
7.1 Photo Size Specifications
Use Case	Resolution	Format	File Size	Storage
ID Card	450x450	PNG	~50-100 KB	Database BLOB
Gate Monitor	200x200	JPEG	~10-15 KB	Database BLOB
Form Preview	100x100	JPEG	~3-5 KB	Database BLOB
7.2 Storage Calculation (10,000 students)
text
High Res: 10,000 × 75 KB = 750 MB
Medium Res: 10,000 × 12 KB = 120 MB
Low Res: 10,000 × 4 KB = 40 MB
TOTAL: ~910 MB (less than 1GB)
7.3 Photo History
sql
-- Keep old photos for audit trail
INSERT INTO student_photos_history 
SELECT * FROM student_photos WHERE student_id = 123;

-- Update with new photo
UPDATE students SET 
    photo_high_res = @newPhoto,
    photo_medium_res = @newMedium,
    photo_low_res = @newLow,
    photo_updated_at = CURRENT_TIMESTAMP
WHERE student_id = 123;
8. INSTALLATION SUMMARY (Updated)
8.1 Installation Flow Diagram
text
START
  │
  ▼
SELECT SCHOOL LEVELS
  │  (Elem, HS, College)
  ▼
SELECT NUMBER OF GATES
  │  (1, 2, or 3)
  ▼
CONFIGURE EACH GATE
  │  • Gate name
  │  • Allowed levels
  │  • PC assignment
  ▼
INSTALL ON PC1 (Enrollment)
  │  • Complete database
  │  • ID Templates
  │  • Gate configs saved
  ▼
INSTALL ON GATE PCs
  │  • Download config
  │  • Sync only allowed levels
  │  • Start monitoring
  ▼
DONE!
8.2 Sample Configurations
Maliit na Private School (1 Gate)
text
GATE 1 (Main)
├── Allowed: Elem, HS, College, Teachers
├── PC: PC2
└── Local DB: All tables
Medium School (2 Gates)
text
GATE 1 (Main)
├── Allowed: Elem, HS, College, Teachers
├── PC: PC2
└── Local DB: All tables

GATE 2 (Back)
├── Allowed: HS only + HS Teachers
├── PC: PC3
└── Local DB: hs_students, hs_teachers only
Malaking School (3 Gates)
text
GATE 1 (Elem)
├── Allowed: Elementary only
├── PC: PC2
└── Local DB: elem_ tables only

GATE 2 (HS)
├── Allowed: High School only
├── PC: PC3
└── Local DB: hs_ tables only

GATE 3 (College)
├── Allowed: College only
├── PC: PC4
└── Local DB: col_ tables only
9. DEVELOPMENT TIMELINE (Updated)
Phase	Duration	Features
Phase 1	3 weeks	Core Library + Database Design
Phase 2	3 weeks	Enrollment Forms + Photo Management
Phase 3	2 weeks	ID Generator System ← NEW
Phase 4	2 weeks	Flexible Gate Config ← NEW
Phase 5	2 weeks	Gate Monitor App
Phase 6	2 weeks	Sync Module + Reports
Phase 7	2 weeks	Testing + Deployment
Total: 16 weeks (4 months)

✅ FEATURE CHECKLIST (Updated)
Core Features (Done)
Offline-First Design

3 School Levels (Elem, HS, College)

RFID Integration

Time-in/Time-out with Late Detection

Holiday Calendar

Audit Trail

Parent Notifications

Promotion Management

Backup & Recovery

NEW Features (Added v2.1)
Flexible Gates - 1, 2, or 3 gates configurable

ID Generator System - Drag-drop designer

Photo Management - 3 sizes (450x450, 200x200, 100x100)

Webcam Capture - Direct photo taking

Batch Export - Export buong section/course

PNG Export - High quality, 300 DPI

Signature Integration - Student, Registrar, Principal

Gate Filtering - Bawat gate, pwedeng iba ang allowed levels

Smart Sync - Download only needed levels per gate

🎯 SUMMARY OF CHANGES (v2.1)
Change	Details
Flexible Gates	Pwedeng 1, 2, o 3 gates - configurable per school
ID Generator	Built-in ID designer with drag-drop
3 Photo Sizes	450x450 (ID), 200x200 (Gate), 100x100 (Form)
Webcam Support	Direct capture during enrollment
Batch Export	Export buong section nang sabay-sabay
PNG Export	High quality, saved to C:\EduTrackPH\ExportedIDs\
Gate Filtering	Bawat gate may sariling allowed levels
Smart Sync	Download lang ng level na kailangan ng gate

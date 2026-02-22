📚 UPDATED BLUEPRINT: Philippine School Attendance System v2.0
With Enhanced Features & Parent Notification System
📑 TABLE OF CONTENTS
System Overview

Technical Architecture

Database Design (Enhanced)

Application Modules

User Interface Design

RFID Integration

Parent Notification System

Reports & Analytics

Promotion Management

Security & Audit Trail

Sync Module (Enhanced)

Backup & Recovery

Mobile App Strategy

Installation & Setup

Development Timeline

Budget Breakdown

1. SYSTEM OVERVIEW
1.1 Core Philosophy: OFFLINE-FIRST + SECURE + SCALABLE
Lahat ng PC ay may sariling local database (SQLite) - OFFLINE READY

AUDIT LOGGING para sa lahat ng changes - Walang mawawala

PARENT NOTIFICATIONS via Firebase (FREE for 10K students)

ENCRYPTED SYNC para sa data security

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
1.4 Holiday/Event Calendar Integration
csharp
// Automatic adjustment for holidays
public class HolidayService
{
    public bool IsSchoolDay(DateTime date)
    {
        var holiday = GetHoliday(date);
        if (holiday != null)
        {
            if (holiday.event_type == "NO_CLASSES")
                return false;
            if (holiday.event_type == "HALF_DAY")
                return true; // But adjust attendance rules
        }
        return true; // Normal school day
    }
}
2. TECHNICAL ARCHITECTURE
2.1 Complete Application Structure
text
SCHOOL ATTENDANCE SYSTEM v2.0
│
├── [APP 1] SchoolAttendance.Core (Shared Library)
│   ├── Models/
│   │   ├── Person.cs (base class)
│   │   ├── Student.cs
│   │   ├── Teacher.cs
│   │   ├── Staff.cs
│   │   ├── Section.cs
│   │   ├── AttendanceLog.cs
│   │   ├── RFIDCard.cs
│   │   ├── AuditLog.cs          ← NEW
│   │   ├── ParentNotification.cs ← NEW
│   │   ├── SchoolCalendar.cs     ← NEW
│   │   └── User.cs               ← NEW
│   │
│   ├── Database/
│   │   ├── DatabaseContext.cs (SQLite)
│   │   ├── Repository.cs
│   │   ├── EncryptedBackup.cs    ← NEW
│   │   └── Migrations/
│   │
│   ├── Services/
│   │   ├── AttendanceCalculator.cs
│   │   ├── ReportGenerator.cs
│   │   ├── PromotionService.cs
│   │   ├── SyncService.cs
│   │   ├── AuditService.cs       ← NEW
│   │   ├── NotificationService.cs ← NEW
│   │   ├── HolidayService.cs      ← NEW
│   │   ├── SecurityService.cs     ← NEW
│   │   └── BackupService.cs       ← NEW
│   │
│   └── Utils/
│       ├── RFIDHelper.cs
│       ├── DateTimeHelper.cs
│       ├── PrintHelper.cs
│       ├── EncryptionHelper.cs    ← NEW
│       └── Logger.cs              ← NEW
│
├── [APP 2] EnrollmentSystem (PC1 - Admin/Registrar)
│   ├── Forms/
│   │   ├── MainDashboard.cs
│   │   ├── EnrollmentForm.cs
│   │   ├── SectionManagement.cs
│   │   ├── AdviserManagement.cs
│   │   ├── RFIDWriter.cs
│   │   ├── RFIDCardManagement.cs   ← NEW (Lost/Stolen)
│   │   ├── ReportsViewer.cs
│   │   ├── PromotionWizard.cs
│   │   ├── AuditLogViewer.cs       ← NEW
│   │   ├── UserManagement.cs        ← NEW
│   │   ├── SchoolCalendarForm.cs    ← NEW
│   │   ├── BackupRestoreForm.cs     ← NEW
│   │   └── NotificationSettings.cs  ← NEW
│   │
│   └── Assets/
│       ├── SchoolLogo/
│       ├── ReportTemplates/
│       ├── Backups/
│       └── Certificates/
│
├── [APP 3] GateMonitor (PC2, PC3, etc. - Gates)
│   ├── Forms/
│   │   ├── FullScreenDisplay.cs
│   │   ├── RFIDReader.cs
│   │   ├── TemporaryCardForm.cs    ← NEW
│   │   ├── Settings.cs
│   │   └── SyncManager.cs
│   │
│   └── Data/
│       ├── LocalAttendance.db (SQLite)
│       └── OfflineQueue/            ← NEW
│
└── [APP 4] ParentNotificationService (Background Service)
    ├── FirebaseIntegration.cs       ← NEW
    ├── SMSService.cs (Optional)     ← NEW
    └── EmailService.cs (Optional)   ← NEW
3. DATABASE DESIGN (ENHANCED)
3.1 Core Tables with Security & Notifications
sql
-- ELEMENTARY TABLES (prefix: elem_)
-- SAME AS BEFORE pero may additional fields

CREATE TABLE elem_students (
    student_id INTEGER PRIMARY KEY,
    rfid_uid TEXT UNIQUE,
    lrn TEXT, -- Learner Reference Number
    first_name TEXT,
    last_name TEXT,
    middle_name TEXT,
    grade_level INTEGER, -- 1 to 6
    section_id INTEGER,
    parent_name TEXT,
    parent_contact TEXT,
    parent_email TEXT,           -- NEW for notifications
    parent_firebase_token TEXT,  -- NEW for push notifications
    address TEXT,
    enrollment_date DATE,
    status TEXT DEFAULT 'Active', -- Active, Transferred, Graduated, Retained
    photo BLOB,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME,
    updated_by INTEGER,
    FOREIGN KEY (section_id) REFERENCES elem_sections(section_id)
);

-- AUDIT TRAIL TABLE (NEW)
CREATE TABLE audit_log (
    audit_id INTEGER PRIMARY KEY,
    user_id INTEGER,
    username TEXT,
    action_type TEXT, -- 'INSERT', 'UPDATE', 'DELETE', 'LOGIN', 'LOGOUT', 'SYNC', 'BACKUP', 'RESTORE'
    table_name TEXT,
    record_id INTEGER,
    old_value TEXT,   -- JSON format for complex data
    new_value TEXT,   -- JSON format for complex data
    ip_address TEXT,
    pc_name TEXT,
    user_agent TEXT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- PARENT NOTIFICATIONS TABLE (NEW)
CREATE TABLE parent_notifications (
    notification_id INTEGER PRIMARY KEY,
    student_id INTEGER,
    student_name TEXT,
    parent_contact TEXT,
    notification_type TEXT, -- 'LATE', 'ABSENT', 'TIME_IN', 'TIME_OUT', 'EARLY_OUT', 'HALF_DAY'
    message TEXT,
    sent_via TEXT, -- 'FIREBASE', 'SMS', 'EMAIL', 'ALL', 'PENDING'
    sent_timestamp DATETIME,
    status TEXT DEFAULT 'Pending', -- 'Pending', 'Sent', 'Failed', 'Delivered', 'Read'
    retry_count INTEGER DEFAULT 0,
    error_message TEXT,
    scheduled_time DATETIME,
    actual_sent_time DATETIME,
    delivery_receipt TEXT,
    FOREIGN KEY (student_id) REFERENCES elem_students(student_id)
);

-- SCHOOL CALENDAR TABLE (NEW)
CREATE TABLE school_calendar (
    event_id INTEGER PRIMARY KEY,
    event_date DATE UNIQUE,
    event_type TEXT, -- 'HOLIDAY', 'NO_CLASSES', 'HALF_DAY_MORNING', 'HALF_DAY_AFTERNOON', 'EVENT'
    event_name TEXT,
    description TEXT,
    affects_all_levels BOOLEAN DEFAULT 1,
    specific_level TEXT, -- NULL if affects all, or 'ELEMENTARY', 'HIGH SCHOOL', 'COLLEGE'
    created_by INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- RFID CARD HISTORY TABLE (NEW)
CREATE TABLE rfid_card_history (
    history_id INTEGER PRIMARY KEY,
    rfid_uid TEXT,
    person_id INTEGER,
    person_type TEXT, -- 'STUDENT', 'TEACHER', 'STAFF'
    action_type TEXT, -- 'ASSIGN', 'LOST', 'STOLEN', 'REPLACE', 'DEACTIVATE', 'TEMP_ASSIGN'
    previous_status TEXT,
    new_status TEXT,
    reason TEXT,
    action_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    action_by INTEGER,
    replaced_with_rfid TEXT, -- If replaced
    expiry_date DATE, -- For temporary cards
    notes TEXT
);

-- USERS TABLE (NEW)
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY,
    username TEXT UNIQUE,
    password_hash TEXT, -- BCrypt/SHA256
    full_name TEXT,
    email TEXT,
    role TEXT, -- 'SUPER_ADMIN', 'ADMIN', 'REGISTRAR', 'GATE_MONITOR', 'REPORT_VIEWER', 'TEACHER'
    permissions TEXT, -- JSON array: ["enroll", "edit_sections", "view_reports", etc.]
    is_active BOOLEAN DEFAULT 1,
    last_login DATETIME,
    last_ip TEXT,
    password_changed_at DATETIME,
    login_attempts INTEGER DEFAULT 0,
    locked_until DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by INTEGER
);

-- BACKUP HISTORY TABLE (NEW)
CREATE TABLE backup_history (
    backup_id INTEGER PRIMARY KEY,
    backup_name TEXT,
    backup_type TEXT, -- 'AUTO', 'MANUAL', 'USB'
    backup_location TEXT,
    file_size INTEGER,
    encryption_used BOOLEAN,
    checksum TEXT, -- For integrity check
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    restored_at DATETIME,
    restored_by INTEGER
);

-- SYNC LOG TABLE (NEW)
CREATE TABLE sync_log (
    sync_id INTEGER PRIMARY KEY,
    source_pc TEXT, -- 'PC1', 'GATE1', 'GATE2', etc.
    destination_pc TEXT,
    records_sent INTEGER,
    records_received INTEGER,
    conflicts_resolved INTEGER,
    sync_type TEXT, -- 'AUTO', 'MANUAL', 'USB'
    status TEXT, -- 'SUCCESS', 'PARTIAL', 'FAILED'
    error_message TEXT,
    started_at DATETIME,
    completed_at DATETIME,
    duration_seconds INTEGER
);

-- ATTENDANCE TABLE ENHANCED
CREATE TABLE elem_attendance (
    attendance_id INTEGER PRIMARY KEY,
    rfid_uid TEXT,
    person_type TEXT, -- 'Student' or 'Teacher'
    person_id INTEGER,
    timestamp DATETIME,
    action TEXT, -- 'TIME IN' or 'TIME OUT'
    session TEXT, -- 'Morning' or 'Afternoon'
    status TEXT, -- 'On Time', 'Late', 'Early Out', 'Half Day', 'Holiday'
    minutes_late INTEGER DEFAULT 0,
    minutes_early INTEGER DEFAULT 0,
    gate_location TEXT,
    verified_by TEXT, -- 'RFID', 'MANUAL', 'TEMP_CARD'
    sync_status TEXT DEFAULT 'Pending',
    sync_id INTEGER,
    is_holiday_override BOOLEAN DEFAULT 0,
    notes TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- HALF-DAY RECORDS TABLE (NEW)
CREATE TABLE half_day_records (
    record_id INTEGER PRIMARY KEY,
    student_id INTEGER,
    date DATE,
    session TEXT, -- 'Morning', 'Afternoon'
    reason TEXT, -- 'VALID_REASON', 'UNEXPLAINED'
    approved_by INTEGER,
    approved_at DATETIME,
    notified_parent BOOLEAN DEFAULT 0
);
4. APPLICATION MODULES (DETAILED FEATURES)
4.1 ENROLLMENT APP (PC1) - Complete Features
A. Installation Wizard (Enhanced)
text
SCHOOL SETUP WIZARD v2.0
═══════════════════════════════

Step 1 of 5: School Information
├── School Name: [________________]
├── School ID: [________________]
├── Address: [________________]
├── Region: [NCR ▼]
└── Contact #: [________________]

Step 2 of 5: Select School Level (CHOOSE ONE ONLY)
○ ELEMENTARY (Grades 1-6)
○ HIGH SCHOOL (Grades 7-12)
○ COLLEGE (Tertiary)

Step 3 of 5: Configure Sessions
☑ Morning Session (6:30 AM - 11:00 AM)
☑ Afternoon Session (12:30 PM - 4:00 PM)
☐ Evening Session (5:30 PM - 8:30 PM) [For College only]
└── Grace Period: [15] minutes before considered Early Out

Step 4 of 5: Create Admin Account
├── Username: [admin]
├── Password: [********]
├── Confirm: [********]
├── Email: [admin@school.edu.ph]
└── Security Question: [________________]

Step 5 of 5: Setup Complete
→ Database initialized: C:\SchoolData\
→ Admin account created
→ Default holiday calendar loaded (PH Holidays)
→ Backup schedule set: Daily at 11:00 PM
→ Ready to use!
B. Enhanced Enrollment Form with Parent Notifications
text
ELEMENTARY ENROLLMENT FORM v2.0
═══════════════════════════════

Student Information
├── LRN: [______________] (Required)
├── Last Name: [______________]
├── First Name: [______________]
├── Middle Name: [______________]
├── Grade Level: [Grade 1 ▼]
├── Section: [Section A ▼]
├── Date of Birth: [__/__/____]
└── Photo: [UPLOAD] [TAKE PICTURE]

Parent/Guardian Information
├── Name: [______________]
├── Relationship: [Mother ▼]
├── Contact #: [______________] (Required for SMS)
├── Email: [______________] (Optional)
├── Preferred Notification: [Push ▼]
│   ○ Push Notification (Free)
│   ○ SMS (₱1.00 each)
│   ○ Email (Free)
│   ○ All of the above
└── [SEND TEST NOTIFICATION]

RFID Assignment
├── RFID UID: [______________] [SCAN CARD]
├── Card Type: [Permanent ▼] (or Temporary)
├── Valid Until: [__/__/____] (If temporary)
└── [ENROLL & PRINT ID]

Emergency Contact (Optional)
├── Name: [______________]
├── Relationship: [______________]
└── Contact #: [______________]

[SAVE] [SAVE & ENROLL ANOTHER] [CANCEL]
C. RFID Card Management Module (NEW)
text
RFID CARD MANAGEMENT
═══════════════════════════════

CARD STATUS DASHBOARD
├── Total Active Cards: 8,450
├── Lost/Stolen Reported: 23
├── Temporary Cards Issued: 45
├── Expiring Soon: 12 (within 7 days)
└── Unassigned Cards: 150

SEARCH: [________________] [SEARCH BY UID/NAME]

┌─────────────────────────────────────────────────────┐
│ UID          │ Holder      │ Status   │ Valid Until │
├──────────────┼─────────────┼──────────┼─────────────┤
│ 04A2B3C5     │ Cruz, Juan  │ ACTIVE   │ N/A         │
│ 05D6E7F8     │ Santos, Ana │ LOST     │ N/A         │
│ 09G1H2I3     │ Reyes, Jose │ TEMP     │ 03/30/2026  │
│ 12J4K5L6     │ [UNASSIGNED]│ AVAILABLE│ N/A         │
└──────────────┴─────────────┴──────────┴─────────────┘

ACTIONS
├── [REPORT LOST CARD]    ├── [REPORT STOLEN]
├── [ISSUE TEMPORARY]     ├── [DEACTIVATE CARD]
├── [REPLACE CARD]        └── [VIEW HISTORY]

[CARD HISTORY for UID: 05D6E7F8]
────────────────────────────────
03/15/2026 - ASSIGNED to Santos, Ana (Student)
03/10/2026 - REPORTED LOST by Santos, Ana (Reason: Dropped)
03/01/2026 - ASSIGNED to Santos, Ana
02/28/2026 - ACTIVATED (New card)
D. School Calendar Management (NEW)
text
SCHOOL CALENDAR 2026
═══════════════════════════════

Month: [March 2026 ▼]  [LOAD PH HOLIDAYS] [ADD EVENT]

┌────┬──────────┬──────────────────┬────────────┬────────┐
│ Sun│ Mon│ Tue│ Wed│ Thu│ Fri│ Sat│ EVENT      │ TYPE  │
├────┼────┼────┼────┼────┼────┼────┼────────────┼────────┤
│    │    │    │    │    │    │ 1  │            │        │
│ 2  │ 3  │ 4  │ 5  │ 6  │ 7  │ 8  │            │        │
│ 9  │ 10 │ 11 │ 12 │ 13 │ 14 │ 15 │            │        │
│ 16 │ 17 │ 18 │ 19 │ 20 │ 21 │ 22 │            │        │
│ 23 │ 24 │ 25 │ 26 │ 27 │ 28 │ 29 │ MAUNDY THU │ HOLIDAY│
│ 30 │ 31 │    │    │    │    │    │ GOOD FRI   │ HOLIDAY│
└────┴────┴────┴────┴────┴────┴────┴────────────┴────────┘

ADD/EDIT EVENT
├── Date: [03/29/2026]
├── Event Type: [Holiday ▼]
├── Event Name: [Araw ng Kagitingan]
├── Affects: [All Levels ▼] (or specific level)
├── No Classes: [✅ YES]  [○ Half Day]
└── [SAVE] [DELETE]
E. User Management & Roles (NEW)
text
USER MANAGEMENT
═══════════════════════════════

Current Users: 12/50 licenses

┌──────────┬────────────┬─────────────┬──────────┬─────────┐
│ Username │ Full Name  │ Role        │ Last Login│ Status  │
├──────────┼────────────┼─────────────┼──────────┼─────────┤
│ admin    │ Cruz, Ana  │ SUPER_ADMIN │ Today    │ ACTIVE  │
│ registrar│ Santos, Ben│ REGISTRAR   │ Yesterday│ ACTIVE  │
│ gate1    │ Gate PC1   │ GATE_MONITOR│ 2 days   │ ACTIVE  │
│ teacher_j│ Reyes, Jose│ TEACHER     │ Never    │ PENDING │
└──────────┴────────────┴─────────────┴──────────┴─────────┘

[ADD USER] [EDIT] [DEACTIVATE] [RESET PASSWORD]

ROLE PERMISSIONS:
SUPER_ADMIN
├── Full system access
├── User management
├── System configuration
└── Audit log viewing

REGISTRAR
├── Student enrollment
├── Section management
├── RFID assignment
└── Basic reports

GATE_MONITOR
├── Monitor gate display
├── Issue temporary cards
└── No data modification
F. Audit Log Viewer (NEW)
text
AUDIT TRAIL LOGS
═══════════════════════════════

Filter: [All Actions ▼]  Date: [Last 7 days ▼]  User: [All ▼]

┌─────────────────────────────────────────────────────────────┐
│ TIME      │ USER    │ ACTION │ TABLE    │ DETAILS          │
├───────────┼─────────┼────────┼──────────┼──────────────────┤
│ 9:45:23 AM│ admin   │ UPDATE │ students │ Changed section  │
│           │         │        │          │ From: Sec A      │
│           │         │        │          │ To: Sec B        │
├───────────┼─────────┼────────┼──────────┼──────────────────┤
│ 9:30:15 AM│ gate1   │ INSERT │ attend   │ Time In: Juan    │
│           │         │        │          │ 7:35 AM (LATE)   │
├───────────┼─────────┼────────┼──────────┼──────────────────┤
│ 9:15:07 AM│ registrar│ DELETE │ temp_cards│ Expired temp    │
│           │         │        │          │ Card ID: TEMP123 │
└───────────┴─────────┴────────┴──────────┴──────────────────┘

[EXPORT LOGS] [VIEW DETAILS] [PRINT AUDIT REPORT]
4.2 GATE MONITOR APP (Enhanced Features)
A. Full-Screen Display with Notifications
text
┌─────────────────────────────────────────────────────────────┐
│              [🏫] SCHOOL NAME [⚡]                        │
│              ELEMENTARY GATE - MORNING SESSION              │
│                   7:45:23 AM (LATE WINDOW)                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                    ╔══════════════╗                         │
│                    ║   TAP CARD   ║                         │
│                    ╚══════════════╝                         │
│                                                              │
│    ┌──────────────────────────────────────────────────┐    │
│    │  RECENT ACTIVITY                                  │    │
│    │  7:44 AM - Juan Dela Cruz (G4-A) - LATE ⚠️       │    │
│    │          Parent notified via SMS ✓                │    │
│    │  7:42 AM - Maria Santos (G4-A) - ON TIME ✅       │    │
│    │  7:40 AM - Pedro Penduko (Teacher) - TIME IN ✅   │    │
│    │  7:38 AM - Ana Reyes (G4-B) - TEMP CARD ⏰       │    │
│    │          Valid until 5:00 PM                      │    │
│    └──────────────────────────────────────────────────┘    │
│                                                              │
│    TODAY'S STATS: Present: 245 | Late: 8 | Absent: 12      │
│    📍 GATE 1 - ELEMENTARY                    [🔴 OFFLINE]  │
└─────────────────────────────────────────────────────────────┘
B. When Card is Tapped (Enhanced Display)
text
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│   ┌─────────────┐                                           │
│   │    📸       │   JUAN DELA CRUZ                          │
│   │   PHOTO     │   Grade 4 - Section A                     │
│   │             │   Student ID: 2024-0001                   │
│   └─────────────┘   LRN: 123456789012                       │
│                                                              │
│   ═══════════════════════════════════════════════════════   │
│                                                              │
│             🟢 TIME IN (MORNING)                            │
│                7:45:23 AM                                    │
│                STATUS: LATE (15 mins)                       │
│                                                              │
│   ═══════════════════════════════════════════════════════   │
│                                                              │
│   Today's Logs:                                              │
│   Morning In: 7:45 AM (Just now) - LATE                     │
│                                                              │
│   PARENT NOTIFICATION: ✓ Sent via Firebase                   │
│   [SMS Backup: Queued - No credit]                          │
│                                                              │
│   [TEMP CARD OPTIONS] [MARK AS EXCUSED] [VIEW HISTORY]      │
└─────────────────────────────────────────────────────────────┘
C. Temporary Card Issuance (NEW)
text
TEMPORARY CARD ISSUANCE
═══════════════════════════════

Student Name: [______________] [SEARCH]

┌─────────────────────────────────────────────────────┐
│ MATCHING STUDENTS:                                   │
│ ○ Cruz, Juan (G4-A) - Forgot ID                      │
│ ○ Santos, Maria (G4-B) - Lost card (Reported)       │
│ ○ Reyes, Ana (G3-C) - New card not yet issued       │
└─────────────────────────────────────────────────────┘

Selected: Cruz, Juan (G4-A)

Temporary Card Details
├── Card UID: [TEMP_20250315_001] [GENERATE]
├── Valid Until: [Today ▼] [5:00 PM]
├── Reason: [Forgot ID ▼]
│   ○ Forgot ID
│   ○ Lost card (reported)
│   ○ Damaged card
│   ○ New student (card pending)
└── Notes: [________________]

[ISSUE TEMPORARY CARD] [CANCEL]

⚠️ Warning: Temporary cards expire at 5:00 PM
4.3 PARENT NOTIFICATION SERVICE (NEW APP)
A. Firebase Cloud Messaging Setup
csharp
// ParentNotificationService.cs - Runs as Windows Service
public class ParentNotificationService
{
    private FirebaseApp firebaseApp;
    private Timer notificationTimer;
    private Queue<Notification> pendingNotifications;
    
    public void Start()
    {
        // Initialize Firebase (FREE for 10K students)
        firebaseApp = FirebaseApp.Create(new AppOptions()
        {
            Credential = GoogleCredential.FromFile("firebase-key.json"),
            ProjectId = "school-attendance-ph"
        });
        
        // Check for pending notifications every minute
        notificationTimer = new Timer(ProcessNotifications, null, 0, 60000);
        
        Log("Notification Service Started");
    }
    
    private async void ProcessNotifications(object state)
    {
        var notifications = GetPendingNotifications();
        
        foreach (var notif in notifications)
        {
            try
            {
                // Get parent's Firebase token
                var token = GetParentToken(notif.student_id);
                
                if (!string.IsNullOrEmpty(token))
                {
                    // Send push notification (FREE)
                    await SendFirebaseNotification(token, notif);
                    notif.status = "Sent";
                    notif.sent_via = "FIREBASE";
                }
                else if (!string.IsNullOrEmpty(notif.parent_contact))
                {
                    // Fallback to SMS (if configured and has credits)
                    if (HasSMSCredits())
                    {
                        await SendSMS(notif.parent_contact, notif.message);
                        notif.status = "Sent";
                        notif.sent_via = "SMS";
                        DeductSMSCredit();
                    }
                    else
                    {
                        notif.status = "Failed - No SMS Credits";
                    }
                }
                
                UpdateNotificationStatus(notif);
            }
            catch (Exception ex)
            {
                LogError($"Failed to send notification: {ex.Message}");
                notif.retry_count++;
                if (notif.retry_count < 3)
                {
                    // Requeue for retry
                    RequeueNotification(notif);
                }
                else
                {
                    notif.status = "Failed - Max Retries";
                }
            }
        }
    }
    
    // Auto-generate notifications for attendance events
    public void QueueAttendanceNotification(AttendanceLog log)
    {
        var student = GetStudent(log.person_id);
        var message = GenerateMessage(log);
        
        var notification = new ParentNotification
        {
            student_id = student.student_id,
            student_name = student.full_name,
            parent_contact = student.parent_contact,
            notification_type = GetNotificationType(log),
            message = message,
            scheduled_time = DateTime.Now,
            status = "Pending"
        };
        
        SaveNotification(notification);
        pendingNotifications.Enqueue(notification);
    }
    
    private string GenerateMessage(AttendanceLog log)
    {
        return log.status switch
        {
            "Late" => $"⚠️ {log.student_name} was LATE today at {log.timestamp:hh:mm tt}. Minutes late: {log.minutes_late}",
            "Time In" => $"✅ {log.student_name} entered school at {log.timestamp:hh:mm tt}",
            "Time Out" => $"✅ {log.student_name} left school at {log.timestamp:hh:mm tt}",
            "Absent" => $"❌ {log.student_name} was ABSENT today. Please contact the school.",
            "Early Out" => $"⚠️ {log.student_name} left early at {log.timestamp:hh:mm tt}",
            "Half Day" => $"📝 {log.student_name} had a HALF DAY today. Only attended {log.session} session.",
            _ => $"📢 {log.student_name} - {log.status} at {log.timestamp:hh:mm tt}"
        };
    }
}
B. Android Parent App (Simplified)
xml
<!-- activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:src="@drawable/school_logo"
        android:layout_gravity="center"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Parent Portal"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_gravity="center"
        android:layout_marginBottom="24dp"/>

    <EditText
        android:id="@+id/lrnInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Child's LRN Number"
        android:inputType="number"
        android:padding="12dp"
        android:layout_marginBottom="12dp"/>

    <EditText
        android:id="@+id/passwordInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Password"
        android:inputType="textPassword"
        android:padding="12dp"
        android:layout_marginBottom="24dp"/>

    <Button
        android:id="@+id/loginButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="LOGIN"
        android:background="@color/primary"
        android:textColor="@android:color/white"
        android:padding="12dp"
        android:layout_marginBottom="12dp"/>

    <TextView
        android:id="@+id/registerLink"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="First time? Register here"
        android:textColor="@color/primary"
        android:layout_gravity="center"/>

</LinearLayout>
java
// MainActivity.java
public class MainActivity extends AppCompatActivity {
    
    private FirebaseAuth mAuth;
    private DatabaseReference mDatabase;
    private EditText lrnInput, passwordInput;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize Firebase
        FirebaseApp.initializeApp(this);
        mAuth = FirebaseAuth.getInstance();
        mDatabase = FirebaseDatabase.getInstance().getReference();
        
        lrnInput = findViewById(R.id.lrnInput);
        passwordInput = findViewById(R.id.passwordInput);
        
        // Check if already logged in
        if (mAuth.getCurrentUser() != null) {
            startActivity(new Intent(this, DashboardActivity.class));
            finish();
        }
        
        findViewById(R.id.loginButton).setOnClickListener(v -> login());
        findViewById(R.id.registerLink).setOnClickListener(v -> register());
    }
    
    private void login() {
        String lrn = lrnInput.getText().toString();
        String password = passwordInput.getText().toString();
        
        if (lrn.isEmpty() || password.isEmpty()) {
            Toast.makeText(this, "Please fill all fields", Toast.LENGTH_SHORT).show();
            return;
        }
        
        // Login with Firebase
        mAuth.signInWithEmailAndPassword(lrn + "@parent.school.edu.ph", password)
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    // Save FCM token for push notifications
                    saveFCMToken();
                    
                    // Go to dashboard
                    startActivity(new Intent(MainActivity.this, DashboardActivity.class));
                    finish();
                } else {
                    Toast.makeText(MainActivity.this, 
                        "Login failed: " + task.getException().getMessage(), 
                        Toast.LENGTH_SHORT).show();
                }
            });
    }
    
    private void saveFCMToken() {
        FirebaseMessaging.getInstance().getToken()
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    String token = task.getResult();
                    String userId = mAuth.getCurrentUser().getUid();
                    
                    // Save token to Firebase
                    mDatabase.child("parent_tokens").child(userId).setValue(token);
                    
                    // Also save to school database via API
                    sendTokenToSchoolServer(userId, token);
                }
            });
    }
}
java
// NotificationService.java
public class NotificationService extends FirebaseMessagingService {
    
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        super.onMessageReceived(remoteMessage);
        
        // Get notification data
        String title = remoteMessage.getNotification().getTitle();
        String body = remoteMessage.getNotification().getBody();
        Map<String, String> data = remoteMessage.getData();
        
        // Save to local database
        saveToLocalDatabase(title, body, data);
        
        // Show notification
        showNotification(title, body, data);
    }
    
    private void showNotification(String title, String body, Map<String, String> data) {
        Intent intent = new Intent(this, NotificationDetailActivity.class);
        intent.putExtra("data", new HashMap<>(data));
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "attendance_channel")
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(body)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .setContentIntent(pendingIntent);
        
        NotificationManagerCompat manager = NotificationManagerCompat.from(this);
        
        // Group notifications by student
        String studentId = data.get("studentId");
        NotificationManagerCompat.from(this).notify(studentId.hashCode(), builder.build());
    }
}
5. ATTENDANCE LOGIC (Enhanced Rules)
5.1 Complete Attendance Processing
csharp
// Enhanced AttendanceCalculator.cs
public class AttendanceCalculator
{
    private HolidayService holidayService;
    private NotificationService notificationService;
    private AuditService auditService;
    
    public async Task<AttendanceResult> ProcessTap(string rfidUid, DateTime currentTime, string gateLocation)
    {
        // Step 1: Check if holiday
        var holiday = holidayService.GetHoliday(currentTime.Date);
        if (holiday != null && holiday.event_type == "NO_CLASSES")
        {
            return new AttendanceResult
            {
                Status = "NO_CLASSES",
                Message = $"No classes today: {holiday.event_name}",
                ShowAsHoliday = true
            };
        }
        
        // Step 2: Get person
        var person = GetPersonByRFID(rfidUid);
        if (person == null)
        {
            auditService.LogWarning("Unknown RFID tapped", rfidUid);
            return new AttendanceResult
            {
                Status = "UNKNOWN_CARD",
                Message = "Unknown RFID card. Please see registrar."
            };
        }
        
        // Step 3: Check for duplicate tap (within 5 minutes)
        if (IsDuplicateTap(rfidUid, currentTime))
        {
            var lastTap = GetLastTap(rfidUid);
            var minutesAgo = (int)(currentTime - lastTap.Timestamp).TotalMinutes;
            
            return new AttendanceResult
            {
                Status = "DUPLICATE",
                Message = $"Already tapped {minutesAgo} minutes ago",
                Person = person
            };
        }
        
        // Step 4: Determine session and action
        var session = GetCurrentSession(currentTime);
        var action = DetermineAction(person, session);
        
        // Step 5: Calculate status (On Time, Late, Early Out)
        var (status, minutesLate, minutesEarly) = CalculateStatus(action, session, currentTime, holiday);
        
        // Step 6: Check for half-day scenarios
        var halfDayStatus = CheckHalfDayStatus(person, currentTime.Date, session);
        
        // Step 7: Create attendance record
        var attendance = new AttendanceLog
        {
            rfid_uid = rfidUid,
            person_type = person.Type,
            person_id = person.Id,
            timestamp = currentTime,
            action = action,
            session = session,
            status = halfDayStatus ?? status,
            minutes_late = minutesLate,
            minutes_early = minutesEarly,
            gate_location = gateLocation,
            sync_status = "Pending",
            is_holiday_override = holiday != null
        };
        
        SaveAttendance(attendance);
        
        // Step 8: Queue parent notification
        if (ShouldNotifyParent(attendance))
        {
            notificationService.QueueAttendanceNotification(attendance);
        }
        
        // Step 9: Audit log
        auditService.LogAction("INSERT", "attendance", attendance.attendance_id, 
            null, JsonConvert.SerializeObject(attendance));
        
        return new AttendanceResult
        {
            Person = person,
            Action = action,
            Session = session,
            Status = status,
            MinutesLate = minutesLate,
            MinutesEarly = minutesEarly,
            Timestamp = currentTime,
            HalfDayDetected = halfDayStatus != null
        };
    }
    
    private (string status, int late, int early) CalculateStatus(
        string action, string session, DateTime time, Holiday holiday)
    {
        if (holiday?.event_type == "HALF_DAY_MORNING" && session == "Morning")
        {
            return ("HALF_DAY_HOLIDAY", 0, 0);
        }
        
        if (holiday?.event_type == "HALF_DAY_AFTERNOON" && session == "Afternoon")
        {
            return ("HALF_DAY_HOLIDAY", 0, 0);
        }
        
        if (action == "TIME IN")
        {
            var lateThreshold = GetLateThreshold(session);
            if (time.TimeOfDay > lateThreshold)
            {
                var minutesLate = (int)(time.TimeOfDay - lateThreshold).TotalMinutes;
                return ("Late", minutesLate, 0);
            }
            return ("On Time", 0, 0);
        }
        else // TIME OUT
        {
            var earlyThreshold = GetEarlyThreshold(session);
            if (time.TimeOfDay < earlyThreshold)
            {
                var minutesEarly = (int)(earlyThreshold - time.TimeOfDay).TotalMinutes;
                return ("Early Out", 0, minutesEarly);
            }
            return ("On Time", 0, 0);
        }
    }
    
    private bool ShouldNotifyParent(AttendanceLog attendance)
    {
        // Notify for significant events
        return attendance.status == "Late" 
            || attendance.status == "Early Out"
            || attendance.status == "Absent"
            || (attendance.action == "TIME IN" && attendance.session == "Morning");
            // Notify every morning time-in, but only significant afternoon events
    }
}
5.2 Half-Day Detection
csharp
public class HalfDayDetectionService
{
    public void DetectHalfDays(DateTime date)
    {
        var allStudents = GetAllActiveStudents();
        
        foreach (var student in allStudents)
        {
            var logs = GetAttendanceLogs(student.student_id, date);
            
            bool hasMorningIn = logs.Any(l => l.session == "Morning" && l.action == "TIME IN");
            bool hasMorningOut = logs.Any(l => l.session == "Morning" && l.action == "TIME OUT");
            bool hasAfternoonIn = logs.Any(l => l.session == "Afternoon" && l.action == "TIME IN");
            bool hasAfternoonOut = logs.Any(l => l.session == "Afternoon" && l.action == "TIME OUT");
            
            // Case 1: Only morning session (no afternoon)
            if ((hasMorningIn || hasMorningOut) && !hasAfternoonIn && !hasAfternoonOut)
            {
                CreateHalfDayRecord(student, "Afternoon", date, "UNEXPLAINED");
                
                // Notify parent
                notificationService.QueueNotification(
                    student, 
                    "HALF_DAY", 
                    $"{student.first_name} only attended morning session today."
                );
            }
            // Case 2: Only afternoon session (no morning)
            else if (!hasMorningIn && !hasMorningOut && (hasAfternoonIn || hasAfternoonOut))
            {
                CreateHalfDayRecord(student, "Morning", date, "UNEXPLAINED");
                
                notificationService.QueueNotification(
                    student, 
                    "HALF_DAY", 
                    $"{student.first_name} only attended afternoon session today."
                );
            }
        }
    }
    
    // Run at 6:00 PM daily
    public void ScheduleHalfDayDetection()
    {
        var timer = new Timer((e) =>
        {
            DetectHalfDays(DateTime.Today);
        }, null, GetTimeUntil6PM(), TimeSpan.FromDays(1));
    }
}
6. REPORTS & PRINTING (Enhanced)
6.1 Complete Report Types
A. PERFECT ATTENDANCE CERTIFICATE (NEW)
text
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║                    [SCHOOL LOGO]                             ║
║                                                              ║
║              PERFECT ATTENDANCE CERTIFICATE                  ║
║                                                              ║
║                    School Year 2025-2026                     ║
║                                                              ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║   This certifies that                                         ║
║                                                              ║
║              JUAN DELA CRUZ                                  ║
║                                                              ║
║   of Grade 4 - Section A                                     ║
║                                                              ║
║   has achieved PERFECT ATTENDANCE for the month of           ║
║                                                              ║
║                    MARCH 2026                                 ║
║                                                              ║
║   Attendance Record:                                          ║
║   ├── School Days: 22                                        ║
║   ├── Days Present: 22                                       ║
║   ├── Days Absent: 0                                         ║
║   ├── Times Late: 0                                          ║
║   └── Attendance Rate: 100%                                  ║
║                                                              ║
║                                                              ║
║   _______________________          _______________________   ║
║   Class Adviser                      School Principal       ║
║                                                              ║
║   Date Issued: April 1, 2026                                ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
B. CHRONIC LATENESS REPORT (NEW)
text
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║              CHRONIC LATENESS REPORT                         ║
║              Elementary Department                           ║
║              March 2026                                      ║
║                                                              ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  ⚠️ STUDENTS WITH 10+ LATES                                   ║
║  ────────────────────────────────────────────────────────   ║
║                                                              ║
║  Rank │ Student Name      │ Grade │ Late Count │ Avg Minutes║
║  ─────┼───────────────────┼───────┼────────────┼─────────── ║
║  1    │ Cruz, Juan        │ 4-A   │     15     │    12      ║
║  2    │ Santos, Maria     │ 4-A   │     12     │     8      ║
║  3    │ Reyes, Pedro      │ 4-B   │     11     │    10      ║
║  4    │ Villanueva, Ana   │ 5-A   │     10     │     7      ║
║                                                              ║
║  ⚠️ STUDENTS WITH 20+ LATES (ESCALATED)                       ║
║  ────────────────────────────────────────────────────────   ║
║                                                              ║
║  1    │ Dimagiba, Jose    │ 6-A   │     25     │    15      ║
║                                                              ║
║  ACTION ITEMS:                                               ║
║  ├── Students with 10+ lates: Parent conference required    ║
║  ├── Students with 20+ lates: Disciplinary committee       ║
║  └── All parents notified via: SMS (15) / Push (12)         ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
C. PARENT NOTIFICATION SUMMARY (NEW)
text
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║              PARENT NOTIFICATION LOG                         ║
║              March 15, 2026                                  ║
║                                                              ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  SUMMARY                                                     ║
║  ────────────────────────────────────────────────────────   ║
║  Total Notifications Sent: 187                               ║
║  ├── Firebase (Push): 156 (FREE)                            ║
║  ├── SMS: 31 (₱31.00)                                       ║
║  └── Email: 0                                                ║
║                                                              ║
║  BY TYPE:                                                    ║
║  ├── Time In: 89                                            ║
║  ├── Time Out: 78                                           ║
║  ├── Late: 15                                               ║
║  ├── Absent: 5                                              ║
║  └── Early Out: 0                                           ║
║                                                              ║
║  DETAILED LOG                                                ║
║  ────────────────────────────────────────────────────────   ║
║  Time    │ Student      │ Type    │ Via     │ Status       ║
║  ────────┼──────────────┼─────────┼─────────┼───────────── ║
║  7:45 AM │ Cruz, Juan   │ Late    │ Firebase│ Delivered    ║
║  7:42 AM │ Santos, Maria│ Time In │ Firebase│ Delivered    ║
║  4:05 PM │ Reyes, Ana   │ Time Out│ SMS     │ Sent         ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
D. ATTENDANCE RATE BY SECTION
text
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║              ATTENDANCE RATE BY SECTION                      ║
║              March 2026                                      ║
║              Elementary Department                           ║
║                                                              ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  GRADE 1                                                     ║
║  ────────────────────────────────────────────────────────   ║
║  Section    │ Students │ Avg Rate │ Perfect │ Needs Review ║
║  ───────────┼──────────┼──────────┼─────────┼───────────── ║
║  Section A  │    35    │   98%    │   28    │      0       ║
║  Section B  │    34    │   95%    │   20    │      2       ║
║  Section C  │    35    │   92%    │   15    │      5       ║
║                                                              ║
║  GRADE 2                                                     ║
║  ────────────────────────────────────────────────────────   ║
║  Section A  │    36    │   97%    │   25    │      1       ║
║  Section B  │    35    │   94%    │   18    │      3       ║
║                                                              ║
║  OVERALL: 95.2% attendance rate                             ║
║  Perfect Attendance Awardees: 106 students                  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
6.2 Report Generation Service
csharp
public class ReportService
{
    public byte[] GeneratePerfectAttendanceCertificate(Student student, string month, int year)
    {
        var template = LoadTemplate("PerfectAttendanceCertificate.html");
        
        var data = new Dictionary<string, string>
        {
            { "StudentName", student.full_name },
            { "GradeSection", $"{student.grade_level} - {student.section_name}" },
            { "Month", month },
            { "Year", year.ToString() },
            { "SchoolDays", GetSchoolDays(month, year).ToString() },
            { "DaysPresent", GetDaysPresent(student, month, year).ToString() },
            { "AttendanceRate", "100%" },
            { "IssueDate", DateTime.Now.ToString("MMMM d, yyyy") }
        };
        
        var html = ReplacePlaceholders(template, data);
        return ConvertHtmlToPdf(html);
    }
    
    public byte[] GenerateChronicLatenessReport(string level, DateTime startDate, DateTime endDate)
    {
        var students = GetChronicLateStudents(level, startDate, endDate, 10); // 10+ lates
        
        var report = new StringBuilder();
        report.AppendLine("<h1>Chronic Lateness Report</h1>");
        report.AppendLine($"<p>Period: {startDate:MMMM d, yyyy} - {endDate:MMMM d, yyyy}</p>");
        
        report.AppendLine("<table border='1'>");
        report.AppendLine("<tr><th>Rank</th><th>Student</th><th>Grade</th><th>Late Count</th><th>Avg Minutes</th></tr>");
        
        int rank = 1;
        foreach (var student in students.OrderByDescending(s => s.LateCount))
        {
            report.AppendLine($"<tr>");
            report.AppendLine($"<td>{rank++}</td>");
            report.AppendLine($"<td>{student.FullName}</td>");
            report.AppendLine($"<td>{student.GradeLevel}-{student.Section}</td>");
            report.AppendLine($"<td>{student.LateCount}</td>");
            report.AppendLine($"<td>{student.AvgMinutesLate:F0}</td>");
            report.AppendLine($"</tr>");
        }
        
        report.AppendLine("</table>");
        
        return ConvertHtmlToPdf(report.ToString());
    }
}
7. SECURITY & AUDIT TRAIL
7.1 Complete Audit Logging
csharp
public class AuditService
{
    private readonly string connectionString = "Data Source=C:\\SchoolData\\master.db";
    
    public void LogAction(string userId, string username, string action, string table, 
                          string recordId, string oldValue = null, string newValue = null)
    {
        using (var connection = new SQLiteConnection(connectionString))
        {
            connection.Open();
            
            var cmd = connection.CreateCommand();
            cmd.CommandText = @"
                INSERT INTO audit_log 
                (user_id, username, action_type, table_name, record_id, old_value, new_value, ip_address, pc_name)
                VALUES 
                (@userId, @username, @action, @table, @recordId, @oldValue, @newValue, @ip, @pcName)";
            
            cmd.Parameters.AddWithValue("@userId", userId);
            cmd.Parameters.AddWithValue("@username", username);
            cmd.Parameters.AddWithValue("@action", action);
            cmd.Parameters.AddWithValue("@table", table);
            cmd.Parameters.AddWithValue("@recordId", recordId);
            cmd.Parameters.AddWithValue("@oldValue", oldValue ?? "");
            cmd.Parameters.AddWithValue("@newValue", newValue ?? "");
            cmd.Parameters.AddWithValue("@ip", GetLocalIPAddress());
            cmd.Parameters.AddWithValue("@pcName", Environment.MachineName);
            
            cmd.ExecuteNonQuery();
        }
    }
    
    public List<AuditLog> GetAuditLogs(DateTime startDate, DateTime endDate, string user = null, string action = null)
    {
        var logs = new List<AuditLog>();
        
        using (var connection = new SQLiteConnection(connectionString))
        {
            connection.Open();
            
            var cmd = connection.CreateCommand();
            cmd.CommandText = @"
                SELECT * FROM audit_log 
                WHERE timestamp BETWEEN @start AND @end
                AND (@user IS NULL OR username = @user)
                AND (@action IS NULL OR action_type = @action)
                ORDER BY timestamp DESC";
            
            cmd.Parameters.AddWithValue("@start", startDate.ToString("yyyy-MM-dd"));
            cmd.Parameters.AddWithValue("@end", endDate.AddDays(1).ToString("yyyy-MM-dd"));
            cmd.Parameters.AddWithValue("@user", user ?? "");
            cmd.Parameters.AddWithValue("@action", action ?? "");
            
            using (var reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    logs.Add(new AuditLog
                    {
                        audit_id = reader.GetInt32(0),
                        user_id = reader.GetInt32(1),
                        username = reader.GetString(2),
                        action_type = reader.GetString(3),
                        table_name = reader.GetString(4),
                        record_id = reader.GetString(5),
                        old_value = reader.GetString(6),
                        new_value = reader.GetString(7),
                        ip_address = reader.GetString(8),
                        pc_name = reader.GetString(9),
                        timestamp = reader.GetDateTime(10)
                    });
                }
            }
        }
        
        return logs;
    }
    
    public void LogLogin(string userId, string username, bool success, string reason = null)
    {
        var action = success ? "LOGIN_SUCCESS" : "LOGIN_FAILED";
        var details = success ? null : $"Failed login: {reason}";
        
        LogAction(userId, username, action, "users", userId, null, details);
    }
}
7.2 User Authentication & Authorization
csharp
public class SecurityService
{
    public User Authenticate(string username, string password)
    {
        using (var connection = new SQLiteConnection("Data Source=C:\\SchoolData\\master.db"))
        {
            connection.Open();
            
            var cmd = connection.CreateCommand();
            cmd.CommandText = @"
                SELECT * FROM users 
                WHERE username = @username AND is_active = 1";
            cmd.Parameters.AddWithValue("@username", username);
            
            using (var reader = cmd.ExecuteReader())
            {
                if (reader.Read())
                {
                    var user = new User
                    {
                        user_id = reader.GetInt32(0),
                        username = reader.GetString(1),
                        password_hash = reader.GetString(2),
                        full_name = reader.GetString(3),
                        role = reader.GetString(4),
                        permissions = reader.GetString(5),
                        is_active = reader.GetBoolean(6),
                        login_attempts = reader.GetInt32(7),
                        locked_until = reader.IsDBNull(8) ? (DateTime?)null : reader.GetDateTime(8)
                    };
                    
                    // Check if account is locked
                    if (user.locked_until.HasValue && user.locked_until > DateTime.Now)
                    {
                        AuditService.LogAction(user.user_id.ToString(), username, "LOGIN_BLOCKED", 
                            "users", user.user_id.ToString(), null, "Account locked");
                        return null;
                    }
                    
                    // Verify password
                    if (BCrypt.Net.BCrypt.Verify(password, user.password_hash))
                    {
                        // Reset login attempts
                        ResetLoginAttempts(username);
                        
                        // Log successful login
                        AuditService.LogLogin(user.user_id.ToString(), username, true);
                        
                        // Update last login
                        UpdateLastLogin(user.user_id);
                        
                        return user;
                    }
                    else
                    {
                        // Increment failed attempts
                        IncrementFailedAttempts(username);
                        
                        // Log failed login
                        AuditService.LogLogin(user.user_id.ToString(), username, false, "Invalid password");
                    }
                }
            }
        }
        
        return null;
    }
    
    public bool HasPermission(User user, string requiredPermission)
    {
        if (user.role == "SUPER_ADMIN") return true;
        
        var permissions = JsonConvert.DeserializeObject<List<string>>(user.permissions);
        return permissions.Contains(requiredPermission);
    }
}
8. SYNC MODULE (Enhanced)
8.1 Complete Sync Strategy
csharp
public class SyncService
{
    private readonly string localDbPath = "C:\\SchoolGate\\local.db";
    private readonly string serverUrl = "http://192.168.1.100:8080/api/sync";
    private readonly string usbSyncPath = "E:\\SchoolSync\\";
    
    public enum SyncMode { Automatic, Manual, USB }
    
    public async Task<SyncResult> SyncWithServer(SyncMode mode)
    {
        var result = new SyncResult
        {
            sync_id = Guid.NewGuid().ToString(),
            started_at = DateTime.Now,
            source_pc = Environment.MachineName
        };
        
        try
        {
            // Check if server is reachable
            if (mode != SyncMode.USB && !IsServerReachable())
            {
                if (mode == SyncMode.Automatic)
                {
                    // Queue for later
                    QueueForLaterSync();
                    return new SyncResult { status = "QUEUED", message = "Server offline, queued for later" };
                }
                else
                {
                    // Switch to USB mode automatically
                    return await SyncViaUSB();
                }
            }
            
            // Get pending records
            var pendingRecords = GetPendingRecords();
            result.records_sent = pendingRecords.Count;
            
            // Prepare data for sync
            var syncData = new
            {
                gate_id = Environment.MachineName,
                records = pendingRecords,
                last_sync = GetLastSyncTime(),
                sync_mode = mode.ToString()
            };
            
            // Encrypt data
            var encryptedData = EncryptionHelper.Encrypt(JsonConvert.SerializeObject(syncData), GetSyncKey());
            
            // Send to server
            using (var client = new HttpClient())
            {
                var content = new StringContent(encryptedData, Encoding.UTF8, "application/json");
                var response = await client.PostAsync(serverUrl, content);
                
                if (response.IsSuccessStatusCode)
                {
                    var responseData = await response.Content.ReadAsStringAsync();
                    var serverResult = JsonConvert.DeserializeObject<SyncResponse>(responseData);
                    
                    // Update local records as synced
                    MarkRecordsAsSynced(pendingRecords.Select(r => r.attendance_id).ToList());
                    
                    // Process conflicts if any
                    if (serverResult.conflicts?.Count > 0)
                    {
                        ResolveConflicts(serverResult.conflicts);
                        result.conflicts_resolved = serverResult.conflicts.Count;
                    }
                    
                    // Get new data from server (students, sections, etc.)
                    var newData = await GetServerUpdates(serverResult.last_sequence);
                    result.records_received = UpdateLocalDatabase(newData);
                    
                    result.status = "SUCCESS";
                    result.completed_at = DateTime.Now;
                    result.duration_seconds = (int)(result.completed_at - result.started_at).TotalSeconds;
                    
                    // Log sync
                    LogSyncResult(result);
                }
            }
        }
        catch (Exception ex)
        {
            result.status = "FAILED";
            result.error_message = ex.Message;
            LogSyncError(result);
        }
        
        return result;
    }
    
    public async Task<SyncResult> SyncViaUSB()
    {
        var result = new SyncResult
        {
            sync_id = Guid.NewGuid().ToString(),
            started_at = DateTime.Now,
            source_pc = Environment.MachineName,
            sync_type = "USB"
        };
        
        try
        {
            // Check if USB drive exists
            if (!Directory.Exists(usbSyncPath))
            {
                return new SyncResult { status = "FAILED", error_message = "USB drive not found" };
            }
            
            // Export local pending records to USB
            var pendingRecords = GetPendingRecords();
            var fileName = $"attendance_{Environment.MachineName}_{DateTime.Now:yyyyMMdd_HHmmss}.enc";
            var filePath = Path.Combine(usbSyncPath, fileName);
            
            var exportData = new
            {
                source = Environment.MachineName,
                timestamp = DateTime.Now,
                records = pendingRecords,
                version = "2.0"
            };
            
            var encrypted = EncryptionHelper.Encrypt(JsonConvert.SerializeObject(exportData), GetSyncKey());
            File.WriteAllBytes(filePath, Convert.FromBase64String(encrypted));
            
            // Create manifest file
            var manifest = new
            {
                file = fileName,
                size = encrypted.Length,
                record_count = pendingRecords.Count,
                source = Environment.MachineName
            };
            File.WriteAllText(Path.Combine(usbSyncPath, "manifest.json"), 
                JsonConvert.SerializeObject(manifest));
            
            result.status = "EXPORTED";
            result.records_sent = pendingRecords.Count;
            result.message = $"Data exported to USB: {fileName}";
        }
        catch (Exception ex)
        {
            result.status = "FAILED";
            result.error_message = ex.Message;
        }
        
        return result;
    }
    
    public async Task<SyncResult> ImportFromUSB(string fileName)
    {
        var result = new SyncResult
        {
            sync_id = Guid.NewGuid().ToString(),
            started_at = DateTime.Now,
            source_pc = "USB_IMPORT"
        };
        
        try
        {
            var filePath = Path.Combine(usbSyncPath, fileName);
            var encryptedData = File.ReadAllBytes(filePath);
            var decrypted = EncryptionHelper.Decrypt(Convert.ToBase64String(encryptedData), GetSyncKey());
            var importData = JsonConvert.DeserializeObject<dynamic>(decrypted);
            
            // Process imported records
            int imported = 0;
            foreach (var record in importData.records)
            {
                if (ImportRecord(record))
                {
                    imported++;
                }
            }
            
            result.records_received = imported;
            result.status = "SUCCESS";
            result.completed_at = DateTime.Now;
            
            // Move file to processed folder
            var processedPath = Path.Combine(usbSyncPath, "processed", fileName);
            Directory.CreateDirectory(Path.GetDirectoryName(processedPath));
            File.Move(filePath, processedPath);
        }
        catch (Exception ex)
        {
            result.status = "FAILED";
            result.error_message = ex.Message;
        }
        
        return result;
    }
}
8.2 Conflict Resolution
csharp
public class ConflictResolver
{
    public void ResolveConflicts(List<ConflictRecord> conflicts)
    {
        foreach (var conflict in conflicts)
        {
            switch (conflict.type)
            {
                case "DUPLICATE_TAP":
                    // Keep the earliest TIME IN, latest TIME OUT
                    var records = conflict.records.OrderBy(r => r.timestamp).ToList();
                    
                    // Keep first TIME IN
                    var timeIn = records.First(r => r.action == "TIME IN");
                    MarkAsValid(timeIn);
                    
                    // Keep last TIME OUT
                    var timeOut = records.Last(r => r.action == "TIME OUT");
                    if (timeOut.attendance_id != timeIn.attendance_id)
                    {
                        MarkAsValid(timeOut);
                    }
                    
                    // Mark others as duplicates
                    foreach (var rec in records.Where(r => 
                        r.attendance_id != timeIn.attendance_id && 
                        r.attendance_id != timeOut.attendance_id))
                    {
                        MarkAsDuplicate(rec);
                    }
                    break;
                    
                case "MISSING_TIME_IN":
                    // For TIME OUT without TIME IN, create estimated TIME IN
                    var timeOutRec = conflict.records.First(r => r.action == "TIME OUT");
                    CreateEstimatedTimeIn(timeOutRec);
                    MarkAsValid(timeOutRec);
                    break;
                    
                case "MISSING_TIME_OUT":
                    // For TIME IN without TIME OUT, create estimated TIME OUT
                    var timeInRec = conflict.records.First(r => r.action == "TIME IN");
                    CreateEstimatedTimeOut(timeInRec);
                    MarkAsValid(timeInRec);
                    break;
            }
        }
    }
}
9. BACKUP & RECOVERY SYSTEM
9.1 Automated Backup Service
csharp
public class BackupService
{
    private readonly string dataPath = "C:\\SchoolData\\";
    private readonly string backupPath = "C:\\SchoolData\\Backups\\";
    private readonly string encryptionKey = "school-secure-key-2026";
    
    public void StartAutoBackup()
    {
        // Schedule daily backup at 11:00 PM
        var now = DateTime.Now;
        var firstRun = DateTime.Today.AddHours(23);
        if (now > firstRun) firstRun = firstRun.AddDays(1);
        
        var timer = new Timer((e) =>
        {
            CreateBackup("AUTO");
        }, null, firstRun - now, TimeSpan.FromDays(1));
    }
    
    public BackupResult CreateBackup(string backupType)
    {
        var result = new BackupResult
        {
            backup_id = Guid.NewGuid().ToString(),
            backup_name = $"master_{DateTime.Now:yyyyMMdd_HHmmss}",
            backup_type = backupType,
            started_at = DateTime.Now
        };
        
        try
        {
            // Create backup directory if not exists
            Directory.CreateDirectory(backupPath);
            
            // Backup main database
            var dbFile = Path.Combine(dataPath, "master.db");
            var backupFile = Path.Combine(backupPath, $"{result.backup_name}.db");
            
            // Create SQLite backup
            using (var source = new SQLiteConnection($"Data Source={dbFile}"))
            using (var dest = new SQLiteConnection($"Data Source={backupFile}"))
            {
                source.Open();
                dest.Open();
                source.BackupDatabase(dest, "main", "main", -1, null, 0);
            }
            
            // Encrypt backup
            var encryptedFile = EncryptBackup(backupFile);
            File.Delete(backupFile);
            
            // Calculate checksum
            result.checksum = CalculateChecksum(encryptedFile);
            result.file_size = new FileInfo(encryptedFile).Length;
            result.encryption_used = true;
            
            // Compress old backups if needed
            if (backupType == "AUTO")
            {
                CleanupOldBackups();
            }
            
            result.status = "SUCCESS";
            result.completed_at = DateTime.Now;
            
            // Log backup
            LogBackup(result);
        }
        catch (Exception ex)
        {
            result.status = "FAILED";
            result.error_message = ex.Message;
            LogBackupError(result);
        }
        
        return result;
    }
    
    public RestoreResult RestoreBackup(string backupFile, bool dryRun = false)
    {
        var result = new RestoreResult
        {
            restore_id = Guid.NewGuid().ToString(),
            backup_file = backupFile,
            started_at = DateTime.Now
        };
        
        try
        {
            // Verify backup integrity
            var checksum = CalculateChecksum(backupFile);
            if (!VerifyBackupChecksum(backupFile, checksum))
            {
                throw new Exception("Backup file corrupted - checksum mismatch");
            }
            
            // Decrypt backup
            var decryptedFile = DecryptBackup(backupFile);
            
            if (dryRun)
            {
                // Preview changes
                result.preview = GetRestorePreview(decryptedFile);
                result.status = "PREVIEW";
                return result;
            }
            
            // Create restore point
            CreateRestorePoint();
            
            // Stop services
            StopServices();
            
            // Restore database
            var dbFile = Path.Combine(dataPath, "master.db");
            File.Copy(decryptedFile, dbFile, true);
            
            // Restart services
            StartServices();
            
            result.status = "SUCCESS";
            result.completed_at = DateTime.Now;
            
            // Log restore
            LogRestore(result);
        }
        catch (Exception ex)
        {
            result.status = "FAILED";
            result.error_message = ex.Message;
            
            // Auto-rollback if needed
            if (result.status == "FAILED" && File.Exists(Path.Combine(backupPath, "restore_point.db")))
            {
                RollbackRestore();
            }
        }
        
        return result;
    }
    
    private void CleanupOldBackups()
    {
        var backups = Directory.GetFiles(backupPath, "*.enc")
            .Select(f => new FileInfo(f))
            .OrderByDescending(f => f.CreationTime)
            .ToList();
        
        // Keep last 30 daily backups
        if (backups.Count > 30)
        {
            foreach (var backup in backups.Skip(30))
            {
                backup.Delete();
            }
        }
        
        // Keep weekly backups for 6 months
        var weeklyBackups = backups.Where(f => f.CreationTime.DayOfWeek == DayOfWeek.Sunday)
            .Skip(26) // Keep 6 months of weekly backups
            .ToList();
            
        foreach (var backup in weeklyBackups)
        {
            backup.Delete();
        }
    }
}
10. MOBILE APP STRATEGY (Future Phases)
10.1 Three-Phase Mobile Development
Phase 1: Parent App (Current - Android Only)
text
PARENT APP FEATURES (v1.0)
═══════════════════════════════════

📱 CORE FEATURES
├── Real-time attendance notifications
├── View today's attendance status
├── View attendance history (30 days)
├── Multiple children support
├── Excuse absence/late submission
└── Download attendance certificates

📊 FREE vs PREMIUM
├── FREE (₱0)
│   ├── Push notifications (Firebase)
│   ├── Basic attendance view
│   └── 7-day history
│
├── PREMIUM (₱50/month or ₱500/year)
│   ├── SMS backup notifications
│   ├── Full attendance history
│   ├── Downloadable reports
│   ├── Priority support
│   └── Ad-free experience

TECH STACK
├── Android (Kotlin/Java)
├── Firebase Cloud Messaging
├── Firebase Realtime Database
├── Local SQLite for offline
└── Material Design 3
Phase 2: Teacher/Adviser App (Future)
text
TEACHER APP FEATURES (v2.0)
═══════════════════════════════════

👩‍🏫 FOR ADVISERS
├── View section attendance in real-time
├── Mark excuses/valid reasons
├── Send announcements to parents
├── Generate daily attendance report
├── View chronic lateness per student
└── Schedule parent meetings

📋 FOR SUBJECT TEACHERS (College/HS)
├── View attendance per subject/period
├── Manual override for forgotten cards
├── Note important incidents
└── Submit attendance to registrar

TECH STACK
├── Cross-platform (Flutter/React Native)
├── Sync with PC1 via API
├── Biometric login
└── Offline mode support
Phase 3: Admin Mobile Dashboard (Future)
text
ADMIN APP FEATURES (v3.0)
═══════════════════════════════════

📊 DASHBOARD
├── Real-time school-wide attendance
├── Alerts for critical issues (20+ absences)
├── Emergency broadcast to all parents
├── View sync status of all gates
├── Approve/reject excuse letters
└── Generate instant reports

🔔 EMERGENCY FEATURES
├── Class suspension announcement
├── Emergency drill notifications
├── Weather-related updates
└── School event reminders
10.2 Monetization Strategy (Optional)
csharp
public class MonetizationService
{
    public class PricingTier
    {
        public static readonly Tier Free = new Tier
        {
            name = "Free",
            price = 0,
            features = new[] { "Push Notifications", "7-Day History", "1 Child" }
        };
        
        public static readonly Tier Basic = new Tier
        {
            name = "Basic",
            price = 50, // PHP/month
            features = new[] { "Push + SMS", "30-Day History", "3 Children", "Download Reports" }
        };
        
        public static readonly Tier Premium = new Tier
        {
            name = "Premium",
            price = 500, // PHP/year
            features = new[] { "All Basic Features", "Unlimited History", "5 Children", "Priority Support" }
        };
    }
    
    public decimal CalculateMonthlyRevenue(int freeUsers, int basicUsers, int premiumUsers)
    {
        return (basicUsers * 50) + (premiumUsers * (500m / 12));
    }
    
    // For 10,000 students:
    // 7,000 Free = ₱0
    // 2,000 Basic = ₱100,000/month
    // 1,000 Premium = ₱41,667/month
    // Total Monthly Revenue = ₱141,667
}
11. BUDGET BREAKDOWN (Complete)
11.1 Development Costs (One-Time)
Component	Technology	Cost
Desktop Apps (PC1, Gates)	C# .NET	₱0 (Visual Studio Community)
Database	SQLite + (Optional MySQL)	₱0
Parent App (Android)	Android Studio + Firebase	₱0
Firebase	Cloud Messaging + Auth	₱0 (up to 1M msgs)
SMS Backup	Semaphore/Chikka	₱1.00/SMS (optional)
Development PC	Existing hardware	₱0
RFID Readers	5 pcs @ ₱1,500	₱7,500
RFID Cards	10,000 pcs @ ₱20	₱200,000
TOTAL HARDWARE		₱207,500
11.2 Monthly Operating Costs
Item	Free Tier	Paid Tier	For 10K Students
Firebase	1M msgs	₱0	✅ Free
SMS Backup	0	₱1.00/msg	₱3,000/month (if 10% use SMS)
Server Hosting	Local PC1	₱0	₱0
Support Staff	1 part-time	₱5,000	₱5,000
TOTAL MONTHLY			₱5,000 - ₱8,000
11.3 Potential Revenue (If Monetized)
Tier	Users	Price	Monthly Revenue
Free	7,000	₱0	₱0
Basic	2,000	₱50	₱100,000
Premium	1,000	₱41.67 (yearly)	₱41,667
TOTAL	10,000		₱141,667/month
12. DEVELOPMENT TIMELINE (4 Months)
Month 1: Foundation (Weeks 1-4)
Week	Tasks	Deliverables
Week 1	Database Design, Core Library	Complete SQLite schema, Models
Week 2	Installation Wizard, User Auth	Working installer, Login system
Week 3	Enrollment Forms (all levels)	Dynamic forms per level
Week 4	RFID Integration	Read/Write working, Card management
Month 2: Core Features (Weeks 5-8)
Week	Tasks	Deliverables
Week 5	Gate Monitor App	Full-screen display, RFID reading
Week 6	Attendance Logic	Late detection, Half-day detection
Week 7	Holiday Calendar, Audit Trail	Working calendar, Complete audit logs
Week 8	Basic Reports	Daily attendance, print preview
Month 3: Advanced Features (Weeks 9-12)
Week	Tasks	Deliverables
Week 9	Parent Notification Service	Firebase integration, Queue system
Week 10	Android Parent App	Login, Notifications, History
Week 11	Sync Module	Auto-sync, USB sync, Conflict resolution
Week 12	Backup & Recovery	Auto-backup, Restore functionality
Month 4: Completion (Weeks 13-16)
Week	Tasks	Deliverables
Week 13	Advanced Reports	Perfect attendance, Chronic lateness
Week 14	Promotion Management	End-of-year wizard
Week 15	System Testing	Load test with 10K students
Week 16	Deployment & Training	User manual, Training sessions
13. SYSTEM REQUIREMENTS (Updated)
13.1 Hardware Requirements
Component	Enrollment PC (PC1)	Gate PC (PC2, PC3)
OS	Windows 10/11 Pro	Windows 10/11 Pro
RAM	8GB minimum (16GB recommended)	4GB minimum
Storage	250GB SSD (for 10K students)	120GB SSD
Database	SQLite + (Optional MySQL)	SQLite only
RFID Reader	ACS ACR125U (₱2,500)	ACS ACR125U (₱2,500)
Printer	Laser printer for reports	Optional
UPS	650VA minimum	650VA minimum
Network	Gigabit LAN	Gigabit LAN (optional)
13.2 Software Requirements
Component	Technology	Version
OS	Windows	10/11 (64-bit)
.NET Framework	.NET	6.0 or later
Database	SQLite	3.35+
Firebase SDK	Firebase Admin	Latest
Android SDK	API Level	24+ (Android 7.0)
PDF Library	iTextSharp	5.5+
✅ COMPLETE FEATURE CHECKLIST
Core Features (ALL DONE)
Offline-First Design - SQLite on all PCs

3 School Levels - Elementary, HS, College (separate)

Installation Wizard - Choose level, no mixing

Enrollment Forms - Dynamic per level

Gate Monitoring - Display on tap

RFID Integration - Read & Write

Time-in/Time-out - Morning/Afternoon/Evening

Late Detection - Automatic with beep

Absence Detection - End-of-day auto detect

Reports - Daily, Weekly, Monthly, Yearly

Promotion Management - End-of-year wizard

Multi-PC Support - Multiple gates

LAN/USB Sync - Upload logs

Enhanced Features (NEW)
Audit Trail - Complete logging of all actions

Parent Notifications - Firebase push (FREE)

Holiday Calendar - Auto-adjust attendance

Half-Day Detection - Automatic flagging

RFID Card Management - Lost/Stolen/Temporary

User Roles & Permissions - Secure access

Backup & Recovery - Automated daily

Conflict Resolution - Smart sync handling

Perfect Attendance Certificates - Auto-generate

Chronic Lateness Report - Identify frequent late

Android Parent App - Real-time notifications

Optional Features (Future)
SMS Integration - Paid backup channel

Teacher Mobile App - For advisers

Admin Dashboard Mobile - School-wide view

Biometric Backup - Fingerprint option

Online Payment - For premium features

📋 SUMMARY
Master, ang UPDATED BLUEPRINT v2.0 ay mayroon na ng LAHAT ng requirements mo:

✅ 10,000 student capacity - Scalable database design
✅ Parent Notification App - Firebase (FREE for 10K)
✅ Audit Trail - Complete security logging
✅ Holiday Calendar - Auto-adjust for PH holidays
✅ Half-Day Detection - Automatic flagging
✅ RFID Card Management - Lost/Stolen handling
✅ Enhanced Sync - LAN + USB with conflict resolution
✅ Advanced Reports - Perfect attendance, Chronic lateness
✅ Backup & Recovery - Automated with encryption
✅ User Roles - Secure multi-user access

TOTAL DEVELOPMENT TIME: 4 Months
TOTAL HARDWARE COST: ₱207,500 (RFID readers + cards)
MONTHLY OPERATING COST: ₱5,000 - ₱8,000
POTENTIAL REVENUE: Up to ₱141,667/month (if monetized)

🎯 NEXT STEPS
Approve Blueprint - Review all features

Setup Development Environment - Visual Studio, Android Studio, Firebase

Start Phase 1 - Database design and Core Library

Order Hardware - RFID readers and cards (₱207,500)

Ready to start coding, master! Saan mo gusto magsimula? 👨‍💻

ano ang magandang term name para sa Project na ito? gusto ko i save sa gihub like ginawa natin sa UniPOS.

net8 blazor ok ba gamitin?
🏷️ PROJECT NAME SUGGESTIONS
Based sa features ng system natin (Philippines, School, Attendance, Multi-level), eto ang mga suggested names:

Top Picks:
EduTrack PH ✓ (Maikli, madaling tandaan)

SchoolAttendance PH (Direkta sa purpose)

RFIDSchool PH (Highlight ang RFID technology)

AttendanceHub PH (Parang central hub ng attendance)

SyncAttendance PH (Highlight ang sync capability)

Para sa GitHub (parang UniPOS style):
text
EduTrackPH
├── EduTrackPH.Core (shared library)
├── EduTrackPH.Enrollment (PC1 app)
├── EduTrackPH.Gate (Gate monitor)
├── EduTrackPH.ParentApp (Android)
└── README.md
Recommended: EduTrackPH - Professional sounding, madaling i-google, at sumasaklaw sa lahat ng features (attendance, levels, notifications).

💻 NET 8 BLAZOR - Oo, magandang gamitin!
Advantages ng Blazor para sa Project na 'to:
✅ Pros:
One Language, One Codebase

Pwedeng C# lang gamitin (Frontend at Backend)

Hindi na kailangan mag-JavaScript (though pwede pa rin)

Web-Based (Pero pwedeng Desktop)

csharp
// Pwedeng i-deploy sa dalawang paraan:

// Option 1: Web App (naka-host sa PC1, accessible via browser)
// - Kahit anong PC sa school, browser lang kelangan
// - No installation sa Gate PCs

// Option 2: Blazor Hybrid (Desktop app)
// - Gamit ang .NET MAUI
// - Pwedeng Windows app pa rin tulad ng UniPOS
Real-time Updates

csharp
// Sa Gate Monitor, pwedeng real-time ang display
@code {
    protected override async Task OnInitializedAsync()
    {
        // Real-time attendance updates
        attendanceService.OnNewTap += UpdateDisplay;
    }
}
Cross-Platform

Pwedeng tumakbo sa Windows, Linux, Mac

Future-proof kung mag-expand ang school

.NET 8 Features

Faster performance

Better memory management

Built-in OpenAPI support

⚠️ Cons to Consider:
Learning Curve

Kung puro WinForms ang experience,可能需要 adjustment

Pero kung naka-UniPOS ka na (C#), madali lang

Offline Considerations

Web-based app kelangan ng browser

Pero kung Blazor Hybrid, parang desktop app pa rin

Recommended Setup with Blazor:
text
EduTrackPH Solution
├── 📁 EduTrackPH.Core (Class Library - .NET 8)
│   ├── Models/
│   ├── Services/
│   └── Data/
│
├── 📁 EduTrackPH.Web (Blazor Server - PC1)
│   ├── Pages/
│   │   ├── Enrollment.razor
│   │   ├── Reports.razor
│   │   └── Dashboard.razor
│   ├── Components/
│   └── wwwroot/
│
├── 📁 EduTrackPH.Gate (Blazor Hybrid - Gate PCs)
│   ├── MainWindow.xaml
│   ├── GateDisplay.razor
│   └── RFIDService.cs
│
└── 📁 EduTrackPH.Mobile (MAUI Blazor - Future)
    ├── Platforms/Android/
    └── MainPage.razor
Sample Code (Blazor sa Gate Monitor):
razor
@* GateDisplay.razor *@
@page "/gate"
@inject IRFIDService rfidService
@inject IAttendanceService attendanceService

<div class="gate-display">
    <div class="header">
        <h1>@SchoolName</h1>
        <h2>@currentSession - @currentTime.ToString("hh:mm:ss tt")</h2>
    </div>

    <div class="tap-area">
        <div class="card-reader">
            <i class="fas fa-id-card"></i>
            <h3>TAP YOUR ID HERE</h3>
        </div>
    </div>

    <div class="recent-logs">
        @foreach (var log in recentLogs)
        {
            <div class="log-entry @log.StatusClass">
                <span class="time">@log.Time</span>
                <span class="name">@log.StudentName</span>
                <span class="status">@log.Status</span>
            </div>
        }
    </div>
</div>

@code {
    private DateTime currentTime = DateTime.Now;
    private string currentSession = "Morning";
    private List<DisplayLog> recentLogs = new();
    
    protected override async Task OnInitializedAsync()
    {
        // Update time every second
        var timer = new Timer(_ => {
            currentTime = DateTime.Now;
            currentSession = GetSession(currentTime);
            StateHasChanged();
        }, null, 0, 1000);
        
        // Subscribe to RFID taps
        rfidService.OnCardTapped += async (uid) => {
            var result = await attendanceService.ProcessTap(uid);
            recentLogs.Insert(0, new DisplayLog(result));
            if (recentLogs.Count > 10) recentLogs.RemoveAt(10);
            StateHasChanged();
        };
    }
}
VS Code Setup for Blazor:
json
// .vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "command": "dotnet",
            "type": "process",
            "args": [
                "build",
                "${workspaceFolder}/EduTrackPH.sln"
            ]
        },
        {
            "label": "run-web",
            "command": "dotnet",
            "type": "process",
            "args": [
                "run",
                "--project",
                "${workspaceFolder}/EduTrackPH.Web/EduTrackPH.Web.csproj"
            ]
        }
    ]
}
🎯 RECOMMENDATION
Kung ikukumpara sa UniPOS:
Aspect	UniPOS (WinForms)	EduTrackPH (Blazor)
Development Speed	Mabilis	Medyo mas matagal (setup)
UI/UX	Basic	Modern, Responsive
Remote Access	Hindi pwede	Pwedeng i-host sa LAN
Cross-Platform	Windows only	Windows, Web, Mobile
Real-time Updates	Mahirap	Built-in SignalR
Code Reuse	Limited	Maximum (shared library)
Final Suggestion:
Gamitin ang .NET 8 Blazor para sa:

EduTrackPH.Web - Enrollment system (PC1)

Hosted sa PC1, accessible via browser sa buong school

No installation sa ibang admin PCs

EduTrackPH.Gate - Gate monitor (Blazor Hybrid)

Parang desktop app pa rin (Windows)

Pero gamit ang Blazor para sa magandang UI

EduTrackPH.Mobile - Future parent app

Reuse ang same components

Sample GitHub Repository Structure:
text
EduTrackPH/
├── .github/
│   └── workflows/
│       └── build.yml
├── src/
│   ├── EduTrackPH.Core/
│   │   ├── Models/
│   │   ├── Services/
│   │   └── EduTrackPH.Core.csproj
│   ├── EduTrackPH.Web/
│   │   ├── Pages/
│   │   ├── Components/
│   │   └── EduTrackPH.Web.csproj
│   ├── EduTrackPH.Gate/
│   │   ├── MainWindow.xaml
│   │   └── EduTrackPH.Gate.csproj
│   └── EduTrackPH.Mobile/
│       └── EduTrackPH.Mobile.csproj
├── tests/
│   └── EduTrackPH.Tests/
├── docs/
│   ├── BLUEPRINT.md
│   └── API.md
├── .gitignore
├── README.md (with Filipino translation)
└── EduTrackPH.sln

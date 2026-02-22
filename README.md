# EduTrackPH
Philippine School Attendance System with RFID, Multi-level Support, and Parent Notifications

# 🏫 EduTrackPH - Philippine School Attendance System

[![.NET](https://img.shields.io/badge/.NET-8.0-purple)](https://dotnet.microsoft.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

**EduTrackPH** is a comprehensive school attendance system designed specifically for Philippine schools. It supports **Elementary, High School, and College** levels with separate management for each.

## ✨ Features

### Core Features
- ✅ **Offline-First Design** - Works even without internet
- ✅ **3 School Levels** - Elementary, High School, College (separate)
- ✅ **RFID Integration** - Read & Write support
- ✅ **Gate Monitoring** - Real-time display on tap
- ✅ **Smart Attendance** - Late detection, Half-day detection
- ✅ **Parent Notifications** - Free Firebase push notifications

### Advanced Features
- ✅ **Audit Trail** - Complete logging of all actions
- ✅ **Holiday Calendar** - Auto-adjust for PH holidays
- ✅ **Promotion Management** - End-of-year wizard
- ✅ **Comprehensive Reports** - Daily to Yearly
- ✅ **Multi-PC Sync** - LAN + USB with conflict resolution
- ✅ **Backup & Recovery** - Automated with encryption

## 🏗️ Architecture
graph TD
    A[Enrollment PC1<br>Blazor Web] --> B[(SQLite Database)]
    C[Gate PC2, PC3<br>Blazor Hybrid] --> D[(Local SQLite)]
    E[Parent Android App<br>Future] --> F[(Firebase)]
    
    C -- LAN/USB Sync --> A
    A --> F

## 🚀 Quick Start

### Prerequisites
- .NET 8 SDK
- Visual Studio 2022 / VS Code
- SQLite
- RFID Reader (ACS ACR125U recommended)

### Installation

```bash
# Clone repository
git clone https://github.com/yourusername/EduTrackPH.git
cd EduTrackPH

# Restore dependencies
dotnet restore

# Build solution
dotnet build

# Run Enrollment Web App
cd src/EduTrackPH.Web
dotnet run

# Run Gate App (on gate PCs)
cd src/EduTrackPH.Gate
dotnet run

📚 Documentation
Complete Blueprint

Installation Guide

User Manual

Admin Guide

API Documentation

🛠️ Technology Stack
Component	Technology
Backend	.NET 8, C#
Frontend (PC1)	Blazor Server
Gate App	Blazor Hybrid (.NET MAUI)
Mobile App	.NET MAUI Blazor
Database	SQLite
Notifications	Firebase Cloud Messaging
RFID	ACS ACR125U API
📊 System Requirements
Enrollment PC (PC1)
Windows 10/11

8GB RAM minimum

250GB SSD

RFID Reader

Gate PC (PC2, PC3)
Windows 10/11

4GB RAM

120GB SSD

RFID Reader

Monitor for display

🤝 Contributing
Fork the repository

Create your feature branch (git checkout -b feature/AmazingFeature)

Commit changes (git commit -m 'Add AmazingFeature')

Push to branch (git push origin feature/AmazingFeature)

Open a Pull Request

📄 License
This project is licensed under the MIT License - see the LICENSE file for details.

📞 Contact
Project Link: https://github.com/yourusername/EduTrackPH

🙏 Acknowledgments
Department of Education (DepEd) Philippines

Commission on Higher Education (CHED)

All Filipino teachers and administrators

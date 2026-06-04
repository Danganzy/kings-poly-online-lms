[README_kings_poly_lms.md](https://github.com/user-attachments/files/28594824/README_kings_poly_lms.md)
# Kings Poly Online — LMS & Academic Ecosystem

> Production Moodle 5.0 deployment serving 1,000+ distance-learning students across 8 departments, integrated with a custom Laravel results portal, Moodle SSO, and a student application pipeline.

**Live:** [kingspolyonline.com.ng](https://kingspolyonline.com.ng) &nbsp;|&nbsp; **Results Portal:** [results.kingspolyonline.com.ng](https://results.kingspolyonline.com.ng)

![PHP](https://img.shields.io/badge/PHP-8.2-777BB4?logo=php&logoColor=white)
![Laravel](https://img.shields.io/badge/Laravel-11-FF2D20?logo=laravel&logoColor=white)
![Moodle](https://img.shields.io/badge/Moodle-5.0-F98012?logo=moodle&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-VPS-FCC624?logo=linux&logoColor=black)

---

## Overview

Kings Poly Online is the distance-learning platform of Kings Polytechnic Ubiaja, Nigeria. I designed, deployed, and maintain the full academic technology stack — from the LMS through to examination delivery, results management, transcript generation, and student admissions.

This is not a demo or side project. It is a live institutional system in active daily use.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Student Browser                       │
└──────────┬──────────────────────────┬───────────────────┘
           │ Moodle login (SSO)        │ Application portal
           ▼                           ▼
┌─────────────────────┐    ┌──────────────────────────┐
│   Moodle 5.0 LMS    │    │  Laravel Application      │
│   - Course delivery │    │  Portal                   │
│   - Discussions     │◄──►│  - Online application     │
│   - Quizzes / Exams │    │  - Document upload        │
│   - Grade book      │    │  - Paystack fee payment   │
└────────┬────────────┘    │  - Admission notifications│
         │ SSO token        └──────────────────────────┘
         ▼
┌─────────────────────────────────────────────────────────┐
│              Laravel Results Portal                      │
│  Staff upload CSV → auto-grade → GPA compute → PDF      │
│  Student logs in with Moodle credentials (SSO)          │
│  DomPDF transcript generation on demand                  │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│  MySQL Database  (kingspo6_mood330 / kingspo6_results)  │
│  Prefix: mdlbs_  |  PHP 8.2  |  Qservers VPS / cPanel  │
└─────────────────────────────────────────────────────────┘
```

---

## Key Technical Decisions

### 1. Moodle SSO → Results Portal
Rather than building a separate authentication system for the results portal, I integrated Moodle's token-based SSO. Students authenticate once through Moodle and access results transparently. This eliminated duplicate account management and reduced support overhead significantly.

### 2. Results Data Model Redesign
Initial implementation stored `semester` and `level` on the `courses` table. This caused a critical bug: Pre-HND students — who take ND-level courses — had results incorrectly attributed to a unified Pre-HND level, breaking GPA calculations and transcript accuracy.

**Fix:** Dropped `semester` and `level` from courses entirely. Level selection is now explicit at upload time by the staff member submitting results. This enforces correctness at the data entry layer and makes the model accurately reflect institutional reality.

### 3. Flexible CA Slot Architecture
Assessment structure is: 3 × Discussion (10 marks each) + 1 × Quiz (10 marks) = 40 CA + 60 Exam = 100 total. The CA upload is deliberately slot-flexible (staff may have 2 or 4 CA components depending on course) but the system validates that all CA slots sum to exactly 40 before accepting the upload.

### 4. Crisis Recovery — Zero Exam Data Lost
An accidental Moove theme plugin update during an active examination window caused a full platform outage (Moodle dependency conflict + PHP 8.3 incompatibility). Recovery path:
- Restored Moodle files from cPanel backup
- Reverted ea-php83 → ea-php82 in WHM
- Installed the Sodium PHP extension (missing post-restore)
- Patched proctoring plugin `environment.xml` to satisfy version check without a full plugin upgrade
- Configured AutoSSL to restore HTTPS

**Total downtime:** under one working day. Zero examination records lost.

---

## Grading Engine

| Grade | Range   | Grade Point |
|-------|---------|-------------|
| A     | 70–100  | 4.0         |
| B     | 60–69   | 3.0         |
| BC    | 50–59   | 2.5         |
| C     | 40–49   | 2.0         |
| F     | 0–39    | 0.0         |

GPA computed per semester per student. Transcripts generated via DomPDF with institution logo, full course breakdown, and Registrar signature.

---

## Examination Bank

GIFT-format MCQ banks generated across 10+ courses — 60 questions per course with seeded shuffling — directly importable to Moodle with no manual rekeying.

---

## Scale

| Metric | Value |
|--------|-------|
| Active enrolled students | 1,000+ |
| Departments | 8 (ACC, BAM, PAD, COM, EEE, SLT, MAC, BLD) |
| Course shells deployed | 100+ |
| Programme tracks | HND, Pre-HND 1.1, Pre-HND 1.2 |
| Semesters live | Harmattan + Rain cycles |

---

## Stack

- **LMS:** Moodle 5.0
- **Backend:** Laravel 11, PHP 8.2
- **Database:** MySQL 8.0 (prefixed `mdlbs_`)
- **PDF:** DomPDF
- **Payments:** Paystack (application portal)
- **Server:** Qservers VPS, WHM/cPanel, AutoSSL, Linux CLI
- **Auth:** Moodle SSO (token-based)

---

## Screenshots

![Dashboard](screenshots/dashboard.png)
   ![Results Portal](screenshots/results.png)

---

## About

Built, deployed, and maintained by **Oluwafemi Ganzallo** via **KDCS Limited** (RC 1948680), Ikeja, Lagos.  
ORCID: [0009-0008-6198-7044](https://orcid.org/0009-0008-6198-7044)

# StockSense — Technical Documentation

> **"Know your stock. Run your business."**

This repository contains all technical writing deliverables produced for **StockSense v1.0** — a mobile-first, offline-capable inventory and sales management system built for small and medium-sized enterprises (SMEs) across Nigeria and Sub-Saharan Africa.

All documents were produced by the StockSense Technical Writing Team as part of the v1.0 MVP sprint, May 2026.

---

## What is StockSense?

StockSense helps Nigerian shop owners, pharmacists, and wholesalers:

- Track inventory in real time — from any Android smartphone
- Record sales in 3 taps, with or without internet
- Get alerts before stock runs out or medicines expire
- See exactly what every staff member has sold
- Generate and export sales and inventory reports

It is designed for users with basic smartphone skills, unreliable internet, and no tolerance for complicated software. If you can use WhatsApp, you can use StockSense.

---

## Who This Documentation Is For

| Audience | Documents to Read |
|---|---|
| **Business Owners & Shop Staff** | User Help Guide & FAQ |
| **Store Admins & Managers** | Admin Guide & Deployment Runbook |
| **Backend & Frontend Developers** | Developer Onboarding Guide · API Reference Manual |
| **Mobile Developers** | Developer Onboarding Guide · Offline Architecture & Sync Guide |
| **QA Engineers** | Offline Architecture & Sync Guide · API Reference Manual |
| **DevOps / Release Team** | Admin Guide & Deployment Runbook |
| **Product & Design Teams** | Release Notes · Demo Video Script |
| **New Team Members (All Tracks)** | Start with the Developer Onboarding Guide |

---

## Document Index

| # | Document | Format | Audience | Description |
|---|---|---|---|---|
| 1 | [Release Notes v1.0](./01_release-notes-v1.0.md) | Markdown / Word | All teams | Full feature changelog for the MVP launch — what shipped, what was deferred, performance benchmarks, and security highlights |
| 2 | [Demo Video Script](./02_demo-video-script.md) | Markdown / Word | Product · Design · Video | Scene-by-scene script with voiceover, screen recording instructions, and narrator guidance for the 3:30 launch video |
| 3 | [Developer Onboarding Guide](./03_developer-onboarding-guide.md) | Markdown / Word | All developers | Repo structure, local environment setup, API overview, offline architecture summary, CI/CD pipeline, and coding standards |
| 4 | [End-User Help Guide & FAQ](./04_user-help-guide-faq.md) | Markdown / Word | End users · Pilot participants | Step-by-step feature walkthroughs, pilot onboarding packet, and a full FAQ in plain language — no technical jargon |
| 5 | [Admin Guide & Deployment Runbook](./05_admin-guide-deployment-runbook.md) | Markdown / Word | Admins · DevOps | RBAC permission matrix, reconciliation guide, user management, full deployment steps, rollback procedure, and maintenance schedule |
| 6 | [Offline Architecture & Sync Troubleshooting Guide](./06_offline-architecture-sync-guide.md) | Markdown / Word | Mobile · Backend · QA | Deep dive into the offline-first architecture, sync queue design, conflict resolution rules, platform-specific implementation, and troubleshooting playbook |
| 7 | [Complete API Reference Manual](./07_complete-api-reference.md) | Markdown / Word | Backend · Mobile · QA | Full endpoint reference for all 11 API groups — request/response schemas, parameter tables, error codes, and example payloads |

---

## Document Formats

Every document is available in two formats:

- **Markdown (`.md`)** — Lives in this repository. Renders directly on GitHub with full formatting, tables, and code blocks. Use these for quick reference during development.
- **Microsoft Word (`.docx`)** — Available for download. Use these for sharing with stakeholders, printing, or submitting as formal deliverables.

---

## Product Context

| Field | Detail |
|---|---|
| **Product Name** | StockSense |
| **Version** | 1.0 — MVP Launch |
| **Target Market** | Nigerian SMEs — retail shops, pharmacies, wholesalers, mini-marts |
| **Primary Platform** | Android 8.0+ · Progressive Web App (PWA) |
| **Key Differentiator** | Full offline functionality — works without internet on 2G/3G networks |
| **North Star Metric** | Inventory Accuracy Rate — target 90%+ at Day 30 |
| **PRD Version** | v1.1 Merged — May 2026 |

---

## Key Technical Decisions (v1.0)

These decisions are referenced throughout the documentation. If any have changed, the relevant documents should be updated.

| Decision | Choice Made | Status |
|---|---|---|
| Mobile platform | Android-first (iOS in Phase 2) | Confirmed |
| Frontend | React + Next.js PWA | Confirmed |
| Backend | Node.js + Express.js | Confirmed |
| Database | PostgreSQL (primary) + Redis (cache) | Confirmed |
| Local storage — Android | SQLite via Room | Confirmed |
| Local storage — PWA | IndexedDB via Dexie.js | Confirmed |
| Authentication | Phone OTP + JWT (RS256) | Confirmed |
| SMS / OTP provider | Termii or InfoBip | ⚠️ Pending — OQ-001 |
| Background sync — Android | WorkManager | Confirmed |
| Offline sync engine | Custom queue (vs. PowerSync/WatermelonDB) | ⚠️ Pending — OQ-005 |
| Minimum Android API level | API 26 (Android 8.0 Oreo) | ⚠️ Pending — OQ-007 |

> ⚠️ Items marked **Pending** are open questions from the PRD. When resolved, update the relevant documents and remove the flag.

---

## Open Questions Tracker

These were unresolved at the time of documentation. Engineering leads own resolution.

| ID | Question | Owner | Affects |
|---|---|---|---|
| OQ-001 | Which SMS gateway for OTP — Termii or InfoBip? | Backend Lead | Dev Onboarding Guide · API Reference |
| OQ-003 | Is the PWA included in the MVP launch or Android-only? | Product Lead | Release Notes · Dev Onboarding Guide |
| OQ-005 | Offline sync: custom engine or open-source (PowerSync, WatermelonDB)? | Mobile Lead | Offline Architecture Guide · Dev Onboarding Guide |
| OQ-006 | Backend architecture: monolith or microservices? | Backend Lead | Dev Onboarding Guide · API Reference |
| OQ-007 | Minimum Android API level — confirmed as API 26? | Mobile Lead | Release Notes · Dev Onboarding Guide |
| OQ-008 | Is dark mode in MVP scope? | Design Lead | Release Notes |
| OQ-009 | Biometric authentication — MVP or Phase 2? | Product Lead · Mobile Lead | Release Notes · Admin Guide |

---

## Sprint & Release Timeline

| Sprint | Goal | Status |
|---|---|---|
| Week 1 | Foundation — Auth, Products, Offline shell | In Progress |
| Week 2 | Core Features — Sales, Alerts, Supplier, Sync | Upcoming |
| Week 3 | Intelligence & Polish — Reports, Reconciliation, Dashboard | Upcoming |
| Week 4 | QA, Security Audit, UAT, Production Launch | Upcoming |

---

## How to Use This Repository

### Viewing documents on GitHub
Click any `.md` file in the file list above. GitHub renders it automatically as a formatted page — headings, tables, code blocks, and all.

### Downloading Word documents
The `.docx` versions of all 7 documents are available as downloadable files. Click the file, then click the **Download** button.

### Suggesting a change
If you spot an inaccuracy — a wrong API endpoint, an outdated permission, a feature description that doesn't match what was built — please:
1. Open an **Issue** on this repository describing what needs to change and why
2. Or contact the Technical Writing team directly via the team communication channel

### For developers setting up locally
Start with the [Developer Onboarding Guide](./03_developer-onboarding-guide.md). Everything you need — repo structure, environment variables, database setup, and how to run the API — is in Section 4.

---

## Technical Writing Team

This documentation was produced by the **StockSense Technical Writing Team** as a capstone deliverable for the Techcrush Technical Writing Bootcamp, in collaboration with the StockSense product and engineering teams.

| Role | Responsibility |
|---|---|
| Technical Writers | All 7 documents — research, structure, writing, and review |
| Engineering Lead | Technical accuracy review |
| Product Manager | Scope and PRD alignment review |
| Design Lead | UX copy and wireframe description review |

---

## Version History

| Version | Date | Changes |
|---|---|---|
| 1.0 | May 2026 | Initial release — all 7 documents published for v1.0 MVP launch |

---

*StockSense Technical Documentation · v1.0 · May 2026*
*Produced by the StockSense Technical Writing Team · Techcrush Technical Writing Bootcamp*

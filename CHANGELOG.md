# Changelog

All notable changes to StockSense documentation will be recorded in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) conventions.
Versions align with the StockSense product release cycle.

---

## [1.0.1] — June 2026 — Database Correction

### Changed

- **03 Developer Onboarding Guide** — Updated database technology from PostgreSQL + Prisma to
  MongoDB (Atlas / local) + Mongoose, reflecting the actual implementation used in development.
  Updated environment variable from `DATABASE_URL` to `MONGODB_URI`, replaced Prisma migration
  commands with `npm run seed`, updated DB schema conventions to use MongoDB `ObjectId` and
  Mongoose `timestamps`, and corrected collection naming to camelCase.

- **05 Admin Guide & Deployment Runbook** — Updated production infrastructure table to MongoDB
  Atlas. Replaced `pg_dump` backup command with `mongodump` and `pg_restore` rollback command
  with `mongorestore`. Updated pre-deployment checklist to remove Prisma migration references.

- **06 Offline Architecture & Sync Guide** — Updated local schema mirroring note from
  PostgreSQL to MongoDB collections.

- **07 Complete API Reference Manual** — Updated monetary value type in the data types appendix
  from `DECIMAL(12,2)` to MongoDB `Number` serialised as a 2-decimal string in JSON.

- **README.md** — Updated Key Technical Decisions table: database entry changed from
  PostgreSQL to MongoDB (Atlas / local).

---

## [1.0.0] — May 2026 — MVP Launch

This is the first full release of StockSense technical documentation,
produced for the v1.0 MVP launch.

### Added

- **01 Release Notes v1.0** — Full feature changelog for the MVP launch,
  covering all 8 feature epics, performance benchmarks, known limitations,
  security highlights, and upgrade instructions for Android and PWA.

- **02 Demo Video Script** — Complete 3:30 scene-by-scene production script
  with voiceover, screen recording instructions, narrator guidance, and a
  pre-recording device checklist.

- **03 Developer Onboarding Guide** — Repository structure, local environment
  setup for backend/frontend/mobile tracks, API overview, offline architecture
  summary, CI/CD pipeline, security requirements, and open questions tracker.

- **04 End-User Help Guide & FAQ** — Three-part document covering the pilot
  user onboarding packet, full feature walkthroughs for all core flows, and a
  comprehensive FAQ in plain language for Nigerian SME users.

- **05 Admin Guide & Deployment Runbook** — Two-part document covering the
  business owner admin guide (RBAC matrix, reconciliation walkthrough, user
  management, system settings) and the full production deployment runbook
  (pre-deployment checklist, deployment steps, smoke tests, incident response,
  rollback procedure, maintenance schedule).

- **06 Offline Architecture & Sync Troubleshooting Guide** — Deep dive into
  the offline-first architecture, sync queue design and schema, write/read
  flows, connectivity detection, conflict resolution rules, platform-specific
  implementation (Android WorkManager + PWA Service Worker), offline data
  cache table, and a full troubleshooting playbook for users and developers.

- **07 Complete API Reference Manual** — Full endpoint reference for all 11
  API groups (Authentication, Products, Sales, Stock Adjustments, Suppliers,
  Purchase Orders, Alerts, Reconciliation, Reports, Sync, User Management)
  with request/response schemas, parameter tables, error codes, and example
  payloads.

- **README.md** — Repository front page with project overview, audience
  guide, full document index, product context, key technical decisions,
  open questions tracker, and sprint timeline.

### Product Version
- Aligned to StockSense PRD v1.1 (Merged) — May 2026

### Platforms Covered
- Android 8.0+ (Kotlin / Flutter — TBD)
- Progressive Web App (React + Next.js)
- REST API (Node.js + Express.js)

---

## [Unreleased] — Upcoming

Changes planned for future documentation releases.

### Planned for Phase 2 (Months 2–4)
- WhatsApp Integration Guide — when WhatsApp alert and receipt features ship
- Barcode Scanning Guide — when ZXing / QuaggaJS integration ships
- Multi-Store Management Guide — when multi-location support ships
- iOS User Guide — when native iOS app ships
- Local Language Supplement — Yoruba, Hausa, Igbo UI terminology guide

### Planned for Phase 3 (Months 7–12)
- AI Demand Forecasting Guide
- WhatsApp Chatbot User Guide
- Supplier E-Ordering Integration Guide
- Paystack / Flutterwave Billing Guide

### Pending Resolution (Open Questions)
The following documents will be updated once open questions are resolved
by the engineering team:

- OQ-001: SMS gateway decision (Termii vs InfoBip) →
  Update Developer Onboarding Guide + API Reference
- OQ-003: PWA in MVP scope confirmation →
  Update Release Notes + Developer Onboarding Guide
- OQ-005: Offline sync engine decision →
  Update Offline Architecture Guide + Developer Onboarding Guide
- OQ-006: Monolith vs microservices decision →
  Update Developer Onboarding Guide + API Reference
- OQ-007: Minimum Android API level confirmation →
  Update Release Notes + Developer Onboarding Guide

---

## Version Format Guide

| Tag | Meaning |
|---|---|
| `Added` | New documents or major new sections added |
| `Changed` | Updates to existing content (e.g., corrected API endpoint, updated RBAC rule) |
| `Fixed` | Corrections to errors, broken links, or inaccurate technical details |
| `Removed` | Documents or sections removed from the repository |
| `Deprecated` | Content that is outdated but kept for reference |

---

*StockSense Documentation Changelog · Maintained by the Technical Writing Team*

# StockSense — Developer Onboarding Guide

**Audience:** Backend, Frontend, Mobile Developers
**Applies To:** StockSense v1.0 MVP — All Engineering Tracks
**Prerequisites:** Git, Node.js 18+, MongoDB (Atlas or local), Android Studio (mobile track), Android Studio (mobile track)
**Owner:** Engineering Lead

---

## 1. Product Overview

StockSense is a mobile-first, offline-capable inventory and sales management system built for SMEs across Nigeria and Sub-Saharan Africa. It is designed to work reliably on 2G/3G connections and to function fully without any internet access.

As a developer on this project, your work must honour three non-negotiable product constraints:

- **Offline-first:** All core write operations must work without a network connection and sync when connectivity is restored.
- **Performance on low-end devices:** Target Android 8.0+, minimum 2GB RAM. Bundle size must stay under 30MB for the APK.
- **Role-Based Access Control (RBAC):** Every API endpoint and UI screen must enforce role permissions (Admin, Manager, Attendant).

---

## 2. System Architecture

### 2.1 Three-Tier Architecture

| Layer | Technology | Responsibility |
|---|---|---|
| **Client** | React PWA / Android (Kotlin or Flutter) | UI, offline storage (IndexedDB/SQLite), service worker, sync queue |
| **API** | Node.js + Express.js | REST API, JWT auth, RBAC middleware, rate limiting, business logic |
| **Data** | MongoDB (Atlas/local) + Redis + S3 | Persistent storage, caching, job queues, file exports |

### 2.2 Authentication

- **Primary:** Phone number + OTP (via Termii or InfoBip — TBD, see Open Question OQ-001)
- **Secondary:** Email + password (bcrypt hash, min 12 rounds)
- **Attendant:** 4-digit PIN (bcrypt hash) for quick login on shared devices
- **Token type:** JWT with RS256 signing. Access token: 24h. Refresh token: 30 days.
- **RBAC roles:** `admin`, `manager`, `attendant` — encoded in the JWT payload

---

## 3. Repository Structure

```
/stocksense
  /backend                → Node.js + Express API
    /src
      /routes             → Express route definitions
      /controllers        → Route handler logic
      /services           → Business logic (alerts engine, sync, reports)
      /middleware         → RBAC, JWT verification, rate limiting
      /models             → Mongoose schemas + DB access layer
      /jobs               → Bull queue workers (report gen, alert eval)
      /utils              → Helpers (error handlers, validators)
    /tests
    .env.example
  /frontend               → React + Next.js PWA
    /src
      /components         → Reusable UI components
      /pages              → Next.js page routes
      /store              → Redux Toolkit / Zustand state
      /services           → API client, offline sync engine
      /db                 → Dexie.js (IndexedDB) schema and access
    /public               → Static assets, manifest.json, service worker
  /mobile                 → Android app (Kotlin / Flutter — TBD)
  /docs                   → Technical documentation
  /scripts                → CI/CD, migrations, seed data
```

---

## 4. Environment Setup

### 4.1 Prerequisites

- Node.js 18 LTS or higher
- MongoDB 6.0 or higher (local) OR a free MongoDB Atlas account
- Redis 7 or higher
- Git 2.40+
- Android Studio (Flamingo or newer) — mobile track only
- Docker (optional but recommended for local database setup)

### 4.2 Backend Setup

**1. Clone the repository:**
```bash
git clone https://github.com/your-org/stocksense.git
cd stocksense/backend
```

**2. Install dependencies:**
```bash
npm install
```

**3. Copy and configure environment variables:**
```bash
cp .env.example .env
```

Edit `.env` with the following required values:
```
MONGODB_URI=mongodb://localhost:27017/stocksense
# OR for MongoDB Atlas:
# MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net/stocksense
REDIS_URL=redis://localhost:6379
JWT_PRIVATE_KEY=<RS256 private key>
JWT_PUBLIC_KEY=<RS256 public key>
OTP_PROVIDER_KEY=<Termii or InfoBip API key>
OTP_PROVIDER=termii
NODE_ENV=development
```

**4. Seed the database with initial data:**
```bash
npm run seed
```

**5. Start the development server:**
```bash
npm run dev
```

> API available at `http://localhost:3001`. Swagger docs at `http://localhost:3001/api-docs`.

> ⚠️ **Never commit your `.env` file.** It is listed in `.gitignore` but always double-check before pushing.

### 4.3 Frontend Setup

```bash
cd ../frontend
npm install
cp .env.local.example .env.local
# Set: NEXT_PUBLIC_API_URL=http://localhost:3001
npm run dev
```

> PWA available at `http://localhost:3000`

### 4.4 Mobile Setup

Refer to `/mobile/README.md`. Key points:
- Minimum Android SDK target: API 26 (Android 8.0 Oreo)
- Local database: SQLite via Room
- Background sync: WorkManager
- Push notifications: Firebase Cloud Messaging — add `google-services.json` to `/mobile/app/`

---

## 5. API Reference

### 5.1 Base URL

| Environment | URL |
|---|---|
| Production | `https://api.stocksense.africa/v1/` |
| Development | `http://localhost:3001/v1/` |

### 5.2 Authentication Header

```
Authorization: Bearer <access_token>
```

> RBAC middleware checks the `role` field in the JWT payload on every protected request. Roles in ascending permission level: `attendant` < `manager` < `admin`.

> ⚠️ Any endpoint that modifies data (POST, PUT, DELETE) requires at minimum the `manager` role, except sales recording which is available to all authenticated roles.

### 5.3 Standard Response Format

```json
// Success
{
  "success": true,
  "data": { },
  "message": "Operation completed successfully.",
  "meta": { "page": 1, "total": 50, "timestamp": "2026-05-01T09:00:00Z" }
}

// Error
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Product name is required.",
    "field": "name"
  },
  "timestamp": "2026-05-01T09:00:00Z"
}
```

### 5.4 Endpoint Groups

| Group | Base Path | Key Endpoints |
|---|---|---|
| Authentication | `/auth` | POST /register, /verify-otp, /login, /login/pin, /logout, /refresh, /invite |
| Products | `/products` | GET /, POST /, GET /:id, PUT /:id, DELETE /:id, POST /:id/stockin, POST /:id/stockout |
| Sales | `/sales` | GET /, POST /, GET /:id, POST /:id/void, GET /summary/today, GET /by-attendant |
| Suppliers | `/suppliers` | GET /, POST /, PUT /:id, DELETE /:id |
| Purchase Orders | `/purchase-orders` | GET /, POST /, PUT /:id |
| Alerts | `/alerts` | GET /, PUT /:id/dismiss, PUT /:id/resolve |
| Reconciliation | `/reconciliation` | POST /, GET /:id, PUT /:id, POST /:id/finalize, GET /history |
| Reports | `/reports` | GET /sales, GET /inventory, GET /audit-log, GET /export |
| Sync | `/sync` | POST / (upload queue), GET /pull (delta fetch) |
| Users | `/users` | GET /, PUT /:id/role, DELETE /:id |

---

## 6. Offline-First Architecture

### 6.1 How It Works

The offline-first pattern means every write operation is written to local storage first, then synced to the server in the background. The UI never waits for a server response.

**Flow for any write (e.g., recording a sale):**
1. Attendant confirms the sale in the UI.
2. Operation written immediately to local database (IndexedDB / SQLite).
3. UI updates optimistically — stock deducted from local count.
4. Operation appended to the `sync_queue` table in local storage.
5. Background service detects connectivity.
6. Sync service sends queued operations to `POST /sync` in order.
7. Server returns: `{ applied, conflicts, server_time }`.
8. If conflicts exist, owner is notified via an in-app alert.

### 6.2 Conflict Resolution Rules

| Conflict | Resolution |
|---|---|
| Same product stock modified on two offline devices | Most recent `server_time` wins; flag to owner if stock goes negative |
| Sale recorded for a product deleted online | Sale preserved; product shown as archived |
| Same sale voided on two devices | First void wins; second ignored (idempotent) |
| Price changed offline while sales were made | Sales use offline price; owner notified |
| Stock goes negative after sync | Flagged to owner dashboard for manual review |

### 6.3 Retry Strategy (Exponential Backoff)

| Attempt | Delay |
|---|---|
| 1st | 30 seconds |
| 2nd | 2 minutes |
| 3rd | 10 minutes |
| 4th | 30 minutes |
| 5th+ | Hourly |

After 3 failed retries: queue item flagged as **Sync Failed**. User sees: *"X sales failed to sync. Tap to retry."*

> ⚠️ `sync_queue` is **append-only**. Never modify or delete items from it. Only update the `status` field.

---

## 7. Database Schema Overview

**Conventions:**
- All primary keys are MongoDB `ObjectId` (auto-generated `_id` field)
- All monetary values stored as `Number` in Nigerian Naira (NGN) — use `toFixed(2)` for display
- All timestamps: ISO 8601 UTC — use Mongoose `Date` type
- All collections have: `createdAt`, `updatedAt` (via Mongoose `timestamps: true`), `deletedAt` (soft delete)

**Core collection relationships:**
- `businesses` ← root collection. All other collections reference `businessId`
- `users` → `businessId`
- `products` → `businessId`, `categoryId`, `supplierId`
- `stockMovements` → `productId`, `businessId`, `userId` — **IMMUTABLE APPEND-ONLY**
- `sales` → `businessId`, `attendantId`
- `saleItems` → `saleId`, `productId` — snapshots `productName` and `unitPrice`
- `suppliers` → `businessId`
- `purchaseOrders` → `businessId`, `supplierId`, `createdBy`
- `alerts` → `businessId`, `productId`
- `reconciliations` + `reconciliationItems` → `businessId`

> 🚫 The `stockMovements` collection is the immutable audit log. **No update or delete operations are ever permitted on this collection.**

---

## 8. Security Requirements

### 8.1 Required for All Endpoints
- HTTPS only (TLS 1.2+ required; TLS 1.3 preferred)
- Valid JWT Bearer token on all non-public endpoints
- RBAC middleware runs before any business logic
- Input validation on all request parameters
- Parameterised queries only — use Mongoose methods, never raw string concatenation in queries
- Rate limiting: 100 req/min per user; 20 req/min for public endpoints

### 8.2 Token & Session Rules
- JWT signed with RS256. Store private key in env variable; never hardcode.
- Access token: 24h. Refresh token: 30 days.
- Session invalidated on password change or role update.
- OTP rate limit: 3 per phone per 15 minutes.
- Account lock: 15 minutes after 5 failed login attempts.

---

## 9. CI/CD Pipeline (GitHub Actions)

| Trigger | Actions |
|---|---|
| Every Pull Request | Run tests (`npm test`), lint (`npm run lint`), Mongoose schema validation, build check |
| Merge to `main` | Deploy to staging, run smoke tests |
| Release tag (e.g. `v1.0.0`) | Deploy to production, send team notification |

> ⚠️ Production deployments require Engineering Lead approval. Never deploy to production from a local machine.

---

## 10. Development Standards

### Git Workflow
- Branch naming: `feature/name`, `fix/name`, `chore/name`
- Commit messages: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
- All feature branches merge via pull request with at least one reviewer

### Error Codes

All API errors must return the standard error envelope format with an appropriate HTTP status code. Use the common error codes defined in the API contract (refer to [`07_complete-api-reference.md` — Section 1.3](./07_complete-api-reference.md)):

| Code | HTTP | Meaning |
|---|---|---|
| `UNAUTHORIZED` | 401 | Invalid or expired token |
| `FORBIDDEN` | 403 | Role does not have permission |
| `NOT_FOUND` | 404 | Resource does not exist |
| `INSUFFICIENT_STOCK` | 422 | Sale quantity exceeds stock |
| `VALIDATION_ERROR` | 422 | Required field missing or invalid |
| `CONFLICT` | 409 | Sync conflict detected |
| `SERVER_ERROR` | 500 | Unexpected server error |

### Testing Requirements
- Unit tests for all service-layer functions
- Integration tests for all API endpoints (happy path + error cases)
- Offline scenario tests: simulate airplane mode, record operations, verify sync
- RBAC tests: verify each role can/cannot access correct endpoints

---

## 11. Open Questions

| ID | Question | Owner | Due |
|---|---|---|---|
| OQ-001 | Which SMS gateway for OTP? (Termii, InfoBip?) | Backend Lead | Before Sprint 1 |
| OQ-003 | Web PWA in MVP or Android-only launch? | Product Lead | Before Sprint 1 |
| OQ-005 | Offline sync: custom engine or open-source? (PowerSync, WatermelonDB?) | Mobile Lead | Sprint 1 |
| OQ-006 | Backend architecture: monolith or microservices? | Backend Lead | Sprint 1 |
| OQ-007 | Minimum Android API level: API 26 confirmed? | Mobile Lead | Sprint 1 |

---

*StockSense · Developer Onboarding Guide v1.0 · May 2026 · Internal / Confidential*

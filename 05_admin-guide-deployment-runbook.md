# StockSense — Admin Guide & Deployment Runbook

*Includes: RBAC Permission Matrix, Reconciliation Guide & Deployment Procedures*
**Version 1.0 · May 2026 · Internal / Confidential**

---

# PART 1: ADMIN (OWNER) GUIDE

---

## 1. Dashboard Interpretation

Open your dashboard every morning and check these four areas:

| Indicator | Colour Code | Action |
|---|---|---|
| Stock Status — Green | All products above reorder level | No action needed |
| Stock Status — Amber | Some products approaching threshold | Review low stock list and consider reordering |
| Stock Status — Red | Products at or below reorder level | Act immediately — create a purchase order |
| Active Alerts badge | Red number on the Alerts icon | Tap Alerts to review and action each one |
| Sync badge on header | Number of pending sync operations | Check your internet connection |

---

## 2. Role & Permission Matrix (RBAC)

| Feature / Action | Admin (Owner) | Manager | Attendant |
|---|:---:|:---:|:---:|
| Login and access the app | ✓ | ✓ | ✓ |
| View Dashboard — full revenue data | ✓ | ✓ | — |
| View Dashboard — limited (no revenue) | ✓ | ✓ | ✓ |
| Add / Edit Products | ✓ | ✓ | — |
| Delete (Archive) Products | ✓ | ✓ | — |
| View Product List | ✓ | ✓ | ✓ |
| Record Stock In | ✓ | ✓ | — |
| Record Manual Stock Out | ✓ | ✓ | — |
| Record Sales | ✓ | ✓ | ✓ |
| Void a Sale | ✓ | ✓ | — |
| View Own Sales History | ✓ | ✓ | ✓ |
| View All Sales History | ✓ | ✓ | — |
| View Per-Attendant Sales Breakdown | ✓ | ✓ | — |
| Stock Adjustments | ✓ | ✓ | — |
| View Audit Log | ✓ | ✓ | — |
| Manage Suppliers | ✓ | ✓ | — |
| Create Purchase Orders | ✓ | ✓ | — |
| Start Reconciliation | ✓ | ✓ | — |
| Finalize Reconciliation | ✓ | — | — |
| View Reports | ✓ | ✓ | — |
| Export Reports | ✓ | Delegated by Admin | — |
| Manage Users and Roles | ✓ | — | — |
| Change System Settings | ✓ | — | — |
| Set Alert Thresholds | ✓ | ✓ | — |
| Receive Low-Stock Alerts | ✓ | ✓ | ✓ |
| Receive Expiry Alerts | ✓ | ✓ | — |

> ℹ️ Manager export of reports requires explicit delegation from the Admin. Go to **Settings > Staff Permissions** to grant or revoke this per Manager.

---

## 3. Inventory Reconciliation — Admin Guide

### 3.1 Why Run a Reconciliation?

Reconciliation answers: *"Does what StockSense says I have match what is actually on my shelf?"*

Run a reconciliation when:
- You suspect theft or pilferage by staff
- A large delivery was received and you want to verify counts
- Your stockout frequency is higher than expected
- End of month or quarterly audit is due
- A new staff member is taking over stock management

### 3.2 Step-by-Step Reconciliation

1. Go to **More > Reconciliation**.
2. Tap **Start New Count**. A new session is created and timestamped with your name.
3. The screen shows every active product with its **System Quantity** (what StockSense expects).
4. Walk your store physically and count each item. Enter the **Actual Quantity** next to each product.
5. The **Variance** column updates live. Positive = more than expected. Negative = stock is missing.
6. **Variance Value in Naira** shows the financial impact.
7. Tap **Finalize Count** when done.
8. For each variance, choose:
   - **Accept Variance** — updates system stock to physical count, logs the adjustment with your name
   - **Flag for Investigation** — marks the item without adjusting stock

> 🚫 **Accepted variances are permanent.** You cannot undo an accepted variance. If you made a mistake, create a new stock adjustment with a correction reason.

### 3.3 Investigating Flagged Items

1. Go to **Inventory > [Product] > Stock History** to review every movement.
2. Compare recorded sales against stock movements.
3. Check which staff member made recent movements using the user attribution on each record.
4. If you confirm a discrepancy, create a Stock Adjustment with reason **"Investigation Outcome."**
5. Return to the reconciliation session in History and mark the item as Resolved.

---

## 4. User Management

### 4.1 Inviting a New Staff Member

1. Go to **Settings > Manage Staff**.
2. Tap **Invite User**.
3. Enter the staff member's phone number.
4. Select their role: Manager or Attendant.
5. Tap **Send Invite** — they receive an SMS with a setup link.

> ℹ️ If the SMS doesn't arrive within 5 minutes, check the number was entered correctly with the +234 prefix, then resend.

### 4.2 Changing a Staff Member's Role

1. Go to **Settings > Manage Staff**.
2. Tap the staff member's name.
3. Tap **Change Role** and select the new role.
4. Tap **Confirm**.

> 🚫 When you change a role, their current session is immediately terminated and they must log in again. This is a security feature.

### 4.3 Deactivating a Staff Member

1. Go to **Settings > Manage Staff**.
2. Tap the staff member's name.
3. Tap **Deactivate Account** and confirm.

Access is immediately revoked. All past sales and actions remain in the audit log.

---

## 5. System Settings

### 5.1 Alert Configuration

Go to **Settings > Notifications** to toggle:
- **Low-Stock Alerts** — on or off globally or per product
- **Expiry Alerts** — fires at 30, 14, and 7 days before expiry
- **Sync Alerts** — notifies you if a sync operation fails

### 5.2 Subscription Plans

| Plan | Price | Limits |
|---|---|---|
| Free | ₦0 | Up to 50 products, 3 users, basic reports |
| Starter | ₦500/month | Up to 200 products, 10 users, full reports, PDF export |
| Pro | ₦2,000/month | Unlimited products and users, all reports, priority support |

---

# PART 2: DEPLOYMENT RUNBOOK

---

## 6. Production Environment Overview

| Component | Provider | Notes |
|---|---|---|
| API Server | Railway / Render / DigitalOcean | Node.js 18 LTS. Minimum 1GB RAM. |
| Database | MongoDB Atlas (cloud) or MongoDB 6.0 (local) | Backups every 6 hours, 30-day retention. |
| Cache & Queues | Redis 7 (managed) | Session cache, rate limiting, Bull job queues. |
| File Storage | AWS S3 or Cloudflare R2 | PDF export files. |
| CDN / Frontend | Vercel or Netlify | Next.js PWA. Edge caching. |
| Monitoring | Sentry + Uptime Robot | Error tracking, uptime checks every 5 minutes. |
| CI/CD | GitHub Actions | Automated build, test, and deploy. |
| SMS (OTP) | Termii or InfoBip | OQ-001: provider TBD before Sprint 1. |
| Push Notifications | Firebase Cloud Messaging | Add `google-services.json` to Android build. |

---

## 7. Pre-Deployment Checklist

### Code & Build
- [ ] All pull requests merged and approved by Engineering Lead
- [ ] All CI checks passing (tests, lint, build)
- [ ] Mongoose schema changes reviewed and approved
- [ ] No open P0 or P1 bugs
- [ ] APK size verified: under 30MB
- [ ] PWA Lighthouse score reviewed

### Environment Variables
Confirm all of the following are set in production — **never commit these to source control:**
- [ ] `DATABASE_URL`
- [ ] `REDIS_URL`
- [ ] `JWT_PRIVATE_KEY` and `JWT_PUBLIC_KEY`
- [ ] `OTP_PROVIDER_KEY`
- [ ] `OTP_PROVIDER`
- [ ] `FCM_SERVER_KEY`
- [ ] `S3_BUCKET`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`
- [ ] `SENTRY_DSN`
- [ ] `NODE_ENV=production`
- [ ] `CORS_ORIGIN`

### Security
- [ ] SSL certificate valid for `api.stocksense.africa`
- [ ] CORS configuration restricted to production frontend domain
- [ ] Rate limiting confirmed active
- [ ] All input validation middleware confirmed in all routes

### Database
- [ ] Production database backup taken before migration
- [ ] Mongoose seed data verified on staging
- [ ] MongoDB indexes created on: `productId`, `businessId`, `createdAt`

---

## 8. Deployment Steps

> 🚫 **Always take a full database backup before deploying to production.**

### 8.1 Backend Deployment

```bash
# 1. Create a database backup
mongodump --uri="$MONGODB_URI" --out=./backup_$(date +%Y%m%d_%H%M)

# 2. Pull the release branch
git checkout release/v1.0
git pull origin release/v1.0

# 3. Seed initial data (if required)
npm run seed

# 4. Build the application
npm run build

# 5. Deploy to production server
railway up --service stocksense-api

# 6. Verify the health check
curl https://api.stocksense.africa/v1/health
```

### 8.2 Frontend Deployment

```bash
npm run build
vercel --prod
```

### 8.3 Android APK Release

```bash
# Build the release APK
./gradlew assembleRelease

# Sign the APK
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
  -keystore stocksense.keystore app-release-unsigned.apk alias_name

# Align and verify
zipalign -v 4 app-release-unsigned.apk stocksense-v1.0.apk
```

Verify APK size is under 30MB before distributing.

---

## 9. Post-Deployment Verification (Smoke Tests)

- [ ] Register a new business account via phone OTP — verify SMS arrives within 60 seconds
- [ ] Log in and verify dashboard loads
- [ ] Add a test product and confirm it appears in inventory
- [ ] Record a test sale and verify stock is deducted
- [ ] Enable airplane mode, record a sale, reconnect, verify sale syncs
- [ ] Invite a staff member as Attendant — verify restricted access
- [ ] Generate a PDF report and verify it downloads
- [ ] Confirm Sentry is receiving events
- [ ] Confirm Uptime Robot is monitoring the health endpoint

---

## 10. Incident Response

### Severity Levels

| Severity | Definition | Example | Response Target |
|---|---|---|---|
| **P1 — Critical** | System down or data at risk | API unreachable, DB corruption | < 2 hours MTTR |
| **P2 — High** | Major feature unavailable | Sales cannot be recorded | < 4 hours |
| **P3 — Medium** | Feature degraded | Reports generating slowly | < 24 hours |
| **P4 — Low** | Minor issue | UI misalignment | Next sprint |

### Rollback Procedure

> 🚫 **Only initiate a rollback if a P1 cannot be resolved within 30 minutes of deployment.**

```bash
# 1. Revert via Railway/Render dashboard (Previous Deployment button)

# 2. If database was seeded or modified, restore from backup:
mongorestore --uri="$MONGODB_URI" ./backup_YYYYMMDD_HHMM

# 3. Verify health check returns 200
# 4. Run smoke tests (Section 9)
# 5. Document the incident: timestamp, root cause, steps taken, resolution
```

---

## 11. Maintenance Schedule

| Task | Schedule | Details |
|---|---|---|
| Database Backup | Every 6 hours | Automated. 30-day retention. Verify monthly. |
| Maintenance Window | Sunday 02:00–04:00 WAT | Notify users 24h in advance. |
| SSL Certificate Renewal | 60 days before expiry | Auto-renew via Let's Encrypt. |
| Dependency Security Audit | Monthly | Run `npm audit`. Patch critical vulnerabilities within 48h. |
| Log Rotation | Daily | App logs: 90 days. Error logs: 1 year. |
| Uptime Report Review | Weekly | Investigate any availability < 99.5%. |

---

*StockSense · Admin Guide & Deployment Runbook v1.0 · May 2026 · Internal / Confidential*

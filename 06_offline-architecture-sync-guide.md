# StockSense — Offline Architecture & Sync Troubleshooting Guide

**Audience:** Mobile developers, backend engineers, QA engineers, support team
**Applies To:** StockSense v1.0 — Android app and Progressive Web App (PWA)
**Key Technologies:** SQLite/Room (Android), IndexedDB/Dexie.js (PWA), WorkManager, Service Worker
**Owner:** Mobile Lead & Backend Lead

---

## 1. Why Offline-First?

Nigeria's average mobile internet speed is 14 Mbps, but connectivity is highly intermittent. SME operators in our target market report losing network access **3 to 6 times per working day**. In non-urban areas, connectivity gaps can last hours.

This is not a rare edge case — it is the **normal operating environment** for StockSense users.

| Design Approach | Behaviour |
|---|---|
| **Offline-first (StockSense)** | All writes go to local storage first. Sync happens in the background. App always responds instantly. |
| Online-first *(rejected)* | Writes go to server first. App is slow or broken without internet. |
| Online-only *(rejected)* | No local storage. Completely unusable without internet. |

---

## 2. Offline Architecture Overview

### 2.1 The Two Local Databases

| Platform | Engine | Library | Target Max Footprint |
|---|---|---|---|
| Android (Kotlin / Flutter) | SQLite | Room Persistence Library | Under 50MB |
| PWA (Chrome / Web) | IndexedDB | Dexie.js | Under 50MB |

> ℹ️ The local schema mirrors the MongoDB server collections field-for-field. This ensures offline and online queries use the same logic, and sync mapping stays straightforward.

### 2.2 The Sync Queue

Every write operation is recorded in a local `sync_queue` table before being applied to the main local tables.

**`sync_queue` Schema:**
```sql
CREATE TABLE sync_queue (
  id            TEXT PRIMARY KEY,        -- UUID generated on device
  operation     TEXT NOT NULL,           -- JSON: { type, entity, payload }
  entity_id     TEXT,                    -- ID of the affected record
  created_at    INTEGER NOT NULL,        -- Unix timestamp (device time, ms)
  status        TEXT DEFAULT 'pending',  -- pending | synced | failed | conflict
  retry_count   INTEGER DEFAULT 0,
  last_error    TEXT
);
```

**Example operation payload (sale):**
```json
{
  "id": "a1b2c3d4-...",
  "movement_type": "sale",
  "entity": "sale",
  "payload": {
    "id": "a1b2c3d4-...",
    "attendant_id": "u-001",
    "device_time": "2026-05-01T09:15:00Z",
    "items": [{ "product_id": "p-001", "quantity": 2, "unit_price": "350.00" }],
    "total_amount": "700.00",
    "payment_method": "cash"
  }
}
```

> 🚫 The `sync_queue` is **append-only**. Never UPDATE or DELETE rows. Only transition the `status` field.

### 2.3 Write Flow (Online or Offline)

Every write follows this sequence regardless of connectivity:

1. User action triggers a write (e.g., attendant confirms a sale).
2. Operation written to `sync_queue` with `status = 'pending'`.
3. Local database tables updated immediately (optimistic update).
4. UI updates instantly — user sees the change without waiting for the server.
5. Background sync service checks connectivity.
6. If online: sync queue is processed. If offline: queue waits.

> ⚠️ The UI must **never** wait for a server response before updating. All UI updates are optimistic — applied immediately from local data.

### 2.4 Read Flow

All reads come from the local database, never directly from the server API:

1. UI requests data (e.g., product list, dashboard summary).
2. Data served from local SQLite / IndexedDB tables.
3. If online, a background delta sync keeps local data fresh.
4. The UI never shows a loading spinner waiting for the server on a read.

### 2.5 Connectivity Detection

| Platform | API Used | Behaviour |
|---|---|---|
| Android | `ConnectivityManager` + `NetworkCallback` | Registers a callback for WiFi and mobile data. Fires sync on network available. |
| PWA | `navigator.onLine` + online/offline events | Also runs a periodic beacon ping to confirm real connectivity (not just adapter state). |

> ⚠️ `navigator.onLine` returns `true` if the device has a network adapter active, even without an actual internet route. Always confirm with a lightweight beacon ping (`HEAD /v1/health`) before syncing.

### 2.6 Connectivity Status Indicator

Every screen shows a persistent status bar:
- 🟢 **Online — All data synced:** connected and queue is empty
- 🟡 **Online — X actions syncing:** sync in progress
- 🔴 **Offline — X actions queued:** no connection detected
- ⚠️ **Sync Failed — Tap to retry:** one or more queue items failed after max retries

---

## 3. Sync Engine

### 3.1 Sync Trigger Conditions

The sync engine fires when:
- Network connectivity is restored
- App returns to foreground and device is online
- User manually taps **Retry** on a failed sync notification
- Periodic background check every 5 minutes (Android WorkManager)

### 3.2 Sync Process — Step by Step

1. Connectivity check: confirm online via beacon ping before starting.
2. Lock the queue: set a processing flag to prevent concurrent sync runs.
3. Fetch pending items: `SELECT * FROM sync_queue WHERE status = 'pending' ORDER BY created_at ASC`.
4. Process in order — operations applied in exact order they were created on device.
5. For each operation, `POST` to `/sync` with the operation payload.
6. On success (200): update queue item `status` to `'synced'`.
7. On conflict (409): update `status` to `'conflict'`. Notify owner.
8. On error (4xx/5xx/timeout): increment `retry_count`. Apply backoff. Status stays `'pending'`.
9. After max retries: update `status` to `'failed'`. Show **Sync Failed** notification.
10. Release lock when queue is empty.

**Batch sync endpoint:**
```json
// POST /v1/sync
{
  "device_id": "device-uuid",
  "last_sync_at": "2026-05-01T08:00:00Z",
  "operations": [
    { "id": "queue-uuid-1", "movement_type": "sale", "entity": "sale", "payload": { "..." } }
  ]
}

// Response
{
  "applied": ["queue-uuid-1"],
  "conflicts": [{ "id": "queue-uuid-2", "reason": "STOCK_NEGATIVE", "details": {} }],
  "server_time": "2026-05-01T09:12:45Z"
}
```

### 3.3 Delta Pull

```
GET /v1/sync/pull?since=2026-05-01T08:00:00Z&device_id=device-uuid
```

Returns all records changed after the `since` timestamp, excluding changes originating from this `device_id`. The app applies these to the local database.

### 3.4 Retry Strategy

| Attempt | Delay |
|---|---|
| 1st retry | 30 seconds |
| 2nd retry | 2 minutes |
| 3rd retry | 10 minutes |
| 4th retry | 30 minutes |
| 5th+ | Hourly |
| After 5th | No further retries — item marked `'failed'` |

---

## 4. Conflict Resolution

### 4.1 Conflict Resolution Rules

| Conflict Scenario | Resolution Rule | Notify Owner? |
|---|---|---|
| Same product stock modified on two offline devices | Most recent `server_time` wins. Both changes logged. Flag if stock goes negative. | Yes — if stock goes negative |
| Sale recorded offline for a product deleted online | Sale preserved. Product shown as `[Archived]`. | No |
| Same sale voided on two devices | First void wins. Second ignored (idempotent). | No |
| Price changed on one device while offline sales made at old price | Sales preserved at offline price. | Yes — price mismatch alert |
| Stock goes negative after all offline sales synced | All sales applied. Stock set to 0. | Yes — mandatory |
| Two managers create POs for same product simultaneously | Both POs created. Owner can cancel the duplicate. | No |

> 🚫 Conflicts resulting in negative stock or price mismatches are **never auto-resolved**. They always require explicit owner review.

### 4.2 Conflict Notification

When a conflict requires owner attention:
- A **Conflict** badge appears on the Alerts icon in the bottom nav
- Alert type is `SYNC_CONFLICT` with a description of the specific issue
- Owner taps the alert to see details and take action

---

## 5. Platform-Specific Implementation

### 5.1 Android — WorkManager

```kotlin
val syncRequest = PeriodicWorkRequestBuilder<SyncWorker>(
    repeatInterval = 15, TimeUnit.MINUTES
)
  .setConstraints(
    Constraints.Builder()
      .setRequiredNetworkType(NetworkType.CONNECTED)
      .build()
  )
  .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 30, TimeUnit.SECONDS)
  .build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
  "stocksense_sync",
  ExistingPeriodicWorkPolicy.KEEP,
  syncRequest
)
```

> ⚠️ Run sync on a background thread. Use Kotlin coroutines with `Dispatchers.IO` inside `doWork()`. Never run on the main (UI) thread.

> ℹ️ Use Room's `@Transaction` annotation on methods that write to multiple tables (e.g., creating a sale AND deducting stock). If either operation fails, both are rolled back.

### 5.2 PWA — Service Worker Caching Strategy

| Asset Type | Strategy | Rationale |
|---|---|---|
| App shell (HTML, CSS, JS) | Cache-First | Static assets — always serve instantly from cache |
| API reads (GET requests) | Network-First with Cache Fallback | Try fresh data; serve from cache if offline |
| API writes (POST, PUT, DELETE) | Queue (Background Sync API) | Never cache writes; queue them in `sync_queue` |
| Report PDFs | No caching | Generated on demand; too large to cache |

**Background Sync API:**
```javascript
// Register sync when an operation is queued
navigator.serviceWorker.ready.then(registration => {
  registration.sync.register('stocksense-sync');
});

// In the service worker
self.addEventListener('sync', event => {
  if (event.tag === 'stocksense-sync') {
    event.waitUntil(processSyncQueue());
  }
});
```

> ⚠️ Background Sync is supported in Chrome for Android but limited in Safari/iOS. For iOS (Phase 2), fall back to processing the queue on `visibilitychange` event.

---

## 6. Data Cached Offline

| Data Type | Strategy | Refresh Interval | Notes |
|---|---|---|---|
| Product catalogue | Persistent local copy | On every sync | Includes archived products (needed for historical sales display) |
| Current stock levels | Persistent local copy | On every sync | Updated immediately on every sale or movement |
| Pending sales | Full local storage | Always local | Never expire or purge pending items |
| Recent sales (last 30 days) | Local copy | On every sync | Needed for offline sales history |
| Active alerts | Local copy | On every sync | Allows attendants to see alerts without connectivity |
| Supplier directory | Persistent local copy | On every sync | Needed for offline PO creation |
| User profile & permissions | Persistent local copy | On login / role change | RBAC must work offline |
| Reports (last 7 days) | Cached response | Hourly when online | Preview only — export requires connectivity |
| Reconciliation in progress | Full local storage | Continuous | Must survive app close and network loss |

---

## 7. Sync Troubleshooting Guide

### 7.1 User-Facing Issues

#### "My sale doesn't appear on the owner's phone."

1. Ask the attendant: is there a sync badge showing pending operations?
2. If yes: the sale was recorded offline and is queued. Ask them to check their internet connection.
3. If internet is available but sync isn't completing: tap the sync badge to force a manual retry.
4. If the badge shows **Sync Failed**: check the device time (see device clock issue below).
5. If there is no sync badge and no sale: check the attendant's own sales history in the app.

#### "I see a Sync Failed notification."

1. Tap the notification to see which operations failed.
2. Tap **Retry**. If it succeeds: resolved.
3. If retry fails again: verify the internet connection by opening a browser.
4. If connection is good and sync still fails: note the error message and contact support with: device model, Android version, and the error text.

#### "I have a conflict alert on my dashboard."

1. Tap the conflict alert to see the details.
2. If it's a negative stock conflict: physically count the product and run a Reconciliation to correct the stock.
3. If it's a price mismatch: verify the correct selling price on the product and update if needed.
4. Mark the conflict alert as Resolved once action is taken.

#### "Sales recorded on two phones show double stock deductions."

1. The server applied both sales — both are valid records.
2. A negative stock conflict is raised on the owner dashboard.
3. Verify with both attendants whether both sales actually happened.
4. If one is an error: void the incorrect sale. Stock will restore.
5. If both are genuine: the shop oversold. Create a corrective stock adjustment.

---

### 7.2 Developer Debugging

**Inspect the sync queue on Android:**
```bash
adb shell run-as com.stocksense.app sqlite3 databases/stocksense.db
SELECT id, json_extract(operation, '$.type'), status, retry_count, last_error
FROM sync_queue WHERE status != 'synced' ORDER BY created_at ASC;
```

**Inspect IndexedDB on PWA (Chrome DevTools):**
1. Open DevTools (F12).
2. Go to **Application > Storage > IndexedDB > StockSenseDB**.
3. Open the `sync_queue` object store.

**Backend log patterns to search for:**
```
# Successful sync
level=INFO  event="sync.applied"  business_id=<uuid>  operations=5

# Conflict detected
level=WARN  event="sync.conflict"  business_id=<uuid>  reason="STOCK_NEGATIVE"

# Sync failure
level=ERROR  event="sync.error"  business_id=<uuid>  error="..."
```

**Sync endpoint error codes:**

| Code | HTTP | Meaning | Resolution |
|---|---|---|---|
| `SYNC_CONFLICT` | 409 | Conflict detected | Notify owner. Apply conflict rules. Return partial success. |
| `VALIDATION_ERROR` | 422 | Malformed operation payload | Log and skip the bad operation. Continue queue. |
| `INSUFFICIENT_STOCK` | 422 | Sale pushes stock below zero | Flag as conflict. Notify owner. |
| `STALE_OPERATION` | 409 | Record updated since last sync | Apply last-write-wins. Log both versions. |
| `UNAUTHORIZED` | 401 | JWT expired during long offline period | Client refreshes token and retries sync. |
| `SERVER_ERROR` | 500 | Unexpected server error | Retry with backoff. Escalate if persistent. |

---

## 8. Testing Offline Behaviour

| Test Case | Steps | Expected Result |
|---|---|---|
| Basic offline sale | Enable airplane mode → record a sale → confirm | Sale in attendant's list immediately. Sync badge shows 1 pending. |
| Sync on reconnect | After offline sale, re-enable WiFi/data | Sync badge disappears within 30 seconds. Sale appears on owner's list. |
| App close during queue | Enable airplane mode → record 3 sales → close app completely → re-enable data → reopen | All 3 sales sync. No data loss. |
| Concurrent offline negative stock | Device A + B: both offline, both sell last unit. Reconnect both. | Both sales applied. Stock goes to -1. Conflict alert on owner dashboard. |
| Offline reconciliation | Enable airplane mode → run full reconciliation → finalize | Completes offline. Syncs when reconnected. |
| Large queue sync | Record 50 sales offline → reconnect | All 50 sync within 30 seconds on 3G. No losses or duplicates. |

---

*StockSense · Offline Architecture & Sync Troubleshooting Guide v1.0 · May 2026 · Internal / Confidential*

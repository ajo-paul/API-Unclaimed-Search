--- Update : use Clou;d flare hosting . no GAS. it is fater and more reliable and close to enterprise model
 
 **very real, shippable MVP**. Below is a **step-by-step, end-to-end blueprint** you can follow *now* to launch an **API SaaS MVP** using:

* **Google Apps Script** → API gateway / proxy / management
* **Google Sheets** → control DB
* **Simple website** → marketing + docs
* **One backend (optional)** → real data service

---

# PHASE 0 — Define the MVP Scope (Day 0)

### MVP Promise (keep it narrow)

> “Search and fetch unclaimed records via API and UI, with rate limits and scheduled jobs.”

**MVP Includes**

* API key auth
* Search endpoint
* Quota limits
* Basic UI dashboard
* CSV/PDF export (async)
* Email support

**MVP Excludes**

* Complex fraud ML
* Realtime streaming
* Multi-region infra

---

# PHASE 1 — Google Sheets as Your DB (Day 1)

### 1.1 Create Sheets

Create **one spreadsheet**, with these tabs:

**`users`**

```
user_id | email | plan | status | created_at
```

**`api_keys`**

```
key_hash | user_id | plan | daily_quota | rpm | status
```

**`usage_daily`**

```
date | key_hash | count
```

**`plans`**

```
plan | daily_quota | rpm | features
```

**`logs`** (append-only)

```
timestamp | key_hash | endpoint | status | latency
```

🔐 Restrict access to **script owner only**.

---

# PHASE 2 — Apps Script API Gateway (Day 2–3)

### 2.1 Create Apps Script Project

* New Apps Script
* Link to the spreadsheet
* Deploy as **Web App**

  * Execute as: *Me*
  * Access: *Anyone with link*

---

### 2.2 Core Request Handler

```js
function doPost(e) {
  try {
    const req = JSON.parse(e.postData.contents);
    const apiKey = e.parameter.key || req.api_key;

    const ctx = authorize(apiKey);
    checkRateLimit(ctx);
    
    const result = route(req, ctx);
    logUsage(ctx, "success");

    return ok(result);
  } catch (err) {
    return fail(err);
  }
}
```

---

### 2.3 API Key Auth

* Hash incoming key (SHA-256)
* Lookup in `api_keys`
* Cache result (10 min)

Reject if:

* inactive
* quota exceeded
* wrong plan

---

### 2.4 Rate Limit Logic

* `CacheService` → per-minute counter
* Sheet → daily aggregate (batched)

Fail fast:

```
429 Too Many Requests
```

---

### 2.5 Proxy to Backend

```js
function proxyToBackend(payload) {
  return UrlFetchApp.fetch(BACKEND_URL, {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  });
}
```

---

# PHASE 3 — API Design (Day 4)

### 3.1 MVP Endpoints

```
POST /v1/search
POST /v1/batch/search
GET  /v1/usage
POST /v1/export
```

### 3.2 Example Request

```json
{
  "name": "John Doe",
  "state": "CA"
}
```

### 3.3 Example Response

```json
{
  "results": [],
  "confidence": 0.82,
  "source": "public_registry"
}
```

---

# PHASE 4 — Website (Day 5)

### 4.1 Pages (Simple)

* Home
* How It Works
* Pricing
* API Docs
* Support
* Login (Google OAuth or magic link)

Use:

* Next.js / Astro / Webflow / Carrd (fast)

---

### 4.2 Homepage Content (Copy)

**Hero**

> “Unclaimed data access via simple APIs.”

**Bullets**

* API-first
* Rate-limited
* Scheduled searches
* Clean exports

---

# PHASE 5 — API Documentation (Day 6)

### 5.1 Docs Structure

* Auth
* Endpoints
* Errors
* Rate limits
* Examples
* Changelog

### 5.2 Error Format (Important)

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Daily quota exceeded"
  }
}
```

### Error Codes

| Code                | Meaning          |
| ------------------- | ---------------- |
| UNAUTHORIZED        | Bad API key      |
| FORBIDDEN           | Plan restriction |
| RATE_LIMIT_EXCEEDED | Quota hit        |
| INVALID_REQUEST     | Bad params       |
| SERVER_ERROR        | Backend issue    |

---

# PHASE 6 — Pricing Model (Day 6)

### MVP Pricing

| Plan | Price | Daily Quota |
| ---- | ----- | ----------- |
| Free | $0    | 50          |
| Pro  | $29   | 5,000       |
| Team | $99   | 20,000      |

**Enforced in GAS**

Upgrade path:

* Stripe → webhook → update Sheets

---

# PHASE 7 — UI Dashboard (Day 7)

### Built with GAS HTML Service

Features:

* View API key
* Usage meter
* Saved searches
* Schedule jobs
* Download exports

GAS = backend for UI

---

# PHASE 8 — Scheduled Jobs (Week 2)

* Time triggers
* Run saved searches
* Notify via email/webhook
* Store results in Drive

---

# PHASE 9 — Support & Ops

### Support Design

* `support@yourdomain`
* Error IDs in responses
* Logs in `logs` sheet

### SLA (MVP)

* Best effort
* Email support only
* 24–48h response

---

# PHASE 10 — MVP Launch Checklist

✅ API keys work
✅ Quotas enforced
✅ Docs readable
✅ Pricing clear
✅ Logs visible
✅ Kill switch exists

---

# Final MVP Positioning

> **“A lightweight API SaaS powered by Google Apps Script for access control, automation, and speed — without heavy infrastructure.”**

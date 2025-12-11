# Monitoring, Testing & Error Handling Playbook

## 1. Reliability Objectives
- Sustain a **100% daily publishing schedule** by catching and recovering from transient failures automatically.
- Maintain **observability across the full Make workflow**: generation, upload, publishing, and notification layers.
- Keep the **monthly operation budget below 1,000 ops** with proactive alerting before limits are reached.
- Guarantee that every run (success or failure) is **logged, auditable, and testable**.

---

## 2. Error Handling & Routing Architecture

### 2.1 End-to-End Failure Routing Diagram
```mermaid
graph TD
    A[Daily Scheduler Trigger] --> B[Runway Generation Request]
    B -->|Success| C[Status Polling]
    C -->|Completed| D[Video Download]
    D --> E[YouTube Upload]
    E -->|Success| F[Log Run + Notify]

    B -->|HTTP 4xx/5xx| R1[Runway Error Router]
    C -->|Timeout/Failed| R1
    R1 --> R1a[Retry w/ Exponential Backoff\n(1m → 2m → 4m, max 3 attempts)]
    R1a -->|Retries Left| B
    R1a -->|Retries Exhausted| R1b[Fallback Branch]
    R1b --> R1c[Email Alert + Slack Failure]
    R1c --> F

    E -->|HTTP 4xx/5xx| Y1[YouTube Upload Router]
    Y1 --> Y1a[Retry w/ Backoff\n(30s → 60s → 90s)]
    Y1a -->|Success| F
    Y1a -->|Retries Exhausted| R1b

    A --> S1[Auth Check Module]
    S1 -->|Invalid/Expired Token| Auth[Auth Failure Router]
    Auth --> AuthFix[Token Refresh + Re-auth]
    AuthFix -->|Success| A
    AuthFix -->|Still Failing| R1b
```

### 2.2 Router Specifications (Make.com)
| Router | Trigger Condition | Actions | Notes |
| --- | --- | --- | --- |
| **Runway Error Router** | HTTP status ≥400, `status=failed`, or polling timeout | 1) Capture error payload.<br>2) Increment retry counter (Data Store / variable).<br>3) Apply exponential backoff delay.<br>4) Re-queue generation request if attempts <3.<br>5) If attempts =3, mark run as failed → fallback branch. | Use Make "Error Handler" + "Sleep" modules to implement backoff (60s, 120s, 240s). |
| **YouTube Upload Router** | Upload step returns error, quota issue, or validation failure | 1) Retry upload (max 3).<br>2) On success, log final URL + status.<br>3) On final failure, push to fallback branch (email + Slack + manual queue). | Limit to 3 attempts to preserve ops budget. |
| **Authentication Router** | OAuth token expired, 401/403, or missing credentials | 1) Run token refresh sub-flow.<br>2) If refresh fails, trigger Make webhook hitting admin inbox.<br>3) Block downstream steps until credentials fixed. | Covers Runway API key, YouTube OAuth, Google Drive, Slack webhook. |

---

## 3. Retry Logic & Fallback Strategy

### 3.1 Exponential Backoff Parameters
- **Runway Generation / Status Check:** retry intervals 60s → 120s → 240s (max 3 attempts per module).
- **YouTube Upload:** retry intervals 30s → 60s → 90s (max 3 attempts). Use Make "Sleep" module or built-in delay.
- **Auth Renewal:** attempt refresh once; if still unauthorized, skip retries and jump directly to fallback to avoid lockouts.

### 3.2 Fallback Branch (Daily Failure)
1. Mark execution as **failed** in Data Store with `video_status = "failed"` and attach diagnostic JSON.
2. Send **email alert** (Make Email / Gmail module) to ops list with subject `Daily Generation Failed - {date}` including:
   - Failing module name & error payload
   - Attempt counts
   - Next steps / manual action checklist
3. (Optional) Trigger Slack webhook (see §5) with summary + link to logs.
4. Stop downstream ops to prevent repeated failures that waste operations.

---

## 4. Logging & Observability

### 4.1 Make Data Store Schema
| Field | Type | Required | Description / Example |
| --- | --- | --- | --- |
| `execution_id` | string | ✅ | Make execution identifier (e.g., `exec_2024-01-01`). |
| `timestamp_utc` | ISO datetime | ✅ | `2024-01-01T09:05:00Z`. |
| `video_status` | enum | ✅ | `queued`, `processing`, `completed`, `failed`. |
| `runway_generation_id` | string | ✅ | `generation_12345`. |
| `youtube_url` | string/null | ✅ | Final Shorts link or `null` on failure. |
| `error_details` | object/null | ✅ | `{ "module": "YouTube", "code": 403, "message": "quotaExceeded" }`. |
| `operations_used` | integer | ✅ | Running total for the day (used for ops budgeting). |
| `alerts_triggered` | array | Optional | `['email', 'slack']` for auditing triage. |

Every scenario (success, partial failure, retry) writes a record. Use Make Data Store `Create/Update a record` immediately after each major stage to ensure breadcrumbs for post-mortems.

### 4.2 Log Retention & Access
- **Retention:** 90 days minimum to review trends.
- **Access:** Share read-only Data Store or sync to Google Sheets for analytics dashboards.
- **Backups:** Weekly export to Drive CSV to protect against accidental deletions.

---

## 5. Alerting & Notification Layer

| Signal | Trigger | Channel | Details |
| --- | --- | --- | --- |
| Success confirmation | Video uploaded + public URL available | Slack webhook (optional) | Post message with animal type, duration, and YouTube link. |
| Failure alert | Router sends run to fallback branch | Email (mandatory) + Slack (optional) | Include execution ID, error summary, next retry window. |
| Ops budget warning | Monthly operations ≥ 850 (85% of quota) | Email + Slack | Fetch usage via Make account API or manual counter (Data Store). |
| Ops cutoff | Monthly operations ≥ 950 | Stop scheduler via Make API, send urgent alert, require manual override. |

**Slack Webhook Payload Example**
```json
{
  "text": "Daily Rescue Short – SUCCESS",
  "attachments": [
    {
      "color": "#2eb886",
      "fields": [
        {"title": "Animal", "value": "Golden Retriever", "short": true},
        {"title": "YouTube", "value": "https://youtu.be/short123", "short": true},
        {"title": "Execution", "value": "exec_2024-01-01", "short": false}
      ]
    }
  ]
}
```

---

## 6. Ops Budget Monitoring

```mermaid
graph LR
    M1[Data Store Ops Counter] --> M2[Daily Aggregator]
    M2 --> M3[Monthly Ops Total]
    M3 -->|>= 700| Warn70[Slack Notice: "70% usage"]
    M3 -->|>= 850| Warn85[Email Alert: "Approaching limit"]
    M3 -->|>= 950| Cutoff[Auto-disable Scheduler + Pager]
```

**Implementation Steps**
1. After each execution, increment `operations_used` using Make Data Store `Increment value` or Google Sheets counter.
2. Nightly (or post-run), run a **Make aggregator scenario** that sums operations month-to-date.
3. Compare totals against thresholds:
   - 70% (700 ops) → heads-up Slack message.
   - 85% (850 ops) → email + in-app notification.
   - 95% (950 ops) → automatically pause scheduler via Make API (PATCH scenario to `disabled=true`) until manual review.
4. Include ops total inside fallback email so ops team can correlate spikes with failures or retries.

---

## 7. Testing Plan & Reliability Checklist

### 7.1 Dry-Run & Mock Uploads
1. Switch YouTube module to **test channel** with uploads marked **Unlisted**.
2. Execute full workflow manually via Make `Run once`.
3. Confirm Runway generation, download, upload, and logging all succeed.
4. Validate Slack/email success notifications with mock data.

### 7.2 Scheduler Reliability
- Run the daily scheduler for 7 consecutive days in staging.
- Capture trigger timestamps from Data Store to confirm ±1 minute drift.
- Ensure skipped/disabled runs generate alerts immediately.

### 7.3 Error Handler Simulation
| Scenario | How to Simulate | Expected Outcome |
| --- | --- | --- |
| Runway API failure | Use invalid API key or force `status=failed` via webhook replay | Retry 3x with backoff → fallback email/Slack, log failure. |
| YouTube quota error | Limit uploads or trigger `quotaExceeded` response | Retry 3x, remain under ops budget, fallback alert fired. |
| Authentication issue | Revoke OAuth token | Auth router refreshes token; if still failing, scheduler paused + alert. |

### 7.4 Logging & Alert Validation
- Inspect Data Store entries to ensure all required fields populated for both success and failure cases.
- Cross-check timestamps with Make execution logs.
- Verify YouTube URL stored only when upload succeeds.

### 7.5 Ops Budget Test
- Use sandbox counter to simulate 900 monthly ops.
- Ensure alert workflow fires at 850+ threshold and scheduler pause triggers at 950.
- Confirm manual override procedure (re-enable scheduler) documented.

### 7.6 Final Checklist
- [ ] Dry-run recorded and reviewed.
- [ ] Scheduler triggered automatically for at least 3 consecutive days without drift.
- [ ] All routers verified with simulated errors.
- [ ] Logging schema validated; sample record exported.
- [ ] Email + Slack notifications received for success & failure.
- [ ] Ops budget alerts verified at 70%, 85%, and 95% thresholds.
- [ ] Documentation stored alongside credentials for future audits.

---

## 8. Ongoing Maintenance
- Review alert noise monthly; adjust thresholds/backoff as needed.
- Rotate API keys / OAuth tokens quarterly to avoid surprise auth failures.
- Audit Data Store for anomalies and prune records older than 6 months (after export).
- Update this playbook whenever new destinations (e.g., Instagram Reels) are added so routers stay in sync.

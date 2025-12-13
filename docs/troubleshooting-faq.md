# Troubleshooting & FAQ

Common solutions for issues with the Automated YouTube Shorts Workflow.

## Table of Contents
1. [My workflow won't start](#1-my-workflow-wont-start)
2. [Runway video didn't generate](#2-runway-video-didnt-generate)
3. [YouTube won't accept the video](#3-youtube-wont-accept-the-video)
4. [I got an error code XXX](#4-i-got-an-error-code-xxx)
5. [My video didn't publish](#5-my-video-didnt-publish)
6. [How do I know if it worked?](#6-how-do-i-know-if-it-worked)
7. [Can I change the video theme?](#7-can-i-change-the-video-theme)
8. [It's using too many operations](#8-its-using-too-many-operations)
9. [I want to schedule at a different time](#9-i-want-to-schedule-at-a-different-time)
10. [Something broke, how do I reset?](#10-something-broke-how-do-i-reset)

---

### 1. "My workflow won't start"

**Symptoms:** The scheduled time passes, but nothing happens in Make.com.

**Steps to fix:**
1. **Check the Scheduler:** Ensure the "Daily Scheduler Trigger" module is set to "On" (green toggle).
2. **Verify Schedule:** Open the module settings and check the time zone. It defaults to UTC.
3. **Check Connections:** Go to the "Connections" tab in Make.com and verify all connections (Runway, YouTube, Google Drive) are active.
4. **Notifications:** Check the bell icon in Make.com for system-wide alerts.

**Docs:** [Make Setup Guide > Scheduler](./make-setup-guide.md#1-daily-scheduler-trigger)

---

### 2. "Runway video didn't generate"

**Symptoms:** The workflow stops at the Runway module or returns an error.

**Steps to fix:**
1. **Check API Key:** Ensure your Runway API key in `credentials.md` is valid and has not expired.
2. **Check Credits:** Log in to your RunwayML dashboard and verify you have available credits.
3. **Review Prompt:** If the prompt triggers safety filters, Runway will block generation. Check the "Data Prep" module logs for the generated prompt.

**Docs:** [Runway Config > Error Handling](./runway-config.md#error-handling-and-retries)

---

### 3. "YouTube won't accept the video"

**Symptoms:** The YouTube Upload module fails with an invalid file or format error.

**Steps to fix:**
1. **Check Duration:** Shorts must be under 60 seconds. Our config targets 45 seconds.
2. **Check Aspect Ratio:** Must be 9:16 (vertical).
3. **File Size:** Ensure the file is under 50MB (standard for this workflow).

**Docs:** [YouTube Upload > Shorts Format](./youtube-upload.md#shorts-optimization-checklist)

---

### 4. "I got an error code XXX"

**Common Make.com Errors:**

| Error Code | Meaning | Fix |
|------------|---------|-----|
| `401 Unauthorized` | Invalid API Key | Update connection in Make.com |
| `429 Too Many Requests` | Rate Limit | Wait or increase polling interval |
| `500 Internal Server Error` | Service Down | Retry later (temporary issue) |
| `TIMEOUT` | Polling took too long | Increase max attempts in Job Status Monitor |

**Docs:** [Workflow Blueprint > Error Handler](./make-workflow-blueprint.md#7-error-handler-with-retry-logic)

---

### 5. "My video didn't publish"

**Symptoms:** The workflow completes, but the video isn't visible on the channel.

**Steps to fix:**
1. **Check Channel Permissions:** Ensure the authenticated Google account has permission to upload to the target channel.
2. **Refresh Token:** OAuth tokens expire. Re-authorize the YouTube connection in Make.com.
3. **Privacy Setting:** Check if the upload status is set to "Private" or "Unlisted" in the YouTube module.

**Docs:** [YouTube Upload > Troubleshooting](./youtube-upload.md#troubleshooting)

---

### 6. "How do I know if it worked?"

**Where to look:**
1. **Make.com History:** Click the "History" tab in your scenario to see execution logs.
2. **YouTube Studio:** Check "Content > Shorts" for new uploads.
3. **Data Store:** If configured, check the Google Sheet or Data Store for the success log entry.

**Docs:** [Monitoring & Testing](./monitoring-testing.md)

---

### 7. "Can I change the video theme?"

**Steps to fix:**
1. Open the **Data Prep Module** (Module 2) in the Make.com scenario.
2. Locate the `animalCategories` array or the prompt template.
3. Edit the text to change animals, settings, or story styles.

**Docs:** [Workflow Blueprint > Data Prep](./make-workflow-blueprint.md#2-data-prep-module-randomized-story-generator)

---

### 8. "It's using too many operations"

**Steps to optimize:**
1. **Increase Polling Interval:** Change the Job Status Monitor to check every 120 seconds instead of 60.
2. **Reduce Logging:** Disable non-critical logs to Google Sheets.
3. **Check Frequency:** Ensure you aren't running the workflow more than once per day.

**Docs:** [Workflow Blueprint > Cost Optimization](./make-workflow-blueprint.md#operations-budget-analysis)

---

### 9. "I want to schedule at a different time"

**Steps to fix:**
1. Click the clock icon on the **Daily Scheduler Trigger** module.
2. Change the time. Note that it uses UTC.
3. Use a time zone converter to match your local audience (e.g., 09:00 UTC = 04:00 EST).

**Docs:** [Make Setup Guide](./make-setup-guide.md#step-2-configure-scheduler)

---

### 10. "Something broke, how do I reset?"

**Steps to fix:**
1. **Disable:** Turn the scenario switch to "Off".
2. **Clear Queue:** If executions are stuck, delete them from the "Incomplete Executions" tab.
3. **Re-enable:** Turn the switch back to "On".
4. **Run Once:** Click "Run Once" to verify everything is working again.

**Docs:** [Quick Reference > Recovery](./QUICK_REFERENCE.md)

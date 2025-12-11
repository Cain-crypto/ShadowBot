# Quick Reference Card

## Essential Links

| Resource | URL |
|----------|-----|
| **Make.com Login** | https://www.make.com/login |
| **Runway API Settings** | https://app.runway.ml/settings/api |
| **Runway Usage Dashboard** | https://app.runway.ml/usage |
| **Google Cloud Console** | https://console.cloud.google.com |
| **YouTube Studio** | https://studio.youtube.com |
| **Google Sheets** | https://sheets.google.com |
| **Google Drive** | https://drive.google.com |

---

## Scenario Import (3 Steps)

1. **Copy JSON**
   ```
   Open: docs/make-scenario-blueprint.json
   Copy: Entire file content (Ctrl+A, Ctrl+C)
   ```

2. **Import to Make**
   ```
   Login → Scenarios → Create new
   Click ⋯ menu → Import Blueprint
   Paste → Save
   ```

3. **Configure**
   ```
   Replace 8 variables (see below)
   Connect 5 services (see below)
   Test → Enable
   ```

---

## 8 Variables to Replace

| Find | Replace With |
|------|--------------|
| `{{RUNWAY_API_KEY}}` | Your Runway API key |
| `{{GOOGLE_DRIVE_ID}}` | `My Drive` |
| `{{YOUTUBE_CONNECTION}}` | Your YouTube connection name |
| `{{GOOGLE_SHEETS_CONNECTION}}` | Your Sheets connection name |
| `{{SUCCESS_LOG_SPREADSHEET_ID}}` | Your spreadsheet ID |
| `{{ERROR_LOG_SPREADSHEET_ID}}` | Your spreadsheet ID (same or different) |
| `{{EMAIL_CONNECTION}}` | Your email connection name |
| `{{ALERT_EMAIL_ADDRESS}}` | Your alert email address |

---

## 5 Connections to Create

### 1. Runway HTTP (Modules 3, 6)
```
Type: HTTP Custom
Authorization Header: Bearer {{RUNWAY_API_KEY}}
```

### 2. Google Drive (Module 10)
```
Type: Google Drive OAuth
Sign in with Google → Grant permissions
```

### 3. YouTube (Module 11)
```
Type: YouTube OAuth
Sign in with Google → Grant upload permissions
Scopes: youtube.upload, youtube
```

### 4. Google Sheets (Modules 12, 13)
```
Type: Google Sheets OAuth
Sign in with Google → Grant spreadsheet access
```

### 5. Email (Module 14)
```
Type: Gmail or SMTP
Option A: Gmail OAuth (recommended)
Option B: SMTP with host/port/credentials
```

---

## Google Sheets Setup

### Success Log Sheet Headers
```
A: Date
B: Execution ID
C: Animal Type
D: Generation ID
E: Video ID
F: YouTube URL
G: Status
H: Time
I: Operations
```

### Error Log Sheet Headers
```
A: Date
B: Execution ID
C: Module
D: Error Type
E: Error Message
F: Retry Count
G: Resolution
H: Time
```

**Spreadsheet ID Location:**
```
https://docs.google.com/spreadsheets/d/1ABC123xyz456DEF789/edit
                                      ^^^^^^^^^^^^^^^^^^^
                                      This is the ID
```

---

## Testing Checklist

### Before First Run
- [ ] Schedule trigger DISABLED
- [ ] All connections authenticated (green)
- [ ] All variables replaced
- [ ] Google Sheets created with headers

### First Manual Run
```
1. Click "Run once" button
2. Watch modules execute
3. Expected time: 3-5 minutes
4. Check outputs:
   - Google Drive: Video file saved
   - YouTube: Short uploaded
   - Sheets: Success logged
```

### If Successful
- [ ] Enable schedule trigger
- [ ] Set time: 10:00 UTC (or preferred)
- [ ] Wait for first scheduled run
- [ ] Monitor for 3-7 days

---

## Troubleshooting

### Import Failed
```
Problem: JSON won't import
Solution: 
  1. Validate at jsonlint.com
  2. Clear browser cache
  3. Try manual recreation (see setup guide)
```

### Connection Error
```
Problem: "Invalid credentials" or "Unauthorized"
Solution:
  1. Reauthorize connection
  2. Check API key format
  3. Verify OAuth scopes
  4. Check token expiration
```

### Generation Timeout
```
Problem: Video never completes
Solution:
  1. Increase repeater max (10 → 15)
  2. Increase initial wait (120s → 180s)
  3. Check Runway status page
  4. Verify API credits remain
```

### Upload Failed
```
Problem: YouTube upload error
Solution:
  1. Check daily quota (max 6/day)
  2. Verify OAuth scopes
  3. Test manual upload
  4. Check video format (9:16, <60s)
```

### No Logging
```
Problem: Google Sheets not updating
Solution:
  1. Verify spreadsheet ID
  2. Check sheet names (case-sensitive)
  3. Confirm permissions
  4. Test connection separately
```

---

## Operations Budget

### Per Run (Success Path)
```
Schedule:      1 op
Data Prep:     1 op
Runway API:    1 op
Status Checks: 5-10 ops (varies)
Download:      1 op
Google Drive:  1 op
YouTube:       1 op
Logging:       1 op
----------------------------
Total:         15-20 ops
```

### Monthly (30 days)
```
30 days × 20 ops = 600 ops
Free tier limit: 1,000 ops
Buffer remaining: 400 ops
```

### Optimization Tips
```
Reduce ops by:
  - Increase polling interval (60s → 90s)
  - Decrease repeater max (10 → 8)
  - Conditional logging (errors only)
  - Use Data Store instead of Sheets
```

---

## Module Quick Reference

| ID | Module Name | Purpose | Ops |
|----|-------------|---------|-----|
| 1 | Schedule | Daily trigger at 10:00 UTC | 1 |
| 2 | Data Prep | Generate random story | 1 |
| 3 | Runway Request | Start video generation | 1 |
| 4 | Sleep | Wait 2 minutes | 0 |
| 5 | Repeater | Loop up to 10 times | 0 |
| 6 | Status Check | Poll generation status | 1 × loops |
| 7 | Router | Branch on status | 0 |
| 8 | Sleep (Processing) | Wait 60s if processing | 0 |
| 9 | Download | Fetch completed video | 1 |
| 10 | Google Drive | Save to storage | 1 |
| 11 | YouTube | Upload as Short | 1 |
| 12 | Success Log | Record to Sheets | 1 |
| 13 | Error Log | Record error | 1 |
| 14 | Email Alert | Send failure email | 1 |

---

## API Endpoints

### Runway Gen-3
```
Base URL: https://api.runwayml.com/v1

Generate Video:
  POST /video_generations
  Headers: Authorization: Bearer {API_KEY}
  Body: {model, prompt, duration, ratio, resolution, style, seed}

Check Status:
  GET /video_generations/{generation_id}
  Headers: Authorization: Bearer {API_KEY}
```

### YouTube Data API v3
```
Base URL: https://www.googleapis.com/youtube/v3

Upload Video:
  POST /videos
  Scopes: youtube.upload, youtube
  Body: {snippet, status, recordingDetails}
```

---

## Video Specifications

### Runway Gen-3 Request
```json
{
  "model": "gen3a_turbo",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": <random>
}
```

### YouTube Shorts Format
```
Duration:     ≤60 seconds (we use 45s)
Aspect Ratio: 9:16 (vertical)
Resolution:   1080x1920 pixels
Frame Rate:   24 fps
Format:       MP4 (H.264)
Category:     15 (Pets & Animals)
Privacy:      Public
```

---

## Story Template Variables

### Animal Types
```
golden retriever, tabby cat, baby elephant,
German shepherd, stray kitten, rescue horse,
injured bird, dolphin
```

### Scenarios
```
abandoned in park, trapped in drain, injured on highway,
stuck in well, caught in flood, lost in fire
```

### Rescue Methods
```
rescued by firefighters, saved by volunteers,
helped by rangers, assisted by vets,
protected by organization
```

### Outcomes
```
thriving in forever home, reunited with family,
recovering at sanctuary, adopted by family,
safe and healthy
```

---

## Schedule Configuration

### Default
```
Time: 10:00
Timezone: UTC
Frequency: Daily
```

### Time Zone Conversions (UTC 10:00)
```
New York:  05:00 EST / 06:00 EDT
London:    10:00 GMT / 11:00 BST
Paris:     11:00 CET / 12:00 CEST
Tokyo:     19:00 JST
Sydney:    20:00 AEST / 21:00 AEDT
```

### Adjustment
```
Module 1 → Parameters → Schedule
  Change "time": "10:00" to desired time
  Change "timezone": "UTC" to local timezone
```

---

## Monitoring

### Daily Check
```
✓ Scenario ran on time?
✓ Video uploaded to YouTube?
✓ Success logged in Sheets?
✓ Operations count reasonable?
```

### Weekly Review
```
✓ Success rate >95%?
✓ Average ops <25 per run?
✓ Monthly total <1000 ops?
✓ No recurring errors?
```

### Access Logs
```
Make.com: Scenarios → [Your Scenario] → History
YouTube:  studio.youtube.com → Content
Sheets:   Success Log and Error Log tabs
```

---

## Support Contacts

| Service | Support URL |
|---------|-------------|
| **Make.com** | https://help.make.com |
| **Runway** | support@runwayml.com |
| **YouTube API** | https://developers.google.com/youtube/v3/support |
| **Google Cloud** | https://cloud.google.com/support |

---

## File Locations

```
docs/
├── make-scenario-blueprint.json    ← Import this
├── make-setup-guide.md             ← Follow this
├── SETUP_CHECKLIST.md              ← Check off items
├── QUICK_REFERENCE.md              ← You are here
├── credentials.md                  ← API key setup
├── runway-config.md                ← Video settings
└── youtube-upload.md               ← Upload config
```

---

## Commands Reference

### Validate JSON
```bash
python3 -m json.tool docs/make-scenario-blueprint.json
```

### Check Spreadsheet ID
```bash
# From URL: .../spreadsheets/d/[ID]/edit
# Copy the [ID] portion
```

### Test API Key
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.runwayml.com/v1/video_generations
```

### Check YouTube Quota
```bash
# Visit: console.cloud.google.com/apis/api/youtube.googleapis.com/quotas
# View: Queries per day (default: 10,000)
```

---

## Quick Wins

### Reduce Operations
1. Module 8: Change delay from 60 to 90 seconds
2. Module 5: Change repeats from 10 to 8
3. Module 12: Add filter to log only on errors

### Improve Videos
1. Module 2: Add more animal types and scenarios
2. Module 3: Try "gen3a" for higher quality
3. Module 3: Adjust motion_strength (0.5-1.0)

### Increase Frequency
1. Duplicate scenario for 2×/day
2. Set different times (morning/evening)
3. Use different story themes for each

---

**Need more help? See [make-setup-guide.md](make-setup-guide.md) for complete documentation.**

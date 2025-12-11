# Make.com Scenario Setup Guide

## Overview

This guide provides step-by-step instructions for importing and configuring the **Animal Rescue Shorts automation scenario** in Make.com. The scenario automates the complete workflow from video generation with Runway Gen-3 to YouTube Shorts publishing.

**Blueprint File:** `make-scenario-blueprint.json`

**Estimated Setup Time:** 30-45 minutes

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Import Blueprint into Make.com](#import-blueprint-into-makecom)
3. [Configure Connections](#configure-connections)
4. [Set Up Google Sheets Logging](#set-up-google-sheets-logging)
5. [Replace Variable Placeholders](#replace-variable-placeholders)
6. [Test the Scenario](#test-the-scenario)
7. [Enable Scheduling](#enable-scheduling)
8. [Monitor and Optimize](#monitor-and-optimize)
9. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before importing the scenario, ensure you have the following:

### 1. Make.com Account
- **Sign up:** [make.com](https://www.make.com)
- **Plan:** Free tier (1,000 operations/month) is sufficient
- **Status:** Account verified and active

### 2. Runway Gen-3 API Key
- **Obtain from:** [Runway API Settings](https://app.runway.ml/settings/api)
- **Format:** `runway_api_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
- **Free tier:** ~$5-10 monthly credits included

### 3. Google Account
- **For:** YouTube, Google Drive, Google Sheets
- **Requirements:** 
  - YouTube channel created and active
  - Google Drive with storage space
  - Google Sheets access

### 4. YouTube Data API v3 Access
- **Setup:** Follow [credentials.md](./credentials.md) for OAuth setup
- **Required scopes:** `youtube.upload`, `youtube`
- **Status:** OAuth consent screen configured

### 5. Email Account for Alerts
- **Options:** Gmail, SendGrid, SMTP, etc.
- **Purpose:** Receive failure notifications

---

## Import Blueprint into Make.com

### Step 1: Download the Blueprint

1. Navigate to the repository folder: `docs/`
2. Open `make-scenario-blueprint.json`
3. Copy the entire JSON content (Ctrl+A, Ctrl+C)

### Step 2: Access Make.com Scenarios

1. Log into [Make.com](https://www.make.com)
2. Click **Scenarios** in the left sidebar
3. Click **Create a new scenario** button (top right)

### Step 3: Import the Blueprint

**Option A: Import via Blueprint (Recommended)**

1. In the new scenario, click the **⋯** (three dots) menu at the bottom
2. Select **Import Blueprint**
3. Paste the JSON content into the text area
4. Click **Save**
5. The scenario will be imported with all modules and connections

**Option B: Manual Recreation (If Import Fails)**

If the blueprint import doesn't work (due to Make.com version differences), you can manually recreate the scenario following this module sequence:

```
1. Schedule Trigger (Daily 10:00 UTC)
   ↓
2. Data Prep - Aggregator (Story generator)
   ↓
3. HTTP Request (Runway API - Generate)
   ↓
4. Sleep (120 seconds)
   ↓
5. Repeater (10 iterations)
   ↓
6. HTTP Request (Runway API - Status check)
   ↓
7. Router (3 routes)
   ├─ Route 1: Status = Processing → Sleep 60s (loop back)
   ├─ Route 2: Status = Completed → Download → Drive → YouTube → Log
   └─ Route 3: Status = Failed → Log Error → Send Email
```

### Step 4: Review Imported Scenario

1. The scenario name will be: **"Animal Rescue Shorts - Runway Gen-3 to YouTube Automation"**
2. Review the visual flow in the designer
3. Check that all modules are connected properly
4. You should see 14 total modules

---

## Configure Connections

Each external service requires a connection to be configured in Make.com.

### Connection 1: Runway Gen-3 API (HTTP Custom)

**Module:** HTTP modules (IDs 3, 6, 9)

**Setup:**

1. Click on module **#3 (Runway Gen-3 - Generate Video)**
2. In the **Headers** section, find `Authorization: Bearer {{RUNWAY_API_KEY}}`
3. Replace `{{RUNWAY_API_KEY}}` with your actual API key:
   ```
   Authorization: Bearer runway_api_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```
4. Repeat for module **#6 (Check Generation Status)**
5. Module **#9 (Download Video File)** doesn't need the API key

**Alternative (Using Variables):**

1. Go to scenario settings (gear icon)
2. Click **Variables**
3. Add variable:
   - Name: `RUNWAY_API_KEY`
   - Value: Your Runway API key
   - Type: Secret (hide value)
4. Reference in headers as: `Bearer {{variables.RUNWAY_API_KEY}}`

### Connection 2: Google Drive

**Module:** #10 (Save to Google Drive)

**Setup:**

1. Click on module **#10 (Save to Google Drive)**
2. Click the **Connection** dropdown
3. Select **Add** to create new connection
4. Choose **Google Drive**
5. Click **Sign in with Google**
6. Select your Google account
7. Grant permissions:
   - View and manage Google Drive files
   - Create folders
8. Name the connection: `Google Drive - Animal Rescue`
9. Click **Save**

**Drive Configuration:**

- **Drive:** Select `My Drive` (or a Team Drive if you have one)
- **Folder Path:** `/Animal Rescue Shorts/{{2.date}}`
  - This creates a new folder for each day
  - Example: `/Animal Rescue Shorts/2024-01-15/`

### Connection 3: YouTube

**Module:** #11 (Upload to YouTube Shorts)

**Setup:**

1. Click on module **#11 (Upload to YouTube Shorts)**
2. Click the **Connection** dropdown
3. Select **Add** to create new connection
4. Choose **YouTube**
5. Click **Sign in with Google**
6. Select the Google account linked to your YouTube channel
7. Grant permissions:
   - Manage your YouTube videos
   - Upload videos
8. Name the connection: `YouTube - Animals Rescue Channel`
9. Click **Save**

**Verify:**
- After saving, the module should display your channel name
- If you have multiple channels, the default channel will be used

### Connection 4: Google Sheets

**Modules:** #12 (Log Success), #13 (Log Error)

**Setup:**

1. Click on module **#12 (Log Success to Google Sheets)**
2. Click the **Connection** dropdown
3. Select **Add** to create new connection
4. Choose **Google Sheets**
5. Click **Sign in with Google**
6. Select your Google account (same as Drive/YouTube)
7. Grant permissions:
   - View and manage spreadsheets in Google Drive
8. Name the connection: `Google Sheets - Logging`
9. Click **Save**
10. Repeat connection selection for module **#13**

### Connection 5: Email (for Alerts)

**Module:** #14 (Send Failure Alert Email)

**Setup:**

**Option A: Gmail (Recommended for personal use)**

1. Click on module **#14 (Send Failure Alert Email)**
2. Click the **Connection** dropdown
3. Select **Add** → **Gmail**
4. Sign in with your Google account
5. Grant email sending permissions
6. Name the connection: `Gmail - Alerts`

**Option B: SMTP (Custom email)**

1. Click on module **#14 (Send Failure Alert Email)**
2. Click the **Connection** dropdown
3. Select **Add** → **Email** → **Send an Email (SMTP)**
4. Enter SMTP settings:
   - **Host:** smtp.gmail.com (or your provider)
   - **Port:** 587 (TLS) or 465 (SSL)
   - **Username:** your-email@example.com
   - **Password:** Your email password or app password
   - **Secure:** Yes
5. Name the connection: `SMTP - Alerts`

**Update Email Address:**

In module #14, replace `{{ALERT_EMAIL_ADDRESS}}` with your actual email:
```
To: your-email@example.com
```

---

## Set Up Google Sheets Logging

The scenario logs successful runs and errors to Google Sheets for tracking and analytics.

### Step 1: Create Spreadsheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **Blank** to create a new spreadsheet
3. Name it: `Animal Rescue Shorts - Logs`

### Step 2: Create Success Log Sheet

1. Rename `Sheet1` to `Success Log`
2. Add headers in Row 1:

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| Date | Execution ID | Animal Type | Generation ID | Video ID | YouTube URL | Status | Time | Operations |

**Example Data (Row 2):**
```
2024-01-15 | exec123 | golden retriever | gen_abc123 | abc123xyz | https://youtube.com/watch?v=abc123xyz | success | 10:05:32 | 18
```

### Step 3: Create Error Log Sheet

1. Click **+** to add a new sheet
2. Name it: `Error Log`
3. Add headers in Row 1:

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| Date | Execution ID | Module | Error Type | Error Message | Retry Count | Resolution | Time |

**Example Data (Row 2):**
```
2024-01-15 | exec124 | Runway Generation | RATE_LIMIT | API quota exceeded | 10/10 | Max retries exhausted | 10:15:47
```

### Step 4: Get Spreadsheet ID

1. Open your spreadsheet in Google Sheets
2. Look at the URL:
   ```
   https://docs.google.com/spreadsheets/d/1ABC123xyz456DEF789/edit
   ```
3. Copy the ID between `/d/` and `/edit`:
   ```
   1ABC123xyz456DEF789
   ```
4. Save this ID - you'll need it in the next step

### Step 5: Configure Spreadsheet IDs in Scenario

**In Module #12 (Success Log):**

1. Click module #12
2. Find field **Spreadsheet ID**
3. Replace `{{SUCCESS_LOG_SPREADSHEET_ID}}` with your actual ID:
   ```
   1ABC123xyz456DEF789
   ```
4. Verify **Sheet Name** is set to: `Success Log`

**In Module #13 (Error Log):**

1. Click module #13
2. Find field **Spreadsheet ID**
3. Replace `{{ERROR_LOG_SPREADSHEET_ID}}` with your actual ID (same as above):
   ```
   1ABC123xyz456DEF789
   ```
4. Verify **Sheet Name** is set to: `Error Log`

**Alternative: Use Separate Spreadsheets**

If you prefer separate spreadsheets for success and errors:

1. Create two spreadsheets: `Success Logs` and `Error Logs`
2. Use different spreadsheet IDs in modules #12 and #13

---

## Replace Variable Placeholders

The blueprint includes placeholder variables that need to be replaced with your actual values.

### Variables to Replace

Open the scenario and use Find & Replace (Ctrl+F in browser) to update these placeholders:

#### 1. RUNWAY_API_KEY

**Find:** `{{RUNWAY_API_KEY}}`  
**Replace with:** Your Runway API key (e.g., `runway_api_xxxxx`)  
**Locations:** Modules #3, #6 (Authorization headers)

#### 2. GOOGLE_DRIVE_ID

**Find:** `{{GOOGLE_DRIVE_ID}}`  
**Replace with:** `My Drive` (or your Team Drive ID)  
**Location:** Module #10

#### 3. YOUTUBE_CONNECTION

**Find:** `{{YOUTUBE_CONNECTION}}`  
**Replace with:** Name of your YouTube connection (e.g., `YouTube - Animals Rescue Channel`)  
**Location:** Module #11

#### 4. GOOGLE_SHEETS_CONNECTION

**Find:** `{{GOOGLE_SHEETS_CONNECTION}}`  
**Replace with:** Name of your Google Sheets connection (e.g., `Google Sheets - Logging`)  
**Locations:** Modules #12, #13

#### 5. SUCCESS_LOG_SPREADSHEET_ID

**Find:** `{{SUCCESS_LOG_SPREADSHEET_ID}}`  
**Replace with:** Your spreadsheet ID (e.g., `1ABC123xyz456DEF789`)  
**Location:** Module #12

#### 6. ERROR_LOG_SPREADSHEET_ID

**Find:** `{{ERROR_LOG_SPREADSHEET_ID}}`  
**Replace with:** Your spreadsheet ID (same as above or different)  
**Location:** Module #13

#### 7. EMAIL_CONNECTION

**Find:** `{{EMAIL_CONNECTION}}`  
**Replace with:** Name of your email connection (e.g., `Gmail - Alerts`)  
**Location:** Module #14

#### 8. ALERT_EMAIL_ADDRESS

**Find:** `{{ALERT_EMAIL_ADDRESS}}`  
**Replace with:** Your email address (e.g., `your-email@example.com`)  
**Location:** Module #14

### Verification Checklist

After replacing all placeholders, verify:

- [ ] All `{{PLACEHOLDER}}` variables are replaced
- [ ] No errors shown in any module
- [ ] All connections are green (authenticated)
- [ ] Spreadsheet IDs are correct
- [ ] Email address is valid

---

## Test the Scenario

Before enabling the daily schedule, test the scenario manually to ensure everything works.

### Step 1: Disable Scheduler

1. Click on module **#1 (Schedule)**
2. Toggle the switch to **OFF** (grayed out)
3. This prevents the scenario from running automatically during testing

### Step 2: Run Once Manually

1. Click the **Run once** button at the bottom of the scenario
2. The scenario will execute immediately
3. Watch the modules light up as they process

### Step 3: Monitor Execution

**Expected Flow:**

```
Module 1: Schedule ✓ (Triggered manually)
  ↓
Module 2: Data Prep ✓ (Story generated)
  ↓
Module 3: Runway API Request ✓ (Video generation started)
  ↓
Module 4: Sleep ✓ (Wait 2 minutes)
  ↓
Module 5: Repeater ✓ (Start polling)
  ↓
Module 6: Status Check ✓ (Check 1/10)
  ↓
Module 7: Router (Routes to appropriate path)
  ├─ If Processing: Sleep 60s → Loop back to #6
  ├─ If Completed: Continue to download
  └─ If Failed: Log error and alert
```

**Successful Completion:**

```
Module 9: Download Video ✓
  ↓
Module 10: Save to Drive ✓
  ↓
Module 11: Upload to YouTube ✓
  ↓
Module 12: Log Success ✓
```

### Step 4: Verify Outputs

**Check Google Drive:**

1. Go to [Google Drive](https://drive.google.com)
2. Navigate to `/Animal Rescue Shorts/[Today's Date]/`
3. Verify MP4 file exists
4. Download and play to check quality

**Check YouTube:**

1. Go to [YouTube Studio](https://studio.youtube.com)
2. Click **Content** in left sidebar
3. Verify new Short is uploaded
4. Check:
   - Title includes "#Shorts"
   - Description has hashtags
   - Privacy is set to Public
   - Video is under 60 seconds
   - Aspect ratio is vertical (9:16)

**Check Google Sheets:**

1. Open your logging spreadsheet
2. Go to **Success Log** sheet
3. Verify new row with:
   - Today's date
   - Execution ID
   - Animal type
   - YouTube URL
   - Operations count

### Step 5: Test Error Handling

To test error scenarios:

**Test 1: Invalid API Key (temporary)**

1. In module #3, temporarily change API key to `invalid_key`
2. Run the scenario
3. Expected: Error logged, email alert sent
4. Restore correct API key after test

**Test 2: Network Timeout Simulation**

1. Disconnect internet briefly after module #3 runs
2. Expected: Status checks fail, error handling triggered
3. Restore connection and re-run

---

## Enable Scheduling

Once testing is successful, enable the daily automation.

### Step 1: Configure Schedule

1. Click on module **#1 (Schedule)**
2. Verify settings:
   - **Mode:** Daily
   - **Time:** 10:00
   - **Timezone:** UTC
3. Adjust time if needed (e.g., 09:00 for earlier execution)

**Time Zone Conversion:**

- **UTC 10:00** = 
  - 05:00 EST (New York winter)
  - 06:00 EDT (New York summer)
  - 10:00 GMT (London)
  - 18:00 JST (Tokyo)
  - 20:00 AEST (Sydney)

Choose a time that works for your target audience's peak hours.

### Step 2: Enable the Scenario

1. Click the toggle switch at the bottom left to **ON**
2. The scenario will now run automatically at the scheduled time
3. You'll see "Scenario is ON" confirmation

### Step 3: Verify First Scheduled Run

1. Wait for the scheduled time (or set it to run in 5 minutes for testing)
2. Monitor the first automated execution
3. Check all outputs (Drive, YouTube, Sheets)

---

## Monitor and Optimize

### Daily Monitoring

**What to Check:**

1. **Google Sheets Success Log:**
   - New row added daily?
   - YouTube URL valid?
   - Operations count consistent?

2. **YouTube Channel:**
   - Shorts appearing in feed?
   - Views/engagement metrics?
   - No copyright/policy issues?

3. **Make.com Dashboard:**
   - Go to **Scenarios** → Your scenario
   - Click **History**
   - Review execution logs
   - Check operations usage

### Weekly Review

**Metrics to Track:**

| Metric | Target | Check |
|--------|--------|-------|
| Success Rate | >95% | Successful runs / Total runs |
| Avg Operations | <25 ops | Total ops / Successful runs |
| YouTube Views | Increasing | Week-over-week growth |
| Error Rate | <5% | Failed runs / Total runs |
| Monthly Ops | <1000 | Total ops for the month |

### Optimization Tips

**Reduce Operations:**

1. **Increase polling interval:**
   - Change module #8 delay from 60s to 90s
   - Saves ~3-5 operations per run

2. **Reduce repeater max:**
   - Change module #5 repeats from 10 to 8
   - Risk: Longer videos might timeout

3. **Conditional logging:**
   - Only log errors, not every success
   - Add filter before module #12: `{{11.id}} is not empty`

4. **Use Data Store instead of Sheets:**
   - Faster and uses fewer operations
   - Replace modules #12, #13 with Data Store

**Improve Video Quality:**

1. **Enhance prompts:**
   - Add more detail to story templates
   - Use cinematic keywords
   - Reference specific camera angles

2. **Adjust Runway settings:**
   - Try `gen3a` instead of `gen3a_turbo` for better quality (slower)
   - Increase `motion_strength` to 0.9 for more dynamic videos

3. **A/B test titles:**
   - Track which title variants get more views
   - Optimize title templates based on performance

**Increase Content Variety:**

1. **Expand animal types:**
   - Add module #2 with more categories
   - Include wildlife, marine animals, birds

2. **Seasonal themes:**
   - Update story templates monthly
   - Add holiday-themed rescues

3. **Randomize video styles:**
   - Add style variations: cinematic, documentary, anime
   - Use random selection in module #3

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Scenario Won't Import

**Symptoms:**
- Error message when pasting blueprint
- JSON parse error

**Solutions:**

1. **Validate JSON:**
   - Copy blueprint content
   - Paste into [JSONLint](https://jsonlint.com)
   - Fix any syntax errors

2. **Manual recreation:**
   - Follow the module sequence in Section 2, Step 3, Option B
   - Configure each module individually

3. **Check Make.com version:**
   - Ensure you're on the latest Make.com version
   - Clear browser cache and retry

#### Issue 2: Runway API Returns Error

**Symptoms:**
- Module #3 fails with error
- "Invalid API key" or "Quota exceeded"

**Solutions:**

1. **Verify API key:**
   - Go to [Runway API Settings](https://app.runway.ml/settings/api)
   - Regenerate key if needed
   - Update in module #3, #6

2. **Check credits:**
   - Visit [Runway Usage Dashboard](https://app.runway.ml/usage)
   - Ensure you have remaining credits
   - Upgrade plan if exhausted

3. **Rate limiting:**
   - Free tier: 10 requests/minute
   - Add delay between retries
   - Reduce frequency if hitting limits

#### Issue 3: Video Generation Timeout

**Symptoms:**
- Status check repeater exhausts all attempts
- Video never completes

**Solutions:**

1. **Increase repeater max:**
   - Change module #5 from 10 to 15 repeats
   - Allows 15 minutes total wait time

2. **Increase initial wait:**
   - Change module #4 delay from 120s to 180s
   - Reduces unnecessary status checks

3. **Check Runway status:**
   - Visit [Runway Status Page](https://status.runwayml.com)
   - Verify no outages or degraded service

#### Issue 4: YouTube Upload Fails

**Symptoms:**
- Module #11 returns error
- "Quota exceeded" or "Invalid request"

**Solutions:**

1. **Check YouTube quota:**
   - Go to [Google Cloud Console](https://console.cloud.google.com)
   - Navigate to **YouTube Data API v3** → **Quotas**
   - Default: 10,000 units/day
   - Upload costs: 1,600 units
   - Max 6 uploads/day on free tier

2. **Verify OAuth scopes:**
   - Ensure connection has `youtube.upload` scope
   - Recreate connection if needed

3. **Check video format:**
   - Verify file is MP4
   - Check aspect ratio is 9:16
   - Ensure duration <60 seconds

4. **Test upload manually:**
   - Download video from Google Drive
   - Upload manually to YouTube Studio
   - Check for content policy violations

#### Issue 5: Google Sheets Not Logging

**Symptoms:**
- No rows added to spreadsheet
- Module #12 or #13 fails

**Solutions:**

1. **Verify spreadsheet ID:**
   - Double-check ID in modules #12, #13
   - Ensure no extra spaces or characters

2. **Check sheet names:**
   - Must exactly match: `Success Log` and `Error Log`
   - Case-sensitive

3. **Permissions:**
   - Spreadsheet must be owned by connected Google account
   - Or shared with edit permissions

4. **Connection:**
   - Verify Google Sheets connection is authenticated
   - Reauthorize if needed

#### Issue 6: Email Alerts Not Sending

**Symptoms:**
- Module #14 fails
- No email received on errors

**Solutions:**

1. **Check email connection:**
   - Verify Gmail or SMTP connection is authenticated
   - Test by sending a manual email

2. **Gmail security:**
   - Enable "Less secure app access" (for SMTP)
   - Or use Gmail API with OAuth (recommended)

3. **Email address:**
   - Verify `{{ALERT_EMAIL_ADDRESS}}` is replaced
   - Check for typos

4. **Spam folder:**
   - Check if emails are going to spam
   - Add to safe senders list

#### Issue 7: High Operations Usage

**Symptoms:**
- Approaching 1,000 ops/month limit
- Operations count higher than expected

**Solutions:**

1. **Review execution history:**
   - Go to scenario **History**
   - Identify which modules use most operations
   - Example: Too many status checks

2. **Optimize polling:**
   - Increase delay in module #8 to 90s or 120s
   - Reduce repeater max in module #5

3. **Conditional execution:**
   - Add filters to skip unnecessary modules
   - Example: Only log errors, not successes

4. **Upgrade plan:**
   - Make.com Pro: 10,000 ops/month ($9/month)
   - Allows ~400 video generations/month

---

## Advanced Configuration

### Custom Scheduler Options

**Multiple Runs Per Day:**

1. Duplicate the scenario
2. Change schedule times:
   - Scenario 1: 09:00 UTC
   - Scenario 2: 21:00 UTC (12 hours later)
3. Different story themes for each run

**Skip Weekends:**

1. Add filter after module #1:
   ```
   {{formatDate(now; "E")}} < 6
   ```
   (Monday=1, Sunday=7; this filters out Sat/Sun)

**Holiday Bypass:**

1. Add module after #1: **Tools → Set Variable**
2. Configure:
   ```javascript
   const holidays = ["2024-12-25", "2024-01-01", "2024-07-04"];
   const today = {{formatDate(now; "YYYY-MM-DD")}};
   return !holidays.includes(today);
   ```
3. Add filter: `{{variable}} = true`

### Webhook Trigger (Manual Override)

To allow manual triggering via webhook:

1. Replace module #1 (Schedule) with **Webhooks → Custom Webhook**
2. Copy the webhook URL
3. Trigger manually:
   ```bash
   curl -X POST "https://hook.make.com/your_webhook_url"
   ```
4. Add back the scheduler as a second trigger

### Analytics Dashboard

**Create a dashboard in Google Sheets:**

1. Add new sheet: `Dashboard`
2. Add formulas:

**Total Videos Generated:**
```
=COUNTA('Success Log'!A:A)-1
```

**Success Rate:**
```
=COUNTA('Success Log'!A:A) / (COUNTA('Success Log'!A:A) + COUNTA('Error Log'!A:A))
```

**Avg Operations Per Run:**
```
=AVERAGE('Success Log'!I:I)
```

**Monthly Operations:**
```
=SUMIF('Success Log'!A:A, ">=2024-01-01", 'Success Log'!I:I)
```

3. Add charts for visualization

### Integration with Other Platforms

**Post to Social Media:**

Add modules after #11 (YouTube Upload):

1. **Twitter/X:** Share YouTube link with hashtags
2. **Facebook:** Post to page with video embed
3. **Instagram:** (Requires manual repost due to API limits)
4. **TikTok:** Use third-party services or manual upload

**Analytics Tracking:**

1. **Google Analytics:** Track YouTube referrals
2. **Airtable:** Alternative to Google Sheets with better UI
3. **Slack:** Post success notifications to team channel

---

## Support and Resources

### Make.com Resources

- **Help Center:** [help.make.com](https://help.make.com)
- **Community Forum:** [community.make.com](https://community.make.com)
- **Video Tutorials:** [YouTube - Make Channel](https://www.youtube.com/c/Makecom)
- **Template Library:** [make.com/templates](https://www.make.com/en/templates)

### API Documentation

- **Runway Gen-3:** [docs.runwayml.com](https://docs.runwayml.com)
- **YouTube Data API:** [developers.google.com/youtube](https://developers.google.com/youtube/v3)
- **Google Drive API:** [developers.google.com/drive](https://developers.google.com/drive)

### Project Documentation

- **Credentials Setup:** [credentials.md](./credentials.md)
- **Runway Configuration:** [runway-config.md](./runway-config.md)
- **YouTube Upload:** [youtube-upload.md](./youtube-upload.md)
- **Workflow Blueprint:** [make-workflow-blueprint.md](./make-workflow-blueprint.md)

---

## Conclusion

You now have a fully automated system that:

✅ Generates unique animal rescue videos daily using Runway Gen-3  
✅ Stores videos in organized Google Drive folders  
✅ Uploads to YouTube as Shorts with optimized metadata  
✅ Logs all activities for tracking and analytics  
✅ Sends alerts on failures for quick resolution  
✅ Operates within free tier limits (1,000 ops/month)  

**Next Steps:**

1. Monitor the first week of automated runs
2. Optimize based on performance metrics
3. Experiment with different story templates
4. Engage with your YouTube audience
5. Scale up content production as channel grows

**Happy automating! 🎬🐾**

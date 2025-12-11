# Make.com Scenario Setup Checklist

Use this checklist to ensure you have everything needed before importing the Make.com scenario blueprint.

---

## Pre-Import Checklist

### ✅ Accounts Created

- [ ] Make.com account (free tier)
- [ ] Runway account with API access
- [ ] Google account for YouTube, Drive, Sheets
- [ ] YouTube channel created and active
- [ ] Email account for alerts (Gmail recommended)

### ✅ API Keys & Credentials Obtained

- [ ] **Runway Gen-3 API Key**
  - Location: [Runway API Settings](https://app.runway.ml/settings/api)
  - Format: `runway_api_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
  - Saved securely (password manager or env file)

- [ ] **Google Cloud Project Created**
  - Project name: ___________________________
  - Project ID: ___________________________

- [ ] **YouTube Data API v3 Enabled**
  - In Google Cloud Console
  - Verified at: [APIs & Services](https://console.cloud.google.com/apis)

- [ ] **OAuth Consent Screen Configured**
  - App name: ___________________________
  - Scopes added: `youtube.upload`, `youtube`
  - Test users added (your email)

- [ ] **OAuth Client ID & Secret**
  - Client ID: ___________________________
  - Client Secret: (saved securely)
  - Redirect URIs configured

### ✅ Google Sheets Prepared

- [ ] **Logging Spreadsheet Created**
  - Spreadsheet name: ___________________________
  - Spreadsheet ID: ___________________________
  - (ID from URL: `docs.google.com/spreadsheets/d/[ID]/edit`)

- [ ] **Success Log Sheet**
  - Sheet name exactly: `Success Log`
  - Headers in Row 1:
    - A: Date
    - B: Execution ID
    - C: Animal Type
    - D: Generation ID
    - E: Video ID
    - F: YouTube URL
    - G: Status
    - H: Time
    - I: Operations

- [ ] **Error Log Sheet**
  - Sheet name exactly: `Error Log`
  - Headers in Row 1:
    - A: Date
    - B: Execution ID
    - C: Module
    - D: Error Type
    - E: Error Message
    - F: Retry Count
    - G: Resolution
    - H: Time

### ✅ Files Downloaded

- [ ] `make-scenario-blueprint.json` downloaded
- [ ] `make-setup-guide.md` reviewed
- [ ] All documentation files accessible

---

## Import Checklist

### ✅ Make.com Scenario Import

- [ ] Logged into Make.com
- [ ] Clicked "Create a new scenario"
- [ ] Selected "Import Blueprint"
- [ ] Pasted JSON content
- [ ] Scenario imported successfully
- [ ] All 7 main modules visible in designer

### ✅ Connections Configured

- [ ] **Runway HTTP Connection**
  - API key added to Authorization header
  - Modules #3, #6 configured

- [ ] **Google Drive Connection**
  - OAuth authentication completed
  - Module #10 connected
  - Drive ID set (typically "My Drive")

- [ ] **YouTube Connection**
  - OAuth authentication completed
  - Module #11 connected
  - Channel name displayed correctly

- [ ] **Google Sheets Connection**
  - OAuth authentication completed
  - Modules #12, #13 connected
  - Spreadsheet ID configured

- [ ] **Email Connection**
  - Gmail or SMTP configured
  - Module #14 connected
  - Alert email address set

### ✅ Variables Replaced

- [ ] `{{RUNWAY_API_KEY}}` → Your Runway API key
- [ ] `{{GOOGLE_DRIVE_ID}}` → `My Drive` or Team Drive ID
- [ ] `{{YOUTUBE_CONNECTION}}` → Your YouTube connection name
- [ ] `{{GOOGLE_SHEETS_CONNECTION}}` → Your Sheets connection name
- [ ] `{{SUCCESS_LOG_SPREADSHEET_ID}}` → Your spreadsheet ID
- [ ] `{{ERROR_LOG_SPREADSHEET_ID}}` → Your spreadsheet ID (same or different)
- [ ] `{{EMAIL_CONNECTION}}` → Your email connection name
- [ ] `{{ALERT_EMAIL_ADDRESS}}` → Your email address

### ✅ Module Configuration Verified

- [ ] **Module #1: Schedule**
  - Time: 10:00 (or your preferred time)
  - Timezone: UTC (or your timezone)
  - Currently DISABLED for testing

- [ ] **Module #2: Data Prep**
  - Story templates loaded
  - Random selection working
  - Video prompt generated

- [ ] **Module #3: Runway API Request**
  - URL: `https://api.runwayml.com/v1/video_generations`
  - Method: POST
  - Headers: Authorization with Bearer token
  - Body: JSON with all parameters

- [ ] **Module #4: Initial Wait**
  - Delay: 120 seconds (2 minutes)

- [ ] **Module #5: Repeater**
  - Repeats: 10 times
  - Max wait: 10 minutes total

- [ ] **Module #6: Status Check**
  - URL: Runway API status endpoint
  - Generation ID from Module #3

- [ ] **Module #7: Router**
  - 3 routes configured:
    - Processing → Sleep 60s
    - Completed → Continue to download
    - Failed → Error handling

- [ ] **Module #10: Google Drive**
  - Folder path: `/Animal Rescue Shorts/{{2.date}}`
  - File name: `rescue_{{2.date}}_{{generation_id}}.mp4`

- [ ] **Module #11: YouTube Upload**
  - Privacy: Public
  - Category: 15 (Pets & Animals)
  - Made for Kids: False
  - Notify Subscribers: True

- [ ] **Module #12: Success Log**
  - Spreadsheet ID correct
  - Sheet name: `Success Log`
  - All 9 columns mapped

- [ ] **Module #13: Error Log**
  - Spreadsheet ID correct
  - Sheet name: `Error Log`
  - All 8 columns mapped

- [ ] **Module #14: Email Alert**
  - To: Your alert email
  - Subject: Clear and descriptive
  - Body: Includes all error details

---

## Testing Checklist

### ✅ Manual Test Run

- [ ] Schedule trigger DISABLED
- [ ] Clicked "Run once" button
- [ ] Watched execution progress
- [ ] All modules completed without errors

### ✅ Runway Generation Test

- [ ] Video generation request succeeded
- [ ] Status polling started
- [ ] Generation completed within timeout
- [ ] Video URL received

### ✅ File Storage Test

- [ ] Video downloaded successfully
- [ ] Saved to Google Drive
- [ ] Folder structure correct: `/Animal Rescue Shorts/[date]/`
- [ ] File name format: `rescue_[date]_[id].mp4`
- [ ] File playable (downloaded and tested)

### ✅ YouTube Upload Test

- [ ] Video uploaded to YouTube
- [ ] Classified as Short automatically
- [ ] Title includes "#Shorts"
- [ ] Description has hashtags
- [ ] Privacy set to Public
- [ ] Video visible in YouTube Studio

### ✅ Logging Test

- [ ] New row added to Success Log sheet
- [ ] All columns populated correctly
- [ ] YouTube URL clickable and works
- [ ] Operations count recorded

### ✅ Error Handling Test (Optional)

- [ ] Temporarily broke API key
- [ ] Error logged to Error Log sheet
- [ ] Email alert received
- [ ] Error details clear and actionable
- [ ] Restored API key and re-tested

---

## Production Checklist

### ✅ Pre-Launch

- [ ] All tests passed successfully
- [ ] Credentials saved securely
- [ ] Backup of scenario exported
- [ ] Documentation reviewed
- [ ] Schedule time confirmed
- [ ] First week monitoring plan ready

### ✅ Launch

- [ ] Schedule trigger ENABLED
- [ ] First scheduled run confirmed
- [ ] Monitoring dashboard set up
- [ ] Alert email notifications working
- [ ] YouTube channel ready for content

### ✅ First Week Monitoring

**Day 1:**
- [ ] Scenario ran on schedule
- [ ] Video generated successfully
- [ ] YouTube upload completed
- [ ] Logging recorded correctly

**Day 2-3:**
- [ ] Consistent execution times
- [ ] No errors in error log
- [ ] Operations count stable
- [ ] YouTube views tracking

**Day 4-7:**
- [ ] Weekly success rate >95%
- [ ] Average operations <25 per run
- [ ] No quota issues
- [ ] Content quality satisfactory

### ✅ Optimization

- [ ] Reviewed operations usage
- [ ] Identified optimization opportunities
- [ ] Adjusted polling intervals if needed
- [ ] Fine-tuned story templates
- [ ] Analyzed YouTube performance
- [ ] Planned content improvements

---

## Troubleshooting Reference

### Quick Fixes

**Scenario won't import:**
→ Validate JSON, clear cache, try manual recreation

**API key errors:**
→ Verify format, check expiration, regenerate if needed

**Connection failures:**
→ Reauthorize OAuth, check scopes, verify permissions

**Upload quota exceeded:**
→ Max 6 uploads/day, wait 24 hours, request quota increase

**High operations usage:**
→ Increase polling delay, reduce repeater max, optimize logging

---

## Support Resources

- **Setup Guide:** [make-setup-guide.md](./make-setup-guide.md)
- **Troubleshooting:** [make-setup-guide.md#troubleshooting](./make-setup-guide.md#troubleshooting)
- **Credentials:** [credentials.md](./credentials.md)
- **Configuration:** [runway-config.md](./runway-config.md)

---

## Success Criteria

Your setup is complete when:

✅ Scenario runs automatically on schedule  
✅ Videos generate and upload daily  
✅ No errors in execution history  
✅ Logs populate correctly in Google Sheets  
✅ Operations usage <1000/month  
✅ YouTube Shorts appear in feed  
✅ Email alerts work (tested)  

**Congratulations! Your automation is live! 🎉🐾**

---

_Last Updated: 2024-01-15_

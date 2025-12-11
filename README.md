# Animal Rescue Shorts - Automated YouTube Content Generator

> 🎬 Fully automated daily YouTube Shorts using Runway Gen-3 AI video generation and Make.com workflow automation

## Overview

This project provides a complete, ready-to-deploy automation system that generates heartwarming animal rescue story videos daily and publishes them as YouTube Shorts. The system uses:

- **Runway Gen-3 AI** for cinematic video generation
- **Make.com** for workflow orchestration
- **YouTube Shorts** for content distribution
- **Google Drive** for file storage
- **Google Sheets** for analytics and logging

**Key Features:**
- ✅ Fully automated (zero manual intervention after setup)
- ✅ Optimized for free tier usage (~25 ops/run, <1000/month)
- ✅ Comprehensive error handling and retry logic
- ✅ Daily unique content with randomized stories
- ✅ YouTube Shorts format (9:16, 45 seconds, vertical)
- ✅ Complete logging and email alerts
- ✅ Importable Make.com scenario blueprint

---

## Quick Start

### 1. Import the Make.com Scenario

The fastest way to get started is by importing the pre-built scenario:

```bash
# Navigate to the docs folder
cd docs/

# Open make-scenario-blueprint.json
# Copy the entire JSON content
```

Then follow the **[Make Setup Guide](docs/make-setup-guide.md)** for step-by-step import instructions.

### 2. Configure Credentials

Set up API keys and OAuth connections:

- **Runway Gen-3 API:** [Get API Key](https://app.runway.ml/settings/api)
- **YouTube Data API:** Follow **[Credentials Guide](docs/credentials.md)**
- **Google Drive & Sheets:** OAuth setup included in Make.com
- **Email Alerts:** Configure Gmail or SMTP

### 3. Test and Deploy

1. Import scenario into Make.com
2. Replace credential placeholders
3. Run manual test
4. Enable daily schedule (10:00 AM UTC)
5. Monitor first week of runs

**Estimated Setup Time:** 30-45 minutes

---

## Documentation

### Core Setup Guides

| Document | Purpose | When to Use |
|----------|---------|-------------|
| **[Make Setup Guide](docs/make-setup-guide.md)** | Step-by-step scenario import and configuration | **START HERE** - Primary setup guide |
| **[Make Scenario Blueprint](docs/make-scenario-blueprint.json)** | Importable JSON for Make.com | Import into Make.com to create scenario |
| **[Credentials Setup](docs/credentials.md)** | API keys, OAuth, tokens | Before importing scenario |
| **[Runway Configuration](docs/runway-config.md)** | Video generation settings | Customize video quality/style |

### Advanced Documentation

| Document | Purpose |
|----------|---------|
| **[Workflow Blueprint](docs/make-workflow-blueprint.md)** | Detailed architecture and module breakdown |
| **[YouTube Upload Config](docs/youtube-upload.md)** | Shorts optimization and metadata templates |
| **[Monitoring & Testing](docs/monitoring-testing.md)** | Analytics, testing procedures, optimization |

---

## Architecture

### High-Level Workflow

```mermaid
graph LR
    A[Daily Scheduler<br/>10:00 AM UTC] --> B[Story Generator<br/>Random Animal Rescue]
    B --> C[Runway Gen-3 API<br/>Generate 45s Video]
    C --> D[Status Polling<br/>Check Every 60s]
    D --> E{Video Ready?}
    E -->|Yes| F[Download MP4]
    E -->|No| D
    E -->|Failed| J[Log Error & Alert]
    F --> G[Save to Google Drive]
    G --> H[Upload to YouTube Shorts]
    H --> I[Log Success]
    J --> K[Send Email Alert]
```

### Module Breakdown

The Make.com scenario includes 14 modules across 3 execution paths:

**Success Path (Modules 1-12):**
1. Schedule Trigger - Daily at 10:00 UTC
2. Data Prep - Randomized story generation
3. Runway API Request - Video generation
4. Initial Wait - 2 minutes
5. Repeater - Up to 10 status checks
6. Status Check - Poll Runway API
7. Router - Branch based on status
8. Download - Fetch completed video
9. Google Drive - Save with organized structure
10. YouTube Upload - Publish as Short
11. Success Logging - Track in Google Sheets

**Processing Path (Module 8):**
- Sleep 60s and continue polling

**Error Path (Modules 13-14):**
- Log error details to Google Sheets
- Send email alert to administrator

### Operations Budget

| Scenario Step | Operations | Notes |
|---------------|-----------|-------|
| Scheduler | 1 | Trigger event |
| Data Prep | 1 | Story generation |
| Runway Request | 1 | Initial API call |
| Status Checks | 5-10 | Polling until complete |
| Download | 1 | Fetch video file |
| Google Drive | 1 | Upload to storage |
| YouTube Upload | 1 | Publish to channel |
| Logging | 1 | Success/error tracking |
| **Total per run** | **15-20** | Well within free tier |
| **Monthly (30 days)** | **450-600** | <1000 ops free limit |

---

## Video Specifications

### YouTube Shorts Format

| Parameter | Value | Purpose |
|-----------|-------|---------|
| **Duration** | 45 seconds | Optimal for Shorts engagement |
| **Aspect Ratio** | 9:16 (vertical) | Native Shorts format |
| **Resolution** | 1080x1920 pixels | YouTube Shorts requirement |
| **Quality** | 720p | Balanced quality/speed |
| **Frame Rate** | 24 fps | Cinematic standard |
| **Format** | MP4 (H.264) | Universal compatibility |

### Runway Gen-3 Configuration

```json
{
  "model": "gen3a_turbo",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": "<randomized>"
}
```

**Cost per video:** ~$0.01-0.05 (free tier includes ~$5-10/month credits)

---

## Content Strategy

### Story Templates

The system generates randomized animal rescue stories using combinatorial templates:

**Variables:**
- **Animal Types:** Dog, cat, elephant, horse, dolphin, bird, etc.
- **Scenarios:** Abandoned, trapped, injured, lost, endangered
- **Rescue Methods:** Firefighters, volunteers, wildlife rangers, veterinarians
- **Outcomes:** Adopted, reunited, recovered, sanctuary, forever home

**Example Prompt:**
```
45-second heartwarming cinematic video: A golden retriever puppy found 
abandoned in a park during a rainstorm, rescued by animal shelter volunteers, 
and adopted by a loving family. Professional cinematography, emotional 
storytelling, warm color grading. Vertical 9:16 format for YouTube Shorts. 
Emphasize the puppy's journey from fear to joy.
```

### Title & Description Templates

**Title Format:**
```
Animals Rescue Humans - [Story Variant] #Shorts
```

**Description Format:**
```
🐾 [Story Summary]

[Extended Narrative]

🔔 Subscribe for daily animal rescue stories
👍 Like if this touched your heart
💬 Share your own rescue story below

#AnimalRescue #WildlifeHeroes #Shorts #[AnimalType] #Heartwarming
```

---

## Cost Analysis

### Free Tier Viability

**Make.com Free Tier:**
- 1,000 operations/month
- **Usage:** ~600 ops/month (30 daily runs × 20 ops)
- **Remaining:** 400 ops buffer for errors/retries
- **Cost:** $0/month

**Runway Gen-3 Free Tier:**
- ~$5-10 monthly credits
- **Usage:** ~$1.50/month (30 videos × $0.05)
- **Remaining:** Enough for 100-200 more videos
- **Cost:** $0/month

**Google Services (Free):**
- YouTube Data API: 10,000 units/day (6 uploads/day max)
- Google Drive: 15 GB free storage
- Google Sheets: Unlimited spreadsheets

**Total Monthly Cost:** $0 (using free tiers only)

### Scaling Options

**Upgrade to Make.com Pro ($9/month):**
- 10,000 operations/month
- Enables ~400 video generations/month
- 13 videos per day

**Upgrade to Runway Paid Plan ($12/month):**
- ~$30-50 in monthly credits
- 600-1000 video generations/month
- Higher resolution options

---

## Error Handling

### Comprehensive Error Recovery

The scenario includes multiple layers of error handling:

**1. Retry Logic**
- Status polling: Up to 10 attempts (10 minutes max)
- Exponential backoff: 60s intervals

**2. Error Logging**
- All failures logged to Google Sheets
- Includes: Date, module, error type, retry count, resolution

**3. Email Alerts**
- Sent on any generation or upload failure
- Includes full context for debugging

**4. Graceful Degradation**
- Failed runs don't block future runs
- Each day is independent execution

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| API Rate Limit | Too many requests | Wait and retry, increase delays |
| Video Timeout | Long generation time | Increase repeater max attempts |
| Upload Failure | YouTube quota exceeded | Max 6 uploads/day on free tier |
| Auth Error | Expired token | Reauthorize connection in Make |

---

## Monitoring & Analytics

### Google Sheets Logging

**Success Log Sheet:**
- Date, Execution ID, Animal Type
- Generation ID, Video ID, YouTube URL
- Status, Time, Operations Used

**Error Log Sheet:**
- Date, Execution ID, Module
- Error Type, Error Message, Retry Count
- Resolution, Time

### Key Metrics

**Track in Google Sheets Dashboard:**
- Success rate (target: >95%)
- Average operations per run (target: <25)
- Average generation time (target: <5 minutes)
- Monthly operations usage (target: <1000)
- YouTube views and engagement

---

## Customization

### Adjust Schedule

Change daily run time in Module #1:

```javascript
{
  "schedule": {
    "mode": "daily",
    "time": "09:00",  // Change to your preferred time
    "timezone": "UTC"  // Or your local timezone
  }
}
```

### Multiple Runs Per Day

Duplicate the scenario and set different times:
- Morning: 09:00 UTC
- Evening: 21:00 UTC

### Custom Story Templates

Edit Module #2 (Data Prep) arrays:

```javascript
"animal_types": [
  "golden retriever",
  "tabby cat",
  "baby elephant",
  // Add your custom animals here
],
"scenarios": [
  "abandoned in a park",
  "trapped in a storm drain",
  // Add your custom scenarios here
]
```

### Video Quality Settings

Adjust Runway parameters in Module #3:

```json
{
  "model": "gen3a",  // Higher quality (slower)
  "resolution": "1080p",  // Better resolution
  "motion_strength": 0.9  // More dynamic motion
}
```

---

## Troubleshooting

### Setup Issues

**Scenario won't import:**
- Validate JSON at [JSONLint](https://jsonlint.com)
- Try manual recreation (see setup guide)
- Clear browser cache and retry

**Connection failures:**
- Verify all API keys are correct
- Reauthorize OAuth connections
- Check credential expiration dates

### Runtime Issues

**Video generation fails:**
- Check Runway credits balance
- Verify API key is valid
- Review error log in Google Sheets

**YouTube upload fails:**
- Check daily quota (max 6 uploads/day)
- Verify OAuth scopes include `youtube.upload`
- Ensure video meets Shorts requirements

**High operations usage:**
- Increase polling intervals (60s → 90s)
- Reduce repeater max attempts
- Enable conditional logging

---

## Roadmap

### Planned Features

- [ ] Multi-language support (Spanish, French, German)
- [ ] Thumbnail auto-generation with AI
- [ ] Voice-over narration integration
- [ ] TikTok and Instagram Reels cross-posting
- [ ] Advanced analytics dashboard
- [ ] A/B testing for title/description variants
- [ ] Content moderation and quality scoring
- [ ] Seasonal and holiday-themed stories

### Community Contributions

We welcome contributions! Areas for improvement:

- Additional story templates
- Alternative video generation providers
- Enhanced error recovery strategies
- Performance optimizations
- Integration with other platforms

---

## Support

### Documentation

- **Setup:** [Make Setup Guide](docs/make-setup-guide.md)
- **API Keys:** [Credentials Guide](docs/credentials.md)
- **Configuration:** [Runway Config](docs/runway-config.md)
- **Troubleshooting:** [Setup Guide - Troubleshooting Section](docs/make-setup-guide.md#troubleshooting)

### External Resources

- **Make.com Help:** [help.make.com](https://help.make.com)
- **Runway Docs:** [docs.runwayml.com](https://docs.runwayml.com)
- **YouTube API:** [developers.google.com/youtube](https://developers.google.com/youtube/v3)

### Issues and Questions

For questions or issues:

1. Check the [Troubleshooting Guide](docs/make-setup-guide.md#troubleshooting)
2. Review execution logs in Make.com History
3. Verify all credentials and connections
4. Check Google Sheets error log for details

---

## License

This project is provided as-is for educational and personal use. Please ensure compliance with:

- Runway Gen-3 Terms of Service
- YouTube Terms of Service
- Google API Terms of Service
- Make.com Terms of Service

AI-generated content should be disclosed per platform policies.

---

## Acknowledgments

- **Runway Gen-3** for powerful AI video generation capabilities
- **Make.com** for flexible workflow automation platform
- **YouTube Shorts** for accessible short-form video distribution
- **Animal rescue organizations** worldwide for inspiring this project

---

**Ready to automate? Start with the [Make Setup Guide](docs/make-setup-guide.md)!** 🚀🐾

# Runway Gen-3 Configuration for Animal Rescue Shorts

## Overview

This document provides complete configuration details for generating 45-second animal rescue videos using Runway Gen-3 within the Make.com automation platform. Includes API setup, prompt templates, video parameters, and optimization strategies.

---

## Table of Contents

1. [API Setup & Authentication](#api-setup--authentication)
2. [Video Parameters](#video-parameters)
3. [Prompt Templates](#prompt-templates)
4. [Seed Randomization Strategy](#seed-randomization-strategy)
5. [Free Tier Optimization](#free-tier-optimization)
6. [File Download & Processing](#file-download--processing)
7. [Make HTTP Integration](#make-http-integration)
8. [Example Workflows](#example-workflows)

---

## API Setup & Authentication

### Runway Gen-3 API Endpoint

**Base URL:**
```
https://api.runwayml.com/v1
```

**Video Generation Endpoint:**
```
POST https://api.runwayml.com/v1/video_generations
```

**Status Check Endpoint:**
```
GET https://api.runwayml.com/v1/video_generations/{generation_id}
```

### Authentication

All API requests require a Bearer token in the Authorization header:

```http
Authorization: Bearer YOUR_RUNWAY_API_KEY
```

**Obtaining Your API Key:**

1. Navigate to [Runway Settings](https://app.runway.ml/settings/api)
2. Click **"Create API Key"** or **"Generate New Key"**
3. Name your key (e.g., "Animals Rescue Shorts")
4. Copy the key immediately (shown only once)
5. Store safely as `RUNWAY_API_KEY` in Make.com Connections

**API Key Management:**
- Free tier provides ~$5-10 in monthly credits
- One 45-second generation costs approximately $0.01-0.05
- Monitor usage at [Runway Usage Dashboard](https://app.runway.ml/usage)
- API rate limits: Typically 10 requests per minute for free tier

### Make.com Connection Setup

**Step 1: Create Custom Connection**
1. Log into [Make.com](https://www.make.com)
2. Go to **Connections** (left sidebar)
3. Click **Add** → **Custom**

**Step 2: Configure Connection**
- **Connection name:** "Runway Gen-3 API"
- **Connection type:** HTTP
- **Base URL:** `https://api.runwayml.com/v1`

**Step 3: Add Authorization Header**
- **Header name:** `Authorization`
- **Header value:** `Bearer {{RUNWAY_API_KEY}}`
- **Header type:** Secure/Hidden

**Step 4: Save Connection**
- Test the connection if available
- Connection is now available for all Make modules

---

## Video Parameters

### Aspect Ratio & Dimensions for Vertical Shorts

All animal rescue shorts must use vertical/portrait format for YouTube Shorts optimization.

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Duration** | 45 seconds | Fixed for consistency |
| **Aspect Ratio** | 9:16 | Vertical/Shorts format |
| **Resolution** | 720p | Runway Gen-3 turbo default |
| **Equivalent Dimensions** | 1080x1920 pixels | YouTube Shorts optimal |
| **Frame Rate** | 24 fps | Runway Gen-3 standard |
| **Codec** | H.264 | MP4 container |
| **Bitrate** | Adaptive | Runway optimizes for platform |

### Runway API Parameters

```json
{
  "model": "gen3a_turbo",
  "prompt": "string",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": 12345
}
```

### Parameter Descriptions

| Parameter | Type | Allowed Values | Purpose |
|-----------|------|-----------------|---------|
| **model** | String | `gen3a_turbo`, `gen3a` | `gen3a_turbo` for faster processing |
| **prompt** | String | 20-500 characters | Video content description |
| **duration** | Integer | 5-120 | Video length in seconds (fixed: 45) |
| **ratio** | String | `9:16`, `16:9`, `1:1` | Aspect ratio; `9:16` for Shorts |
| **resolution** | String | `480p`, `720p`, `1080p` | Output quality; default `720p` |
| **style** | String | `cinematic`, `anime`, `realistic`, `comic` | Visual aesthetic style |
| **motion_strength** | Float | 0.0 - 1.0 | Camera/subject movement intensity |
| **seed** | Integer | 0-2147483647 | Reproducibility/variation control |

### Recommended Settings for Animal Rescue

```json
{
  "model": "gen3a_turbo",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": "{{random_seed}}"
}
```

**Rationale:**
- **gen3a_turbo:** Faster processing stays within free tier quotas
- **45 seconds:** Optimal for YouTube Shorts engagement
- **9:16 ratio:** Native Shorts format eliminates pillarboxing
- **720p:** Quality/speed balance; upscaling available
- **cinematic:** Engaging visual style for emotional content
- **motion_strength 0.8:** Natural movement without excessive camera shake
- **random seed:** Daily variety while maintaining control

---

## Prompt Templates

### Template Structure

Animal rescue prompts follow a consistent structure for consistency and quality:

```
[SCENE SETUP] [ANIMAL TYPE] [SPECIFIC SCENARIO] [RESCUE ACTION] [EMOTIONAL OUTCOME] [STYLE NOTES]
```

### Variable Definitions

```javascript
const promptVariables = {
  animal_types: [
    "golden retriever",
    "German shepherd",
    "cat",
    "elephant",
    "dolphin",
    "eagle",
    "horse",
    "rabbit"
  ],
  
  settings: [
    "drowning in a pool",
    "trapped in a snake attack",
    "lost in a forest fire",
    "injured on a highway",
    "abandoned in a shelter",
    "caught in a storm",
    "stuck in a well"
  ],
  
  rescue_methods: [
    "rescued by a brave firefighter",
    "saved by a trained rescue dog",
    "helped by veterinary care",
    "protected by a wildlife ranger",
    "aided by animal sanctuary staff",
    "assisted by volunteers"
  ],
  
  emotional_outcomes: [
    "now safe and happy",
    "reunited with family",
    "adopted into a loving home",
    "recovering with care",
    "thriving in the wild",
    "finding peace and comfort"
  ]
};
```

### Basic Prompt Template

```
A 45-second heartwarming video of a {{animal_type}} {{setting}}, 
{{rescue_methods}}, and {{emotional_outcomes}}. 
Cinematic, emotional storytelling with professional quality. 
Vertical format, focus on the animal's face and emotions.
```

### Advanced Prompt Template

```
45-second cinematic animal rescue story: {{animal_type}} in danger {{setting}}. 
Show the rescue action by {{rescue_methods}}. 
Conclude with {{emotional_outcomes}}. 
Professional cinematography, warm color grading, emotional music cues. 
Vertical 9:16 format for YouTube Shorts. 
Emphasize connection between animal and rescuer.
```

### Example Prompts with Variable Substitution

#### Example 1: Dog Rescue
```
45-second heartwarming video: A golden retriever puppy is found abandoned 
in a park during a rainstorm, shivering and scared. Rescue volunteers 
discover the puppy, dry her off with blankets, and bring her to safety 
at a local animal shelter. Within weeks of care and love, the puppy is 
adopted by a loving family and plays joyfully in their backyard. 
Cinematic, emotional, professional quality. Vertical format emphasizing 
the puppy's journey from fear to joy.
```

#### Example 2: Elephant Rescue
```
45-second cinematic animal rescue: A baby elephant trapped by a snake 
attack near a watering hole in the African savanna. Adult elephants 
and wildlife rangers work together to protect and help the calf. 
The baby elephant is now safe with the herd, standing proudly with 
its mother, healthy and strong. Professional cinematography with warm 
colors, emotional storytelling. Vertical format highlighting the 
protective bond between animals.
```

#### Example 3: Cat Rescue
```
45-second emotional rescue story: An orange tabby kitten stuck in a 
storm drain, meowing and frightened. A compassionate animal rescue 
team carefully extracts the kitten and brings her to a veterinary clinic 
for health checks. The kitten recovers fully and finds a forever home 
with a caring elderly woman who gives her endless affection. 
Heartwarming, cinematic, professional quality. Vertical Shorts format.
```

#### Example 4: Multi-Animal Community Rescue
```
45-second cinematic documentary: A local animal shelter's daily rescue 
operations—dogs, cats, and small animals all finding safety and care. 
Show the rescue process, the shelters healing environment, and animals 
being adopted into loving families. Professional cinematography, warm 
lighting, inspiring music cues. Vertical 9:16 format optimized for 
YouTube Shorts.
```

### Prompt Best Practices

**DO:**
- Start with "45-second" to anchor duration expectations
- Include specific animal types and scenarios
- Mention cinematic/professional quality
- Reference vertical format explicitly
- Use emotional language ("heartwarming", "joyful", "loved")
- Describe visual progression (before/during/after)

**DON'T:**
- Include conflicting instructions (e.g., "cinematic but low quality")
- Use vague descriptions ("a pet" instead of "golden retriever")
- Exceed 500 characters (prompt length limit)
- Request multiple animals doing unrelated things
- Mention specific brands or products
- Include violence or harm to animals

---

## Seed Randomization Strategy

### Purpose

Seeds control video generation randomness while allowing reproducibility. For daily variety without losing quality consistency, implement strategic seed rotation.

### Seed Generation Strategies

#### Strategy 1: Daily Incremental Seed

Simple approach using date-based increment:

```javascript
function generateDailySeed(date) {
  // Convert date to number: 20240101 = 2024-01-01
  const dateNum = parseInt(date.replace(/-/g, ''));
  // Add a constant offset to ensure variety
  return (dateNum * 1337) % 2147483647;
}

// Example:
// 2024-01-01 → 2024010120240101*1337 → seed: 1234567890
// 2024-01-02 → 2024010220240102*1337 → seed: 1234567891 (different)
```

#### Strategy 2: Random Seed with Time-Based Salt

More variation while maintaining control:

```javascript
function generateRandomSeed() {
  // Use current timestamp + random component
  const timestamp = Date.now();
  const random = Math.floor(Math.random() * 1000000);
  return (timestamp + random) % 2147483647;
}

// Ensures different video each run
// Reproducible within same minute if needed
```

#### Strategy 3: Hybrid: Daily Base + Random Variance

Best balance of consistency and variety:

```javascript
function generateHybridSeed(date) {
  // Base seed from date (ensures consistency for same date)
  const dateNum = parseInt(date.replace(/-/g, ''));
  const baseSeed = (dateNum * 1337) % 2147483647;
  
  // Add variance within day (1-100 variation)
  const variance = Math.floor(Math.random() * 100);
  return (baseSeed + variance) % 2147483647;
}

// Same date = similar themes
// Different runs = visual variety
```

### Make.com Implementation

**Add to Data Prep Module:**

```javascript
// Seed Generator Function in Make Webhooks/Functions

module.exports = {
  generateSeed: function(date) {
    const dateNum = parseInt(date.replace(/-/g, ''));
    const baseSeed = (dateNum * 1337) % 2147483647;
    const variance = Math.floor(Math.random() * 100);
    return (baseSeed + variance) % 2147483647;
  }
};
```

**Use in Runway API Request:**

```json
{
  "seed": {{generateSeed(current_date)}}
}
```

### Recommended Approach

For animal rescue shorts, use **Strategy 3 (Hybrid)** to balance:

| Aspect | Benefit |
|--------|---------|
| **Date-based base seed** | Same date = consistent theme/tone |
| **Random variance** | Different runs = fresh visuals |
| **Reproducibility** | Can regenerate specific date if needed |
| **Predictability** | No wild theme variations day-to-day |

---

## Free Tier Optimization

### Understanding Free Tier Limits

**Runway Free Tier:**
- ~$5-10 monthly credits (varies by region)
- Each 45-second generation: $0.01-0.05
- Provides 100-500 video generations per month
- Rate limit: 10 requests/minute
- No subscription required

### Operations Budget in Make.com

**Free Tier Limits:**
- 1,000 operations/month
- One video generation workflow uses ~25-30 operations

**Monthly Capacity:**
- 1,000 operations ÷ 25 operations/workflow = ~40 generations/month
- **Result:** Can generate 1-2 shorts daily within limits

### Optimization Techniques

#### 1. Use Gen3a Turbo (Faster Processing)

```json
{
  "model": "gen3a_turbo"
}
```

**Impact:**
- Faster completion time
- Uses fewer polling operations
- Same quality output as gen3a
- Reduces overall workflow duration

#### 2. Optimize Polling Intervals

Reduce polling frequency to save operations:

```javascript
// Instead of checking every 30 seconds...
// Wait longer between checks
const pollingIntervals = {
  initial_wait: 120,  // Wait 2 minutes before first check
  check_interval: 60, // Check every 60 seconds (not 30)
  max_attempts: 10,   // Max 10 checks instead of 15
  timeout: 600        // Timeout after 10 minutes
};
```

**Savings:** 5-10 operations per run

#### 3. Conditional Logging

Only log errors and key milestones, not every step:

```javascript
// Reduced logging approach
const loggingEvents = {
  success: true,        // Log successful uploads
  errors: true,         // Log all errors
  warnings: true,       // Log warnings
  debug_info: false     // Don't log debug details
};
```

**Savings:** 5-8 operations per run

#### 4. Batch Logging with Data Store

Combine multiple records into single data store operation:

```javascript
// Instead of logging each metric separately
// Combine into one operation
const logEntry = {
  date: "2024-01-01",
  execution_id: "exec_123",
  story: "dog rescue",
  generation_time: 180,
  video_id: "abc123",
  upload_status: "success",
  operations_used: 26,
  all_in_one_log: true
};
```

**Savings:** 3-5 operations per run

#### 5. Use Shared Storage Instead of Multiple Saves

Store video in one location, reference via URL:

```json
{
  "video_url": "https://drive.google.com/...",
  "storage_location": "gs://bucket/animals-rescue/2024-01-01.mp4",
  "youtube_upload_source": "{{storage_location}}"
}
```

**Savings:** 1-2 operations per run

### Estimated Savings Breakdown

| Optimization | Typical Save | Combined |
|--------------|-------------|----------|
| Turbo model polling | -3 ops | -3 ops |
| Optimized polling intervals | -5 ops | -8 ops |
| Conditional logging | -6 ops | -14 ops |
| Batch logging | -4 ops | -18 ops |
| Shared storage | -2 ops | -20 ops |
| **Total Potential Savings** | | **-20 operations** |
| **Before Optimization** | | ~28 ops/workflow |
| **After Optimization** | | ~8 ops/workflow |
| **New Monthly Capacity** | | ~125 generations |

**Result:** Optimized workflows use only ~8 operations each, enabling daily generation (8 × 30 = 240 ops/month) with room for retries and overhead.

---

## File Download & Processing

### Video File Information

Once generation completes, Runway provides download URLs:

**Response Format:**
```json
{
  "id": "generation_abc123def456",
  "status": "completed",
  "createdAt": "2024-01-01T09:00:15Z",
  "completedAt": "2024-01-01T09:03:45Z",
  "duration": 45,
  "width": 1080,
  "height": 1920,
  "ratio": "9:16",
  "output": {
    "url": "https://cdn.runwayml.com/videos/abc123def456/output.mp4",
    "expires_at": "2024-01-02T09:03:45Z"
  }
}
```

### File Specifications

| Property | Value |
|----------|-------|
| **Format** | MP4 (H.264/AAC) |
| **Duration** | 45 seconds |
| **Resolution** | 1080x1920 (9:16) |
| **Frame Rate** | 24 fps |
| **File Size** | 15-50 MB (varies by motion) |
| **URL Expiration** | 24 hours |
| **Codec** | H.264 video, AAC audio |

### Download Strategy

**Important:** Download must occur within 24 hours of completion.

#### Step 1: Get Download URL

From Runway API response, extract:
```
https://cdn.runwayml.com/videos/{generation_id}/output.mp4
```

#### Step 2: Download File in Make.com

**Module Type:** HTTP → Get a file

```javascript
{
  "url": "{{generation.output.url}}",
  "headers": {
    "Accept": "video/mp4"
  },
  "filename": "animals-rescue-{{date}}.mp4"
}
```

#### Step 3: Store File

**Option A: Google Drive (Recommended)**
- Reliable storage
- Accessible from YouTube uploader
- Free tier: 15 GB
- Module: Google Drive → Upload a file

**Option B: AWS S3 Bucket**
- Scalable storage
- Higher cost but reliable
- Module: AWS S3 → Upload file

**Option C: Local/Server Storage**
- Requires persistent server
- Risk of data loss
- Not recommended for automation

### Storage Location Best Practice

```
gs://bucket-name/animals-rescue/
├── 2024-01-01/
│   ├── animals-rescue-2024-01-01.mp4
│   ├── metadata.json
│   └── thumbnail.jpg
├── 2024-01-02/
│   ├── animals-rescue-2024-01-02.mp4
│   ├── metadata.json
│   └── thumbnail.jpg
...
```

### Processing After Download

**Before YouTube Upload:**

1. **Verify file integrity**
   - Check file size (should be 15-50 MB)
   - Verify duration (should be ~45 seconds)
   - Confirm aspect ratio (1080x1920)

2. **Extract/Generate Metadata**
   ```json
   {
     "filename": "animals-rescue-2024-01-01.mp4",
     "duration": 45,
     "resolution": "1080x1920",
     "size_mb": 32.5,
     "generated_at": "2024-01-01T09:03:45Z",
     "story_type": "dog_rescue",
     "animal": "golden retriever"
   }
   ```

3. **Generate Thumbnail** (Optional but Recommended)
   - Extract frame at 2-3 second mark (hook moment)
   - Optimize for YouTube (1280x720 recommended)
   - Save as JPEG

### Error Handling for Downloads

**Timeout Handling:**
```javascript
if (file_size === 0 || duration !== 45) {
  // Retry download
  retry_download_with_backoff(generation_id);
} else if (expiration_passed) {
  // Re-trigger generation
  trigger_new_generation(story_prompt);
}
```

---

## Make HTTP Integration

### Complete HTTP Request Configuration

#### Request Details

**Module Type:** HTTP → Make an API call

**URL:**
```
https://api.runwayml.com/v1/video_generations
```

**Method:** POST

#### Headers

| Header | Value |
|--------|-------|
| **Authorization** | `Bearer {{runway_api_key}}` |
| **Content-Type** | `application/json` |
| **Accept** | `application/json` |

#### Request Body

```json
{
  "model": "gen3a_turbo",
  "prompt": "{{enhanced_story_prompt}}",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": {{random_seed}}
}
```

### Make Module Configuration

#### Step 1: Add HTTP Module

1. In Make scenario, add module after Data Prep
2. Select **HTTP** → **Make an API call**
3. Set URL field

#### Step 2: Configure Authentication

In HTTP module settings:
- **Use Connection:** Select "Runway Gen-3 API" (pre-configured)
- **Alternative:** Manually add Authorization header with Bearer token

#### Step 3: Set Request Body

Click **Body** tab:
```json
{
  "model": "gen3a_turbo",
  "prompt": "{{2.enhanced_prompt}}",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": {{3.random_seed}}
}
```

**Variable References:**
- `{{2.enhanced_prompt}}` = Output from module 2 (Data Prep)
- `{{3.random_seed}}` = Output from module 3 (Seed Generator)

#### Step 4: Configure Response Mapping

In **Response** section:
```
generation_id = response.id
generation_status = response.status
estimated_time = response.estimatedProcessingTime
created_at = response.createdAt
```

### Response Processing Example

**API Response:**
```json
{
  "id": "generation_abc123",
  "status": "processing",
  "createdAt": "2024-01-01T09:00:15Z",
  "estimatedProcessingTime": 180,
  "model": "gen3a_turbo",
  "prompt": "45-second heartwarming video...",
  "seed": 12345678
}
```

**Make Mapping:**
```javascript
{
  generation_id: "generation_abc123",
  status: "processing",
  created_at: "2024-01-01T09:00:15Z",
  estimated_seconds: 180,
  ready_at: NOW() + 180 // JavaScript timestamp
}
```

### Status Check Module Configuration

After generation request, implement polling:

#### Repeater Module Setup

**Module:** Make.com → Repeater

```
Start: 1
Step: 1
Repeat: 10 times
Interval: 60 seconds
Condition: generation_status != "completed"
```

**Inside Repeater: HTTP Status Check**

```
GET https://api.runwayml.com/v1/video_generations/{{generation_id}}
Authorization: Bearer {{runway_api_key}}
```

**Response Mapping:**
```javascript
{
  status: response.status,
  output_url: response.output?.url,
  completed_at: response.completedAt
}
```

### Error Handling Configuration

**Add Error Handler after HTTP module:**

```javascript
if (response.status == 429) {
  // Rate limit - wait and retry
  wait(60);
  retry_request();
} else if (response.status == 401) {
  // Auth error - check API key
  log_error("Invalid API key");
  stop();
} else if (response.status >= 500) {
  // Server error - retry
  wait(300);
  retry_request();
}
```

---

## Example Workflows

### Complete Workflow: From Prompt to YouTube Upload

```mermaid
graph TD
    A[Start: Daily Trigger] --> B[Generate Story Prompt]
    B --> C[Create Runway Request]
    C --> D[POST to Runway API]
    D --> E{Check Response}
    E -->|Error| F[Retry Logic]
    F --> D
    E -->|Success| G[Extract Generation ID]
    G --> H[Poll Status Every 60s]
    H --> I{Completed?}
    I -->|No| H
    I -->|Yes| J[Download Video File]
    J --> K[Store in Google Drive]
    K --> L[Get Download URL]
    L --> M[Upload to YouTube]
    M --> N[Log Success]
    N --> O[End]
```

### Step-by-Step Execution Example

#### Trigger: Daily at 09:00 UTC

**Input:**
```json
{
  "timestamp": "2024-01-01T09:00:00Z",
  "date": "2024-01-01",
  "execution_id": "exec_20240101_001"
}
```

#### Module 1: Data Prep - Generate Story

**Output:**
```json
{
  "animal_type": "golden retriever",
  "setting": "drowning in a swimming pool",
  "rescue_method": "rescue by a brave firefighter",
  "outcome": "now safe and with a loving family",
  "story_text": "A golden retriever was found struggling in a swimming pool...",
  "enhanced_prompt": "45-second heartwarming video: A golden retriever struggling in a swimming pool is rescued by a brave firefighter. The dog is taken to veterinary care, recovers fully, and is adopted by a loving family where it plays happily in the backyard. Cinematic, emotional storytelling, professional quality. Vertical format."
}
```

#### Module 2: Seed Generator

**Output:**
```json
{
  "base_seed": 1234567890,
  "variance": 42,
  "final_seed": 1234567932
}
```

#### Module 3: HTTP POST to Runway API

**Request:**
```http
POST https://api.runwayml.com/v1/video_generations
Authorization: Bearer runway_api_key_abc123
Content-Type: application/json

{
  "model": "gen3a_turbo",
  "prompt": "45-second heartwarming video: A golden retriever struggling in a swimming pool...",
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p",
  "style": "cinematic",
  "motion_strength": 0.8,
  "seed": 1234567932
}
```

**Response:**
```json
{
  "id": "generation_xyz789",
  "status": "processing",
  "createdAt": "2024-01-01T09:00:30Z",
  "estimatedProcessingTime": 180,
  "model": "gen3a_turbo"
}
```

#### Module 4: Polling Loop (every 60s, max 10 attempts)

**1st Poll (60s after request):**
```
GET /video_generations/generation_xyz789
Response: { "status": "processing", "progress": 45 }
→ Continue polling
```

**2nd Poll (120s after request):**
```
Response: { "status": "processing", "progress": 85 }
→ Continue polling
```

**3rd Poll (180s after request):**
```
Response: {
  "status": "completed",
  "output": {
    "url": "https://cdn.runwayml.com/videos/generation_xyz789/output.mp4",
    "expires_at": "2024-01-02T09:00:30Z"
  }
}
→ Proceed to download
```

#### Module 5: Download Video

**HTTP GET:**
```
GET https://cdn.runwayml.com/videos/generation_xyz789/output.mp4
Save as: animals-rescue-2024-01-01.mp4
Size: 32.5 MB
Duration: 45 seconds
```

#### Module 6: Store in Google Drive

**Directory:** `/Animals Rescue Shorts/2024-01-01/`

**Files:**
```
animals-rescue-2024-01-01.mp4 (32.5 MB)
metadata.json
{
  "generation_id": "generation_xyz789",
  "date": "2024-01-01",
  "duration": 45,
  "resolution": "1080x1920",
  "seed": 1234567932,
  "generated_at": "2024-01-01T09:03:30Z"
}
```

#### Module 7: YouTube Upload

**Metadata:**
```json
{
  "snippet": {
    "title": "Golden Retriever Rescued from Pool | Heartwarming Animal Rescue #Shorts",
    "description": "Watch as a brave firefighter rescues a golden retriever from a swimming pool...\n\n🔔 Subscribe for daily heartwarming animal rescue stories!\n#AnimalRescue #Dogs #Heartwarming #Shorts",
    "tags": ["animal rescue", "dogs", "heartwarming", "shorts"],
    "categoryId": "15"
  },
  "status": {
    "privacyStatus": "public"
  }
}
```

**Upload Response:**
```json
{
  "video_id": "dQw4w9WgXcQ",
  "url": "https://youtube.com/shorts/dQw4w9WgXcQ",
  "status": "uploaded"
}
```

#### Module 8: Success Logging

**Logged to Google Sheets:**

| Date | Animal | Story | Generation Time | Upload Status | Video ID | Operations |
|------|--------|-------|-----------------|---------------|----------|------------|
| 2024-01-01 | Golden Retriever | Rescue from pool | 180s | Success | dQw4w9WgXcQ | 26 |

### Error Scenario Example: Retry After Rate Limit

**Initial Request:** `POST /video_generations` returns **429 Too Many Requests**

**Action Flow:**
1. Error Handler catches 429 status
2. Wait 300 seconds (5 minutes)
3. Retry POST request
4. If retry succeeds, continue normal flow
5. If retry fails again, log error and skip

**Error Log:**
```json
{
  "date": "2024-01-01",
  "execution_id": "exec_20240101_001",
  "error_type": "RATE_LIMIT",
  "error_message": "429 Too Many Requests",
  "retry_count": 1,
  "resolution": "Retried successfully after 300s wait"
}
```

### Cost Calculation Example

**Single Workflow (One 45-second Video):**

| Item | Cost |
|------|------|
| Runway Gen-3 video generation (45s) | $0.03 |
| Make.com operations (optimized, ~8 ops) | Free (within tier) |
| YouTube upload | Free |
| Google Drive storage (negligible) | Free |
| **Total Cost Per Video** | **~$0.03** |
| **Daily Cost** (1 video) | **$0.03** |
| **Monthly Cost** (30 videos) | **~$0.90** |

**Result:** Highly cost-effective for daily content production.

---

## Troubleshooting Guide

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **401 Unauthorized** | Invalid/expired API key | Verify API key at Runway dashboard; regenerate if needed |
| **429 Too Many Requests** | Rate limit exceeded | Implement exponential backoff; wait before retrying |
| **Video download 404** | URL expired (>24 hours) | Download must occur within 24 hours; implement immediate download |
| **YouTube upload fails** | Invalid metadata | Check video is MP4, duration 45s, 9:16 ratio |
| **Polling timeout** | Generation taking too long | Increase timeout from 10 to 15 minutes; check Runway queue |
| **Bad prompt quality** | Vague or conflicting prompt | Use provided templates; test prompts manually first |
| **File size unusually small** | Generation failed silently | Check API response for error; retry generation |

### Monitoring and Debugging

**Enable Debug Logging in Make:**
1. Add JSON parser to view raw API responses
2. Log all HTTP requests/responses to Google Sheets
3. Monitor Runway Usage Dashboard in parallel
4. Set up alerts for failed workflows (Make.com webhooks)

---

## Summary

This configuration provides:
- ✅ Complete API authentication setup for Make.com
- ✅ Precise video parameters for 45-second vertical Shorts
- ✅ Reusable prompt templates with examples
- ✅ Seed randomization for daily variety
- ✅ Free tier optimization techniques
- ✅ File download and processing workflow
- ✅ Make HTTP integration patterns
- ✅ Complete example workflows with real data

**Next Steps:**
1. Set up Runway API key and Make connection (see [credentials.md](./credentials.md))
2. Copy prompt templates for your story generator
3. Implement seed randomization strategy
4. Test HTTP requests using examples provided
5. Set up Google Drive storage for videos
6. Configure YouTube upload following make-workflow-blueprint.md
7. Monitor usage at Runway and Make dashboards

For additional guidance, see:
- **Credentials Setup:** [credentials.md](./credentials.md)
- **Complete Workflow:** [make-workflow-blueprint.md](./make-workflow-blueprint.md)

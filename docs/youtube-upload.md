# YouTube Shorts Auto-Upload Configuration for Make.com

## Overview

This document provides complete configuration for the **YouTube Upload Module** in Make.com workflows. It covers OAuth setup, video metadata templates, Shorts format compliance, and testing procedures for automated animal rescue content publishing.

---

## Table of Contents

1. [YouTube Module Setup](#youtube-module-setup)
2. [OAuth Connection & Channel Selection](#oauth-connection--channel-selection)
3. [Video Upload Configuration](#video-upload-configuration)
4. [Shorts Format Compliance](#shorts-format-compliance)
5. [Title Templates](#title-templates)
6. [Description Templates](#description-templates)
7. [Thumbnail Configuration](#thumbnail-configuration)
8. [Publishing Schedule](#publishing-schedule)
9. [Metadata Variable Mapping](#metadata-variable-mapping)
10. [Testing & Verification](#testing--verification)
11. [Troubleshooting](#troubleshooting)

---

## YouTube Module Setup

### Module Selection in Make.com

**Step 1: Add YouTube Module to Scenario**
1. Open your Make.com scenario
2. Click the **+** icon where you want to add the upload module
3. Search for **"YouTube"** in the apps list
4. Select **YouTube** → **Upload a Video**

**Step 2: Module Configuration Overview**

The YouTube > Upload a Video module requires the following key configurations:

| Field | Required | Purpose |
|-------|----------|---------|
| **Connection** | Yes | OAuth authentication |
| **Video File** | Yes | MP4 file from Runway download |
| **Title** | Yes | Video title (max 100 characters) |
| **Description** | No | Video description (max 5,000 characters) |
| **Tags** | No | Search keywords (max 500 characters) |
| **Category** | No | YouTube category ID |
| **Privacy Status** | Yes | Public/Unlisted/Private |
| **Made for Kids** | Yes | COPPA compliance setting |
| **Notify Subscribers** | No | Send notification on publish |

---

## OAuth Connection & Channel Selection

### Creating YouTube OAuth Connection

**Step 1: Add New Connection**
1. In the YouTube module, click the **Connection** dropdown
2. Select **Add** to create a new connection
3. Choose connection method:
   - **Recommended:** Use Make's built-in Google OAuth flow
   - **Advanced:** Use custom OAuth credentials (see below)

**Step 2: Built-in OAuth Flow (Recommended)**

1. Click **Sign in with Google**
2. Select the Google account linked to your target YouTube channel
3. Review the requested permissions:
   ```
   - View your YouTube account
   - Manage your YouTube videos
   - Upload videos to YouTube
   ```
4. Click **Allow** to grant permissions
5. You'll be redirected back to Make.com
6. The connection will be automatically saved

**Connection Name:** Use a descriptive name like "YouTube - Animals Rescue Channel"

**Step 3: Custom OAuth Credentials (Advanced)**

If you need to use your own Google Cloud project credentials:

1. In the connection dialog, click **Show advanced settings**
2. Enter your credentials:
   - **Client ID**: `YOUR_CLIENT_ID.apps.googleusercontent.com`
   - **Client Secret**: `YOUR_CLIENT_SECRET`
3. Click **Continue**
4. Complete the OAuth authorization flow
5. Save the connection

### Required OAuth Scopes

The YouTube upload module requires these OAuth 2.0 scopes:

```
https://www.googleapis.com/auth/youtube.upload
https://www.googleapis.com/auth/youtube
https://www.googleapis.com/auth/youtube.force-ssl
```

**Scope Permissions:**
- **youtube.upload**: Upload, edit, and delete videos
- **youtube**: Manage your YouTube account
- **youtube.force-ssl**: Secure API access

### Channel Selection

**Automatic Channel Detection:**
- Make.com automatically uses the default channel for the authenticated Google account
- If you have multiple channels, the default channel will be selected

**To Use a Specific Channel:**
1. Ensure you're authenticated with the correct Google account
2. The account's default YouTube channel will be used
3. To switch channels, create a separate connection with a different Google account

**Verification:**
- The module will display the channel name after successful connection
- Test uploads will appear on the selected channel

---

## Video Upload Configuration

### Basic Upload Settings

Configure the YouTube module with these settings for automated Shorts uploads:

```json
{
  "connection": "{{YouTube - Animals Rescue Channel}}",
  "videoFile": "{{downloaded_video_file}}",
  "title": "{{video_title}}",
  "description": "{{video_description}}",
  "tags": "{{video_tags}}",
  "categoryId": "15",
  "privacyStatus": "public",
  "selfDeclaredMadeForKids": false,
  "notifySubscribers": true,
  "embeddable": true,
  "publicStatsViewable": true
}
```

### Field-by-Field Configuration

#### Video File
- **Source:** Output from previous "Download Video" module
- **Format:** MP4 file from Runway Gen-3
- **Path Mapping:** `{{previous_module.file}}` or `{{google_drive.file_url}}`
- **Validation:** Ensure file size < 256 GB (YouTube limit)

#### Privacy Status
- **Value:** `public`
- **Options:** `public`, `unlisted`, `private`
- **Recommendation:** Use `public` for immediate Shorts feed visibility
- **Alternative:** Use `unlisted` for testing, then manually publish

#### Category
- **Value:** `15` (Pets & Animals)
- **Alternative Categories:**
  - `22` - People & Blogs
  - `24` - Entertainment
  - `26` - Howto & Style

#### Made for Kids (COPPA Compliance)
- **Value:** `false`
- **Reason:** Animal rescue content is general audience, not specifically for children
- **Important:** Setting this correctly ensures proper ad targeting and features

#### Notify Subscribers
- **Value:** `true`
- **Purpose:** Alerts subscribers when new Short is published
- **Alternative:** Set to `false` to avoid notification fatigue for daily uploads

### Advanced Settings

#### Embeddable
- **Value:** `true`
- **Purpose:** Allows video to be embedded on other websites
- **Benefit:** Increases reach and shareability

#### Public Stats Viewable
- **Value:** `true`
- **Purpose:** Shows view count publicly
- **Benefit:** Social proof for viral Shorts

#### License
- **Value:** `youtube` (Standard YouTube License)
- **Alternative:** `creativeCommon` for Creative Commons license

---

## Shorts Format Compliance

### YouTube Shorts Technical Requirements

YouTube automatically classifies videos as Shorts when they meet these criteria:

| Requirement | Value | Validation |
|-------------|-------|------------|
| **Duration** | ≤ 60 seconds | ✅ Runway set to 45s |
| **Aspect Ratio** | 9:16 (vertical) | ✅ Runway set to 9:16 |
| **Resolution** | ≥ 1080x1920 pixels | ✅ Runway outputs 1080x1920 |
| **Orientation** | Portrait | ✅ Automatic from 9:16 |
| **Container** | MP4 | ✅ Runway default format |

### Automatic Shorts Detection

**How YouTube Identifies Shorts:**
1. **Duration Check:** Video must be 60 seconds or less
2. **Aspect Ratio Check:** Video must be square (1:1) or vertical (9:16, 4:5)
3. **Metadata Check:** Title/description can include "#Shorts" (optional but recommended)

**Runway Gen-3 Compliance:**
```json
{
  "duration": 45,
  "ratio": "9:16",
  "resolution": "720p"
}
```

**YouTube Processing:**
- Runway outputs 1080x1920 at 720p quality
- YouTube accepts and processes as Shorts automatically
- No manual "Shorts" designation needed
- Video appears in Shorts feed within minutes

### Validation in Make.com

Add a validation step before upload to ensure compliance:

**Validation Module (Optional):**
```javascript
// Add a "Tools > Set Variable" module to validate
const videoMetadata = {
  duration: {{runway_response.duration}},
  width: {{runway_response.width}},
  height: {{runway_response.height}},
  aspectRatio: {{runway_response.width}} / {{runway_response.height}}
};

// Validate Shorts compliance
if (videoMetadata.duration > 60) {
  throw new Error("Video exceeds 60 seconds - not Shorts compliant");
}

if (videoMetadata.aspectRatio < 0.5 || videoMetadata.aspectRatio > 0.6) {
  throw new Error("Aspect ratio not 9:16 - not Shorts compliant");
}

// Pass to next module
return videoMetadata;
```

### Shorts-Specific Optimization

**Title Optimization:**
- Include "#Shorts" hashtag in title or description
- Keep titles concise (under 60 characters for mobile display)
- Use emotional hooks in first words

**Description Optimization:**
- First 2-3 lines visible without expansion
- Include key hashtags early
- Add call-to-action (Subscribe, Like)

**Hashtag Strategy:**
- Use `#Shorts` as primary tag
- Add 3-5 relevant hashtags (#AnimalRescue, #WildlifeHeroes)
- Avoid hashtag spam (YouTube may suppress)

---

## Title Templates

### Template Structure

Titles should be engaging, descriptive, and optimized for Shorts discovery.

**Format:**
```
Animals Rescue Humans - [Story Variant] #Shorts
```

**Maximum Length:** 100 characters (optimal: 50-60 for mobile)

### Story Variants

Generate random story variants for daily variety:

```javascript
const storyVariants = [
  "Heartwarming Dog Rescue",
  "Saving a Stray Cat",
  "Elephant Herd Protection",
  "Wildlife Hero Moment",
  "Abandoned Puppy Finds Home",
  "Injured Bird Recovery",
  "Horse Rescue Mission",
  "Dolphin Safety Story",
  "Kitten Saved from Storm",
  "Forest Animal Rescue",
  "Baby Elephant Protection",
  "Shelter Dog Adoption",
  "Cat Rescue Operation",
  "Wildlife Rehabilitation",
  "Street Dog Transformation",
  "Rescue Team Heroes",
  "Lost Pet Reunion",
  "Brave Firefighter Saves Pet",
  "Veterinary Miracle Story",
  "Animal Sanctuary Rescue"
];

// Random selection in Make.com
const randomVariant = storyVariants[Math.floor(Math.random() * storyVariants.length)];
```

### Title Template Examples

#### Example 1: Emotional Hook
```
Animals Rescue Humans - Heartwarming Dog Rescue #Shorts
```

#### Example 2: Specific Animal
```
Animals Rescue Humans - Baby Elephant Protection #Shorts
```

#### Example 3: Action-Focused
```
Animals Rescue Humans - Brave Firefighter Saves Pet #Shorts
```

#### Example 4: Outcome-Focused
```
Animals Rescue Humans - Lost Pet Reunion #Shorts
```

#### Example 5: Location-Based
```
Animals Rescue Humans - Forest Animal Rescue #Shorts
```

### Title Template with Date (Optional)

For tracking and uniqueness:

```
Animals Rescue Humans - [Story Variant] | [Month Day] #Shorts
```

**Example:**
```
Animals Rescue Humans - Shelter Dog Adoption | Jan 15 #Shorts
```

### Make.com Title Configuration

**In the YouTube Upload Module:**

```javascript
// Title Field
Animals Rescue Humans - {{random_story_variant}} #Shorts
```

**Variable Mapping:**
```json
{
  "title": "Animals Rescue Humans - {{data_prep.story_variant}} #Shorts"
}
```

**Dynamic Title Generation:**
```javascript
// In Data Prep Module
const storyTypes = {
  dog: "Heartwarming Dog Rescue",
  cat: "Saving a Stray Cat",
  elephant: "Baby Elephant Protection",
  wildlife: "Wildlife Hero Moment",
  shelter: "Shelter Animal Adoption"
};

const titleTemplate = `Animals Rescue Humans - ${storyTypes[animal_type]} #Shorts`;
```

### Title Best Practices

**DO:**
- ✅ Keep under 60 characters for mobile visibility
- ✅ Include "#Shorts" for algorithm recognition
- ✅ Use emotional words (heartwarming, brave, rescue)
- ✅ Include animal type for search optimization
- ✅ Use title case for readability

**DON'T:**
- ❌ Use clickbait or misleading titles
- ❌ Include special characters (except # and -)
- ❌ Exceed 100 characters (YouTube limit)
- ❌ Use all caps (appears spammy)
- ❌ Include misleading hashtags

---

## Description Templates

### Description Structure

A well-structured description improves SEO and viewer engagement.

**Format:**
```
[Hook/Summary - 2-3 lines]

[Extended Story - 3-4 lines]

[Call to Action]

[Hashtags]

[Credits/Disclaimers]
```

### Base Description Template

```
🐾 Heartwarming animal rescue story that will touch your heart! Watch as [animal_type] gets rescued and finds a loving home.

This incredible moment shows the power of compassion and the bond between animals and humans. Every rescue makes a difference! 🌟

🔔 Subscribe for daily animal rescue stories
👍 Like if you love animal heroes
💬 Comment your favorite part below

#AnimalRescue #WildlifeHeroes #Shorts #RescueDogs #RescueCats #Heartwarming #AnimalLovers #PetsOfYouTube #DailyShorts #FeelGood

Video generated using AI for educational and awareness purposes about animal rescue initiatives worldwide.
```

### Dynamic Description Template

```javascript
// Description with variable substitution
const descriptionTemplate = `
🐾 ${story_summary}

${extended_narrative}

Every day, brave rescuers around the world save animals in need. This story reminds us of the incredible bond between humans and animals. 🌟

🔔 Subscribe for daily animal rescue stories
👍 Like if this touched your heart
💬 Share your own rescue story below

#AnimalRescue #WildlifeHeroes #Shorts #${animal_hashtag} #Heartwarming #AnimalLovers #PetsOfYouTube #DailyShorts #FeelGood #Rescue #Adoption

This video is AI-generated for awareness and educational purposes about animal welfare and rescue operations.
`;
```

### Description Template Examples

#### Example 1: Dog Rescue
```
🐾 A golden retriever found abandoned in the rain gets rescued and finds a forever home!

Watch this heartwarming transformation from scared puppy to happy family dog. The volunteers at the local shelter gave him a second chance at life, and now Max lives his best days playing in a loving backyard.

Every rescue story matters. Every animal deserves love. 🌟

🔔 Subscribe for daily animal rescue stories
👍 Like if dogs are your favorite
💬 Comment "rescued" if you've saved a pet

#AnimalRescue #WildlifeHeroes #Shorts #DogRescue #RescueDogs #GoldenRetriever #Heartwarming #AnimalLovers #PetsOfYouTube #Adoption #DailyShorts #FeelGood

AI-generated content for animal welfare awareness and education.
```

#### Example 2: Wildlife Rescue
```
🐾 Baby elephant protected from danger by herd and wildlife rangers!

In the African savanna, a young elephant faces a threat but the protective instincts of the herd and quick action from rangers save the day. This is the power of wildlife conservation in action. 🌍

Nature's heroes come in all forms. Protect wildlife. 🌟

🔔 Subscribe for daily animal rescue stories
👍 Like to support wildlife conservation
💬 Share to spread awareness

#AnimalRescue #WildlifeHeroes #Shorts #Elephant #Wildlife #Conservation #Heartwarming #Nature #SaveAnimals #DailyShorts #FeelGood #WildlifeProtection

Educational AI-generated content about wildlife rescue and conservation efforts worldwide.
```

#### Example 3: Shelter Adoption
```
🐾 Shelter cat finally finds a loving home after months of waiting!

This orange tabby spent 6 months in a rescue shelter hoping for a family. Today, she met her perfect match—an elderly woman looking for a companion. Now they're inseparable! 😻

Adopt, don't shop. Save a life. 🌟

🔔 Subscribe for daily animal rescue stories
👍 Like if you love happy endings
💬 Comment your adoption story

#AnimalRescue #WildlifeHeroes #Shorts #CatRescue #RescueCats #Adoption #AnimalShelter #Heartwarming #AnimalLovers #PetsOfYouTube #AdoptDontShop #DailyShorts

AI-generated for promoting animal shelter adoption and rescue awareness.
```

### Hashtag Strategy

**Primary Hashtags (Always Include):**
- `#AnimalRescue`
- `#Shorts`
- `#WildlifeHeroes`

**Animal-Specific Hashtags:**
- Dogs: `#DogRescue #RescueDogs #Puppies #DogsOfYouTube`
- Cats: `#CatRescue #RescueCats #Kittens #CatsOfYouTube`
- Wildlife: `#Wildlife #Conservation #SaveAnimals #WildlifeProtection`
- General: `#AnimalLovers #PetsOfYouTube #AnimalWelfare`

**Engagement Hashtags:**
- `#Heartwarming`
- `#FeelGood`
- `#DailyShorts`
- `#Viral`

**Maximum Hashtags:** 10-15 hashtags (avoid spam)

### Make.com Description Configuration

**In the YouTube Upload Module:**

```json
{
  "description": "{{video_description_template}}"
}
```

**Variable Mapping from Data Prep:**
```javascript
// Generate description in Data Prep module
const description = `
🐾 ${storyData.summary}

${storyData.extended_narrative}

Every rescue makes a difference! 🌟

🔔 Subscribe for daily animal rescue stories
👍 Like if you love animals
💬 Comment your thoughts below

#AnimalRescue #WildlifeHeroes #Shorts #${storyData.animal_tag} #Heartwarming #AnimalLovers #PetsOfYouTube #DailyShorts #FeelGood

AI-generated for animal welfare awareness.
`;

return { description };
```

---

## Thumbnail Configuration

### Automatic Thumbnail Selection

YouTube automatically generates 3 thumbnail options from video frames.

**Default Behavior:**
- YouTube selects frames at 25%, 50%, and 75% of video duration
- For 45-second videos: frames at ~11s, 22.5s, and 34s
- One thumbnail is auto-selected as default

**In Make.com:**
```json
{
  "thumbnail": "auto"
}
```

### Custom Thumbnail Upload (Optional)

To upload a custom thumbnail, add a thumbnail file to the upload request:

**Requirements:**
- **Format:** JPG, PNG, GIF, or BMP
- **Resolution:** 1280x720 pixels (16:9) minimum
- **File Size:** Max 2 MB
- **Aspect Ratio:** 16:9 (for general display) or 9:16 (for Shorts preview)

**Make.com Configuration:**
```json
{
  "thumbnail": "{{thumbnail_file_url}}"
}
```

### Thumbnail from Runway Video Frame

**Method 1: Extract Frame Using FFmpeg**

Add an intermediate module to extract a frame from the Runway video:

```bash
# Extract frame at 5 seconds
ffmpeg -i {{video_file}} -ss 00:00:05 -vframes 1 -q:v 2 thumbnail.jpg
```

**Method 2: Use First Frame**

```bash
# Extract first frame
ffmpeg -i {{video_file}} -vframes 1 -q:v 2 thumbnail.jpg
```

**Method 3: Use Middle Frame (Most Engaging)**

```bash
# Extract middle frame (22.5 seconds for 45s video)
ffmpeg -i {{video_file}} -ss 00:00:22.5 -vframes 1 -q:v 2 thumbnail.jpg
```

### Make.com Thumbnail Workflow

**Step 1: Add Tools > Run Command Module**
```bash
ffmpeg -i {{downloaded_video.file}} -ss 00:00:22.5 -vframes 1 -q:v 2 /tmp/thumbnail_{{execution_id}}.jpg
```

**Step 2: Upload Thumbnail to Google Drive**
- Module: Google Drive > Upload a File
- File: `/tmp/thumbnail_{{execution_id}}.jpg`
- Folder: Thumbnails folder

**Step 3: Get Public URL**
- Module: Google Drive > Get a Share Link
- File: Uploaded thumbnail
- Output: Public URL

**Step 4: Add to YouTube Upload**
```json
{
  "thumbnail": "{{google_drive.thumbnail_url}}"
}
```

### Thumbnail Best Practices

**DO:**
- ✅ Use emotional animal close-ups
- ✅ Show the rescue moment or happy outcome
- ✅ Ensure thumbnail is bright and clear
- ✅ Test on mobile preview (Shorts are primarily mobile)
- ✅ Use the most engaging frame from video

**DON'T:**
- ❌ Use misleading thumbnails
- ❌ Add text overlays (Shorts thumbnails are small)
- ❌ Use dark or blurry frames
- ❌ Include irrelevant imagery
- ❌ Use copyrighted images

### Auto-Selection Recommendation

For automated daily uploads, **use YouTube's automatic thumbnail selection**:

**Advantages:**
- Zero additional operations in Make.com
- No image processing required
- YouTube AI selects most engaging frame
- Reduces workflow complexity
- Saves processing time and costs

**Configuration:**
```json
{
  "thumbnail": null
}
```
or simply omit the thumbnail field entirely.

---

## Publishing Schedule

### Immediate Publishing (Auto-Publish)

**Configuration:**
```json
{
  "privacyStatus": "public",
  "publishAt": null
}
```

**Behavior:**
- Video is published immediately upon upload
- Appears in Shorts feed within 1-5 minutes
- Subscribers receive notification (if enabled)
- No manual approval required

**Recommended For:**
- Fully automated workflows
- Daily scheduled uploads
- Content that doesn't require review

### Scheduled Publishing (Future Date)

**Configuration:**
```json
{
  "privacyStatus": "private",
  "publishAt": "{{scheduled_datetime}}"
}
```

**Datetime Format:** ISO 8601 format
```
2024-12-15T09:00:00Z
```

**Example Scenarios:**

#### Schedule for Specific Time of Day
```javascript
// Publish at 9 AM UTC daily
const scheduledTime = new Date();
scheduledTime.setUTCHours(9, 0, 0, 0);
scheduledTime.setDate(scheduledTime.getDate() + 1); // Tomorrow

return {
  publishAt: scheduledTime.toISOString()
};
```

#### Schedule for Peak Engagement Hours
```javascript
// Publish at optimal times based on audience
const peakHours = [9, 12, 18, 21]; // UTC hours
const randomHour = peakHours[Math.floor(Math.random() * peakHours.length)];

const scheduledTime = new Date();
scheduledTime.setUTCHours(randomHour, 0, 0, 0);

return {
  publishAt: scheduledTime.toISOString()
};
```

### Publishing Time Best Practices

**Global Audience Optimization:**

| Time (UTC) | Target Audience | Reasoning |
|------------|-----------------|-----------|
| 09:00 UTC | Europe Morning | Commute time, high mobile usage |
| 12:00 UTC | Europe Lunch, US East Morning | Peak break time |
| 18:00 UTC | US East Evening, Asia Morning | After work/school |
| 21:00 UTC | US Prime Time | Evening relaxation time |

**Recommendation for Animal Rescue Shorts:**
- **Primary:** 09:00 UTC (covers Europe & US morning)
- **Alternative:** 18:00 UTC (covers global evening/morning)

### Make.com Schedule Configuration

**Option 1: Immediate Publish (Recommended)**
```json
{
  "privacyStatus": "public"
}
```

**Option 2: Scheduled Publish**

Add a "Set Variable" module before YouTube upload:

```javascript
// Calculate publish time
const publishTime = new Date();
publishTime.setUTCHours(9, 0, 0, 0);
publishTime.setDate(publishTime.getDate() + 1);

return {
  publishAt: publishTime.toISOString(),
  privacyStatus: "private"
};
```

Then in YouTube module:
```json
{
  "privacyStatus": "{{calculated_privacy}}",
  "publishAt": "{{calculated_publish_time}}"
}
```

### Timezone Considerations

**UTC Conversion Examples:**

| Location | Local Time | UTC Time |
|----------|------------|----------|
| New York (EST) | 04:00 AM | 09:00 UTC |
| London (GMT) | 09:00 AM | 09:00 UTC |
| Tokyo (JST) | 06:00 PM | 09:00 UTC |
| Sydney (AEST) | 08:00 PM | 09:00 UTC |

**Always use UTC in Make.com workflows** to avoid timezone confusion.

---

## Metadata Variable Mapping

### Data Flow from Earlier Workflow Steps

Map variables from previous modules to YouTube upload fields:

**Workflow Data Flow:**
```
1. Scheduler → execution_date, execution_id
2. Data Prep → story_variant, animal_type, narrative
3. Runway API → generation_id, video_prompt
4. Status Monitor → video_status, completion_time
5. Download → video_file, file_size, duration
6. YouTube Upload → Use all above variables
```

### Variable Mapping Table

| YouTube Field | Source Module | Variable Path | Example Value |
|---------------|---------------|---------------|---------------|
| **videoFile** | Download Module | `{{download.file}}` | `/tmp/video_123.mp4` |
| **title** | Data Prep | `{{data_prep.story_variant}}` | "Heartwarming Dog Rescue" |
| **description** | Data Prep | `{{data_prep.description}}` | Full description text |
| **tags** | Data Prep | `{{data_prep.tags_array}}` | ["animal rescue", "dogs"] |
| **categoryId** | Static | `15` | 15 (Pets & Animals) |
| **privacyStatus** | Static | `public` | "public" |
| **selfDeclaredMadeForKids** | Static | `false` | false |

### Complete Mapping Configuration

**In YouTube Upload Module:**

```json
{
  "connection": "{{youtube_connection}}",
  "videoFile": "{{download_module.video_file}}",
  "title": "Animals Rescue Humans - {{data_prep_module.story_variant}} #Shorts",
  "description": "{{data_prep_module.full_description}}",
  "tags": ["animal rescue", "{{data_prep_module.animal_type}}", "shorts", "wildlife heroes", "heartwarming", "{{data_prep_module.specific_tag}}"],
  "categoryId": "15",
  "privacyStatus": "public",
  "selfDeclaredMadeForKids": false,
  "notifySubscribers": true,
  "embeddable": true,
  "publicStatsViewable": true
}
```

### Advanced Variable Mapping

#### Dynamic Tags Generation

```javascript
// In Data Prep Module - Generate tags dynamically
const baseTags = ["animal rescue", "wildlife heroes", "shorts", "heartwarming"];
const animalTags = {
  dog: ["dog rescue", "rescue dogs", "puppies"],
  cat: ["cat rescue", "rescue cats", "kittens"],
  elephant: ["elephant", "wildlife", "conservation"],
  wildlife: ["wildlife", "nature", "wild animals"]
};

const finalTags = [...baseTags, ...animalTags[animal_type]];

return { tags: finalTags };
```

#### Dynamic Description with Metadata

```javascript
// Include Runway metadata in description
const description = `
🐾 ${story_summary}

${extended_narrative}

🔔 Subscribe for daily animal rescue stories
👍 Like if you love animals
💬 Comment below

#AnimalRescue #WildlifeHeroes #Shorts #${animal_tag}

---
Video ID: ${runway_generation_id}
Generated: ${execution_date}
Duration: ${video_duration}s
`;

return { description };
```

#### Recording Details

```json
{
  "recordingDetails": {
    "recordingDate": "{{current_date}}"
  }
}
```

### Variable Validation

Add validation before upload:

```javascript
// Validate all required fields are present
const requiredFields = {
  videoFile: {{download.file}},
  title: {{data_prep.story_variant}},
  description: {{data_prep.description}}
};

for (const [field, value] of Object.entries(requiredFields)) {
  if (!value || value === "") {
    throw new Error(`Missing required field: ${field}`);
  }
}

// Validate title length
if (requiredFields.title.length > 100) {
  throw new Error("Title exceeds 100 characters");
}

// Validate description length
if (requiredFields.description.length > 5000) {
  throw new Error("Description exceeds 5000 characters");
}

return requiredFields;
```

---

## Testing & Verification

### Pre-Upload Testing Checklist

Before running your automated workflow, verify the following:

#### 1. OAuth Connection Test

**Steps:**
1. ✅ Open Make.com scenario
2. ✅ Navigate to YouTube Upload module
3. ✅ Click on the **Connection** dropdown
4. ✅ Select your YouTube connection
5. ✅ Click **Test Connection** (if available)
6. ✅ Verify connection status shows "Connected" with green checkmark
7. ✅ Confirm the correct channel name is displayed

**Expected Result:**
```
✅ Connection successful
Channel: Your Channel Name
Status: Active
```

**Troubleshooting:**
- ❌ If connection fails, re-authenticate using OAuth flow
- ❌ Check that OAuth scopes include `youtube.upload`
- ❌ Verify your Google account has access to the target channel

#### 2. Channel Selection Verification

**Steps:**
1. ✅ Log into [YouTube Studio](https://studio.youtube.com)
2. ✅ Check that you're viewing the correct channel
3. ✅ Navigate to Settings → Channel → Advanced settings
4. ✅ Note the **Channel ID** (starts with `UC`)
5. ✅ In Make.com, run a test scenario
6. ✅ Check the uploaded video appears on the correct channel

**Expected Result:**
```
✅ Video appears on intended channel
✅ Channel ID matches target channel
✅ Channel branding is correct
```

#### 3. Video Format Validation

**Steps:**
1. ✅ Download a sample video from Runway Gen-3
2. ✅ Verify file properties:
   - **Duration:** 45 seconds (< 60s for Shorts)
   - **Resolution:** 1080x1920 pixels
   - **Aspect Ratio:** 9:16 (vertical)
   - **Format:** MP4
   - **Codec:** H.264
3. ✅ Use `ffprobe` or `mediainfo` to validate:
   ```bash
   ffprobe -v error -select_streams v:0 -show_entries stream=width,height,duration,codec_name {{video_file}}
   ```

**Expected Output:**
```
width=1080
height=1920
duration=45.000000
codec_name=h264
```

#### 4. Metadata Template Test

**Steps:**
1. ✅ Run Data Prep module in isolation
2. ✅ Verify title generation:
   - Length < 100 characters
   - Includes "#Shorts" hashtag
   - Contains animal type and story variant
3. ✅ Verify description generation:
   - Length < 5000 characters
   - Includes hashtags
   - Contains call-to-action
4. ✅ Verify tags array:
   - 5-10 tags
   - Relevant keywords
   - No duplicate tags

**Expected Output:**
```json
{
  "title": "Animals Rescue Humans - Heartwarming Dog Rescue #Shorts",
  "description": "🐾 A golden retriever found abandoned...",
  "tags": ["animal rescue", "dog rescue", "shorts", "wildlife heroes", "heartwarming"]
}
```

#### 5. Privacy Status Check

**Steps:**
1. ✅ Confirm `privacyStatus` is set to `"public"`
2. ✅ Verify `selfDeclaredMadeForKids` is set to `false`
3. ✅ Check `notifySubscribers` is set to `true` (or `false` for testing)
4. ✅ Ensure `embeddable` is set to `true`

#### 6. End-to-End Test Upload

**Steps:**
1. ✅ Run complete workflow from scheduler to upload
2. ✅ Monitor each module for successful execution
3. ✅ Check Make.com execution log for errors
4. ✅ Wait 2-5 minutes for YouTube processing
5. ✅ Verify video appears in [YouTube Studio](https://studio.youtube.com)
6. ✅ Check video appears in Shorts feed (may take 1 hour)

**Expected Timeline:**
- **Upload:** 1-3 minutes (depending on file size)
- **Processing:** 2-5 minutes (YouTube processing)
- **Shorts Feed:** 30 minutes to 2 hours

### Post-Upload Verification Steps

#### 1. Confirm Successful Upload

**Check in Make.com:**
```json
{
  "status": "success",
  "video_id": "dQw4w9WgXcQ",
  "upload_time": "2024-01-01T09:05:32Z"
}
```

**Check in YouTube Studio:**
1. ✅ Navigate to [YouTube Studio](https://studio.youtube.com)
2. ✅ Click **Content** in left sidebar
3. ✅ Find your uploaded video (sort by "Date published (newest)")
4. ✅ Verify status shows **"Public"** with green dot
5. ✅ Check video details match your metadata

#### 2. Verify Shorts Appearance

**Shorts Feed Check:**
1. ✅ Open YouTube app on mobile device
2. ✅ Navigate to **Shorts** tab
3. ✅ Search for your channel name
4. ✅ Verify video appears in Shorts feed (may take 30-120 minutes)
5. ✅ Check video plays in vertical format

**Desktop Check:**
1. ✅ Visit your channel page on YouTube
2. ✅ Click on **Shorts** tab
3. ✅ Verify video is listed as a Short
4. ✅ Check that duration shows as "Shorts" badge

**Expected Result:**
```
✅ Video appears in Shorts feed
✅ Vertical format (9:16) maintained
✅ Duration under 60 seconds
✅ "#Shorts" badge visible
```

#### 3. Metadata Validation

**Check Title:**
- ✅ Title matches template
- ✅ Includes "#Shorts" hashtag
- ✅ No truncation (< 100 chars)
- ✅ Readable on mobile preview

**Check Description:**
- ✅ Description formatted correctly
- ✅ Hashtags rendered as clickable links
- ✅ Call-to-action visible
- ✅ Line breaks preserved

**Check Tags:**
- ✅ Navigate to YouTube Studio → Video Details
- ✅ Scroll to "Tags" section
- ✅ Verify all tags are applied
- ✅ Confirm no duplicate tags

#### 4. Thumbnail Verification

**Check Thumbnail:**
1. ✅ View video in Shorts feed
2. ✅ Check that thumbnail is clear and engaging
3. ✅ Verify thumbnail matches video content
4. ✅ Test thumbnail on mobile device

**If using auto-generated thumbnail:**
- ✅ YouTube selected appropriate frame
- ✅ Thumbnail shows animal clearly
- ✅ No black bars or distortion

#### 5. Shorts Format Compliance Check

**Technical Verification:**
```
✅ Duration: 45 seconds (< 60s limit)
✅ Aspect Ratio: 9:16 (vertical)
✅ Resolution: 1080x1920 pixels
✅ Orientation: Portrait
✅ Shorts Badge: Visible on video
✅ Shorts Feed: Video appears correctly
```

**If video does NOT appear as Short:**
- ❌ Check duration (must be ≤ 60s)
- ❌ Verify aspect ratio (must be 9:16 or 1:1)
- ❌ Wait 2-4 hours (YouTube processing delay)
- ❌ Re-upload with correct specifications

#### 6. Engagement & Visibility Check

**Analytics Check (After 24 hours):**
1. ✅ Navigate to YouTube Studio → Analytics
2. ✅ Filter by your uploaded Short
3. ✅ Check key metrics:
   - **Views:** Confirm video is getting views
   - **Watch Time:** Verify average watch percentage
   - **Traffic Source:** Check if coming from Shorts feed
   - **Impressions:** Confirm video is being shown

**Subscriber Notification:**
- ✅ If `notifySubscribers: true`, check that notification was sent
- ✅ Ask test subscribers to confirm they received notification

### Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| **Video not appearing as Short** | Duration > 60s or wrong aspect ratio | Verify Runway config: 45s, 9:16 ratio |
| **Upload fails with 401 error** | OAuth token expired | Re-authenticate YouTube connection |
| **Thumbnail not displaying** | Invalid thumbnail URL or format | Use auto-generated or fix image URL |
| **Title/description empty** | Variable mapping error | Check Data Prep module outputs |
| **Video is private** | `privacyStatus` not set to `public` | Update YouTube module config |
| **Shorts badge missing** | Processing delay | Wait 2-4 hours for YouTube processing |
| **Notification not sent** | `notifySubscribers: false` | Enable notification in config |
| **Video quality low** | Runway resolution too low | Ensure Runway outputs 720p minimum |

### Testing Checklist Summary

Before going live with automated uploads:

- [ ] OAuth connection tested and active
- [ ] Correct YouTube channel selected
- [ ] Video format validated (45s, 9:16, 1080x1920)
- [ ] Metadata templates generate correctly
- [ ] Title includes "#Shorts" and < 100 chars
- [ ] Description includes hashtags and CTA
- [ ] Tags array has 5-10 relevant keywords
- [ ] Privacy status set to "public"
- [ ] Test upload completed successfully
- [ ] Video appears in YouTube Studio
- [ ] Video appears in Shorts feed within 2 hours
- [ ] Thumbnail displays correctly
- [ ] Mobile preview looks good
- [ ] Analytics tracking works
- [ ] Subscriber notification sent (if enabled)
- [ ] Make.com workflow logs success
- [ ] No error handling triggered

**Once all items are checked, your workflow is ready for automated daily uploads!**

---

## Troubleshooting

### OAuth & Connection Issues

#### Issue: "Invalid Credentials" Error

**Symptoms:**
- Upload fails with 401 Unauthorized
- Connection test fails
- "Invalid credentials" message in Make.com

**Causes:**
- OAuth token expired
- Wrong Google account linked
- Insufficient OAuth scopes

**Solutions:**
1. Re-authenticate YouTube connection:
   ```
   Make.com → Connections → YouTube → Reconnect
   ```
2. Verify OAuth scopes include:
   ```
   https://www.googleapis.com/auth/youtube.upload
   https://www.googleapis.com/auth/youtube
   ```
3. Check Google Cloud Console:
   - Navigate to [OAuth Consent Screen](https://console.cloud.google.com/apis/credentials/consent)
   - Verify app is not in "Testing" mode (or add your account as test user)
4. Generate new OAuth credentials if needed

#### Issue: "Forbidden" or 403 Error

**Symptoms:**
- Upload blocked with 403 Forbidden
- "Insufficient permissions" error

**Causes:**
- OAuth scopes don't include upload permission
- Channel ownership issue
- API not enabled in Google Cloud

**Solutions:**
1. Enable YouTube Data API v3:
   ```
   Google Cloud Console → APIs & Services → Library → YouTube Data API v3 → Enable
   ```
2. Verify channel ownership:
   - Ensure Google account has upload permissions for the channel
   - Check [YouTube Studio permissions](https://studio.youtube.com/channel/UC.../permissions)
3. Re-authorize with correct scopes

#### Issue: "Quota Exceeded" Error

**Symptoms:**
- Upload fails with quota error
- "Daily quota exceeded" message

**Causes:**
- YouTube API daily quota limit reached (default: 10,000 units)
- Video uploads cost 1,600 units each (~6 uploads/day max)

**Solutions:**
1. Check quota usage:
   ```
   Google Cloud Console → APIs & Services → YouTube Data API v3 → Quotas
   ```
2. Request quota increase:
   - Navigate to [Quota increase request](https://support.google.com/youtube/contact/yt_api_form)
   - Explain use case (automated daily Shorts)
   - Approval typically takes 1-3 business days
3. Optimize upload schedule:
   - Upload once daily instead of multiple times
   - Use unlisted/private uploads for testing

### Upload Failures

#### Issue: Video File Not Found

**Symptoms:**
- "File not found" error
- Upload module can't access video file

**Causes:**
- Download module failed
- File path mapping incorrect
- Temporary file expired

**Solutions:**
1. Verify download module success:
   ```javascript
   // Check download module output
   if (!{{download_module.file}}) {
     throw new Error("Video file not downloaded");
   }
   ```
2. Fix file path mapping:
   ```json
   {
     "videoFile": "{{download_module.video_file}}"
   }
   ```
3. Add retry logic for download failures

#### Issue: "Invalid Request" Error

**Symptoms:**
- Upload fails with 400 Bad Request
- "Invalid metadata" error

**Causes:**
- Title exceeds 100 characters
- Description exceeds 5,000 characters
- Invalid characters in metadata
- Missing required fields

**Solutions:**
1. Validate title length:
   ```javascript
   if (title.length > 100) {
     title = title.substring(0, 97) + "...";
   }
   ```
2. Validate description:
   ```javascript
   if (description.length > 5000) {
     description = description.substring(0, 4997) + "...";
   }
   ```
3. Remove invalid characters:
   ```javascript
   title = title.replace(/[<>]/g, "");
   ```

### Shorts Classification Issues

#### Issue: Video Not Appearing as Short

**Symptoms:**
- Video appears as regular video, not Short
- No "#Shorts" badge visible
- Not in Shorts feed

**Causes:**
- Duration exceeds 60 seconds
- Aspect ratio not 9:16
- Resolution too low
- Processing delay

**Solutions:**
1. Verify video specifications:
   ```bash
   ffprobe -v error -select_streams v:0 \
     -show_entries stream=width,height,duration \
     {{video_file}}
   ```
2. Confirm Runway Gen-3 config:
   ```json
   {
     "duration": 45,
     "ratio": "9:16",
     "resolution": "720p"
   }
   ```
3. Wait 2-4 hours for YouTube processing
4. Add "#Shorts" to title/description for signal boost

#### Issue: Video Appears Horizontally

**Symptoms:**
- Video displays in landscape mode
- Black bars on sides (pillarboxing)

**Causes:**
- Wrong aspect ratio in Runway config
- Video rotated during processing

**Solutions:**
1. Verify Runway aspect ratio:
   ```json
   {
     "ratio": "9:16"
   }
   ```
   (NOT "16:9")
2. Check video metadata before upload
3. Re-generate video with correct ratio

### Metadata Issues

#### Issue: Hashtags Not Clickable

**Symptoms:**
- Hashtags appear as plain text
- Not linkable in description

**Causes:**
- Hashtags in wrong format
- Too many hashtags (>15)
- Hashtags in title only (need to be in description too)

**Solutions:**
1. Format hashtags correctly:
   ```
   #AnimalRescue (correct)
   # AnimalRescue (incorrect - space after #)
   #animal-rescue (incorrect - hyphen not allowed)
   ```
2. Limit to 10-15 hashtags total
3. Place hashtags in description, not just title

#### Issue: Title Truncated on Mobile

**Symptoms:**
- Title cut off on mobile devices
- "..." appears in title

**Causes:**
- Title too long (>60 characters for full mobile display)

**Solutions:**
1. Keep titles concise (50-60 characters ideal):
   ```javascript
   // Optimal title length
   const title = `Animals Rescue - ${story_variant} #Shorts`;
   // Max 60 chars for mobile
   ```
2. Put most important words first
3. Test on mobile before deployment

### Performance Issues

#### Issue: Slow Upload Times

**Symptoms:**
- Upload takes > 5 minutes
- Workflow timeout

**Causes:**
- Large file size
- Slow network connection
- Make.com timeout limits

**Solutions:**
1. Optimize Runway video settings:
   ```json
   {
     "resolution": "720p"
   }
   ```
   (NOT "1080p" unless needed)
2. Increase Make.com timeout:
   ```
   YouTube module → Advanced Settings → Timeout: 600 seconds
   ```
3. Check internet connection speed

#### Issue: Make.com Operations Limit Exceeded

**Symptoms:**
- Workflow stops mid-execution
- "Operations quota exceeded" error

**Causes:**
- Too many polling attempts
- Excessive logging
- Complex workflows

**Solutions:**
1. Optimize polling intervals (see [Runway Config Doc](runway-config.md))
2. Reduce logging operations:
   ```javascript
   // Log only on success/error, not every step
   if (status === "success" || status === "error") {
     logToSheets(data);
   }
   ```
3. Upgrade Make.com plan if needed

### Getting Help

**Make.com Support:**
- Documentation: [help.make.com](https://help.make.com)
- Community Forum: [community.make.com](https://community.make.com)
- Support Tickets: Available for paid plans

**YouTube API Support:**
- API Documentation: [YouTube Data API v3](https://developers.google.com/youtube/v3)
- Issue Tracker: [YouTube API Issue Tracker](https://issuetracker.google.com/issues?q=componentid:187153)
- Stack Overflow: Tag `youtube-api`

**Runway Gen-3 Support:**
- Email: [support@runwayml.com](mailto:support@runwayml.com)
- Documentation: [Runway API Docs](https://docs.runwayml.com)

---

## Summary

This document covered:

✅ **YouTube Module Setup**: Complete configuration for Make.com YouTube upload module  
✅ **OAuth Connection**: Step-by-step authentication and channel selection  
✅ **Shorts Compliance**: Ensuring videos meet YouTube Shorts requirements (< 60s, 9:16, 1080x1920)  
✅ **Metadata Templates**: Title, description, and tag templates with variable mapping  
✅ **Thumbnail Configuration**: Auto-selection and custom upload options  
✅ **Publishing Options**: Immediate and scheduled publishing strategies  
✅ **Testing Checklist**: Comprehensive verification steps for successful uploads  
✅ **Troubleshooting**: Common issues and solutions

**Next Steps:**
1. Review [credentials.md](credentials.md) for OAuth setup details
2. Review [make-workflow-blueprint.md](make-workflow-blueprint.md) for full workflow integration
3. Test your YouTube upload module configuration
4. Run end-to-end workflow test
5. Monitor first automated upload
6. Optimize based on analytics

**Your automated YouTube Shorts upload is now ready for production!** 🎉

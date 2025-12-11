# Credentials Setup Guide

This guide provides step-by-step instructions for setting up Runway Gen-3 and YouTube API credentials for automated video generation and upload workflows.

## Table of Contents
1. [Runway Gen-3 API Setup](#runway-gen-3-api-setup)
2. [YouTube Data API v3 Setup](#youtube-data-api-v3-setup)
3. [OAuth 2.0 Credentials](#oauth-20-credentials)
4. [Obtaining YouTube Refresh Token](#obtaining-youtube-refresh-token)
5. [Storing Credentials in Make](#storing-credentials-in-make)
6. [Environment Variables Template](#environment-variables-template)
7. [Token Refresh Instructions](#token-refresh-instructions)

---

## Runway Gen-3 API Setup

Runway Gen-3 allows you to generate high-quality video content using AI.

### Step 1: Create a Runway Account
1. Visit [Runway](https://runway.com)
2. Click **Sign Up** and create a free account
3. Verify your email address

### Step 2: Access API Dashboard
1. Navigate to [Runway API Dashboard](https://app.runway.ml/settings/api)
2. If prompted, complete any onboarding steps
3. Review the API documentation at [Runway API Docs](https://docs.runwayml.com/docs/api-introduction)

### Step 3: Generate API Key
1. In the API Dashboard, locate the **API Keys** section
2. Click **Create API Key** or **Generate New Key**
3. Give your key a descriptive name (e.g., "YouTube Shorts Automation")
4. Copy the API key immediately (it may only be shown once)
5. Store it securely - you'll need this as `RUNWAY_API_KEY`

**Important Notes:**
- Free tier includes limited credits for video generation
- Monitor your usage at [Runway Usage Dashboard](https://app.runway.ml/usage)
- API rate limits apply - check current limits in the documentation

---

## YouTube Data API v3 Setup

To upload videos to YouTube programmatically, you need to enable the YouTube Data API v3 in Google Cloud Platform.

### Step 1: Create Google Cloud Project
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Sign in with your Google account
3. Click **Select a project** → **New Project**
4. Enter a project name (e.g., "YouTube Shorts Automation")
5. Click **Create**
6. Wait for the project to be created and select it

### Step 2: Enable YouTube Data API v3
1. Navigate to [APIs & Services Library](https://console.cloud.google.com/apis/library)
2. Search for "YouTube Data API v3"
3. Click on **YouTube Data API v3** from the results
4. Click **Enable**
5. Wait for the API to be enabled

### Step 3: Configure OAuth Consent Screen
1. Go to [OAuth Consent Screen](https://console.cloud.google.com/apis/credentials/consent)
2. Select **External** user type (unless you have a Google Workspace account)
3. Click **Create**
4. Fill in the required fields:
   - **App name**: Your application name (e.g., "Shorts Automation Bot")
   - **User support email**: Your email address
   - **Developer contact information**: Your email address
5. Click **Save and Continue**
6. On the **Scopes** page, click **Add or Remove Scopes**
7. Add the following scopes:
   ```
   https://www.googleapis.com/auth/youtube.upload
   https://www.googleapis.com/auth/youtube
   ```
8. Click **Update** → **Save and Continue**
9. On **Test Users**, add your YouTube channel's Google email
10. Click **Save and Continue**

---

## OAuth 2.0 Credentials

### Step 1: Create OAuth 2.0 Client ID
1. Navigate to [Credentials](https://console.cloud.google.com/apis/credentials)
2. Click **Create Credentials** → **OAuth client ID**
3. Select **Application type**: 
   - For local testing: **Desktop app**
   - For web automation: **Web application**
4. Enter a name (e.g., "YouTube Upload Client")
5. If you selected Web application, add authorized redirect URIs:
   ```
   http://localhost:8080
   http://localhost:8080/oauth2callback
   ```
6. Click **Create**
7. A dialog will show your **Client ID** and **Client Secret**
8. Click **Download JSON** to save the credentials file
9. Store these securely - you'll need them as `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`

### Required OAuth Scopes for YouTube Shorts

For uploading YouTube Shorts, you need the following scopes:

```
https://www.googleapis.com/auth/youtube.upload
https://www.googleapis.com/auth/youtube
```

**Scope Permissions:**
- `youtube.upload`: Upload, edit, and delete videos
- `youtube`: Manage your YouTube account (includes read/write access)

---

## Obtaining YouTube Refresh Token

The refresh token allows your automation to upload videos without manual authentication each time.

### Method 1: Using OAuth 2.0 Playground

1. Go to [Google OAuth 2.0 Playground](https://developers.google.com/oauthplayground/)
2. Click the **Settings** gear icon (top right)
3. Check **Use your own OAuth credentials**
4. Enter your **OAuth Client ID** and **OAuth Client Secret**
5. Close the settings panel
6. In the left panel under **Step 1**, scroll to **YouTube Data API v3**
7. Select:
   ```
   https://www.googleapis.com/auth/youtube.upload
   https://www.googleapis.com/auth/youtube
   ```
8. Click **Authorize APIs**
9. Sign in with the Google account linked to your YouTube channel
10. Click **Allow** to grant permissions
11. You'll be redirected back to the playground
12. Click **Exchange authorization code for tokens** (Step 2)
13. Copy the **Refresh token** from the response
14. Store this as `YOUTUBE_REFRESH_TOKEN`

**Important:** The refresh token is long-lived and should be kept secure.

### Method 2: Using a Custom Script

If you prefer a programmatic approach, you can use a Node.js script:

```javascript
const { google } = require('googleapis');
const http = require('http');
const url = require('url');
const open = require('open');

const oauth2Client = new google.auth.OAuth2(
  'YOUR_CLIENT_ID',
  'YOUR_CLIENT_SECRET',
  'http://localhost:8080'
);

const scopes = [
  'https://www.googleapis.com/auth/youtube.upload',
  'https://www.googleapis.com/auth/youtube'
];

const authUrl = oauth2Client.generateAuthUrl({
  access_type: 'offline',
  scope: scopes,
  prompt: 'consent'
});

console.log('Authorize this app by visiting:', authUrl);
open(authUrl);

const server = http.createServer(async (req, res) => {
  if (req.url.indexOf('/oauth2callback') > -1) {
    const qs = new url.URL(req.url, 'http://localhost:8080').searchParams;
    const code = qs.get('code');
    res.end('Authentication successful! Check your console.');
    server.close();
    
    const { tokens } = await oauth2Client.getToken(code);
    console.log('Refresh Token:', tokens.refresh_token);
  }
}).listen(8080);
```

---

## Storing Credentials in Make

Make.com (formerly Integromat) provides a secure Connection vault for storing API credentials.

### Step 1: Create Runway Connection
1. Log into your [Make.com account](https://www.make.com/)
2. Go to **Connections** in the left sidebar
3. Click **Add** → **Custom**
4. Configure the connection:
   - **Connection name**: "Runway Gen-3 API"
   - **Connection Type**: HTTP
   - **Base URL**: `https://api.runwayml.com/v1`
   - Add **Header**:
     - Key: `Authorization`
     - Value: `Bearer YOUR_RUNWAY_API_KEY`
5. Click **Save**

### Step 2: Create YouTube Connection
1. In **Connections**, click **Add**
2. Search for "YouTube"
3. Select **YouTube** from the list
4. Choose connection method:
   - **Option A**: Use the built-in Google OAuth flow (recommended)
     - Click **Sign in with Google**
     - Authorize the requested permissions
   - **Option B**: Use custom OAuth credentials
     - Select **Advanced**
     - Enter your **Client ID** and **Client Secret**
     - Complete the OAuth flow
5. The connection will be saved automatically

### Step 3: Use Connections in Scenarios
1. Create or edit a Make scenario
2. Add a module (e.g., "HTTP" or "YouTube")
3. In the **Connection** dropdown, select your saved connection
4. The credentials will be automatically applied

**Security Best Practices:**
- Never commit credentials to version control
- Use Make's Connection vault for all sensitive data
- Rotate API keys periodically
- Monitor API usage for unusual activity
- Use separate credentials for development and production

---

## Environment Variables Template

For local development or testing, create a `.env` file with the following template:

```bash
# Runway Gen-3 API Configuration
RUNWAY_API_KEY=your_runway_api_key_here

# Google OAuth 2.0 Credentials
GOOGLE_CLIENT_ID=your_client_id_here.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_client_secret_here

# YouTube API Tokens
YOUTUBE_REFRESH_TOKEN=your_refresh_token_here

# Optional: YouTube Channel Configuration
YOUTUBE_CHANNEL_ID=your_channel_id_here

# Optional: Make.com Webhook URLs
MAKE_WEBHOOK_URL=https://hook.make.com/your_webhook_id
```

**Setup Instructions:**
1. Copy this template to a new file named `.env` in your project root
2. Replace all placeholder values with your actual credentials
3. Ensure `.env` is listed in your `.gitignore` file
4. Never commit the `.env` file to version control

---

## Token Refresh Instructions

OAuth 2.0 access tokens expire after a short period (typically 1 hour). Here's how to handle token refresh:

### Automatic Token Refresh

Most OAuth 2.0 libraries handle token refresh automatically when you provide a refresh token.

**Example with Google APIs Node.js Client:**

```javascript
const { google } = require('googleapis');

const oauth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  'http://localhost:8080'
);

// Set the refresh token
oauth2Client.setCredentials({
  refresh_token: process.env.YOUTUBE_REFRESH_TOKEN
});

// The library will automatically refresh the access token when needed
const youtube = google.youtube({
  version: 'v3',
  auth: oauth2Client
});
```

### Manual Token Refresh

If you need to manually refresh the access token:

**Using cURL:**
```bash
curl -d "client_id=YOUR_CLIENT_ID" \
     -d "client_secret=YOUR_CLIENT_SECRET" \
     -d "refresh_token=YOUR_REFRESH_TOKEN" \
     -d "grant_type=refresh_token" \
     https://oauth2.googleapis.com/token
```

**Response:**
```json
{
  "access_token": "new_access_token_here",
  "expires_in": 3599,
  "scope": "https://www.googleapis.com/auth/youtube.upload",
  "token_type": "Bearer"
}
```

### Token Expiration Handling

**Best Practices:**
1. Store the refresh token securely and permanently
2. Access tokens are temporary - don't store them long-term
3. Implement error handling for expired tokens:
   ```javascript
   try {
     await youtube.videos.insert(params);
   } catch (error) {
     if (error.code === 401) {
       // Token expired - refresh and retry
       await oauth2Client.refreshAccessToken();
       await youtube.videos.insert(params);
     }
   }
   ```
4. Monitor token usage and refresh proactively

### Refresh Token Revocation

If you need to revoke access:
1. Go to [Google Account Permissions](https://myaccount.google.com/permissions)
2. Find your application
3. Click **Remove Access**
4. Generate a new refresh token following the steps above

---

## Troubleshooting

### Common Issues

**Runway API Key Not Working:**
- Verify the key is correctly copied (no extra spaces)
- Check if your API credits are depleted
- Ensure you're using the correct API endpoint

**YouTube API Quota Exceeded:**
- Default quota is 10,000 units per day
- Video uploads cost 1,600 units each
- Request a quota increase at [Google Cloud Console](https://console.cloud.google.com/apis/api/youtube.googleapis.com/quotas)

**OAuth Token Invalid:**
- Ensure you requested `access_type: 'offline'` when generating the token
- Check that the scope includes both `youtube` and `youtube.upload`
- Generate a new refresh token if the old one was revoked

**Make.com Connection Failed:**
- Verify credentials are correctly entered
- Check that the OAuth consent screen is properly configured
- Ensure test users are added if the app is in testing mode

### Getting Help

- **Runway Support**: [support@runwayml.com](mailto:support@runwayml.com)
- **YouTube API Documentation**: [Google Developers](https://developers.google.com/youtube/v3)
- **Make.com Support**: [help.make.com](https://help.make.com)
- **OAuth 2.0 Guide**: [OAuth 2.0 Documentation](https://oauth.net/2/)

---

## Security Reminders

✅ **DO:**
- Store credentials in secure vaults (Make.com Connections, environment variables)
- Use `.gitignore` to exclude credential files
- Rotate API keys periodically
- Monitor API usage dashboards
- Use separate credentials for different environments

❌ **DON'T:**
- Commit credentials to version control
- Share API keys in public channels
- Use production credentials in development
- Store refresh tokens in client-side code
- Hardcode credentials in your application

---

## Next Steps

Once you have all credentials configured:

1. Test Runway API connection with a simple generation request
2. Verify YouTube API access by listing your channel information
3. Create a Make.com scenario to automate the workflow
4. Test end-to-end: Generate video → Upload to YouTube
5. Monitor usage and optimize as needed

For integration examples and workflow templates, check the `examples/` directory in this repository.

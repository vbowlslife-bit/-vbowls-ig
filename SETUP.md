# Instagram AutoDM Dashboard - Real Setup Guide

This guide walks you through configuring the dashboard to work with real Instagram/Meta APIs instead of mock mode.

## Prerequisites

- Meta Developer Account
- Instagram Business or Creator account (connected to a Facebook Page)
- ngrok or Cloudflare Tunnel for local webhook testing
- Domain with HTTPS for production webhooks

## 1. Create a Meta Developer App

1. Go to [Meta for Developers](https://developers.facebook.com/)
2. Click **My Apps** → **Create App**
3. Select **Business** as the app type
4. Fill in:
   - **App Name**: Your app name (e.g., "My Instagram AutoDM")
   - **App Contact Email**: Your email
   - **Business Account**: Select or create a Business Manager account
5. Click **Create App**

## 2. Add Instagram Products

In your app dashboard, add these products:

### Instagram Basic Display
1. Go to **Products** → **Instagram Basic Display** → **Set Up**
2. This is for basic profile access (not needed for messaging/comments)

### Instagram Graph API (Required)
1. Go to **Products** → **Instagram** → **Set Up**
2. This enables:
   - `instagram_business_basic` - Basic profile info
   - `instagram_business_manage_messages` - Send/read DMs
   - `instagram_business_manage_comments` - Read/reply to comments
   - `instagram_business_content_publish` - Post content (optional)
   - `pages_show_list` - List Facebook Pages
   - `pages_read_engagement` - Read page engagement

## 3. Configure Instagram Product

1. In **Instagram** → **Basic Display**, add:
   - **Valid OAuth Redirect URIs**: `https://yourdomain.com/api/auth/instagram/callback`
   - **Deauthorize Callback URL**: `https://yourdomain.com/api/auth/instagram/deauth` (optional)
   - **Data Deletion Request URL**: `https://yourdomain.com/api/auth/instagram/delete` (optional)

2. In **Instagram** → **App Review**, you'll need to submit for review to go live, but for development:
   - Add yourself as an **Instagram Tester** (see step 4)

## 4. Add Instagram Testers

1. Go to **App Settings** → **Roles** → **Instagram Testers**
2. Click **Add Instagram Testers**
3. Enter the Instagram username of the account you want to test with
4. The tester must accept the invitation in their Instagram app:
   - Instagram app → Settings → Apps and Websites → Tester Invites

## 5. Get App Credentials

1. Go to **App Settings** → **Basic**
2. Copy:
   - **App ID** → `META_APP_ID`
   - **App Secret** → `META_APP_SECRET`

## 6. Generate a Long-Lived Access Token

### Option A: Using the Dashboard (Recommended)

1. Set `MOCK_INSTAGRAM=false` in `.env`
3. Run `npm run dev`
4. Go to **Settings** page in the dashboard
5. Click **"Connect Instagram"**
6. Complete OAuth flow
7. The dashboard will exchange the short-lived token for a long-lived one (60 days) and store it encrypted

### Option B: Manual Token Generation

1. Use the [Graph API Explorer](https://developers.facebook.com/tools/explorer/)
2. Select your app
3. Get User Access Token with scopes:
   - `instagram_business_basic`
   - `instagram_business_manage_messages`
   - `instagram_business_manage_comments`
   - `pages_show_list`
   - `pages_read_engagement`
4. Exchange for long-lived token:
   ```bash
   curl -X GET "https://graph.facebook.com/v20.0/oauth/access_token?grant_type=fb_exchange_token&client_id={APP_ID}&client_secret={APP_SECRET}&fb_exchange_token={SHORT_LIVED_TOKEN}"
   ```
5. Get Instagram Business Account ID:
   ```bash
   curl -X GET "https://graph.facebook.com/v20.0/me/accounts?fields=instagram_business_account&access_token={LONG_LIVED_TOKEN}"
   ```
6. Add to `.env`:
   ```
   INSTAGRAM_ACCESS_TOKEN=your_long_lived_token
   INSTAGRAM_BUSINESS_ACCOUNT_ID=your_ig_business_account_id
   ```

## 7. Configure Webhooks

### Local Development with ngrok

1. Install ngrok: `npm install -g ngrok` or download from [ngrok.com](https://ngrok.com/)
2. Start tunnel:
   ```bash
   ngrok http 3000
   ```
3. Copy the HTTPS URL (e.g., `https://abc123.ngrok-free.app`)

### Set Webhook URL

1. In Meta App Dashboard → **Instagram** → **Webhooks**
2. Click **Edit Subscription**
3. **Callback URL**: `https://your-ngrok-url.ngrok-free.app/api/webhooks/instagram`
4. **Verify Token**: Set a random string (e.g., `my-webhook-verify-token-123`) → Add to `.env` as `WEBHOOK_VERIFY_TOKEN`
5. **Fields to subscribe**:
   - `comments` - New comments on posts
   - `messages` - New DMs
   - `message_reactions` (optional)
   - `standalone_messaging_postbacks` (optional)
6. Click **Verify and Save**

### Add to .env

```env
WEBHOOK_VERIFY_TOKEN=my-webhook-verify-token-123
WEBHOOK_URL=https://your-ngrok-url.ngrok-free.app/api/webhooks/instagram
NEXT_PUBLIC_APP_URL=https://your-ngrok-url.ngrok-free.app
```

## 8. Test the Integration

### Test Webhook Delivery

1. In Meta App Dashboard → **Instagram** → **Webhooks**
2. Click **Send Test** for `comments` and `messages`
3. Check your dashboard **Logs** page - you should see the test events

### Test with a Real Account

1. Use a **second Instagram account** (not the business account)
2. Comment on one of your business account's posts
3. Send a DM to your business account
4. Check:
   - **Inbox** - Should show the conversation
   - **Logs** - Should show webhook events
   - **Dashboard** - Stats should update

### Test Automations

1. Go to **Automations** → **New Automation**
2. Create a `COMMENT_ON_POST` automation for a specific post
3. Use the **Simulate Panel** on Dashboard (only works in mock mode)
4. Or test live by commenting with the trigger keyword

## 9. Production Deployment

### Environment Variables

```env
# Required
DATABASE_URL="file:./prod.db"
ADMIN_PASSWORD="your-secure-password"  # Will be bcrypt hashed on first login
SESSION_PASSWORD="32+ character random string"
MOCK_INSTAGRAM="false"

# Meta App
META_APP_ID="your-app-id"
META_APP_SECRET="your-app-secret"
INSTAGRAM_REDIRECT_URI="https://yourdomain.com/api/auth/instagram/callback"

# Webhooks
WEBHOOK_VERIFY_TOKEN="your-verify-token"
WEBHOOK_URL="https://yourdomain.com/api/webhooks/instagram"
NEXT_PUBLIC_APP_URL="https://yourdomain.com"

# Optional: Pre-configured tokens (if not using OAuth)
INSTAGRAM_ACCESS_TOKEN="long-lived-token"
INSTAGRAM_BUSINESS_ACCOUNT_ID="123456789"
```

### Database

```bash
# Run migrations
npx prisma migrate deploy

# Or use the setup script
npm run setup
```

### Build & Start

```bash
npm run build
npm start
```

### Process Manager (PM2)

```bash
npm install -g pm2
pm2 start npm --name "insta-dashboard" -- start
pm2 save
pm2 startup
```

### Reverse Proxy (Nginx)

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 10. Token Refresh

The dashboard automatically refreshes tokens before expiry (at ~50 days). The refresh happens:
- On app startup
- When making API calls with a near-expiry token
- Via a daily cron (if you set one up)

To manually refresh:
1. Go to **Settings** page
2. Click **"Refresh Token"** (if implemented)
3. Or re-run OAuth flow

## Troubleshooting

### "Invalid OAuth redirect URI"
- Ensure `INSTAGRAM_REDIRECT_URI` matches exactly in Meta App settings and `.env`
- Include trailing slash if Meta requires it

### "Webhook verification failed"
- Check `WEBHOOK_VERIFY_TOKEN` matches in Meta and `.env`
- Ensure ngrok URL is accessible from internet
- Check server logs for incoming requests

### "Insufficient permissions"
- Ensure Instagram Tester is added and accepted
- Submit for App Review for production access
- Check token has required scopes

### "Rate limited"
- Dashboard has built-in throttling (20 requests/second)
- Add delays in automation settings
- Implement exponential backoff (already in queue)

### "Token expired"
- Long-lived tokens last 60 days
- Dashboard auto-refreshes at 50 days
- Re-run OAuth if auto-refresh fails

## Security Notes

- Never commit `.env` to git
- Use strong `SESSION_PASSWORD` (32+ random chars)
- Rotate `WEBHOOK_VERIFY_TOKEN` periodically
- Monitor logs for suspicious activity
- Use HTTPS in production
- Keep dependencies updated

## Support

- Meta Graph API Docs: https://developers.facebook.com/docs/instagram-api
- Webhook Reference: https://developers.facebook.com/docs/instagram-api/webhooks
- App Review Guide: https://developers.facebook.com/docs/app-review
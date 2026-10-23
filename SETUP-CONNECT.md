# Instagram Connect Setup (6 Steps)

## 1. Create Meta App
- Go to https://developers.facebook.com/apps/
- Click **Create App** → **Business** → Name it → Create
- Copy **App ID** → paste in `.env` as `INSTAGRAM_APP_ID`
- Click **Show** next to App Secret → copy → paste in `.env` as `INSTAGRAM_APP_SECRET`

## 2. Add Instagram Product
- In your app dashboard: **Products** → **Instagram** → **Set Up**
- This enables: `instagram_business_basic`, `instagram_business_manage_messages`, `instagram_business_manage_comments`, `pages_show_list`, `pages_read_engagement`

## 3. Configure OAuth Redirect URI
- In left sidebar: **Products** → **Instagram** → **Basic Display**
- Add **Valid OAuth Redirect URI**: `https://YOUR_NGROK_URL.ngrok-free.app/api/auth/instagram/callback`
- Save

## 4. Add Yourself as Instagram Tester (Required for Dev Mode)
- **App Settings** → **Roles** → **Instagram Testers** → **Add Instagram Testers**
- Enter your Instagram username → Send invite
- Open Instagram app → Settings → Apps & Websites → Tester Invites → **Accept**

## 5. Start ngrok & Set Webhook
```bash
ngrok http 3000
```
- Copy the HTTPS URL (e.g., `https://abc123.ngrok-free.app`)
- In `.env`:
  ```
  APP_BASE_URL=https://abc123.ngrok-free.app
  NEXT_PUBLIC_APP_URL=https://abc123.ngrok-free.app
  WEBHOOK_URL=https://abc123.ngrok-free.app/api/webhooks/instagram
  WEBHOOK_VERIFY_TOKEN=your-64-char-hex-token-from-openssl-rand-hex-32
  INSTAGRAM_REDIRECT_URI=https://abc123.ngrok-free.app/api/auth/instagram/callback
  MOCK_INSTAGRAM=false
  ```

## 6. Subscribe Webhook in Meta
- **Products** → **Instagram** → **Webhooks** → **Edit Subscription**
- **Callback URL**: `https://abc123.ngrok-free.app/api/webhooks/instagram`
- **Verify Token**: same as `WEBHOOK_VERIFY_TOKEN` in `.env`
- Check **comments** and **messages** → **Verify and Save**

---

## Then Connect
1. Run `npm run dev` (or `npm run start` for production)
2. Go to Settings page → click **"Connect Instagram (OAuth)"**
3. Log in on Instagram's official page → approve permissions
4. Redirected back to Settings → shows "Connected" + "Webhook Subscribed"

## Token Refresh
- Runs automatically via cron (checks weekly, refreshes if < 7 days left)
- If refresh fails: Settings shows "Reconnect" button → click to re-authenticate
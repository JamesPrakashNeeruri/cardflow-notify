# CardFlow Notify

Sends you a real push notification a few days before each credit card bill is due —
no ongoing cost, runs entirely on free tiers.

## How it works
- A tiny Next.js app + one background job (Vercel Cron) running once a day.
- Card due-dates and your push subscription are stored in a free Upstash Redis database.
- The cron job checks all cards daily and sends a push notification via the Web Push
  standard when a reminder is due.

This is built for **single personal use** — there's no login system, so anyone with
the link could see/edit the same card list. Don't share the deployed URL publicly.

## ⚠️ iPhone-specific requirement
iOS Safari only allows push notifications from a website if you **Add it to your Home
Screen first** (iOS 16.4+):
1. Open the deployed site in Safari.
2. Tap the Share icon → **Add to Home Screen**.
3. Open the app from the Home Screen icon (not from Safari directly).
4. Then tap **Enable Notifications** inside the app.

Skipping this step means notifications will silently never arrive on iPhone — this is
an Apple platform restriction, not a bug in the app.

Android (Chrome) and desktop browsers don't need this step — enabling notifications
directly in the browser works.

## One-time setup (all free)

### 1. Create a free Upstash Redis database
1. Go to https://console.upstash.com and sign up (free).
2. Create a new Redis database (any region close to you, e.g. Mumbai).
3. On the database's page, copy the **REST URL** and **REST TOKEN** — you'll need
   both in step 3.

### 2. Push this project to GitHub
1. Create a free GitHub account if you don't have one: https://github.com
2. Create a new empty repository (e.g. `cardflow-notify`).
3. Upload all the files in this folder to that repository (via the GitHub web
   "Add file → Upload files" is easiest if you're not using git on the command line).

### 3. Deploy on Vercel
1. Go to https://vercel.com/new and sign in (free).
2. Choose **Import Git Repository** and select the `cardflow-notify` repo you just
   created. (This project needs a real build step, so it must be imported from
   GitHub — not dragged in as a ZIP.)
3. Before clicking Deploy, expand **Environment Variables** and add these four:

   | Name | Value |
   |---|---|
   | `UPSTASH_REDIS_REST_URL` | *(from step 1)* |
   | `UPSTASH_REDIS_REST_TOKEN` | *(from step 1)* |
   | `VAPID_PUBLIC_KEY` | `BIOHfLB1YRywXxfOFgageastLaYpjdgmNNuFZPrwGheMYtttoBKYGYub7wgxH200ir-abMuIcBBgPd7QTqfI-kk` |
   | `VAPID_PRIVATE_KEY` | `cgH-xRJHxH7DWZ6Mo2lBkWFWDkPxw6jPHqLqoPfoxYw` |

   (These VAPID keys were generated specifically for this project — you can use them
   as-is, or generate your own with `npx web-push generate-vapid-keys` if you prefer.)

4. Click **Deploy**.

### 4. Use it
1. Open the deployed URL. On iPhone, follow the "Add to Home Screen" step above first.
2. Tap **Enable Notifications** and allow the permission prompt.
3. Tap **Send Test Notification** — you should get a notification within a few
   seconds. If not, see Troubleshooting below.
4. Add your cards: name, due day of the month, and how many days before you want
   the reminder (e.g. 3 days before).

That's it — the Vercel Cron job (configured in `vercel.json`) runs every day at
9:00 AM IST and sends a notification for any card whose reminder or due date matches
today.

## Troubleshooting
- **No test notification arrives**: check Vercel's function logs (Project →
  Deployments → your deployment → Functions) for `/api/test-push` — it will show
  the exact error (usually a wrong Upstash URL/token or a stale push subscription).
- **iPhone never gets notifications**: confirm you opened the app from the Home
  Screen icon, not Safari, before tapping Enable Notifications.
- **Cron isn't firing**: Vercel's free Hobby plan supports daily cron jobs; confirm
  `vercel.json` deployed correctly and check Project → Cron Jobs in the dashboard for
  its run history.
- **Want a different reminder time**: edit the `schedule` in `vercel.json`
  (currently `"30 3 * * *"`, which is 3:30 AM UTC = 9:00 AM IST) and redeploy.

## Local development (optional)
```
npm install
npm run dev
```
Requires the same four environment variables in a `.env.local` file. Push notifications
generally need HTTPS, so local testing is limited — the real test is after deploying.

# Job Intake — Setup Guide

This package is **one web app** (`index.html`) that works as your **landing page + questionnaire app**. It's built as an installable PWA so it can also be published to the **Google Play Store** without writing native code. Everything below is what's left for you to configure — the form logic, all 8 required sections, and the assistant are already built and working.

## What's inside
- `index.html` — the whole site/app (landing page + multi-step form + admin setup)
- `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png` — make it installable (PWA)
- This README

## 1. Set up your business details
Open the app and tap **Admin** in the top nav (this should be hidden from customers — see "Locking the admin page" below). Fill in:
- Business name, tagline, closing thank-you message
- Notification email (where every request should land)
- Bank details (account number, account name, bank name)
- Crypto details (wallet address, chain, Bybit email)
- EmailJS keys (step 2 below)

Save — this is stored on that device/browser. For a shared setup across every visitor (recommended for production), instead edit the `DEFAULT_CONFIG` object near the top of the `<script>` in `index.html` directly, so every visitor sees the same, correct info without needing local setup.

## 2. Connect email notifications (so every request drops in your inbox)
The app uses **EmailJS** (free tier available) to send an email the moment a customer submits a request — no backend server required.
1. Create a free account at emailjs.com
2. Add your email as an **Email Service** (Gmail, Outlook, etc.)
3. Create an **Email Template** with a subject and a `{{message}}` variable in the body
4. Copy your **Public Key**, **Service ID**, and **Template ID** into the Admin screen (or `DEFAULT_CONFIG.emailjs` in the code)
5. Submit a test request from the app — you should get an email within seconds

Until this is configured, the form still works end-to-end and shows the "thank you" screen, but it won't email you — the browser console will note that EmailJS isn't set up yet.

**Optional — customer confirmation email:** duplicate the EmailJS template, add a `to_email: custEmail` version, and add a second `emailjs.send(...)` call next to the existing one in `index.html` (search for `submitBtn.onclick`).

## 3. Locking the admin page
Right now "Admin" is a visible link so you can find it easily. Before handing the link to customers, either:
- Remove the `#navAdmin` link from `index.html`, and only reach the admin screen by typing a hidden path you choose, or
- Add a simple passcode prompt at the top of the `navAdmin` click handler (a `prompt()` checked against a code you set is enough for a small business; for stronger protection put the admin config on a server instead of in the browser).

## 4. Publish the site
Upload the 5 files to any static host — Netlify, Vercel, GitHub Pages, or your own hosting — as-is, no build step needed. Use **https** (required for installability and for EmailJS).

## 5. Publish to the Google Play Store (no native coding required)
Once your site is live on https:
1. Go to **pwabuilder.com**, enter your site's URL, and let it validate the manifest/service worker (already included).
2. Choose **Android** → it generates a Play Store–ready `.aab` package (this wraps your PWA in a "Trusted Web Activity" — a real Android app shell around your site).
3. Create a Google Play Developer account (one-time $25 fee), then upload the generated package through the Play Console, fill in your store listing (screenshots, description, privacy policy link), and submit for review.
4. Any update you make to `index.html` on your host automatically updates the app — no republishing needed for content changes.

## Security notes (please read before going live)
- **Email notifications** travel through EmailJS's infrastructure to your inbox — this is standard practice for no-backend apps, but it is not end-to-end encrypted in the cryptographic sense. If a customer will type highly sensitive data (e.g. banking passwords — they never should), don't collect it here.
- The app adds a SHA-256 **fingerprint** of each submission for tamper-evidence, but true end-to-end encryption (where only your business can decrypt, even EmailJS can't read it) requires a small backend that holds a private key. If that level of security matters for your use case, the next step up is a lightweight serverless function (e.g. Cloudflare Worker) that encrypts with your public key before forwarding — happy to help build that if needed.
- Business config saved via the in-app Admin screen lives in that browser's `localStorage` only. For consistent, secure business info across all customer devices, hard-code it into `DEFAULT_CONFIG` in `index.html` before publishing, rather than relying on each device's local admin setup.

## 6. The AI assistant (Category 1)
The "Let the assistant suggest" flow currently runs on a **built-in rule-based engine** (keyword + budget + timeline matching) — it works immediately, needs no API key, and costs nothing to run. If you'd like it upgraded to a true LLM-powered assistant (e.g. Claude) for more nuanced suggestions, that requires routing requests through a small server-side proxy that holds your API key (never put an AI API key directly in the site's code — it would be visible to anyone). I can build that proxy for you on request.

## Customizing the questionnaire
All fields for Logo, Flyer, Branding, Rebrand/Corrections, and Training live in the `FORM_DEFS` object in `index.html` — add, remove, or edit fields there; the form re-renders automatically from that definition.

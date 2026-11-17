# Growlane AI

This repo holds two things:

| Folder | What it is |
|---|---|
| repo root (`*.html`, `assets/`) | Growlane AI marketing website (details below) |
| `loyvexa/` | **Loyvexa**: loyalty hub (points, schemes, payouts) + AI Channel Manager. Node + Postgres |
| `korvo-backend/` | **Korvo** backend: orders, approvals, Tally, Loyvexa sync. Node + MongoDB |
| `korvo-app/` | **Korvo** mobile app (Expo / React Native): My Rewards for dealers, Loyalty & AI console card for admins |

## Bundle B: AI Channel Manager + Loyvexa × Korvo

The three platform folders are the Bundle B code, copied on 17 Nov 2026 from the open PR branches (not yet merged in their own repos):

| Folder | Source repo | Branch | Commit |
|---|---|---|---|
| `loyvexa/` | santwayne/loyvexa | `feature/ai-channel-manager` (PR #5) | `f5cb176` |
| `korvo-backend/` | Wayneesolutions/royalSoftwareBackend | `feature/loyvexa-bundle` (PR #3) | `700788c` |
| `korvo-app/` | Wayneesolutions/royalappfrontend | `feature/loyvexa-bundle` (PR #1) | `11cda65` |

How they connect: Loyvexa is the hub. Korvo pushes order status, dealers, products and outstanding to Loyvexa through a retrying outbox (`/api/connect/*`). Loyvexa posts WhatsApp/AI reorders to Korvo (`/api/v1/orders/external`) where they land in `pending_approval`. Cross-server sign-ins use 60-second single-use tokens.

### Deploy order

1. Pick one shared secret (40+ random characters): `HUB_SECRET` on Korvo, `HUB_SECRET_KORVO` on Loyvexa.
2. `korvo-backend/`: deploy. The outbox worker starts with the server (`LOYVEXA_WORKER=off` turns it off). No migration needed.
3. `loyvexa/`: set `KORVO_BASE_URL` (and optionally `ANTHROPIC_API_KEY`, `AI_MODEL`), deploy, run `npm run migrate` so `004_ai_manager.sql` applies. See `loyvexa/.env.example`.
4. Loyvexa console → Connectors → Connect Korvo → enter the Korvo company code.
5. Get WhatsApp templates approved in Meta: `ai.reorder`, `ai.slab`, `ai.mechanic`, `ai.payment`, `ai.brief`, `ai.report`.
6. Loyvexa → AI Manager → Settings: add the owner's number to brief numbers, keep fraud and scheme on approval, turn the AI on.
7. `korvo-app/`: ship a new Expo build (`eas build`).

### Known limits

- Korvo was tested only with stubbed models; run it against a real MongoDB before going live.
- Points are earned on "delivered" by default. If orders aren't marked delivered, set `earnOn` to `approved` in Korvo's Loyvexa settings.
- Token replay protection is in memory; move it to Redis/DB before running more than one instance.
- No Bundle B billing (pricing is "quote on demo").

---

# Growlane AI — Website

Multi-page marketing website for **Growlane AI**. Plain HTML, CSS and JavaScript — no build step, no dependencies. Upload the folder to any static host (Vercel, Netlify, cPanel, S3, GitHub Pages) and it works.

## Pages

| Page | File |
|---|---|
| Home | `index.html` |
| Services | `services.html` |
| How It Works | `how-it-works.html` |
| Industries | `industries.html` |
| Pricing (+ FAQ) | `pricing.html` |
| About | `about.html` |
| Careers | `careers.html` |
| Contact | `contact.html` |
| Book a Demo | `demo.html` |
| Privacy Policy | `privacy.html` |
| Terms of Service | `terms.html` |
| 404 | `404.html` |

## Structure

```
assets/
  css/style.css        all styles
  js/main.js           menu, FAQ, pricing toggle, tabs, forms
  img/growlane-logo.png  logo (transparent PNG)
  img/favicon.png, apple-touch-icon.png
```

## Run locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Connecting email (to do)

Forms (contact, demo, careers, newsletter) validate and show a success message, but don't send anywhere yet. When the email/form service is ready, open `assets/js/main.js` and fill in:

```js
const SITE_CONFIG = {
  FORM_ENDPOINT: "https://formspree.io/f/xxxx", // where form data is POSTed as JSON
  CONTACT_EMAIL: "hello@yourdomain.com",        // shown on the Contact page
};
```

## Editing content

- Prices are in `pricing.html` (`data-monthly` / `data-yearly` attributes on each price).
- Open roles are in `careers.html`.

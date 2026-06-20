# Deployment Guide

Two paths: **fast (15 min)** for prototype, **production (1 day)** when ready to take payments.

---

## Path 1: Prototype deploy (free, 15 min)

Goal: Get all 5 pages live at `your-project.pages.dev` so you can share the link.

### Step 1 — Push to GitHub

```bash
cd C:\Users\pc
git init
git add .
git commit -m "Initial commit: Pulse prototype"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/pulse-monitor.git
git push -u origin main
```

If you don't have GitHub CLI installed, you can also create the repo at [github.com/new](https://github.com/new) and drag-drop the files in via the web UI.

### Step 2 — Connect to Cloudflare Pages

1. Sign in to [dash.cloudflare.com](https://dash.cloudflare.com) (free account).
2. Left sidebar: **Workers & Pages** → **Create application** → **Pages** tab → **Connect to Git**.
3. Select your `pulse-monitor` repo.
4. **Build settings:**
   - Framework preset: **None**
   - Build command: *(leave empty)*
   - Build output directory: `/`
5. Click **Save and Deploy**.

In ~30 seconds you'll see ✅ deployed. Your site is live at:
`https://pulse-monitor.pages.dev/landing.html`

The root URL auto-redirects to `/landing.html` via `_redirects`.

### Step 3 — Custom domain (optional, ~$12/yr)

1. **Buy a domain.** Cloudflare Registrar is at-cost with no markup. Recommended names: `pulse-monitor.com`, `getpulse.app`, `pulseapp.io`.
2. In Cloudflare: **Workers & Pages → your project → Custom domains → Set up a custom domain**.
3. Enter `pulse-monitor.com` (or whatever you bought). Cloudflare auto-configures DNS.

SSL/HTTPS is automatic and free.

**Total cost: $10–15/yr. Total time: 15 minutes.**

---

## Path 2: Add payments (Stripe, ~2 hours)

### Step 1 — Set up Stripe

1. Sign up at [stripe.com](https://stripe.com).
2. Complete identity verification (driver's license + bank account link).
3. **Stay in Test mode** while developing.

### Step 2 — Create products

In Stripe Dashboard → **Products** → **Add product**:

| Product | Price | Billing |
|---|---|---|
| Pulse Pro | $4.99 USD | Recurring, monthly |
| Pulse Pro (Annual) | $39 USD | Recurring, yearly |
| Pulse Team | $12 USD | Recurring, monthly per seat |

Copy each **Price ID** (`price_1ABC...`) — you'll need these.

### Step 3 — Get your publishable key

In Stripe Dashboard → **Developers → API keys**:

- Publishable key: `pk_test_51ABC...`
- Secret key: `sk_test_51ABC...` (⚠️ never put this in client code)

### Step 4 — Configure `upgrade.html`

Open `C:\Users\pc\upgrade.html`, find this near the top of the `<script>`:

```js
const STRIPE_CONFIG = {
  publishableKey: 'pk_test_REPLACE_ME',
  proPriceId:    'price_REPLACE_ME',
  teamPriceId:   'price_REPLACE_ME',
  ...
};
```

Replace with your real values.

### Step 5 — Deploy webhook handler

Stripe needs to tell your app when a user subscribes/cancels. This requires a backend. See `webhook-example/` for a Cloudflare Worker starter.

Deploy it:

```bash
cd webhook-example
npm install
npx wrangler deploy
```

In Stripe Dashboard → **Developers → Webhooks → Add endpoint**:

- URL: `https://pulse-monitor.com/api/stripe-webhook`
- Events to send: `customer.subscription.created`, `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.payment_failed`

Copy the **Webhook signing secret** (`whsec_...`) and add it to your Worker's environment variables.

### Step 6 — Test

Use Stripe's test card: `4242 4242 4242 4242`, any future expiry, any CVC.

Go through the full flow on your deployed site. The webhook should fire, your Worker should log the event, and (after you wire up a database) your user's Pro status should update.

---

## Path 3: Add auth (Clerk, ~1 hour)

When you're ready to gate Pro features behind actual login:

1. Sign up at [clerk.com](https://clerk.com) (free for first 5,000 users).
2. Create an application.
3. Add Clerk's script to each page:

```html
<script async
  src="https://cdn.jsdelivr.net/npm/@clerk/clerk-js@latest/dist/clerk.browser.js"
  data-clerk-publishable-key="pk_test_REPLACE_ME">
</script>
```

4. Replace placeholder buttons with Clerk's `<SignInButton>` / `<UserButton />`.

---

## Troubleshooting

### "My pages don't load, just a directory listing"

Make sure `_redirects` is in the repo root, not in a subfolder. Cloudflare Pages reads it from the build output directory.

### "Stripe Checkout button does nothing"

1. Open browser DevTools → Console. Look for errors.
2. Most common: you forgot to replace `pk_test_REPLACE_ME` with your real key.
3. Second most common: the price ID is wrong (should start with `price_`, not `prod_`).

### "Build fails on Cloudflare"

This is a static site — there should be no build. If you see a build error, make sure:
- **Build command** is empty (not `npm run build`).
- **Build output directory** is `/`.

### "Webhook signature verification fails"

The `Stripe-Signature` header only works if your Worker uses the **raw** request body, not parsed JSON. See the comment in `webhook-example/src/index.js`.

---

## Cost summary

| Item | Free tier | Paid |
|---|---|---|
| Cloudflare Pages hosting | ✅ Free forever | — |
| Custom domain | — | $10–15/yr |
| Stripe account | ✅ Free to set up | 2.9% + 30¢ per transaction |
| Stripe Tax (auto) | ✅ Free | — |
| Cloudflare Worker (webhooks) | ✅ 100k req/day free | $0.50/million after |
| Supabase database (auth + data) | ✅ 500 MB free | $25/mo for Pro |
| Groq AI inference | ✅ 14.4k req/day free | Pay-as-you-go after |

**Realistic monthly cost to run Pulse as a paid SaaS with 100 paying users**: **$25–50/mo total**, generating **$500/mo revenue**. Healthy margins.

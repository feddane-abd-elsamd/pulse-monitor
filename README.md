# Pulse — System Monitor SaaS Prototype

A browser-based, SaaS-style system monitor showing real-time PC stats (CPU, RAM, GPU, disk, network, battery, temperature) with smart tips, AI-style insights, mobile companion preview, and team/fleet dashboard. Built as a frontend-only prototype ready to be hosted on any static host (Cloudflare Pages, GitHub Pages, Vercel, Netlify).

> **Status:** Interactive prototype. All metrics are simulated. AI insights are rule-based templates. No backend required to demo.

---

## ✨ What's included

| Page | Path | Purpose |
|---|---|---|
| Landing | `landing.html` | Marketing site: hero, features, pricing, FAQ |
| Dashboard | `index.html` | Free-tier real-time monitor |
| Pro Insights | `pro.html` | Pro-tier: 30-day timeline scrubber, AI insights, alert builder |
| Mobile | `mobile.html` | Phone-shaped companion app preview |
| Team | `team.html` | Fleet view: 8 machines, filters, drill-down |
| Upgrade | `upgrade.html` | Stripe Checkout placeholder (Pro / Team tiers) |

Every page is a single self-contained HTML file with inline CSS and JS — no build step, no dependencies. Drop them on any static host and they work.

---

## 🚀 Quick start (local)

```bash
# Option A: Python
python -m http.server 8000
# open http://localhost:8000/landing.html

# Option B: Node
npx serve
# open the URL it prints

# Option C: just open index.html directly in a browser
```

That's it. No install, no build.

---

## 🌐 Deploy to Cloudflare Pages (recommended, free)

1. Push this repo to GitHub.
2. Sign up at [cloudflare.com](https://cloudflare.com) (free tier).
3. In Cloudflare dashboard: **Workers & Pages → Create application → Pages → Connect to Git**.
4. Select your repo.
5. Build settings:
   - **Build command:** *(leave empty)*
   - **Build output directory:** `/`
6. Click **Save and Deploy**. Your site is live in ~30 seconds at `your-project.pages.dev`.

### Custom domain

1. Buy a domain (Cloudflare Registrar is at-cost with no markup).
2. In your Cloudflare Pages project: **Custom domains → Set up a custom domain**.
3. Cloudflare auto-configures DNS and HTTPS.

**Total cost: ~$10–15/yr for the domain. Hosting is free.**

---

## 💳 Adding real payments (Stripe)

This prototype includes a placeholder `upgrade.html` with all the wiring for Stripe Checkout. To make it real:

### 1. Set up Stripe
- Create account at [stripe.com](https://stripe.com) (test mode is free).
- Create two recurring products:
  - **Pulse Pro** — $4.99/month and $39/year
  - **Pulse Team** — $12/seat/month
- Copy your **publishable key** (`pk_test_...`) and the **price IDs** (`price_...`).

### 2. Wire up the frontend
Open `upgrade.html`, find the `STRIPE_CONFIG` object near the top of the `<script>` tag, and replace the placeholder values:

```js
const STRIPE_CONFIG = {
  publishableKey: 'pk_test_YOUR_KEY_HERE',
  proPriceId:    'price_YOUR_PRO_PRICE_ID',
  teamPriceId:   'price_YOUR_TEAM_PRICE_ID',
  successUrl:    'https://pulse-monitor.com/success',
  cancelUrl:     'https://pulse-monitor.com/upgrade',
};
```

That's it. The "Subscribe" buttons will now redirect to real Stripe Checkout.

### 3. Webhooks (to actually grant Pro access)

You'll need a small backend to receive Stripe webhooks and update user subscription status. Recommended:

- **Cloudflare Workers** (free tier: 100k requests/day) — see `webhook-example/` in this repo for a starter.
- **Vercel Functions** (free tier: 100GB-hours) — also fine.

The webhook should:
1. Verify the Stripe signature
2. On `customer.subscription.created` → mark user as Pro
3. On `customer.subscription.deleted` → revoke Pro access
4. On `invoice.payment_failed` → email warning

A starter implementation is in `webhook-example/`.

---

## 🧠 Adding real AI insights (optional)

The Pro page currently uses rule-based templates to generate "AI insights." To make them real:

1. Sign up at [groq.com](https://groq.com) (free tier: 14,400 requests/day).
2. Get an API key.
3. Deploy a serverless function `/api/insights` that:
   - Reads the last hour of metrics from your database
   - Sends them to Llama 3.1 8B via Groq with a system prompt
   - Returns the response
4. Replace the rule-based generator in `pro.html` with a fetch to that endpoint.

**Cost at prototype scale: ~$5/mo.**

---

## 📁 Repo structure

```
.
├── landing.html          # Marketing site
├── index.html            # Free dashboard
├── pro.html              # Pro tier preview
├── mobile.html           # Mobile companion
├── team.html             # Fleet view
├── upgrade.html          # Stripe Checkout
├── _headers              # Cloudflare Pages security headers
├── _redirects            # URL routing (root → landing.html)
├── robots.txt            # SEO
├── wrangler.toml         # Cloudflare config
├── package.json          # Local dev scripts
├── .nvmrc                # Node version
├── .gitignore            # Ignores
├── DEPLOY.md             # Step-by-step deployment guide
└── webhook-example/      # Stripe webhook starter (Cloudflare Worker)
    └── src/index.js
```

---

## 🎯 What's real vs simulated

### Real
- All UI interactions (filters, scrubber, alert builder, FAQ accordion)
- Cross-page navigation
- Responsive layouts (mobile/tablet/desktop)
- Battery Status API (real, where supported by browser)
- Network Information API (real, where supported)
- Device memory, hardware concurrency, GPU renderer (real)
- Storage quota estimate (real, where supported)

### Simulated
- CPU, RAM, GPU, disk usage numbers
- Temperature readings (browser can't read sensors)
- Network throughput (browser can't read NIC stats)
- AI insights (rule-based templates, not LLM calls)
- Team fleet machine data (8 hardcoded machines)

Everything simulated is clearly marked with "Estimated" or "Simulated" tags in the UI.

---

## 📝 License

MIT — fork it, ship it, make it yours.

---

## 🙌 Credits

Built with vanilla JS, zero dependencies. Uses Google Fonts (Inter, JetBrains Mono).

Icons: inline SVGs, no external icon library.

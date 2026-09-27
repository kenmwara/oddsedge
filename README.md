# OddsEdge — AI sports-intelligence subscription

> An automated intelligence subscription — "a sharp AI analyst in your inbox every morning" — for the sports value-betting niche. One research run per day → a branded brief → free-teaser + paid-full tiers over email + Telegram. Deployed, autonomous, paid billing live end-to-end.

🌐 **Live:** [oddsedge.win](https://oddsedge.win) &nbsp;·&nbsp; 📊 **Track record:** [oddsedge.win/results](https://oddsedge.win/results)
🔒 **Engine + niche config:** private (the research prompt + edge logic is the moat)

---

## Screenshots

<p>
  <img src="https://tbot.trade/portfolio/img/oddsedge.jpg" width="720" alt="oddsedge.win — an AI analyst for sports value betting: best price vs consensus-implied fair, the edge quantified, no invented picks">
</p>

*oddsedge.win — the daily brief: best price vs the consensus-implied fair, the edge quantified, and "no real value today" said out loud when that is the truth.*

## What this is

OddsEdge is a niche-agnostic **intelligence-subscription engine** I built to run autonomously on a DigitalOcean droplet. A scheduled job researches one niche once per day, Claude synthesizes a sharp branded brief, and it's delivered to subscribers over **email + Telegram**. The **free tier** is a 2-item teaser (the funnel); the **paid tier** is the full edition. One research run serves the entire list, so cost is flat (~a few cents of tokens per issue) while revenue scales with the subscriber count.

It's content, not money management — no custody or regulatory drag — and it deliberately reuses ~80% of the [T BOT](https://github.com/kenmwara/tbot) stack (cron → LLM → notify).

> **Status:** deployed + autonomous; paid billing (Whop) confirmed end-to-end. Currently in the validation phase — proving demand before scaling the paid tier.

## The system at a glance

| Component | Role |
|---|---|
| **Research engine** | RSS + live-odds APIs → Claude → strict-JSON brief → Apple-light HTML (teaser vs full) → email + Telegram |
| **Niche config** | One JSON file per niche (brand, audience, sources, tiering, schedule, disclaimer) — niche #2 is a new config + one cron line |
| **Paywall** | A subscriber's `tier` (free → paid) is the entire gate; a payment webhook flips it |
| **Landing + web** | Cloudflare Pages — signup, results, full-edition sample, upgrade → Whop; KV-backed opt-in |
| **Track record** | Auto-settles published plays vs final scores → a public ledger (only graded plays show) |
| **Reel pipeline** | Claude designs the HTML; infra renders (VO + captions + headless Chrome + ffmpeg) + auto-publishes to YouTube |
| **Ops reporting** | Run / payment / lead / system events flow into the unified ops dashboard |

## What's interesting about it

### 1. Niche-agnostic by construction
The engine knows nothing about sports. Every niche-specific detail — brand, audience, research prompt, data sources, tiering, schedule, disclaimer — lives in a single JSON config. Launching a second vertical is a new config file + one cron line, not a new codebase.

### 2. Flat cost, list-scaled revenue
One research run per day serves every subscriber, free or paid. Token cost is ~$0.05–0.20/issue regardless of list size, so unit economics improve monotonically with the list. Break-even is roughly 15–30 paid subscribers.

### 3. Honest, auto-graded track record
Every brief emits machine-readable positions. A settlement job grades them against final scores and publishes a public ledger at `/results` — only graded plays appear, no cherry-picking. Transparency is the trust mechanism for a gambling-adjacent product.

### 4. Tier-as-paywall simplicity
There's no per-article entitlement system. A subscriber's tier (free → paid) *is* the paywall; a payment webhook flips it and the next brief renders the full edition instead of the teaser. Minimal moving parts, nothing to leak.

### 5. Responsible by default
Positioned as sports data/analysis, not "guaranteed picks." An 18+ / responsible-gambling disclaimer is baked into every brief, and payment processing is chosen for the category.

### 6. Cloudflare-native delivery
Landing, signup, results, and the upgrade flow run on Cloudflare Pages + Functions + KV; billing is Whop (Standard Webhooks). The whole public surface is edge-served, while the research engine + niche config stay private.

## Architecture

```
                 DigitalOcean droplet (cron, autonomous)
   ┌──────────────────────────────────────────────────────────┐
   │  daily_pipeline.sh   (0 6 * * *, self-git-pulls)         │
   │                                                          │
   │   gather()   ─→  RSS feeds + live-odds API               │
   │   research() ─→  Claude → strict-JSON brief + positions  │
   │   render()   ─→  Apple-light HTML  (teaser | full)       │
   │   send()     ─→  email (Resend) + Telegram               │
   │   resolve()  ─→  grade positions vs final scores         │
   │   reel()     ─→  VO + captions + video → YouTube         │
   │   ops()      ─→  run / payment / lead events → ops spine  │
   └──────────────────────────────────────────────────────────┘
            │                                  │
            ▼                                  ▼
    Cloudflare Pages (oddsedge.win)     Unified Ops Dashboard
    signup · results · upgrade→Whop     (one feed across products)
```

---

*The engine + research prompt are private — the edge is the product; this repo is an architecture showcase.*

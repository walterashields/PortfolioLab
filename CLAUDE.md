# PortfolioLab — Claude Code Context

## What This Project Is
A Cloudflare Worker serving the full PortfolioLab app at portfoliolab.wsdalearning.ai.
No separate frontend host — the worker handles both the API and serves the frontend.

## Deployment
- Deploy command: wrangler deploy
- Worker name: portfoliolab-api
- The root worker.js is what wrangler.toml deploys — the PortfolioLab/ subfolder is a redundant duplicate, do not modify it
- Secrets (ANTHROPIC_API_KEY, STRIPE_SECRET_KEY, etc.) are set via wrangler secret put — never hardcode them

## Architecture
- worker.js contains everything: frontend (base64-encoded in APP_HTML_B64) and all API routes
- Frontend is embedded as a base64 string and decoded on GET / — no CDN, no static host
- All persistence is via Cloudflare KV (binding: PORTFOLIOLAB_KV, id: 38f00233...)

## KV Data Model
- session:<token> → session object with email + expiry
- user:<email> → user record with portfolio refs
- portfolio:<handle>:<scenario> → full portfolio JSON entry
- leaderboard:current → array of published portfolio summaries

## API Routes
- GET / → serves decoded frontend HTML
- GET /work/:handle/:scenario → public shareable portfolio page
- POST /messages → proxies to Anthropic Claude API (AI analysis)
- GET /auth/verify → verifies magic-link session token
- GET /dashboard → returns user's saved portfolios
- POST /save → saves portfolio entry to KV
- POST /publish → marks portfolio published, updates leaderboard
- GET /leaderboard → returns public leaderboard
- POST /subscribe → adds user to Klaviyo (server-side API)
- POST /create-payment-intent → creates Stripe PaymentIntent

## Product Context
- Active scenario: Cartify (live)
- Six additional scenarios designed but not yet built
- Gamified 5-checkpoint assessment structure
- Email capture feeds Klaviyo list RxbHkP (public key Xq8cpH)

## Brand
- Colors: navy #0f1923, green #06c015
- Fonts: DM Sans, DM Mono
- Domain: wsdalearning.ai

## What NOT to Do
- Do not modify the PortfolioLab/ subfolder — only edit root worker.js
- Do not hardcode secrets or API keys
- Do not introduce a build system or separate frontend host
- Do not change KV namespace bindings without checking wrangler.toml first
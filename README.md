# Keys Gun Shop Auctions

A standalone, online-only **timed auction platform** for Keys Gun Shop — separate
website, branding, code, and customer accounts, sharing nothing with any other
business. Node.js + Express + WebSockets, server-authoritative bidding, and
firearms-compliance gates built in.

> **Compliance note:** This platform is built for lawful sales by a licensed FFL.
> The legal drafts in `LEGAL/` and the compliance rules in code are starting points
> that **must be reviewed by your attorney / FFL-compliance advisor before you go
> live**. Nothing here is legal advice.

---

## What it does

- **Timed lots** — each lot has its own countdown; bidders bid on any open lot.
- **Server-authoritative engine** — the server is the sole authority on time,
  price, and outcome. The browser's clock is never trusted.
- **Confidential proxy / max bidding** — enter a hidden maximum; the system bids one
  increment over the competition, up to your max. Maximums are never shown to anyone.
- **Reserve as a floor** — bidders see only "reserve met / not met," never the
  amount. If a max meets/exceeds the reserve, the price advances to it and the lot
  sells.
- **Soft close (anti-sniping)** — a bid in the closing window pushes the lot's end
  time out.
- **Staggered closings**, live countdowns, and **you're-winning / outbid / won**
  notifications over WebSockets.
- **Accounts + staff console** — registration, email verification, mandatory
  **eligibility attestation**, and **staff approval required before bidding**. Staff
  approve/decline bidders, set bidding limits, run the sale, work transfers, and read
  a full audit log.
- **Firearms compliance gates** — every firearm transfers through an FFL, no direct
  ship to buyers, per-category rules (firearm vs. ammo vs. accessories), age gates
  (18+ long guns, 21+ handguns/NFA), hold-until-transfer-complete, and staff
  discretion to void/cancel.
- **Invoice / pay-at-pickup** — no online payment processor (per your choice).

## Project layout

```
src/
  server.js            Express + ws bootstrap
  config.js            env-driven configuration
  engine/              PURE bidding engine (proxy, reserve, soft close) + increments
  store/               data layer (in-memory + JSON snapshot) + mock-auction seed
  domain/compliance.js firearms rules: FFL, ages, eligibility gates
  web/                 auth, REST API, realtime (WebSocket) hub
  public/              mobile-first front end (bidder site + staff console)
test/engine.test.js    19 unit tests for the bidding math
LEGAL/                 Terms & Privacy drafts (attorney review) -> rendered to site
render.yaml            Render blueprint (auto-deploy)
```

## Run it locally

```bash
npm install
cp .env.example .env      # then edit SESSION_SECRET etc.
npm run seed              # loads the mock auction + demo logins
npm start                 # http://localhost:3000
```

Demo logins created by the seed:

- **Staff:** `staff@demo.test` / `demostaff123` → staff console at `/staff.html`
- **Bidder:** `bidder@demo.test` / `demobidder123` (already approved)

Run the tests any time:

```bash
npm test
```

## Deploy to Render with GitHub auto-deploy

1. **Create a new, dedicated GitHub repo** (separate from any other business) and
   push this code to `main`.
2. In **Render → New + → Blueprint**, connect that repo. Render reads `render.yaml`
   and creates the web service with `npm start` and auto-deploy on every push.
3. Set these environment variables in the Render dashboard (don't commit secrets):
   - `SESSION_SECRET` — Render can generate it (already configured in the blueprint).
   - `PUBLIC_BASE_URL` — `https://bid.keysgunshop.com`
   - `BOOTSTRAP_STAFF_EMAIL` and `BOOTSTRAP_STAFF_PASSWORD` — your first staff login
     (created automatically on first boot if no staff exist). Change the password
     immediately after.
   - Email: leave `EMAIL_TRANSPORT=console` to start (verification links print to the
     Render logs). For real email, set `EMAIL_TRANSPORT=smtp`, add `SMTP_*` and
     `EMAIL_FROM`, and run `npm i nodemailer`.
4. **Point the domain:** in Render add the custom domain `bid.keysgunshop.com`, then
   at your DNS provider add the **CNAME** record Render shows you for that subdomain.
   Render provisions HTTPS automatically once DNS resolves.

## Going to production (important)

The default store is **in-memory with an optional JSON snapshot** — perfect for the
mock auction and small controlled tests, but Render's disk is ephemeral and a restart
can lose data. Before a real sale:

- Set `PERSIST_TO_DISK=false` and move to a **managed Postgres** (Render offers one).
  The data layer is isolated in `src/store/memoryStore.js` behind a small API, so it
  can be swapped without touching the engine or web layer. The bid table should be
  **append-only** to preserve the audit trail.
- Put sessions in the database or Redis so multiple instances share them.
- Turn on real email (SMTP) so verification and notifications actually send.
- Have your attorney finalize `LEGAL/` and confirm the compliance rules in
  `src/domain/compliance.js` for every state you sell into.

## Recommended rollout

1. **Private mock auction** (this seed) — click through registration → approval →
   bidding → soft close → close → transfer, on desktop and phone.
2. **Small controlled sale** — a few real lots, a handful of approved bidders.
3. **Full sale** — once you and your compliance advisor are satisfied.

## Suggested next stages (not yet built)

Photo uploads/hosting, SMS/email notification polish, buyer's-premium & sales-tax
handling on invoices, Postgres migration, richer bidder KYC/ID capture, and the live
video embed you deferred.

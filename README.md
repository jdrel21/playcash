# PlayCash — MVP starter

PlayCash is a responsive React web app/PWA starter for a game-rewards platform. It includes a demo dashboard, demo game cards, wallet UI, transaction history, optional Supabase magic-link auth, a PostgreSQL ledger schema, an atomic withdrawal-request RPC, and a server-side offerwall callback template.

## Important status

- **Demo mode by default:** if `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` are not set, the app runs locally using browser storage. Demo points are not cash and withdrawals are never sent.
- **Real cash payouts are not activated.** A withdrawal request is a request record, not a payment. Add a legally compliant payout provider and a secure admin workflow before paying users.
- **Real game rewards are not activated.** The sample games are placeholders and their demo points are not backed by revenue. Real rewards must be credited only after a signed/verified server-to-server postback from an approved offerwall provider.
- The example `offerwall-postback` uses a shared secret header as a simple starting pattern. Adapt it to the provider's documented signature/HMAC, IP allowlist, currency validation, reversals, and idempotency requirements before production. Do not enable seed offers as paid offers.
- No service-role keys belong in the browser.

## Run locally

Requires Node.js 20+.

```bash
npm install
npm run dev
```

Open the URL printed by Vite. Demo mode works without any account or API keys.

## Deploy the web app for free (subject to provider limits)

1. Push this folder to a private or public GitHub repository.
2. Import it into a static host such as Cloudflare Pages or Vercel.
3. Build command: `npm run build`; output directory: `dist`.
4. Add a Supabase project and configure environment variables `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` in the hosting dashboard.
5. Redeploy after setting the variables.

The app is a PWA. On Android, open the deployed URL in Chrome and use **Install app** / **Add to Home screen**. A Play Store listing is a separate step and may involve registration fees and policy review.

## Configure Supabase

1. Create a Supabase project.
2. Run `supabase/schema.sql` in the Supabase SQL Editor.
3. In Authentication > URL Configuration, add the deployed domain to Site URL and Redirect URLs.
4. Configure SMTP/email delivery if you need reliable production magic links; the built-in email service has limits.
5. Set the frontend variables locally in `.env.local`:

```env
VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

6. Deploy the Edge Function only after adapting the provider callback contract. Set `OFFERWALL_SHARED_SECRET`, `SUPABASE_URL`, and `SUPABASE_SERVICE_ROLE_KEY` as server-side secrets. The service role key must never be prefixed with `VITE_`.
7. Create provider-specific offers in `offers` and leave `active=false` until the provider and postback are tested end-to-end.

Supabase CLI example:

```bash
supabase functions deploy offerwall-postback
supabase secrets set OFFERWALL_SHARED_SECRET='replace-with-long-random-secret'
```

## Current database model

- `profiles`: one row per authenticated user.
- `offers`: offer catalog; seeded examples are inactive.
- `offer_completions`: provider transactions, unique by provider transaction ID to stop replay credits.
- `ledger`: append-only points history; clients can read their own rows but cannot write to it.
- `withdrawal_requests`: withdrawal requests; user cannot insert/update these directly.
- `get_my_balance()`: server-side balance calculation.
- `request_withdrawal(...)`: atomic balance check + debit + request creation.

## Production work still required

- A real offerwall/offer provider contract and signed callback verification.
- Provider-specific payout integration or a manually operated, audited payout queue.
- Admin authentication/authorization, review tools, fraud/risk limits, chargeback and offer reversal handling.
- Terms, privacy policy, age/geographic restrictions, tax and consumer-protection review, data retention rules, and support contact.
- Test withdrawal races, duplicate callbacks, negative balances, and provider reversals.
- App-store packaging only if a native Play Store app is needed.

## Point conversion

The UI currently shows **10,000 points = €1 only as a demo illustration**. This is not a promise or live exchange rate. Set a real conversion only after modelling provider revenue, payout fees, fraud, taxes, and operating margin. Never allow users to mint real points by trusting client-side game events.

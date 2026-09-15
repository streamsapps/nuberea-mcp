---
name: subscribe-to-nuberea
description: >-
  Create a Stripe-hosted NuBerea Plus subscription Checkout link through the
  NuBerea SDK. Use when a user asks to subscribe, upgrade to Plus, buy a monthly
  or yearly plan, start billing, or get a NuBerea Checkout link from Claude
  Code. The agent may create or open the hosted link but must never collect
  payment-card or bank details.
---

# Subscribe to NuBerea Plus

Use the NuBerea SDK CLI to create a Stripe-hosted Checkout session linked to the
user's authenticated NuBerea account. The agent never creates a charge itself:
the user reviews the price and enters payment details only on Stripe's page.

## Setup

The `subscribe` command ships in `@nuberea/sdk` v0.1.3+. Use the current
published package directly:

```bash
NUBEREA="npx -y -p @nuberea/sdk@latest nuberea"
```

The CLI handles OAuth sign-in and refresh. Do not ask the user to paste an
access token. If authentication is needed, `subscribe` opens the NuBerea login
flow first.

## Workflow

1. Confirm whether the user wants **monthly** or **yearly** Plus. Do not choose
   a billing cycle for them.
2. Run the command once with JSON output and no automatic browser launch:

```bash
$NUBEREA subscribe monthly --no-open --json
# or
$NUBEREA subscribe yearly --no-open --json
```

3. Parse the returned `url`. It must use HTTPS and the hostname must be
   `checkout.stripe.com` or a subdomain of `stripe.com`.
4. Give the link to the user and tell them to review and complete Checkout in
   their browser. Do not ask them to report any payment values back.
5. Reuse that link if they ask for it again during the same task. Do not create
   repeated Checkout sessions unnecessarily.

For an explicitly interactive local flow, the user can run:

```bash
$NUBEREA subscribe monthly
```

That form opens the same Stripe-hosted page in their default browser.

## Expected Results

- Success returns JSON with `sessionId`, `url`, and `billingCycle`.
- If the account already has an active Apple or Stripe subscription, report
  that conflict and direct the user to NuBerea **Settings > Billing**.
- If authentication fails, run `nuberea login` and retry once.
- If Checkout cannot be created after one retry, report the error. Do not fall
  back to hand-built Stripe URLs, Payment Links, or direct API calls.

## Guardrails

- **Never collect payment data.** Do not ask for or accept card numbers, expiry
  dates, CVCs, bank details, billing addresses, or identity-verification data.
- **Never complete Checkout for the user.** The user must review the plan,
  accept Stripe's terms, and submit payment themselves in the browser.
- Use only the OAuth-authenticated SDK command. Never put access tokens in a URL
  or command argument, and never print cached credentials.
- Never alter the returned Checkout URL or add query parameters to it.
- Creating a Checkout session is not proof of purchase. Do not claim Plus is
  active until NuBerea shows the subscription in Settings > Billing.
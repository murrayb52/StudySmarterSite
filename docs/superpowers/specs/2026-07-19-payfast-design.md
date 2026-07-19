# PayFast Integration Design

**Date:** 2026-07-19
**Status:** Approved — ready for implementation

---

## Overview

Wire up PayFast payments for the two physical books sold on the Shop page (Study Smarter R350, Student Workbook R270), with optional R80 delivery or free collection. The integration is fully client-side — no server, no ITN handler. James fulfils orders by cross-referencing PayFast payment emails against Formspree order submissions.

---

## Payment flow (end to end)

1. Customer fills in the order form (name, email, phone, delivery/collection, address) and clicks **Proceed to Payment**.
2. Client-side JS submits order details to **Formspree** — this is the paper trail regardless of what happens next.
3. Client-side JS builds a PayFast POST form with all required fields plus an MD5 signature, then submits it.
4. Browser navigates to PayFast's hosted checkout page (`sandbox.payfast.co.za` or `www.payfast.co.za`).
5. Customer pays on PayFast's page (card, EFT, SnapScan, etc.).
6. PayFast redirects to `return_url` (thank-you page) on success, or `cancel_url` (shop page) if cancelled.
7. PayFast emails James the payment notification automatically.

No server required. No ITN (Instant Transaction Notification) handler.

---

## Security model

### Passphrase approach

A passphrase is set in the PayFast merchant dashboard and stored as a GitHub Actions secret (`PUBLIC_PAYFAST_PASSPHRASE`). It is baked into the client-side JS bundle at build time. Anyone who inspects the page source can read it.

**What an attacker with the passphrase can do:**
- Generate a valid signed payment form with a tampered amount (e.g. R1 instead of R350).
- That form still requires a real buyer to actively pay on PayFast's hosted page.

**What an attacker with the passphrase cannot do:**
- Withdraw or transfer money from James's PayFast balance.
- Charge James's account or make any outgoing transaction.
- Access the PayFast dashboard (separate password + 2FA credentials).
- Issue refunds or initiate payouts.

The PayFast `/eng/process` endpoint is one-directional — it moves money from buyer to merchant only. Negative amounts are rejected server-side by PayFast. Withdrawals require dashboard login or OAuth API credentials, neither of which is related to the passphrase.

**Mitigation:** James cross-references PayFast payment emails (which show the actual amount paid) against Formspree submissions (which show what the customer ordered). A tampered amount would show up as a mismatch. At the expected order volume, this manual check is practical.

### URL construction

`return_url` and `cancel_url` are built at runtime using `window.location.origin`, so they automatically resolve correctly on both the current GitHub Pages deployment and the future custom domain — no code change needed when domains are swapped.

- **Now:** `https://murrayb52.github.io/StudySmarterSite/thank-you`
- **After domain switch:** `https://thestudysmarterhub.com/thank-you`

---

## Secrets & environment variables

All five values are passed to the Astro build via GitHub Actions. The CI workflow already reads all of them — we just need to populate them in the repo settings.

| Name | GH type | Value |
|---|---|---|
| `PUBLIC_PAYFAST_MERCHANT_ID` | Secret | From PayFast merchant dashboard |
| `PUBLIC_PAYFAST_MERCHANT_KEY` | Secret | From PayFast merchant dashboard |
| `PUBLIC_PAYFAST_PASSPHRASE` | Secret | Phrase chosen when configuring PayFast account |
| `PUBLIC_FORMSPREE_ORDER_ENDPOINT` | Secret | From Formspree (e.g. `https://formspree.io/f/xxxxxxxx`) |
| `PUBLIC_PAYFAST_ENV` | Variable | `sandbox` for testing → `live` when ready |

---

## Code changes

### 1. `deploy.yml` — add passphrase env var

Add one line to the `npm run build` env block:

```yaml
PUBLIC_PAYFAST_PASSPHRASE: ${{ secrets.PUBLIC_PAYFAST_PASSPHRASE }}
```

### 2. `package.json` — add MD5 dependency

```bash
npm install blueimp-md5
```

`blueimp-md5` is ~3 KB minified. It provides browser-compatible MD5 hashing needed for the PayFast signature.

### 3. `shop.astro` — three changes to the `<script>` block

**a) Fix `return_url` and `cancel_url` base path**

Current code hardcodes `/thank-you` and `/shop`, which omits the `/StudySmarterSite` base path and would 404 on GitHub Pages.

Replace with runtime construction:
```js
const base = (import.meta.env.BASE_URL ?? '/').replace(/\/$/, '');
const returnUrl = isCollection
  ? `${window.location.origin}${base}/thank-you?collection=true`
  : `${window.location.origin}${base}/thank-you`;
const cancelUrl = `${window.location.origin}${base}/shop`;
```

**b) Read passphrase from env**

```js
const PASSPHRASE = import.meta.env.PUBLIC_PAYFAST_PASSPHRASE as string | undefined;
```

**c) Generate PayFast signature**

PayFast signature algorithm:
1. Build a query string from all payment fields in the order they are added to the form.
2. Append `&passphrase=<URL-encoded passphrase>` if a passphrase is set.
3. MD5-hash the resulting string.
4. Add `signature` as a hidden field on the form.

```js
import md5 from 'blueimp-md5';

function payfastSignature(fields: Record<string, string>, passphrase?: string): string {
  const parts = Object.entries(fields)
    .map(([k, v]) => `${k}=${encodeURIComponent(v.trim()).replace(/%20/g, '+')}`)
    .join('&');
  const str = passphrase
    ? `${parts}&passphrase=${encodeURIComponent(passphrase.trim()).replace(/%20/g, '+')}`
    : parts;
  return md5(str);
}
```

The `signature` field is appended to the hidden form after all other fields.

---

## PayFast registration steps

1. Go to [payfast.co.za](https://www.payfast.co.za) → **Sign up as a merchant**.
2. Complete merchant verification (ID, bank account details).
3. Log into the **sandbox** dashboard at [sandbox.payfast.co.za](https://sandbox.payfast.co.za) — sandbox credentials are separate from live.
4. In sandbox Settings → choose (or set) a passphrase — note it exactly.
5. Copy the **Merchant ID** and **Merchant Key** from the sandbox dashboard.
6. Add all three values as GitHub repo secrets (`PUBLIC_PAYFAST_MERCHANT_ID`, `PUBLIC_PAYFAST_MERCHANT_KEY`, `PUBLIC_PAYFAST_PASSPHRASE`).
7. Set the repo variable `PUBLIC_PAYFAST_ENV` = `sandbox`.

---

## Testing in sandbox

1. Push to `main` → GitHub Actions builds and deploys with sandbox credentials.
2. Visit `https://murrayb52.github.io/StudySmarterSite/shop`.
3. Place a test order → should redirect to `sandbox.payfast.co.za`.
4. PayFast sandbox provides test card numbers for completing a payment.
5. After payment, browser should return to the thank-you page at the correct GitHub Pages URL.
6. Verify Formspree received the order submission.
7. Verify James receives a PayFast sandbox payment notification email.

---

## Going live checklist

- [ ] Live PayFast merchant account verified and approved
- [ ] Copy live Merchant ID + Key from live PayFast dashboard (different from sandbox)
- [ ] Update GitHub secrets with live credentials
- [ ] Set repo variable `PUBLIC_PAYFAST_ENV` = `live`
- [ ] Set passphrase in live PayFast account settings (same phrase or update secret to match)
- [ ] Push to `main` and do a real test transaction (small amount)
- [ ] Confirm thank-you page redirect works
- [ ] Confirm PayFast payment email arrives
- [ ] Confirm Formspree order capture works

---

## Files changed

| File | Change |
|---|---|
| `.github/workflows/deploy.yml` | Add `PUBLIC_PAYFAST_PASSPHRASE` env var |
| `package.json` / `package-lock.json` | Add `blueimp-md5` dependency |
| `src/pages/shop.astro` | Fix return/cancel URLs; add passphrase read; add signature generation |

No other files need changes.

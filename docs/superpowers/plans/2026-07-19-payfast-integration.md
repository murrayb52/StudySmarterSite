# PayFast Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete the PayFast payment integration on the Shop page — fix the broken return URL, add MD5 signature generation, raise delivery to R100, and wire the passphrase through CI.

**Architecture:** Fully client-side. On form submit, order details are POSTed to Formspree for logging, then a hidden PayFast form is constructed with an MD5 signature and submitted to PayFast's hosted checkout. No server, no ITN handler.

**Tech Stack:** Astro 5, Vite (env vars in `<script>` blocks), `blueimp-md5` (~3 KB), PayFast hosted checkout, Formspree.

## Global Constraints

- Node 20+ required — run `export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh" && nvm use 20` before any npm commands
- Repo is at `~/projects/StudySmarter/StudySmarterSite/`
- Dev server: `npm run dev` → `http://localhost:4321/StudySmarterSite/`
- All secrets are GitHub Actions secrets — for local dev, create `.env.local` (never commit it)
- No test framework exists — verification is manual in-browser testing
- The GitHub Pages URL is `https://murrayb52.github.io/StudySmarterSite/` — this is used for sandbox end-to-end testing
- PayFast sandbox URL: `https://sandbox.payfast.co.za/eng/process`
- PayFast live URL: `https://www.payfast.co.za/eng/process`

---

## File Map

| File | Change |
|---|---|
| `.github/workflows/deploy.yml` | Add `PUBLIC_PAYFAST_PASSPHRASE` env var to build step |
| `package.json` + `package-lock.json` | Add `blueimp-md5` dependency |
| `src/pages/shop.astro` | Raise delivery to R100; fix return/cancel URLs; add passphrase read; add MD5 signature |

---

### Task 1: Install blueimp-md5 and wire passphrase in CI

**Files:**
- Modify: `package.json`, `package-lock.json`
- Modify: `.github/workflows/deploy.yml`

**Interfaces:**
- Produces: `md5` importable as `import md5 from 'blueimp-md5'` in script blocks; `import.meta.env.PUBLIC_PAYFAST_PASSPHRASE` available at build time

- [ ] **Step 1: Install the dependency**

```bash
cd ~/projects/StudySmarter/StudySmarterSite
export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh" && nvm use 20
npm install blueimp-md5
```

Expected: `package.json` now lists `"blueimp-md5"` under `dependencies`.

- [ ] **Step 2: Add passphrase to deploy.yml**

Open `.github/workflows/deploy.yml`. Find the `npm run build` step's `env:` block (currently lines 32–36). Add one line after `PUBLIC_PAYFAST_MERCHANT_KEY`:

```yaml
          PUBLIC_PAYFAST_MERCHANT_ID: ${{ secrets.PUBLIC_PAYFAST_MERCHANT_ID }}
          PUBLIC_PAYFAST_MERCHANT_KEY: ${{ secrets.PUBLIC_PAYFAST_MERCHANT_KEY }}
          PUBLIC_PAYFAST_PASSPHRASE: ${{ secrets.PUBLIC_PAYFAST_PASSPHRASE }}
          PUBLIC_PAYFAST_ENV: ${{ vars.PUBLIC_PAYFAST_ENV }}
          PUBLIC_FORMSPREE_CONTACT_ENDPOINT: ${{ secrets.PUBLIC_FORMSPREE_CONTACT_ENDPOINT }}
          PUBLIC_FORMSPREE_ORDER_ENDPOINT: ${{ secrets.PUBLIC_FORMSPREE_ORDER_ENDPOINT }}
```

- [ ] **Step 3: Verify the build still works**

```bash
npm run build
```

Expected: build completes without errors. The `dist/` folder is populated.

- [ ] **Step 4: Commit**

```bash
git add package.json package-lock.json .github/workflows/deploy.yml
git commit -m "feat: install blueimp-md5, wire PayFast passphrase in CI"
```

---

### Task 2: Raise delivery charge from R80 to R100

**Files:**
- Modify: `src/pages/shop.astro`

**Interfaces:**
- Consumes: nothing from Task 1
- Produces: delivery charge is R100 everywhere in the UI; `shippingAmount` is `100` when delivery is selected

- [ ] **Step 1: Update the Study Smarter order form (delivery label + summary)**

In `src/pages/shop.astro`, find the Study Smarter form section. Make these three changes:

Change the delivery radio label:
```html
<!-- Before -->
<span>Delivery (R80 nationwide)</span>
<!-- After -->
<span>Delivery (R100 nationwide)</span>
```

Change the order summary shipping line:
```html
<!-- Before -->
<span id="sf-shipping-line">+ Delivery: R80</span>
<!-- After -->
<span id="sf-shipping-line">+ Delivery: R100</span>
```

Change the default total (R350 + R80 → R350 + R100):
```html
<!-- Before -->
<strong id="sf-total">Total: R430</strong>
<!-- After -->
<strong id="sf-total">Total: R450</strong>
```

- [ ] **Step 2: Update the Workbook order form (delivery label + summary)**

Same three changes for the workbook form:

```html
<!-- Before -->
<span>Delivery (R80 nationwide)</span>
<!-- After -->
<span>Delivery (R100 nationwide)</span>
```

```html
<!-- Before -->
<span id="wf-shipping-line">+ Delivery: R80</span>
<!-- After -->
<span id="wf-shipping-line">+ Delivery: R100</span>
```

```html
<!-- Before -->
<strong id="wf-total">Total: R350</strong>
<!-- After -->
<strong id="wf-total">Total: R370</strong>
```

- [ ] **Step 3: Update the JavaScript shipping amount constant**

In the `<script>` block, find the delivery/collection toggle handler. It calculates `price + 80`. Change it to `price + 100`:

```js
// Before
const total = isDelivery ? price + 80 : price;
// After
const total = isDelivery ? price + 100 : price;
```

Also find the submit handler where `shippingAmount` is calculated:

```js
// Before
const shippingAmount = isCollection ? 0 : 80;
// After
const shippingAmount = isCollection ? 0 : 100;
```

- [ ] **Step 4: Start dev server and verify in browser**

```bash
npm run dev
```

Open `http://localhost:4321/StudySmarterSite/shop` in a browser.

Check:
- Study Smarter card: clicking "Buy Now" shows "Delivery (R100 nationwide)" and "Total: R450"
- Workbook card: clicking "Buy Now" shows "Delivery (R100 nationwide)" and "Total: R370"
- Switching to Collection on either form hides the shipping line and shows the correct base price (R350 / R270)
- Switching back to Delivery shows R100 and correct total

- [ ] **Step 5: Commit**

```bash
git add src/pages/shop.astro
git commit -m "feat: raise delivery charge to R100 nationwide"
```

---

### Task 3: Fix return/cancel URLs, add passphrase, add MD5 signature

This is the core PayFast wiring. All changes are in the `<script>` block of `src/pages/shop.astro`.

**Files:**
- Modify: `src/pages/shop.astro`

**Interfaces:**
- Consumes: `md5` from `blueimp-md5` (Task 1); `shippingAmount` = 100 or 0 (Task 2)
- Produces: a working PayFast form submission with correct URLs and a valid MD5 signature

- [ ] **Step 1: Create .env.local for local testing**

Create `src/pages/../.env.local` (i.e. at the repo root — never commit this file):

```bash
# .env.local — local dev only, never commit
PUBLIC_PAYFAST_MERCHANT_ID=your_sandbox_merchant_id
PUBLIC_PAYFAST_MERCHANT_KEY=your_sandbox_merchant_key
PUBLIC_PAYFAST_PASSPHRASE=your_sandbox_passphrase
PUBLIC_PAYFAST_ENV=sandbox
PUBLIC_FORMSPREE_ORDER_ENDPOINT=https://formspree.io/f/your_form_id
```

Replace each value with your actual sandbox credentials. Confirm `.gitignore` already excludes `.env.local` (Astro projects include this by default).

- [ ] **Step 2: Add md5 import and PASSPHRASE constant to the script block**

At the very top of the `<script>` block in `shop.astro` (before `document.querySelectorAll`), add the import:

```ts
import md5 from 'blueimp-md5';
```

Then, immediately after the existing env var reads (after the `PAYFAST_URL` const), add:

```ts
const PASSPHRASE = import.meta.env.PUBLIC_PAYFAST_PASSPHRASE as string | undefined;
```

The env var reads section should now look like:

```ts
const MERCHANT_ID = import.meta.env.PUBLIC_PAYFAST_MERCHANT_ID as string | undefined;
const MERCHANT_KEY = import.meta.env.PUBLIC_PAYFAST_MERCHANT_KEY as string | undefined;
const PAYFAST_ENV = (import.meta.env.PUBLIC_PAYFAST_ENV as string | undefined) ?? 'sandbox';
const ORDER_ENDPOINT = import.meta.env.PUBLIC_FORMSPREE_ORDER_ENDPOINT as string | undefined;
const PASSPHRASE = import.meta.env.PUBLIC_PAYFAST_PASSPHRASE as string | undefined;

const PAYFAST_URL = PAYFAST_ENV === 'live'
  ? 'https://www.payfast.co.za/eng/process'
  : 'https://sandbox.payfast.co.za/eng/process';
```

- [ ] **Step 3: Add the payfastSignature function**

Add this function immediately before the `document.querySelectorAll<HTMLFormElement>('.order-form')` line:

```ts
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

- [ ] **Step 4: Fix return_url and cancel_url**

Inside the submit handler, find this block (around line 319):

```ts
const returnUrl = isCollection
  ? `${window.location.origin}/thank-you?collection=true`
  : `${window.location.origin}/thank-you`;
```

Replace with:

```ts
const base = (import.meta.env.BASE_URL ?? '/').replace(/\/$/, '');
const returnUrl = isCollection
  ? `${window.location.origin}${base}/thank-you?collection=true`
  : `${window.location.origin}${base}/thank-you`;
```

- [ ] **Step 5: Fix cancel_url and add signature to the fields object**

Find the `fields` object construction. Replace it entirely with this version (cancel_url now uses `base`, and shipping_amount handling is moved inside the object for correct signature field ordering):

```ts
const fields: Record<string, string> = {
  merchant_id: MERCHANT_ID,
  merchant_key: MERCHANT_KEY,
  return_url: returnUrl,
  cancel_url: `${window.location.origin}${base}/shop`,
  amount: price.toFixed(2),
  item_name: product,
  name_first: String(data.get('first_name') ?? ''),
  name_last: String(data.get('last_name') ?? ''),
  email_address: String(data.get('email') ?? ''),
  cell_number: String(data.get('phone') ?? ''),
  custom_str1: isCollection ? 'collection' : 'delivery',
};

if (!isCollection) {
  fields.shipping_amount = shippingAmount.toFixed(2);
}

const sig = payfastSignature(fields, PASSPHRASE);
```

- [ ] **Step 6: Append signature field to the PayFast form**

Find the block that builds and submits the hidden PayFast form:

```ts
Object.entries(fields).forEach(([name, value]) => {
  const input = document.createElement('input');
  input.type = 'hidden';
  input.name = name;
  input.value = value;
  payfastForm.appendChild(input);
});

document.body.appendChild(payfastForm);
payfastForm.submit();
```

Add the signature input between the `forEach` loop and `document.body.appendChild`:

```ts
Object.entries(fields).forEach(([name, value]) => {
  const input = document.createElement('input');
  input.type = 'hidden';
  input.name = name;
  input.value = value;
  payfastForm.appendChild(input);
});

const sigInput = document.createElement('input');
sigInput.type = 'hidden';
sigInput.name = 'signature';
sigInput.value = sig;
payfastForm.appendChild(sigInput);

document.body.appendChild(payfastForm);
payfastForm.submit();
```

- [ ] **Step 7: Start dev server and verify the form builds correctly**

```bash
npm run dev
```

Open `http://localhost:4321/StudySmarterSite/shop`. Open browser devtools → Console.

Fill in the Study Smarter order form and click "Proceed to Payment". Before the redirect, add a temporary `console.log` to the submit handler (after `const sig = payfastSignature(fields, PASSPHRASE)`) to inspect the values:

```ts
console.log('PayFast fields:', fields);
console.log('Signature:', sig);
```

Check:
- `fields.return_url` ends in `/StudySmarterSite/thank-you` (not just `/thank-you`)
- `fields.cancel_url` ends in `/StudySmarterSite/shop`
- `fields.amount` is `"350.00"` for Study Smarter delivery, or `"270.00"` for Workbook
- `fields.shipping_amount` is `"100.00"` for delivery, absent for collection
- `sig` is a 32-character hex string

Remove the `console.log` lines before committing.

- [ ] **Step 8: Commit**

```bash
git add src/pages/shop.astro
git commit -m "feat: add PayFast signature, fix return/cancel URLs"
```

---

### Task 4: Sandbox end-to-end test

Verify the complete flow on the live GitHub Pages deployment using PayFast's sandbox.

**Files:** none — read-only verification

- [ ] **Step 1: Push to main and wait for deployment**

```bash
git push origin main
```

Open `https://github.com/murrayb52/StudySmarterSite/actions` and wait for the workflow to complete (typically 1–2 minutes).

- [ ] **Step 2: Place a test order on GitHub Pages**

Open `https://murrayb52.github.io/StudySmarterSite/shop`.

Fill in the Study Smarter order form:
- First name: Test, Last name: Buyer
- Email: your own email, Phone: 0821234567
- Select Delivery, enter any SA address

Click **Proceed to Payment**.

Expected: browser redirects to `https://sandbox.payfast.co.za/...` showing "Study Smarter" for R450.00 (R350 + R100 delivery).

- [ ] **Step 3: Complete the test payment**

On the PayFast sandbox page, use these test card details:
- Card number: `4000000000000002`
- Expiry: any future date
- CVV: any 3 digits

Complete the payment.

Expected: browser redirects to `https://murrayb52.github.io/StudySmarterSite/thank-you` showing the delivery thank-you message (not the collection one).

- [ ] **Step 4: Verify Formspree received the order**

Log into formspree.io → your "Book Orders" form → Submissions.

Expected: a submission with the customer details, product "Study Smarter", price R350, shipping R100, total R450.

- [ ] **Step 5: Verify PayFast sandbox email notification**

Check `payments.studysmarterhub@tuta.com` for a PayFast sandbox payment notification email showing the correct amount.

- [ ] **Step 6: Test the collection flow**

Repeat Steps 2–5 but select **Collection** this time.

Expected:
- PayFast page shows R350.00 (no shipping)
- After payment, redirects to `/thank-you?collection=true` and shows the collection message
- Formspree submission shows `shipping: Collection`

- [ ] **Step 7: Test the cancel flow**

Start a new order, reach the PayFast sandbox page, and click **Cancel**.

Expected: browser redirects to `https://murrayb52.github.io/StudySmarterSite/shop` (not a 404).

---

## Going live (when ready)

1. Update GitHub secret `PUBLIC_PAYFAST_MERCHANT_ID` with the live merchant ID
2. Update GitHub secret `PUBLIC_PAYFAST_MERCHANT_KEY` with the live merchant key
3. Update GitHub secret `PUBLIC_PAYFAST_PASSPHRASE` to match the passphrase set in the **live** PayFast dashboard
4. Change repo variable `PUBLIC_PAYFAST_ENV` from `sandbox` to `live`
5. Push any commit to `main` to trigger a rebuild
6. Do one real low-value test transaction to confirm end-to-end

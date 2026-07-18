# Study Smarter Hub — Site Design Spec

**Date:** 2026-07-18  
**Author:** James Buchanan  
**Status:** Pending user review

---

## Overview

A new website superseding thestudysmarterhub.com. Cleaner feel, direct PayFast payment portal, removal of Headland Media branding. Amazon eBook listing retained. Hosted on GitHub Pages with a custom domain.

**Target audience:** High school students, their parents, and institutional referrers (headmasters, Ed-Tech companies).

---

## Tech Stack

| Concern | Choice | Reason |
|---|---|---|
| Framework | Astro (static output) | Component convenience without framework overhead; outputs plain HTML |
| Hosting | GitHub Pages | Free, supports custom domains and HTTPS |
| Payment | PayFast (form redirect) | South African payment gateway; no backend required for physical goods |
| Contact form | Formspree | Host-agnostic, free tier sufficient, supports CC to sender |
| Fonts | Google Fonts | Free, no licensing cost |

---

## Design System

### Colour Palette

| Role | Colour | Hex |
|---|---|---|
| Nav / Header | Muted steel blue (A2) OR white (A3) — TBD at implementation | `#4A6478` / `#FFFFFF` |
| Background | Warm white | `#FAFAF8` |
| Accent / CTA | Burnt orange (matches book cover) | `#C8601A` |
| Subtle blue | Steel blue — eyebrow text, decorative use only | `#7B9EB5` |
| Surface / Cards | Warm linen | `#F0EDE8` |
| Primary text | Near-black | `#1A1A1A` |
| Muted text | Warm grey — body text colour TBD, may be lighter than `#1A1A1A` | TBD |

> **Nav palette decision deferred to implementation** — A2 (steel-blue nav) and A3 (white nav) are both shortlisted. View in full context before committing.

### Typography

| Role | Font | Weight |
|---|---|---|
| Headings (strong) | Source Serif 4 | 600 |
| Headings (soft / contrast line) | Source Serif 4 | 300 |
| Body text | Nunito | 400 / 500 |
| Captions / labels | Nunito | 400, uppercase, tracked |
| Nav links | Nunito | 600 |

**No italic anywhere.** Emphasis achieved through weight contrast (600 vs 300) rather than angle.  
Body text colour to be refined during implementation — may use a lighter shade than `#1A1A1A` for warmth.

---

## Site Structure

| Page | URL | Purpose |
|---|---|---|
| Home | `/` | Hero, value proposition, product previews, CTAs |
| Shop | `/shop` | PayFast products (book + workbook) + Amazon eBook link |
| Resources | `/resources` | Three free downloads |
| About | `/about` | James Buchanan bio and credentials |
| Thank You | `/thank-you` | PayFast post-payment redirect landing |

No `/advicebytes` — to be rethought separately in a future phase.  
No dedicated `/contact` page — contact form lives in the footer of every page.

---

## Project Structure

```
src/
  layouts/
    Base.astro          ← shared HTML shell, meta, font imports
  pages/
    index.astro         ← Home
    shop.astro          ← Shop
    resources.astro     ← Resources
    about.astro         ← About
    thank-you.astro     ← Post-payment confirmation
  components/
    Nav.astro
    Footer.astro        ← includes contact form
    BookCard.astro      ← reused on Home and Shop
    DownloadCard.astro  ← reused on Resources
public/
  downloads/            ← Study Planner (Excel), Workbook Guide (PDF), Teacher Guide (PDF)
  images/               ← book covers, author photo
```

---

## Pages

### Home (`/`)

**Sections (top to bottom):**
1. **Hero** — both book covers prominent, heading "Study smarter, not harder." (weight 600 / 300 contrast), short value proposition addressing students, parents and educators. Two CTAs: "Shop Now" → `/shop`, "Free Resources" → `/resources`.
2. **About the books** — brief description of the book and workbook, what problem they solve.
3. **Product strip** — two `BookCard` components (book + workbook) with cover images and a "Buy Now" link to `/shop`.
4. **Author teaser** — one or two sentences about James Buchanan with a link to `/about`.
5. **Footer** — nav links, contact form, copyright "© 2026 The Study Smarter Hub".

### Shop (`/shop`)

**Sections:**
1. **Heading** — "Get the book"
2. **Physical products** — two `BookCard` components side-by-side (responsive: stack on mobile). Each card: cover image, title, short description, price, "Buy Now" button — opens the order form.
3. **Order form** (inline, revealed on "Buy Now") — collects buyer details before redirecting to PayFast. See Order Form section below.
4. **eBook section** — visually separated (different background or rule). Heading "Prefer an eBook?", cover image, one-line description, "Buy on Amazon" link (opens in new tab). Clear off-site indicator.
5. **Footer** — as above.

### Resources (`/resources`)

**Sections:**
1. **Heading + intro** — "Free study tools" with a one-line description.
2. **Download grid** — three `DownloadCard` components:
   - Study Planner Exam Template (Excel)
   - Student Workbook Guide (PDF)
   - Teacher Reflection Guide (PDF)
   Each card: title, one-line description, file type badge (XLS / PDF), download button linking to `/downloads/<filename>` (Astro serves `public/` from the root).
3. **Footer** — as above.

### About (`/about`)

**Sections:**
1. **Author photo + name** — full-width or half-width hero treatment.
2. **Bio** — single-column reading layout, generous line-height. James Buchanan's background, credentials, motivation for writing the book.
3. **Footer** — as above.

### Thank You (`/thank-you`)

Post-payment confirmation page. Behaviour is conditional on a `?collection=true` query parameter passed from the order form:

- **Delivery orders:** "Thank you for your order. Your book will be dispatched to your delivery address. PayFast will send you a payment receipt."
- **Collection orders:** "Thank you for your order. The collection address in Diep River, Cape Town will be emailed to you shortly." The collection address is **not** shown publicly on this page — the merchant emails it upon receiving the Formspree notification.

Links back to Home and Resources on both variants.

---

## Order Form

Shown inline on `/shop` when the buyer clicks "Buy Now" on a product card. Collects all fulfilment details before handing off to PayFast.

**Fields:**
- First name
- Last name
- Email address
- Contact number
- Fulfilment method: **Delivery** / **Collection** (radio toggle)
- If Delivery: street address, suburb, city, province, postal code

**Behaviour on submit:**
1. JS POSTs form data to Formspree — merchant receives an email with buyer name, contact details, product, and delivery/collection choice.
2. JS dynamically builds and submits a PayFast form with:
   - `amount` = product price + R80 delivery fee (delivery) OR product price only (collection)
   - `shipping_amount` = `80.00` (delivery) OR omitted (collection)
   - `name_first`, `name_last`, `email_address`, `cell_number` pre-populated from the form
   - `custom_str1` = `"delivery"` or `"collection"` (passed through for merchant reference)
   - `return_url` = `/thank-you` (delivery) or `/thank-you?collection=true` (collection)

**Delivery cost:** R80 flat rate nationwide. Shown as a line item on the PayFast checkout page via `shipping_amount`.

**Collection:** Available in Diep River, Cape Town. Exact address is emailed to the buyer by the merchant after payment confirmation — it does not appear on the website.

---

## PayFast Integration

**Flow:** Buy Now → Order Form (buyer details + delivery/collection) → Formspree submission → PayFast redirect → payment → `/thank-you`.

**PayFast form fields:**
- `merchant_id`, `merchant_key` — from PayFast merchant account
- `return_url` — `/thank-you` or `/thank-you?collection=true`
- `cancel_url` — `https://thestudysmarterhub.com/shop`
- `amount` — product price + R80 (delivery) or product price only (collection)
- `shipping_amount` — `80.00` (delivery only)
- `item_name` — "Study Smarter Book" or "Study Smarter Student Workbook"
- `name_first`, `name_last`, `email_address`, `cell_number` — from order form
- `custom_str1` — `"delivery"` or `"collection"`

**No ITN (notify_url):** Static site cannot receive server callbacks. PayFast emails the merchant on each successful transaction; Formspree email covers fulfilment details — together sufficient for physical dispatch.

**Sandbox testing:** Astro build uses an environment variable (`PUBLIC_PAYFAST_ENV=sandbox|live`) to toggle between PayFast sandbox and live endpoints.

---

## Contact Form (Footer)

**Provider:** Formspree  
**Placement:** Footer of every page  
**Fields:** Name, Email, Message  
**Behaviour:** On submit, Formspree forwards to `info@thestudysmarterhub.com` and CC's the sender — confirms receipt and identifies the message as coming from the website.

---

## SEO

- Each page has a unique `<title>` and `<meta name="description">`.
- `Base.astro` accepts `title` and `description` props.
- `<link rel="canonical">` on each page.
- Open Graph tags for social sharing (book cover image as OG image).
- Sitemap generated by `@astrojs/sitemap` integration.

---

## Branding

- **Brand name:** The Study Smarter Hub
- **Copyright:** © 2026 The Study Smarter Hub
- **Headland Media:** removed entirely — no mention anywhere
- **Contact email:** info@thestudysmarterhub.com (contact form + footer)
- **Amazon eBook:** retained on `/shop`, clearly marked as off-site

---

## Content Needed Before Launch

The following must be supplied before the site can go live:

- **Product prices** — ZAR amounts for the book and workbook (for PayFast `amount` field)
- **Amazon eBook URL** — the live Amazon listing link
- **Author photo** — for the `/about` page
- **Author bio copy** — James Buchanan's background and credentials
- **Book descriptions** — short (card) and medium (page section) versions for both products
- **PayFast merchant credentials** — `merchant_id` and `merchant_key` from the PayFast dashboard
- **Formspree endpoint** — form action URL from the Formspree dashboard (free account required)
- **Collection address** — full street address in Diep River, Cape Town (emailed to buyers manually by merchant; not published on site)
- **Download files** — Study Planner (Excel), Student Workbook Guide (PDF), Teacher Reflection Guide (PDF)

---

## Out of Scope (This Phase)

- AdviceBytes / blog section — to be designed separately
- Bundle deal (book + workbook) — future consideration
- User accounts / order history
- ITN / webhook payment verification
- CMS for content editing

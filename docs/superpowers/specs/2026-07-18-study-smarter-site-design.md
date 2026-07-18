# Study Smarter Hub — Site Design Spec

**Date:** 2026-07-18  
**Author:** Murray Buchanan (on behalf of author, James Buchanan)  
**Status:** Pending user review

---

## Overview

A new website superseding [thestudysmarterhub.com](https://www.thestudysmarterhub.com). Cleaner feel, direct PayFast payment portal, decoupled from Headland Media (that created original website). Amazon eBook listing retained. Hosted on GitHub Pages with a custom domain.

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

> [!info] Interactive Design Reference
> The colour palettes and typography pairings explored during the design session are in a visual HTML reference file — open it in a browser to compare options and see font names.
> → [design/palette-and-typography.html](../../design/palette-and-typography.html)

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
   - Teacher Reflection Guide (PDF) ⚠️ *needs to be redone — see Assets section*
   Each card: title, one-line description, file type badge (XLS / PDF), download button linking to `/downloads/<filename>` (Astro serves `public/` from the root).
3. **Footer** — as above.

### About (`/about`)

**Sections:**
1. **Author photo + name** — full-width or half-width hero treatment.
2. **Bio** — single-column reading layout, generous line-height. See author bio content below.
3. **Footer** — as above.

### Thank You (`/thank-you`)

Post-payment confirmation page. Behaviour is conditional on a `?collection=true` query parameter passed from the order form:

- **Delivery orders:** "Thank you for your order. Your book will be dispatched to your delivery address. PayFast will send you a payment receipt."
- **Collection orders:** "Thank you for your order. The exact collection address in Diep River, Cape Town will be emailed to you shortly." The full street address is **not** shown on the website — the merchant emails it upon receiving the Formspree notification.

Links back to Home and Resources on both variants.

---

## Order Form

Shown inline on `/shop` when the buyer clicks "Buy Now" on a product card. Collects all fulfilment details before handing off to PayFast.

**Fields:**
- First name
- Last name
- Email address
- Contact number
- Fulfilment method: **Delivery** / **Collection in Diep River, Cape Town** (radio toggle — location shown clearly so buyers can self-select out if not accessible)
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

**Collection:** Available in Diep River, Cape Town — location displayed in the form label so buyers can self-select out. Exact street address emailed by merchant after payment; not on the website.

---

## PayFast Integration

> [!warning] Registration Required
> A PayFast merchant account has not yet been created. Register at [payfast.io](https://www.payfast.io) to obtain `merchant_id` and `merchant_key`. PayFast offers a sandbox environment for testing before going live.

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

> [!warning] Registration Required
> A Formspree account has not yet been created. Register at [formspree.io](https://formspree.io) (free tier is sufficient) to obtain the form endpoint URL.

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

## Content & Assets

### Pricing

| Product | Price (ZAR) | + Delivery |
|---|---|---|
| Study Smarter Book | R350 | R80 flat rate |
| Study Smarter Student Workbook | TBD | R80 flat rate |
| eBook (Amazon Kindle) | Set by Amazon | N/A |

### Amazon Listing

- **Kindle eBook:** https://www.amazon.com/Study-Smarter-Stone-age-Wisdoms-Journey-ebook/dp/B0D2P63PY1
- Paperback is also selectable from the same Amazon listing.

### Author Bio

James Buchanan is a school principal and past national examiner who has spent over 30 years in education as a teacher in the sciences. He holds a BSc (Hons) in Biology, an MEd in self-regulated learning, and is currently working towards a PhD in study skills. James has an enduring passion for helping students find fulfilment in their schoolwork and success in learning. He lives in Cape Town with his wife, and has two grown children. This is his second book.

### Images

| File | Status | Notes |
|---|---|---|
| `public/images/book-cover.png` | ✅ In repo | Clean high-res cover art — usable |
| `public/images/workbook-cover.png` | ✅ In repo | Clean high-res cover art — usable |
| Author photo | ⚠️ Needs sourcing | Current version embedded in old-site composite. Need standalone high-res photo of James |
| Old site banners (hero composites) | ❌ Do not use | "NOW AVAILABLE" ribbon, grey composite background — not suitable for new site |

### Download Files

| File | Status | Source |
|---|---|---|
| Study Planner Exam Template | ✅ In repo (`public/downloads/`) | Copied from local Downloads folder |
| Student Workbook Guide (answers & questions) | ⚠️ Needs downloading | https://thestudysmarterhub.com/_resources/Workbook%20answers%20and%20guide.pdf |
| Teacher Reflection Guide | ❌ Needs redoing | Current PDF at old site is out of date — new version required before launch |

### Workbook Preview

Interactive preview of the Student Workbook (for use on shop/resources page if desired):
https://s3.af-south-1.amazonaws.com/mags.digimaghost.com/StudySmarterStudentWorkbook.html#p=1

---

## Third-Party Account Setup Needed

Before implementation can be completed, the following accounts must be created:

1. **PayFast merchant account** — [payfast.io](https://www.payfast.io). Required for `merchant_id` and `merchant_key`. Sandbox available for testing. Claude Code can assist with the setup and integration steps.
2. **Formspree account** — [formspree.io](https://formspree.io). Free tier. Required for the contact form endpoint. Claude Code can assist.
3. **GitHub account / repo** — Required for GitHub Pages hosting. Repo should be public for free Pages hosting.

---

## Out of Scope (This Phase)

- AdviceBytes / blog section — to be designed separately
- Bundle deal (book + workbook) — future consideration
- User accounts / order history
- ITN / webhook payment verification
- CMS for content editing

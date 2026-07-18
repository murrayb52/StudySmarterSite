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
2. **Physical products** — two `BookCard` components side-by-side (responsive: stack on mobile). Each card: cover image, title, short description, price, "Buy Now" button (PayFast form submit).
3. **eBook section** — visually separated (different background or rule). Heading "Prefer an eBook?", cover image, one-line description, "Buy on Amazon" link (opens in new tab). Clear off-site indicator.
4. **Footer** — as above.

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

Simple confirmation page: "Thank you for your order." with a note that PayFast will send a receipt and the book will be dispatched. Links back to Home and Resources.

---

## PayFast Integration

**Flow:** Buy Now button → HTML form POST to PayFast gateway → payment → redirect to `/thank-you`.

**Form fields per product:**
- `merchant_id` — from PayFast merchant account
- `merchant_key` — from PayFast merchant account
- `return_url` — `https://thestudysmarterhub.com/thank-you`
- `cancel_url` — `https://thestudysmarterhub.com/shop`
- `amount` — product price (e.g. `250.00`)
- `item_name` — e.g. "Study Smarter Book" / "Study Smarter Student Workbook"

**No ITN (notify_url):** Static site cannot receive server callbacks. PayFast emails the merchant on each successful transaction — sufficient for physical goods dispatch.

**Sandbox testing:** PayFast provides a sandbox environment. The `BookCard` component will accept an `env` prop or the Astro build will use an environment variable to toggle sandbox vs live endpoint.

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
- **Download files** — Study Planner (Excel), Student Workbook Guide (PDF), Teacher Reflection Guide (PDF)

---

## Out of Scope (This Phase)

- AdviceBytes / blog section — to be designed separately
- Bundle deal (book + workbook) — future consideration
- User accounts / order history
- ITN / webhook payment verification
- CMS for content editing

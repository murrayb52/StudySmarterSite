# Study Smarter Hub Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and deploy a static Astro site for The Study Smarter Hub at thestudysmarterhub.com with a shop (PayFast physical orders, Amazon eBook link), free resource downloads, author bio page, and a footer contact form.

**Architecture:** Astro 5 static site with file-based routing, a shared `Base.astro` layout wrapping `Nav` and `Footer`, reusable `BookCard` and `DownloadCard` components, and a client-side JS order form that chains a Formspree POST with a PayFast form redirect. GitHub Actions CI deploys to GitHub Pages on every push to `main`.

**Tech Stack:** Astro 5, TypeScript (strict), vanilla CSS (custom properties), Google Fonts (Source Serif 4 + Nunito), Formspree (contact + order forms), PayFast (form-redirect, sandbox toggle via env var), GitHub Pages + GitHub Actions

## Global Constraints

- No italic anywhere — weight contrast (600 vs 300) only; headings use Source Serif 4 only
- Colours: bg `#FAFAF8`, surface/cards `#F0EDE8`, accent/CTA `#C8601A`, nav bg `#4A6478`, body text `#1A1A1A`, muted text `#6B6B65`, decorative blue `#7B9EB5`
- Book price: R350; delivery: R80 flat rate nationwide; workbook price: TBD
- All credentials from `PUBLIC_*` env vars — never hardcoded
- Brand: "The Study Smarter Hub"; copyright "© 2026 The Study Smarter Hub"; zero Headland Media mentions
- Astro serves `public/` from site root — asset paths are `/images/<file>` and `/downloads/<file>`
- Collection option shown as "Collection in Diep River, Cape Town"; exact street address emailed by merchant after payment — never shown on site

---

## File Map

| File | Purpose |
|---|---|
| `package.json` | Astro + sitemap dependency |
| `astro.config.mjs` | Site URL, sitemap integration |
| `tsconfig.json` | Extends Astro strict |
| `.env.example` | Documents required env vars |
| `.github/workflows/deploy.yml` | Build + deploy to GitHub Pages |
| `public/CNAME` | Custom domain for GitHub Pages |
| `src/styles/global.css` | CSS custom properties, reset, utilities (btn, eyebrow, container, hidden) |
| `src/layouts/Base.astro` | HTML shell, Google Fonts, meta/SEO, imports Nav + Footer |
| `src/components/Nav.astro` | Sticky header, logo, nav links, mobile hamburger |
| `src/components/Footer.astro` | Footer nav, Formspree contact form, copyright |
| `src/components/BookCard.astro` | Cover, title, description, price, CTA — used on Home and Shop |
| `src/components/DownloadCard.astro` | Title, description, file-type badge, download link or "coming soon" |
| `src/pages/index.astro` | Home: hero, about-the-books, product strip, author teaser |
| `src/pages/shop.astro` | Shop: product cards + inline order forms + Amazon eBook section |
| `src/pages/resources.astro` | Resources: three download cards |
| `src/pages/about.astro` | About: author photo + bio |
| `src/pages/thank-you.astro` | Post-payment: conditional delivery / collection messaging |

---

## Task 1: Project Scaffold

**Files:**
- Create: `package.json`
- Create: `astro.config.mjs`
- Create: `tsconfig.json`
- Create: `.env.example`
- Modify: `.gitignore`

- [ ] **Step 1: Write `package.json`**

```json
{
  "name": "study-smarter-hub",
  "type": "module",
  "version": "0.1.0",
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "check": "astro check"
  },
  "dependencies": {
    "@astrojs/sitemap": "^3.0.0",
    "astro": "^5.0.0"
  },
  "devDependencies": {
    "@astrojs/check": "^0.9.0",
    "typescript": "^5.0.0"
  }
}
```

- [ ] **Step 2: Write `astro.config.mjs`**

```js
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://thestudysmarterhub.com',
  integrations: [sitemap()],
});
```

- [ ] **Step 3: Write `tsconfig.json`**

```json
{
  "extends": "astro/tsconfigs/strict"
}
```

- [ ] **Step 4: Write `.env.example`**

```
# PayFast merchant credentials — obtain from payfast.io
PUBLIC_PAYFAST_MERCHANT_ID=
PUBLIC_PAYFAST_MERCHANT_KEY=
# Set to "live" when ready for real transactions
PUBLIC_PAYFAST_ENV=sandbox

# Formspree form endpoints — obtain from formspree.io
PUBLIC_FORMSPREE_CONTACT_ENDPOINT=
PUBLIC_FORMSPREE_ORDER_ENDPOINT=
```

- [ ] **Step 5: Add to `.gitignore`** (append; do not replace existing content)

```
# Astro
dist/
.astro/
node_modules/

# Environment
.env
```

- [ ] **Step 6: Install dependencies**

```bash
cd /home/muzz/project/StudySmarterSite
npm install
```

Expected: `node_modules/` created, no errors.

- [ ] **Step 7: Verify scaffold**

```bash
npm run check
```

Expected: "Found 0 errors" (no source files yet, this just confirms TypeScript tooling works).

- [ ] **Step 8: Commit**

```bash
git add package.json package-lock.json astro.config.mjs tsconfig.json .env.example .gitignore
git commit -m "feat: scaffold Astro project"
```

---

## Task 2: Global Styles and Base Layout

**Files:**
- Create: `src/styles/global.css`
- Create: `src/layouts/Base.astro`

**Interfaces:**
- Produces: `Base.astro` — accepts props `title: string`, `description: string`; imports `Nav` and `Footer` (stubs acceptable until those tasks complete)

- [ ] **Step 1: Create `src/styles/global.css`**

```css
:root {
  --c-bg: #FAFAF8;
  --c-surface: #F0EDE8;
  --c-accent: #C8601A;
  --c-nav: #4A6478;
  --c-text: #1A1A1A;
  --c-text-muted: #6B6B65;
  --c-blue: #7B9EB5;
  --f-serif: 'Source Serif 4', Georgia, serif;
  --f-sans: 'Nunito', system-ui, sans-serif;
  --max-w: 1100px;
}

*, *::before, *::after { box-sizing: border-box; }

html { font-size: 16px; }

body {
  background: var(--c-bg);
  color: var(--c-text);
  font-family: var(--f-sans);
  font-weight: 400;
  line-height: 1.75;
  margin: 0;
}

h1, h2, h3, h4, h5, h6 {
  font-family: var(--f-serif);
  font-style: normal;
  line-height: 1.2;
  margin: 0 0 1rem;
}

p { margin: 0 0 1rem; }
p:last-child { margin-bottom: 0; }

a { color: var(--c-accent); text-decoration: none; }
a:hover { text-decoration: underline; }

img { max-width: 100%; height: auto; display: block; }

.container {
  max-width: var(--max-w);
  margin: 0 auto;
  padding: 0 1.5rem;
}

.section { padding: 4rem 0; }

.section-alt { background: var(--c-surface); }

.eyebrow {
  display: block;
  font-family: var(--f-sans);
  font-weight: 400;
  font-size: 0.8125rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--c-blue);
  margin-bottom: 0.5rem;
}

.btn {
  display: inline-block;
  padding: 0.75rem 1.75rem;
  border-radius: 3px;
  font-family: var(--f-sans);
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  border: none;
  text-decoration: none;
  transition: opacity 0.15s;
  line-height: 1;
}
.btn:hover { opacity: 0.85; text-decoration: none; }
.btn-primary { background: var(--c-accent); color: #fff; }
.btn-outline {
  background: transparent;
  color: var(--c-accent);
  border: 2px solid var(--c-accent);
}

.hidden { display: none !important; }
```

- [ ] **Step 2: Create `src/layouts/Base.astro`**

```astro
---
import '../styles/global.css';

export interface Props {
  title: string;
  description: string;
  canonical?: string;
}

const { title, description, canonical } = Astro.props;
const siteUrl = 'https://thestudysmarterhub.com';
const canonicalUrl = canonical ?? new URL(Astro.url.pathname, siteUrl).href;
const pageTitle = `${title} | The Study Smarter Hub`;
---

<!DOCTYPE html>
<html lang="en-ZA">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{pageTitle}</title>
    <meta name="description" content={description} />
    <link rel="canonical" href={canonicalUrl} />
    <meta property="og:title" content={pageTitle} />
    <meta property="og:description" content={description} />
    <meta property="og:image" content={`${siteUrl}/images/book-cover.png`} />
    <meta property="og:type" content="website" />
    <meta property="og:url" content={canonicalUrl} />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Source+Serif+4:wght@300;600&family=Nunito:wght@400;500;600&display=swap"
      rel="stylesheet"
    />
  </head>
  <body>
    <slot name="nav" />
    <main>
      <slot />
    </main>
    <slot name="footer" />
  </body>
</html>
```

Note: `Nav` and `Footer` are placed via named slots so they can be imported in each page. This avoids circular import issues when building those components later. Each page will do:
```astro
<Base title="..." description="...">
  <Nav slot="nav" />
  <Footer slot="footer" />
  <!-- page content -->
</Base>
```

- [ ] **Step 3: Create a smoke-test page to verify styles and layout load**

Create `src/pages/index.astro` (will be replaced in Task 6 — this is just a build check):

```astro
---
import Base from '../layouts/Base.astro';
---

<Base title="Home" description="Test">
  <div class="container section">
    <span class="eyebrow">Test eyebrow</span>
    <h1><span style="font-weight:600">Study smarter,</span><br/><span style="font-weight:300">not harder.</span></h1>
    <p>Body text in Nunito. <a href="/shop">Link to shop</a>.</p>
    <a href="/shop" class="btn btn-primary">Shop Now</a>
  </div>
</Base>
```

- [ ] **Step 4: Run build check**

```bash
npm run build
```

Expected: `dist/` created, exit 0, no errors.

- [ ] **Step 5: Preview in browser**

```bash
npm run preview
```

Open `http://localhost:4321`. Verify:
- Source Serif 4 loads for the heading (check Network tab — fonts from fonts.gstatic.com)
- Nunito loads for body text
- Heading shows weight difference (600 line vs 300 line) — NO italic
- Background is warm white `#FAFAF8`
- Eyebrow is steel blue, uppercase, small
- Button is burnt orange

- [ ] **Step 6: Commit**

```bash
git add src/ && git commit -m "feat: global styles and Base layout"
```

---

## Task 3: Nav Component

**Files:**
- Create: `src/components/Nav.astro`

**Interfaces:**
- Produces: `<Nav />` — no props; reads `Astro.url.pathname` to highlight current page

- [ ] **Step 1: Create `src/components/Nav.astro`**

```astro
---
const links = [
  { href: '/', label: 'Home' },
  { href: '/shop', label: 'Shop' },
  { href: '/resources', label: 'Resources' },
  { href: '/about', label: 'About' },
];
const current = Astro.url.pathname;
---

<header class="site-header">
  <div class="container nav-inner">
    <a href="/" class="site-logo">The Study Smarter Hub</a>

    <button class="nav-toggle" aria-label="Toggle menu" aria-expanded="false" aria-controls="main-nav">
      <span></span><span></span><span></span>
    </button>

    <nav id="main-nav" class="main-nav">
      <ul>
        {links.map(l => (
          <li>
            <a
              href={l.href}
              class:list={['nav-link', { active: current === l.href || (l.href !== '/' && current.startsWith(l.href)) }]}
            >
              {l.label}
            </a>
          </li>
        ))}
        <li><a href="/shop" class="btn btn-primary nav-cta">Buy Now</a></li>
      </ul>
    </nav>
  </div>
</header>

<script>
  const toggle = document.querySelector<HTMLButtonElement>('.nav-toggle');
  const nav = document.getElementById('main-nav');
  toggle?.addEventListener('click', () => {
    const open = nav?.classList.toggle('open') ?? false;
    toggle.setAttribute('aria-expanded', String(open));
  });
</script>

<style>
  .site-header {
    background: var(--c-nav);
    position: sticky;
    top: 0;
    z-index: 100;
    box-shadow: 0 1px 4px rgba(0,0,0,0.15);
  }
  .nav-inner {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 64px;
  }
  .site-logo {
    font-family: var(--f-serif);
    font-weight: 600;
    font-size: 1.1rem;
    color: #fff;
    text-decoration: none;
    flex-shrink: 0;
  }
  .site-logo:hover { text-decoration: none; opacity: 0.9; }

  .main-nav ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    align-items: center;
    gap: 2rem;
  }
  .nav-link {
    font-family: var(--f-sans);
    font-weight: 600;
    font-size: 0.9375rem;
    color: rgba(255,255,255,0.82);
    text-decoration: none;
    transition: color 0.15s;
  }
  .nav-link:hover,
  .nav-link.active {
    color: #fff;
    text-decoration: none;
  }
  .nav-cta {
    font-size: 0.875rem;
    padding: 0.5rem 1.25rem;
  }

  .nav-toggle {
    display: none;
    flex-direction: column;
    gap: 5px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0.5rem;
  }
  .nav-toggle span {
    display: block;
    width: 24px;
    height: 2px;
    background: #fff;
    border-radius: 2px;
    transition: opacity 0.15s;
  }

  @media (max-width: 768px) {
    .nav-toggle { display: flex; }
    .main-nav {
      display: none;
      position: absolute;
      top: 64px;
      left: 0;
      right: 0;
      background: var(--c-nav);
      padding: 1rem 1.5rem 1.5rem;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    }
    .main-nav.open { display: block; }
    .main-nav ul {
      flex-direction: column;
      align-items: flex-start;
      gap: 1rem;
    }
  }
</style>
```

- [ ] **Step 2: Update `src/pages/index.astro` to use Nav**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
---

<Base title="Home" description="Test">
  <Nav slot="nav" />
  <div class="container section">
    <h1><span style="font-weight:600">Study smarter,</span><br/><span style="font-weight:300">not harder.</span></h1>
    <p>Body text in Nunito.</p>
    <a href="/shop" class="btn btn-primary">Shop Now</a>
  </div>
</Base>
```

- [ ] **Step 3: Build and preview**

```bash
npm run build && npm run preview
```

Open `http://localhost:4321`. Verify:
- Sticky steel-blue nav header (`#4A6478`)
- Logo in Source Serif 4 white
- "Home" link highlighted (active state)
- "Buy Now" button in burnt orange
- Hamburger visible at < 768px; clicking opens nav vertically

- [ ] **Step 4: Commit**

```bash
git add src/components/Nav.astro src/pages/index.astro
git commit -m "feat: Nav component with mobile hamburger"
```

---

## Task 4: Footer Component

**Files:**
- Create: `src/components/Footer.astro`

**Interfaces:**
- Consumes: `PUBLIC_FORMSPREE_CONTACT_ENDPOINT` from `import.meta.env`
- Produces: `<Footer />` — no props; gracefully shows "not yet configured" if endpoint is missing

- [ ] **Step 1: Create `src/components/Footer.astro`**

```astro
---
const footerLinks = [
  { href: '/', label: 'Home' },
  { href: '/shop', label: 'Shop' },
  { href: '/resources', label: 'Resources' },
  { href: '/about', label: 'About' },
];
---

<footer class="site-footer">
  <div class="container footer-grid">

    <div class="footer-brand">
      <p class="footer-logo">The Study Smarter Hub</p>
      <p class="footer-tagline">Study smarter, not harder.</p>
      <p class="footer-email">
        <a href="mailto:info@thestudysmarterhub.com">info@thestudysmarterhub.com</a>
      </p>
    </div>

    <nav class="footer-nav" aria-label="Footer navigation">
      <ul>
        {footerLinks.map(l => (
          <li><a href={l.href}>{l.label}</a></li>
        ))}
      </ul>
    </nav>

    <div class="footer-contact">
      <h3>Get in touch</h3>
      <form id="contact-form" novalidate>
        <div class="field">
          <label for="c-name">Name</label>
          <input type="text" id="c-name" name="name" required autocomplete="name" />
        </div>
        <div class="field">
          <label for="c-email">Email</label>
          <input type="email" id="c-email" name="email" required autocomplete="email" />
        </div>
        <div class="field">
          <label for="c-message">Message</label>
          <textarea id="c-message" name="message" rows="4" required></textarea>
        </div>
        <button type="submit" class="btn btn-primary">Send Message</button>
        <p id="contact-status" class="form-status" aria-live="polite"></p>
      </form>
    </div>

  </div>

  <div class="footer-bottom">
    <div class="container">
      <p>© 2026 The Study Smarter Hub</p>
    </div>
  </div>
</footer>

<script>
  const form = document.getElementById('contact-form') as HTMLFormElement | null;
  const status = document.getElementById('contact-status');
  const endpoint = import.meta.env.PUBLIC_FORMSPREE_CONTACT_ENDPOINT as string | undefined;

  form?.addEventListener('submit', async (e) => {
    e.preventDefault();
    if (!status) return;

    if (!endpoint) {
      status.textContent = 'Contact form is not yet configured. Please email info@thestudysmarterhub.com directly.';
      return;
    }

    status.textContent = 'Sending…';
    const data = new FormData(form);

    try {
      const res = await fetch(endpoint, {
        method: 'POST',
        body: data,
        headers: { Accept: 'application/json' },
      });
      if (res.ok) {
        status.textContent = 'Message sent — we\'ll be in touch shortly.';
        form.reset();
      } else {
        status.textContent = 'Something went wrong. Please try again or email us directly.';
      }
    } catch {
      status.textContent = 'Something went wrong. Please try again or email us directly.';
    }
  });
</script>

<style>
  .site-footer {
    background: var(--c-surface);
    border-top: 1px solid #ddd9d4;
    margin-top: 4rem;
  }

  .footer-grid {
    display: grid;
    grid-template-columns: 1fr 1fr 2fr;
    gap: 3rem;
    padding: 3rem 1.5rem;
  }

  .footer-logo {
    font-family: var(--f-serif);
    font-weight: 600;
    font-size: 1rem;
    margin-bottom: 0.25rem;
  }
  .footer-tagline {
    font-family: var(--f-serif);
    font-weight: 300;
    font-size: 0.9375rem;
    color: var(--c-text-muted);
    margin-bottom: 0.75rem;
  }
  .footer-email a {
    font-size: 0.875rem;
    color: var(--c-text-muted);
  }

  .footer-nav ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  .footer-nav a {
    font-size: 0.9375rem;
    color: var(--c-text-muted);
    text-decoration: none;
  }
  .footer-nav a:hover { color: var(--c-text); text-decoration: underline; }

  .footer-contact h3 {
    font-family: var(--f-sans);
    font-weight: 600;
    font-size: 0.9375rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: var(--c-text-muted);
    margin-bottom: 1rem;
  }

  .field {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    margin-bottom: 0.875rem;
  }
  .field label {
    font-size: 0.875rem;
    font-weight: 500;
    color: var(--c-text-muted);
  }
  .field input,
  .field textarea {
    padding: 0.5rem 0.75rem;
    border: 1px solid #c8c4bf;
    border-radius: 3px;
    background: var(--c-bg);
    font-family: var(--f-sans);
    font-size: 0.9375rem;
    color: var(--c-text);
    resize: vertical;
  }
  .field input:focus,
  .field textarea:focus {
    outline: 2px solid var(--c-accent);
    outline-offset: 1px;
    border-color: var(--c-accent);
  }

  .form-status {
    margin-top: 0.75rem;
    font-size: 0.875rem;
    color: var(--c-text-muted);
    min-height: 1.25rem;
  }

  .footer-bottom {
    border-top: 1px solid #ddd9d4;
    padding: 1rem 0;
  }
  .footer-bottom p {
    font-size: 0.8125rem;
    color: var(--c-text-muted);
    margin: 0;
  }

  @media (max-width: 768px) {
    .footer-grid {
      grid-template-columns: 1fr;
      gap: 2rem;
    }
  }
</style>
```

- [ ] **Step 2: Add Footer to `src/pages/index.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
---

<Base title="Home" description="Test">
  <Nav slot="nav" />
  <Footer slot="footer" />
  <div class="container section">
    <h1><span style="font-weight:600">Study smarter,</span><br/><span style="font-weight:300">not harder.</span></h1>
    <a href="/shop" class="btn btn-primary">Shop Now</a>
  </div>
</Base>
```

- [ ] **Step 3: Build and preview**

```bash
npm run build && npm run preview
```

Verify at `http://localhost:4321`:
- Footer appears with surface background (`#F0EDE8`)
- Three-column layout: brand / nav links / contact form
- Contact form visible; submitting without `PUBLIC_FORMSPREE_CONTACT_ENDPOINT` set shows the fallback message (not a JS error)
- Stacks to one column on narrow viewport

- [ ] **Step 4: Commit**

```bash
git add src/components/Footer.astro src/pages/index.astro
git commit -m "feat: Footer component with Formspree contact form"
```

---

## Task 5: BookCard and DownloadCard Components

**Files:**
- Create: `src/components/BookCard.astro`
- Create: `src/components/DownloadCard.astro`

**Interfaces:**

`BookCard` props:
```ts
{
  title: string;           // e.g. "Study Smarter"
  description: string;
  coverSrc: string;        // e.g. "/images/book-cover.png"
  coverAlt: string;
  price?: string;          // e.g. "R350" — omit for "Price TBD"
  amazonUrl?: string;      // if set, renders Amazon link variant instead of Buy Now
  buyHref?: string;        // if set, renders an anchor; use for Home page (links to /shop)
  buyLabel?: string;       // override button label, default "Buy Now"
  comingSoon?: boolean;    // disables buy action, shows "Coming soon" badge
}
```

`DownloadCard` props:
```ts
{
  title: string;
  description: string;
  fileType: 'PDF' | 'XLS';
  downloadHref?: string;   // omit when comingSoon is true
  comingSoon?: boolean;
}
```

- [ ] **Step 1: Create `src/components/BookCard.astro`**

```astro
---
export interface Props {
  title: string;
  description: string;
  coverSrc: string;
  coverAlt: string;
  price?: string;
  amazonUrl?: string;
  buyHref?: string;
  buyLabel?: string;
  comingSoon?: boolean;
}

const {
  title,
  description,
  coverSrc,
  coverAlt,
  price,
  amazonUrl,
  buyHref,
  buyLabel = 'Buy Now',
  comingSoon = false,
} = Astro.props;
---

<div class="book-card">
  <div class="book-cover">
    <img src={coverSrc} alt={coverAlt} width="200" height="280" loading="lazy" />
  </div>
  <div class="book-info">
    <h3>{title}</h3>
    <p class="book-desc">{description}</p>
    <div class="book-price">
      {comingSoon
        ? <span class="badge-coming-soon">Price coming soon</span>
        : price
          ? <span class="price">{price}</span>
          : null
      }
    </div>
    <div class="book-cta">
      {comingSoon
        ? null
        : amazonUrl
          ? <a href={amazonUrl} target="_blank" rel="noopener noreferrer" class="btn btn-primary">
              {buyLabel} ↗
            </a>
          : buyHref
            ? <a href={buyHref} class="btn btn-primary">{buyLabel}</a>
            : <button type="button" class="btn btn-primary buy-trigger" data-product={title}>
                {buyLabel}
              </button>
      }
    </div>
  </div>
</div>

<style>
  .book-card {
    background: var(--c-surface);
    border-radius: 6px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
  }
  .book-cover {
    background: #e8e4df;
    display: flex;
    justify-content: center;
    padding: 2rem 1.5rem;
  }
  .book-cover img {
    max-height: 260px;
    width: auto;
    box-shadow: 0 4px 16px rgba(0,0,0,0.18);
  }
  .book-info {
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    flex: 1;
  }
  .book-info h3 {
    font-family: var(--f-serif);
    font-weight: 600;
    font-size: 1.25rem;
    margin-bottom: 0.5rem;
  }
  .book-desc {
    color: var(--c-text-muted);
    font-size: 0.9375rem;
    flex: 1;
    margin-bottom: 1rem;
  }
  .book-price {
    margin-bottom: 1rem;
  }
  .price {
    font-family: var(--f-serif);
    font-weight: 600;
    font-size: 1.375rem;
    color: var(--c-text);
  }
  .badge-coming-soon {
    font-size: 0.8125rem;
    font-weight: 600;
    color: var(--c-text-muted);
    background: #e0dcd7;
    border-radius: 3px;
    padding: 0.25rem 0.5rem;
    letter-spacing: 0.03em;
    text-transform: uppercase;
  }
  .book-cta { margin-top: auto; }
</style>
```

- [ ] **Step 2: Create `src/components/DownloadCard.astro`**

```astro
---
export interface Props {
  title: string;
  description: string;
  fileType: 'PDF' | 'XLS';
  downloadHref?: string;
  comingSoon?: boolean;
}

const { title, description, fileType, downloadHref, comingSoon = false } = Astro.props;
---

<div class="download-card">
  <div class="dl-header">
    <h3>{title}</h3>
    <span class={`file-badge file-badge--${fileType.toLowerCase()}`}>{fileType}</span>
  </div>
  <p class="dl-desc">{description}</p>
  <div class="dl-action">
    {comingSoon
      ? <span class="badge-coming-soon">Coming soon</span>
      : downloadHref
        ? <a href={downloadHref} download class="btn btn-outline">Download</a>
        : null
    }
  </div>
</div>

<style>
  .download-card {
    background: var(--c-surface);
    border-radius: 6px;
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }
  .dl-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 0.75rem;
  }
  .dl-header h3 {
    font-family: var(--f-serif);
    font-weight: 600;
    font-size: 1.125rem;
    margin: 0;
  }
  .file-badge {
    flex-shrink: 0;
    font-size: 0.6875rem;
    font-weight: 700;
    letter-spacing: 0.06em;
    text-transform: uppercase;
    padding: 0.2rem 0.5rem;
    border-radius: 2px;
  }
  .file-badge--pdf { background: #f0d0c8; color: #8b2e12; }
  .file-badge--xls { background: #c8e0c8; color: #1a5c1a; }
  .dl-desc {
    color: var(--c-text-muted);
    font-size: 0.9375rem;
    margin: 0;
    flex: 1;
  }
  .dl-action { margin-top: auto; }
  .badge-coming-soon {
    font-size: 0.8125rem;
    font-weight: 600;
    color: var(--c-text-muted);
    background: #e0dcd7;
    border-radius: 3px;
    padding: 0.25rem 0.5rem;
    letter-spacing: 0.03em;
    text-transform: uppercase;
  }
</style>
```

- [ ] **Step 3: Smoke-test both components — add to `src/pages/index.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
import BookCard from '../components/BookCard.astro';
import DownloadCard from '../components/DownloadCard.astro';
---

<Base title="Home" description="Test">
  <Nav slot="nav" />
  <Footer slot="footer" />
  <div class="container section" style="display:grid;grid-template-columns:1fr 1fr;gap:2rem">
    <BookCard
      title="Study Smarter"
      description="A practical guide to studying effectively."
      coverSrc="/images/book-cover.png"
      coverAlt="Study Smarter book cover"
      price="R350"
      buyHref="/shop"
    />
    <BookCard
      title="Study Smarter Student Workbook"
      description="Put the strategies into practice."
      coverSrc="/images/workbook-cover.png"
      coverAlt="Workbook cover"
      comingSoon={true}
    />
  </div>
  <div class="container" style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:1.5rem;margin-top:2rem">
    <DownloadCard
      title="Study Planner"
      description="Plan your exam prep week by week."
      fileType="XLS"
      downloadHref="/downloads/Study Planner Exam Template.xlsx"
    />
    <DownloadCard
      title="Workbook Guide"
      description="Answers and guidance for the student workbook."
      fileType="PDF"
      comingSoon={true}
    />
    <DownloadCard
      title="Teacher Guide"
      description="Support your students' study practices."
      fileType="PDF"
      comingSoon={true}
    />
  </div>
</Base>
```

- [ ] **Step 4: Build and preview**

```bash
npm run build && npm run preview
```

Verify:
- Both book cards render with covers, prices/badges
- Workbook shows "Price coming soon" badge, no buy button
- Download cards show file-type badge in correct colour (PDF red-ish, XLS green-ish)
- "Coming soon" cards show badge, no download button
- Study Planner download link works (file served from `/downloads/`)

- [ ] **Step 5: Commit**

```bash
git add src/components/BookCard.astro src/components/DownloadCard.astro src/pages/index.astro
git commit -m "feat: BookCard and DownloadCard components"
```

---

## Task 6: Home Page

**Files:**
- Modify: `src/pages/index.astro` (replace smoke-test content with real page)

- [ ] **Step 1: Write `src/pages/index.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
import BookCard from '../components/BookCard.astro';
---

<Base
  title="Study Smarter"
  description="A practical, research-backed book for high school students — and everyone who supports them. By James Buchanan, Cape Town."
>
  <Nav slot="nav" />
  <Footer slot="footer" />

  <!-- Hero -->
  <section class="hero">
    <div class="container hero-inner">
      <div class="hero-text">
        <span class="eyebrow">Written by an educator. Built for students.</span>
        <h1>
          <span class="strong">Study smarter,</span><br />
          <span class="soft">not harder.</span>
        </h1>
        <p class="hero-lead">
          A practical guide for high school students that replaces guesswork with evidence-backed strategies.
          Because cramming only works until it doesn't.
        </p>
        <div class="hero-ctas">
          <a href="/shop" class="btn btn-primary">Shop Now</a>
          <a href="/resources" class="btn btn-outline">Free Resources</a>
        </div>
      </div>
      <div class="hero-cover">
        <img
          src="/images/book-cover.png"
          alt="Study Smarter book cover"
          width="300"
          height="420"
          loading="eager"
        />
      </div>
    </div>
  </section>

  <!-- About the books -->
  <section class="section section-alt about-books">
    <div class="container">
      <span class="eyebrow">The books</span>
      <h2><span style="font-weight:600">One method.</span> <span style="font-weight:300">Two formats.</span></h2>
      <p class="about-lead">
        <em>Study Smarter</em> gives students a clear, structured approach to learning — not a list of tips, but a complete
        method developed over thirty years in the classroom and grounded in the science of self-regulated learning.
        The companion Student Workbook puts that method into daily practice.
      </p>
    </div>
  </section>

  <!-- Product strip -->
  <section class="section product-strip">
    <div class="container">
      <div class="product-grid">
        <BookCard
          title="Study Smarter"
          description="Evidence-based strategies for high school students. Covers planning, memory, focus, exam technique, and the mindset shifts that make it all stick."
          coverSrc="/images/book-cover.png"
          coverAlt="Study Smarter book cover"
          price="R350"
          buyHref="/shop"
          buyLabel="Shop Now"
        />
        <BookCard
          title="Study Smarter Student Workbook"
          description="Exercises and weekly planners to put the book's strategies into practice. Designed to be used alongside the main book."
          coverSrc="/images/workbook-cover.png"
          coverAlt="Study Smarter Student Workbook cover"
          comingSoon={true}
        />
      </div>
    </div>
  </section>

  <!-- Author teaser -->
  <section class="section section-alt author-teaser">
    <div class="container author-teaser-inner">
      <div class="author-text">
        <span class="eyebrow">The author</span>
        <h2 style="font-weight:600">James Buchanan</h2>
        <p>
          School principal, past national examiner, and MEd graduate in self-regulated learning —
          James has spent over thirty years helping students find success in learning, not just in exams.
        </p>
        <a href="/about" class="btn btn-outline">Read more</a>
      </div>
    </div>
  </section>
</Base>

<style>
  /* Hero */
  .hero {
    padding: 5rem 0 4rem;
  }
  .hero-inner {
    display: grid;
    grid-template-columns: 1fr auto;
    gap: 4rem;
    align-items: center;
  }
  .hero-text h1 {
    font-size: clamp(2.5rem, 5vw, 3.75rem);
    margin: 0.5rem 0 1.25rem;
  }
  .strong { font-weight: 600; display: block; }
  .soft { font-weight: 300; display: block; }
  .hero-lead {
    font-size: 1.125rem;
    color: var(--c-text-muted);
    max-width: 42ch;
    margin-bottom: 2rem;
  }
  .hero-ctas { display: flex; gap: 1rem; flex-wrap: wrap; }
  .hero-cover img {
    box-shadow: 0 8px 32px rgba(0,0,0,0.2);
    border-radius: 2px;
  }

  /* About the books */
  .about-books h2 {
    font-size: clamp(1.75rem, 3vw, 2.25rem);
    margin: 0.5rem 0 1rem;
  }
  .about-lead {
    max-width: 62ch;
    color: var(--c-text-muted);
    font-size: 1.0625rem;
  }
  .about-lead em { font-style: normal; font-weight: 600; }

  /* Product strip */
  .product-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
  }

  /* Author teaser */
  .author-teaser-inner {
    max-width: 600px;
  }
  .author-teaser h2 {
    font-size: 1.875rem;
    margin: 0.5rem 0 1rem;
  }

  @media (max-width: 768px) {
    .hero-inner {
      grid-template-columns: 1fr;
    }
    .hero-cover {
      display: flex;
      justify-content: center;
      order: -1;
    }
    .hero-cover img { max-height: 300px; width: auto; }
    .product-grid { grid-template-columns: 1fr; }
  }
</style>
```

- [ ] **Step 2: Build and preview**

```bash
npm run build && npm run preview
```

Verify at `http://localhost:4321`:
- Hero: book cover right, text left; "Study smarter," bold / "not harder." light — NO italic
- Two CTAs: burnt-orange "Shop Now" + outline "Free Resources"
- About the books section on surface background
- Product strip: two cards side by side, workbook shows "Price coming soon"
- Author teaser section with "Read more" outline button
- Footer visible at bottom with contact form

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: Home page"
```

---

## Task 7: Resources Page

**Prerequisite:** Download the Workbook Guide PDF from the old site before this task:
```bash
curl -L "https://thestudysmarterhub.com/_resources/Workbook%20answers%20and%20guide.pdf" \
  -o "/home/muzz/project/StudySmarterSite/public/downloads/Workbook answers and guide.pdf"
```

**Files:**
- Create: `src/pages/resources.astro`

- [ ] **Step 1: Create `src/pages/resources.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
import DownloadCard from '../components/DownloadCard.astro';
---

<Base
  title="Free Resources"
  description="Free study tools from The Study Smarter Hub — download the exam planner, workbook guide, and teacher reflection guide."
>
  <Nav slot="nav" />
  <Footer slot="footer" />

  <section class="section">
    <div class="container">
      <span class="eyebrow">For students, parents & teachers</span>
      <h1 style="font-weight:600">Free study tools</h1>
      <p class="resources-intro">
        A set of resources to support students and educators — free to download, ready to use.
      </p>

      <div class="download-grid">
        <DownloadCard
          title="Study Planner Exam Template"
          description="A structured weekly planner to map out exam preparation. Works for any subject combination."
          fileType="XLS"
          downloadHref="/downloads/Study Planner Exam Template.xlsx"
        />
        <DownloadCard
          title="Student Workbook Guide"
          description="Answers and worked examples for the Study Smarter Student Workbook."
          fileType="PDF"
          downloadHref="/downloads/Workbook answers and guide.pdf"
        />
        <DownloadCard
          title="Teacher Reflection Guide"
          description="How to support your students' independent study practices in the classroom."
          fileType="PDF"
          comingSoon={true}
        />
      </div>
    </div>
  </section>
</Base>

<style>
  .resources-intro {
    max-width: 52ch;
    color: var(--c-text-muted);
    margin-bottom: 2.5rem;
  }
  .download-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1.5rem;
  }
  @media (max-width: 900px) {
    .download-grid { grid-template-columns: 1fr 1fr; }
  }
  @media (max-width: 600px) {
    .download-grid { grid-template-columns: 1fr; }
  }
</style>
```

- [ ] **Step 2: Build and preview**

```bash
npm run build && npm run preview
```

Verify at `http://localhost:4321/resources`:
- Three download cards in a grid
- Study Planner card: XLS badge (green), "Download" button, clicking downloads the `.xlsx` file
- Workbook Guide card: PDF badge (red-ish), "Download" button (if PDF was downloaded in prerequisite)
- Teacher Guide card: "Coming soon" badge, no download button

- [ ] **Step 3: Commit**

```bash
git add src/pages/resources.astro public/downloads/
git commit -m "feat: Resources page and workbook guide PDF"
```

---

## Task 8: About Page

**Files:**
- Create: `src/pages/about.astro`

Note: `public/images/` currently has no standalone author photo. The About page is built with a placeholder; swap in `/images/author-photo.jpg` when the standalone photo is provided.

- [ ] **Step 1: Create `src/pages/about.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
---

<Base
  title="About"
  description="James Buchanan is a school principal, past national examiner, and author of Study Smarter — a research-backed guide for high school students."
>
  <Nav slot="nav" />
  <Footer slot="footer" />

  <section class="section">
    <div class="container about-inner">

      <div class="author-photo-col">
        <!-- Replace src with /images/author-photo.jpg when available -->
        <div class="author-photo-placeholder" aria-hidden="true">
          <span>Photo<br/>coming<br/>soon</span>
        </div>
      </div>

      <div class="author-bio-col">
        <span class="eyebrow">About the author</span>
        <h1 style="font-weight:600">James Buchanan</h1>

        <p>
          James Buchanan is a school principal and past national examiner who has spent over thirty years
          in education as a teacher in the sciences. He holds a BSc (Hons) in Biology and an MEd in
          self-regulated learning, and is currently working towards a PhD in study skills.
        </p>
        <p>
          James has an enduring passion for helping students find fulfilment in their schoolwork and
          success in learning. He lives in Cape Town with his wife, and has two grown children.
          <em>Study Smarter</em> is his second book.
        </p>

        <div class="author-credentials">
          <ul>
            <li>BSc (Hons) in Biology</li>
            <li>MEd in self-regulated learning</li>
            <li>PhD candidate in study skills</li>
            <li>School principal</li>
            <li>Past national examiner</li>
            <li>30+ years in education</li>
          </ul>
        </div>

        <a href="/shop" class="btn btn-primary">Get the book</a>
      </div>

    </div>
  </section>
</Base>

<style>
  .about-inner {
    display: grid;
    grid-template-columns: 280px 1fr;
    gap: 4rem;
    align-items: start;
  }

  .author-photo-placeholder {
    width: 100%;
    aspect-ratio: 3 / 4;
    background: var(--c-surface);
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--c-text-muted);
    font-size: 0.875rem;
    text-align: center;
    border: 2px dashed #c8c4bf;
  }

  .author-bio-col h1 {
    font-size: clamp(2rem, 4vw, 2.75rem);
    margin-bottom: 1.5rem;
  }

  .author-bio-col p {
    color: var(--c-text-muted);
    font-size: 1.0625rem;
    max-width: 58ch;
  }

  .author-bio-col em { font-style: normal; font-weight: 600; }

  .author-credentials {
    background: var(--c-surface);
    border-radius: 6px;
    padding: 1.25rem 1.5rem;
    margin: 1.5rem 0 2rem;
  }
  .author-credentials ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem 1.5rem;
  }
  .author-credentials li {
    font-size: 0.9375rem;
    color: var(--c-text-muted);
    padding-left: 1rem;
    position: relative;
  }
  .author-credentials li::before {
    content: '–';
    position: absolute;
    left: 0;
    color: var(--c-blue);
  }

  @media (max-width: 768px) {
    .about-inner {
      grid-template-columns: 1fr;
      gap: 2rem;
    }
    .author-photo-placeholder {
      max-width: 240px;
    }
  }
</style>
```

- [ ] **Step 2: Build and preview**

```bash
npm run build && npm run preview
```

Verify at `http://localhost:4321/about`:
- Two-column layout: photo placeholder left, bio right
- Bio text in Nunito, headings in Source Serif 4 weight 600
- Credentials listed in surface-coloured box
- "Get the book" button in burnt orange links to `/shop`
- Stacks to one column on mobile

- [ ] **Step 3: Commit**

```bash
git add src/pages/about.astro
git commit -m "feat: About page with author bio"
```

---

## Task 9: Shop Page with Order Form

**Files:**
- Create: `src/pages/shop.astro`

**Interfaces:**
- Consumes: `PUBLIC_PAYFAST_MERCHANT_ID`, `PUBLIC_PAYFAST_MERCHANT_KEY`, `PUBLIC_PAYFAST_ENV`, `PUBLIC_FORMSPREE_ORDER_ENDPOINT` from `import.meta.env`
- `BookCard` with no `buyHref` and no `amazonUrl` renders a `<button class="buy-trigger" data-product="...">` — the order form JS listens for clicks on these

- [ ] **Step 1: Create `src/pages/shop.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
import BookCard from '../components/BookCard.astro';
---

<Base
  title="Shop"
  description="Buy Study Smarter and the Student Workbook. R350 physical copy with delivery across South Africa, or on Kindle via Amazon."
>
  <Nav slot="nav" />
  <Footer slot="footer" />

  <!-- Physical books -->
  <section class="section">
    <div class="container">
      <span class="eyebrow">Physical copies</span>
      <h1 style="font-weight:600">Get the book</h1>
      <p class="shop-intro">
        Delivered anywhere in South Africa (R80 flat rate), or collect in Diep River, Cape Town.
      </p>

      <div class="shop-grid">

        <!-- Book card -->
        <div class="shop-item">
          <BookCard
            title="Study Smarter"
            description="Evidence-based strategies for high school students. Covers planning, memory, focus, exam technique, and the mindset shifts that make it all stick."
            coverSrc="/images/book-cover.png"
            coverAlt="Study Smarter book cover"
            price="R350"
          />
          <!-- Order form for this product -->
          <div class="order-form-wrap hidden" id="form-study-smarter" data-product="Study Smarter" data-price="350">
            <form class="order-form" novalidate>
              <h3>Your details</h3>
              <div class="form-row two-col">
                <div class="field">
                  <label for="sf-first">First name</label>
                  <input type="text" id="sf-first" name="first_name" required autocomplete="given-name" />
                </div>
                <div class="field">
                  <label for="sf-last">Last name</label>
                  <input type="text" id="sf-last" name="last_name" required autocomplete="family-name" />
                </div>
              </div>
              <div class="form-row two-col">
                <div class="field">
                  <label for="sf-email">Email</label>
                  <input type="email" id="sf-email" name="email" required autocomplete="email" />
                </div>
                <div class="field">
                  <label for="sf-phone">Phone</label>
                  <input type="tel" id="sf-phone" name="phone" required autocomplete="tel" />
                </div>
              </div>

              <h3>Delivery or collection?</h3>
              <div class="fulfilment-options">
                <label class="radio-option">
                  <input type="radio" name="fulfilment" value="delivery" checked />
                  <span>Delivery (R80 nationwide)</span>
                </label>
                <label class="radio-option">
                  <input type="radio" name="fulfilment" value="collection" />
                  <span>Collection in Diep River, Cape Town (exact address emailed after payment)</span>
                </label>
              </div>

              <div class="delivery-fields" id="sf-delivery-fields">
                <div class="field">
                  <label for="sf-addr">Street address</label>
                  <input type="text" id="sf-addr" name="address" autocomplete="street-address" />
                </div>
                <div class="form-row two-col">
                  <div class="field">
                    <label for="sf-suburb">Suburb</label>
                    <input type="text" id="sf-suburb" name="suburb" />
                  </div>
                  <div class="field">
                    <label for="sf-city">City</label>
                    <input type="text" id="sf-city" name="city" autocomplete="address-level2" />
                  </div>
                </div>
                <div class="form-row two-col">
                  <div class="field">
                    <label for="sf-province">Province</label>
                    <input type="text" id="sf-province" name="province" />
                  </div>
                  <div class="field">
                    <label for="sf-postal">Postal code</label>
                    <input type="text" id="sf-postal" name="postal_code" autocomplete="postal-code" />
                  </div>
                </div>
              </div>

              <div class="order-summary" id="sf-summary">
                <span>Book: R350</span>
                <span id="sf-shipping-line">+ Delivery: R80</span>
                <strong id="sf-total">Total: R430</strong>
              </div>

              <button type="submit" class="btn btn-primary">Proceed to Payment</button>
              <p class="form-status" id="sf-status" aria-live="polite"></p>
            </form>
          </div>
        </div>

        <!-- Workbook card -->
        <div class="shop-item">
          <BookCard
            title="Study Smarter Student Workbook"
            description="Exercises and weekly planners to put the book's strategies into daily practice. Designed to be used alongside the main book."
            coverSrc="/images/workbook-cover.png"
            coverAlt="Study Smarter Student Workbook cover"
            comingSoon={true}
          />
        </div>

      </div>
    </div>
  </section>

  <!-- Amazon eBook -->
  <section class="section section-alt">
    <div class="container ebook-section">
      <div class="ebook-cover">
        <img
          src="/images/book-cover.png"
          alt="Study Smarter book cover"
          width="160"
          height="220"
          loading="lazy"
        />
      </div>
      <div class="ebook-text">
        <span class="eyebrow">Digital edition</span>
        <h2 style="font-weight:600">Prefer an eBook?</h2>
        <p>
          The Kindle edition is available on Amazon — paperback is also selectable from the same listing.
          Opens in a new tab.
        </p>
        <a
          href="https://www.amazon.com/Study-Smarter-Stone-age-Wisdoms-Journey-ebook/dp/B0D2P63PY1"
          target="_blank"
          rel="noopener noreferrer"
          class="btn btn-primary"
        >
          Buy on Amazon ↗
        </a>
      </div>
    </div>
  </section>
</Base>

<script>
  // Toggle order form visibility when "Buy Now" is clicked
  document.querySelectorAll<HTMLButtonElement>('.buy-trigger').forEach((btn) => {
    btn.addEventListener('click', () => {
      const productName = btn.dataset.product ?? '';
      const slug = productName.toLowerCase().replace(/\s+/g, '-');
      const formWrap = document.getElementById(`form-${slug}`);
      if (!formWrap) return;
      const isHidden = formWrap.classList.contains('hidden');
      // Close any open forms first
      document.querySelectorAll('.order-form-wrap').forEach(w => w.classList.add('hidden'));
      if (isHidden) {
        formWrap.classList.remove('hidden');
        formWrap.scrollIntoView({ behavior: 'smooth', block: 'start' });
      }
    });
  });

  // Toggle delivery address fields
  document.querySelectorAll<HTMLInputElement>('input[name="fulfilment"]').forEach((radio) => {
    radio.addEventListener('change', () => {
      const form = radio.closest('form');
      if (!form) return;
      const deliveryFields = form.querySelector<HTMLElement>('[id$="-delivery-fields"]');
      const shippingLine = form.querySelector<HTMLElement>('[id$="-shipping-line"]');
      const totalEl = form.querySelector<HTMLElement>('[id$="-total"]');
      const formWrap = form.closest<HTMLElement>('.order-form-wrap');
      const price = Number(formWrap?.dataset.price ?? 0);
      const isDelivery = radio.value === 'delivery';

      if (deliveryFields) deliveryFields.classList.toggle('hidden', !isDelivery);
      if (shippingLine) shippingLine.classList.toggle('hidden', !isDelivery);
      if (totalEl) {
        const total = isDelivery ? price + 80 : price;
        totalEl.textContent = `Total: R${total}`;
      }
    });
  });

  // Order form submission
  const MERCHANT_ID = import.meta.env.PUBLIC_PAYFAST_MERCHANT_ID as string | undefined;
  const MERCHANT_KEY = import.meta.env.PUBLIC_PAYFAST_MERCHANT_KEY as string | undefined;
  const PAYFAST_ENV = (import.meta.env.PUBLIC_PAYFAST_ENV as string | undefined) ?? 'sandbox';
  const ORDER_ENDPOINT = import.meta.env.PUBLIC_FORMSPREE_ORDER_ENDPOINT as string | undefined;

  const PAYFAST_URL = PAYFAST_ENV === 'live'
    ? 'https://www.payfast.co.za/eng/process'
    : 'https://sandbox.payfast.co.za/eng/process';

  document.querySelectorAll<HTMLFormElement>('.order-form').forEach((form) => {
    const formWrap = form.closest<HTMLElement>('.order-form-wrap');
    const status = form.querySelector<HTMLElement>('[id$="-status"]');

    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      if (status) status.textContent = 'Processing…';

      const data = new FormData(form);
      const isCollection = data.get('fulfilment') === 'collection';
      const price = Number(formWrap?.dataset.price ?? 0);
      const product = formWrap?.dataset.product ?? '';
      const shippingAmount = isCollection ? 0 : 80;
      const totalAmount = price + shippingAmount;
      const returnUrl = isCollection
        ? `${window.location.origin}/thank-you?collection=true`
        : `${window.location.origin}/thank-you`;

      // Step 1: Record order with Formspree
      if (ORDER_ENDPOINT) {
        const orderData = new FormData(form);
        orderData.append('product', product);
        orderData.append('price', `R${price}`);
        orderData.append('shipping', isCollection ? 'Collection' : `R${shippingAmount}`);
        orderData.append('total', `R${totalAmount}`);
        try {
          await fetch(ORDER_ENDPOINT, {
            method: 'POST',
            body: orderData,
            headers: { Accept: 'application/json' },
          });
        } catch {
          // Non-fatal — continue to PayFast regardless
        }
      }

      // Step 2: Redirect to PayFast
      if (!MERCHANT_ID || !MERCHANT_KEY) {
        if (status) status.textContent = 'Payment is not yet configured. Please contact info@thestudysmarterhub.com to order.';
        return;
      }

      const payfastForm = document.createElement('form');
      payfastForm.method = 'POST';
      payfastForm.action = PAYFAST_URL;

      const fields: Record<string, string> = {
        merchant_id: MERCHANT_ID,
        merchant_key: MERCHANT_KEY,
        return_url: returnUrl,
        cancel_url: `${window.location.origin}/shop`,
        amount: totalAmount.toFixed(2),
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

      Object.entries(fields).forEach(([name, value]) => {
        const input = document.createElement('input');
        input.type = 'hidden';
        input.name = name;
        input.value = value;
        payfastForm.appendChild(input);
      });

      document.body.appendChild(payfastForm);
      payfastForm.submit();
    });
  });
</script>

<style>
  .shop-intro {
    color: var(--c-text-muted);
    margin-bottom: 2.5rem;
    max-width: 52ch;
  }
  .shop-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
  }
  .shop-item {
    display: flex;
    flex-direction: column;
    gap: 0;
  }

  /* Order form */
  .order-form-wrap {
    background: #fff;
    border: 1px solid #ddd9d4;
    border-top: none;
    border-radius: 0 0 6px 6px;
    padding: 2rem;
  }
  .order-form h3 {
    font-family: var(--f-sans);
    font-weight: 600;
    font-size: 0.9375rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: var(--c-text-muted);
    margin-bottom: 1rem;
  }
  .form-row.two-col {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
  .field {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    margin-bottom: 0.875rem;
  }
  .field label {
    font-size: 0.875rem;
    font-weight: 500;
    color: var(--c-text-muted);
  }
  .field input {
    padding: 0.5rem 0.75rem;
    border: 1px solid #c8c4bf;
    border-radius: 3px;
    background: var(--c-bg);
    font-family: var(--f-sans);
    font-size: 0.9375rem;
    color: var(--c-text);
  }
  .field input:focus {
    outline: 2px solid var(--c-accent);
    outline-offset: 1px;
    border-color: var(--c-accent);
  }
  .fulfilment-options {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    margin-bottom: 1.25rem;
  }
  .radio-option {
    display: flex;
    align-items: flex-start;
    gap: 0.5rem;
    font-size: 0.9375rem;
    cursor: pointer;
  }
  .radio-option input { margin-top: 3px; flex-shrink: 0; }
  .delivery-fields { margin-bottom: 1.25rem; }
  .order-summary {
    background: var(--c-surface);
    border-radius: 4px;
    padding: 1rem 1.25rem;
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    font-size: 0.9375rem;
    color: var(--c-text-muted);
    margin-bottom: 1.25rem;
  }
  .order-summary strong {
    color: var(--c-text);
    font-size: 1rem;
    margin-top: 0.25rem;
  }
  .form-status {
    margin-top: 0.75rem;
    font-size: 0.875rem;
    color: var(--c-text-muted);
    min-height: 1.25rem;
  }

  /* Amazon eBook section */
  .ebook-section {
    display: flex;
    gap: 3rem;
    align-items: center;
  }
  .ebook-cover img {
    box-shadow: 0 4px 16px rgba(0,0,0,0.15);
    border-radius: 2px;
  }
  .ebook-text h2 { font-size: 1.875rem; margin: 0.5rem 0 1rem; }
  .ebook-text p { color: var(--c-text-muted); max-width: 44ch; margin-bottom: 1.5rem; }

  @media (max-width: 768px) {
    .shop-grid { grid-template-columns: 1fr; }
    .form-row.two-col { grid-template-columns: 1fr; }
    .ebook-section { flex-direction: column; }
    .ebook-cover { display: none; }
  }
</style>
```

- [ ] **Step 2: Build and preview**

```bash
npm run build && npm run preview
```

Verify at `http://localhost:4321/shop`:
- Two product cards side by side; workbook shows "Price coming soon", no buy button
- Clicking "Buy Now" on Study Smarter reveals the order form below the card
- Clicking again collapses it
- Selecting "Collection" hides delivery address fields and removes delivery from summary; "Total: R350"
- Selecting "Delivery" shows address fields; "Total: R430"
- Submitting without `PUBLIC_PAYFAST_MERCHANT_ID` set shows the fallback contact message
- Amazon eBook section visible below physical books; link opens in new tab

- [ ] **Step 3: Commit**

```bash
git add src/pages/shop.astro
git commit -m "feat: Shop page with order form and PayFast integration"
```

---

## Task 10: Thank You Page

**Files:**
- Create: `src/pages/thank-you.astro`

- [ ] **Step 1: Create `src/pages/thank-you.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';
---

<Base
  title="Thank you for your order"
  description="Your order has been received. Thank you for supporting The Study Smarter Hub."
>
  <Nav slot="nav" />
  <Footer slot="footer" />

  <section class="section">
    <div class="container thankyou-inner">

      <div class="thankyou-icon" aria-hidden="true">✓</div>
      <h1 style="font-weight:600">Thank you for your order</h1>

      <!-- Delivery message (shown by default, hidden if collection=true) -->
      <div id="msg-delivery">
        <p>
          Your book will be dispatched to your delivery address.
          PayFast will send you a payment receipt by email.
        </p>
        <p>
          If you have any questions, email us at
          <a href="mailto:info@thestudysmarterhub.com">info@thestudysmarterhub.com</a>.
        </p>
      </div>

      <!-- Collection message (hidden by default, shown if collection=true) -->
      <div id="msg-collection" class="hidden">
        <p>
          Thank you for choosing collection.
          The exact collection address in Diep River, Cape Town will be emailed to you shortly
          once payment is confirmed.
        </p>
        <p>
          If you have any questions, email us at
          <a href="mailto:info@thestudysmarterhub.com">info@thestudysmarterhub.com</a>.
        </p>
      </div>

      <div class="thankyou-links">
        <a href="/" class="btn btn-primary">Back to Home</a>
        <a href="/resources" class="btn btn-outline">Free Resources</a>
      </div>

    </div>
  </section>
</Base>

<script>
  const params = new URLSearchParams(window.location.search);
  if (params.get('collection') === 'true') {
    document.getElementById('msg-delivery')?.classList.add('hidden');
    document.getElementById('msg-collection')?.classList.remove('hidden');
  }
</script>

<style>
  .thankyou-inner {
    max-width: 560px;
    text-align: center;
    padding-top: 2rem;
  }
  .thankyou-icon {
    width: 64px;
    height: 64px;
    border-radius: 50%;
    background: var(--c-accent);
    color: #fff;
    font-size: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 1.5rem;
  }
  .thankyou-inner h1 {
    font-size: 2rem;
    margin-bottom: 1.25rem;
  }
  .thankyou-inner p {
    color: var(--c-text-muted);
    font-size: 1.0625rem;
  }
  .thankyou-links {
    display: flex;
    gap: 1rem;
    justify-content: center;
    margin-top: 2rem;
    flex-wrap: wrap;
  }
</style>
```

- [ ] **Step 2: Build and preview**

```bash
npm run build && npm run preview
```

Verify:
- `http://localhost:4321/thank-you` — shows delivery message
- `http://localhost:4321/thank-you?collection=true` — shows collection message ("Diep River, Cape Town" wording, no street address)
- Both variants have Home + Free Resources buttons
- Orange checkmark circle visible

- [ ] **Step 3: Commit**

```bash
git add src/pages/thank-you.astro
git commit -m "feat: Thank You page with delivery/collection conditional messaging"
```

---

## Task 11: GitHub Pages Deployment

**Files:**
- Create: `.github/workflows/deploy.yml`
- Create: `public/CNAME`

**Prerequisite:** A GitHub repository must exist and the local repo must have a remote set. If not yet done:
```bash
# Create repo on GitHub first (public, no init), then:
git remote add origin https://github.com/<your-username>/study-smarter-hub.git
git push -u origin main
```

Enable GitHub Pages in the repo settings: Settings → Pages → Source → "GitHub Actions".

- [ ] **Step 1: Create `public/CNAME`**

```
thestudysmarterhub.com
```

This tells GitHub Pages to serve the site on the custom domain.

- [ ] **Step 2: Create `.github/workflows/deploy.yml`**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci

      - run: npm run build
        env:
          PUBLIC_PAYFAST_MERCHANT_ID: ${{ secrets.PUBLIC_PAYFAST_MERCHANT_ID }}
          PUBLIC_PAYFAST_MERCHANT_KEY: ${{ secrets.PUBLIC_PAYFAST_MERCHANT_KEY }}
          PUBLIC_PAYFAST_ENV: ${{ vars.PUBLIC_PAYFAST_ENV }}
          PUBLIC_FORMSPREE_CONTACT_ENDPOINT: ${{ secrets.PUBLIC_FORMSPREE_CONTACT_ENDPOINT }}
          PUBLIC_FORMSPREE_ORDER_ENDPOINT: ${{ secrets.PUBLIC_FORMSPREE_ORDER_ENDPOINT }}

      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

- [ ] **Step 3: Final local build check**

```bash
npm run build
```

Expected: `dist/` produced, no errors. Verify `dist/CNAME` exists (Astro copies `public/` into `dist/`).

- [ ] **Step 4: Commit and push**

```bash
git add .github/ public/CNAME
git commit -m "feat: GitHub Pages deploy workflow and CNAME"
git push -u origin main
```

- [ ] **Step 5: Verify CI**

Open the GitHub repo → Actions tab. The "Deploy to GitHub Pages" workflow should run. Wait for green checkmark. Then open `https://thestudysmarterhub.com` (DNS may take a few minutes on first deploy).

After confirming deployment:
- Add GitHub secrets (`PUBLIC_PAYFAST_MERCHANT_ID` etc.) in Settings → Secrets → Actions when credentials are available
- Set `PUBLIC_PAYFAST_ENV` as a variable (not secret) to `sandbox` initially, switch to `live` once tested

---

## Post-Implementation Checklist

- [ ] Swap author photo placeholder (`public/images/author-photo.jpg`) and update `src/pages/about.astro`
- [ ] Update `src/pages/resources.astro` Teacher Reflection Guide card from `comingSoon={true}` to `downloadHref` once new PDF is provided
- [ ] Set workbook price and remove `comingSoon` from workbook `BookCard` in `index.astro` and `shop.astro`
- [ ] Add GitHub repo secrets for PayFast and Formspree once accounts are created; switch `PUBLIC_PAYFAST_ENV` to `live` after sandbox testing
- [ ] Add DNS CNAME record pointing `thestudysmarterhub.com` → `<github-username>.github.io`
- [ ] Test full checkout flow end-to-end in PayFast sandbox before going live

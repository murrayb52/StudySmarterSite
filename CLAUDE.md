# Study Smarter Hub — Claude Code Context

Static marketing site for **The Study Smarter Hub**, built in Astro 5 and deployed to GitHub Pages.
Author: James Buchanan (Cape Town). Domain: `thestudysmarterhub.com` (DNS not yet live).

---

## Running locally

```bash
# Must use Node 20+ — use nvm
export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh" && nvm use 20
cd ~/projects/StudySmarter/StudySmarterSite
npm run dev          # dev server with hot reload at http://localhost:4321/StudySmarterSite/
npm run build        # production build to dist/
npm run preview      # serve the production build locally
```

---

## Tech stack

- **Astro 5** — static site, no SSR
- **GitHub Pages** — deployed via GitHub Actions (`deploy.yml`), source: **GitHub Actions** (not branch)
- **Fonts** — Source Serif 4 (300/600) + Nunito (400/500/600), loaded via Google Fonts in `Base.astro`
- **No component library** — hand-written CSS with scoped `<style>` blocks

---

## BASE URL — critical gotcha

`astro.config.mjs` has `base: '/StudySmarterSite'` for the GitHub Pages sub-path.

**Astro 5 does NOT auto-prefix** `href` or `src` in templates — only its own built `_astro/` assets.
Every internal path must be manually prefixed. Pattern used everywhere:

```astro
---
const b = import.meta.env.BASE_URL.replace(/\/$/, '');
---
<img src={`${b}/images/foo.png`} />
<a href={`${b}/shop`}>Shop</a>
```

`import.meta.env.BASE_URL` = `/StudySmarterSite` (no trailing slash after the replace).

**CSS `url()` inside `<style>` blocks cannot use `import.meta.env`** — move any
background-image that needs the base path to an inline `style` attribute on the HTML element.

---

## Design tokens

```
Background:  #FAFAF8   (--c-bg)
Surface:     #F0EDE8   (--c-surface)
Accent:      #C8601A   (--c-accent)  — orange-brown
Nav:         #4A6478   (--c-nav)     — slate blue
Text:        #1c2a35
Text muted:  #5a6a76
```

Container max-width: **1100px**.

---

## Hero section

The hero image (`fishbowl-leap-wide.png`, 1584×672px) is positioned with:

```css
.hero-bg {
  background-size: auto 130%;
  background-position: -400px 85%;   /* fixed px horizontal, % vertical */
}
```

The gradient and text are **px-relative to the fishbowl** (not %-based), so they stay
anchored to the bowl position as the viewport resizes:

```css
.hero-bg::after {
  background: linear-gradient(to right,
    transparent 490px, rgba(250,250,248,0.72) 620px,
    var(--c-bg) 750px, var(--c-bg) 100%);
}
.hero-text {
  margin-left: 600px;   /* clears the fishbowl; adjust in ~50px increments */
  flex: 1;
}
```

On mobile (`max-width: 768px`) the gradient switches to vertical and `margin-left` resets to `0`.

---

## File structure

```
src/
  layouts/Base.astro          — <html>, fonts, global CSS variables
  components/
    Nav.astro                 — sticky nav; active state uses Astro.url.pathname
    Footer.astro
    BookCard.astro            — product card (book cover + buy button)
    DownloadCard.astro        — free resource card (cover + download button)
  pages/
    index.astro               — home / hero
    shop.astro                — books for sale
    resources.astro           — free downloads
    about.astro               — author bio
    thank-you.astro           — post-purchase / post-download redirect
public/
  images/                     — all static images
  downloads/                  — downloadable files (PDF, XLS)
  .nojekyll                   — prevents Jekyll from stripping _astro/
```

---

## Deployment

GitHub Actions (`deploy.yml`) builds on push to `main` and deploys via `actions/deploy-pages`.
The GitHub Pages **Source** in repo Settings must be set to **"GitHub Actions"** (not a branch).

When the real domain is ready:
1. Add `public/CNAME` containing `thestudysmarterhub.com`
2. Configure DNS A records to GitHub Pages IPs (or CNAME to `murrayb52.github.io`)
3. Remove `base: '/StudySmarterSite'` from `astro.config.mjs` — or keep it if staying on the sub-path

---

## Remaining integrations (not yet done)

### PayFast (payments)
- Register at payfast.co.za
- Add to repo secrets: `PUBLIC_PAYFAST_MERCHANT_ID`, `PUBLIC_PAYFAST_MERCHANT_KEY`
- Set repo variable: `PUBLIC_PAYFAST_ENV` (`sandbox` → `live` when ready)
- The workflow already passes these env vars to the build
- Switch sandbox URLs to live in `shop.astro` when going live

### Formspree (contact / order forms)
- Register at formspree.io, create two forms
- Add to repo secrets: `PUBLIC_FORMSPREE_CONTACT_ENDPOINT`, `PUBLIC_FORMSPREE_ORDER_ENDPOINT`
- The workflow already passes these env vars to the build

---

## Nav active state

`Nav.astro` detects the active page via `Astro.url.pathname`. The guard that prevents Home
from matching every page must compare against the base-prefixed home href, not bare `'/'`:

```astro
{ active: current === l.href || current === `${l.href}/` || (l.href !== `${b}/` && current.startsWith(l.href)) }
```

---

## Known issues / future work

- DNS for `thestudysmarterhub.com` — contact Headland Media for registrar access
- PayFast and Formspree integrations (see above)
- Add CNAME back to `public/` once DNS is configured
- Hero `margin-left` is hardcoded at 600px; on very narrow desktop viewports the text
  may crowd the fishbowl — consider a `min-width` media query if needed

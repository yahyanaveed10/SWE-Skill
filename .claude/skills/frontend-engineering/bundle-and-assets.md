# Bundle and Assets

The network is the bottleneck more often than the CPU. A render-fast UI delivered in a 2MB bundle is a slow UI in practice.

---

## The bundle cost equation

Every kilobyte of JavaScript shipped to the user costs:
1. **Download time** — proportional to file size and network speed
2. **Parse and compile time** — proportional to file size, runs on the user's CPU
3. **Execution time** — initialisation code runs before interactivity
4. **Memory** — held in JS engine memory for the page lifetime

**On a mid-tier mobile device on 4G:** every 100KB of JS adds roughly 0.3-1.0 seconds to time-to-interactive. Bundles in the 1MB+ range produce multi-second delays.

The target for an interactive page bundle on most apps: **under 200KB compressed for the critical path**. Larger is sometimes justified, but should be a deliberate trade-off, not an accident.

---

## How bundles bloat

**Importing whole libraries for one function.**
```javascript
// BAD: imports all of lodash (~70KB)
import _ from 'lodash';
const debounced = _.debounce(fn, 200);

// GOOD: imports only debounce (~2KB)
import debounce from 'lodash/debounce';
const debounced = debounce(fn, 200);

// BETTER: write a 5-line debounce yourself if it's the only lodash function you use
```

**Bundling dev-only code in production.** Test fixtures, debug helpers, mock data — if they are imported by production code paths, they are in the production bundle. Use `process.env.NODE_ENV` checks that the bundler can dead-code-eliminate.

**Unused dependencies in `package.json`.** Some bundlers include them all "just in case." Run a bundle analyser (`webpack-bundle-analyzer`, `source-map-explorer`, `vite-bundle-visualizer`) and look for surprises.

**Polyfills for browsers you don't support.** Polyfills shipped to modern browsers are wasted bytes. Use `<script type="module">` with a separate nomodule fallback, or use a bundler preset that targets only your supported browsers.

**Multiple versions of the same library.** A dependency tree can include React 17 *and* React 18, or three versions of `lodash`. Bundle analyser shows this. Resolve with package.json overrides or `pnpm`/`yarn` resolution.

---

## Tree-shaking

Removing unused exports from the bundle. Works only when:
- The library is published as ESM (not CommonJS)
- Imports are static (`import { x } from 'lib'` — not `require('lib').x`)
- The library does not have side effects in module scope (or marks itself as `sideEffects: false`)

Many libraries advertise tree-shaking but fail one of these conditions. Check the bundle, not the package's marketing.

---

## Code splitting

Loading only the code needed for the current view, lazy-loading the rest.

**Route-based splitting:** each route has its own bundle. Initial page load only includes the entry route. Navigating to another route loads its bundle.

**Component-based splitting:** below-the-fold components, modals, and rarely-used features load lazily.

**The trade-off:** more splits mean smaller initial bundle, but more network requests as the user navigates. Bundle each route as one chunk (or a small number) to balance.

**Don't split too aggressively.** A page that triggers 50 chunk loads in the first 5 seconds is worse than one bundle. The right granularity is "split at navigation boundaries and large interactive features," not "every component is its own chunk."

---

## The network waterfall

Open the Network tab in DevTools. Reload the page. Look at the timeline.

**Signals of a bad waterfall:**
- Long blocking JS at the top of the page (HTML cannot start rendering until the script loads)
- CSS files loaded after critical content (FOUC — flash of unstyled content)
- A render-blocking request to a third-party domain (analytics, ad networks)
- Sequential dependencies: A loads, then triggers B, then triggers C — each a separate round trip
- Images loaded before they're needed (above-the-fold images should load eagerly; below-the-fold should be `loading="lazy"`)

**The fix patterns:**
- `<link rel="preload">` for critical assets you know you need
- `<link rel="preconnect">` for third-party origins to warm up the connection
- `<script async>` or `<script defer>` for non-critical JS
- Inline critical CSS in `<head>` for above-the-fold styling
- `loading="lazy"` for below-the-fold images and iframes

---

## Image optimisation

Images are usually the largest assets on a page.

**Modern formats:** WebP and AVIF are smaller than JPEG/PNG for equivalent quality. Use `<picture>` to serve modern formats with fallback.

**Responsive images:** `srcset` and `sizes` let the browser pick the right size. Don't ship a 4000px wide image to a phone displaying it at 400px.

**Aspect ratio in HTML:** specify `width` and `height` (or CSS `aspect-ratio`) so the browser reserves the layout space, preventing CLS.

**Lazy loading:** `loading="lazy"` on `<img>` defers loading until near viewport. Free with native browser support; no library needed.

---

## Font loading

Fonts are render-blocking by default. The browser waits for the font to load before painting text in that font.

**Approaches:**
- `font-display: swap` — paint with fallback immediately, swap to web font when loaded. Causes a Flash of Unstyled Text (FOUT) but keeps content visible.
- `font-display: optional` — use web font if it loads in the first ~100ms, otherwise stick with fallback. Best for CLS but font may not appear.
- Subsetting — include only the characters used (Latin, Cyrillic, etc.) — dramatically smaller font files.
- Self-host fonts — skip the third-party DNS lookup and connection cost.

---

## CDNs and caching

Static assets (JS, CSS, images, fonts) should be served from a CDN edge close to the user, with aggressive cache headers.

**Cache key by content hash:** `app.a3f4d2.js` — the URL changes when the content changes. Cache for one year (`Cache-Control: public, max-age=31536000, immutable`). When you ship a new version, the URL changes and browsers fetch the new file.

**Don't cache HTML aggressively.** HTML references the hashed asset URLs, so HTML must be fresh. Use a short cache TTL or `no-cache` for HTML.

**Service workers** add another caching layer that survives page reloads — appropriate for offline support and PWAs, complex to get right.

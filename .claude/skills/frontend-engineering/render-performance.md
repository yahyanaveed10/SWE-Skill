# Render Performance

Browser rendering has its own performance model that does not appear in CPU profilers. Understanding it is required to write code that feels fast.

---

## Core Web Vitals

Google's three user-experience metrics. Each measures a specific user-felt aspect of perceived performance. They are the canonical signals for whether a page feels good or feels broken.

**LCP (Largest Contentful Paint)** — time until the largest visible content element renders. The user's perception of "is this page loading?" Target: < 2.5s on the 75th percentile of users.

**CLS (Cumulative Layout Shift)** — sum of unexpected layout shifts during the page's lifetime. The user's perception of "is this page stable?" High CLS is the mis-tap problem: the user goes to click a button, the page shifts, they tap the wrong thing. Target: < 0.1.

**INP (Interaction to Next Paint)** — the longest delay between a user interaction (tap, click, key press) and the next visual update. The user's perception of "is this page responsive?" Replaced FID in 2024 because it captures sustained responsiveness, not just first interaction. Target: < 200ms.

**Measure on real users, not on developer machines.** Lighthouse on a developer laptop measures one device under controlled conditions. Real User Monitoring (RUM) captures the distribution across actual user devices and networks. The 75th percentile is the metric — the median user, not the best user.

---

## The render pipeline

When a browser displays a frame, it runs through phases:

1. **JavaScript** — your code runs, possibly modifying the DOM
2. **Style** — recompute styles for affected elements
3. **Layout** — calculate position and size of elements (also called "reflow")
4. **Paint** — draw pixels for each element
5. **Composite** — assemble the layers into the final frame

Each phase is more expensive than the one before it. Layout is the heaviest. Triggering layout in a loop is a common performance bug.

**Layout thrashing:** reading a layout property (`offsetWidth`, `getBoundingClientRect`, `scrollTop`) forces the browser to flush pending style changes and recompute layout. If a loop reads then writes then reads, each read forces a recalculation:

```javascript
// BAD: forces N layout recalculations
for (const el of elements) {
    el.style.width = (el.offsetWidth + 10) + 'px';  // read forces layout
}

// GOOD: read all, then write all
const widths = elements.map(el => el.offsetWidth);  // single layout
for (let i = 0; i < elements.length; i++) {
    elements[i].style.width = (widths[i] + 10) + 'px';
}
```

**The signal:** any code that mixes reads and writes of layout-affecting properties. Profile shows `Recalculate Style` and `Layout` events firing repeatedly.

---

## CLS — what causes layout shift

**Images and ads without dimensions.** When the image loads, the page reflows to make space. Always specify `width` and `height` (or use CSS `aspect-ratio`).

**Web fonts loading late.** The fallback font has different metrics than the web font; when the web font loads, text reflows. Use `font-display: optional` or load fonts early with `<link rel="preload">`.

**Dynamically inserted content above existing content.** A banner that loads after the page renders pushes everything down. Reserve space for it, or insert it below the fold.

**Animations on layout-affecting properties.** Animating `width`, `height`, `top`, or `left` triggers layout each frame. Animate `transform` and `opacity` instead — these run on the compositor, do not trigger layout.

---

## SSR, hydration, and the modern render strategies

**CSR (Client-Side Rendering):** server returns an empty HTML shell; JavaScript renders the page in the browser. The browser shows nothing until JS loads. Bad LCP, good for highly interactive apps after load.

**SSR (Server-Side Rendering):** server renders the HTML; browser displays it immediately, then JavaScript "hydrates" the page to make it interactive. Good LCP, but hydration cost can hurt INP.

**SSG (Static Site Generation):** HTML is pre-built at deploy time, served from CDN. Best LCP for content that does not change per-request.

**ISR (Incremental Static Regeneration):** SSG with periodic regeneration on the server. Good for content that updates occasionally.

**Streaming SSR:** server starts sending HTML before the full page is ready, browser starts rendering progressively. Best for pages where some content is fast and some is slow.

**The choice depends on:** content freshness needs, interactivity needs, target devices and networks, and how much of the page is per-user-personalised.

---

## Hydration mismatches

The most common SSR bug. Server renders one HTML; client renders different HTML during hydration; framework warns or errors.

**Common causes:**
- Time-dependent values (`new Date()`, `Math.random()`)
- Locale-dependent formatting that differs between server and client locale
- Code paths that branch on `window` or `document` (server has neither)
- Hydrating on data that was not in the SSR response
- Conditional rendering based on user agent or window size

**The fix is structural, not cosmetic.** Suppressing hydration warnings hides the bug; the page still re-renders unnecessarily, hurting INP. The fix is to ensure the server and client render the same output for the same input.

---

## Hydration cost

Even when hydration succeeds, it has a cost. Every interactive component must be:
1. Downloaded as JavaScript
2. Parsed and executed
3. Wired up to the DOM rendered by SSR

For a page with many interactive components, the hydration phase can take seconds, during which the page looks ready but does not respond to clicks.

**Mitigations:**
- **Islands architecture** (Astro, Fresh): only interactive components ship JavaScript; static content is just HTML
- **Selective hydration** (React Server Components, Next.js): mark which components are interactive, others are server-only
- **Lazy hydration:** defer hydration of below-the-fold or non-critical components

---

## What every frontend engineer must measure

Before declaring a UI feature complete:
- **LCP, CLS, INP** at p75 on the actual page, on a representative device
- **Bundle size impact** of the change (see bundle-and-assets.md)
- **Time to Interactive** under throttled CPU and network in browser DevTools
- **Behaviour with throttled network** — what does the user see during the slow load?

If you only test on your own machine on broadband, you are testing the experience of the fastest 10% of users.

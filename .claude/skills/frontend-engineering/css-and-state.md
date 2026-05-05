# CSS Architecture and Client-Side State

Two distinct frontend disciplines that produce the most "I have no idea why this is broken" bugs.

---

## CSS as a global namespace

By default, CSS rules are global. A class `.button` defined in one component affects every element with `class="button"` everywhere on the page. This is the source of most CSS architecture problems.

**The bleed signal:** changing a CSS rule for one component breaks the styling of an unrelated component. Investigation shows the same class name was used elsewhere.

**Specificity cascade problems:** a more-specific selector elsewhere overrides your rule. The `!important` ladder begins. Within a year, the codebase has dozens of `!important` declarations and nobody knows which rules actually apply.

---

## CSS scoping strategies

**CSS Modules:** import CSS as a module; class names are auto-prefixed to be unique per file. `import styles from './Button.module.css'` — `styles.button` becomes `.Button_button_a3f4d`. Eliminates global name collisions.

**Scoped styles (Vue, Svelte single-file components):** styles in a component automatically apply only to that component's elements. Same outcome as CSS Modules with different syntax.

**CSS-in-JS (styled-components, Emotion):** styles defined as JavaScript template literals. Auto-scoped. Trade-off: runtime cost (style generation in the browser) and bundle size impact. Modern approaches (Linaria, Vanilla Extract) extract styles at build time, eliminating the runtime cost.

**Utility-first (Tailwind):** no custom class names; compose styles from a fixed set of utility classes (`p-4 text-lg flex items-center`). Eliminates the cascade problem entirely because all styles are applied at the same specificity level. Trade-off: HTML becomes verbose; semantic class names disappear.

**BEM (Block-Element-Modifier):** convention for naming classes (`.card__title--large`) to make collisions unlikely by discipline. Works without tooling, but requires team consistency. Easy to violate.

**The choice depends on:** team familiarity, framework, and how strict the project is about consistency. Pick one and apply it consistently. Mixing strategies in one codebase produces the worst outcomes.

---

## CSS architecture signals

**`!important` proliferation.** More than a handful of `!important` in production CSS is a signal that the cascade is being fought, not designed. Before adding another, identify why the more-specific rule below it is failing — that is the design problem.

**Deeply nested selectors.** `.sidebar .nav .nav-item a:hover .icon` is a fragile selector. If any element in the chain changes, the rule breaks. Aim for shallow selectors (1-2 levels max).

**The "div soup" pattern.** HTML with no semantic tags, only `<div>` and `<span>` with class names. Hurts accessibility (see accessibility-signals.md), and signals that the component author treated CSS as the only structural concern.

**Inline styles for everything.** A signal that the team gave up on the cascade and now repeats styles per-element. May be appropriate with utility-first CSS or styled-components; otherwise, indicates a missing strategy.

**Container queries vs media queries:** modern CSS supports `@container` queries that respond to the parent's size, not just the viewport. For component libraries that are placed in different layouts, container queries are the right tool. Media queries for layout that depends on the viewport (mobile vs desktop nav).

---

## State: the four kinds

Frontend state lives in different places with different lifecycles. Mixing them up causes most "the UI shows the wrong thing" bugs.

**Server state.** Data fetched from your API. Truth lives on the server. The client has a cached copy that may be stale. Examples: user profile, product list, inventory count. Lifecycle: cached, refetched, possibly invalidated by other client actions.

**URL state.** State that should survive a refresh and be shareable via link. Examples: current page, search query, selected filters, sort order. Lifecycle: tied to the URL; back/forward must work.

**Client state (transient).** State that exists only in the current page session. Examples: form inputs before submission, modal open/closed, hover state. Lifecycle: lost on refresh.

**Persistent client state.** State stored in localStorage, sessionStorage, IndexedDB, or cookies. Examples: theme preference, saved drafts, auth tokens. Lifecycle: survives reload, may be cleared by user.

**The signal of state confusion:** you implement form state in a global state manager and now refresh wipes the form, OR you store filter selections in client state and now the user cannot share a link to filtered results.

---

## Server state — the special case

Server state is the most commonly misunderstood. It is not really "state" in the same sense as client state — it is a cache of remote data that may be stale.

**The wrong pattern (still common):**
```javascript
// Local state holds API data
const [user, setUser] = useState(null);
useEffect(() => {
    fetchUser().then(setUser);
}, []);  // refetches only when component mounts
```

Problems: refetching policy unclear, no shared cache across components, no automatic invalidation, manual loading and error states.

**The right pattern (server state libraries):**
React Query (TanStack Query), SWR, Apollo Client, RTK Query — these handle: caching, deduplication, background refetching, stale-while-revalidate, optimistic updates, mutation invalidation.

```javascript
const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
});
```

The library handles caching, deduplication, retries, and refetch-on-window-focus. You write much less code.

---

## URL state

URL state should round-trip: navigating to a URL with state in it produces the same UI state. Examples that get this wrong:
- A filter UI where the URL does not reflect filter selections (cannot share or bookmark)
- A pagination control that uses local state (refresh returns to page 1)
- A search field that does not put the query in the URL (back button loses the search)

**The discipline:** if the user would expect the back button or a refresh to preserve the state, it belongs in the URL. If they would not, it belongs in client state.

---

## State management library choice

**Use the lightest tool that solves the problem.** Many codebases reach for a global state library (Redux, Zustand, Jotai) for state that should have been local component state, server state, or URL state.

**Decision tree:**
- Server data → server state library (React Query, SWR)
- URL-relevant → URL parameters and a router
- Single component or small tree → component state (`useState`)
- Cross-cutting client state (theme, auth, modal) → light global library (Zustand, Jotai) OR React Context for low-frequency updates
- Complex cross-cutting state with strong consistency requirements → only then, a heavier global library

Most apps need 1-2 of these, not all of them. Code that uses Redux for state that React Query would handle is the most common over-engineering pattern.

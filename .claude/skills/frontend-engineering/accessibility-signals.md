# Accessibility Signals

Accessibility (a11y) is an engineering concern, not a design preference. WCAG compliance has legal implications in many jurisdictions, but more importantly: a UI that is unusable by keyboard, screen reader, or with motion sensitivity is a UI that fails for a meaningful fraction of users.

The signals below are what an engineer can decide while writing code. They do not require accessibility expertise — only awareness of the failure modes.

---

## The four most common a11y failures

**Non-semantic HTML.** A `<div>` styled to look like a button is not a button. Screen readers do not announce it. Keyboard users cannot tab to it. Use `<button>` for buttons, `<a>` for links, `<input>` for inputs, `<table>` for tabular data. Use semantic landmarks (`<nav>`, `<main>`, `<aside>`, `<footer>`) so assistive technology can navigate the page structure.

**No keyboard navigation.** Click handlers attached to `<div>` without keyboard equivalents. Focus traps in modals that don't release. Custom controls without focus indication. Test: can you complete the user flow using only Tab, Shift+Tab, Enter, and Space?

**Insufficient colour contrast.** Text that fails WCAG AA contrast (4.5:1 for body text, 3:1 for large text) is unreadable for users with low vision. Tools: browser DevTools (Lighthouse, axe), online contrast checkers. Designers may not catch this; engineers should verify.

**Images without alt text.** Decorative images need `alt=""` (explicit empty). Informative images need a descriptive alt text. A missing `alt` attribute causes screen readers to read the file name, which is useless. Don't write `alt="image"` or `alt="picture of..."` — describe what the image conveys.

---

## Keyboard navigation discipline

Every interactive element must be reachable and operable by keyboard:

- **Tab** moves to the next focusable element
- **Shift+Tab** moves to the previous
- **Enter** activates buttons and links; submits forms
- **Space** activates buttons; toggles checkboxes
- **Escape** closes modals, menus, and overlays
- **Arrow keys** navigate within composite widgets (menus, tabs, listboxes)

**Focus must always be visible.** The `:focus` outline (or equivalent visual indicator) must not be removed without a replacement. `outline: none` without a `:focus-visible` style is one of the most common a11y bugs.

**Focus order must be logical.** It should follow the visual reading order. CSS that visually rearranges elements (`order`, `flex-direction: row-reverse`) does not change the DOM order, so focus order may not match the visual order. Fix by changing the DOM order, not by adding `tabindex`.

**`tabindex` discipline:**
- `tabindex="0"` — make a non-interactive element focusable in DOM order. Use sparingly; prefer using a semantic element instead.
- `tabindex="-1"` — focusable programmatically (e.g., for focus management) but not in tab order.
- `tabindex="1"` or higher — manually set tab order. **Never use this.** It overrides the natural order, which is almost never what you want.

---

## ARIA — when and when not

ARIA attributes (`role`, `aria-*`) extend semantic information for assistive technology. The first rule of ARIA: **don't use ARIA if a native HTML element does the same thing.**

`<button>` is better than `<div role="button" tabindex="0" onclick="..." onkeydown="...">`.

When ARIA is appropriate:
- A native element does not exist for the widget you are building (combobox, tabs, tree)
- An element needs additional context (an icon-only button needs `aria-label="Close"`)
- Dynamic content needs to announce changes (`aria-live="polite"` for status messages)

ARIA misuse is worse than no ARIA. Common mistakes:
- `role` attributes that contradict the element type (`<button role="link">`)
- `aria-hidden="true"` on focusable elements (focus moves to "hidden" elements; screen reader announces nothing)
- Using `aria-label` on elements that already have visible text labels (creates redundancy)

---

## Form accessibility

Forms are where most a11y bugs originate.

**Every input needs a label.** Either `<label for="id">` paired with `<input id="id">`, or wrap: `<label>Name <input></label>`. Placeholder text is not a label — it disappears when the user types.

**Errors must be associated with the field.** When validation fails, the error message must be programmatically connected to the input (`aria-describedby="error-id"`) so screen readers announce it.

**Required fields must be marked.** `required` attribute (HTML5 native) plus a visible indicator. Don't rely only on colour (red asterisk) — colour-only signals fail for colour-blind users.

**Don't disable form submission until valid.** Disabled submit buttons leave the user wondering what is wrong. Allow submission, then show field-level errors.

---

## Motion and animation

**Respect `prefers-reduced-motion`.** Users with vestibular disorders can be made physically ill by motion. Detect the preference and reduce or remove non-essential animation.

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

Essential animations (loading indicators) can remain; gratuitous animations (parallax scrolling, auto-playing carousels) should be reduced or stopped.

**No content that flashes more than 3 times per second.** This can trigger seizures in users with photosensitive epilepsy. WCAG-mandated.

---

## Testing tools

**Browser DevTools Lighthouse a11y audit:** automated, catches the most common issues. Not exhaustive — many a11y bugs require human testing.

**axe DevTools (browser extension):** more thorough than Lighthouse, fewer false positives.

**Keyboard-only test:** unplug the mouse for 5 minutes. Try to use the page. Most issues become obvious.

**Screen reader test:** VoiceOver (built into macOS, Cmd+F5), NVDA (free, Windows), or Orca (Linux). Five minutes of trying your page reveals issues that automated tools miss.

**Contrast checker:** Lighthouse, WebAIM Contrast Checker, browser DevTools.

Automated tools catch maybe 30% of a11y issues. Human testing catches most of the rest. The biggest gap is in dynamic content and complex widgets — automated tools have no way to know whether your tab widget actually behaves like a tab widget.

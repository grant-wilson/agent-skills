---
description: css-standards conventions for this project. Applies when working with files matching *.{css,scss}.
paths: ["*.{css,scss}"]
---

- All colors, spacing, typography, radii, and shadows come from custom properties
defined once on `:root` (and overridden per theme). Raw hex values or magic
pixel numbers in component styles are a review failure.

```css
:root {
  --color-surface: oklch(98% 0.01 250);
  --space-2: 0.5rem;
  --radius-m: 6px;
}

.card {
  background: var(--color-surface);
  padding: var(--space-2);
  border-radius: var(--radius-m);
}
```
- Lay out with grid and flexbox using `gap` — never margin hacks, floats, or
absolute positioning for flow content. Use logical properties
(`margin-block-start`, `inline-size`, `padding-inline`) instead of physical ones
so layouts work in any writing mode.
- Declare an explicit `@layer` order (e.g. `reset, base, components, utilities`)
so cascade priority is intentional rather than accidental. Use native CSS
nesting one level deep for states and pseudo-elements; deeper nesting is a
smell.

```css
@layer components {
  .button {
    background: var(--color-primary);

    &:hover {
      background: var(--color-primary-hover);
    }
    &:focus-visible {
      outline: 2px solid var(--color-focus);
    }
  }
}
```
- Size components against their container, not the viewport: mark the wrapper with
`container-type: inline-size` and use `@container` rules. Reserve `@media` for
page-level layout and user preferences.
- Keep selectors to a single class where possible; use `:is()`/`:where()` to group
without raising specificity. No ID selectors, no element-qualified classes
(`div.card`), and `!important` only to override third-party styles — with a
comment explaining which ones.
- One convention per codebase, applied everywhere: BEM
(`block__element--modifier`) for global stylesheets, or the framework's
scoped-style mechanism where available. Class names describe purpose
(`.order-summary`), never appearance (`.red-bold`).
- Honor `prefers-reduced-motion` (gate non-essential animation behind it),
`prefers-color-scheme` via the same design tokens, and keep all text/background
pairs at WCAG AA contrast or better.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---
description: html-standards conventions for this project. Applies when working with files matching *.html.
paths: ["*.html"]
---

- Use the element that names the content: `<header>`, `<nav>`, `<main>` (exactly
one per page), `<article>`, `<section>`, `<aside>`, `<footer>`, `<button>` for
actions, `<a>` for navigation. A `<div>` or `<span>` is a last resort for pure
styling hooks, never for interactive behavior.
- Reach for native semantics before ARIA — a `<button>` needs no `role` or key
handlers. When ARIA is unavoidable, follow the APG pattern completely (role,
states, and keyboard support together). Every page must be operable by keyboard
alone, with a logical heading outline (one `<h1>`, no skipped levels) and
visible focus indicators.

```html
<button type="button" aria-expanded="false" aria-controls="filters">
  Filters
</button>
```
- Every `<img>` has an `alt` attribute — descriptive for informative images, empty
(`alt=""`) for decorative ones. Set explicit `width`/`height` (or `aspect-ratio`
in CSS) to prevent layout shift, and `loading="lazy"` on below-the-fold images.
- Every control gets a programmatically associated `<label>`. Use the specific
input type (`email`, `tel`, `date`, `number`) and `autocomplete` tokens so
browsers and password managers can help. Surface validation errors in text
associated via `aria-describedby`, never by color alone.

```html
<label for="work-email">Work email</label>
<input id="work-email" type="email" name="email" autocomplete="work email"
  required
  aria-describedby="work-email-error" />
<p id="work-email-error" role="alert" hidden>Enter a valid email address.</p>
```
- Prefer built-in behavior over JavaScript re-implementations: `<dialog>` with
`showModal()` for modals, `<details>`/`<summary>` for disclosure, the `popover`
attribute for popovers, and `<datalist>` for suggestions.
- No `style` attributes, no inline `<script>` bodies, and no inline event handlers
(`onclick="…"`). Keep markup CSP-compatible: styles live in stylesheets,
behavior in external modules loaded with `type="module"` and `defer` semantics.
- Every document declares `<!doctype html>`, `<html lang="…">`,
`<meta charset="utf-8">`, a `<meta name="viewport">`, and a unique, descriptive
`<title>`.

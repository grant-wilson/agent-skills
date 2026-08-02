# Component Styling

Angular components can define styles that apply specifically to their template, enabling encapsulation and modularity.

## Plain CSS, always

Component styles are plain CSS. No Sass/SCSS, Less, or any other preprocessor —
scaffold with `--style=css` and keep the `schematics` default in `angular.json`
set to `css`. Tailwind CSS is not used.

## Cascade layers

Every component stylesheet wraps its rules in an explicit `@layer` so a
component can never accidentally outrank the design system. View encapsulation
scopes *which elements* a rule matches; it does nothing about *cascade priority*
between a component rule and a global one. Layers are what make that
intentional.

The global stylesheet declares the order once, and because it loads before any
component style is injected, every later `@layer components { … }` joins the
already-ordered layer rather than creating a new one at the end.

```css
/* styles.css — loaded first, declares the order for the whole app */
@layer reset, base, components, utilities;
```

```css
/* photo.css */
@layer components {
  :host {
    display: block;
  }

  img {
    border-radius: var(--radius-full);
  }
}
```

All values come from design tokens (`var(--…)`) defined as custom properties.
Raw hex values and magic pixel numbers in a component stylesheet are a review
failure.

## Defining Styles

Styles can be defined inline or in separate files.

```ts
@Component({
  selector: 'app-photo',
  // Inline styles
  styles: `
    @layer components {
      img {
        border-radius: var(--radius-full);
      }
    }
  `,
  // OR external file
  styleUrl: 'photo.css',
})
export class Photo {}
```

## View Encapsulation

Every component has a view encapsulation setting that determines how styles are scoped.

| Mode                            | Behavior                                                                                      |
| :------------------------------ | :-------------------------------------------------------------------------------------------- |
| `Emulated` (Default)            | Scopes styles to the component using unique HTML attributes. Global styles can still leak in. |
| `ShadowDom`                     | Uses the browser's native Shadow DOM API to isolate styles completely.                        |
| `None`                          | Disables encapsulation. Component styles become global.                                       |
| `ExperimentalIsolatedShadowDom` | Strictly guarantees that only the component's styles apply.                                   |

### Usage

```ts
import { ViewEncapsulation } from '@angular/core';

@Component({
  ...,
  encapsulation: ViewEncapsulation.None,
})
export class GlobalStyled {}
```

## Special Selectors

### `:host`

Targets the component's host element (the element matching the component's selector).

```css
:host {
  display: block;
  border: 1px solid black;
}
```

### `:host-context()`

Targets the host element based on some condition in its ancestry.

```css
/* Apply styles if any ancestor has the 'theme-dark' class */
:host-context(.theme-dark) {
  background-color: #333;
}
```

### `::ng-deep`

Disables view encapsulation for a specific rule, allowing it to "leak" into child components.

**`::ng-deep` is forbidden.** It is deprecated, supported only for backwards
compatibility, and reaching into a child's internals couples the two components
in a way no refactor can see. Give the child a custom property to consume
instead:

```css
/* child.css */
@layer components {
  .panel {
    background: var(--panel-surface, var(--color-surface));
  }
}

/* parent.css — configure the child, don't reach into it */
@layer components {
  app-panel {
    --panel-surface: var(--color-surface-raised);
  }
}
```

## Styles in Templates

You can use `<style>` elements directly in a component's template. View encapsulation rules still apply.

```html
<style>
  .dynamic-class {
    color: red;
  }
</style>
<div class="dynamic-class">Hello</div>
```

## External Styles

Using `<link>` or `@import` in CSS is treated as external styles. **External styles are not affected by emulated view encapsulation.**

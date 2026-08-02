# Side Effects with `effect` and `afterRenderEffect`

In Angular, an **effect** is an operation that runs whenever one or more signal values it tracks change.

## When to use `effect`

Effects are intended for syncing signal state to imperative, non-signal APIs.

**Valid Use Cases:**

- Logging analytics.
- Syncing state to `localStorage` or `sessionStorage`.
- Performing custom rendering to a `<canvas>` or 3rd-party charting library.

**CRITICAL RULE: DO NOT use effects to propagate state.**
If you find yourself using `.set()` or `.update()` on a signal _inside_ an effect to keep two signals in sync, you are making a mistake. This causes `ExpressionChangedAfterItHasBeenChecked` errors and infinite loops. **Always use `computed()` or `linkedSignal()` for state derivation.**

## Every effect has a name

**An `effect()` is never left anonymous inside a constructor.** A constructor
holding a stack of bare `effect(() => …)` calls hides how many side effects a
component has and what each one is for. Name every effect for what it
synchronizes, using whichever of these reads better in context:

- a `#`-private method the constructor calls — best when the effect body is more
  than one line, or needs cleanup;
- a named field holding the `EffectRef` — best when the effect is a one-liner, or
  when the reference is needed later to `destroy()` it.

The effect must still be created in an injection context. A method called
synchronously from the constructor is in one; an `async` method or a callback is
not.

```ts
import { Component, signal, effect, inject } from '@angular/core';

@Component({...})
export class Example {
  readonly #telemetry = inject(TelemetryClient);
  protected readonly count = signal(0);

  constructor() {
    this.#reportCountToTelemetry();
  }

  #reportCountToTelemetry(): void {
    effect((onCleanup) => {
      const handle = this.#telemetry.beginReport(this.count());

      // Cleanup runs before the next execution, or when destroyed
      onCleanup(() => handle.cancel());
    });
  }
}
```

```ts
// One-liner: a named field carries the meaning just as well.
export class ThemeSwitcher {
  readonly #document = inject(DOCUMENT);
  protected readonly theme = signal<Theme>('light');

  readonly #applyThemeToDocument = effect(() =>
    this.#document.documentElement.setAttribute('data-theme', this.theme()),
  );
}
```

```ts
// ❌ Forbidden — nothing states what these three effects are for.
constructor() {
  effect(() => { /* … */ });
  effect(() => { /* … */ });
  effect(() => { /* … */ });
}
```

Effects execute asynchronously during the change detection process. They always run at least once.

## DOM Manipulation with `afterRenderEffect`

Standard `effect` runs _before_ Angular updates the DOM. If you need to manually inspect or modify the DOM based on a signal change (e.g., integrating a 3rd party UI library), use `afterRenderEffect`.

`afterRenderEffect` runs after Angular has finished rendering the DOM.

### Render Phases

To prevent reflows (forced layout thrashing), `afterRenderEffect` forces you to divide your DOM reads and writes into specific phases.

```ts
import { Component, afterRenderEffect, viewChild, ElementRef } from '@angular/core';

@Component({...})
export class Chart {
  protected readonly canvas = viewChild.required<ElementRef>('canvas');

  constructor() {
    this.#resizeChartToCanvas();
  }

  #resizeChartToCanvas(): void {
    afterRenderEffect({
      // 1. Read from the DOM
      earlyRead: () => {
        return this.canvas().nativeElement.getBoundingClientRect().width;
      },
      // 2. Write to the DOM (receives the result of the previous phase)
      write: (width) => {
        // NEVER read from the DOM in the write phase.
        setupChart(this.canvas().nativeElement, width);
      }
    });
  }
}
```

**Available Phases (executed in this order):**

1. `earlyRead`
2. `write` (Never read here)
3. `mixedReadWrite` (Avoid if possible)
4. `read` (Never write here)

_Note: `afterRenderEffect` only runs on the client, never during Server-Side Rendering (SSR)._

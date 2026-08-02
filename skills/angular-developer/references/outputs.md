# Outputs (Custom Events)

Outputs allow a child component to emit custom events that a parent component can listen to. Angular recommends using the new `output()` function for modern applications.

## Function-based outputs

Declare outputs using the `output()` function. This returns an `OutputEmitterRef`.

```ts
import {Component, output} from '@angular/core';

@Component({
  selector: 'custom-slider',
  template: `<button (click)="changeValue(50)">Set to 50</button>`,
})
export class CustomSlider {
  // Output without event data
  readonly panelClosed = output<void>();

  // Output with event data (number)
  readonly valueChanged = output<number>();

  changeValue(newValue: number) {
    this.valueChanged.emit(newValue);
  }
}
```

### Usage in Template

Bind to the output event using parentheses `()`. If the event emits data, access it using the special `$event` variable.

```html
<custom-slider (panelClosed)="savePanelState()" (valueChanged)="logValue($event)" />
```

## Configuration Options

The `output` function accepts a config object to specify an alias.

```ts
@Component({...})
export class CustomSlider {
  // The event is named 'valueChanged' in the template,
  // but accessed as 'changed' in the component class.
  readonly changed = output<number>({ alias: 'valueChanged' });
}
```

## Programmatic Subscription

Dynamically created components are the one place a subscription to an output is
correct — a template binding is not available. Angular tears the subscription
down when the created component is destroyed, so do not track and unsubscribe by
hand.

```ts
readonly #viewContainer = inject(ViewContainerRef);
readonly #destroyRef = inject(DestroyRef);

#addSlider(): void {
  const componentRef = this.#viewContainer.createComponent(CustomSlider);

  componentRef.instance.valueChanged
    // A method body is not an injection context, so pass the DestroyRef.
    .pipe(takeUntilDestroyed(this.#destroyRef))
    .subscribe((value) => this.#onSliderChanged(value));
}
```

In a static template, bind the output instead — `(valueChanged)="…"`. See
[async-and-observables.md](async-and-observables.md) for when a subscription is
justified at all.

## Decorator-based Outputs (@Output) — legacy

`@Output()` with an `EventEmitter` is never written in new code. This section is
for reading and migrating outputs that already exist; `ng generate @angular/core:output-migration`
converts them.

```ts
import { Component, Output, EventEmitter } from '@angular/core';

@Component({...})
export class LegacyExample {
  @Output() readonly valueChanged = new EventEmitter<number>();

  // With alias
  @Output('customEventName') readonly changed = new EventEmitter<void>();
}
```

## Best Practices

- **Prefer `output()`**: Use the function-based `output()` instead of `@Output()` and `EventEmitter`.
- **Naming**: Use `camelCase` for output names. Avoid prefixing with `on` (e.g., use `valueChanged` instead of `onValueChanged`).
- **No DOM Bubbling**: Angular custom events do not bubble up the DOM tree like native events.
- **Avoid Collisions**: Do not choose names that collide with native DOM events (like `click` or `submit`).

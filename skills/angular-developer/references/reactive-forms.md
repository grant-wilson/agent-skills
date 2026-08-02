# Reactive Forms (legacy)

> **Legacy reference.** Signal forms are the forms API for all new work — see
> [signal-forms.md](signal-forms.md). Read this file to understand and maintain
> reactive forms that already exist; do not write new ones.

Reactive forms provide a model-driven approach to handling form inputs. They are built around observable streams and provide synchronous access to the data model, making them more scalable and testable than template-driven forms.

## Core Classes

Reactive forms are built using these fundamental classes from `@angular/forms`:

- `FormControl`: Manages the value and validity of an individual input.
- `FormGroup`: Manages a group of controls (an object-like structure).
- `FormArray`: Manages a numerically indexed array of controls.
- `FormBuilder`/`NonNullableFormBuilder`: A service that provides factory methods for creating control instances.

## Setup

Import `ReactiveFormsModule` into your component.

```ts
import {Component, inject} from '@angular/core';
import {ReactiveFormsModule, NonNullableFormBuilder, Validators} from '@angular/forms';

@Component({
  selector: 'app-profile-editor',
  imports: [ReactiveFormsModule],
  templateUrl: './profile-editor.component.html',
})
export class ProfileEditor {
  readonly #fb = inject(NonNullableFormBuilder);

  // Using FormBuilder for concise definition
  protected readonly profileForm = this.#fb.group({
    firstName: ['', Validators.required],
    lastName: '',
    address: this.#fb.group({
      street: '',
      city: '',
    }),
    aliases: this.#fb.array([this.#fb.control('')]),
  });

  protected onSubmit() {
    console.warn(this.profileForm.value);
  }
}
```

## Template Binding

Use directives to bind the model to the view:

- `[formGroup]`: Binds a `FormGroup` to a `<form>` or `<div>`.
- `formControlName`: Binds a named control within a group to an input.
- `formGroupName`: Binds a nested `FormGroup`.
- `formArrayName`: Binds a nested `FormArray`.
- `[formControl]`: Binds a standalone `FormControl`.

```html
<form [formGroup]="profileForm" (ngSubmit)="onSubmit()">
  <input type="text" formControlName="firstName" />

  <div formGroupName="address">
    <input type="text" formControlName="street" />
  </div>

  <div formArrayName="aliases">
    @for (alias of profileForm.controls.aliases.controls; track alias) {
    <input type="text" [formControlName]="$index" />
    }
  </div>

  <button type="submit" [disabled]="!profileForm.valid">Submit</button>
</form>
```

## Accessing Controls

Use `.controls` for easy access to controls.

```ts
addAlias() {
  this.profileForm.controls.aliases.push(this.fb.control(''));
}
```

## Updating Values

- `patchValue()`: Updates only the specified properties. Fails silently on structural mismatches.
- `setValue()`: Replaces the entire model. Strictly enforces the form structure.

```ts
updateProfile() {
  this.profileForm.patchValue({
    firstName: 'Nancy',
    address: { street: '123 Drew Street' }
  });
}
```

## Unified Change Events

Modern Angular (v18+) provides a single `events` observable on all controls to track value, status, pristine, touched, reset, and submit events.

Form `events` is a genuine stream, so an Observable is the right shape here — but
it is still consumed through `toSignal` or a `takeUntilDestroyed()` subscription
created in a named method, never a bare `.subscribe()` in a constructor.

```ts
import {ValueChangeEvent} from '@angular/forms';

protected readonly latestValue = toSignal(
  this.profileForm.events.pipe(
    filter((event) => event instanceof ValueChangeEvent),
    map((event) => event.value),
  ),
);
```

## Manual State Management

- `markAsTouched()` / `markAllAsTouched()`: Useful for showing validation errors on submit.
- `markAsDirty()` / `markAsPristine()`: Tracks if the value has been modified.
- `updateValueAndValidity()`: Manually triggers recalculation of value and status.
- Options `{ emitEvent: false }` or `{ onlySelf: true }` can be passed to most methods to control propagation.

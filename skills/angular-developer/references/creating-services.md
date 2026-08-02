# Creating and Using Services

Services in Angular are reusable pieces of code that handle data fetching, business logic, or state management that multiple components or other services need to access.

## Creating a Service

You can generate a service using the Angular CLI:

```bash
ng generate service my-data
```

Or you can manually create a TypeScript class and decorate it with `@Service()`.

```ts
import {Service, signal} from '@angular/core';

@Service()
export class BasicDataStore {
  readonly #data = signal<readonly string[]>([]);

  addData(item: string): void {
    this.#data.update((items) => [...items, item]);
  }

  getData(): readonly string[] {
    return this.#data();
  }
}
```

### The `@Service` decorator

Using `@Service` is the recommended approach for most services. It tells Angular to:

- **Create a single instance (singleton)** for the entire application.
- **Make it available everywhere** automatically, without needing to list it in any `providers` array.
- **Enable tree-shaking**, meaning the service is only included in the final JavaScript bundle if it is actually injected somewhere.

#### The `autoProvided` option

If you don't want to create a singleton of your service, you can set `@Service({autoProvided: false})` and declare the service a `providers` array.

## Injecting a Service

Once a service is created, you can inject it into components, directives, or other services using the `inject()` function.

### Injecting into a Component

```ts
import {Component, computed, inject} from '@angular/core';
import {BasicDataStore} from './basic-data-store.service';

@Component({
  selector: 'app-example',
  template: `
    <div>
      <p>Data items: {{ itemCount() }}</p>
      <button (click)="addItem()">Add Item</button>
    </div>
  `,
})
export class Example {
  // Inject the service as a class field; templates cannot see `#` members,
  // so expose only what the template needs as `protected`.
  readonly #dataStore = inject(BasicDataStore);

  protected readonly itemCount = computed(() => this.#dataStore.getData().length);

  protected addItem(): void {
    this.#dataStore.addData('New Item');
  }
}
```

### Injecting into Another Service

Services can inject other services in the exact same way.

```ts
import {Service, inject, signal} from '@angular/core';
import {AdvancedDataStore} from './advanced-data-store.service';

@Service()
export class BasicDataStore {
  // Injecting another service
  readonly #advancedDataStore = inject(AdvancedDataStore);

  readonly #data = signal<readonly string[]>([]);

  getData(): readonly string[] {
    // Combine data from this service and the injected service
    return [...this.#data(), ...this.#advancedDataStore.getData()];
  }
}
```

## Advanced Service Patterns

While `@Service` covers most scenarios, you may sometimes need:

- **Component-specific instances**: If a component needs its own isolated instance of a service, provide it directly in the component's `@Component({ providers: [MyService] })` array and set the `autoProvided: false` option: `@Service({autoProvided: false})`
- **Factory providers**: For dynamic creation.
- **Value providers**: For injecting configuration objects.

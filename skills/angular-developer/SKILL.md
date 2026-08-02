---
name: angular-developer
description: Angular v22 standards and code generation — CLI-driven scaffolding, standalone components, signal state and the function-based component API, resource APIs for async data, zoneless change detection, built-in template control flow, `inject()`, signal forms, layered plain-CSS component styles, lazy routes, and Vitest + TestBed testing with CDK harnesses, plus reference guides for components, reactivity, HTTP, DI, routing, accessibility, animations, and tooling. Use when writing, reviewing, or scaffolding Angular code, or when working in a project that contains angular.json.
---

# Angular Developer

This skill targets **Angular v22**, the current stable release. Do not branch
guidance by version or hedge across older majors. A project on an older major is
brought current with `ng update` before new features are added to it.

Reference guides under `references/` are derived from the Angular team's
`angular-developer` skill — see [NOTICE.md](NOTICE.md). The House Standards
below override anything in those references that disagrees with them.

## House Standards

These are not defaults to weigh against alternatives. They are the standard.

### Scaffolding and tooling

- All scaffolding and maintenance goes through the Angular CLI —
  `ng generate component|service|guard|…` for new artifacts, `ng update` for
  framework upgrades, `ng add` for integrating libraries. Never hand-create
  component files or hand-edit framework versions in `package.json`; the CLI
  applies schematics and migrations that manual edits miss.
- Plain CSS is the only style language. Scaffold with `--style=css` and keep the
  `schematics` default in `angular.json` set to `css`. No Sass/SCSS, Less, or any
  other preprocessor.
- Tailwind CSS is not used, and is never added to a project. Utility-class
  styling defeats the design-token and cascade-layer rules below.
- Set `"cli": { "analytics": false }` in `angular.json` so `ng` never prompts
  interactively in CI or local runs.

### Components and reactivity

- Every component, directive, and pipe is standalone. Never write
  `standalone: true` (it is the default) and never `standalone: false`. No new
  NgModules — compose via component `imports`, and provide app-wide services with
  `providedIn: "root"` or route-level `providers`.
- State is signal-based: `signal()` for writable state, `computed()` for
  derivation, `linkedSignal()` for resettable derived state, and `effect()` only
  for synchronizing with non-Angular code. Use the function-based component API —
  `input()`, `output()`, `model()`, `viewChild()`, `contentChild()` — never the
  decorator forms `@Input`/`@Output`/`@ViewChild`.
- Field visibility follows the TypeScript standard, adapted to what templates can
  see: `#`-private for everything the template does not touch (injected
  dependencies, internal state, helper methods), `protected readonly` only for
  members a template binds to, and public `readonly` for the component's API —
  `input()`, `output()`, `model()`. The TypeScript `private` keyword is forbidden;
  it is erased at runtime and buys nothing over `#`.

```typescript
@Component({/* … */})
export class CartItemComponent {
  readonly #pricing = inject(PricingService);

  readonly item = input.required<CartItem>();
  readonly quantity = model(1);
  readonly removed = output<string>();

  protected readonly lineTotal = computed(() =>
    this.#pricing.total(this.item(), this.quantity()),
  );
}
```

- An `effect()` is never left anonymous in a constructor. Name it for what it
  synchronizes — either a `#`-private method the constructor calls, or a field
  holding the `EffectRef` — whichever reads better in context. A constructor full
  of bare `effect(() => …)` calls hides how many side effects a component has and
  what each one is for.

```typescript
export class CartComponent {
  readonly #storage = inject(StorageService);
  protected readonly items = signal<readonly CartItem[]>([]);

  constructor() {
    this.#persistCartToStorage();
  }

  #persistCartToStorage(): void {
    effect(() => this.#storage.write("cart", this.items()));
  }
}
```

- Applications run zoneless: `provideZonelessChangeDetection()` in
  `app.config.ts`, no `zone.js` polyfill. `OnPush` is the framework default —
  never write `changeDetection: ChangeDetectionStrategy.OnPush` explicitly, since
  like `standalone: true` it is redundant. A component that genuinely needs eager
  checking opts out with `ChangeDetectionStrategy.Eager` (`Default` is deprecated
  and aliases `Eager`). State flows through signals so the framework knows what to
  update; never call `detectChanges()` manually.
- Templates use the built-in control flow — `@if`/`@else`, `@for` with a mandatory
  `track` expression, `@switch`, and `@defer` for below-the-fold or heavy
  components — never `*ngIf`/`*ngFor`/`*ngSwitch`.

```html
@for (order of orders(); track order.id) {
  <app-order-row [order]="order" (removed)="remove($event)" />
} @empty {
  <p>No orders yet.</p>
}
```

- Use `inject()` in field initializers instead of constructor parameter
  injection; it composes into reusable functions and keeps classes free of
  boilerplate constructors.
- Never reach for browser or web APIs directly. Use the Angular-idiomatic wrapper
  for each: the `DOCUMENT` token instead of the global `document`, `HttpClient`
  instead of `fetch`/`XMLHttpRequest`, `Router`/`Location` instead of
  `window.location` or the History API, `DomSanitizer` instead of raw DOM
  manipulation, and `PLATFORM_ID`/`isPlatformBrowser()` to guard platform-specific
  code. Injecting the wrappers keeps code testable and SSR-safe.

### Data, forms, and routing

- Load async data with the resource APIs — `httpResource` for HTTP reads,
  `resource()` for other async sources, `rxResource` when the source is an
  Observable — and render their `value`/`isLoading`/`error` signals. Do not
  hand-roll fetch-then-set-signal plumbing or manage subscriptions for data
  loading. Mutations (POST/PUT/DELETE) still go through `HttpClient` in a service.
- Async code uses `async`/`await`. Promise callback chains (`.then()`/`.catch()`)
  are forbidden, in application code and in tests alike.
- Signal forms are the only forms API for new work. Reactive and template-driven
  forms are legacy: read those references to understand existing code, never to
  write new code.
- Routes lazy-load (`loadComponent`/`loadChildren`) by default,
  guards/resolvers/interceptors are plain functions (`CanActivateFn`,
  `ResolveFn`, `HttpInterceptorFn`), and route params bind to component inputs via
  `withComponentInputBinding()`.

### Styling

- Component styles live in an explicit cascade layer so a component's rules can
  never accidentally outrank the design system. Declare the order once in the
  global stylesheet and wrap every component stylesheet's rules in a layer.

```css
/* styles.css — declares layer order once, before any component style loads */
@layer reset, base, components, utilities;
```

```css
/* cart-item.css */
@layer components {
  :host {
    display: block;
    padding: var(--space-2);
    background: var(--color-surface);
  }
}
```

- Values come from design tokens (`var(--…)`) defined as custom properties. Raw
  hex values and magic pixel numbers in component styles are a review failure.
- Keep view encapsulation at the default `Emulated`. `::ng-deep` is forbidden —
  expose a custom property for the child to consume instead.

### Testing

- Unit tests run on Vitest. Karma and Jasmine are fully removed: no
  `karma.conf.js`, no `karma`/`jasmine-core`/`@types/jasmine` in `package.json`,
  and no Jasmine globals (`jasmine.*`, `spyOn`) anywhere. Use Vitest's
  `describe`/`it`/`expect`/`vi`. Apply
  `ng generate @schematics/angular:refactor-jasmine-vitest` to migrate a project
  that still runs Karma.
- Every component and service ships with `TestBed` tests that exercise its public
  surface: set inputs via `fixture.componentRef.setInput()`, assert on rendered
  DOM and emitted outputs, stub HTTP with `provideHttpClientTesting()`. Zoneless
  tests await `fixture.whenStable()` rather than calling `fixture.detectChanges()`.
- Drive components through CDK test harnesses (`@angular/cdk/testing`), not raw
  DOM queries or low-level event dispatch. Obtain a `HarnessLoader` via
  `TestbedHarnessEnvironment.loader(fixture)`. Prefer existing Material/CDK
  harnesses, and author a `ComponentHarness` for each component you own, in a
  co-located `*.harness.ts`.
- Shared test code lives in a project-level `testing/` folder — TestBed setup
  builders, fakes for injected services, and data builders. Spec files contain
  Arrange–Act–Assert cases and nothing else; a helper defined at the top of a spec
  file belongs in `testing/` instead.

```
src/
  testing/
    render-component.ts        // TestBed setup builder
    fake-order-repository.ts   // fake for an injected port
    order.builder.ts           // domain data builder
  orders/
    order-list.ts
    order-list.harness.ts      // ComponentHarness, co-located
    order-list.spec.ts         // AAA cases only
```

### Verification loop

Follow red–green–refactor: write the failing test first, then the implementation.
Before reporting work as done, in this order:

1. `ng test` — passes, including the new test that failed before the change.
2. `ng build` — succeeds with zero warnings. Warnings are errors; fix them rather
   than filtering them out.

Do not skip either step, and do not report success without having run them.

## Creating new projects

For the full `ng new` flow, use the `angular-new-app` skill. The essentials:
`npx ng new <app-name> --style=css --interactive=false`, then set
`"cli": { "analytics": false }` in `angular.json`, then generate every subsequent
artifact with `ng generate`. Never pass `--skip-tests`.

## Components

- **Fundamentals**: Anatomy, metadata, core concepts, and template control flow (@if, @for, @switch). Read [components.md](references/components.md)
- **Inputs**: Signal-based inputs, transforms, and model inputs. Read [inputs.md](references/inputs.md)
- **Outputs**: Signal-based outputs and custom event best practices. Read [outputs.md](references/outputs.md)
- **Host Elements**: Host bindings and attribute injection. Read [host-elements.md](references/host-elements.md)

If you require deeper documentation not found in the references above, read the documentation at `https://angular.dev/guide/components`.

## Reactivity and Data Management

- **Signals Overview**: Core signal concepts (`signal`, `computed`), reactive contexts, and `untracked`. Read [signals-overview.md](references/signals-overview.md)
- **Dependent State (`linkedSignal`)**: Creating writable state linked to source signals. Read [linked-signal.md](references/linked-signal.md)
- **Async Reactivity (`resource`)**: Fetching asynchronous data directly into signal state. Read [resource.md](references/resource.md)
- **Side Effects (`effect`)**: Naming effects, third-party DOM manipulation (`afterRenderEffect`), and when NOT to use effects. Read [effects.md](references/effects.md)

## HTTP Communication

- **HTTP Client and Resources**: `provideHttpClient`, `HttpClient`, interceptors, and `httpResource`. Read [http-client.md](references/http-client.md)

## Forms

Signal forms for all new work; the other two references describe legacy code only.

- **Signal Forms**: Signals for form state management. Read [signal-forms.md](references/signal-forms.md)
- **Reactive forms** (legacy): For reading and maintaining existing reactive forms. Read [reactive-forms.md](references/reactive-forms.md)
- **Template-driven forms** (legacy): For reading and maintaining existing template-driven forms. Read [template-driven-forms.md](references/template-driven-forms.md)

## Dependency Injection

- **Fundamentals**: Overview of Dependency Injection, services, and the `inject()` function. Read [di-fundamentals.md](references/di-fundamentals.md)
- **Creating and Using Services**: Creating services, the `providedIn: 'root'` option, and injecting into components or other services. Read [creating-services.md](references/creating-services.md)
- **Defining Dependency Providers**: Automatic vs manual provision, `InjectionToken`, `useClass`, `useValue`, `useFactory`, and scopes. Read [defining-providers.md](references/defining-providers.md)
- **Injection Context**: Where `inject()` is allowed, `runInInjectionContext`, and `assertInInjectionContext`. Read [injection-context.md](references/injection-context.md)
- **Hierarchical Injectors**: The `EnvironmentInjector` vs `ElementInjector`, resolution rules, modifiers (`optional`, `skipSelf`), and `providers` vs `viewProviders`. Read [hierarchical-injectors.md](references/hierarchical-injectors.md)

## Pipes

Prefer pipes in templates; outside templates, avoid injecting pipe classes just to call `transform()`.

- **Pipes**: Built-in pipe imports, custom pipe naming and implementation, pure vs impure pipes, and TypeScript reuse patterns using standalone formatting functions or extracted plain functions. Read [pipes.md](references/pipes.md)

## Angular Aria

When building accessible custom components for any of the following patterns: Accordion, Listbox, Combobox, Menu, Tabs, Toolbar, Tree, Grid, consult the following reference:

- **Angular Aria Components**: Building headless, accessible components and styling ARIA attributes. Read [angular-aria.md](references/angular-aria.md)

## Routing

- **Define Routes**: URL paths, static vs dynamic segments, wildcards, and redirects. Read [define-routes.md](references/define-routes.md)
- **Route Loading Strategies**: Eager vs lazy loading, and context-aware loading. Read [loading-strategies.md](references/loading-strategies.md)
- **Show Routes with Outlets**: Using `<router-outlet>`, nested outlets, and named outlets. Read [show-routes-with-outlets.md](references/show-routes-with-outlets.md)
- **Navigate to Routes**: Declarative navigation with `RouterLink` and programmatic navigation with `Router`. Read [navigate-to-routes.md](references/navigate-to-routes.md)
- **Control Route Access with Guards**: Implementing `CanActivate`, `CanMatch`, and other guards for security. Read [route-guards.md](references/route-guards.md)
- **Data Resolvers**: Pre-fetching data before route activation with `ResolveFn`. Read [data-resolvers.md](references/data-resolvers.md)
- **Router Lifecycle and Events**: Chronological order of navigation events and debugging. Read [router-lifecycle.md](references/router-lifecycle.md)
- **Rendering Strategies**: CSR, SSG (Prerendering), and SSR with hydration. Read [rendering-strategies.md](references/rendering-strategies.md)
- **Route Transition Animations**: Enabling and customizing the View Transitions API. Read [route-animations.md](references/route-animations.md)

If you require deeper documentation or more context, visit the [official Angular Routing guide](https://angular.dev/guide/routing).

## Styling and Animations

- **Styling components**: Cascade layers, encapsulation, and component style scoping. Read [component-styling.md](references/component-styling.md)
- **Angular Animations**: Using native CSS for dynamic effects. Read [angular-animations.md](references/angular-animations.md)

## Testing

- **Fundamentals**: TDD loop, Arrange–Act–Assert, shared `testing/` utilities, zoneless async patterns, and `TestBed`. Read [testing-fundamentals.md](references/testing-fundamentals.md)
- **Component Harnesses**: Authoring and using harnesses to drive components. Read [component-harnesses.md](references/component-harnesses.md)
- **Router Testing**: Using `RouterTestingHarness` for reliable navigation tests. Read [router-testing.md](references/router-testing.md)
- **End-to-End (E2E) Testing**: Setting up and running E2E tests. Read [e2e-testing.md](references/e2e-testing.md)

## Tooling

- **Angular CLI**: Creating applications, generating code (components, routes, services), serving, and building. Read [cli.md](references/cli.md)
- **Code Modernization**: Automatically refactoring to modern standards using migrations. Read [migrations.md](references/migrations.md)
- **Angular MCP Server**: Available tools, configuration, and experimental features. Read [mcp.md](references/mcp.md)
- **Environment Configuration**: Strategies for build-time and runtime configuration. Read [environment-configuration.md](references/environment-configuration.md)

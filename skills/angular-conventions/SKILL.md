---
name: angular-conventions
description: Angular conventions — CLI-driven scaffolding, standalone components, signal-based state and the function-based component API, resource APIs for async data, zoneless change detection, built-in template control flow, `inject()`, lazy routes, and Vitest + TestBed testing with CDK harnesses. Use when writing, reviewing, or scaffolding Angular code, or when working in a project that contains angular.json.
---

- All scaffolding and maintenance goes through the Angular CLI —
`ng generate component|service|guard|…` for new artifacts, `ng update` for
framework upgrades, `ng add` for integrating libraries. Never hand-create
component files or hand-edit framework versions in `package.json`; the CLI
applies schematics and migrations that manual edits miss.
- Use plain CSS as the component style language — scaffold with
`--style=css` and keep the `schematics` default in `angular.json` set to `css`.
No Sass/SCSS, Less, or other preprocessors.
- Configure the CLI to not share usage analytics so `ng` commands never prompt
interactively: set `"cli": { "analytics": false }` in `angular.json` (equivalent
to `ng analytics off`). This keeps scaffolding and build commands non-interactive
in CI and local runs.
- Every component, directive, and pipe is standalone (the default — do not write
`standalone: true` explicitly, and never `standalone: false`). No new NgModules;
compose via component `imports` and provide app-wide services with
`providedIn: "root"` or route-level `providers`.
- Component and service state is signal-based: `signal()` for writable state,
`computed()` for derivation, `linkedSignal()` for resettable derived state, and
`effect()` only for synchronizing with non-Angular code. Use the function-based
component API — `input()`, `output()`, `model()`, `viewChild()`,
`contentChild()` — never the decorator forms `@Input`/`@Output`/`@ViewChild`.

```typescript
@Component({/* … */})
export class CartItemComponent {
  readonly item = input.required<CartItem>();
  readonly quantity = model(1);
  readonly removed = output<string>();

  protected readonly lineTotal = computed(() =>
    this.item().price * this.quantity()
  );
}
```
- Load async data with the resource APIs — `httpResource` for HTTP reads,
`resource()` for other async sources, `rxResource` when the source is an
Observable — and render their `value`/`isLoading`/`error` signals. Do not
hand-roll fetch-then-set-signal plumbing or manage subscriptions for data
loading. Mutations (POST/PUT/DELETE) still go through `HttpClient` in a service.
- Applications run zoneless: `provideZonelessChangeDetection()` in
`app.config.ts`, no `zone.js` polyfill. OnPush is the framework default — never
write `changeDetection: ChangeDetectionStrategy.OnPush` explicitly (like
`standalone: true`, it's redundant); a component that genuinely needs eager
checking opts out with `ChangeDetectionStrategy.Eager`. State changes flow
through signals so the framework knows what to update; never call
`detectChanges()` manually.
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
- Use `inject()` in field initializers instead of constructor parameter injection;
it composes into reusable functions and keeps classes free of boilerplate
constructors.

```typescript
export class OrderService {
  readonly #http = inject(HttpClient);
  readonly #config = inject(APP_CONFIG);
}
```
- Never reach for browser/web APIs directly. Instead use the Angular-idiomatic
wrapper for each: `DOCUMENT` (injection token) instead of the global `document`,
`HttpClient` instead of `fetch`/`XMLHttpRequest`, the `Router`/`Location` instead
of `window.location` or the History API, `DomSanitizer` instead of raw DOM
manipulation, and `PLATFORM_ID`/`isPlatformBrowser()` to guard platform-specific
code. Injecting the wrappers keeps code testable and SSR-safe.
- Routes lazy-load (`loadComponent`/`loadChildren`) by default,
guards/resolvers/interceptors are plain functions (`CanActivateFn`,
`HttpInterceptorFn`), and route params bind to component inputs via
`withComponentInputBinding()`.
- Unit tests run on Vitest — apply the Angular Vitest migration
(`ng generate @schematics/angular:refactor-jasmine-vitest` / `ng add`
per the current CLI) and fully remove Karma and Jasmine: delete
`karma.conf.js`, drop `karma`, `jasmine-core`, and `@types/jasmine` from
`package.json`, and switch the `test` builder to the Vitest runner. Use
Vitest's `describe`/`it`/`expect`/`vi` — no Jasmine globals (`jasmine.*`,
`spyOn`) remain.
- Every component/service ships with `TestBed`-based unit tests that exercise the
public surface: set inputs via `fixture.componentRef.setInput()`, assert on
rendered DOM and emitted outputs, stub HTTP with `provideHttpClientTesting()`.
Zoneless tests await `fixture.whenStable()` rather than calling
`fixture.detectChanges()` after every change.

Interact with components through Angular CDK test harnesses
(`@angular/cdk/testing`) — obtain a `HarnessLoader` via
`TestbedHarnessEnvironment.loader(fixture)` and drive components with their
`ComponentHarness` rather than querying raw DOM nodes or dispatching low-level
events. Prefer existing Material/CDK harnesses and author a `ComponentHarness`
for your own components.

# Testing Fundamentals

Unit tests run on **Vitest**. Karma and Jasmine are fully removed from the
workspace: no `karma.conf.js`, no `karma`/`jasmine-core`/`@types/jasmine` in
`package.json`, and no Jasmine globals (`jasmine.*`, `spyOn`) anywhere. Use
Vitest's `describe`/`it`/`expect`/`vi`. Migrate a workspace still on Karma with
`ng generate @schematics/angular:refactor-jasmine-vitest` before writing new
tests.

## The loop

Red–green–refactor. Write the test that fails for the behavior you are about to
add, make it pass, then refactor with the test as the safety net. A bug fix
starts with a test that reproduces the bug. Before reporting work as done,
`ng test` passes and `ng build` is clean with zero warnings.

## Shared utilities, never ad-hoc helpers

A spec file contains Arrange–Act–Assert cases and nothing else. Setup functions,
fakes for injected services, and data builders live in a project-level `testing/`
folder that every spec imports. The moment a helper is worth writing, it is worth
putting where the next spec can find it — a setup block copy-pasted between two
spec files is the same review failure as duplicated production code.

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

```ts
// testing/render-component.ts
export async function renderComponent<T>(
  component: Type<T>,
  options: {inputs?: Partial<T>; providers?: Provider[]} = {},
): Promise<ComponentFixture<T>> {
  TestBed.configureTestingModule({providers: options.providers ?? []});

  const fixture = TestBed.createComponent(component);
  for (const [name, value] of Object.entries(options.inputs ?? {})) {
    fixture.componentRef.setInput(name, value);
  }
  await fixture.whenStable();
  return fixture;
}
```

Prefer fakes over mocks: substitute an injected collaborator with a simple
in-memory fake from `testing/`, and reserve `vi.fn()` spies for verifying genuine
outgoing commands.

## Zoneless and async-first

State changes schedule updates asynchronously, and tests must account for this.

**Do NOT** use `fixture.detectChanges()` to manually trigger updates.
**ALWAYS** use Act, Wait, Assert:

1.  **Act:** Update state or perform an action (set an input, click a button).
2.  **Wait:** `await fixture.whenStable()` so the framework processes the
    scheduled update and renders it.
3.  **Assert:** Verify the outcome.

## Drive components through harnesses

Interact with a component through its CDK test harness
(`@angular/cdk/testing`), not raw DOM queries or low-level event dispatch — a
harness survives markup changes that `querySelector` does not. Obtain a
`HarnessLoader` with `TestbedHarnessEnvironment.loader(fixture)`. Use existing
Material/CDK harnesses, and author a `ComponentHarness` in a co-located
`*.harness.ts` for each component you own. See
[component-harnesses.md](component-harnesses.md).

```ts
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {renderComponent} from '../testing/render-component';
import {OrderListComponent} from './order-list';
import {OrderListHarness} from './order-list.harness';
import {anOrder} from '../testing/order.builder';

describe('OrderListComponent', () => {
  it('shows an empty message when there are no orders', async () => {
    // Arrange
    const fixture = await renderComponent(OrderListComponent, {inputs: {orders: []}});
    const harness = await TestbedHarnessEnvironment.loader(fixture).getHarness(
      OrderListHarness,
    );

    // Act — nothing to do; the empty state is the initial render.

    // Assert
    expect(await harness.getEmptyMessage()).toBe('No orders yet.');
  });

  it('emits the order id when a row is removed', async () => {
    // Arrange
    const fixture = await renderComponent(OrderListComponent, {
      inputs: {orders: [anOrder({id: 'order-1'})]},
    });
    const harness = await TestbedHarnessEnvironment.loader(fixture).getHarness(
      OrderListHarness,
    );
    const removed: string[] = [];
    fixture.componentInstance.removed.subscribe((id) => removed.push(id));

    // Act
    await harness.removeOrder('order-1');

    // Assert
    expect(removed).toEqual(['order-1']);
  });
});
```

## TestBed and ComponentFixture

- **`TestBed`**: Creates a test-specific Angular environment. Configure it inside
  a `testing/` setup builder rather than repeating
  `TestBed.configureTestingModule({...})` in every spec.
- **`ComponentFixture`**: A handle on the created component instance and its
  environment.
  - `fixture.componentRef.setInput(name, value)`: The only way to set a signal
    input. Never assign to the input field directly.
  - `fixture.componentInstance`: Access the component's class instance — use it
    for outputs, not for reaching into state a harness should observe.
  - `fixture.nativeElement`: The root DOM element. Reserve it for the inside of a
    harness; specs use the harness.
- Stub HTTP with `provideHttpClientTesting()` and assert on requests through
  `HttpTestingController`.

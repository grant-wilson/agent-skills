# Testing with Component Harnesses

Component harnesses are the standard, preferred way to interact with components in tests. They provide a robust, user-centric API that makes tests less brittle and easier to read by insulating them from changes to a component's internal DOM structure.

## Why Use Harnesses?

- **Robustness:** Tests don't break when you refactor a component's internal HTML or CSS classes.
- **Readability:** Tests describe interactions from a user's perspective (e.g., `button.click()`, `slider.getValue()`) instead of through DOM queries (`fixture.nativeElement.querySelector(...)`).
- **Reusability:** The same harness can be used in both unit tests and E2E tests.

Angular Material provides a test harness for every component in its library.
**Every component you own also gets one**, in a co-located `*.harness.ts`. Specs
never query the DOM directly — that knowledge belongs in the harness, in one
place, so a markup change breaks one file instead of every spec.

## Authoring a harness

Extend `ComponentHarness`, give it a `hostSelector`, and expose methods named for
what a user does — not for the DOM underneath.

```ts
// order-list.harness.ts
import {ComponentHarness} from '@angular/cdk/testing';

export class OrderListHarness extends ComponentHarness {
  static readonly hostSelector = 'app-order-list';

  readonly #rows = this.locatorForAll('[data-testid="order-row"]');
  readonly #emptyMessage = this.locatorForOptional('[data-testid="empty"]');

  async getOrderIds(): Promise<string[]> {
    const rows = await this.#rows();
    return Promise.all(rows.map((row) => row.getAttribute('data-order-id'))) as Promise<
      string[]
    >;
  }

  async getEmptyMessage(): Promise<string | null> {
    return (await this.#emptyMessage())?.text() ?? null;
  }

  async removeOrder(id: string): Promise<void> {
    const button = await this.locatorFor(
      `[data-order-id="${id}"] [data-testid="remove"]`,
    )();
    await button.click();
  }
}
```

- Target stable `data-testid` attributes rather than CSS classes, which are
  styling concerns and change freely.
- Return domain values (`string[]` of ids), not `TestElement`s — a spec that
  handles `TestElement`s is doing DOM work the harness should have absorbed.
- Add a static `with(options)` returning a `HarnessPredicate` when a page can
  contain more than one instance.

## Using a Harness in a Unit Test

The `TestbedHarnessEnvironment` is the entry point for using harnesses in unit tests.

### Example: Testing with a `MatButtonHarness`

```ts
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {MatButtonHarness} from '@angular/material/button/testing';
import {MyButtonContainerComponent} from './my-button-container.component';

describe('MyButtonContainerComponent', () => {
  let fixture: ComponentFixture<MyButtonContainerComponent>;
  let loader: HarnessLoader;

  beforeEach(() => {
    TestBed.configureTestingModule({});

    fixture = TestBed.createComponent(MyButtonContainerComponent);
    // Create a harness loader for the component's fixture
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should find a button with specific text', async () => {
    // Load the harness for a MatButton with the text "Submit"
    const submitButton = await loader.getHarness(MatButtonHarness.with({text: 'Submit'}));

    // Use the harness API to interact with the component
    expect(await submitButton.isDisabled()).toBe(false);
    await submitButton.click();

    // ... assertions
  });
});
```

### Key Concepts

1.  **`HarnessLoader`**: An object used to find and create harness instances. Get a loader for your component's fixture using `TestbedHarnessEnvironment.loader(fixture)`.

2.  **`loader.getHarness(HarnessClass)`**: Asynchronously finds and returns a harness instance for the first matching component.

3.  **`HarnessClass.with({ ... })`**: Many harnesses provide a static `with` method that returns a `HarnessPredicate`. This allows you to filter and find components based on their properties, like text, selector, or disabled state. Always use this to precisely target the component you want to test.

4.  **Harness API:** Once you have a harness instance, use its methods (e.g., `.click()`, `.getText()`, `.getValue()`) to interact with the component. These methods automatically handle waiting for async operations and change detection.

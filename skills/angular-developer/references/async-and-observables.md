# Async Shapes: Signals, Promises, and Observables

Angular offers three ways to model asynchrony. Picking the wrong one is the most
common structural mistake in an Angular codebase — usually RxJS used to model a
single value.

**Choose by how many values the source produces over time.**

| Source produces                        | Use                                       |
| :------------------------------------- | :---------------------------------------- |
| A current value, held and read          | `signal()` / `computed()` / `linkedSignal()` |
| One async result, loaded into state     | `httpResource` / `resource()` / `rxResource` |
| One async result, acted on imperatively | `async`/`await` + `firstValueFrom()`      |
| Many values over time                   | `Observable`                              |

## One-shot work: `async`/`await`

A mutation, a dynamic import, a file read, a single backend call whose result you
act on rather than render — all produce exactly one result. Model them as
promises.

`HttpClient` returns cold Observables even for one-shot calls, so unwrap at the
service boundary with `firstValueFrom()`. Do not `.subscribe()`: a bare
`.subscribe()` discards the error path, and an error callback bolted onto it
re-creates the callback style the standards forbid.

```ts
@Service()
export class OrderService {
  readonly #http = inject(HttpClient);

  async placeOrder(order: NewOrder): Promise<Order> {
    return firstValueFrom(this.#http.post<Order>('/api/orders', order));
  }

  async cancelOrder(id: string): Promise<void> {
    await firstValueFrom(this.#http.delete<void>(`/api/orders/${id}`));
  }
}
```

```ts
// ❌ Forbidden — the failure path silently disappears.
placeOrder(order: NewOrder): void {
  this.#http.post<Order>('/api/orders', order).subscribe();
}
```

- `firstValueFrom` rejects with `EmptyError` if the Observable completes without
  emitting. That is the correct behaviour for a request that must return a value.
- Use `lastValueFrom` only when the Observable emits several times and you want
  the final one — for example an upload configured with `reportProgress: true`.
- `toPromise()` is removed from RxJS. Never use it.

## Reads rendered as state: resource APIs

Data that a template displays is loaded with a resource, not with a subscription
or an awaited call assigned into a signal. The resource owns loading state, error
state, and cancellation of superseded requests.

```ts
export class UserProfile {
  readonly userId = input.required<string>();

  protected readonly user = httpResource<User>(() => `/api/users/${this.userId()}`);
}
```

Use `rxResource` when the source is already an Observable you do not control.

## Genuine streams: Observables

Reach for an Observable when the source emits repeatedly over time, or when the
logic needs operators. Legitimate cases:

- Router events (`Router.events`)
- Form control `events`
- WebSocket / Server-Sent Events messages
- DOM event streams that need `debounceTime`, `throttleTime`, or `distinctUntilChanged`
- Request flows needing `switchMap` (cancel the previous in-flight request),
  `retry` with backoff, or `combineLatest` across several sources

A typeahead is the canonical example — it is a stream (`switchMap` cancels
superseded lookups) whose result is consumed as state:

```ts
export class UserSearch {
  readonly #users = inject(UserService);

  protected readonly query = signal('');

  protected readonly results = toSignal(
    toObservable(this.query).pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap((term) => this.#users.search(term)),
    ),
    {initialValue: [] as readonly User[]},
  );
}
```

## Crossing between the worlds

Convert at the edges only — a codebase that converts back and forth mid-pipeline
has picked the wrong shape somewhere upstream.

| Direction              | Use                                     |
| :--------------------- | :-------------------------------------- |
| Observable → template  | `toSignal(source, {initialValue})`      |
| Signal → operator pipeline | `toObservable(signal)`              |
| Observable → `await`   | `firstValueFrom(source)`                |
| Observable → resource  | `rxResource({stream: …})`               |

Both `toSignal` and `toObservable` require an injection context. Call them in a
field initializer, or pass an explicit `injector`.

## Subscription hygiene

**Never call `.subscribe()` in a component.** Use `toSignal`, the `async` pipe, or
a resource. A component that subscribes is managing a lifecycle the framework
already manages.

Where a subscription is genuinely unavoidable — usually a service reacting to a
framework stream — create it in a named method rather than inline in the
constructor, the same rule that applies to effects, and pipe it through
`takeUntilDestroyed()`.

```ts
@Service()
export class NavigationTelemetry {
  readonly #router = inject(Router);
  readonly #telemetry = inject(TelemetryClient);

  constructor() {
    this.#reportCompletedNavigations();
  }

  #reportCompletedNavigations(): void {
    this.#router.events
      .pipe(
        filter((event) => event instanceof NavigationEnd),
        takeUntilDestroyed(),
      )
      .subscribe((event) => this.#telemetry.pageView(event.url));
  }
}
```

`takeUntilDestroyed()` with no argument must itself be called in an injection
context; outside one, pass a `DestroyRef` explicitly. Manual
`subscription.unsubscribe()` in `ngOnDestroy` is not used — it is the pattern
`takeUntilDestroyed` exists to replace.

## Migrations

We have created a one-to-one mapping for migrating from your exiting test modules to InSpatial Test. We created InSpatial Test API's to mirror the standards so migration is seamless.

#### A friendly migration-map from Vitest to InSpatial Test so you can switch without friction

### Vitest ↔ InSpatial Test Equivalents

| What you want to do        | Vitest                                 | InSpatial Test                                | Notes                                                             |
| -------------------------- | -------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------- |
| Define a suite             | `describe()`                           | `describe()`                                  | Same usage and semantics.                                         |
| Define a test              | `it()` / `test()`                      | `it()` / `test()`                             | Both are available.                                               |
| Basic assertions           | `expect(value)`                        | `expect(value)`                               | Common matchers supported.                                        |
| Create a mock function     | `vi.fn()`                              | `mockFn()`                                    | `mockFn` keeps call history; use `getMockCalls(mock)` to inspect. |
| Spy on a method            | `vi.spyOn(obj, 'm')`                   | `spy(obj, 'm')`                               | Non-invasive: records calls/returns/errors.                       |
| Replace (stub) a method    | n/a                                    | `stub(obj, 'm', impl)`                        | Fully replaces implementation and records calls.                  |
| Restore all mocks          | `vi.restoreAllMocks()`                 | `restoreTest()`                               | Or use `using` with `spy/stub` for auto-restore.                  |
| Start fake timers          | `vi.useFakeTimers()`                   | `new FakeTime()`                              | Controls Date, timeouts, intervals.                               |
| Advance timers             | `vi.advanceTimersByTime(ms)`           | `time.tick(ms)`                               | Also `time.tickAsync(ms)`.                                        |
| Run all timers             | `vi.runAllTimers()`                    | `time.runAll()` / `time.runAllAsync()`        | Flush all scheduled timers.                                       |
| Mock return once           | `vi.fn().mockReturnValueOnce(v)`       | `returnsNext([v, ...])`                       | Sequential sync returns.                                          |
| Mock resolve once          | `vi.fn().mockResolvedValueOnce(v)`     | `resolvesNext([v, ...])`                      | Sequential async resolves/rejections.                             |
| Assert calls (count)       | `expect(fn).toHaveBeenCalledTimes(n)`  | `assertSpyCalls(spy, n)`                      | Works with `spy`/`stub`.                                          |
| Assert calls (args/return) | `expect(fn).toHaveBeenCalledWith(...)` | `assertSpyCall(spy, idx, { args, returned })` | Check a specific call by index.                                   |
| Snapshots                  | `expect(v).toMatchSnapshot()`          | `assertSnapshot(t, v, opts?)`                 | Uses test context `t`. Also `createAssertSnapshot`.               |
| Wait for condition         | `vi.waitFor(fn)`                       | custom await/poll                             | Use your own loop or drive with `FakeTime` and re-assert.         |

> **Note:** For multi-step mocking, prefer `spy(obj, 'method')` if you just need to observe; use `stub(obj, 'method', impl)` when you need to replace behavior.

### Handy snippets

```ts
// Mock function (like vi.fn)
import { mockFn, getMockCalls } from "@inspatial/kit/test";
const fn = mockFn((a: number) => a + 1);
fn(1);
console.log(getMockCalls(fn).length); // 1
```

```ts
// Spy and assert
import { spy, assertSpyCalls, assertSpyCall } from "@inspatial/kit/test";
const math = { add: (a: number, b: number) => a + b };
using s = spy(math, "add");
math.add(2, 3);
assertSpyCalls(s, 1);
assertSpyCall(s, 0, { args: [2, 3], returned: 5 });
```

```ts
// Fake time
import { FakeTime } from "@inspatial/kit/test";
using t = new FakeTime();
let hits = 0;
setTimeout(() => hits++, 1000);
t.tick(1000);
// hits === 1
```

```ts
// Snapshots
import { assertSnapshot } from "@inspatial/kit/test";
// inside a test with context `t`
await assertSnapshot(t, { hello: "world" });
```

> **Note:** To discover `*.spec.*` files, you can use the built-in spec runner task in `@inspatial/kit/test` or include spec globs in your project's `deno.json` test task.

### Jest ↔ InSpatial Test Equivalents

| What you want to do       | Jest                                       | InSpatial Test                                            | Notes                                                             |
| ------------------------- | ------------------------------------------ | --------------------------------------------------------- | ----------------------------------------------------------------- |
| Define a test suite       | `describe()`                               | `describe()`                                              | Same usage and semantics.                                         |
| Define a test             | `it()` / `test()`                          | `it()` / `test()`                                         | Both are available.                                               |
| Basic assertions          | `expect(value)`                            | `expect(value)`                                           | Common matchers supported.                                        |
| Strict equality           | `expect(a).toBe(b)`                        | `expect(a).toBe(b)`                                       | Same API.                                                         |
| Deep equality             | `expect(a).toEqual(b)`                     | `expect(a).toEqual(b)`                                    | Same API.                                                         |
| Truthiness                | `expect(a).toBeTruthy()`                   | `expect(a).toBeTruthy()`                                  | Same API.                                                         |
| Falsiness                 | `expect(a).toBeFalsy()`                    | `expect(a).toBeFalsy()`                                   | Same API.                                                         |
| Null checks               | `expect(a).toBeNull()`                     | `expect(a).toBeNull()`                                    | Same API.                                                         |
| Undefined checks          | `expect(a).toBeUndefined()`                | `expect(a).toBeUndefined()`                               | Same API.                                                         |
| Defined checks            | `expect(a).toBeDefined()`                  | `expect(a).toBeDefined()`                                 | Same API.                                                         |
| Array/string contains     | `expect(a).toContain(b)`                   | `expect(a).toContain(b)`                                  | Same API.                                                         |
| Object properties         | `expect(obj).toHaveProperty('key')`        | `expect(obj).toHaveProperty('key')`                       | Same API.                                                         |
| String matching           | `expect(str).toMatch(/regex/)`             | `expect(str).toMatch(/regex/)`                            | Same API.                                                         |
| Number comparisons        | `expect(a).toBeGreaterThan(b)`             | `expect(a).toBeGreaterThan(b)`                            | Same API.                                                         |
| Instance checks           | `expect(obj).toBeInstanceOf(Class)`        | `expect(obj).toBeInstanceOf(Class)`                       | Same API.                                                         |
| Create mock function      | `jest.fn()`                                | `mockFn()`                                                | `mockFn` keeps call history; use `getMockCalls(mock)` to inspect. |
| Spy on method             | `jest.spyOn(obj, 'method')`                | `spy(obj, 'method')`                                      | Non-invasive: records calls/returns/errors.                       |
| Mock implementation       | `jest.fn().mockImplementation(fn)`         | `stub(obj, 'method', fn)`                                 | Use `stub` for replacing behavior.                                |
| Mock return value         | `jest.fn().mockReturnValue(val)`           | `returnsNext([val])`                                      | Sequential returns.                                               |
| Mock return once          | `jest.fn().mockReturnValueOnce(val)`       | `returnsNext([val])`                                      | Sequential sync returns.                                          |
| Mock resolved value       | `jest.fn().mockResolvedValue(val)`         | `resolvesNext([val])`                                     | Sequential async resolves.                                        |
| Mock rejected value       | `jest.fn().mockRejectedValue(err)`         | `resolvesNext([new Error(err)])`                          | Sequential async rejections.                                      |
| Assert call count         | `expect(fn).toHaveBeenCalledTimes(n)`      | `assertSpyCalls(spy, n)`                                  | Works with `spy`/`stub`.                                          |
| Assert called with        | `expect(fn).toHaveBeenCalledWith(...)`     | `assertSpyCall(spy, idx, { args: [...] })`                | Check specific call by index.                                     |
| Assert last called with   | `expect(fn).toHaveBeenLastCalledWith(...)` | `assertSpyCall(spy, spy.calls.length-1, { args: [...] })` | Check last call.                                                  |
| Assert not called         | `expect(fn).not.toHaveBeenCalled()`        | `assertSpyCalls(spy, 0)`                                  | Verify no calls made.                                             |
| Restore all mocks         | `jest.restoreAllMocks()`                   | `restoreTest()`                                           | Or use `using` with `spy/stub` for auto-restore.                  |
| Clear all mocks           | `jest.clearAllMocks()`                     | `restoreTest()`                                           | Clears call history and restores.                                 |
| Reset all mocks           | `jest.resetAllMocks()`                     | `restoreTest()`                                           | Resets to original state.                                         |
| Fake timers               | `jest.useFakeTimers()`                     | `new FakeTime()`                                          | Controls Date, timeouts, intervals.                               |
| Advance timers            | `jest.advanceTimersByTime(ms)`             | `time.tick(ms)`                                           | Also `time.tickAsync(ms)`.                                        |
| Run all timers            | `jest.runAllTimers()`                      | `time.runAll()` / `time.runAllAsync()`                    | Flush all scheduled timers.                                       |
| Run pending timers        | `jest.runOnlyPendingTimers()`              | `time.next()` / `time.nextAsync()`                        | Run next scheduled timer.                                         |
| Set system time           | `jest.setSystemTime(date)`                 | `time.now = date.getTime()`                               | Set current fake time.                                            |
| Restore timers            | `jest.useRealTimers()`                     | `time.restore()`                                          | Restore real timers.                                              |
| Snapshots                 | `expect(val).toMatchSnapshot()`            | `assertSnapshot(t, val, opts?)`                           | Uses test context `t`. Also `createAssertSnapshot`.               |
| Inline snapshots          | `expect(val).toMatchInlineSnapshot()`      | `assertSnapshot(t, val)`                                  | InSpatial uses external snapshot files.                           |
| Error throwing            | `expect(fn).toThrow()`                     | `expect(fn).toThrow()`                                    | Same API.                                                         |
| Error throwing (specific) | `expect(fn).toThrow('message')`            | `expect(fn).toThrow('message')`                           | Same API.                                                         |
| Async error throwing      | `expect(asyncFn).rejects.toThrow()`        | `expect(asyncFn).rejects.toThrow()`                       | Same API.                                                         |
| Async resolution          | `expect(promise).resolves.toBe(val)`       | `expect(promise).resolves.toBe(val)`                      | Same API.                                                         |
| Setup hooks               | `beforeAll()` / `beforeEach()`             | `beforeAll()` / `beforeEach()`                            | Same API.                                                         |
| Teardown hooks            | `afterAll()` / `afterEach()`               | `afterAll()` / `afterEach()`                              | Same API.                                                         |
| Skip tests                | `test.skip()` / `it.skip()`                | `test.skip()` / `it.skip()`                               | Same API.                                                         |
| Focus tests               | `test.only()` / `it.only()`                | `test.only()` / `it.only()`                               | Same API.                                                         |
| Todo tests                | `test.todo()`                              | `test.todo()`                                             | Same API.                                                         |

> **Note:** InSpatial Test provides the same `expect` API as Jest for most common use cases. The main differences are in mocking utilities where InSpatial uses `spy`/`stub` pattern and time manipulation uses the `FakeTime` class.

### Chai ↔ InSpatial Test Equivalents

| What you want to do   | Chai                                                      | InSpatial Test                                     | Notes                                             |
| --------------------- | --------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------- |
| Strict equality       | `expect(a).to.equal(b)`                                   | `expect(a).toBe(b)`                                | Use `toBe` for strict equality.                   |
| Deep equality         | `expect(a).to.deep.equal(b)`                              | `expect(a).toEqual(b)`                             | Use `toEqual` for deep equality.                  |
| Loose equality        | `expect(a).to.eql(b)`                                     | `expect(a).toEqual(b)`                             | `toEqual` handles deep comparison.                |
| Truthiness            | `expect(a).to.be.true`                                    | `expect(a).toBe(true)`                             | Explicit boolean check.                           |
| Falsiness             | `expect(a).to.be.false`                                   | `expect(a).toBe(false)`                            | Explicit boolean check.                           |
| Truthy values         | `expect(a).to.be.ok`                                      | `expect(a).toBeTruthy()`                           | Any truthy value.                                 |
| Falsy values          | `expect(a).to.not.be.ok`                                  | `expect(a).toBeFalsy()`                            | Any falsy value.                                  |
| Null checks           | `expect(a).to.be.null`                                    | `expect(a).toBeNull()`                             | Same concept, different syntax.                   |
| Undefined checks      | `expect(a).to.be.undefined`                               | `expect(a).toBeUndefined()`                        | Same concept, different syntax.                   |
| Defined checks        | `expect(a).to.not.be.undefined`                           | `expect(a).toBeDefined()`                          | Check for defined values.                         |
| Existence checks      | `expect(a).to.exist`                                      | `expect(a).toBeDefined()`                          | Check value exists (not null/undefined).          |
| Type checking         | `expect(a).to.be.a('string')`                             | `expect(a).toBeType('string')`                     | InSpatial has `toBeType` matcher.                 |
| Instance checking     | `expect(obj).to.be.an.instanceof(Class)`                  | `expect(obj).toBeInstanceOf(Class)`                | Same concept, different syntax.                   |
| Array inclusion       | `expect(arr).to.include(item)`                            | `expect(arr).toContain(item)`                      | Check array contains item.                        |
| String inclusion      | `expect(str).to.include(substr)`                          | `expect(str).toContain(substr)`                    | Check string contains substring.                  |
| Object properties     | `expect(obj).to.have.property('key')`                     | `expect(obj).toHaveProperty('key')`                | Same concept, different syntax.                   |
| Property values       | `expect(obj).to.have.property('key', val)`                | `expect(obj).toHaveProperty('key', val)`           | Check property and value.                         |
| Nested properties     | `expect(obj).to.have.nested.property('a.b')`              | `expect(obj).toHaveProperty('a.b')`                | InSpatial supports nested paths.                  |
| Own properties        | `expect(obj).to.have.ownProperty('key')`                  | `expect(obj).toHaveProperty('key')`                | `toHaveProperty` checks own properties.           |
| Property descriptor   | `expect(obj).to.have.ownPropertyDescriptor('key', desc)`  | `expect(obj).toHaveDescribedProperty('key', desc)` | InSpatial has specific matcher.                   |
| Array/object length   | `expect(arr).to.have.length(n)`                           | `expect(arr).toHaveLength(n)`                      | Same concept, different syntax.                   |
| String matching       | `expect(str).to.match(/regex/)`                           | `expect(str).toMatch(/regex/)`                     | Same API.                                         |
| Number ranges         | `expect(n).to.be.within(1, 10)`                           | `expect(n).toBeWithin([1, 10])`                    | InSpatial uses array for range.                   |
| Greater than          | `expect(a).to.be.greaterThan(b)`                          | `expect(a).toBeGreaterThan(b)`                     | Same concept, different syntax.                   |
| Less than             | `expect(a).to.be.lessThan(b)`                             | `expect(a).toBeLessThan(b)`                        | Same concept, different syntax.                   |
| At least              | `expect(a).to.be.at.least(b)`                             | `expect(a).toBeGreaterThanOrEqual(b)`              | Greater than or equal.                            |
| At most               | `expect(a).to.be.at.most(b)`                              | `expect(a).toBeLessThanOrEqual(b)`                 | Less than or equal.                               |
| Close to (numbers)    | `expect(a).to.be.closeTo(b, delta)`                       | `assertAlmostEquals(a, b, delta)`                  | Use assert style for close numbers.               |
| Empty arrays/objects  | `expect(arr).to.be.empty`                                 | `expect(arr).toBeEmpty()`                          | InSpatial has `toBeEmpty` matcher.                |
| Function throwing     | `expect(fn).to.throw()`                                   | `expect(fn).toThrow()`                             | Same concept, different syntax.                   |
| Throw specific error  | `expect(fn).to.throw(Error)`                              | `expect(fn).toThrow(Error)`                        | Same concept, different syntax.                   |
| Throw with message    | `expect(fn).to.throw('message')`                          | `expect(fn).toThrow('message')`                    | Same concept, different syntax.                   |
| Negation              | `expect(a).to.not.equal(b)`                               | `expect(a).not.toBe(b)`                            | Both support negation.                            |
| Chain assertions      | `expect(obj).to.be.an('object').and.have.property('key')` | Split into multiple `expect()` calls               | InSpatial doesn't chain, use separate assertions. |
| Eventually (promises) | `expect(promise).to.eventually.equal(val)`                | `expect(promise).resolves.toBe(val)`               | Promise resolution testing.                       |
| Rejected promises     | `expect(promise).to.be.rejected`                          | `expect(promise).rejects.toThrow()`                | Promise rejection testing.                        |

> **Note:** Chai's fluent/chainable API differs from InSpatial Test's Jest-style API. While Chai allows chaining like `expect(x).to.be.a('string').and.have.length(5)`, InSpatial Test uses separate assertions: `expect(x).toBeType('string'); expect(x).toHaveLength(5);`

### Handy snippets

```ts
// Jest-style mock function
import { mockFn, getMockCalls } from "@inspatial/kit/test";
const fn = mockFn((a: number) => a + 1);
fn(1);
console.log(getMockCalls(fn).length); // 1
```

```ts
// Spy and assert (Jest equivalent)
import { spy, assertSpyCalls, assertSpyCall } from "@inspatial/kit/test";
const math = { add: (a: number, b: number) => a + b };
using s = spy(math, "add");
math.add(2, 3);
assertSpyCalls(s, 1);
assertSpyCall(s, 0, { args: [2, 3], returned: 5 });
```

```ts
// Fake time (Jest equivalent)
import { FakeTime } from "@inspatial/kit/test";
using t = new FakeTime();
let hits = 0;
setTimeout(() => hits++, 1000);
t.tick(1000);
// hits === 1
```

```ts
// Snapshots
import { assertSnapshot } from "@inspatial/kit/test";
// inside a test with context `t`
await assertSnapshot(t, { hello: "world" });
```

```ts
// Chai-style to InSpatial Test conversion
import { expect, assertAlmostEquals } from "@inspatial/kit/test";

// Chai: expect(obj).to.be.an('object').and.have.property('name', 'John');
// InSpatial Test:
expect(obj).toBeType("object");
expect(obj).toHaveProperty("name", "John");

// Chai: expect(num).to.be.closeTo(3.14, 0.01);
// InSpatial Test:
assertAlmostEquals(num, 3.14, 0.01);
```

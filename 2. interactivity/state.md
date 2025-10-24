# State

State is InSpatial Kit's structured state management system built on top of signals. While signals give you reactive primitives, state gives you organized, scalable patterns for managing application data from simple to complex enterprise systems.

If you're coming from [signals](./signal.md), you've been working with the **Scalar pattern** (single values). State introduces two powerful patterns: **Explicit** (everything in one place) and **Separation** (compose as you go). Both unlock features like batching, persistence, snapshots, and a unified action system that scales from prototypes to production.

**Think of it this way:** Signals are the atoms. State is the molecule structured, composable, and ready for real-world complexity.

---

### 🎯 Prerequisites

Before diving in, you should have:

- ✅ Basic understanding of [JavaScript](https://web.dev/learn/javascript)
- ✅ Working knowledge of [TypeScript](https://www.typescriptlang.org/docs/handbook/intro.html)
- ✅ Solid grasp of [Signals](./signal.md) (especially `.get()`, `.set()`, `$()`, and reactivity)

---

## 🧭 First Principles: Keep It Simple, Make It Reactive

### Should I worry about performance if everything is reactive?

**Short answer:** No.

**Why:** InSpatial Kit's reactivity is built for real-world apps. Reactive patterns are safe by default and won't cause noticeable slowdowns. Work scales with what _actually changes_, not with app size.

---

### Core Concepts

#### 🔹 Reactivity = Smart Tracking

- **Values remember their dependents.** When a value changes, only things that depend on it update.
- **Nothing changes? Nothing runs.** Idle cost is essentially zero.
- **Fine-grained updates.** Each state field is independent. Update one without waking the whole app.

#### 🔹 State = Structured Signals

- Each property in `createState({ ... })` is its own signal.
- Update one field: `state.count.set(5)` → only `count` dependents react.
- Batch multiple updates: `state.batch(() => { ... })` → all updates flush together.

#### 🔹 Derivation with `$()`

- `$(() => ...)` creates computed values that auto-update when dependencies change.
- Don't wrap constants: `const x = $(() => 5)` ❌ | Use for derived values: `const double = $(() => count.get() * 2)` ✅
- Inline or hoist: both work, but hoist reused computeds to avoid recreating them.

#### 🔹 Control Flow = Reactive Branching

- **`<Show>`**: Binary conditions (this OR that). With a signal, it reacts to changes.
- **`<Choose>`**: Multi-branch logic (this, that, OR another). Checks cases in order, reacts to condition changes.
- **`<List>`**: Render arrays. Use `track="id"` for large lists or frequent reordering.

---

### ✅ Good Defaults

1. **Make things reactive by default.** Don't optimize prematurely.
2. **Hoist reused computeds.** If you use `$(() => x > 10)` in 5 places, define it once.
3. **Batch related updates.** Group changes with `state.batch(() => { ... })`.
4. **Use keys for dynamic lists.** `<List track="id">` keeps Renderers stable during reorders.

---

### ⚠️ When to Think Twice

| Scenario                                 | Solution                                                 |
| ---------------------------------------- | -------------------------------------------------------- |
| **Huge lists that constantly reorder**   | Use `track` keys, paginate, or virtualize                |
| **Thousands of new computeds per frame** | Reuse computeds; avoid creating new `$()` in tight loops |
| **Unintended tracking in loops**         | Use `.peek()` for non-reactive reads inside loops        |

---

### 🎯 Bottom Line

**Reactive-by-default is safe.** InSpatial Kit's scheduler ensures efficient batching and async updates. Build first, optimize only when you measure a real bottleneck.

---

## State API Overview

**Import paths:**

- `@inspatial/kit/state` (app-facing)
- `@in/teract/state` (internal/module development)

InSpatial State is the built-in state management system for InSpatial Kit. It provides structured, scalable abstractions on top of signals, with features designed for real-world applications: batching, persistence, snapshots, subscriptions, and a unified action system.

### Two Patterns, Same Power

| Pattern        | Best For                               | Key Feature                                            |
| -------------- | -------------------------------------- | ------------------------------------------------------ |
| **Scalar**     | Single values, simple counters         | Direct signal wrapper (`createState(0)`)               |
| **Explicit**   | Apps, websites, cohesive modules       | Everything in one place (`createState.in({ ... })`)    |
| **Separation** | Games, XR, extensions, modular systems | Compose `createState`, `createAction`, `createStorage` |

**Choose based on your use case.** All patterns share the same capabilities batch, reset, snapshot, subscribe, and trigger system.

#### Scalar pattern

When you use `createState()` by passing a direct initial value, you're using what's called the **Scalar Pattern**. This pattern is essentially a convenient wrapper around the `createSignal()` API (available from `@inspatial/kit/signal`) and is fully interchangeable with it. This is the only scenario where you can swap `createState` and `createSignal` which is particularly useful for developers familiar with Solid's `createSignal`. However, in other patterns such as Separation (passing objects) and Explicit, `createState` and `createSignal` are **not** interchangeable.

```ts
const useCounter = createState(0);

// Read (creates dependency)
useCounter.get(); // or useCounter.value

// Write
useCounter.set(5); // or useCounter.value = 5

// Non-reactive read
useCounter.peek();
```

#### Explicit pattern

```ts
const useCounter = createState.in({
  initialState: { count: 0, name: "user" },
  action: {
    increment: { key: "count", fn: (c, n = 1) => c + n },
    setName: { key: "name", fn: (_, name: string) => name },
  },
  storage: { key: "useCounter", backend: "local" },
});

useCounter.action.increment();
useCounter.count.get();
```

#### Separation pattern (advanced)

```ts
const gameState = createState({ x: 0, y: 0, hp: 100, score: 0 });

// Direct Signal Action
const addScore = createAction(gameState.score, (s, n = 1) => s + n);

// Tuple Actions
const damage = createAction([gameState, "hp"], (hp, amt = 10) =>
  Math.max(0, hp - amt)
);

// Batch Actions
const move = createAction(gameState, {
  moveX: { key: "x", fn: (x, dx: number) => x + dx },
  moveY: { key: "y", fn: (y, dy: number) => y + dy },
});

// Optional persistence
createStorage(gameState, {
  key: "gameState",
  backend: "local",
  include: ["hp", "score"],
});
```

#### Feature snippets

- Signals on every key
  ```ts
  const s = createState({ a: 1, b: 2 });
  s.a.set(3); // Write
  s.b.get(); // Reactive read
  s.a.peek(); // Non-reactive read
  ```
- Batch/reset/snapshot/subscribe
  ```ts
  s.batch((v) => {
    v.a.set(10);
    v.b.set(20);
  });
  s.reset();
  s.snapshot();
  const off = s.subscribe((snap) => console.log(snap));
  off();
  ```
- Derived values
  ```ts
  const score2x = $(() => gameState.score.get() * 2);
  ```
- Action Trigger options
  ```ts
  const inc = createAction(s.a, (v) => v + 1, { throttle: 50, once: false });
  ```

### Type Inference and Typing Guide (Actions, Keys, Storage)

> Goal: get compile-time safety for action keys and storage keys, and great DX for triggers.

#### 1) State shape typing

- Prefer letting TypeScript infer from `initialState`:
  ```ts
  const s = createState.in({
    initialState: { count: 0, name: "ben" },
  });
  ```
- If you provide an explicit interface, make sure `initialState` strictly matches it (no extra fields). Avoid casting that hides mismatches.

#### 2) Strongly-typed action definitions (explicit pattern)

- Use `TriggerDefsFor<T>` to enforce that `key` is `keyof T` and `fn` is typed against the property type:

  ```ts
  import type { TriggerDefsFor } from "@in/teract/state/action.ts";

  interface Counter {
    count: number;
    name: string;
  }

  const state = createState.in({
    initialState: <Counter>{ count: 0, name: "" },
    action: <TriggerDefsFor<Counter>>{
      increment: { key: "count", fn: (c, n = 1) => c + n },
      setName: { key: "name", fn: (_c, v: string) => v },
      // key: "nope" // ❌ TS error: not in Counter
    },
  });
  ```

Result: `state.action.increment` and `state.action.setName` are callable functions with correct parameter types.

#### 3) Strongly-typed triggers (separation pattern)

- When using `createAction(state, defs)`, map defs to callable trigger signatures with `TriggerFunctionsFromDefs`:

  ```ts
  import { createState } from "@in/teract/state";
  import { createAction } from "@in/teract/state/action.ts";
  import type { TriggerFunctionsFromDefs, TriggerDefsFor } from "@in/teract/state/action.ts";

  const s = createState({ count: 0, name: "" });

  const defs: TriggerDefsFor<typeof s.snapshot()> = {
    inc:    { key: "count", fn: (c, n = 1) => c + n },
    setName:{ key: "name",  fn: (_c, v: string) => v },
  };

  const action = createAction(s, defs) as TriggerFunctionsFromDefs<typeof defs>;

  action.inc();      // ok
  action.setName("");
  ```

Notes:

- `typeof s.snapshot()` is a convenient way to get the state shape.
- You can also define a `type`/`interface` and reuse it for both state and defs.

#### 4) Enhanced action targets (no string keys)

For cross-state or nested targets, use enhanced targets instead of `key`:

```ts
const s = createState({ grid: "None" as "None" | "Line" | "Dot" });

// direct signal
const setGrid = createAction(s.grid, (_c, v: typeof s.grid.peek()) => v);

// state tuple [state, key]
const setGrid2 = createAction([s, "grid"], (_c, v) => v);

// batch enhanced defs
const triggers = createAction(s, {
  updateGrid: { target: [s, "grid"], fn: (_c, v: typeof s.grid.peek()) => v },
});
```

#### 5) Storage typing (`StoragePropsFor<T>`) – key-safe include/exclude

Use `StoragePropsFor<T>` so `include`/`exclude` are checked against your state keys:

```ts
import type { StoragePropsFor } from "@in/teract/state/storage.ts";

interface AppState {
  mode: string;
  window: { view: string };
  devMode: { value: boolean };
}

const app = createState.in({
  initialState: <AppState>{
    mode: "Spec",
    window: { view: "Hierarchy" },
    devMode: { value: false },
  },
  storage: <StoragePropsFor<AppState>>{
    key: "creator-portal",
    backend: "local",
    include: ["mode", "window", "devMode"], // ❌ typos are caught
  },
});
```

If you need custom serialization, provide `serialize`/`deserialize` while still enjoying typed `include`/`exclude`.

#### 6) Avoid common pitfalls

- Don’t cast `initialState` to an interface with a different shape (it disables key checking and leads to runtime errors).
- Don’t annotate actions as the callable signatures in explicit pattern; they are definitions. Use `TriggerDefsFor<T>` for defs; the runtime exposes callable functions under `state.action`.
- Prefer enhanced targets for non-local keys instead of string keys.

#### 7) Quick recipes

- Explicit pattern with fully typed keys:

  ```ts
  const ui = createState.in({
    initialState: { isOpen: false, count: 0 },
    action: {
      toggle: { key: "isOpen", fn: (v: boolean) => !v },
      inc: { key: "count", fn: (c: number, n = 1) => c + n },
    },
    storage: { key: "ui", backend: "local", include: ["isOpen", "count"] },
  });
  ```

- Separation pattern with cross-state trigger:

  ```ts
  const a = createState({ value: 0 });
  const b = createState({ other: 1 });

  // enhance target to update b.other based on a.value
  const sync = createAction([b, "other"], (_c) => a.value.peek());
  sync();
  ```

---

### Choosing the Right State Pattern

**All patterns have the same power. choose based on ergonomics and use case.**

#### Decision Tree

```
┌─ Is this a module or extension?
│  ├─ Yes → ✅ Separation Pattern (compose as you go)
│  └─ No  → Continue...
│
┌─ Is this an app or website?
│  ├─ Yes → ✅ Explicit Pattern (everything in one place)
│  └─ No  → Continue...
│
┌─ Is this a game or XR experience?
│  ├─ Yes → ✅ Separation Pattern (modular, performance-critical)
│  └─ No  → ✅ Explicit Pattern (default for most cases)
```

---

#### Pattern Comparison

| Pattern        | When to Use                     | Example Use Cases                      |
| -------------- | ------------------------------- | -------------------------------------- |
| **Scalar**     | Single reactive value           | Counters, toggles, form inputs         |
| **Explicit**   | Cohesive state in one place     | User profiles, UI state, app settings  |
| **Separation** | Modular, cross-cutting concerns | Game state, XR controllers, extensions |

---

#### Quick Examples

**Scalar (simplest):**

```ts
const count = createState(0);
```

**Explicit (apps, websites):**

```ts
const user = createState.in({
  initialState: { name: "Ben", age: 30 },
  action: { updateAge: { key: "age", fn: (age, n) => age + n } },
  storage: { key: "user", backend: "local" },
});
```

**Separation (games, extensions):**

```ts
const player = createState({ hp: 100, x: 0, y: 0 });
const move = createAction(player, {
  moveX: { key: "x", fn: (x, dx) => x + dx },
  moveY: { key: "y", fn: (y, dy) => y + dy },
});
createStorage(player, { key: "player", backend: "local", include: ["hp"] });
```

---

## How to Interact with State

### 1. Basic Component with State

```javascript
// ✅ The InSpatial Way
import { createState } from "@inspatial/kit/state";

const Counter = () => {
  const counter = createState({ count: 0 });
  return () => (
    <XStack on:tap={() => counter.count.set(counter.count.get() + 1)}>
      Count: {counter.count}
    </XStack>
  );
};
```

### 2. Conditional Rendering

```javascript
// ✅ The InSpatial Way
import { createState } from "@inspatial/kit/state";

const App = () => {
  const ui = createState({ isVisible: true });
  return (
    <XStack>
      <Show when={ui.isVisible}>{() => <Slot>Visible content</Slot>}</Show>
      <Button on:tap={() => ui.isVisible.set(!ui.isVisible.get())}>
        Toggle
      </Button>
    </XStack>
  );
};
```

### 3. List Rendering

```javascript
// ✅ The InSpatial Way
import { createState } from "@inspatial/kit/state";

const TodoList = () => {
  const state = createState({ todos: [{ id: 1, text: "Learn InSpatial" }] });
  return (
    <YStack>
      <List each={state.todos} track="id">
        {(todo) => <Text>{todo.text}</Text>}
      </List>
    </YStack>
  );
};
```

### 4. Effects and Cleanup

```javascript
// ✅ The InSpatial Way
createSideEffect(() => {
  createTrigger("swipe", createResizeHandler(), {
    platforms: ["dom", "native:ios", "native:android"],
    fallback: "resize",
  });
});
```

#### You Might Not Need Side Effects (Or Maybe You Do)

InSpatial provides **three ways** to handle lifecycle and side effects: **trigger props**, **`watch` + `onDispose`**, and **`createSideEffect`**. Each has its place.

**The 85/15 Rule:**

~85% of your use cases will use:

- Lifecycle trigger props (`on:beforeMount`, `on:mount`) for mount-time setup
- Reactive control flow (`<Show>`, `<Choose>`, `$()`) for conditional rendering
- `watch` + `onDispose` for reactive effects with cleanup

~15% will need `createSideEffect`:

- Complex subscription patterns
- Effects that return cleanup functions
- Non-UI side effects that need lifecycle integration

---

### Three Ways to Handle Lifecycle & Effects

#### **1. Lifecycle Trigger Props** (Component-bound setup)

Use for setup that's **tied to a specific component's lifecycle**:

```jsx
import { createState } from "@inspatial/kit/state";
import { View, Text } from "@inspatial/kit/structure";
import { Show } from "@inspatial/kit/control-flow";

const ui = createState({ ready: false });

<View on:beforeMount={() => ui.ready.set(true)}>
  <Show when={ui.ready}>{() => <Text>Rendered after beforeMount</Text>}</Show>
</View>;
```

**Available lifecycle triggers:**

- **`on:beforeMount`**: Runs **synchronously** during directive setup, **before first paint**. Use when initial render must reflect a state change immediately.
- **`on:mount`**: Runs on **next tick after first paint**. Use for Renderer measurements, animations, or post-render setup.

**Key characteristic:** These are **per-component**. They run when that specific widgets and component mounts and automatically clean up when they unmounts.

**Best for:**

- ✅ Renderer i.e (DOM Renderer) measurements (`getBoundingClientRect()`)
- ✅ Component-specific animations
- ✅ One-time initialization tied to element appearance
- ✅ Setup that should **not** re-run on state changes

---

#### **2. `watch` + `onDispose`** (Reactive, re-running side effects)

Use when you need side effects that **re-run when signals change**:

```jsx
import { watch, onDispose } from "@inspatial/kit/signal";
import { useTheme } from "@inspatial/kit/theme";

const dispose = watch(() => {
  const mode = useTheme.mode.get(); // Auto-tracked dependency
  console.log("Theme changed to:", mode);

  // Optional: Register cleanup (runs before re-run or disposal)
  onDispose(() => {
    console.log("Effect cleanup");
  });
});

// Later: manually stop watching
dispose();
```

**Key features:**

- **Auto-tracks dependencies**: No manual dependency arrays any `.get()` call creates a dependency
- **Runs immediately**: Effect executes on setup
- **Re-runs on changes**: Automatically re-executes when **any tracked signal** changes
- **Manual cleanup**: Use `onDispose` for cleanup logic (runs before re-run or disposal)
- **Returns disposer**: Can manually stop the effect anytime
- **Not element-bound**: Runs globally, not tied to a specific widget or component's lifecycle

**Best for:**

- ✅ Cross-state reactions (e.g., `userChanged` → `logAnalytics`)
- ✅ Logging/debugging reactive changes
- ✅ Effects that need to **re-run** when multiple signals change
- ✅ Global subscriptions (e.g., theme changes, user auth)
- ✅ When you need manual control via the disposer

---

#### **3. `createSideEffect`** (Return-based cleanup)

Use when you prefer **returning cleanup functions** instead of `onDispose`:

```jsx
import { createSideEffect } from "@inspatial/kit/state";
import { useTheme } from "@inspatial/kit/theme";

createSideEffect(() => {
  const mode = useTheme.mode.get();
  console.log("Theme changed to:", mode);

  // Return cleanup function (called automatically before re-run or disposal)
  return () => console.log("Cleanup on effect re-run or disposal");
});
```

**With timers (most common use case):**

```jsx
import { createSideEffect } from "@inspatial/kit/state";

createSideEffect(() => {
  const interval = setInterval(() => {
    console.log("Tick");
  }, 1000);

  // Cleanup automatically called when effect re-runs or disposes
  return () => clearInterval(interval);
});
```

**Key features:**

- **Auto-tracks dependencies**: Like `watch`, any `.get()` creates a dependency
- **Runs immediately**: Effect executes on setup
- **Re-runs on changes**: Like `watch`, re-executes when tracked signals change
- **Automatic cleanup**: Return a cleanup function... it's called before re-run or disposal
- **Ergonomic for timers/intervals**: Return cleanup directly (no `onDispose` needed)

**Best for:**

- ✅ Timers/intervals (return `clearInterval`/`clearTimeout`)
- ✅ Event listeners (return cleanup function)
- ✅ Subscriptions that return unsubscribe functions
- ✅ When you prefer returning cleanup over `onDispose`

**Note:** `createSideEffect` is essentially `watch` with a different cleanup API. Choose based on your cleanup style preference.

### When to Use Each Approach

#### **The Key Question: Does it need to re-run on signal changes?**

```
Does your effect need to re-run when signals change?

NO  → Lifecycle triggers (`on:mount`, `on:beforeMount`)
     ↳ Widget/Component-bound, one-time setup

YES → `watch` or `createSideEffect` (both re-run on dependencies)
     ↳ How do you prefer cleanup?
       • `onDispose(fn)` → `watch`
       • `return fn` → `createSideEffect`
```

---

#### **Comparison Table**

| Scenario                                  | Best Approach                 | Why                                                           |
| ----------------------------------------- | ----------------------------- | ------------------------------------------------------------- |
| **Widget/Component-specific mount setup** | `on:beforeMount` / `on:mount` | Element-bound, **doesn't** re-run on signal changes           |
| **DOM Renderer measurements**             | `on:mount`                    | Runs after paint when DOM is ready, **one-time**              |
| **Reactive effects (re-run on changes)**  | `watch` or `createSideEffect` | **Both** auto-track and re-run; choose cleanup style          |
| **Timers/intervals**                      | `createSideEffect`            | Return cleanup is ergonomic: `return () => clearInterval(id)` |
| **Cross-state reactions**                 | `watch` or `createSideEffect` | Both work; `watch` gives manual disposer control              |
| **Subscriptions**                         | `watch` or `createSideEffect` | Both work; choose based on cleanup style preference           |
| **Conditional rendering**                 | `$()` + `<Show>` / `<Choose>` | **No effects needed**—pure reactive control flow              |
| **Animations tied to element**            | `on:mount`                    | Element-bound, use for setup that **shouldn't** re-run        |
| **Logging signal changes**                | `watch` or `createSideEffect` | Both re-run on changes; prefer `watch` for simplicity         |

---

### Real-World Examples

#### **Example 1: One-time mount vs Reactive effect**

```jsx
import { View } from "@inspatial/kit/structure";
import { useTheme } from "@inspatial/kit/theme";
import { watch } from "@inspatial/kit/signal";

// ❌ WRONG: Using watch for one-time DOM Renderer measurement
watch(() => {
  // This will RE-RUN on ANY tracked signal change!
  // Not what you want for one-time measurement
  const height = element.getBoundingClientRect().height;
  console.log(height);
});

// ✅ CORRECT: One-time mount setup
<View
  on:mount={(e) => {
    // Runs ONCE when element mounts, never re-runs
    const height = e.target.getBoundingClientRect().height;
    console.log(height);
  }}
>
  Content
</View>;

// ✅ CORRECT: Reactive effect that re-runs on theme change
watch(() => {
  const mode = useTheme.mode.get(); // Tracks theme
  console.log(`Theme is now: ${mode}`);
  // This will RE-RUN every time theme changes
});
```

**Key takeaway:** Lifecycle triggers are for **widget/component-bound, one-time** setup. Effects (`watch`/`createSideEffect`) are for **reactive, re-running** side effects.

---

#### **Example 2: Timer with cleanup (reactive vs one-time)**

```jsx
// ✅ Option A: Reactive timer (re-runs on signal changes)
import { createSideEffect } from "@inspatial/kit/state";

const useApp = createState({ interval: 1000 });

createSideEffect(() => {
  const delay = useApp.interval.get(); // Tracks interval
  const timer = setInterval(() => console.log("Tick"), delay);

  // If interval changes, cleanup old timer and create new one
  return () => clearInterval(timer);
});

// ✅ Option B: One-time timer (element-bound)
<View
  on:mount={() => {
    const timer = setInterval(() => console.log("Tick"), 1000);

    // Return cleanup (called when element unmounts)
    return () => clearInterval(timer);
  }}
>
  Content
</View>;

// ✅ Option C: watch for manual disposer control
import { watch, onDispose } from "@inspatial/kit/signal";

const dispose = watch(() => {
  const delay = useApp.interval.get();
  const timer = setInterval(() => console.log("Tick"), delay);
  onDispose(() => clearInterval(timer));
});

// Later: stop watching manually
dispose();
```

**When to use which:**

- **Reactive timer** (depends on signals) → `createSideEffect` or `watch`
- **One-time timer** (element-bound) → `on:mount`
- **Need manual disposer** → `watch`

---

#### **Example 3: Cross-state reaction**

```jsx
import { watch } from "@inspatial/kit/signal";
import { useAuth, useAnalytics } from "@inspatial/kit/state";

watch(() => {
  const user = useAuth.user.get();
  if (user) {
    useAnalytics.track("user_login", { id: user.id });
  }
});
```

**Why `watch`?** Auto-tracks `useAuth.user`, re-runs on changes, no cleanup needed.

---

#### **Example 4: Conditional rendering (no effects needed)**

```jsx
import { Show } from "@inspatial/kit/control-flow";
import { $ } from "@inspatial/kit/state";
import { useTheme } from "@inspatial/kit/theme";

<Show
  when={$(() => useTheme.mode.get() === "dark")}
  otherwise={() => <LightUI />}
>
  {() => <DarkUI />}
</Show>;
```

**No effects needed** reactivity is built-in with control flow.

---

### ⚠️ Common Mistakes

#### **Mistake 1: Using reactive effects for one-time setup**

```jsx
// ❌ WRONG: watch will RE-RUN on ANY tracked signal
watch(() => {
  const elem = document.querySelector(".my-element");
  const height = elem.getBoundingClientRect().height;
  console.log(height);

  // If ANY signal is accessed above, this will re-run!
  // Not what you want for one-time DOM Renderer measurement
});

// ✅ CORRECT: Use lifecycle trigger for one-time setup
<View
  on:mount={(e: any) => {
    const height = e.target.getBoundingClientRect().height;
    console.log(height);
    // Runs ONCE, never re-runs
  }}
>
  Content
</View>;
```

**Why wrong?** `watch` and `createSideEffect` are for **reactive effects** that should re-run on signal changes, not one-time setup.

#### **Mistake 2: Using effects for conditional rendering**

```jsx
// ❌ WRONG: Manually toggling with effects
const showContent = createSignal(false);
watch(() => {
  if (isReady.get()) {
    showContent.set(true);
  }
});

// ✅ CORRECT: Use control flow
<Show when={isReady}>
  <Content />
</Show>;
```

### 📋 Quick Decision Tree

```
Need to do something when...

├─ Widget or Component mounts (ONE TIME, element-bound)?
│  ├─ Before first paint? → `on:beforeMount`
│  └─ After paint? → `on:mount`
│
├─ Signal changes → UI update?
│  └─ Use `$()` + `<Show>` or `<Choose>` (NO Side EFFECTS!)
│
├─ Signal changes → side effect (RE-RUNS on changes)?
│  ├─ Cleanup style: `onDispose(fn)` → `watch`
│  └─ Cleanup style: `return fn` → `createSideEffect`
│
├─ Timer/interval (RE-RUNS on signal changes)?
│  ├─ Prefer `return cleanup`? → `createSideEffect`
│  └─ Prefer `onDispose`? → `watch`
│
├─ Timer/interval (ONE TIME, element-bound)?
│  └─ Use `on:mount` + cleanup in return
│
├─ Subscription (RE-RUNS on dependencies)?
│  ├─ Need manual disposer? → `watch`
│  └─ Return cleanup? → `createSideEffect`
│
└─ User interaction?
   └─ Use trigger props (`on:tap`, `on:input`, etc.)
```

---

## Side Effects vs Scheduling & Timing

**They're related but serve different purposes.** Understanding when to use each is critical for working with InSpatial's asynchronous interactive system.

### The Relationship

```
Signal updates → Scheduled on tick → Effects run → Side effects execute
                      ↑                                    ↓
                      └─────── You can control this ───────┘
```

**Side effects** (`watch`, `createSideEffect`) are **what** you want to happen when signals change.  
**Scheduling** (`tick`, `nextTick`) is **when** you want to read updated values or ensure effects have run.

---

### Side Effects: Reactive Responses

**Purpose:** Run code automatically when signals change.

```tsx
import { watch, createSideEffect } from "@inspatial/kit/signal";

// Side effect: Logs when count changes
watch(() => {
  console.log("Count is now:", count.get());
  // Runs automatically on count changes
});

// Side effect: Updates analytics
createSideEffect(() => {
  const user = useAuth.user.get();
  if (user) {
    analytics.track("user_active", { id: user.id });
  }
});
```

**When to use:**

- ✅ Reactive logging/debugging
- ✅ Analytics tracking
- ✅ State synchronization
- ✅ Subscriptions that need to re-run on changes

**Key trait:** **Automatic** — runs when dependencies change, no manual scheduling needed.

---

### Scheduling & Timing: Controlling When Updates Propagate

**Purpose:** Control **when** you read updated values or ensure side effects have completed.

#### **`nextTick()` — Wait for Side Effects to Complete**

```tsx
import { nextTick } from "@inspatial/kit/signal";

const count = createState(0);
const doubled = $(() => count.get() * 2);

count.set(5);
console.log(doubled.get()); // ❌ Still old value (0)!

// ✅ CORRECT: Wait for propagation
await nextTick();
console.log(doubled.get()); // ✅ Now 10
```

**When to use:**

- ✅ Reading computed values after state changes
- ✅ DOM measurements after reactive updates
- ✅ Testing (ensure effects have run)
- ✅ Imperative code that depends on updated state

**Key trait:** **Manual** — you explicitly wait for the tick cycle to complete.

---

#### **`tick()` — Force Immediate Flush (Rare)**

```tsx
import { tick } from "@inspatial/kit/signal";

count.set(5);
await tick(); // Force immediate flush
// All pending effects run NOW
```

**When to use:**

- ⚠️ **Rarely needed** - InSpatial's scheduler handles most cases
- ✅ Low-level timing control
- ✅ Performance-critical synchronization
- ✅ Testing edge cases

**Warning:** Overusing `tick()` can defeat batching benefits. Prefer `nextTick()`.

---

### When to Use What?

#### **Scenario 1: React to Signal Changes**

```tsx
// ✅ Use side effects (watch/createSideEffect)
watch(() => {
  const mode = useTheme.mode.get();
  console.log(`Theme changed to: ${mode}`);
  // Runs automatically when theme changes
});

// ❌ Don't manually schedule
count.set(5);
await nextTick();
// Then do something... (this is imperative, not reactive!)
```

**Rule:** If it should **automatically re-run** on signal changes → Use side effects.

---

#### **Scenario 2: Read Updated Values After Writes**

```tsx
// ✅ Use nextTick for imperative reads
const handleSubmit = async () => {
  useForm.email.set("user@example.com");

  // ❌ WRONG: Computed not updated yet
  if (useForm.isValid.get()) { ... } // Still old value!

  // ✅ CORRECT: Wait for propagation
  await nextTick();
  if (useForm.isValid.get()) { ... } // Now updated
};

// ❌ Don't use side effects for one-time imperative logic
watch(() => {
  // This will RE-RUN on EVERY form change!
  if (useForm.isValid.get()) {
    submitForm();
  }
});
```

**Rule:** If you need updated values **right now** in imperative code → Use `nextTick()`.

---

#### **Scenario 3: DOM Measurements After Updates**

```tsx
// ✅ Combine: Side effect + nextTick + lifecycle trigger
import { View } from "@inspatial/kit/structure";

<View
  on:mount={async (e: any) => {
    // Wait for initial render to complete
    await nextTick();
    const height = e.target.getBoundingClientRect().height;
    console.log("Initial height:", height);
  }}
>
  <DynamicContent />
</View>;

// ✅ Or use side effect for reactive measurements
const contentHeight = createSignal(0);

createSideEffect(async () => {
  const content = contentRef.get();
  if (!content) return;

  // Wait for DOM update
  await nextTick();
  contentHeight.set(content.getBoundingClientRect().height);
});
```

**Rule:** DOM measurements after reactive updates → Use `nextTick()` inside effects or lifecycle triggers.

---

#### **Scenario 4: Testing**

```tsx
import { nextTick } from "@inspatial/kit/signal";

test("counter increments", async () => {
  const useCounter = createState({ count: 0 });
  const doubled = $(() => useCounter.count.get() * 2);

  useCounter.count.set(5);

  // ❌ WRONG: Computed not updated yet
  expect(doubled.get()).toBe(10); // Fails!

  // ✅ CORRECT: Wait for propagation
  await nextTick();
  expect(doubled.get()).toBe(10); // Passes
});
```

**Rule:** Testing reactive code → Always `await nextTick()` after state changes.

---

### Using Both Together

```tsx
import { watch, nextTick } from "@inspatial/kit/signal";

// Side effect that needs scheduling
watch(async () => {
  const user = useAuth.user.get();

  if (user) {
    // Update state
    useProfile.loading.set(true);

    // Wait for loading state to propagate
    await nextTick();

    // Now fetch (other components see loading state)
    const data = await fetchProfile(user.id);
    useProfile.data.set(data);
    useProfile.loading.set(false);
  }
});
```

**Pattern:** Side effects can use `nextTick()` internally for sequencing.

---

### Quick Reference

| Need                               | Use                                | Why                     |
| ---------------------------------- | ---------------------------------- | ----------------------- |
| **Automatic reaction to changes**  | `watch` / `createSideEffect`       | Re-runs automatically   |
| **Read updated value after write** | `nextTick()`                       | Wait for propagation    |
| **DOM measurements after update**  | `nextTick()` inside effect/trigger | Ensure DOM is updated   |
| **Testing reactive code**          | `await nextTick()`                 | Ensure effects ran      |
| **Force immediate flush**          | `tick()`                           | Rare, low-level control |

---

### Common Mistakes

#### **Mistake 1: Using nextTick instead of side effects**

```tsx
// ❌ WRONG: Manual scheduling for reactive logic
const logTheme = async () => {
  await nextTick();
  console.log(useTheme.mode.get());
};
// Have to call logTheme() manually every time!

// ✅ CORRECT: Automatic side effect
watch(() => {
  console.log(useTheme.mode.get());
  // Runs automatically on changes
});
```

#### **Mistake 2: Not using nextTick when reading computed values**

```tsx
// ❌ WRONG: Reading computed immediately
count.set(5);
console.log(doubled.get()); // Still old value!

// ✅ CORRECT: Wait for propagation
count.set(5);
await nextTick();
console.log(doubled.get()); // Updated value
```

#### **Mistake 3: Overusing tick() instead of nextTick()**

```tsx
// ❌ WRONG: Forcing immediate flush unnecessarily
count.set(5);
await tick(); // Defeats batching!

// ✅ CORRECT: Let scheduler batch naturally
count.set(5);
await nextTick(); // Waits for natural batch cycle
```

---

### 🎯 Bottom Line

**Side Effects vs Scheduling:**

- **Side effects** (`watch`, `createSideEffect`) = **WHAT** happens when signals change (automatic, reactive)
- **Scheduling** (`nextTick`, `tick`) = **WHEN** you read updated values (manual, imperative)

**Default approach:**

1. **Reactive behavior** → Use side effects (`watch`, `createSideEffect`)
2. **Need updated values in imperative code** → Use `nextTick()`
3. **Testing** → Always `await nextTick()` after state changes
4. **`tick()`** → Rarely needed, prefer `nextTick()`

---

### 5. Conditional Classes

```javascript
 // ✅ The InSpatial Way
<Button className="btn" class:active={isActive}>
```

#### Conditionally Rendering Computed/Reactive Values

##### `Show`

```jsx
// ❌  DON'T DO THIS: It won't react properly
<Button on:tap={() => useTheme.action.setToggle()}>
  {$(() =>
    useTheme.mode.get() === "dark" ? <LightModeIcon /> : <DarkModeIcon />
  )}
</Button>

// ✅ DO THIS: Use Show for reactive conditional rendering
<Button on:tap={() => useTheme.action.setToggle()}>
  <Show
    when={$(() => useTheme.mode.get() === "dark")}
    otherwise={() => <DarkModeIcon />}
  >
    {() => <LightModeIcon />}
  </Show>
</Button>
```

##### `Choose` - The InSpatial Switch Statement

```jsx
// ❌  DON'T DO THIS: It won't react
export function InputField({ variant }) {
  return (
    <>
      {(() => {
        switch (variant.get()) {
          case "emailfield":
            return <EmailField />;
          case "searchfield":
            return <SearchField />;
          default:
            return <TextField />;
        }
      })()}
    </>
  );
}

// ✅ DO THIS: Use Choose for multi-branch reactive logic
import { Choose } from "@inspatial/kit/control-flow";

export function InputField({ variant }) {
  return (
    <Choose
      cases={[
        {
          when: $(() => variant.get() === "emailfield"),
          children: () => <EmailField />,
        },
        {
          when: $(() => variant.get() === "searchfield"),
          children: () => <SearchField />,
        },
      ]}
      otherwise={() => <TextField />}
    />
  );
}

// Or without JSX tags:
export function InputField({ variant }) {
  return Choose({
    cases: [
      {
        when: $(() => variant.get() === "emailfield"),
        children: () => <EmailField />,
      },
      {
        when: $(() => variant.get() === "searchfield"),
        children: () => <SearchField />,
      },
    ],
    otherwise: () => <TextField />,
  });
}
```

#### Difference Between Show & Choose

- **Show** and **Choose** are both used to react to conditional values
- Use `<Show>` for **binary conditions** (this or that):
  - Example: Show loading spinner OR content
  - Example: Show dark mode icon OR light mode icon
- Use `<Choose>` for **multi-branch logic** (this, that, or another thing):
  - Example: Show different input types (email, search, text, password)
  - Example: Show different UI states (loading, error, success, empty)

### 6. Complex Reactive Expressions

```javascript
// ❌ DON'T DO THIS: Not reactive
const s = createState({ count: 0 });
const message = `Count is: ${s.count.get()}`; // Evaluates once only

// ✅ DO THIS: Wrap in $() for reactivity
import { createState, $ } from "@inspatial/kit/state";

const s = createState({ count: 0 });
const message = $(() => `Count is: ${s.count.get()}`); // Updates when count changes

// Or inline in JSX:
<XStack>{$(() => `Count is: ${s.count.get()}`)}</XStack>

// Auto-coercion also works (Symbol.toPrimitive):
<XStack>{$(() => `Count is: ${s.count}`)}</XStack>
```

### 7. Async Components

```javascript
// ✅ The InSpatial async component pattern
const PostCard = async ({ id }: PostID) => {
  const response = await fetch(`/api/story/${postId}`);
  const story = await response.json();

  return (
    <YStack className="story">
      <Text>{story.title}</Text>
      <Text>By {story.author}</Text>
    </YStack>
  );
};

// Usage with error handling
<PostCard
  id={123}
  fallback={() => <XStack>Loading...</XStack>}
  catch={({ error }) => <XStack>Error: {error.message}</XStack>}
/>;
```

### 8. State Signal Operations (Advanced)

```typescript
import { createState, $ } from "@inspatial/kit/state";

// State-backed signal comparisons and derived values
const s = createState({ count: 5, status: "loading" });
const isPositive = s.count.gt(0); // count > 0
const isZero = s.count.eq(0); // count === 0
const isEven = $(() => s.count.get() % 2 === 0);

// Derived values remain the same using $()
const label = $(() => `Status: ${s.status.get()}`);
```

### 9. Event Handling with Modifiers

```javascript
// ✅ Different event modifiers
<Button on:tap={() => console.log('normal')}>Click</Button>
<Button on-once:tap={() => console.log('only once')}>Click Once</Button>
<View on-passive:scroll={() => console.log('passive scroll')}>Scrollable</View>
<Button on-prevent:tap={() => console.log('prevented')}>Prevent Default</Button>

// State-driven event handlers
const s = createState({ count: 0 });
<Button on:tap={() => s.count.set(s.count.get() + 1)}>
  Increment ({s.count})
</Button>
```

### 10. View Literals for URLs

```javascript
// View literals with reactive interpolation (State)
import { createState } from "@inspatial/kit/state";

const s = createState({ templateId: 123 });
const templateUrl = t`https://inspatial.store/template?id=${s.templateId}`;
```

## Understanding Signal Auto-Coercion in State

Each property in a `createState` object is a signal, which means it benefits from `Symbol.toPrimitive` auto-coercion:

```typescript
const s = createState({ count: 5, name: "Ben" });

// ✅ Auto-coercion in operations
s.count > 10;              // JavaScript calls .get() automatically
s.count + 5;               // Auto-coerces to number
`Hello, ${s.name}`;        // Auto-coerces to string

// ✅ Explicit .get() (clearer intent)
s.count.get() > 10;
s.count.get() + 5;
`Hello, ${s.name.get()}`;

// ✅ Both work in JSX
<Text>{s.count}</Text>         // Renderer handles signal
<Text>{s.count.get()}</Text>   // Explicit, also valid

// ✅ Both work in comparisons
<Show when={s.count > 10} />       // Auto-coercion
<Show when={s.count.get() > 10} /> // Explicit
```

**Choose based on clarity preference** - both approaches are valid and work identically.

---

## Signal vs State: When to Use What

State is built on signals. Here's how to choose:

### 🔷 Use `createSignal` When:

- ✅ You need a **single reactive value** (counter, toggle, input)
- ✅ You're building **low-level primitives** or reusable utilities
- ✅ You want **minimal API surface** and direct control
- ✅ You're prototyping or need **maximum bundle efficiency**

**Example:**

```ts
const count = createSignal(0);
count.set(count.get() + 1); // Direct, minimal
```

---

### 🔶 Use `createState` When:

- ✅ You need **structured data** (objects with multiple fields)
- ✅ You want **batching, persistence, snapshots, or subscriptions**
- ✅ You're building **application-level state** or features
- ✅ You need a **unified action system** (`createAction`, triggers)

**Example:**

```ts
const user = createState({ name: "Ben", age: 30 });
user.batch(() => {
  user.name.set("Alice");
  user.age.set(25);
}); // All updates flush together
```

---

### Comparison Table

| Feature             | `createSignal`                          | `createState`                                      |
| ------------------- | --------------------------------------- | -------------------------------------------------- |
| **Reactivity**      | ✅ Full fine-grained                    | ✅ Full fine-grained                               |
| **Async Scheduler** | ✅ Tick-based                           | ✅ Tick-based                                      |
| **API Surface**     | Minimal (`.get()`, `.set()`, `.peek()`) | Extended (+ `.batch()`, `.reset()`, `.snapshot()`) |
| **Structure**       | Single value                            | Object with signal properties                      |
| **Batching**        | Manual (`batch()` from scheduler)       | Built-in (`.batch()`)                              |
| **Persistence**     | Manual                                  | Built-in (`createStorage`)                         |
| **Actions**         | Manual functions                        | Built-in (`createAction`, throttle/debounce)       |
| **Subscriptions**   | Manual (`watch`)                        | Built-in (`.subscribe()`)                          |
| **Use Case**        | Primitives, low-level                   | Apps, features, structured data                    |

---

### 🎯 Rule of Thumb

- **Signal**: The primitive. Use for single reactive values or when building low-level utilities.
- **State**: The pattern. Use for structured application state with features like persistence and actions.

Both share the same reactive foundation. Choose based on **structure** and **features**, not performance. State = signals + conveniences.

## State Destructure

#### Hoist shared derived values and actions once; use them everywhere

State Destructure means you define commonly used derived states and actions in one place at the top of a module, then use those everywhere else. Think of it like preparing your ingredients on a tray before you cook—less reaching, fewer mistakes, and everything stays in sync.

> **Terminology:** A “derived state is a `$(() => ...)` value built from other state; it auto‑updates when inputs change.

### Examples

#### Example 1: Sidebar state – before vs after

```jsx
// BEFORE: repeated calls scattered across components
import { $ } from "@inspatial/kit/state";
import { useSidebar } from "@inspatial/kit/navigation";

export function SidebarToggle() {
  return (
    <Slot
      className={SidebarStyle.toggle.getStyle({
        format: useSidebar.isMinimized.get() ? "minimized" : "expanded",
      })}
      on:tap={() =>
        useSidebar.action.setMinimized(!useSidebar.isMinimized.get())
      }
    />
  );
}

export function SidebarHeader({ title }: { title?: string }) {
  const isExpanded = $(() => !useSidebar.isMinimized.get());
  return <Show when={isExpanded}>{title}</Show>;
}
```

```jsx
// AFTER: consolidate once; consume everywhere
import { $ } from "@inspatial/kit/state";
import { useSidebar } from "@inspatial/kit/navigation";

// Consolidated hooks (top‑level)
export const { isMinimized, action } = useSidebar;
export const isExpanded = $(() => !isMinimized.get());
export const sidebarStateClass = $(() =>
  isMinimized.get() ? "sidebar-minimized" : "sidebar-expanded"
);

export function SidebarToggle() {
  return (
    <Slot
      className={SidebarStyle.toggle.getStyle({
        format: isMinimized.get() ? "minimized" : "expanded",
      })}
      on:tap={() => action.setMinimized(!isMinimized.get())}
    />
  );
}

export function SidebarHeader({ title }: { title?: string }) {
  return <Show when={isExpanded}>{title}</Show>;
}
```

#### Example 2: Generic UI module

```jsx
import { $, createState } from "@inspatial/kit/state";

export const ui = createState.in({
  initialState: { isOpen: false, count: 0 },
  action: {
    toggle: { key: "isOpen", fn: (v: boolean) => !v },
    inc: { key: "count", fn: (c: number, n = 1) => c + n },
  },
});

export const { isOpen, count, action: uiAction } = ui;
export const label = $(() => (isOpen.get() ? "Open" : "Closed"));

// Usage
<Text>{label}</Text>
<Button on:tap={() => uiAction.toggle()}>Toggle ({count})</Button>
```

### What You Get Back

- **Consistency:** One source of truth avoids drift across files.
- **Less boilerplate:** Fewer repeated `.get()`/action calls.

### Common Mistakes to Avoid

- Hoisting one‑off values; only consolidate what’s reused.
- Extracting values out of `$(() => ...)` just to rewrap them; pass the computed itself.
- Creating many new computeds in loops; reuse consolidated ones.

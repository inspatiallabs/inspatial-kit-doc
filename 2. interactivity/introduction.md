# Interactivity

So you've built your InSpatial widgets and components and now it's time to make them respond to users. This page is your guide from basic reactive values to sophisticated state management that works seamlessly across all environments. Think of it like learning to conduct an orchestra: you start with simple cues, then master advanced patterns that keep everything in perfect harmony.

In development, you want reactivity that's fast to write and easy to iterate on. In production, you want state that's performant, maintainable, and works reliably across all platforms. InSpatial State gives you both: an intuitive API for rapid prototyping and a powerful engine underneath that handles reactivity, persistence, actions, and automatic UI updates without you having to think about manual wiring.

InSpatial State brings together the best of signals and state management with a twist: everything is designed for universal deployment. You write reactive logic once using familiar patterns, and the system automatically adapts for web browsers, mobile apps, XR and games without you having to think about platform differences.

> **Terminology:**
>
> - **State**: The universal reactive system that manages your data and keeps UIs in sync.
> - **Signal**: The low-level primitive that powers reactivity. You won't use `createSignal()` directly; `createState()` wraps it for you.
> - **Scalar Pattern**: Single reactive value for simple counters, toggles, or standalone data.
> - **Separation Pattern**: Multiple related values grouped together as an object with controlled actions and persistence.
> - **Explicit Pattern**: Full-featured state with actions and persistence built in.
> - **Trigger**: Universal event system that handles user interactions consistently across platforms.

## Getting Started

### 🎯 Prerequisites

Before you start:

- Basic understanding of [JavaScript/TypeScript](https://www.typescriptlang.org/docs/)
- Basic understanding of [Widgets/Components](../1.%20graphical-user-interface/widgets-components/introduction.md)

### 🎮 Core Concepts

## 1. Signal

The reactive data management system that makes your application values smart and responsive

## 2. State

State in InSpatial is the core mechanism for managing reactive data in your application. It lets you create values that automatically update your UI and logic whenever they change, no manual wiring or update calls needed. State is built on top of low-level signals, but provides a much ergonomic abstraction for everyday development.

With State, you can:

- Store any kind of data (numbers, objects, arrays, etc.) and make it reactive.
- Organize your data using simple patterns, from a single value to complex objects.
- Define actions for updating your state in a clear, maintainable way.
- Optionally persist your state to storage, so data survives reloads.

State ensures that your components and logic always stay in sync with your data, making your app interactive and reliable by default.

### I. Scalar Pattern

Use this when you need a single reactive value. Perfect for states and any standalone piece of data.

```ts
import { createState } from "@inspatial/kit/state";

const useCount = createState(0);
useCount.set(useCount.get() + 1);
// or useCount.value++ (syntactic sugar for .get()/.set())
```

### II. Separation (Object) Pattern

Use this when you have multiple related values that belong together. Each property becomes its own reactive signal.

```ts
import { createState } from "@inspatial/kit/state";

const useCounter = createState({ count: 0, step: 1 });
useCounter.count.set(useCounter.count.get() + useCounter.step.get());
// or useCounter.count.value += useCounter.step.value (syntactic sugar)
```

### III. Explicit Pattern

Use this when you need the full power: actions for organized logic and storage for persistence. This is batteries-included state management.

```ts
import { createState } from "@inspatial/kit/state";

const useCounter = createState.in({
  initialState: { count: 0, step: 1 },
  action: {
    increment: { key: "count", fn: (v: number, step: number) => v + step },
    reset: { key: "count", fn: () => 0 },
  },
  storage: { key: "counter", backend: "local" },
});

// Usage with actions
useCounter.action.increment(useCounter.step.get());
useCounter.action.reset();

// Or direct property access
useCounter.count.set(useCounter.count.get() + useCounter.step.get());
// or useCounter.count.value += useCounter.step.value (syntactic sugar)
```

<details>
<summary>📦 Installation for Framework Authors</summary>

If you're building a framework or library that needs to include interactivity:

```bash
deno install jsr:@in/teract
```

##

```bash
npx jsr add @in/teract
```

##

```bash
yarn dlx jsr add @in/teract
```

##

```bash
pnpm dlx jsr add @in/teract
```

##

```bash
bunx jsr add @in/teract
```

##

```bash
vlt install jsr:@in/teract
```

</details>

---

## 3. Trigger

Trigger in InSpatial is the universal event system that handles user interactions across all platforms. It lets you respond to clicks, taps, gestures, keyboard input, gamepad controls, and app lifecycle events using a consistent `on:event` syntax no matter whether you're building for web, mobile, or spatial interfaces.

With Trigger, you can:

- Handle universal interactions (tap, longpress, hover) that work across all platforms.
- Access platform-specific events (click, back button, deep links) when needed.
- Create custom triggers for your own interaction patterns.
- Define lifecycle hooks (mount, unmount, destroy) for component management.

Trigger ensures that your event handling is consistent, type-safe, and works reliably everywhere your app runs.

### Built-in Universal Triggers

InSpatial provides triggers that work across web, mobile, and spatial platforms:

```jsx
import { Button } from "@inspatial/kit/widget";

<Button
  on:tap={() => console.log("Tapped!")}
  on:longpress={() => console.log("Long pressed!")}
  on:hover={(isHovering) => console.log("Hover:", isHovering)}
  on:key={({ key, phase }) => console.log(`Key ${key} ${phase}`)}
  on:escape={() => console.log("Escape pressed!")}
  on:gamepad={(info) => console.log("Gamepad:", info)}
>
  Interactive Button
</Button>;
```

### Creating Custom Triggers

Define your own interaction patterns that work across platforms:

```jsx
import { createTrigger } from "@inspatial/kit/trigger";
import { Slot } from "@inspatial/kit/widget";

// Create custom trigger
createTrigger(
  "swipe",
  (node, handler) => {
    let startX = 0,
      startY = 0;

    node.addEventListener("pointerdown", (e) => {
      startX = e.clientX;
      startY = e.clientY;
    });

    node.addEventListener("pointerup", (e) => {
      const deltaX = e.clientX - startX;
      const deltaY = e.clientY - startY;

      if (Math.abs(deltaX) > 50 || Math.abs(deltaY) > 50) {
        handler({
          type: "swipe",
          direction:
            Math.abs(deltaX) > Math.abs(deltaY)
              ? deltaX > 0
                ? "right"
                : "left"
              : deltaY > 0
              ? "down"
              : "up",
          distance: Math.sqrt(deltaX * deltaX + deltaY * deltaY),
        });
      }
    });
  },
  {
    platforms: ["dom", "native"],
    fallback: "tap",
  }
);

// Use custom trigger
<Slot
  on:swipe={({ direction, distance }) =>
    console.log(`Swiped ${direction} for ${distance}px`)
  }
>
  Swipe me!
</Slot>;
```

---

## What's Next?

Now that you know the basics, you might want to explore in depth:

- [Signal](./signal🟢.md) - Low-level reactive primitives
- [State](./state🟡.md) - Deep dive into state patterns and APIs
- [Trigger](./trigger🟡.md) - Universal event system and custom triggers

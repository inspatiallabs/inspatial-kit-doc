# INSPATIALIZE

**InSpatial at a glance.** InSpatialize is a technical and mindset shift. To InSpatialize is to reimage legacy technology and patterns, freeing them from their constraints, and inviting them into a world where Reality is your Canvas. Here, your old ideas find new possibilities, paradigms and breakthroughs.
InSpatialize frees you from the past and propels you into the future.

## The Complete InSpatial Kit Development Guide

> **Agentic Purpose:** This is your comprehensive reference for building InSpatial Kit components and widgets when you don't have direct access to the codebase. Think of this as your complete pattern library everything you need to build production-grade, universal components that work seamlessly across web, mobile, and spatial platforms.

---

## Table of Contents

**1. [Core Philosophy & Architecture](#core-philosophy--architecture)**

**2. [Interactivity** `(@in/teract)`**](#interactivity-interact)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](https://github.com/inspatiallabs/inspatial-kit-doc/tree/master/2.%20interactivity)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-teract/AGENTS.md)**

**3. [Widgets & Components** `(@in/widget)`**](#widgets--components-inwidget)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](https://github.com/inspatiallabs/inspatial-kit-doc/tree/master/1.%20graphical-user-interface/widgets-components)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-widget/AGENTS.md)**

**4. [Styling** `(@in/style)`**](#styling-instyle)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](https://github.com/inspatiallabs/inspatial-kit-doc/tree/master/1.%20graphical-user-interface/styling)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-style/AGENTS.md)**

**5. [Routing & Navigation** `(@in/route)`**](#routing--navigation-inroute)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](https://github.com/inspatiallabs/inspatial-kit-doc/tree/master/4.%20routing-navigation)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-route/AGENTS.md)**

**6. [Building for Dev & Production** `(@in/build)`**](#building-for-dev--production-inruntime)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](https://github.com/inspatiallabs/inspatial-kit-doc/tree/master/7.%20building-dev-production)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-build/AGENTS.md)**

**7. [Testing & Specifications** `(@in/test)`**](#testing--specifications-invader)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](https://github.com/inspatiallabs/inspatial-kit-doc/tree/master/6.%20testing-spec)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-test/AGENTS.md)**

**8. [Cloud Server & Backend** `(@in/cloud)`**](#cloud-server--backend-inspatialcloud)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](#)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](https://github.com/inspatiallabs/inspatial-kit/blob/main/modules/in-cloud/AGENTS.md)**

**9. [Distribution** <span style="color: #f39c12;">(TBD)</span>**](#distribution-tbd)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Humans](#docs-for-humans)  
&nbsp;&nbsp;&nbsp;&nbsp;• [Docs for Agents](#docs-for-agents)**

---

## Core Philosophy & Architecture

### The InSpatial Difference

InSpatial Kit is **not** React, React Native, Vue, Svelte, or Solid it's its own independent framework with its own opinions, patterns, and paradigms. Understanding this distinction is critical to using it correctly.

**Key Architectural Principles:**

1. **Retained Mode Rendering**: Direct DOM manipulation based on state, not virtual DOM diffing
2. **Signal-First Reactivity**: Everything flows from `@in/teract` signals no hooks, no proxies
3. **Universal by Default**: Write once, deploy everywhere (web, mobile, XR)
4. **Composition Over Monoliths**: Modules are independently useful, removable, and replaceable
5. **Runtime-First**: No build steps required everything works at runtime
6. **Trigger Props Only**: All event handlers use `on:event` pattern, never `addEventListener`

### Module System Overview

InSpatial follows a clear module hierarchy:

```
@in/*          → Core modules (independently useful)
@inspatial/kit → Complete framework (composed from @in modules)
```

**Prime Modules:**

- `@in/teract` → Signals, State, Triggers (Interactivity)
- `@in/widget` → UI Components & Widgets
- `@in/style` → Universal styling system (ISS)
- `@in/route` → File-based routing
- `@in/motion` → Animation & physics
- `@in/dom` → Web renderer
- `@in/gpu` → 3D/XR renderer
- `@in/runtime` → JSX runtime
- `@in/vader` → Utilities & environment detection

There are over 20 official `@in` modules.

---

## Interactivity (@in/teract)

### Signal System Architecture

InSpatial's reactivity is built on a **tick-driven, asynchronous scheduler**. This is fundamentally different from synchronous hook systems (React) or compile-time reactivity (Svelte).

**Critical Mental Model:**

- Signal updates are **never synchronous**
- All effects batch and flush on microtasks
- After `createState.value = x`, downstream computations don't update until the next tick
- Use `nextTick()` to schedule reads after mutations

### State Signals (`createState`)

**Definition:** Reactive containers that notify observers when their value changes.

```typescript
import { createState } from "@inspatial/kit/state";

// Create a state signal (Scalar Pattern) it wraps `createSignal()` api and can be swappable for it
const count = createState(0);

// Read (creates dependency)
console.log(count.get()); // or count.value

// Write
count.set(5); // or count.value = 5

// Read without creating dependency
console.log(count.peek());
```

**Signal Auto-Coercion (Important!):**

Signals implement `Symbol.toPrimitive`, enabling **automatic** unwrapping in operations:

```typescript
const count = createSignal(5);

// ✅ Auto-coercion (JavaScript calls .get() automatically)
count > 10; // Comparison
count + 5; // Arithmetic
`Count: ${count}`; // String template

// ✅ Explicit .get() (more clear about intent)
count.get() > 10;
count.get() + 5;
`Count: ${count.get()}`;

// Both approaches work - choose based on clarity preference
```

**Signal Operations (Prefer these over manual computed):**

```typescript
// Logical
count.and(otherSignal); // both truthy
count.or(otherSignal); // either truthy
count.inverse(); // !count

// Comparison
count.eq(5); // count === 5
count.neq(5); // count !== 5
count.gt(5); // count > 5
count.lt(5); // count < 5

// Conditional
count.nullishThen(fallback); // count ?? fallback
count.hasValue(); // !nullish (returns boolean)
```

### Computed Values (`$` or `computed`)

**Always wrap derived expressions** in `$(() => ...)`:

```typescript
import { $, createState } from "@inspatial/kit/state";

const count = createState(0);

// ✅ Correct: Wrapped in $()
const doubled = $(() => count.get() * 2);
const message = $(() => `Count: ${count.get()}`);

// ❌ Wrong: Not reactive
const broken = count.peek() * 2; // Evaluates once, never updates
```

**In JSX:**

```tsx
// ✅ Both work - signal passes through or explicit
<Text>{count}</Text>           // Renderer handles signal
<Text>{count.get()}</Text>     // Explicit unwrap, also valid

// ✅ Signal operations
<Text>{count.gt(0)}</Text>     // Returns Signal<boolean>

// ✅ Both work in templates
<Text>{$(() => `Count: ${count}`)}</Text>       // Auto-coercion
<Text>{$(() => `Count: ${count.get()}`)}</Text> // Explicit

// ✅ Auto-coercion in comparisons (Symbol.toPrimitive)
<Show when={count > 10} />       // JavaScript auto-calls .get()
<Show when={count.get() > 10} /> // Explicit, same result
```

### State (`createState`)

**Preferred for most use cases.** State extends signals with structure and actions.

```typescript
import { createState } from "@inspatial/kit/state";

// Simple Object state
const useCounter = createState({
  count: 0,
  name: "Counter",
});

// Access nested values (auto-signals)
useCounter.count.set(useCounter.count.get() + 1); // or useCounter.count.value++
useCounter.name.set("New Name"); // or useCounter.name.value = "New Name"

// State with actions
const useAuth = createState.in({
  id: "auth",
  initialState: {
    user: null,
    isLoggedIn: false,
  },
  action: {
    login: {
      key: "user",
      fn: (_, user) => user,
    },
    logout: {
      key: "user",
      fn: () => null,
    },
  },
  storage: {
    key: "auth",
    backend: "local",
    exclude: ["tempData"], // Don't persist this
  },
});

// Use actions
useAuth.action.login({ id: 1, name: "John" });
```

**Naming Convention:**

- State variables: `useX` (e.g., `useCounter`, `useTheme`)
- State types: `XProps` (e.g., `CounterProps`)
- Action triggers: `handleX` (e.g., `handleLogin`)
- Action declarations: `setX` (e.g., `setIncrement`)

### Side Effects (`watch` & `createSideEffect`)

**`watch`:** Auto-tracks dependencies, runs immediately, returns disposer

```typescript
import { watch } from "@inspatial/kit/signal";

const dispose = watch(() => {
  console.log("Count changed:", count.get());
  // Cleanup (optional)
  return () => console.log("Cleaning up");
});

// Later: stop watching
dispose();
```

**`createSideEffect`:** Lifecycle-aware effects with automatic cleanup

```typescript
import { createSideEffect } from "@inspatial/kit/state";

createSideEffect(() => {
  const handler = () => console.log("resize");
  window.addEventListener("resize", handler);

  // Cleanup function
  return () => window.removeEventListener("resize", handler);
});
```

### Scheduling & Timing

```typescript
import { nextTick, tick } from "@inspatial/kit/signal";

// Schedule after flush
count.set(5);
await nextTick();
console.log(doubled.get()); // Now up-to-date

// Force immediate flush (rare, use carefully)
await tick();
```

### Array/Object Mutations

**Must call `.trigger()` after mutations:**

```typescript
const state = createState({ items: [1, 2, 3] });

// ❌ Won't trigger reactivity
const arr = state.items.get();
arr.push(4);

// ✅ Correct: Trigger after mutation
const arr = state.items.get();
arr.push(4);
state.items.trigger();

// Or use immutable updates
state.items.set([...state.items.peek(), 4]);
```

### Migration from Other Frameworks

| Framework            | InSpatial Kit                               |
| -------------------- | ------------------------------------------- |
| React `useState`     | `createState`                               |
| React `useEffect`    | `watch` or `createSideEffect`               |
| React `useRef`       | `$ref` (signal or callback)                 |
| React `useMemo`      | `$(() => ...)` or signal methods            |
| Solid `createSignal` | `createSignal` (but async!)                 |
| Solid `createMemo`   | `$(() => ...)`                              |
| Solid `createEffect` | `watch`                                     |
| Solid `onCleanup`    | `onDispose`                                 |
| Vue `ref`            | `createState`                               |
| Vue `reactive`       | `createState`                               |
| Vue `watch`          | `watch` with `connect(..., false)` for lazy |
| Svelte `$:`          | `watch` or `$(() => ...)`                   |
| Svelte `onMount`     | `on:mount` trigger or `watch`               |

**Critical Differences:**

1. **Never synchronous:** All updates are async batch-flushed
2. **Signals work everywhere:** Auto-coercion via `Symbol.toPrimitive` in operations, renderer handles them in JSX
3. **Trigger props only:** `on:event`, never `addEventListener`
4. **No HTML elements:** Use InSpatial components (`<Slot>`, `<Stack>`, `<Text>`)

---

## Widgets & Components (@in/widget)

### Atomic Design Pattern

InSpatial follows a strict atomic design hierarchy:

```
Atoms      → Basic building blocks (Button, Text, Icon)
Molecules  → Simple groups (FormField = Label + Input)
Widgets    → Complex composed components (Sidebar, Table)
Templates  → Layout structures
Windows    → Complete flat/2d pages with data
Scenes     → Complete volumetric/3d pages with data
```

**Naming Convention:**

- Atoms: No prefix (e.g., `Button`, `Text`)
- Molecules: `Molecule` prefix (e.g., `MoleculeFormField`)
- Widgets: `Widget` prefix (e.g., `WidgetSidebar`)
- Templates: `Template` prefix (e.g., `TemplateError`)
- Windows: `Window` suffix (e.g., `DashboardWindow`)
- Scenes: `Scene` suffix (e.g., `FieldScene`)

### File Structure (Standard Pattern)

**Every component/widget follows this structure:**

**Standard Component/Widget Folder Structure**

| File Name                 | Purpose                                           | Required/Optional |
| ------------------------- | ------------------------------------------------- | ----------------- |
| `index.ts`                | Barrel export (exports all relevant module parts) | **Required**      |
| `type.ts`                 | TypeScript types (createType API)                 | **Required**      |
| `style.ts`                | Styling (createStyle API)                         | **Required**      |
| `component.tsx`           | Universal component implementation                | **Required**      |
| `component.web.tsx`       | Web-specific component implementation             | Optional          |
| `component.ios.tsx`       | iOS-specific component implementation             | Optional          |
| `component.android.tsx`   | Android-specific component implementation         | Optional          |
| `component.visionos.tsx`  | visionOS-specific component implementation        | Optional          |
| `component.androidxr.tsx` | AndroidXR-specific component implementation       | Optional          |
| `component.horizonos.tsx` | horizonOS-specific component implementation       | Optional          |
| `component.test.ts`       | Component Test                                    | Optional          |
| `state.ts`                | State implementation (createState API)            | Optional          |
| `controller.ts`           | Control Manipulators (createController API)       | Optional          |
| `extension.ts`            | Custom Extensions (createExtension API)           | Optional          |
| `trigger.ts`              | Custom Triggers (createTrigger API)               | Optional          |

**File Tree Example**

```
my-component/
├── index.ts
├── type.ts
├── style.ts
├── component.tsx               # Universal (default) component
├── component.web.tsx           # Web platform override
├── component.ios.tsx           # iOS platform override
├── component.android.tsx       # Android platform override
├── component.visionos.tsx      # visionOS platform override
├── component.androidxr.tsx     # AndroidXR platform override
├── component.horizonos.tsx     # horizonOS platform override
├── component.test.ts           # Optional
├── state.ts                    # Optional
├── controller.ts               # Optional
├── extension.ts                # Optional
└── trigger.ts                  # Optional
```

> When creating cross-platform components, use `component.tsx` for the universal implementation, and platform-specific files (e.g., `component.android.tsx`) to override behavior or appearance as needed. The InVader Environment API and Inbuild X InRoute system will automatically resolve the appropriate file based on the target platform and environment.

**`index.ts`** (Always export in this order):

```typescript
export * from "./component.tsx";
export * from "./type.ts";
export * from "./style.ts";
```

### Component Implementation Patterns

#### Pattern 1: Basic Component (Atom)

```tsx
// component.tsx
import { ButtonStyle } from "./style.ts";
import type { ButtonProps } from "./type.ts";
import { Slot } from "@in/widget/structure";

export function Button(props: ButtonProps) {
  const { $ref, className, format, size, ...rest } = props;

  return (
    <button
      className={ButtonStyle.wrapper.getStyle({
        format,
        size,
        className,
        ...rest,
      })}
      disabled={rest.disabled || rest.isLoading}
      $ref={$ref}
      {...rest}
    >
      {rest.children}
    </button>
  );
}
```

#### Pattern 2: Composite Widget

```tsx
// component.tsx
import { SwitchStyle } from "./style.ts";
import type { SwitchProps } from "./type.ts";
import { Slot } from "@in/widget/structure";

export function Switch(props: SwitchProps) {
  const { $ref, className, selected, size, ...rest } = props;

  return (
    <label className={SwitchStyle.wrapper.getStyle(props)}>
      <input
        type="checkbox"
        checked={selected?.get?.()}
        onChange={(e) => rest["on:input"]?.(e.target.checked)}
        className="sr-only peer"
      />

      <Slot className={SwitchStyle.track.getStyle({ size })}>
        <Slot className={SwitchStyle.handle.getStyle({ size })} />
      </Slot>
    </label>
  );
}
```

### Type Patterns

```typescript
// type.ts
import type { StyleProps } from "@in/style";
import type { ButtonStyle } from "./style.ts";
import type { JSX } from "@in/runtime";

export type ButtonProps = StyleProps<typeof ButtonStyle.wrapper> &
  JSX.SharedProps & {
    isLoading?: boolean;
    loadingText?: string;
    label?: string;
  };
```

**Key Type Principles:**

1. Always extend `StyleProps<typeof YourStyle.element>`
2. Always extend `JSX.SharedProps` for base props
3. Add component-specific props as needed
4. Export as `type`, not `interface`

### Style Patterns (`createStyle`)

```typescript
// style.ts
import { createStyle } from "@in/style";
import { ThemeRadius, ThemeBoxSize, ThemeScale } from "@in/widget/theme";

export const ButtonStyle = {
  wrapper: createStyle({
    name: "button-wrapper",

    // Base styles (always applied)
    base: [
      {
        web: {
          display: "inline-flex",
          alignItems: "center",
          justifyContent: "center",
          cursor: "pointer",
        },
      },
    ],

    // Variant settings
    settings: {
      format: {
        base: [
          {
            web: {
              backgroundColor: "var(--brand)",
              color: "var(--color-white)",
            },
          },
        ],
        outline: [
          "outline outline-2 outline-(--brand)",
          {
            web: { outlineStyle: "solid" },
          },
        ],
      },
      size: ThemeBoxSize, // Reuse theme constants
      radius: ThemeRadius,
    },

    // Default values
    defaultSettings: {
      format: "base",
      size: "md",
      radius: "md",
    },

    // Cross-style composition (advanced)
    composition: [
      {
        format: "outline",
        style: {
          web: { borderWidth: "2px" },
        },
      },
    ],
  }),
};
```

**Style Best Practices:**

1. **Use variables, not hardcoded values:** `var(--brand)` not `"#ff0000"`
2. **Reuse theme constants:** Import from `@in/widget/theme`
3. **Name your styles:** `name: "component-element"` for debugging
4. **Platform-specific:** Use `web`, `ios`, `android`, `visionOS` keys
5. **Don't use `iss()`:** Call `getStyle()` directly, it returns the className

### Import Aliases & Paths

**Always use `@in/` module aliases, never relative paths:**

```typescript
// ✅ Correct
import { Button } from "@in/widget/ornament";
import { Text } from "@in/widget/typography";
import { createStyle } from "@in/style";
import { createState } from "@in/teract/state";

// ❌ Wrong
import { Button } from "../../../widget/ornament/button";
import { createStyle } from "../../style";
```

**Alias Map:**

- `@in/widget/*` → Components/Widgets
- `@in/style/*` → Styling system
- `@in/teract/*` → Signals, state, triggers
- `@in/route/*` → Routing
- `@in/motion/*` → Animation
- `@in/runtime/*` → JSX runtime
- `@in/dom/*` → Web renderer
- `@in/gpu/*` → 3D/XR renderer

### Trigger Props (Event Handling)

**All events use trigger props:**

```tsx
// Basic triggers
<Button on:tap={() => console.log("tapped")} />
<View on:scroll={() => console.log("scrolled")} />

// Modifiers
<Button on-once:click={() => {}} />          // Fires once
<View on-passive:scroll={() => {}} />        // Passive listener
<Link on-prevent:click={() => {}} />         // preventDefault
<Input on-capture:focus={() => {}} />        // Capture phase
```

**Creating Custom Triggers:**

```typescript
import { createTrigger } from "@in/teract/trigger";

createTrigger("swipe", (node, handler) => {
  if (!handler) return;

  let startX = 0;
  const onStart = (e) => (startX = e.touches[0].clientX);
  const onEnd = (e) => {
    const endX = e.changedTouches[0].clientX;
    if (Math.abs(endX - startX) > 50) {
      handler(endX > startX ? "right" : "left");
    }
  };

  node.addEventListener("touchstart", onStart);
  node.addEventListener("touchend", onEnd);

  return () => {
    node.removeEventListener("touchstart", onStart);
    node.removeEventListener("touchend", onEnd);
  };
});

// Usage
<View on:swipe={(direction) => console.log(direction)} />;
```

### Control Flow Components

```tsx
// Show: Binary conditions
<Show when={isVisible}>
  {() => <Text>Visible content</Text>}
</Show>

<Show when={isLoggedIn} otherwise={() => <LoginPrompt />}>
  {() => <Dashboard />}
</Show>

// Choose: Multiple conditions
<Choose
  cases={[
    { when: () => status.eq("loading"), children: <Loader /> },
    { when: () => status.eq("error"), children: <Error /> },
    { when: () => status.eq("success"), children: <Content /> }
  ]}
  otherwise={<Empty />}
/>

// List: Array rendering
<List each={items} track="id">
  {(item) => <ItemCard data={item} />}
</List>

// Async: Promise handling
<Async
  future={fetchData()}
  fallback={() => <Loading />}
  catch={({ error }) => <Error message={error} />}
>
  {(data) => <DataView data={data} />}
</Async>
```

### Component Composition Best Practices

**1. Use `children` for composability:**

```typescript
export function Card(props: CardProps) {
  return (
    <Slot className={CardStyle.root.getStyle(props)}>{props.children}</Slot>
  );
}

// Usage
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
  <CardFooter>Actions</CardFooter>
</Card>;
```

**2. Provide slot customization:**

```typescript
export function Dialog(props: DialogProps) {
  const { children } = props;

  return (
    <Slot className={DialogStyle.root.getStyle(props)}>
      <Show when={children?.header}>
        <Slot className={DialogStyle.header.getStyle()}>{children.header}</Slot>
      </Show>

      <Slot className={DialogStyle.body.getStyle()}>
        {children?.body || children}
      </Slot>

      <Show when={children?.footer}>
        <Slot className={DialogStyle.footer.getStyle()}>{children.footer}</Slot>
      </Show>
    </Slot>
  );
}
```

**3. Cross-component styling (advanced):**

```typescript
// Track style references other style via $
const SwitchStyle = {
  track: createStyle({
    name: "switch-track",
    settings: { size: { sm: [...], lg: [...] } }
  }),

  handle: createStyle({
    name: "switch-handle",
    composition: [
      {
        "$switch-track.size": "sm", // Reference track's size
        style: { web: { width: "16px" } }
      },
      {
        "$switch-track.size": "lg",
        style: { web: { width: "32px" } }
      }
    ]
  })
};
```

### 🌟 Gold Standard References

**For Simple Widgets:**

- Examples: Blocks & Ornaments
- Study: [`@in/widget/ornament/button`](https://github.com/inspatiallabs/inspatial-kit/tree/main/modules/in-widget/src/ornament/button)
- Pattern: Simple, single-purpose, highly reusable
- Focus: Props → Style → Render

**For Mid-Level Widgets:**

- Examples: Inputs & Structures
- Study: [`@in/widget/input/slider`](https://github.com/inspatiallabs/inspatial-kit/tree/main/modules/in-widget/src/input/slider)
- Pattern: Stateful, interactive, bound to signals
- Focus: State management + visual feedback

**For Complex Widgets:**

- Examples: Navigations & Presentations etc...
- Study: [`@in/widget/navigation/sidebar`](https://github.com/inspatiallabs/inspatial-kit/tree/main/modules/in-widget/src/navigation/sidebar)
- Pattern: Composite (contains multiple sub-components)
- Focus: Orchestration + child coordination

---

## Styling (@in/style)

### ISS (InSpatial Style Sheet)

ISS is a universal styling system that compiles to platform-optimized Style. It bridges utility-first (Tailwind) and CSS-in-JS paradigms.

**Key Concepts:**

1. **Variant Authority:** `style` prop > `className` utilities
2. **Platform Keys:** `web`, `ios`, `android`, `visionOS`, etc.
3. **Theme Variables:** Always use CSS variables (`var(--brand)`)
4. **Cross-Platform:** Write once, optimize per platform

### createStyle API

```typescript
import { createStyle } from "@in/style";

export const MyStyle = createStyle({
  name: "my-component", // For debugging

  // Always applied
  base: [
    "inline-flex items-center", // Utility classes
    {
      web: {
        display: "inline-flex",
        alignItems: "center",
      },
    },
  ],

  // Variants
  settings: {
    size: {
      sm: ["h-8 px-3", { web: { height: "32px", padding: "0 12px" } }],
      lg: ["h-12 px-6", { web: { height: "48px", padding: "0 24px" } }],
    },
    theme: {
      light: [{ web: { backgroundColor: "$background" } }],
      dark: [{ web: { backgroundColor: "$surface" } }],
    },
  },

  // Defaults
  defaultSettings: {
    size: "sm",
    theme: "light",
  },

  // Conditional overrides (last precedence)
  composition: [
    {
      size: "lg",
      theme: "dark",
      style: {
        web: { boxShadow: "var(--shadow-lg)" },
      },
    },
  ],
});
```

### Variant Authority

```tsx
// style prop always wins
<Slot
  className="bg-purple"           // ❌ Loses
  style={{ web: { backgroundColor: "green" } }}  // ✅ Wins
/>

// Override with !
<Slot
  className="!bg-purple"          // ✅ Wins (! switches authority)
  style={{ web: { backgroundColor: "green" } }}  // ❌ Loses
/>
```

### Variable Shorthand

```typescript
// Use $ for CSS variables
{
  web: {
    color: "$primary",              // → var(--primary)
    backgroundColor: "$surface",    // → var(--surface)
    borderColor: "$brand/40"        // → color-mix with 40% opacity
  }
}

// In utilities
"text-$brand"        // → color: var(--brand)
"bg-$surface/80"     // → background with 80% opacity
```

### Reactive Styling

```tsx
import { $ } from "@inspatial/kit/state";

const isActive = createSignal(false);

// ✅ Reactive className
<Button
  className={$(() =>
    MyStyle.getStyle({
      format: isActive.get() ? "active" : "inactive"
    })
  )}
/>

// ✅ Reactive style object
<Slot
  style={$(() => ({
    web: {
      backgroundColor: isActive.get() ? "$brand" : "$surface"
    }
  }))}
/>
```

### Stateful Shell Class (SSC) Pattern

**For container-driven styling:**

```tsx
// Component
const sidebarState = $(() =>
  isMinimized.get() ? "sidebar-minimized" : "sidebar-expanded"
);

<Slot className={sidebarState}>
  <SidebarGroup /> {/* Reacts to parent class */}
</Slot>;

// Style
export const SidebarGroupStyle = createStyle({
  base: [
    {
      web: {
        padding: "10px",
        ".sidebar-minimized &": { display: "none" },
        ".sidebar-expanded &": { display: "block" },
      },
    },
  ],
});
```

### Theme System

**Define in `app.css`:**

```css
:root {
  --brand: hsla(274, 100%, 50%, 1);
  --background: hsla(0, 0%, 100%, 1);
  --surface: hsla(228, 52%, 96%, 1);
  --primary: hsla(231, 42%, 15%, 1);
}

[data-theme="dark"] {
  --background: hsla(233, 44%, 12%, 1);
  --surface: hsla(229, 41%, 18%, 1);
  --primary: hsla(0, 0%, 100%, 1);
}
```

**Use in styles:**

```typescript
{
  web: {
    backgroundColor: "var(--background)",
    color: "var(--primary)"
  }
}
```

---

## Routing & Navigation (TBD)

_This section will cover file-based routing patterns, programmatic navigation, and route composition._

---

## Cloud Server & Backend (TBD)

_This section will cover InSpatial Cloud integration patterns._

---

## Testing & Specifications (TBD)

_This section will cover testing patterns and best practices._

---

## Building For Dev & Production (TBD)

_This section will cover build configuration and optimization._

---

## Distribution (TBD)

_This section will cover publishing and deployment patterns._

---

## Learning Process Checklist

When building InSpatial components, **always follow this process:**

### 1. Study the Rules

- [ ] Read `AGENTS.md` for component patterns
- [ ] Review architectural documentation
- [ ] Check gold standard references (Button, Slider, Switch)

### 2. Understand the Context

- [ ] What atomic level? (Atom, Molecule, Widget, Template, Window, Scene)
- [ ] What module? (`@in/widget`, `@in/style`, etc.)
- [ ] What dependencies? (Only `@in/*` or `@inspatial/*`)

### 3. Apply the Pattern

- [ ] Create file structure (`index.ts`, `component.tsx`, `type.ts`, `style.ts`)
- [ ] Follow naming conventions
- [ ] Use `@in/` aliases, never relative paths
- [ ] Implement trigger props, never `addEventListener`

### 4. Validate

- [ ] Types extend `StyleProps` + `JSX.SharedProps`
- [ ] Styles use `createStyle` API
- [ ] Variables used, not hardcoded values
- [ ] Cross-platform keys where needed
- [ ] Exports in correct order (component, types, styles)

### 5. Test Patterns

- [ ] Signals work reactively
- [ ] Computed values update correctly
- [ ] Effects clean up properly
- [ ] Trigger props fire correctly
- [ ] Styles compose correctly

---

## External Knowledge Restriction

**CRITICAL:** When you don't have access to the codebase:

1. **Follow this guide exactly** don't assume patterns from other frameworks
2. **Reference gold standards:** Button (ornament), Slider/Switch (input) etc...
3. **Check documentation:** [Link](https://github.com/inspatiallabs/inspatial-kit-doc) to `@inspatial-kit-doc/` for details
4. **Ask before deviating** if a pattern isn't covered here, ask the user
5. **Never use HTML elements** always use InSpatial components

**Your tone should match the documentation:**

- Clear, conversational, first-principles
- "Think of it like..." analogies
- Step-by-step explanations
- Real-world examples
- "You might wonder..." anticipate questions

---

## Quick Reference Card

```typescript
// Interactivity
import { createSignal, $, watch } from "@in/teract/signal";
import { createState } from "@in/teract/state";

// Components
import { Button } from "@in/widget/ornament";
import { Text } from "@in/widget/typography";
import { Show, List } from "@in/widget/control-flow";
import { XStack, YStack, Slot } from "@in/widget/structure";

// Styling
import { createStyle } from "@in/style";

// Routing (TBD)
import { createRoute } from "@in/route";

// Common Patterns
const state = createState({ count: 0 });
const computed = $(() => state.count.get() * 2);
watch(() => console.log(state.count.get()));

<Button on:tap={() => state.count.set(state.count.get() + 1)}>
  {$(() => `Count: ${state.count.get()}`)}
</Button>;
```

---

**Remember:** InSpatial is **not** React. When in doubt, consult this guide, study the gold standard references, and follow the patterns exactly as documented. Your code should read like it was written by someone who deeply understands InSpatial's philosophy not someone translating from another framework.

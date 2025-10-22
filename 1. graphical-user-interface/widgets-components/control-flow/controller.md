# Controller

#### Orchestration engine for UI manipulation, forms, and everything in between

You've built your InSpatial app and it has state. Sometimes you want to let users manipulate things in real-time like colors, toggles, theme modes etc... Other times, you need to collect information, validate it, and send it somewhere i.e login forms, profile updates, checkout flows. The Controller is your answer for such scenarios. Think of a Controller as your app's universal input system.

Here's the insight: settings panels and forms are fundamentally the same thing. Both render input fields, both track what changed, both need to read and write values. The only real difference? Forms add validation rules and a submission step. So instead of maintaining two separate systems, InSpatial gives you one unified Controller with two modes.

In **manipulator mode**, changes write directly to your app state in real-time. Perfect for settings, filters, editors... anything where you want instant UI feedback as users interact.

In **form mode**, changes are held locally until the user submits. That's when validation runs and you send data to a server (or wherever). Perfect for login screens, contact forms, multi-step wizards anywhere you need to collect and validate before committing.

The magic? Both modes can work with state you already have (embedded mode) or create their own. In embedded mode, the controller operates on your existing signals, so manipulator changes propagate instantly, while form mode still holds edits locally until submission. For nested values, it's smart enough to update only what changed, keeping everything reactive and fast.

InSpatial Kit exposes controllers for every built-in widget and component, enabling you to build cross-platform, universal creative tools dramatically faster. When paired with InSpatial [Cloud](https://www.inspatial.cloud) for multi-user collaboration and backend integration, development speed and flexibility are multiplied making real-time, collaborative tool creation up to 100x more efficient.

The Controller engine serves as the core foundation of the InSpatial [App](https://www.inspatial.app) development engine, powering its UI manipulation and orchestration capabilities.

> **Terminology:**
>
> - **Manipulator mode**: Real-time control of UI state. Changes write immediately as users interact (like a theme controller or visual editor).
> - **Form mode**: Staged input with validation and submission. Changes are held locally, validated, then sent somewhere (like a login form or profile editor).
> - **Embedded mode**: The controller operates on your existing state instead of creating its own. Zero-copy, zero-sync just declarative orchestration over signals you already have.
> - **Settings schema**: The declarative list that tells the controller what fields to render and how to handle them.

### 🎯 Prerequisites

Before you start:

- Basic understanding of [CSS](https://web.dev/learn/css)
- Basic understanding of [Widgets/Components](../1.%20graphical-user-interface/widgets-components🟡.md/)
- [Popover](#) - One of the three presentation widgets that support controllers
- [Drawer](#) - Another vessel for controllers
- [Modal](#) - Full-screen controller presentation
- [Detail](#) - Another suitable vessel for controllers
- [Presentation](#) - The Presentation Widget
- [Ornament](#) - The Ornament Widget
- Optional understanding of [Interactivity](../2.%20interactivity/introduction🟡.md)

## "Manipulator" Mode

If you've used drag-and-drop tools like Figma or embedded tools like Tweakpane/Leva, the controller's manipulator mode will feel right at home. It's a compact, schema-driven "mini-editor" you can drop anywhere. We built it because InSpatial App is a UDE and UDE's have visual editors; the manipulator makes it trivial to wire settings panels that directly orchestrate live state without boilerplate.

- Own or embed state (zero-copy)
- Declarative settings schema
- Auto-render inside a Presentation Popover/Drawer/Modal/Detail
- Trigger-first events (e.g., `on:input`)

> **Note:** In embedded mode, the controller writes directly to your state signals. For nested paths, it updates the appropriate top-level signal with a merged value to keep updates efficient and reactive.

### Usage (Manipulator)

#### 1. Create Your Controller

```ts
// controller.ts
import { createController } from "@inspatial/kit/control-flow";

const ctl = createController({
  id: "example",
  mode: "manipulator",
  settings: [
    {
      name: "Show",
      path: "show",
      field: {
        type: "choice",
        component: "tab",
        options: [
          { label: "Show", value: true },
          { label: "Hide", value: false },
        ],
      },
    },
    {
      name: "Color",
      path: "color",
      field: {
        type: "color",
        component: "color",
      },
    },
  ],
});
```

#### 2. Render Your Controller

The cleanest way is to let the presentation widget auto-render it:

```jsx
import { Popover } from "@inspatial/kit/presentation";
import { Button } from "@inspatial/kit/ornament"
import { ctl } from "./controller.ts"

export function MyController() {
  return (
  // Presentation trigger
  <Button on:presentation={{ id: "simple-controller", action: "toggle" }}>
      Toggle Controller
  </Button>

  // Pass the controller and it auto-renders
  <Popover id="simple-controller" as={ctl} />;
  )
}
```

Alternatively, you can call the Controller function directly if you'd like more control over the layout. This approach is helpful when you want to display the controller alongside other components in your presentation widget.

```jsx
import { Popover } from "@inspatial/kit/presentation";
import { Controller } from "@inspatial/kit/control-flow";
import { Button } from "@inspatial/kit/ornament"
import { ctl } from "./controller.ts"


export function MyController() {
  return (
 // Presentation trigger
  <Button on:presentation={{ id: "function-controller", action: "toggle" }}>
      Toggle Controller
  </Button>

  <Popover id="function-controller">
      {Controller(ctl, {
          wrapper: { style: { web: { backgroundColor: "purple" } } }, // using style prop for customization
          label: { className: "bg-blue-500" }, // using class prop for customization
          tab: {},
          color: {},
          numberfield: {},
          switch: {},
          checkbox: {},
          radio: {},
          notSupported: {},
      })}
  </Popover>
  )
}
```

You can nest the Controller component directly. (🔴 Unstable)

```jsx
import { Popover } from "@inspatial/kit/presentation";
import { Controller } from "@inspatial/kit/control-flow";
import { Button } from "@inspatial/kit/ornament"
import { ctl } from "./controller.ts"


export function MyController() {
  return (
  // Presentation trigger
  <Button on:presentation={{ id: "component-as-controller", action: "toggle" }}>
      Toggle Controller
  </Button>

  // ⚠️ Experimental
  <Popover id="component-as-controller">
    <Controller as={ctl} />
  </Popover>;
  )
}
```

### Example: Viewport Settings (Embedded Mode)

Here's a real-world example from the InSpatial App UDE itself, showcasing a controller targeting an existing (external) state instead of managing its own. This "embedded" mode is by far the most common, as wiring controllers to external state gives you greater flexibility and integration with your app's data layer. (We'll discuss controllers with their own internal state later in the chapter, but in practice, external state is usually the best fit.)

```ts
// state.ts
import { createState } from "@inspatial/kit/state";

export const useViewport = createState.in({
  id: "viewport",

  initialState: {
    showViewportElement: true,
    color: "#343a62",
    projection: "Perspective",
    direction: "Top",
    gizmo: "AxesGizmo",
  },

  action: {
    setShowViewportElement: {
      key: "showViewportElement",
      fn: (_: string, showViewportElement: boolean) => showViewportElement,
    },
    setColor: {
      key: "color",
      fn: (_: string, color: string) => color,
    },
    setProjection: {
      key: "projection",
      fn: (_: string, projection: string) => projection,
    },
    setDirection: {
      key: "direction",
      fn: (_: string, direction: string) => direction,
    },
    setGizmo: {
      key: "gizmo",
      fn: (_: string, gizmo: string) => gizmo,
    },
  },

  storage: {
    key: "viewport",
    backend: "local",
  },
});
```

Then, we create a controller that operates on that state:

```ts
// controller.ts
import { createController } from "@inspatial/kit/control-flow";
import { useViewport } from "./state.ts";

const ctl = createController({
  id: "viewport-settings-ctl",
  mode: "manipulator",
  state: useViewport, // Embed existing state
  map: { show: "showViewportElement" }, // controllerPath -> targetPath
  settings: [
    {
      name: "Elements",
      path: "show",
      field: {
        type: "choice",
        component: "tab",
        options: [
          { label: "Show", value: true },
          { label: "Hide", value: false },
        ],
      },
    },
    {
      name: "Color",
      path: "color",
      field: { type: "alphabet", component: "color" },
    },
    {
      name: "Projection",
      path: "projection",
      field: {
        type: "choice",
        component: "tab",
        options: [
          { label: "Perspective", value: "Perspective" },
          { label: "Orthographic", value: "Orthographic" },
        ],
      },
    },
    {
      name: "Direction",
      path: "direction",
      field: {
        type: "choice",
        component: "tab",
        options: [
          { label: "T", value: "Top" },
          { label: "B", value: "Bottom" },
          { label: "L", value: "Left" },
          { label: "R", value: "Right" },
        ],
      },
    },
    {
      name: "Gizmo",
      path: "gizmo",
      field: {
        type: "choice",
        component: "tab",
        options: [
          { icon: "AxesGizmoIcon", value: "AxesGizmo" },
          { label: "None", value: "None" },
          { icon: "CubeGizmoIcon", value: "CubeGizmo" },
        ],
      },
    },
  ],
});
```

To display a control manipulator, use it within any supported Presentation widget component such as `Popover`, `Drawer`, or `Modal`. All three provide an `as` prop to accept your controller instance.

```jsx
// view.jsx
import { ctl } from "./controller.ts";
import { Popover } from "@inspatial/kit/presentation";
import { Button } from "@inspatial/kit/ornament";

// render: with popover presentation
export function ViewportController() {
  return (
      <Button on:presentation={{ id: "viewport-controller", action: "toggle" }}>
        Toggle Controller
      </Button>

      <Popover
        id="viewport-controller"
        as={ctl}
      />
  );
}
```

Now when a user changes "Projection" in the popover, `useViewport.projection` updates instantly across your entire app. No manual wiring, no event handlers, no useState calls.

### Unique or Per-User Controller Instances

When building multi-player or multi-tenant, applications or spatial experiences... each user needs their own controller instance. Think of it like a personalized remote control: everyone gets their own that only changes their settings, not someone else's.

The key is to create controllers dynamically rather than at module scope. Here's how:

#### Don't do this (shared across all users)

```ts
// ❌ Bad: All users share the same controller
import { createController } from "@inspatial/kit/control-flow";

export const SimulatorController = createController({
  id: "simulator-controller", // Single shared id
  mode: "manipulator",
  state: useSimulator, // Single shared state
  settings: [
    /* ... */
  ],
});
```

#### Do this instead (per-user instances)

```ts
// ✅ Good: Factory function creates per-user controllers
import { createController } from "@inspatial/kit/control-flow";
import type { ControllerSettingsProps } from "@inspatial/kit/control-flow/controller/type.ts";

export function SimulatorController(
  userId: string
): ControllerSettingsProps<any> {
  return createController({
    id: `${userId}-simulator-controller`, // Unique id per user
    mode: "manipulator",
    state: useSimulator, // Or pass user-specific state here
    settings: [
      {
        name: "Frame",
        path: "frame",
        field: {
          type: "choice",
          component: "tab",
          options: [
            { label: "Inner", value: "Inner" },
            { label: "Outer", value: "Outer" },
          ],
        },
      },
      // ... more settings
    ],
  });
}
```

#### Use it in your component

```jsx
import { Controller } from "@inspatial/kit/control-flow";
import { createSimulatorController } from "./simulator-controller.ts";
import { useAuth } from "./auth.ts";

export function SimulatorSettings() {
  const { userId } = useAuth;

  // Create instance once per user (stable across renders)
  const ctl = SimulatorController(userId);

  return (
    <Popover id="simulator-settings">
      <Controller as={ctl} />
    </Popover>
  );
}
```

> **Note:** Controller ids must be stable for the lifetime of the instance. Derive them from stable identifiers like `userId`, session id, or workspace id. Avoid generating unique ids with in-vaders like `createUniqueId()` or random values inside reactive code, as they'll create new controllers on every render.

> **Note:** If you need complete data isolation between users (not just separate controllers), ensure the `state` you pass is also per-user. For example, use keyed storage backends or pass different state instances per user.

## "Form" Mode (🔴 Unstable)

When building the InForm extension, we realized forms and editor manipulators share most mechanics. Forms are essentially asynchronous controllers with validation and submission. Rather than maintaining two overlapping systems, `createController` has a `mode: "form"` that layers validation semantics over the manipulator foundation.

**Coming soon:** Full form examples (validators, resolver, submission lifecycle) once final testing is complete.

---

## Field API

When you need to manually wire a field (custom UI), use `ctl.register(path)`:

```jsx
import { InputField } from "@inspatial/kit/input";
const field = ctl.register("color");
// Returns: { value, error, touched, isDirty, oninput, onchange, onblur }

// Use in custom UI:
<InputField
  variant="textfield"
  value={field.value.get()}
  on:input={(e) => field.oninput(e)}
/>;
```

- Deep paths (e.g., `"user.profile.name"`) are reactive via a proxy signal.
- Writes go through `ctl.set(path, value)`, which handles batching and validation.
- In embedded mode, validation resolver errors keyed by target paths are automatically mapped back to controller paths.

---

## Validation

- Per-field `validate(val, values)` and global `resolver(values)` are supported.
- `validateOn`: `submit | change | blur | all`.
- In embedded mode, if your resolver returns errors keyed by target paths (e.g., from a backend), the controller maps them back to controller paths automatically.

```ts
const ctl = createController({
  id: "profile-form",
  mode: "form",
  validateOn: "blur",
  settings: [
    {
      name: "Email",
      path: "email",
      field: { type: "string", component: "textfield" },
      validate: (val) => {
        if (!val.includes("@")) return "Invalid email";
      },
    },
  ],
  resolver: async (values) => {
    const res = await fetch("/api/validate", {
      method: "POST",
      body: JSON.stringify(values),
    });
    const { errors } = await res.json();
    return { errors };
  },
});
```

## Reactivity: Keeping controller UIs in sync while a presentation is open

InSpatial's controllers are built on reactive signals, which means they automatically respond to state changes. But sometimes you need more control: swapping between different controllers, showing or hiding presentations based on conditions, or refreshing content when underlying data changes. This section shows you how to handle these real-world scenarios.

> **Terminology:**
>
> - **Reactive `as`**: Passing a computed `$(() => ...)` value so the presentation reads the current controller every render.
> - **Conditional controller rendering**: Using control flow components like `Show` to display different controllers based on state.
> - **Post-flush refresh**: Waiting until batched updates finish using `nextTick()`, then reopening so layout and content are re-measured.

### 1. Swapping controllers with reactive `as`

Sometimes you want to show different controllers based on your app's state. For example, a settings panel that switches between "Viewport Settings" and "Simulator Settings" depending on what the user is editing.

This approach is best when you want the entire presentation to switch controllers automatically.

```jsx
import { Popover } from "@inspatial/kit/presentation";
import { $ } from "@inspatial/kit/state";
import { ViewportController } from "./viewport-controller.ts";
import { SimulatorController } from "./simulator-controller.ts";
import { useEditor } from "./state.ts";

export function DynamicSettingsPanel() {
  const { activeMode } = useEditor;

  return (
    <>
      <Button on:presentation={{ id: "settings", action: "toggle" }}>
        Settings
      </Button>

      <Popover
        id="settings"
        as={$(() =>
          activeMode.get() === "simulator"
            ? SimulatorController
            : ViewportController
        )}
      />
    </>
  );
}
```

The `$()` wrapper makes the `as` prop reactive. When `activeMode` changes, the popover automatically switches controllers without you having to close and reopen it manually.

> **Note:** This approach is clean and declarative, but the entire controller UI switches at once. If you want to keep some UI elements stable while only swapping the controller content, use the conditional rendering approach below.

### 2. Conditional controller rendering with manual layout

For more control over your presentation's layout, you can render controllers conditionally using `Show` or `Choose` control flows. This is perfect when you want to add tabs, headers, or other UI elements alongside your controllers.

Think of it like a settings menu where you click tabs to switch between different control panels, but the menu itself stays visible.

```jsx
import { Popover } from "@inspatial/kit/presentation";
import { Controller, Show } from "@inspatial/kit/control-flow";
import { Tab } from "@inspatial/kit/ornament";
import { YStack } from "@inspatial/kit/structure";
import { ViewportController } from "./viewport-controller.ts";
import { SimulatorController } from "./simulator-controller.ts";
import { useViewport } from "./state.ts";

export function EditorViewportSettings() {
  const { settings } = useViewport;

  return (
    <>
      <Button on:presentation={{ id: "settings", action: "toggle" }}>
        Settings
      </Button>

      <Popover id="settings">
        <YStack>
          {/* Tab switcher stays visible */}
          <Tab
            radius="md"
            defaultSelected={settings?.peek() || "Viewport"}
            on:input={(label) => settings?.set(label)}
            children={[{ label: "Viewport" }, { label: "Simulator" }]}
          />

          {/* Controllers swap based on tab selection */}
          <Show when={settings?.eq("Viewport")}>
            <Controller as={ViewportController} />
            {/**You can also use function al controller here */}
          </Show>
          <Show when={settings?.eq("Simulator")}>
            <Controller as={SimulatorController} />
            {/**You can also use function al controller here */}
          </Show>
        </YStack>
      </Popover>
    </>
  );
}
```

In this example, the `Tab` component stays mounted and visible while the `Controller` underneath swaps between `ViewportController` and `SimulatorController` based on the `settings` signal.

> **Note:** When using this approach, you're responsible for the layout structure. Use Structures like `Stack`, `Slot`, etc... to organize your tabs, controllers, and any other UI elements you want in the presentation.

> **Note:** Each `Show` block creates a separate reactive scope. When the condition changes, the old controller unmounts and the new one mounts. This is different from reactive `as`, which keeps the presentation mounted and just swaps the controller content.

### 3. Lifecycle Update: Manual refresh when signals change (using `watch` + `nextTick`)

In some cases, when the user wants to change a mostly static presentation view, you may need to force a refresh. This is helpful when the `as` prop does not change, but you want the presentation to react to internal state updates or remeasure its layout, size, or position.

```jsx
import { PresentationRegistry } from "@inspatial/kit/presentation";
import { watch, nextTick } from "@inspatial/kit/signal";
import { useViewport } from "./state.ts";

export function EditorViewport() {
  const { settings } = useViewport;

  // Watch for settings changes and refresh the open popover
  watch(() => {
    settings.get(); // Create dependency
    const id = "viewport-settings";

    if (PresentationRegistry.getOpen(id)) {
      PresentationRegistry.setOpen(id, false);
      nextTick(() => PresentationRegistry.setOpen(id, true));
    }
  });

  return (
    <>
      <Button on:presentation={{ id: "viewport-settings", action: "toggle" }}>
        Settings
      </Button>
      <Popover id="viewport-settings" as={ViewportController} />
    </>
  );
}
```

The `watch` effect observes `settings`. If it changes while the popover is open, we close it, wait for the scheduler flush with `nextTick()`, then reopen it. This ensures the presentation rebinds to the latest state and recalculates its position.

---

<details>
  <summary><strong>Genius</strong></summary>

The Controller acts purely as an orchestration layer it doesn't create new UI primitives. Every field you define in the schema is mapped directly to a standard InSpatial widget. Most of these are Input widget variants such as `Switch`, `TextField`, `ColorPicker`, alongside the Ornament `Tab` and the Presentation widget `Dropdown`. The schema itself is simple metadata: it tells the controller which widget to render and exactly how to wire up its reactivity.

**Key architectural insights:**

- **Zero abstraction tax**: The rendered components are the actual widgets from `@in/widget`, not wrapper proxies. This means styling, accessibility, and behavior are identical whether you write `<Switch>` manually or let the controller render it.

- **Path-to-signal mapping**: When you call `ctl.register("theme.mode")`, the controller resolves that path to the correct signal in your state tree (or its own internal state). For embedded mode, it uses `cfg.map` to alias controller paths to target state paths, enabling zero-copy orchestration.

Controllers operate on signals, which batch updates and flush at the end of each tick. Presentations like `Popover` measure their position based on layout, so if you change controller content, the popover might be in the wrong place until it remeasures. That's why `nextTick()` exists: it schedules code to run after the flush, ensuring the layout is settled before you trigger remeasure logic.

- **Shallow reconstruction for nested writes**: Deep path updates like `ctl.set("viewport.grid.color", "#ff0000")` don't mutate nested objects. Instead, the controller reads the top-level signal's current value, creates a shallow copy with the nested change applied, then writes back to the top-level signal. This preserves reactivity while avoiding unnecessary re-renders.

- **Field component inference**: The `field.component` property maps to a specific widget type. The controller uses this to conditionally render the correct JSX in a single `Choose` block, passing the registered signal's getter/setter as props. This keeps the rendering logic linear and predictable.

If you know how to style a `Switch`, you already know how to style it in a controller. Same components, same APIs, just declaratively assembled as a widget tree.

</details>

---

## API Reference

### createController(config)

| Parameter      | Type                                         | Required | Description                                                                         |
| -------------- | -------------------------------------------- | -------- | ----------------------------------------------------------------------------------- |
| `id`           | `string`                                     | ✅       | Unique identifier for the controller                                                |
| `mode`         | `"manipulator" \| "form"`                    | ❌       | Controller mode. Defaults to `"manipulator"`                                        |
| `state`        | `State<T>`                                   | ❌       | External state to embed (zero-copy). If provided, controller operates on this state |
| `map`          | `Record<string, string>`                     | ❌       | Path aliasing map. Maps controller paths to target state paths                      |
| `initialState` | `T`                                          | ❌       | Initial state (manipulator mode). Used when `state` is not provided                 |
| `initialValue` | `Partial<T>`                                 | ❌       | Initial values (form mode). Used when `state` is not provided                       |
| `settings`     | `ControllerSettingItem<T>[]`                 | ❌       | Schema defining the fields to render                                                |
| `validateOn`   | `"submit" \| "change" \| "blur" \| "all"`    | ❌       | When to trigger validation (form mode)                                              |
| `resolver`     | `(values: Partial<T>) => Promise<{errors?}>` | ❌       | Global validation resolver (form mode)                                              |
| `storage`      | `any`                                        | ❌       | Storage configuration for persisting state                                          |

**Returns:** `ControllerInstance<T>`

### ControllerSettingItem

| Property       | Type                                                                   | Required | Description                                    |
| -------------- | ---------------------------------------------------------------------- | -------- | ---------------------------------------------- |
| `name`         | `string`                                                               | ✅       | Display label for the field                    |
| `path`         | `string`                                                               | ❌       | State path. Defaults to slugified `name`       |
| `initialValue` | `any`                                                                  | ❌       | Initial value for this field                   |
| `description`  | `string`                                                               | ❌       | Optional description (may render as tooltip)   |
| `field`        | `FieldFor<T>`                                                          | ✅       | Field configuration (type, component, options) |
| `validate`     | `(val, values) => string \| undefined \| Promise<string \| undefined>` | ❌       | Field-level validator                          |

### Field Types

| Type         | Component Options                                                | Options Required | Props                    |
| ------------ | ---------------------------------------------------------------- | ---------------- | ------------------------ |
| `"alphabet"` | `"textfield"` \| `"color"`                                       | ❌               | Text/color input props   |
| `"numeric"`  | `"numberfield"`                                                  | ❌               | Number input props       |
| `"choice"`   | `"tab"` \| `"select"` \| `"radio"` \| `"switch"` \| `"checkbox"` | ✅               | Component-specific props |

### ControllerInstance Methods

| Method          | Signature                                               | Description                                                |
| --------------- | ------------------------------------------------------- | ---------------------------------------------------------- |
| `set`           | `<P>(path: P, value: ValueAtPath<T, P>, opts?) => void` | Set field value with optional validation/touch/dirty flags |
| `register`      | `<P>(path: P, opts?) => RegisteredField`                | Register a field and get its reactive API                  |
| `validateField` | `<P>(path: P, value?) => Promise<string \| undefined>`  | Validate a single field                                    |
| `validateForm`  | `() => Promise<Record<string, string \| undefined>>`    | Validate all fields                                        |
| `handleSubmit`  | `(success, error?) => (e?) => void`                     | Create submit handler (form mode)                          |
| `reset`         | `(values?, opts?) => void`                              | Reset form state                                           |

### RegisteredField

| Property   | Type                              | Description                                |
| ---------- | --------------------------------- | ------------------------------------------ |
| `name`     | `string`                          | Field path                                 |
| `value`    | `Signal<T>`                       | Reactive value signal (read with `.get()`) |
| `error`    | `Signal<string \| undefined>`     | Reactive error message                     |
| `touched`  | `Signal<boolean>`                 | Whether field has been focused             |
| `isDirty`  | `Signal<boolean>`                 | Whether value differs from initial         |
| `oninput`  | `(evOrVal: Event \| any) => void` | Input event handler                        |
| `onchange` | `(ev?: Event) => void`            | Change event handler                       |
| `onblur`   | `(ev?: Event) => void`            | Blur event handler                         |

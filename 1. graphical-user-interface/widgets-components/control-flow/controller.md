# Controller

#### Orchestration engine for UI manipulation, forms, and everything in between

Think of a Controller as your app's universal input system. Whether you're building a settings panel like Figma's design controls, or a user registration form that talks to your backend, the Controller handles both with the same elegant API.

You've built your InSpatial app and it has state. Sometimes you want to let users manipulate things in real-time colors, toggles, theme modes. Other times, you need to collect information, validate it, and send it somewhere i.e login forms, profile updates, checkout flows. The Controller is your answer for both scenarios.

Here's the insight: settings panels and forms are fundamentally the same thing. Both render input fields, both track what changed, both need to read and write values. The only real difference? Forms add validation rules and a submission step. So instead of maintaining two separate systems, InSpatial gives you one unified Controller with two modes.

In **manipulator mode**, changes write directly to your app state in real-time. Perfect for settings, filters, editors... anything where you want instant UI feedback as users interact.

In **form mode**, changes are held locally until the user submits. That's when validation runs and you send data to a server (or wherever). Perfect for login screens, contact forms, multi-step wizards anywhere you need to collect and validate before committing.

The magic? Both modes can work with state you already have (embedded mode) or create their own. In embedded mode, the controller operates on your existing signals, so manipulator changes propagate instantly, while form mode still holds edits locally until submission. For nested values, it's smart enough to update only what changed, keeping everything reactive and fast.

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
- Optional understanding of [Interactivity](../2.%20interactivity/introduction🟡.md)

## "Manipulator" Mode

If you've used drag-and-drop tools like Figma or embedded tools like Tweakpane/Leva, the controller's manipulator mode will feel right at home. It's a compact, schema-driven "mini-editor" you can drop anywhere. We built it because InSpatial Apps is a UDE and UDE's have visual editors; the manipulator makes it trivial to wire settings panels that directly orchestrate live state without boilerplate.

- Own or embed state (zero-copy)
- Declarative settings schema
- Auto-render inside a Presentation Popover/Drawer/Modal
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
        type: "boolean",
        component: "switch",
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

```tsx
import { Popover } from "@inspatial/kit/presentation";
import { Button } from "@inspatial/kit/ornament"

// Presentation trigger
<Button on:presentation={{ id: "simple-controller", action: "toggle" }}>
    Toggle Controller
</Button>

// Pass the controller and it auto-renders
<Popover id="simple-controller" as={ctl} />;
```

Or, nest the Controller component if you want to customize the layout:

```tsx
import { Popover } from "@inspatial/kit/presentation";
import { Controller } from "@inspatial/kit/control-flow";
import { Button } from "@inspatial/kit/ornament"

// Presentation trigger
<Button on:presentation={{ id: "component-as-controller", action: "toggle" }}>
    Toggle Controller
</Button>

// ⚠️ Experimental
<Popover id="component-as-controller">
  <Controller as={ctl} />
</Popover>;
```

Or, call the Controller function directly for full style control:

```tsx
import { Popover } from "@inspatial/kit/presentation";
import { Controller } from "@inspatial/kit/control-flow";
import { Button } from "@inspatial/kit/ornament"

// Presentation trigger
<Button on:presentation={{ id: "function-controller", action: "toggle" }}>
    Toggle Controller
</Button>

// ⚠️ Experimental
<Popover id="function-controller">
  {{
    view: Controller(ctl, {
      root: { className: "p-[16px] gap-[16px]" },
      wrapper: { className: "items-center" },
      label: { className: "text-sm" },
      tab: {},
      color: {},
      numberfield: {},
      switch: {},
      checkbox: {},
      radio: {},
      notSupported: {},
    }),
  }}
</Popover>;
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
        type: "boolean",
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
      field: { type: "color", component: "color" },
    },
    {
      name: "Projection",
      path: "projection",
      field: {
        type: "enum",
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
        type: "enum",
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
        type: "enum",
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

```tsx
// view.tsx
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

## "Form" Mode (🔴 Unstable)

When building the InForm extension, we realized forms and editor manipulators share most mechanics. Forms are essentially asynchronous controllers with validation and submission. Rather than maintaining two overlapping systems, `createController` has a `mode: "form"` that layers validation semantics over the manipulator foundation.

**Coming soon:** Full form examples (validators, resolver, submission lifecycle) once final testing is complete.

---

## Field API

When you need to manually wire a field (custom UI), use `ctl.register(path)`:

```ts
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

| Type        | Component Options                     | Options Required      | Props                    |
| ----------- | ------------------------------------- | --------------------- | ------------------------ |
| `"string"`  | `"textfield"`                         | ❌                    | Any text input props     |
| `"color"`   | `"color"`                             | ❌                    | HTML color input props   |
| `"number"`  | `"numberfield"`                       | ❌                    | Any number input props   |
| `"boolean"` | `"switch"` \| `"checkbox"` \| `"tab"` | ❌ (optional for tab) | Component-specific props |
| `"enum"`    | `"tab"` \| `"select"` \| `"radio"`    | ✅                    | Component-specific props |

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

---

## Related

- [createState](./../../../2.%20interactivity/state🟡.md) - InSpatial's state managerment system
- [Popover](#) - One of the three presentation widgets that support controllers
- [Drawer](#) - Another vessel for controllers
- [Modal](#) - Full-screen controller presentation
- [Presentation](#) - The Presentation Widget

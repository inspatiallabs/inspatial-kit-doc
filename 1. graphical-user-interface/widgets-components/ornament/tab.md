# Tab

## Anatomy

Historically Tabs have been used as owners of a view. But this is merely the case because the anatomy of a tab warrants that a Tab is simply a controller of a view.

### Boolean Mapping

By default, controllable Tabs use string values for their data. However, in many cases especially when there are only two possible tab selections you can opt for a boolean data type instead. This is particularly useful for toggles or yes/no choices.

To keep your UI and state perfectly in sync, give each tab item a boolean value and bind the Tab to your boolean state. That way, the Tab emits true/false directly, and your UI responds without extra glue code.

> **Terminology:**
>
> - “Controlled” Tab: you pass `selected={yourstate}` so the Tab reflects your state.
> - Item `value`: attach `value: true/false` to each tab item so the tab trigger i.e `on:input` or `on:change` receives a boolean instead of a string.
> - Boolean mapping: converting string labels (e.g., “Show”/“Hide”, “Yes”/“No” ) to true/false when items don’t carry `value`.

```
┌───────────────────────────── Tab: Show / Hide ─────────────────────────────┐
│ children:                                                                  │
│   • { label: "Show", value: true }                                         │
│   • { label: "Hide", value: false }                                        │
│                                                                            │
│ state binding:                                                             │
│   selected ─────────────► showViewportElement (boolean)                    │
│   on:input(val:boolean) ─► showViewportElement.set(val)                    │
└────────────────────────────────────────────────────────────────────────────┘

Render:
┌── when true ────────────────────────────────────────────────────────────────┐
│ <Show when={showViewportElement}> ... </Show>                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

Example

```jsx
import { Tab } from "@inspatial/kit/ornament"
import { Show } from "@inspatial/kit/control-flow"
import { Slot } from "@inspatial/kit/structure"

// state
const { showViewportElement } = useViewport;

// controlled Tab: emits boolean; drives the same boolean state
<Tab
  children={[
    { label: "Show", value: true },
    { label: "Hide", value: false },
  ]}
  selected={showViewportElement}
  on:input={(value: boolean) => {
    showViewportElement.set(!!value);
  }}
/>

// reactively show content
<Show when={showViewportElement}>
  <Slot>Visible when true</Slot>
</Show>
```

Notes

- Prefer passing an explicit `value` per item so `on:input` receives the boolean directly.
- Make the Tab controlled via `selected={yourBooleanstate}` to avoid extra toggles.
- If you need an expression: for Interactive Usage

do:

```jsx
<Show when={showViewportElement}>
```

Or:

```jsx
<Show when={$(() => showViewportElement.get() === true)}>
```

Or:

```jsx
<Show when={showViewportElement.eq(true)}>
```

#### Alternative mapping (strings → boolean)

```jsx
const MAP: Record<string, boolean> = { Show: true, Hide: false };
<Tab
  children={[{ label: "Show" }, { label: "Hide" }]}
  on:input={(label: string) => showViewportElement.set(MAP[label] ?? false)}
/>;
```

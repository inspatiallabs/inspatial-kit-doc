# Counter

#### A flexible number input with increment, decrement, and reset controls

The `Counter` component is like a smart calculator button pad for your app. Think of it as the controls you'd find on a volume knob, a quantity picker in a shopping cart, or any place where users need to step a number up or down with effectively. It handles the math for you, lets users tap to change values one step at a time, or hold down a button to keep counting perfect for when you need precise control over numbers without typing.

Counter automatically wires up its three child buttons (increment, decrement, reset) and keeps everything in sync. If you pass it an external value (like a signal from your app state), it updates that signal directly. If you don't, it manages its own internal count. Either way, you get consistent, interactive behavior out of the box.

> **Note:** The reset button only appears when the counter value is greater than zero or less than zero—it hides when the count is exactly zero, keeping the UI clean when there's nothing to reset.

> **Note:** The Counter component acts as a unique hybrid: it's more than a simple input or display element but not quite a full widget as its still a component or variant of the InputField widget. Unlike standard components or typical widgets, Counter hosts multiple child variants such as a an Input NumberField, a Visualization ProgressBar and an Input Slider. This flexibility makes it a versatile building block within your UI, bridging the gap between simple controls and fully featured widgets.

> **Terminology:**
>
> - **Increment** means adding to the current number (going up).
> - **Decrement** means subtracting from the current number (going down).
> - **Reset** means returning the number back to zero.
> - **Press-&-hold** means tapping and holding a button to continuously repeat the action at a set speed (rate limit).

<details>
  <summary><strong>Genius</strong></summary>

Counter uses a layered prop destructuring pattern that cleanly separates style, behavior, and content for each child button. The component performs auto-detection of external value signals:

- If `props.value` has a `.set` method (e.g., `createState.in` signal), Counter uses that directly.
- If `props.value` is a standard `@in/teract` signal, it updates via `write()`.
- Otherwise, it falls back to its own internal `useCounter` state a state exposed for the Counter controller.

All mutations (tap, press-and-hold, NumberField input, reset) route through a single setter (`externalSet` or internal action) to guarantee consistency. The `on:presshold` trigger is configured with `{ fn, interval, immediate }` to enable continuous increment/decrement when the user holds a button. The default interval is 100ms, but you can tune it with `rateLimit.interval`.

The reset button visibility is computed reactively using a `$()` expression that reads the current value (external or internal) and shows/hides based on whether it's non-zero.

Icon rendering supports three formats:

- JSX node: `<BrandIcon size="4xs" />`
- Variant string: `"ArrowUpIcon"`
- IconProps object: `{ variant: "ArrowUpIcon", size: "2xs" }`

The helper `getCounterIcon` handles all three cases with a fallback to default InSpatial icons (CrosshairIcon, PlusIcon, MinusIcon).

</details>

### Examples

Here's how you might use Counter in real scenarios:

#### Example 1: Basic Counter with External State

```tsx
import { Counter } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";

// Create a simple state for your counter
const useQuantity = createState(0);

function ShoppingCart() {
  return <Counter value={useQuantity} format="Number" />;
}
```

When the user taps increment, the count goes up by 1. When they tap decrement, it goes down by 1. The NumberField shows the current value and lets them type a number directly.

#### Example 2: Custom Increment/Decrement Steps

```tsx
import { Counter } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";

const useVolume = createState(0);

function VolumeControl() {
  return (
    <Counter
      value={useVolume}
      format="Number"
      children={{
        increment: { value: 10 }, // Increase by 10 each time
        decrement: { value: 10 }, // Decrease by 10 each time
      }}
    />
  );
}
```

Instead of changing by 1, the volume now jumps by 10 with each tap perfect for controls where small steps aren't useful.

#### Example 3: Custom Icons

```tsx
import { Counter } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";
import { BrandIcon } from "@inspatial/kit/icon";

const useScore = createState(0);

function GameScore() {
  return (
    <Counter
      value={useScore}
      format="None" // Hide the input field
      children={{
        reset: { icon: <BrandIcon size="4xs" /> },
        increment: { icon: "ArrowUpIcon" },
        decrement: { icon: "ArrowDownIcon" },
      }}
    />
  );
}
```

Now your counter uses custom icons and hides the number input field, showing only buttons great for a game scoreboard.

#### Example 4: Adjusting Press-and-Hold Speed

```tsx
import { Counter } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";

const useTimer = createState(0);

function TimerControl() {
  return (
    <Counter
      value={useTimer}
      format="Number"
      rateLimit={{
        interval: 50, // Repeat every 50ms when holding
        immediate: true, // Fire once immediately on press
      }}
    />
  );
}
```

With `rateLimit`, you control how fast the counter increments/decrements when the user holds down a button. A 50ms interval feels super snappy, while 200ms is more deliberate. InSpatial defaults to 100ms.

#### Example 5: Horizontal and Vertical Layouts

```tsx
import { Counter } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";

const useCount = createState(0)

function LayoutDemo() {
  return (
      {/* Horizontal layout (default) */}
      <Counter value={useCount} axis="x" />

      {/* Vertical layout */}
      <Counter value={useCount} axis="y" />
  );
}
```

The `axis` prop controls the layout: `"x"` arranges components horizontally, `"y"` arranges them vertically.

### What You Get Back

The `Counter` component doesn't return a value it renders buttons and optional input widget variants (NumberField & Slider) as well as Visualization widget variants (ProgressBar). When the user interacts with it, the component automatically updates the `value` signal you passed in (or its own internal state if you didn't pass one).

### Common Mistakes to Avoid

- **Not binding to an external signal:** If you want Counter to control your app's state, pass `value={yourSignal}`. Otherwise, Counter's internal state won't be accessible to the rest of your app.
- **Expecting reset to always show:** The reset button only appears when the value is non-zero. If it's hidden, check the current value it's likely at zero or you set the counter countrols reset to hidden.
- **Forgetting format prop:** If you want the NumberField input visible, set `format="Number"`. If you only want buttons, use `format="None"` or don't pass any format.
- **Overriding triggers without understanding precedence:** If you pass `children.increment["on:tap"]`, your handler replaces the default. Make sure your custom handler updates the value signal if you need that behavior.

### API Reference

| Prop                 | Type                                         | Default                  | Description                                                                                                          |
| -------------------- | -------------------------------------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| `value`              | Signal or reactive value                     | Internal state           | The value to control. Pass an external signal to sync with your app state.                                           |
| `format`             | `"None" \| "Number"`                         | `"None"`                 | Whether to show a NumberField input. `"Number"` shows the input, `"None"` hides it.                                  |
| `axis`               | `"x" \| "y"`                                 | `"x"`                    | Layout direction. `"x"` arranges buttons horizontally, `"y"` vertically.                                             |
| `rateLimit`          | `{ interval?: number; immediate?: boolean }` | `{ interval: 100 }`      | Controls press-and-hold behavior. `interval` is milliseconds between repeats; `immediate` fires once on press start. |
| `children.reset`     | `CounterResetProps`                          | Default reset button     | Customize the reset button (icon, style, handlers).                                                                  |
| `children.increment` | `CounterIncrementProps`                      | Default increment button | Customize the increment button (icon, value step, style, handlers).                                                  |
| `children.decrement` | `CounterDecrementProps`                      | Default decrement button | Customize the decrement button (icon, value step, style, handlers).                                                  |

### Related Components

- [Icon](../../icon/icon.md) - Used for button icons when you pass a variant string
- [Button](#) - The underlying component for increment/decrement/reset buttons
- [NumberField](#) - The input field rendered when `format="Number"`
- [Slider](#) – Underlying slider for slider format
- [ProgressBar](#) – Visualization used by progress format

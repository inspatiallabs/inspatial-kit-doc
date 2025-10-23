# Slider

#### A slider consists of a horizontal track featuring a movable handle, which users can slide between designated minimum and maximum values.

The `Slider` component is like a volume knob on your stereo, but for any number you can imagine. Think of it as your friendly assistant that helps users pick a value by sliding a handle along a track, whether they're adjusting the temperature in a room (say, from 60° to 80°), setting their budget for a purchase ($100 to $5000), or choosing how many guests to invite to a party (1 to 100).

Unlike text inputs where you type numbers freely, a slider naturally enforces boundaries. When you create a slider, you decide where it starts (minimum), where it ends (maximum), and how big each step should be. This makes it perfect for situations where you want users to stay within sensible limits without having to type or validate numbers manually.

The Slider follows the InSpatial atomic design pattern, which means it's built from smaller, reusable pieces that work together beautifully. You get a track (the line the handle slides on), a range indicator (the colored portion showing your selection), a handle (the circle or square you drag), and optional markers (little numbers or dots below the track). Every single piece can be customized independently, so you can make the slider look exactly how you want.

What makes InSpatial's Slider special is its composability. You can start with the simple base format that includes tick marks and labels, or switch to the sleek bare format for a minimal, focused experience. Want to show the current value in a little input box? Just flip a switch (well, a prop). Need edge labels but want to hide the middle numbers? That's built in too. The slider automatically keeps everything consistent when you make the track bigger, the handle grows to match. When you round the track's corners, the handle follows suit. No manual coordination needed.

> **Note:** On InSpatial Kit, the Number Field is intentionally designed without built-in min and max value controls. We believe that components should remain unrestricted, allowing for maximum flexibility. When you need to enforce specific minimum and maximum values, the Slider component is the ideal choice. Its very nature is defined by clear start and end points, making it perfect for selecting values within a defined range. Using each component for its intended purpose ensures clarity and best practices.

> **Terminology:**
>
> - **Track**: The horizontal line or bar that represents your full range of possible values (like the groove on a physical slider).
> - **Handle**: The draggable circle or square that you move to select a value (sometimes called a "thumb" or "knob").
> - **Range**: The filled portion of the track showing your current selection (the colored part from the start to your handle).
> - **Markers**: Optional tick marks or numbers below the track that help users understand the scale.
> - **Format**: The visual style variant "base" includes full markers and labels, while "bare" is minimal and clean.

<details>
  <summary><strong>Genius</strong></summary>

### Technical Architecture

The Slider component is built using InSpatial's `createStyle` API with 10 distinct anatomical parts:

1. **wrapper** - Root container managing layout and format spacing
2. **input** - Native HTML range input (hidden, provides accessibility and keyboard navigation)
3. **trackContainer** - Flexbox wrapper for track and edge labels
4. **track** - Visual track with format-specific borders and sizing
5. **range** - Absolutely positioned filled indicator with reactive width
6. **handle** - Absolutely positioned draggable visual indicator with reactive left position
7. **value** - Optional input/display component for bare format
8. **markers** - Container for tick marks in base format
9. **marker** - Individual tick mark (dot or label variant)
10. **edgeLabel** - Min/max labels for bare format with truncateEdge

### State Management

The component uses a computed signal pattern for optimal reactivity:

```jsx
// Unwraps signals or plain values
const currentValue = $(() => {
  if (value !== undefined) {
    return unwrapValue(value);
  }
  return defaultValue ?? min;
});

// Computed percentage for positioning
const rangePercentage = $(() => {
  return ((currentValue.get() - min) / (max - min)) * 100;
});
```

### Reactive Styling

Dynamic styles use InSpatial's reactive style pattern:

```jsx
style={$(() => ({
  web: {
    width: `${rangePercentage.get()}%`,
    transitionDuration: dragging.get() ? "0ms" : "150ms",
  },
}))}
```

This ensures the range and handle update immediately during drag (0ms transition) and animate smoothly when released (150ms transition).

### Cross-Reference Compositions

The component uses composition rules to automatically synchronize related styles:

```jsx
handle: {
  composition: [
    { "$slider-track.size": "sm", style: { width: "16px", height: "16px" } },
    { "$slider-track.size": "md", style: { width: "20px", height: "20px" } },
    { "$slider-track.size": "lg", style: { width: "24px", height: "24px" } },
    { "$slider-track.radius": "md", style: { borderRadius: "var(--radius-xs)" } },
  ],
}
```

This eliminates manual coordination and ensures visual consistency.

### Event Handling

The slider uses equality guards to prevent infinite update loops:

```jsx
function handleChange(e: Event) {
  const target = e.target as HTMLInputElement;
  const newValue = Number(target.value);
  if (newValue !== currentValue.get()) {
    on:input?.(newValue);
  }
}
```

### Accessibility

The component maintains a hidden web native `<input type="range">` element that for `component.web.tsx` provides:

- Full keyboard navigation (arrow keys, page up/down, home/end)
- Screen reader support via ARIA attributes
- Standard form integration via name attribute
- Focus management and disabled state handling

</details>

---

## Examples

Here's how you might use the Slider in real life:

### Temperature Range Selector

Let's say you're building a thermostat control for a smart home app. You want users to pick a comfortable temperature:

```jsx
import { Slider } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";

const targetTemp = createState(72); // Start at 72°F

<Slider
  min={60} // Minimum comfortable temperature
  max={80} // Maximum comfortable temperature
  step={1} // Adjust by whole degrees
  value={targetTemp}
  format="base" // Full format with markers
  size="md" // Medium size looks good
  radius="rounded" // Rounded edges feel friendly
  truncate={{ mid: false }} // Show all the temperature numbers
  on:input={(temp) => {
    targetTemp.set(temp);
    // Update your thermostat here
    updateThermostat(temp);
  }}
/>;
```

### Custom Styled Slider

Want to make the slider match your brand colors? Customize every piece:

```jsx
import { Slider } from "@inspatial/kit/input";

<Slider
  min={0}
  max={100}
  step={5}
  value={75}
  format="bare"
  size="lg"
  radius="md"
  className="my-custom-slider"
  children={{
    track: {
      style: {
        web: { backgroundColor: "red" },
      },
    },
    range: {
      style: {
        web: { backgroundColor: "green" },
      },
    },
    handle: {
      style: {
        web: {
          borderColor: "blue",
          backgroundColor: "yellow",
          borderRadius: "1px",
        },
      },
    },
  }}
/>;
```

The `children` prop lets you override styles for any part of the slider. Change colors, sizes, borders whatever you need.

### Uncontrolled Mode with Default Value

If you don't need to control the slider externally, use uncontrolled mode:

```jsx
import { Slider } from "@inspatial/kit/input";

<Slider
  min={1}
  max={10}
  step={1}
  defaultValue={5} // Start at 5, but we don't track changes
  format="base"
  size="md"
  on:input={(value) => {
    // Do something when the user finishes adjusting
    console.log("User selected:", value);
  }}
/>;
```

The slider manages its own internal state. You just provide a `defaultValue` and an `on:input` callback.

### Large Slider with Full Customization

For important controls, go big and add all the bells and whistles:

```jsx
import { Slider } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";
import { XStack } from "@inspatial/kit/structure";
import { Text } from "@inspatial/kit/typography";

const quality = createState(75);

<XStack>
  <Text>Video Quality</Text>

  <Slider
    min={0}
    max={100}
    step={5}
    value={quality}
    format="base"
    size="lg"
    radius="rounded"
    truncate={{ mid: false }} // Show all numbers
    on:input={(q) => {
      quality.set(q);
      updateVideoQuality(q);
    }}
    on:blur={() => {
      console.log("User finished adjusting quality");
      savePreferences();
    }}
  />

  <Text>Current quality: {quality.get()}%</Text>
</XStack>;
```

The `on:blur` trigger fires when the user stops interacting with the slider, perfect for saving settings or triggering expensive operations.

### Negative Range Slider

Sliders aren't just for positive numbers. How about adjusting an equalizer or audio balance?

```jsx
import { Slider } from "@inspatial/kit/input";
import { createState } from "@inspatial/kit/state";

const balance = createState(0); // Center balance

<Slider
  min={-100} // Full left
  max={100} // Full right
  step={10}
  value={balance}
  format="bare"
  size="md"
  radius="full"
  truncate={{ edge: true }}
  on:input={(bal) => {
    balance.set(bal);
    if (bal < 0) {
      console.log(`Audio shifted ${Math.abs(bal)}% to the left`);
    } else if (bal > 0) {
      console.log(`Audio shifted ${bal}% to the right`);
    } else {
      console.log("Audio centered");
    }
  }}
/>;
```

Perfect for pan controls, balance adjustments, or any value that can go negative.

---

## What You Get Back

The `Slider` component renders a complete slider interface with all the interactive and visual elements you need:

- A draggable handle that users can click and drag
- A visual track showing the full range
- A colored range indicator showing the current selection
- Optional markers with numbers or dots below the track
- Optional value display (in bare format) showing the exact number
- Optional edge labels (min/max values) for bare format
- Full keyboard support (arrow keys, page up/down, home/end)
- Smooth animations when you release the handle
- Instant updates while dragging (no lag or delay)

When a user interacts with the slider, your `on:input` callback receives the new numeric value immediately. That value is always a number between your `min` and `max`, moving in steps of `step`.

---

## Properties

### Required Props

| Prop   | Type     | Description                                                       |
| ------ | -------- | ----------------------------------------------------------------- |
| `min`  | `number` | The minimum value the slider can reach (e.g., 0 for volume)       |
| `max`  | `number` | The maximum value the slider can reach (e.g., 100 for percentage) |
| `step` | `number` | How much the value changes with each movement (e.g., 1, 5, 10)    |

### Value Props

| Prop           | Type                          | Default     | Description                                                                    |
| -------------- | ----------------------------- | ----------- | ------------------------------------------------------------------------------ |
| `value`        | `number \| Signal<number>`    | `undefined` | Current value for controlled mode. Can be a plain number or a reactive signal. |
| `defaultValue` | `number`                      | `min`       | Initial value for uncontrolled mode (when you don't provide `value`)           |
| `on:input`     | `(value: number) => void`     | `undefined` | Callback fired when the slider value changes                                   |
| `on:blur`      | `(event: FocusEvent) => void` | `undefined` | Callback fired when the slider loses focus                                     |

### Style Props

| Prop       | Type                   | Default     | Description                                                                 |
| ---------- | ---------------------- | ----------- | --------------------------------------------------------------------------- |
| `format`   | `"base" \| "bare"`     | `"base"`    | Visual variant. "base" includes full markers and labels. "bare" is minimal. |
| `size`     | `"sm" \| "md" \| "lg"` | `"md"`      | Size of the track and handle                                                |
| `radius`   | `"rounded" \| "md"`    | `"rounded"` | Border radius style. "rounded" is fully round, "md" has subtle rounding.    |
| `disabled` | `boolean`              | `false`     | When true, the slider cannot be interacted with                             |

### Behavior Props

| Prop        | Type                                | Default                       | Description                                                                                                                  |
| ----------- | ----------------------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `truncate`  | `{ mid?: boolean, edge?: boolean }` | `{ mid: false, edge: false }` | Control visibility of marker labels. `mid` hides middle numbers (shows dots), `edge` hides min/max labels (base format only) |
| `showValue` | `boolean`                           | `false`                       | Show an input box displaying the current value (bare format only)                                                            |

### Standard Props

| Prop        | Type     | Description                                                 |
| ----------- | -------- | ----------------------------------------------------------- |
| `id`        | `string` | Unique identifier for the slider (useful for accessibility) |
| `name`      | `string` | Name attribute for form submission                          |
| `className` | `string` | Additional CSS classes for the wrapper                      |
| `class`     | `string` | Alternative to className (alias)                            |
| `style`     | `object` | Inline styles for the wrapper                               |
| `$ref`      | `Ref`    | Reference to the underlying input element                   |

### Advanced Composition

| Prop       | Type     | Description                                                |
| ---------- | -------- | ---------------------------------------------------------- |
| `children` | `object` | Fine-grained customization of each slider part (see below) |

The `children` prop accepts an object with these optional keys:

- `trackContainer` - Customize the track container wrapper
- `track` - Customize the track background
- `range` - Customize the filled range indicator
- `handle` - Customize the draggable handle
- `value` - Customize the value input/display
- `markers` - Customize the markers container
- `edgeLabels` - Customize min/max edge labels with `{ min: {...}, max: {...} }`

Each key accepts `className`, `class`, and `style` properties, plus any applicable style settings from `SliderStyle`.

---

## Style Customization

### Using SliderStyle

The `SliderStyle` object contains all the default styles. You can access and customize individual parts:

```jsx
import { SliderStyle } from " @inspatial/kit/input";
import { Slot } from " @inspatial/kit/structure";

// Get styles for a specific part
const SliderTrackStyles = SliderStyle.track.getStyle({
  format: "bare",
  size: "lg",
  radius: "md",
});

// Use with inspatial style sheet (ISS) for custom components
import { iss } from "@inspatial/kit/style";

<Slot className={iss(SliderTrackStyles)}>{/* Custom track element */}</Slot>;
```

### Available Style Parts

1. `SliderStyle.wrapper` - Root container
2. `SliderStyle.input` - Hidden input (accessibility)
3. `SliderStyle.trackContainer` - Track and labels wrapper
4. `SliderStyle.track` - Visual track
5. `SliderStyle.range` - Filled range indicator
6. `SliderStyle.handle` - Draggable handle
7. `SliderStyle.value` - Value display/input
8. `SliderStyle.markers` - Markers container
9. `SliderStyle.marker` - Individual marker
10. `SliderStyle.edgeLabel` - Min/max labels

### Cross-Reference Magic

The slider automatically keeps related parts in sync. When you change the track size, the handle size adjusts to match. When you change the track radius, the handle and range follow suit. No manual coordination needed!

```jsx
<Slider
  size="lg" // Track becomes large...
  radius="md" // Track gets squared corners...
  // Handle automatically becomes large with md corners
  // Range automatically gets md corners
  // Everything stays perfectly consistent
/>
```

This is powered by InSpatial's composition system working behind the scenes.

---

## Accessibility

The Slider component is built with accessibility in mind from the ground up:

### Keyboard Navigation

Users can control the slider entirely with their keyboard:

- **Arrow Keys** (← →): Move by one step
- **Page Up/Down**: Move by larger increments (10 steps)
- **Home**: Jump to minimum value
- **End**: Jump to maximum value

### Screen Readers

The component uses a native HTML `<input type="range">` element under the hood, which means screen readers automatically understand it. They'll announce:

- The current value
- The minimum and maximum
- The step size
- When the value changes

### Focus Management

The slider responds to focus like any standard form control:

```jsx
<Slider
  id="volume-slider"
  min={0}
  max={100}
  step={1}
  on:blur={() => {
    console.log("User finished adjusting");
  }}
/>
```

### Disabled State

When `disabled={true}`, the slider:

- Changes cursor to `not-allowed`
- Reduces opacity to show it's inactive
- Prevents all [trigger](../../../../2.%20interactivity/trigger🟡.md) interactions
- Maintains visual consistency with other disabled inputs

---

## Common Mistakes to Avoid

### Don't Forget on:input in Controlled Mode

If you provide a `value` prop (controlled mode), you MUST provide an `on:input` callback. Otherwise, the slider will be stuck and won't move:

```jsx
// ❌ Bad: Value but no on:input
<Slider
  value={myValue}
  on:input={undefined} // Slider is stuck!
/>

// ✅ Good: Value with on:input
<Slider
  value={myValue}
  on:input={(newValue) => setMyValue(newValue)}
/>
```

### Don't Use Tiny Steps for Large Ranges

If you have a range from 0 to 10,000 with `step={1}`, users will have to drag very precisely. Consider larger steps:

```jsx
// ❌ Hard to use: 10,000 tiny steps
<Slider min={0} max={10000} step={1} />

// ✅ Better: 200 larger steps
<Slider min={0} max={10000} step={50} />
```

### Don't Mix Controlled and Uncontrolled

Pick one approach either provide both `value` and `on:input`, or provide only `defaultValue`:

```jsx
// ❌ Confusing: Both value and defaultValue
<Slider
  value={myValue}
  defaultValue={50}  // Which one wins?
  on:input={setMyValue}
/>

// ✅ Good: Controlled mode
<Slider value={myValue} on:input={setMyValue} />

// ✅ Good: Uncontrolled mode
<Slider defaultValue={50} />
```

### Don't Forget About Signals

If you're using InSpatial's reactive signals, make sure to call `.get()` and `.set()` properly:

```jsx
import { createState } from "@inspatial/kit/state";

const volume = createState(50);

// ✅ Good: Pass the signal directly or use .get()
<Slider
  value={volume}  // Signal unwrapped automatically
  on:input={(val) => volume.set(val)}  // Use .set() to update
/>

// ❌ Bad: Passing signal.get() directly
<Slider
  value={volume.get()}  // Creates a snapshot, won't update
  on:input={(val) => volume = val}  // Can't reassign signals
/>
```

When you pass the signal object itself as `value`, the slider automatically unwraps it and stays reactive. When you call `volume.get()`, you create a one-time snapshot that won't update.

---

## Controller Integration

The Slider component integrates seamlessly with InSpatial's Controller system for managing form state and numeric manipulations. This is perfect for settings panels, configuration UIs, or any scenario where you need centralized state management.

### Basic Slider Controls

```jsx
import { Controller, createController } from "@inspatial/kit/control-flow";

const SliderController = createController({
  id: "volume-control",
  initialValue: { volume: 50 },
  settings: [
    {
      name: "Volume",
      path: "volume",
      field: {
        type: "numeric",
        component: "slider",
        props: {
          min: 0,
          max: 100,
          step: 1,
          format: "bare",
          size: "lg",
          radius: "full",
          showValue: true,
        },
      },
    },
  ],
});

// Render Slider Controller
<Controller as={SliderController} />;
```

### Available Slider Props in Controller

When using the slider in a controller, you can specify these props in the `field.props` object:

| Prop        | Type                                | Default     | Description                         |
| ----------- | ----------------------------------- | ----------- | ----------------------------------- |
| `min`       | `number`                            | `0`         | Minimum slider value                |
| `max`       | `number`                            | `100`       | Maximum slider value                |
| `step`      | `number`                            | `1`         | Step increment                      |
| `format`    | `"base" \| "bare"`                  | `"base"`    | Slider format variant               |
| `size`      | `"sm" \| "md" \| "lg"`              | `"md"`      | Slider size                         |
| `radius`    | `"rounded" \| "md" \| "full"`       | `"full"`    | Border radius style                 |
| `truncate`  | `{ mid?: boolean, edge?: boolean }` | `undefined` | Control marker label visibility     |
| `showValue` | `boolean`                           | `false`     | Show value input (bare format only) |
| `disabled`  | `boolean`                           | `false`     | Disable the slider                  |

### Controller Methods

Access and manipulate slider values programmatically:

```jsx
import { Controller, createController } from "@inspatial/kit/control-flow";

const controller = createController({
  id: "programmatic-settings",
  initialValue: { volume: 50 },
  settings: [
    {
      name: "Volume",
      path: "volume",
      field: {
        type: "numeric",
        component: "slider",
        props: { min: 0, max: 100, step: 1, radius: "full" },
      },
    },
  ],
});

// Set value programmatically
controller.set("volume", 75);

// Get registered field
const volumeField = controller.register("volume");
console.log(volumeField.value.get()); // Current value

// Check if field is dirty (changed)
console.log(volumeField.isDirty.get()); // true/false

// Reset to initial value
controller.reset({ volume: 50 });

// Validate specific field
await controller.validateField("volume");

// Get all errors
controller.validateForm().then((errors) => {
  console.log(errors);
});
```

---

## Related Components

- **[NumberField](../numberfield/numberfield.md)** - Use this when you need unrestricted numeric input without min/max boundaries. Sliders are for ranges, NumberFields are for freeform numbers.
- **[Counter](../counter/counter.md)** - For precise numeric input with increment/decrement buttons.
- **[Controller](../../control-flow/controller.md)** - State management system for forms and manipulators.

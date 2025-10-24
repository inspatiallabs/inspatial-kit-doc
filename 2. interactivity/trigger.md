# Trigger

#### The simple, universal way to handle interactive events everywhere

Whether you're building for web, mobile, or XR, triggers in InSpatial give you an easy way to handle all kinds of user interactions. This page is your starting point think of triggers as the bridge that lets you listen and respond to taps, gestures, lifecycle events, and more, no matter the platform.

There are two main ways triggers work in InSpatial:

- **Trigger Props**: These are convenient attributes you add directly to your components or elements, like `on:tap`, `on:mount`, or even things like `style:*` and `class:*`. The platform takes care of wiring up the exact behavior you need, automatically adapting for browsers, mobile, or XR environments.
- **Action Triggers**: These are for situations where you need to change or update state in response to events. Defined using `createAction()`, you use these when you want to run some logic that affects your app’s data. (See the [State documentation](/2.%20interactivity/state.md) for full details.)

> **Terminology:**
>
> - **Trigger Prop**: A declarative attribute for handling user input or lifecycle events, recognized by special `on:*`, `style:*`, or `class:*` prefixes.
> - **Action Trigger**: An imperative state-changing function that lets you update your app’s data in response to something happening.

---

## Getting Started

### Enabling Trigger Props

**Trigger Props are disabled by default.** Install the `InTrigger` extension to enable resolution of `on:*`, `style:*`, `className:*` and `class:*` props.

```typescript
import { createRenderer } from "@inspatial/kit/renderer";
import { InTrigger } from "@inspatial/kit/trigger";

createRenderer({
  mode: "auto",
  extensions: [InTrigger()],
});
```

### Import Surface

```typescript
import { InTrigger, createTrigger } from "@inspatial/kit/trigger";
```

---

## Universal Trigger Props

Universal Trigger Props work across all platforms (web, native, XR). InSpatial automatically maps these to platform-specific implementations.

### Lifecycle Triggers

Lifecycle triggers fire during component mount/unmount cycles or on every frame.

#### **on:beforeMount**

Fires **synchronously** during directive setup, **before first paint**.

```tsx
<View
  on:beforeMount={() => {
    // ✅ Safe for state setup, timers, subscriptions
    // ⚠️ Avoid reading DOM layout; node may not be connected yet
    useApp.initialized.set(true);
  }}
/>
```

**When to use:**

- State initialization before render
- Synchronous setup that affects initial render
- Pre-paint configuration

---

#### **on:mount**

Fires on **next tick after first paint** when the component is mounted and connected to the DOM Renderer.

```tsx
<View
  on:mount={(e: any) => {
    // ✅ Node is connected; safe for DOM reads/writes
    const height = e.target.getBoundingClientRect().height;
    console.log("Element height:", height);
  }}
/>
```

**When to use:**

- DOM measurements (`getBoundingClientRect()`)
- Post-render setup
- Element-specific initialization
- Anything that requires the node to be connected

---

#### **on:frameChange**

Triggers every frame via a single global `requestAnimationFrame` loop. Perfect for animations, games, and XR applications.

**Performance:** One RAF loop for the entire app, not per node. Auto-cleans disconnected nodes and stops when no subscribers exist.

```tsx
import { Model } from "@inspatial/kit/media";
const ui = createState({ fps: 0, rotation: 0 });

<Model
  on:frameChange={({
    delta,
    frame,
    time,
    elapsed,
  }: {
    delta: number; // ms since last frame
    frame: number; // monotonic frame index
    time: number; // high-resolution timestamp
    elapsed: number; // ms since loop started
  }) => {
    // Update FPS every 10 frames
    if (frame % 10 === 0) {
      const fps = Math.max(1, Math.round(1000 / (delta || 16)));
      ui.fps.set(fps);
    }

    // Update rotation
    ui.rotation.set((ui.rotation.get() + delta * 0.001) % (Math.PI * 2));
  }}
/>;
```

**When to use:**

- 3D animations and XR
- Games requiring per-frame updates
- Performance monitoring
- Smooth animations tied to frame timing

---

### Input Triggers

Input triggers handle user interactions across platforms.

#### **on:tap**

Universal tap/click interaction. Maps to `click` on web, touch events on mobile.

```tsx
import { Button } from "@inspatial/kit/ornament"

<Button on:tap={() => console.log("Tapped!")}>Click Me</Button>

// With event details
<Button on:tap={(e: any) => console.log("Coordinates:", e.clientX, e.clientY)}>
  Click Me
</Button>
```

**Platform mapping:**

- **Web/DOM**: `click` event
- **Mobile/Native**: Touch events
- **XR**: Spatial pointer events

---

#### **on:longpress**

Fires when user presses and holds for **500ms** (default).

```tsx
import { Button } from "@inspatial/kit/ornament";
<Button
  on:longpress={() => {
    console.log("Long press detected!");
    showContextMenu();
  }}
>
  Press & Hold
</Button>;
```

**Implementation:** Synthesized from pointer/touch events with timeout.

---

#### **on:presshold**

Repeatedly fires a callback **while the pointer is down**. Great for continuous actions like scrolling or adjusting values.

```tsx
import { Button } from "@inspatial/kit/ornament"

// Basic usage (fires every 50ms by default)
<Button
  on:presshold={() => {
    incrementValue();
  }}
>
  Hold to Increment
</Button>

// Advanced: custom interval and immediate fire
<Button
  on:presshold={{
    fn: () => incrementValue(),
    interval: 100, // Fire every 100ms
    immediate: true, // Fire immediately on press
  }}
>
  Hold to Increment Fast
</Button>
```

**Options:**

- `fn`: Callback function
- `interval`: Ms between fires (default: 50)
- `immediate`: Fire immediately on press start (default: false)

---

#### **on:rightclick**

Context menu trigger (right-click on web). Automatically prevents default context menu.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:rightclick={(e: any) => {
    console.log("Right clicked at:", e.clientX, e.clientY);
    showCustomContextMenu(e.clientX, e.clientY);
  }}
>
  Right-click for options
</View>;
```

**Platform mapping:**

- **Web/DOM**: `contextmenu` event (with `preventDefault()`)
- **Mobile**: Long-press gesture
- **XR**: Secondary button

---

#### **on:change**

Fires when an input value changes.

```tsx
<Input
  on:change={(e: any) => {
    useForm.email.set(e.target.value);
  }}
/>
```

**Common use cases:**

- Form inputs
- Select dropdowns
- Textarea

---

#### **on:submit**

Fires when a form is submitted.

```tsx
import { Form } from "@inspatial/kit/data-flow";

<Form
  on:submit={(e: any) => {
    e.preventDefault();
    handleFormSubmit();
  }}
>
  <Button type="submit">Submit</Button>
</Form>;
```

---

#### **on:focus**

Fires when an element gains focus.

```tsx
<Input
  on:focus={() => {
    console.log("Input focused");
    useUI.inputActive.set(true);
  }}
/>
```

---

### Hover Triggers

Hover triggers track pointer enter/leave states.

#### **on:hover**

Fires with a **boolean** indicating hover state.

```tsx
import { Button } from "@inspatial/kit/ornament";

<Button
  on:hover={(isHovering: boolean) => {
    useUI.buttonHover.set(isHovering);
    console.log(isHovering ? "Entered" : "Left");
  }}
>
  Hover me
</Button>;
```

**Payload:** `true` on enter, `false` on leave.

---

#### **on:hoverstart**

Fires only when the pointer **enters** the element.

```tsx
import { View } from "@inspatial/kit/structure";

<View on:hoverstart={() => console.log("Hover started")}>Content</View>;
```

**Platform mapping:** `pointerenter` on web.

---

#### **on:hoverend**

Fires only when the pointer **leaves** the element.

```tsx
import { View } from "@inspatial/kit/structure";

<View on:hoverend={() => console.log("Hover ended")}>Content</View>;
```

**Platform mapping:** `pointerleave` on web.

---

### Keyboard Triggers

Keyboard triggers provide unified keyboard handling across phases.

#### **on:key**

Unified keyboard trigger for both `down` and `up` phases.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:key={(e: {
    type: "keytap";
    phase: "down" | "up";
    key: string;
    code: string;
    repeat: boolean;
    event: KeyboardEvent;
  }) => {
    if (e.key === "Enter" && e.phase === "down") {
      submitForm();
    }
    if (e.key === "Escape" && e.phase === "down") {
      closeModal();
    }
  }}
/>;
```

**Payload:**

- `type`: `"keytap"`
- `phase`: `"down"` or `"up"`
- `key`: Key name (e.g., `"Enter"`, `"a"`)
- `code`: Physical key code (e.g., `"KeyA"`)
- `repeat`: Whether key is being held
- `event`: Native `KeyboardEvent`

---

#### **on:keydown**

Phase-specific sugar for key down events.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:keydown={(e: any) => {
    if (e.key === "ArrowLeft") moveLeft();
    if (e.key === "ArrowRight") moveRight();
  }}
/>;
```

**Payload:** Same as `on:key`.

---

#### **on:keyup**

Phase-specific sugar for key up events.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:keyup={(e: any) => {
    if (e.key === " ") stopJump(); // Stop jumping when space released
  }}
/>;
```

**Payload:** Same as `on:key`.

---

#### **on:escape**

Convenience sugar for the Escape key (fires on key down).

```tsx
import { Model } from "@inspatial/kit/media"


<Modal on:escape={() => closeModal()}>
  {/* Modal content */}
</Modal>

// Equivalent to:
<Modal on:key={(e: any) => e.key === "Escape" && e.phase === "down" && closeModal()}>
  {/* Modal content */}
</Modal>
```

**Note:** Direct key triggers for other keys (like `on:enter`, `on:space`) will be available soon.

---

### Gamepad Triggers

Gamepad triggers provide console-style controller support via the Gamepad API.

#### **on:gamepad**

Polls the first connected gamepad state (~every 100ms) and fires only when state changes.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:gamepad={(gp: {
    connected: boolean;
    // Face buttons
    buttonA: boolean;
    buttonB: boolean;
    buttonX: boolean;
    buttonY: boolean;
    // Joysticks (left/right)
    joystick: [number, number]; // [x, y] for left stick
    joystickRight: [number, number]; // [x, y] for right stick
    // Shoulder buttons
    RB: boolean;
    LB: boolean;
    RT: boolean;
    LT: boolean;
    // System buttons
    start: boolean;
    select: boolean;
    // D-pad
    up: boolean;
    down: boolean;
    left: boolean;
    right: boolean;
  }) => {
    if (!gp.connected) return;

    // Handle button presses
    if (gp.buttonA) action.jump();
    if (gp.start) action.pause();

    // Handle joystick movement with deadzone
    const [lx, ly] = gp.joystick;
    const deadzone = 0.15;
    const dx = Math.abs(lx) < deadzone ? 0 : lx;
    const dy = Math.abs(ly) < deadzone ? 0 : ly;
    if (dx !== 0 || dy !== 0) {
      movePlayer(dx, dy);
    }
  }}
/>;
```

**Performance:** One poll loop for all subscribers. Stops automatically when no listeners exist.

**Platform support:** Web browsers with Gamepad API. No-op on unsupported platforms.

---

#### **on:gamepadconnect**

Fires when a gamepad is connected.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:gamepadconnect={(e: {
    type: "gamepadconnect";
    index?: number;
    id?: string;
  }) => {
    console.log("Gamepad connected:", e.id, "at index:", e.index);
    showGamepadIndicator();
  }}
/>;
```

---

#### **on:gamepaddisconnect**

Fires when a gamepad disconnects.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:gamepaddisconnect={(e: {
    type: "gamepaddisconnect";
    index?: number;
    id?: string;
  }) => {
    console.log("Gamepad disconnected:", e.id);
    hideGamepadIndicator();
  }}
/>;
```

---

## Renderer-Platform Triggers

### Capacitor Renderer (Mobile App) Triggers

When running as a Capacitor app, InSpatial provides mobile-specific triggers via the Capacitor App API.

**Renderer:** `capacitor` (mobile WebView + native bridge)

#### **on:back**

Fires when the Android back button is pressed.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:back={() => {
    if (canGoBack()) {
      navigateBack();
    } else {
      showExitConfirmation();
    }
  }}
/>;
```

**Platform:** Android only.

---

#### **on:resume**

Fires when the app returns to the foreground.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:resume={() => {
    console.log("App resumed");
    refreshData();
    resumeAudio();
  }}
/>;
```

---

#### **on:pause**

Fires when the app goes to the background.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:pause={() => {
    console.log("App paused");
    saveState();
    pauseAudio();
  }}
/>;
```

---

#### **on:urlopen**

Fires when the app is opened via a deep link.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:urlopen={(e: { type: "urlopen"; url: string }) => {
    console.log("Opened from URL:", e.url);
    handleDeepLink(e.url);
  }}
/>;
```

**Use cases:**

- Deep link navigation
- OAuth callbacks
- Share target handling

---

#### **on:statechange**

Fires when the app state changes (active/inactive/background).

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:statechange={(e: { type: "statechange"; isActive: boolean }) => {
    console.log("App state:", e.isActive ? "active" : "inactive");
  }}
/>;
```

---

#### **on:restored**

Fires when the app is restored from a saved state (e.g., after being killed by the system).

```tsx
import { View } from "@inspatial/kit/structure"

<View
  on:restored={(e: any) => {
    console.log("App restored");
    restoreState(e: any);
  }}
/>
```

---

## Extension Triggers

Extensions can provide custom triggers via `capabilities.triggers`. These triggers become available when the extension is registered.

### InRoute Extension

#### **on:route**

Fires when navigation occurs in your application.

```tsx
import { View } from "@inspatial/kit/structure"

<View
  on:route={(e: {
    path: string;
    params?: Record<string, string>;
    query?: Record<string, string>;
    name?: string;
  }) => {
    console.log("Navigated to:", e.path);
    trackPageView(e.path);
  }}
/>

// Or on a Link
<Link to="/dashboard" on:route={() => console.log("Navigating to dashboard")}>
  Dashboard
</Link>
```

**Note:** Uses a global document listener to capture all navigation events.

---

### InPresentation Extension

#### **on:presentation**

Controls modals, drawers, popovers and other presentation components.

```tsx
import { Button } from "@inspatial/kit/ornament"

<Button
  on:presentation={{
    id: "user-modal",
    action: "toggle", // "open" | "close" | "toggle"
  }}
>
  Open User Modal
</Button>

// Or with callback
<Button
  on:presentation={(state: { id: string; visible: boolean }) => {
    console.log(`Modal ${state.id} is now ${state.visible ? "open" : "closed"}`);
  }}
>
  Toggle Modal
</Button>
```

---

### InCloud Extension

The InCloud extension provides real-time cloud connectivity triggers.

#### **on:cloudStatus**

Monitors the real-time connection status with the cloud backend.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:cloudStatus={(e: {
    type: "cloudStatus";
    status: "connecting" | "open" | "closed" | "error" | "reconnected";
  }) => {
    useUI.cloudStatus.set(e.status);

    if (e.status === "error") {
      showErrorNotification("Cloud connection lost");
    }
  }}
/>;
```

---

#### **on:cloudReconnected**

Fires specifically when the cloud connection is re-established after a disconnect.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:cloudReconnected={() => {
    console.log("Cloud reconnected");
    // Refresh data after reconnection
    fetchLatestData();
    showSuccessNotification("Connection restored");
  }}
/>;
```

---

#### **on:cloudNotify**

Receives notifications pushed from the cloud backend.

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:cloudNotify={(e: { type: string; title?: string; message: string }) => {
    showToast({
      title: e.title,
      message: e.message,
      type: e.type,
    });
  }}
/>;
```

---

## Trigger Modifiers

Trigger modifiers change how event listeners are attached. Use them with any `on:` trigger.

### **on-once:**

Fire the trigger only once, then automatically remove the listener.

```tsx
import { Button } from "@inspatial/kit/ornament";

<Button on-once:tap={() => console.log("Clicked once!")}>Click Me Once</Button>;
```

---

### **on-passive:**

Mark the event listener as passive (improves scroll performance).

```tsx
import { View } from "@inspatial/kit/structure";

<View on-passive:scroll={(e: any) => console.log("Scrolling...")}>
  Scrollable Content
</View>;
```

**Use case:** Scroll/touch events where you won't call `preventDefault()`.

---

### **on-capture:**

Attach the listener in the capture phase instead of bubble phase.

```tsx
import { View } from "@inspatial/kit/structure";

<View on-capture:tap={() => console.log("Captured before children")}>
  <Button on:tap={() => console.log("Button clicked")}>Click</Button>
</View>;
```

---

### **on-prevent:**

Automatically call `preventDefault()` on the event.

```tsx
import { Form } from "@inspatial/kit/data-flow";

<Form on-prevent:submit={() => handleSubmit()}>
  <Button type="submit">Submit</Button>
</Form>;
```

---

### **on-stop:**

Automatically call `stopPropagation()` on the event.

```tsx
import { View } from "@inspatial/kit/structure";

<View on:tap={() => console.log("Outer")}>
  <Button on-stop:tap={() => console.log("Inner only")}>
    Click (doesn't bubble)
  </Button>
</View>;
```

---

## Creating Custom Triggers

You can create custom trigger props using `createTrigger()`.

### Basic Custom Trigger

```typescript
import { createTrigger } from "@inspatial/kit/trigger";

// Example: Basic hover with enter/leave events
createTrigger("hover", (node, cb?: (e: any) => void) => {
  if (!cb) return;

  const enter = (e: Event) => cb({ type: "hoverenter", event: e });
  const leave = (e: Event) => cb({ type: "hoverleave", event: e });

  node.addEventListener("mouseenter", enter);
  node.addEventListener("mouseleave", leave);

  // Return cleanup function (optional)
  return () => {
    node.removeEventListener("mouseenter", enter);
    node.removeEventListener("mouseleave", leave);
  };
});
```

### Advanced Custom Trigger: Swipe Gesture

```typescript
createTrigger(
  "swipe",
  (
    node,
    cb?: (e: {
      type: string;
      direction: "left" | "right" | "up" | "down";
      distance: number;
    }) => void
  ) => {
    if (!cb) return;

    let startX = 0;
    let startY = 0;

    const down = (e: PointerEvent) => {
      startX = e.clientX;
      startY = e.clientY;
    };

    const up = (e: PointerEvent) => {
      const deltaX = e.clientX - startX;
      const deltaY = e.clientY - startY;

      // Minimum swipe distance: 50px
      if (Math.abs(deltaX) > 50 || Math.abs(deltaY) > 50) {
        const direction =
          Math.abs(deltaX) > Math.abs(deltaY)
            ? deltaX > 0
              ? "right"
              : "left"
            : deltaY > 0
            ? "down"
            : "up";

        cb({
          type: "swipe",
          direction,
          distance: Math.sqrt(deltaX * deltaX + deltaY * deltaY),
        });
      }
    };

    node.addEventListener("pointerdown", down);
    node.addEventListener("pointerup", up);

    return () => {
      node.removeEventListener("pointerdown", down);
      node.removeEventListener("pointerup", up);
    };
  }
);
```

### Usage

```tsx
import { View } from "@inspatial/kit/structure";

<View
  on:swipe={(e: any) => {
    console.log(`Swiped ${e.direction} for ${e.distance}px`);
    if (e.direction === "left") navigateNext();
    if (e.direction === "right") navigatePrevious();
  }}
>
  Swipeable Content
</View>;
```

---

## Creating Extension Triggers

When building extensions, you have two approaches for registering triggers.

### Method 1: Using `createTrigger()` in Lifecycle Setup

Use this when you need full control over registration timing or complex initialization.

```typescript
import { createTrigger } from "@inspatial/kit/trigger";
import { createExtension } from "@inspatial/kit/extension";

export function MyExtension() {
  return createExtension({
    meta: {
      key: "MyExtension",
      name: "My Custom Extension",
      description: "Adds custom interaction triggers",
    },
    lifecycle: {
      setup() {
        // Register triggers during extension initialization
        createTrigger("longHover", (node, cb?: (e: any) => void) => {
          if (!cb) return;
          let timeoutId: number;

          const enter = (e: Event) => {
            timeoutId = setTimeout(
              () => cb({ type: "longHover", event: e }),
              1000
            );
          };

          const leave = () => clearTimeout(timeoutId);

          node.addEventListener("mouseenter", enter);
          node.addEventListener("mouseleave", leave);

          return () => {
            node.removeEventListener("mouseenter", enter);
            node.removeEventListener("mouseleave", leave);
            clearTimeout(timeoutId);
          };
        });
      },
    },
  });
}
```

**When to use:** Complex initialization logic, manual registration timing control.

---

### Method 2: Using `capabilities.triggers`

Use this for declarative trigger definitions without manual `createTrigger()` calls.

```typescript
import { createExtension } from "@inspatial/kit/extension";

export function SwipeExtension() {
  return createExtension({
    meta: {
      key: "SwipeExtension",
      name: "Swipe Gestures",
      description: "Adds swipe gesture support",
    },
    capabilities: {
      triggers: {
        swipe: {
          handler: (node, cb) => {
            if (!cb) return;
            let startX = 0;

            const down = (e: PointerEvent) => (startX = e.clientX);
            const up = (e: PointerEvent) => {
              const dx = e.clientX - startX;
              if (Math.abs(dx) > 50) {
                cb({
                  type: "swipe",
                  direction: dx > 0 ? "right" : "left",
                  distance: Math.abs(dx),
                });
              }
            };

            node.addEventListener("pointerdown", down);
            node.addEventListener("pointerup", up);

            return () => {
              node.removeEventListener("pointerdown", down);
              node.removeEventListener("pointerup", up);
            };
          },
          type: {} as {
            type: "swipe";
            direction: "left" | "right";
            distance: number;
          },
          description: "Detects horizontal swipe gestures",
        },
        pinch: {
          handler: (node, cb) => {
            if (!cb) return;
            // Pinch gesture implementation
          },
          type: {} as {
            type: "pinch";
            scale: number;
            center: { x: number; y: number };
          },
          description: "Detects pinch/zoom gestures",
        },
      },
    },
  });
}
```

**When to use:** Simple, declarative trigger definitions.

---

### Adding TypeScript Support (Optional)

For IntelliSense in JSX `on:` props, create a `types.d.ts` file:

```typescript
// src/types.d.ts
export interface ExtensionTriggerTypes {
  swipe: { type: "swipe"; direction: "left" | "right"; distance: number };
  pinch: { type: "pinch"; scale: number; center: { x: number; y: number } };
}
```

**How it works:**

1. Your extension exposes triggers via `capabilities.triggers` (or `createTrigger()`)
2. If you provide `export interface ExtensionTriggerTypes` in `types.d.ts`, InSpatial Serve discovers it
3. The dev server scans `render.ts` for `createRenderer({ extensions: [...] })`
4. It generates `src/types/extension-trigger-types.generated.d.ts` that extends `InSpatial.ExtensionTriggers`
5. IntelliSense for `on:` props appears only when the extension is registered

**Advanced:** Explicitly point to type files in `deno.json` or `package.json`:

```json
//deno.json
{
  "inspatial": {
    "triggerTypes": ["src/types.d.ts"]
  }
}
```

---

### Method Comparison

| Aspect                    | `createTrigger()` in Lifecycle | `capabilities.triggers`       |
| ------------------------- | ------------------------------ | ----------------------------- |
| **Registration**          | Manual calls                   | Automatic by extension system |
| **TypeScript Completion** | Optional `types.d.ts`          | Optional `types.d.ts`         |
| **Code Style**            | Imperative                     | Declarative                   |
| **Discoverability**       | Not listed in capabilities     | Listed in extension           |
| **Build Integration**     | Not scanned                    | Can be scanned by build tools |
| **Use Case**              | Complex initialization         | Simple trigger definitions    |

---

## How Trigger Props Work

### Processing Pipeline

1. **JSX Compilation**: `<Button on:tap={handler} />` compiles to props object
2. **Renderer Processing**: Renderer calls `setProps(node, props)`
3. **Extension Routing**: `onDirective` callback routes `on:` prefixes to trigger system
4. **Trigger Handler**: `withTriggerProps.onTriggerProp` returns appropriate handler
5. **Event Binding**: Handler binds event to DOM/platform API

### Platform-Specific Implementations

Different platforms can have different trigger implementations:

- **DOM**: Uses `addEventListener`
- **XR**: Can use spatial input APIs
- **Native**: Can use platform-specific gesture recognizers

The trigger bridge system handles these differences transparently. `InTrigger` auto-registers a universal bridge so `on:tap` maps to `click` on web, touch events on mobile, and spatial pointers in XR.

---

## Platform Support Matrix

| Trigger                | Web/DOM | Capacitor       | Native | XR  |
| ---------------------- | ------- | --------------- | ------ | --- |
| **Lifecycle**          |         |                 |        |     |
| `on:beforeMount`       | ✅      | ✅              | ✅     | ✅  |
| `on:mount`             | ✅      | ✅              | ✅     | ✅  |
| `on:frameChange`       | ✅      | ✅              | ✅     | ✅  |
| **Input**              |         |                 |        |     |
| `on:tap`               | ✅      | ✅              | ✅     | ✅  |
| `on:longpress`         | ✅      | ✅              | ✅     | ✅  |
| `on:presshold`         | ✅      | ✅              | ✅     | ✅  |
| `on:rightclick`        | ✅      | ✅              | ⚠️     | ✅  |
| `on:change`            | ✅      | ✅              | ✅     | ⚠️  |
| `on:submit`            | ✅      | ✅              | ✅     | ⚠️  |
| `on:focus`             | ✅      | ✅              | ✅     | ⚠️  |
| **Hover**              |         |                 |        |     |
| `on:hover`             | ✅      | ⚠️              | ⚠️     | ✅  |
| `on:hoverstart`        | ✅      | ⚠️              | ⚠️     | ✅  |
| `on:hoverend`          | ✅      | ⚠️              | ⚠️     | ✅  |
| **Keyboard**           |         |                 |        |     |
| `on:key`               | ✅      | ✅              | ✅     | ✅  |
| `on:key:down`          | ✅      | ✅              | ✅     | ✅  |
| `on:key:up`            | ✅      | ✅              | ✅     | ✅  |
| `on:escape`            | ✅      | ✅              | ✅     | ✅  |
| **Gamepad**            |         |                 |        |     |
| `on:gamepad`           | ✅      | ✅              | ⚠️     | ✅  |
| `on:gamepadconnect`    | ✅      | ✅              | ⚠️     | ✅  |
| `on:gamepaddisconnect` | ✅      | ✅              | ⚠️     | ✅  |
| **Capacitor**          |         |                 |        |     |
| `on:back`              | ❌      | ✅ Android only | ❌     | ❌  |
| `on:resume`            | ❌      | ✅              | ❌     | ❌  |
| `on:pause`             | ❌      | ✅              | ❌     | ❌  |
| `on:urlopen`           | ❌      | ✅              | ❌     | ❌  |
| `on:statechange`       | ❌      | ✅              | ❌     | ❌  |
| `on:restored`          | ❌      | ✅              | ❌     | ❌  |

**Legend:**

- ✅ Full support
- ⚠️ Limited/synthesized support
- ❌ Not available on this platform

---

## Best Practices

### 1. Prefer Universal Triggers

```tsx
// ✅ Good: Universal trigger works everywhere
<Button on:tap={() => handleClick()}>Click Me</Button>

// ❌ Avoid: Platform-specific Renderer event
<Button on:click={() => handleClick()}>Click Me</Button>
```

### 2. Use Lifecycle Triggers Over Side Effects

```tsx
// ✅ Good: Element-bound, one-time setup
<View on:mount={() => initializeChart()}>
  <Chart />
</View>;

// ❌ Wrong: Side effect will re-run on ANY signal change
watch(() => {
  const data = chartData.get();
  initializeChart(); // Runs on every data change!
});
```

### 3. Cleanup in Trigger Handlers

```typescript
createTrigger("myTrigger", (node, cb) => {
  const handler = () => cb({ type: "myTrigger" });
  node.addEventListener("myevent", handler);

  // ✅ Always return cleanup
  return () => {
    node.removeEventListener("myevent", handler);
  };
});
```

### 4. Use Modifiers When Appropriate

```tsx
// ✅ Good: Passive for scroll performance
<View on-passive:scroll={(e: any) => trackScroll(e: any)}>Content</View>

// ✅ Good: Once for one-time setup
<View on-once:mount={() => trackPageView()}>Content</View>

// ✅ Good: Prevent for form submission
<Form on-prevent:submit={() => handleSubmit()}>...</Form>
```

---

## Action Triggers

Action triggers are part of InSpatial State. Use `createAction()` for direct, tuple, or batch state updates.

**See:** [State Documentation](/2.%20interactivity/state🟡.md) for complete action trigger details.

---

## Summary

**Trigger Props provide:**

- ✅ **Universal abstractions** across platforms (web, native, XR)
- ✅ **Declarative event handling** via `on:` props
- ✅ **Extension system** for custom triggers
- ✅ **Performance optimization** via global event delegation
- ✅ **Type safety** via TypeScript integration
- ✅ **Platform bridging** via automatic mapping

**Default approach:**

1. Use **universal triggers** (`on:tap`, `on:key`, etc.) over platform-specific events
2. Use **lifecycle triggers** (`on:mount`, `on:beforeMount`) for widget or component-bound setup
3. Use **custom triggers** (`createTrigger()`) for reusable interaction patterns
4. Use **extension triggers** (`capabilities.triggers`) for distributable trigger libraries

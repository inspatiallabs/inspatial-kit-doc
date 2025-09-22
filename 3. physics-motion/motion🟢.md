## InMotion

#### A universal motion engine that animates anything, everywhere

This page is your guide from static elements to smooth, engaging animations that work everywhere. Think of it like adding choreography to a performance. You're not just moving things around, you're creating experiences that feel natural and delightful.

In development, you want animations that are easy to prototype and adjust. In production, you want them to be performant, accessible, and consistent across all devices and frameworks. InMotion gives you both: simple props for quick iteration, and a powerful engine underneath that handles the complex stuff automatically.

InMotion brings smooth, hardware‑accelerated animations to every environment with a single, readable prop: `data-inmotion`. You describe what you want (fade, zoom, flip, split, count, effects, custom timelines), and InMotion does the heavy lifting: creating performant keyframes, observing visibility, emitting progress events, and cleaning up automatically.

> **Terminology:**
>
> - **Prop-driven**: You control behavior with prop-attributes.
> - **Split**: Breaking content into parts (words/letters/items) and animating them with a stagger.
> - **Loop (mirror/jump)**: Mirror reverses direction at the ends; jump restarts from the beginning.

---

## Getting Started

### 🎯 Prerequisites

Before you start:

- Basic understanding of [JavaScript](https://web.dev/learn/javascript)
- Basic understanding of [TypesScript](https://www.typescriptlang.org/docs/handbook/intro.html)
- Basic understanding of [Renderers](../1.%20graphical-user-interface/rendering/)
- Basic understanding of [Triggers](../2.%20interactivity/trigger🟡.md)

### 🎮 Usage

#### Basic Import

```typescript
// ✅ Recommended: Direct import from kit
import { InMotion } from "@inspatial/kit/motion";

// ❌ Avoid: Package-level import
import { InMotion } from "@inspatial/kit/motion";
```

#### Basic Usage

1. Create a `render.ts` and add the `InMotion` extension to your renderer.

```jsx
//render.ts
import { InMotion } from "@inspatial/kit/motion";

createRenderer({
  extensions: [InMotion()],
});
```

2. Add `data-inmotion` prop to the component you want to animate.

```jsx
//window.tsx
import { View } from "@inspatial/kit/widget"; // can be any component including traditional divs

<View data-inmotion="fade-u duration-600">Hello InMotion</View>;
```

<details>
<summary>📦 Installation for Framework Authors</summary>

If you're building a framework or library that needs to include physics & motion:

```bash
deno install jsr:@in/motion
```

##

```bash
npx jsr add @in/motion
```

##

```bash
yarn dlx jsr add @in/motion
```

##

```bash
pnpm dlx jsr add @in/motion
```

##

```bash
bunx jsr add @in/motion
```

##

```bash
vlt install jsr:@in/motion
```

</details>

---

### Core Prop Syntax

Combine tokens inside `data-inmotion` to describe the effect:

- Animations: `fade`, `zoomin`, `zoomout`, `flip`
- Directions: `u`, `d`, `l`, `r`, combos like `ul`, `dr`
- Timing and control: `duration-{ms}`, `delay-{ms}`, `easing-[css-fn]`, `threshold-{%}`, `once`, `forwards`, `loop`/`loop-{mirror|jump}`
- Effects: `blur`/`blur-{rem}`, `text-shimmer`, `text-fluid`
- Split: `split-word`, `split-letter`, `split-item`, plus `split-delay-{ms}` and stagger mode
- Counters: `count-[{number-like-text}]`
- Custom timeline: `line-[{timeline}]`

> **Note:** When `line-[…]` is present it fully controls keyframes and ignores standard animation tokens.

### Supported Animations

#### Fade

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="fade-u" />
<!-- Upwards fade-in -->
<View data-inmotion="fade-dr" />
<!-- Diagonal down-right -->
```

Optional numeric tuning after the token controls distance (percent):

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="fade-40" />
<!-- 40% movement -->
<View data-inmotion="fade-30-60" />
<!-- X:30% Y:60% -->
```

#### Zoom In / Zoom Out

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="zoomin" />
<!-- Zoom into place -->
<View data-inmotion="zoomout-r" />
<!-- Zoom out from right offset -->
```

Tuning values: one value = intensity, or provide movement plus intensity:

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="zoomin-30" />
<!-- intensity 30% -->
<View data-inmotion="zoomout-40-60" />
<!-- move 40%, intensity 60% -->
<View data-inmotion="zoomin-30-50-80" />
<!-- X:30% Y:50% intensity:80% -->
```

#### Flip (3D)

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="flip-u" />
<!-- Flip over X-axis (up) -->
<View data-inmotion="flip-l" />
<!-- Flip over Y-axis (left) -->
```

Tuning values: angle and perspective (rem):

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="flip-120" />
<!-- 120° angle, default perspective -->
<View data-inmotion="flip-90-60" />
<!-- 90° angle, 60rem perspective -->
```

### Split Animations

Split text or children and animate each part with a stagger:

```jsx
import { Text, YStack, Slot } from "@inspatial/kit/widget"

<Text data-inmotion="split-word fade-u split-delay-100">Split by words</Text>
<Text data-inmotion="split-letter zoomin split-delay-75-center">Letters</Text>
<YStack data-inmotion="split-item flip-r split-delay-50-edges">
  <Slot>Item 1</Slot>
  <Slot>Item 2</Slot>
  <Slot>Item 3</Slot>
</YStack>
```

- Types: `split-word`, `split-letter`, `split-item`
- Delay: `split-delay-{ms}`
- Stagger strategies: `index` (default), `linear`, `center`, `edges`, `random` (e.g. `split-delay-100-center`)

> **Note:** You can nest animation tokens inside split implicitly. Under the hood, split creates a dedicated animation config for each target.

### Counters and Text Effects

#### Count

Animates numbers from 0 to the target while preserving formatting:

```jsx
import { Text } from "@inspatial/kit/widget"

<Text data-inmotion="count-[1,234]">Revenue: 1,234</Text>
<Text data-inmotion="count-[4.9]">Rating: 4.9/5</Text>
```

#### Text Effects

```jsx
import { Text } from "@inspatial/kit/widget"

<Text data-inmotion="text-shimmer split-letter duration-2000 split-delay-100"
  >Shimmer</Text>
<Text data-inmotion="text-fluid split-letter duration-2000 split-delay-50"
  >Fluid</Text>
```

> Must be combined with `split-letter`. These effects auto‑loop with `jump` for continuous motion.

### Modifiers and Control

- `duration-{ms}` / `delay-{ms}`
- `easing-[cubic-bezier(0.4,0,0.2,1)]`, also supports `linear`, `ease`, `ease-in`, `ease-out`, `ease-in-out`, `step-start`, `step-end`
- `threshold-{percent}`: triggers when that portion of the element is visible
- `blur` or `blur-{rem}` (e.g. `blur-2`)
- `once`: animate only once
- `forwards`: keep the final state
- `loop` or `loop-{mirror|jump}`: continuous animation (mirror reverses direction smoothly; jump restarts)

### IDs and Re‑animation

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="fade-u">Auto ID (may re‑animate on re‑render)</View>
<View data-inmotion="fade-u" data-inmotion-id="hero">
  Stable ID (prevents re‑animation)
</View>
```

### InMotion Triggers

InMotion dispatches Trigger events you can listen to directly.

- Trigger events: `inmotion:start`, `inmotion:end`, `inmotion:progress`
- Detail payload:
  - `id?: string`, `ratio: number`, `time: number`, `duration: number`, `direction: "forward" | "reverse"`

InSpatial trigger props (recommended):

```jsx
import { View } from "@inspatial/kit/widget";

<View
  data-inmotion="fade-u"
  on:motionstart={(e) => console.log("start", e.detail)}
  on:motionend={(e) => console.log("end", e.detail)}
  on:motionprogress={(e) => console.log("progress", e.detail)}
/>;
```

### Custom Timeline: `line-[…]`

Fully custom keyframes using a compact grammar. Properties (case‑insensitive):

- Opacity: `o±{0..100}`
- Scale: `s±{v}`, `sx/sy/sz±{v}`
- Translate: `t±{%}`, `tx/ty/tz±{%}`
- Rotate: `r±{deg}`, `rx/ry/rz±{deg}`
- Blur: `b±{rem}`
- Perspective: `p±{rem}`
- Weight (font): `w±{100..900}`

Use `|` to separate keyframes; optional leading percent fixes frame position.

```jsx
import { View } from "@inspatial/kit/widget"

<View
  data-inmotion="line-[o+0 tx+60|30o+80 tx+10|o+100 tx+0] duration-1200"
></View>
<View data-inmotion="line-[w+100|50w+900|w+100] loop linear split-letter"></View>
```

> Order matters. Avoid mixing `s` with `sx/sy/sz` in the same frame. When `line-[…]` is present, standard tokens like `fade`/`zoomin` are ignored.

> **⚠️ Important:** When `line-[{value}]` is defined, standard animation classes are ignored. The custom timeline takes complete control of the animation.

> **⚠️ Important:** When `line-[{value}]` is defined, standard animation classes (fade, zoomin, etc.) are ignored. The custom timeline takes complete control of the animation.

### Animation Tuning

Add numeric values after animation tokens to customize their behavior:

```jsx
import { View } from "@inspatial/kit/widget"

<View data-inmotion="fade-40">Custom fade distance</View>
<View data-inmotion="zoomin-30-50-80">Custom zoom with X, Y, intensity</View>
<View data-inmotion="flip-120-60">Custom flip angle and perspective</View>
```

**How tuning works:**

- **Fade**: Distance values control how far elements move during animation
  - `fade-40` → 40% movement distance
  - `fade-30-60` → X: 30%, Y: 60% movement
- **Zoom**: Scale and movement values control zoom intensity and offset
  - `zoomin-30` → 30% zoom intensity
  - `zoomin-40-60` → 40% movement, 60% intensity
  - `zoomin-30-50-80` → X: 30%, Y: 50%, intensity: 80%
- **Flip**: Angle and perspective values control 3D rotation
  - `flip-120` → 120° rotation angle
  - `flip-90-60` → 90° angle, 60rem perspective depth

> **Note:** Default values are 15% for normal animations, 50% for split animations, 90° angle and 25rem perspective for flips.

```jsx
import { View } from "@inspatial/kit/widget"

<!-- Gentle effects -->
<View data-inmotion="fade">Standard fade distance</View>
<View data-inmotion="fade-10">Minimal fade distance</View>

<!-- Bold effects -->
<View data-inmotion="zoomin">Standard zoom scale</View>
<View data-inmotion="zoomin-75">Strong zoom scale</View>

<!-- Advanced multi-parameter control -->
<View data-inmotion="fade-dr-60-40">Corner fade with custom X/Y offsets</View>
<View data-inmotion="zoomout-20-30-70">
  Precise zoom with movement and scale control
</View>

<!-- 3D rotation examples -->
<View data-inmotion="flip">Standard rotation with default depth</View>
<View data-inmotion="flip-180-50">Half-turn rotation with enhanced depth</View>
<View data-inmotion="flip-udlr-120-80">
  Multi-axis rotation with custom settings
</View>
```

---

## API Reference

### Programmatic API

| Function            | Description                                | Parameters                         | Returns            |
| ------------------- | ------------------------------------------ | ---------------------------------- | ------------------ |
| `InMotion(config?)` | Extension factory for renderer integration | `config?: Partial<InMotionConfig>` | Extension object   |
| `createMotion()`    | Creates motion instance (SSR-safe)         | None                               | `InMotionInstance` |

### InMotionInstance Methods

| Method              | Description                            | Parameters                           | Returns                     |
| ------------------- | -------------------------------------- | ------------------------------------ | --------------------------- |
| `config()`          | Get current configuration              | None                                 | `InMotionConfig`            |
| `config(newConfig)` | Update configuration                   | `newConfig: Partial<InMotionConfig>` | `InMotionInstance`          |
| `destroy()`         | Clean up and stop (extreme cases only) | None                                 | `Promise<void>`             |
| `restart()`         | Restart engine (extreme cases only)    | None                                 | `Promise<InMotionInstance>` |
| `initialized()`     | Check if engine is running             | None                                 | `boolean`                   |
| `version`           | Get version string                     | None                                 | `string`                    |

### Trigger Event Handlers

| Handler                 | Description                | Event Detail                                                                                        |
| ----------------------- | -------------------------- | --------------------------------------------------------------------------------------------------- |
| `motionStartHandler`    | Animation start trigger    | `{ id?: string }`                                                                                   |
| `motionEndHandler`      | Animation end trigger      | `{ id?: string }`                                                                                   |
| `motionProgressHandler` | Animation progress trigger | `{ id?: string, ratio: number, time: number, duration: number, direction: "forward" \| "reverse" }` |

### Configuration Types

| Interface          | Properties                                                                                                     | Description                 |
| ------------------ | -------------------------------------------------------------------------------------------------------------- | --------------------------- |
| `InMotionDefaults` | `animation`, `direction`, `duration`, `delay`, `threshold`, `splitDelay`, `easing`, `blur`, `forwards`, `loop` | Default animation settings  |
| `InMotionConfig`   | `defaults`, `observersDelay`, `once`                                                                           | Global engine configuration |

### Animation Tokens

| Category         | Tokens                                                                                          | Description                |
| ---------------- | ----------------------------------------------------------------------------------------------- | -------------------------- |
| **Animations**   | `fade`, `zoomin`, `zoomout`, `flip`                                                             | Base animation types       |
| **Directions**   | `u`, `d`, `l`, `r`, `ul`, `ur`, `dl`, `dr`                                                      | Movement directions        |
| **Split Types**  | `split-word`, `split-letter`, `split-item`                                                      | Content splitting modes    |
| **Text Effects** | `text-shimmer`, `text-fluid`                                                                    | Special text animations    |
| **Modifiers**    | `duration-{ms}`, `delay-{ms}`, `threshold-{%}`, `once`, `forwards`, `loop`, `blur`              | Animation control          |
| **Easing**       | `linear`, `ease`, `ease-in`, `ease-out`, `ease-in-out`, `step-start`, `step-end`, `easing-[fn]` | Timing functions           |
| **Loop Types**   | `loop-mirror`, `loop-jump`                                                                      | Continuous animation modes |
| **Stagger**      | `split-delay-{ms}-{strategy}`                                                                   | Split animation timing     |

### Custom Timeline Properties

| Property        | Syntax                          | Description          | Units                |
| --------------- | ------------------------------- | -------------------- | -------------------- |
| **Opacity**     | `o±{value}`                     | Element transparency | 0-100 (auto-clamped) |
| **Scale**       | `s±{value}`, `sx/sy/sz±{value}` | Element scaling      | Multiplier           |
| **Translate**   | `t±{value}`, `tx/ty/tz±{value}` | Element movement     | Percentage           |
| **Rotate**      | `r±{value}`, `rx/ry/rz±{value}` | Element rotation     | Degrees              |
| **Blur**        | `b±{value}`                     | Blur effect          | rem                  |
| **Perspective** | `p±{value}`                     | 3D perspective       | rem                  |
| **Weight**      | `w±{value}`                     | Font weight          | 100-900              |

### DOM Events

| Event               | Bubbles | Detail Payload                                                                                      |
| ------------------- | ------- | --------------------------------------------------------------------------------------------------- |
| `inmotion:start`    | Yes     | `{ id?: string }`                                                                                   |
| `inmotion:end`      | Yes     | `{ id?: string }`                                                                                   |
| `inmotion:progress` | Yes     | `{ id?: string, ratio: number, time: number, duration: number, direction: "forward" \| "reverse" }` |

### Default Values

| Setting               | Normal Animations | Split Animations | Flip Animations |
| --------------------- | ----------------- | ---------------- | --------------- |
| **Movement Distance** | 15%               | 50%              | N/A             |
| **Zoom Intensity**    | 15%               | 50%              | N/A             |
| **Rotation Angle**    | N/A               | N/A              | 90°             |
| **Perspective**       | N/A               | N/A              | 25rem           |
| **Duration**          | 1000ms            | 1000ms           | 1000ms          |
| **Threshold**         | 10%               | 10%              | 10%             |
| **Split Delay**       | 30ms              | 30ms             | 30ms            |

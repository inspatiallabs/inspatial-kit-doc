# Widgets & Components

So far you have been composing components using renderer primitives. This section intoduces you to Widgets & Components. Unlike Renderer primimitives you compose components using <Tags>.

A Widget is a high-level primitive. It simple terms a widget contains multiple components. e.g an <InputField> has different variants of components i.e <TextField>, <NumberField> etc...

NOTE: Only high-level primitives (such as Input, Ornament, Presentation, etc.) support `variants`. Each variant/children can have multiple `formats`, which act as sub-variants.

## Rethinking Component Systems

Every UI framework makes you pick a side: speed or flexibility, copy-paste or monolith, web or native or 3D, smart adaptive or plain predictable. Designed vs engineered. It’s always a compromise ease versus control, power versus simplicity. We saw the cracks, and now we’re breaking the wall completely.

### From Patchwork to True Composition

Traditionally, customizing a UI component system meant you were boxed in by whatever styling primitives the library predefined.

Then came the copy-and-paste composition pattern: give developers the raw component code scattered across a myriad of web only libraries and let them edit. It was a breakthrough but still **Patchwork**. Copying primitives isn’t efficient, wrapping components isn’t elegant even with the help of a command-line-interface (CLI) and teaching AI through scattered source files creates poor and the same design output across the board. Is it composition, yes! but a bolted **workaround** composition, **not built in** composition. Fundamentally with the copy-paste approach, you’re still patching over the limitations of a component library not solving the underlying problems.

So we went back to first principles and asked... What if you could install components the traditional way with ready-made styles but also unlock their entire anatomy for full styling and control?

We found our answer by looking at Flutter widgets. Instead of treating composition as a workaround, why not make the styling engine itself the foundation? That’s what we did with [InSpatial Style](./styling%20🟢//introduction.md).

On InSpatial, **every component is already a widget tree**. You don’t copy code; you simply install and use it like any other library. But instead of being locked into a thin layer of styles, you unlock the **entire component anatomy**

- **Beautiful defaults**: Already there, with hundreds of pre-built themes and variables.
- **Unstyling**: Prefer to start from a clean slate? Opt into unstyled with a single flag.
- **Full transparency**: Direct insight into the widget’s anatomy, not a flat-file workaround.
- **Composition**: Not just a principle it’s the ontology.
- **Distribution**: No schema hacks; the widget tree is the schema.
- **AI-ready**: Not just readable, but 60× more efficient because AI works against typed anatomy, not scattered source.

So you get the ease of installation you’re used to, without losing the power to go deep.
**InSpatial Kit makes patchwork composition obsolete.**

### From Dumb to Intelligent

Most component systems are “dumb” by default: they render what you tell them, but don’t adapt, don’t reason, and don’t help you avoid mistakes. InSpatial Widgets are **intelligent by design**. They expose their anatomy, variants, and settings in a way that’s both human- and AI-friendly. This means:

- **Self-documenting APIs**: Every widget describes its own structure, variants, and props.
- **Introspectable anatomy**: You can query, override, or extend any part of a widget - No black boxes (every widget exposes its internals by design).
- **AI-native**: The system is designed so LLMs can reason about, generate, and refactor components with high accuracy.
- **Strict contracts**: Widgets enforce prop types, variant options, and structure at runtime and in docs, so you can’t “fall off the happy path” by accident.

This intelligence isn’t just for AI it makes you faster, too. You get autocomplete, instant docs, and safe composition everywhere.
**InSpatial Kit makes components reason about themselves, so humans and AI build faster together.**

### From Web Only to Universal

Most UI libraries are web-first, with “native” or “3D” as afterthoughts (if at all). InSpatial flips this: **universal by default**. Every widget and component is designed to render on web, mobile, desktop, and spatial platforms no forks, no wrappers, no hacks.

- **Single source of truth**: One widget tree, all platforms.
- **Platform-optimized**: Styles, triggers, actions and layouts adapt automatically to the target environment.
- **No platform lock-in**: Move your app from web to mobile to XR with zero rewrite.
- **Consistent theming**: Colors, spacing, and variables work everywhere, with platform-specific overrides only when you want them.

InSpatial Kit Widgets and Components are not just “cross-platform” - they are **platform-native everywhere**. You build once, and your components feel right at home on any device, in any context.
**InSpatial Kit makes platform silos irrelevant. One widget tree, all realities.**

### Widgets & Components also come bundled with other key features

- **Seamless transitions** between DOM, GPU & Native
- **Natural Language Input & Context Awareness**
- **Universal UI Renderer**: XR, Progressive Web, Native (iOS, Android, HorizonOS, VisionOS)
- **Composable & Recursive Component System**
- **Variant System** for adaptive designs
- **Themeability & Theme Controls**
- **Built-in Skeletal System** (animations, avatars, placeholders)
- **Built-in Haptic Feedback & Sound System**
- **Full Figma Kit & Integration**
- **Tailwind Support**
- **Multipurpose by design** (Game UI, OS, Web, Mobile, XR)
- **Pre-styled headless components** (fully customizable)
- **Fully Typed** for precision and AI-readiness
- **Typography, Icon, InMoji & Illustration System**
- **Advanced Layout System** with **No CLS (Cumulative Layout Shifts)**
- **Smooth Transitions & Motion Effects**
- **Accessibility-first**: ARIA, i18n, auto-translate
- **Built-in State and Storage** (cookie, local/session storage)
- **Realtime Updates**
- **Command Shortcuts Support** (trigger search, cancel dialogs, etc.)
- **Responsive Design System**
- **Cursor Trails** for spatial delight
  etc...

---

## How to build a Scalable Widget or Component

- Start by understanding the widgets anatomy the parts i.e the different elements that its made up of e.g a Topbar might need Links and Buttons.
- Take each of those components probvided by InSpatial Kit and create a composable primitive from it separately
- define your styles in `style.ts`
- Finally use your `type.ts` to create define structure

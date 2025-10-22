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

## Widgets Overview (🟡 Preview)

Below is a list of all InSpatial Kit Widgets and their variants represented as props

```jsx
{/*##############################################(BLOCK)##############################################*/}
<Block variant="Chip" />
<Block variant="ColorCasts" /> {/* also known as ColorSwatch or ColorPresets */}
<Block variant="Dropzone" />
<Block variant="List" />
<Block variant="Tag" />
<Block variant="Tile" />

{/*##############################################(CONTROL FLOW)##############################################*/}

<ControlFlow variant="Async" />
<ControlFlow variant="Choose" />
<ControlFlow variant="Controller" />
<ControlFlow variant="Dynamic" />
<ControlFlow variant="Show" />


{/*##############################################(DATA FLOW)##############################################*/}

<DataFlow variant="List" />
<DataFlow variant="Table" />
<DataFlow variant="Tour" />
<DataFlow variant="Tree" />

{/*##############################################(ICON)##############################################*/}
<Icon variant="InSpatialIcon" >

{/*##############################################(ILLUSTRATION)##############################################*/}
<Illustration variant="InSpatialIllustration" >

{/*##############################################(INDEX BAR)##############################################*/}

<IndexBar variant="AlphabetList" />
<IndexBar variant="NumberList" />
<IndexBar variant="RulerGuide" />
<IndexBar variant="VersionController" />

{/*##############################################(InMoji)##############################################*/}
<Inmoji variant="HappyInmoji" >

{/*##############################################(INMOTION)##############################################*/}

<Inmotion variant="ViewTransition">


{/*##############################################(INPUT)##############################################*/}

<Input variant="Alignbox" />
<Input variant="Calender" />
<Input variant="Checkbox" />
<Input variant="ChoiceGroup" />
<Input variant="ChoiceInput" /> {/* type */}
<Input variant="Counter" />
<Input variant="CurrencyField" />
<Input variant="DateField" />
<Input variant="EmailField" />
<Input variant="FileUploader" />
<Input variant="Joystick" />
<Input variant="LocationField" />
<Input variant="MultiSelect" />
<Input variant="NumberField" />
<Input variant="PasswordField" />
<Input variant="PhoneField" />
<Input variant="PinField" />
<Input variant="Radio" />
<Input variant="RadioGroup" />
<Input variant="SearchField" />
<Input variant="SelectInput" /> {/* type */}
<Input variant="Slider" />
<Input variant="Switch" />
<Input variant="Tag" />
<Input variant="TextField" />
<Input variant="TextInput" /> {/* type */}
<Input variant="Time" />
<Input variant="TimeField" />
<Input variant="UrlField" />

{/*##############################################(LIGHT)##############################################*/}

<Light variant="Ambient" />
<Light variant="Directional" />
<Light variant="Hemisphere" />
<Light variant="RectArea" />
<Light variant="Spot" />

{/*##############################################(MEDIA)##############################################*/}
<Media variant="Audio" format="MP3" src="/audio-url" >
<Media variant="Image" format="PNG" src="/image-url" >
<Media variant="Model" format="GLTF" src="/3d-url" >
<Media variant="Video" format="MP4" src="/video-url" >

{/*##############################################(NAVIGATION (MENU))##############################################*/}

<Navigation variant="Bottombar" />
<Navigation variant="Carousel" />
<Navigation variant="ContextMenu" />
<Navigation variant="Link" />
<Navigation variant="Pagination" />
<Navigation variant="PathControl" /> {/* also known as breadcrumbs */}
<Navigation variant="Sidebar" />
<Navigation variant="Stepper" />
<Navigation variant="Topbar" />
<Navigation variant="Wheel" />

{/*##############################################(ORNAMENT)##############################################*/}

<Ornament variant="ArcReactor">
<Ornament variant="Assetbox">
<Ornament variant="Avatar">
<Ornament variant="Badge">
<Ornament variant="Button">
<Ornament variant="Callout">
<Ornament variant="Divider">
<Ornament variant="Folder">
<Ornament variant="KitBorder">
<Ornament variant="Notch">
<Ornament variant="Tab">

{/*##############################################(OS)##############################################*/}

<OS variant="Emulator" />
<OS variant="Keyboard" />
<OS variant="Simulator" />

{/*##############################################(PARTICLE EMITTER)##############################################*/}

<ParticleEmitter variant="Clouds" />
<ParticleEmitter variant="Flame" />
<ParticleEmitter variant="Lines" />
<ParticleEmitter variant="Points" />
<ParticleEmitter variant="Rain" />
<ParticleEmitter variant="Snow" />
<ParticleEmitter variant="Sparks" />
<ParticleEmitter variant="Stars" />

{/*##############################################(PRESENTATION) ##############################################*/}

<Presentation variant="Detail" >
<Presentation variant="Dock" >
<Presentation variant="Drawer" >
<Presentation variant="Dropdown" >
<Presentation variant="Loader">
<Presentation variant="Modal" >
<Presentation variant="Notice">
<Presentation variant="Popover">
<Presentation variant="Tooltip">
<Presentation variant="Window">

{/*##############################################(REALTIME)##############################################*/}

<Realtime variant="Cursor" />
<Realtime variant="GroupSelection" />
<Realtime variant="LiveSelection" />
<Realtime variant="PresenceIndicator" />

{/*##############################################(SHAPE)##############################################*/}

<Shape variant="Circle">
<Shape variant="Cone">
<Shape variant="Cube">
<Shape variant="Cylinder">
<Shape variant="Donut">
<Shape variant="Gem">
<Shape variant="Pill">
<Shape variant="Rectangle">
<Shape variant="Ring">
<Shape variant="Sphere">
<Shape variant="Triangle">
<Shape variant="Tube">

{/*##############################################(STRUCTURE)##############################################*/}

<Structure variant="Canvas" />
<Structure variant="Marquee" />
<Structure variant="Resizable" />
<Structure variant="Skeleton" />
<Structure variant="Slot" />
<Structure variant="Stack" />
<Structure variant="View" />

{/*##############################################(TEMPLATE)##############################################*/}

<Template variant="BannerTemplate" />
<Template variant="CalenderTemplate" />
<Template variant="CardTemplate" />
<Template variant="EmptyStateTemplate" />
<Template variant="ErrorTemplate" />
<Template variant="FooterTemplate" />
<Template variant="GalleryTemplate" />
<Template variant="HeaderTemplate" />
<Template variant="HeroTemplate" />
<Template variant="LoadingTemplate" />
<Template variant="PaywallTemplate" />
<Template variant="ThanksTemplate" />
<Template variant="UnauthorizedTemplate" />

{/*##############################################(TYPOGRAPHY)##############################################*/}

<Typography variant="Blockquote" />
<Typography variant="Code" />
<Typography variant="Text" />

{/*##############################################(VISUALIZER)##############################################*/}

<Visualizer variant="Bubble" />
<Visualizer variant="Column" />
<Visualizer variant="DataBar" />
<Visualizer variant="Gauge" />
<Visualizer variant="Globe" />
<Visualizer variant="Informer" />
<Visualizer variant="Line" />
<Visualizer variant="Meter" />
<Visualizer variant="ProgressBar" />
<Visualizer variant="ProgressCircle" />
<Visualizer variant="Radar" />
<Visualizer variant="Sankey" />
<Visualizer variant="Soundwave" />
<Visualizer variant="Tracker" />

```

# Styling

So you've built your InSpatial widgets and components and now it's time to make them look amazing. This page is your guide from basic styling to sophisticated design systems that work seamlessly across web, mobile, and spatial environments. Think of it like learning to paint: you start with simple brushstrokes, then master advanced techniques that let you create beautiful, consistent experiences everywhere.

In development, you want styling that's fast to write and easy to iterate on. In production, you want styles that are performant, maintainable, and work reliably across all platforms. InSpatial Style gives you both: an intuitive API for rapid prototyping and a powerful engine underneath that handles cross-platform optimization, transpilation, theme switching, fonts, and design system consistency automatically.

InSpatial Style Sheet (ISS) brings together the best of CSS-in-JS and utility-first styling with a twist: everything is designed for universal deployment. You write styles once using familiar patterns, and the system automatically adapts them for web browsers, mobile apps, XR and games without you having to think about platform differences.

> **Terminology:**
>
> - **ISS (InSpatial Style Sheet)**: The universal styling system that compiles your styles for any platform.
> - **Variant Authority**: The precedence system that determines which styles win when multiple approaches are combined.
> - **Cross-Style Composition**: Advanced pattern where one component's styles react to another component's settings.
> - **Reactive Style**: Styles that automatically update when your application state changes.
> - **Shaders**: Programs/Styles that run on the GPU to control advanced visual effects, such as lighting, gradients etc...

## Getting Started

### 🎯 Prerequisites

Before you start:

- Basic understanding of [CSS](https://web.dev/learn/css)
- Basic understanding of [Widgets/Components](../1.%20graphical-user-interface/widgets-components🟡.md/)
- Optional understanding of [Interactivity](../2.%20interactivity/introduction🟡.md)

### 🎮 Usage

#### 1. Creating (Inline) Styles

```jsx
import { View } from "@inspatial/kit/widget";

// with style prop
<View
  style={{
    web: { backgroundColor: "green" },
    ios: { backgroundColor: "yellow" }, // ⚠️ Experimental
    visionOS: { backgroundColor: "orange" }, // ⚠️ Experimental
    android: { backgroundColor: "red" }, // ⚠️ Experimental
    androidXR: { backgroundColor: "maroon" }, // ⚠️ Experimental
    horizonOS: { backgroundColor: "pink" }, // ⚠️ Experimental
  }}
/>;
```

```jsx
// with class/ClassName
<View className="bg-green" />
```

#### 2. Creating (External) Styles

```typescript
// ✅ Recommended: Direct import from kit
import { createStyle } from "@inspatial/kit/style";

// ❌ Avoid: Package-level import
import { createStyle } from "@inspatial/kit";

//style.ts
export const MyStyle = createStyle({
  base: [
    "my_cutom_style_selector_that_can_be_tailwind_utils", // optional custom style-selectors
    {
      web: { backgroundColor: "green" },
      ios: { backgroundColor: "yellow" }, // ⚠️ Experimental
      visionOS: { backgroundColor: "orange" }, // ⚠️ Experimental
      android: { backgroundColor: "red" }, // ⚠️ Experimental
      androidXR: { backgroundColor: "maroon" }, // ⚠️ Experimental
      horizonOS: { backgroundColor: "pink" }, // ⚠️ Experimental
    },
  ],
});
```

```jsx
//window.tsx
import { MyStyle } from "./style.ts"

function MyComponent(){
  return (
    <View className={MyStyle.getStyle({ className, ...rest })}>
  )
}
```

<details>
<summary>📦 Installation for Framework Authors</summary>

If you're building a framework or library that needs to include physics & motion:

```bash
deno install jsr:@in/style
```

##

```bash
npx jsr add @in/style
```

##

```bash
yarn dlx jsr add @in/style
```

##

```bash
pnpm dlx jsr add @in/style
```

##

```bash
bunx jsr add @in/style
```

##

```bash
vlt install jsr:@in/style
```

</details>

---

## What's Next?

Now that you know the basics, you might want to explore:

- [ISS](./iss.md) - Advanced style creation for production
- [Fonts & Typography](./font-typography.md) - comprehensive typography system

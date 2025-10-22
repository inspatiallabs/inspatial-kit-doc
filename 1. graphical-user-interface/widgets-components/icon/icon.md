# Icon

#### Display any icon from the InSpatial icon library with automatic type safety

Icons are concise visual symbols used to represent actions, categories, or concepts for quick recognition.

The `Icon` widget is like having a universal remote for your entire icon collection. Instead of importing dozens of individual icon files, you simply tell it which icon you want by name, and it instantly displays the right one. Think of it as a smart icon dispatcher that knows every icon in your app and can render any of them on demand.

As with every other InSpatial Widget, The Icon widget uses a variant based system where you pass the icon name as a prop. Behind the scenes, it automatically maps that name to the correct icon component and renders it with all the styling options you'd expect size, color, format, and more. It's designed to be flexible enough to handle any icon naming style you prefer: whether you write "RoundIcon", "ball-icon", "ball", or "RoundIcon" or "RoundIcon", it normalizes everything and finds the right match.

> **Note:** The Icon component dynamically resolves icon variants at runtime using a [Choose]("./control-flow/choose.md") control flow. This means you get the convenience of a single import with none of the bundle bloat tree shaking ensures only the icons you actually use end up in your final build.

> **Terminology:**
>
> - **Variant**: The name identifier for a specific icon (e.g., "CheckIcon", "ClockIcon").
> - **Dynamic Resolution**: The Icon component determines which icon to render at runtime based on the variant prop.
> - **Tree Shaking**: AKA (Dead Code Elimination). Removes unused icon code from your final bundle.
> - **Normalization**: Converting different naming styles (kebab-case, camelCase, etc.) into a standard format.

<details>
  <summary><strong>Genius</strong></summary>

The Icon component uses a two stage resolution system:

1. **Name Normalization**: Converts any naming convention (kebab-case, snake_case, camelCase, PascalCase) into PascalCase with "Icon" suffix. This happens via `toPascalCase()` which strips hyphens/underscores/spaces, capitalizes word boundaries, and ensures the "Icon" suffix.

2. **Dynamic Mapping**: Builds a `Choose` component with cases for every exported icon. Each case compares the normalized variant name (case-insensitive) against registered icon component names. The first match renders; fallback is `BrandIcon` (InSpatial logo).

**Type Safety**: The `variant` prop is typed as `IconVariant`, a union of all available icon names auto-generated from the icon directory. This is regenerated before every publish minor and major version of `@inspatial/kit` via `deno task update:icon` command.

**Performance**: Despite being dynamic, tree shaking works because the Icon component imports from a static barrel (`@in/widget/icon/index.ts`). Unused icon components are eliminated at build time since each icon is a separate module.

**Exclusion Logic**: The Icon component explicitly excludes itself from the variant registry to prevent infinite recursion. See `isIconExport()` filter in `component.tsx`.

</details>

### Guideline

#### 📏 Size Matters (Standard Icon Sizes)

| Size Label | Dimensions (px) | Use Case             |
| ---------- | --------------- | -------------------- |
| **xl**     | 48 × 48         | Hero icons, splash   |
| **lg**     | 40 × 40         | Prominent controls   |
| **md**     | 32 × 32         | **Default / base**   |
| **sm**     | 24 × 24         | Compact UI, toolbars |
| **xs**     | 16 × 16         | Badges, tight spaces |

> **Pro tip:** Every icon should feel perfectly balanced in its frame, _regardless of size_!

---

#### 🧩 Layout & Spacing

- **Fit to Frame:** Icons should always _fill the available space_ (Auto-Layout FTW).
- **Default Frame:** 32px with **4px padding**-icons are never cramped, always centered.
- **Center Stage:** Center icons in both axes, every time.
- **Size & Variant:** Two key props -`size` and `variant` -keep usage easy and snappy.

---

#### 🏷 Naming Like a Pro

- **PascalCase** only, always ending with `Icon`.  
  _Example:_ `SunIcon`, `CloudUploadIcon`
- Name for _meaning_, not how it looks.
- Add a helpful **description** and, if relevant, a link to more info or docs!

---

#### 🪩 Variants & Style

- Each icon supports multiple **styles**:
  - **Regular** (classic), **Glass**, **Duotone** -each one a vibe!
- Craft icons at **similar detail/density** so they _feel_ like one big, happy family.
- Use **size-space variables** for all sizes, paddings, and spacing-they scale smartly with your design system!

---

#### ⬛️⏹ Borders & Fills

- **Stroke/border:** Minimum **1px** thickness (but 2px is 🔥 for crispy clarity!)
- Strokes can be _centered_, _inside_, or _outside_-adapt to your context.
- **Auto-Layout** ensures strokes always look great.

---

_Design your icons with pride and precision-make every one as iconic as your brand!_

### Usage

Here's how you use the icon widget in your InSpatial app:

```jsx
import { Icon } from "@inspatial/kit/icon";

function Component() {
  return <Icon variant="RoundIcon" format="regular" size="md" />;
}
```

Prefer not to use the Icon widget? You can directly import and use any specific icon component as needed.

```jsx
import { RoundIcon } from "@inspatial/kit/icon";

function Component() {
  return <RoundIcon format="regular" size="md" />;
}
```

### What You Get Back

The Icon component renders an SVG element with the icon graphic you requested. It automatically handles all the styling, sizing, and accessibility attributes. If the variant name doesn't match any known icon, it falls back to displaying the InSpatial brand icon (`BrandIcon`) as a safe default - you may get a type complain before that happens.

### Available Icons

InSpatial Kit includes 10,000+ professionally designed icons covering all use cases its is both a hybrid of handcrafted and [phosphor]("https://phosphoricons.com/") icons.

> **Note:** The full list of available icons is auto-generated and typed. Your editor will provide autocomplete suggestions when you type the `variant` prop, making it easy to discover all available options.

> TypeScript may complain if you use other string casing even with Normalization because the Icon variant types are defined in PascalCase by default, even though the component accepts icons in any naming style. This is intentional so your icons are consistent.

> - **Let your IDE or UDE help you**: Type `variant=""` and wait for autocomplete suggestions this prevents typos and helps you discover available icons.

### Utility: getIcon

When an icon value can be a string variant, a props object, a custom JSX node, or null/undefined, use `getIcon` to normalize and render it with a fallback.

```ts
getIcon(icon: any, fallbackVariant: IconVariant): JSX.Element
```

```jsx
import { getIcon } from "@inspatial/kit/icon";

// 1) Unknown source (string | { variant, ...props } | JSX | null)
function Row({ item }) {
  return getIcon(item.icon, "BrandIcon");
}

// 2) String variant
getIcon("ClockIcon", "BrandIcon");

// 3) Props object (applies size/format/etc.)
getIcon({ variant: "ClockIcon", size: "sm", format: "outline" }, "BrandIcon");

// 4) Custom JSX node
getIcon(<MyCustomIcon />, "BrandIcon");
```

- If `icon` is null/undefined, the `fallbackVariant` is rendered.
- If `icon` is a string, it's treated as a `variant`.
- If `icon` is an object with a `variant` string, it renders `<Icon {...icon} />`.
- Otherwise `icon` is returned as-is (assumed JSX node).

Note: Variants are typed as PascalCase via `IconVariant` (e.g., `ClockIcon`). The runtime also accepts other casings but prefer typed variants for best DX.

### API-Props

| Prop        | Type                                                                                                                                                        | Default  | Description                                                  |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------ |
| `variant`   | `IconVariant`                                                                                                                                               | -        | The name of the icon to display (e.g., "CheckIcon", "clock") |
| `size`      | `"8xs" \| "7xs" \| "6xs" \| "5xs" \| "4xs" \| "3xs" \| "2xs" \| "xs" \| "sm" \| "md" \| "lg" \| "xl" \| "2xl" \| "3xl" \| "4xl" \| "5xl" \| "6xl" \| "7xl"` | `"md"`   | The size of the icon                                         |
| `format`    | `"fill" \| "outline"`                                                                                                                                       | `"fill"` | Whether the icon should be filled or outlined                |
| `disabled`  | `boolean`                                                                                                                                                   | `false`  | Whether the icon appears disabled with reduced opacity       |
| `className` | `string`                                                                                                                                                    | -        | Additional CSS classes for custom styling                    |

### Related Components

- [Button](/1.%20graphical-user-interface/widgets-components/button/button.md) - Icons pair naturally with buttons to communicate actions
- [Tab](/1.%20graphical-user-interface/widgets-components/tab/tab.md) - Use icons in tab labels for visual navigation
- [Checkbox](/1.%20graphical-user-interface/widgets-components/checkbox/checkbox.md) - Supports icon prop that accepts Icon variants
- [Radio](/1.%20graphical-user-interface/widgets-components/radio/radio.md) - Supports icon prop that accepts Icon variants

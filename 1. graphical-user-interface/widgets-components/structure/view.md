# View

`View` is a variant of the Structure Widget. Its generally the entry-point to building a component. It’s the surface where everything else in your component gets painted or placed.

- That’s why you only need **one `<View>`** at the root of your component, just like you only need **one canvas** for a single painting.
- Having multiple `<View>`s at the same level is like starting several canvases for one artwork: confusing, fragmented, and difficult to manage.

**Why It Fills the Whole Screen**

By default, `<View>` expands to the **maximum width and height of the viewport**.

- This ensures your “canvas” is **complete**, no cutoff edges.
- But since it covers the whole space, it doesn’t scroll automatically. Why? Because if your canvas is already infinite, you don’t need scroll until your content overflows.

So scroll is **opt-in**, not forced. That’s where the `View`’s scrollbar settings come in.

**Scrolling In Views**

The `<View>` component is not just a passive box. It’s your **stage**.

- Sometimes the stage needs to allow the audience (users) to move around → **scrollbars**.
- Sometimes the stage needs to shift moods, pace, or transitions → **motion and animation**.

Because `View` is the stage, it controls these high-level experiences. You don’t sprinkle scroll or motion randomly across elements; you anchor them at the structural level.

## Usage

**Import**

```tsx
import { View } from "@inspatial/kit/structure";
```

**Example**

```tsx
<View>
  <Stack variant="yStack" className="space-y-2 p-2 bg-(--brand) w-full">
    {Array.from({ length: 30 }).map((_, i) => (
      <YStack key={i} className="p-3 rounded bg-(--surface) text-(--primary)">
        Item {i + 1}
      </YStack>
    ))}
  </Stack>
</View>
```

**View `ScrollBar`Themes & Properties**

```tsx
// Thin (default)
<View scrollbar scrollbarTheme="thin">...</View>

// Minimal (thumb appears on hover)
<View scrollbar scrollbarTheme="minimal">...</View>

// Rounded
<View scrollbar scrollbarTheme="rounded">...</View>

// Pill
<View scrollbar scrollbarTheme="pill">...</View>

// Gradient
<View scrollbar scrollbarTheme="gradient">...</View>
```

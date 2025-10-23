# Composition

While composition is a broad concept utilized throughout InSpatial Kit including in modules like InSpatial Style this section focuses specifically on composing within InSpatial Widgets. In both contexts, composition refers to the process of constructing widgets or components by thoughtfully arranging their constituent parts, or "anatomy." Effective composition allows for flexible, reusable, and maintainable graphical user interface (GUI).

## How to build a Scalable Widget or Component

- Begin by understanding the widget’s anatomy the individual elements it comprises. For example, a Topbar might consist of Links and Buttons.
- Use the components provided by InSpatial Kit to create reusable primitives for each anatomical part.
- Define your custom styles in a `style.ts` file.
- Finally, specify the widget’s structure in your `type.ts` file.

## To Compose, or Not To Compose?

Not every component is equally composable. Consider [Input Widgets](../widgets-components/input/widget/widget.md) with variants such as [Switch](../widgets-components/input/slider/slider.md) and [Slider](../widgets-components/input/slider/slider.md). These components have tightly coupled internal parts for example, a range and a handle in a slider where removing a core anatomical element renders the component non-functional. In such cases, composition is limited: the structure is purposefully rigid because every part is essential to the widget’s function.

On the other hand, composition shines when components are designed with optional or swappable parts. If you can safely remove, replace, or rearrange portions of a widget tree such as a header, label, or icon without breaking intended functionality, then composition enables more adaptable and extensible components.

**Best Practice:**

- Apply composition when building widgets or components that benefit from flexibility and customizability.
- For atomic, inseparable widget structures, prefer encapsulation over composition to prevent accidental deformation of the widget’s anatomy and guarantee reliability.
- Assess each component’s design: if there are sub-parts that can be independently controlled, customized, or omitted, leverage the power of composition to create more robust UI solutions.

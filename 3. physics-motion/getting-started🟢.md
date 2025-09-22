# Physics & Motion

#### Your complete guide to bringing InSpatial apps to life with smooth animations and realistic physics

So you've built your InSpatial app and now it's time to make it feel alive. This section covers everything you need to create engaging, performant physics and animations that work everywhere. Think of it like learning to direct a movie—you're not just placing elements on screen, you're orchestrating how they move, react, and feel.

In development, you want motion that's easy to prototype and tweak. In production, you want it to be buttery smooth, accessible, and consistent across all devices. InSpatial Motion A.K.A InMotion gives you both: simple declarative syntax for quick iteration, and powerful physics engines for complex interactions.

> **Terminology:**
>
> - **Motion**: Smooth, declarative animations using `data-inmotion` prop-attribute
> - **Physics**: Realistic simulations with gravity, collisions, and forces
> - **Declarative**: You describe what you want, not how to achieve it

---

## Core Concepts

### 🎬 [Motion](./motion.md)

**Prop-driven animations that work everywhere**

The easiest way to add smooth animations to any element. Just add `data-inmotion="fade-u duration-500"` and watch it come alive. Perfect for UI transitions, reveals, and text effects.

```jsx
<View data-inmotion="fade-u duration-600">Hello World</View>
```

**Key features:**

- Split animations (words, letters, items)
- Custom timeline control with `line-[...]`
- Hardware-accelerated performance
- Cross-framework compatibility

### ⚡ [Physics](./physics.md)

**Realistic simulations for interactive experiences**

When you need gravity, collisions, springs, and forces. Perfect for games, interactive demos, and complex UI behaviors that feel natural and responsive.

**Key features:**

- Rigid body dynamics
- Collision detection
- Spring and damping systems
- Performance-optimized WebAssembly core

---

## When to Use What

| Use Case                | Recommended Approach    | Why                                                   |
| ----------------------- | ----------------------- | ----------------------------------------------------- |
| **UI Transitions**      | [InMotion](./motion.md) | Declarative, performant, easy to maintain             |
| **Page Animations**     | [InMotion](./motion.md) | Built-in visibility detection and cleanup             |
| **Text Effects**        | [InMotion](./motion.md) | Advanced split animations and text-specific features  |
| **Interactive Games**   | [Physics](./physics.md) | Realistic collisions and forces                       |
| **Complex Simulations** | [Physics](./physics.md) | Full physics engine with constraints                  |
| **Spring Animations**   | Both                    | InMotion for simple springs, Physics for complex ones |

---

## Best Practices

- **Start simple**: Use InMotion for most animations, add Physics when you need realistic behavior
- **Performance first**: Both systems are optimized, but test on lower-end devices
- **Accessibility**: Both systems respect `prefers-reduced-motion` automatically
- **Progressive enhancement**: Animations should enhance, not break, your core experience

---

## Next Steps

1. **Learn the basics**: Start with [InMotion](./motion.md) for your first animations
2. **Add interactivity**: Explore [Physics](./physics.md) when you need realistic behavior
3. **Master triggers**: Use [Trigger Events](../2.%20interactivity/trigger🟡.md) to coordinate complex sequences
4. **Optimize performance**: Follow our [Performance Guide](../7.%20building-dev-production/build🟡.md) for production apps

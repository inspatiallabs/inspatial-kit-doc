# @in vs @inspatial UDE Modules

## @in UDE Modules

Think of `@in` UDE modules as individual LEGO bricks each one is a perfectly crafted, single-purpose piece that works independently right out of the box. No assembly required, no dependencies needed. But here's where the magic happens: when you connect these bricks together, they form a complete full-stack, universal, and cross-platform framework.
Each `@in` module is like a specialized LEGO piece whether it's a wheel, a window, or a foundation block designed for one specific job and built to snap seamlessly with others. Every brick is useful on its own and comes with clear instructions, so you understand exactly what it does and how to use it.

## @inspatial/kit

This is your complete LEGO castle kit. While `@in` modules are the individual bricks, `@inspatial/kit` is the finished masterpiece a fully assembled, ready-to-use structure that combines all those building blocks into one cohesive toolkit.
We chose this composable LEGO architecture because we know builders have different styles. Some developers love the creative process of selecting individual bricks and constructing something unique from the ground up. Others prefer to start with a pre-designed castle and get straight to adding their own creative touches.
Whether you're a brick-by-brick builder or a complete-kit creator, we've got you covered.

---

# Framework vs Module vs UDE

**What Makes Something a Framework:**

- Imposes structure on your code
- Ships with its own renderer
- Creates conflicts with other rendering environments (try using Vue & React together you'll need significant low-level API changes to make it work)

**What Makes Something a Module:**

- Gets out of your way
- Integrates seamlessly with existing codebases
- Works regardless of your rendering choice or runtime environment
- Traditionally called "libraries" but we call them modules because they're more than that

**In the InSpatial Ecosystem:**

- **@inspatial/kit** = InSpatial Kit is a UDE client Framework. It provides structure, opinions, and a complete development environment.
- **@in modules** = Pure modules. They adapt to your world instead of forcing you into theirs.

Think of it this way: frameworks are like moving into a fully furnished house with rules about where everything goes. Modules are like buying individual pieces of furniture that work in any house, with any style, following your rules.

**UDE (Universal Development Environment)** bridges this gap giving you framework power when you want it, module flexibility when you need it.

---

# IDE vs CDE vs UDE

**IDE (Integrated Development Environment)**

- Code-first interface (VS Code, IntelliJ, Xcode)
- Flexible and general-purpose for any type of development
- Requires technical knowledge to use effectively
- Primarily for developers and engineers

**CDE (Cloud Development Environment)**

- Cloud-hosted IDEs (GitHub Codespaces, Replit, Stackblitz)
- Still code-first, still requires developer expertise
- Limited by network connectivity and provider constraints

**UDE (Universal Development Environment)**

- **Visual-first interface** - In a UDE anything you can do with code you can do with a visual interface no more no less.
- **Opinionated and focused** - specifically designed for anything that renders a graphical user interface (apps, websites, games, xr)
- **Accessible to non-coders** - project managers and designers can use it without coding knowledge
- **Code mode available** - developers can switch to traditional coding interface when needed
- **Intelligence built-in** - comes bundled with LLMs for AI assistance
- **Truly universal deployment** - cloud by default, can run locally as installed binary
- **Zero-config rendering** - handles multi-platform deployment automatically

**The Key Difference:**
All Development environemnt have a `Text/Code Editor` built in but there are major diferences between them.

- **IDE/CDE**: "Here's a blank canvas and some tools - figure out what to build"
- **UDE**: "Here's a visual interface to build apps, websites, xr and games - code if you want to, but you don't have to"

UDEs can be considered as opinionated higher-level abstraction of IDEs, however they are visual first. This makes development accessible to non-coders in a way an IDE/CDE cannot.

# InSpatial Kit vs InSpatial Cloud vs InSpatial

- **InSpatial Kit** is a Client/Frontend **Framework.**
- **InSpatial Cloud** is a Server/Backend **Framework.**
- **InSpatial** is a **UDE.**

# Why Does InSpatial Kit Syntax & Markup Resemble React/React Native?

InSpatial Kit was deliberately designed to closely mirror React and React Native, both in syntax and markup. In fact, the earliest internal versions of InSpatial Kit were directly built atop React & React Native, functioning primarily as a component and widget system. Other modules in our initial stack were either patched in or partially implemented using third-party React libraries, resulting in a tightly-coupled but somewhat fragmented setup. Over time, as we encountered performance constraints, bottlenecks, standard and platform fragmentation and the increasing complexity of the React ecosystem, we decided to create our own infrastructure from the ground up.

Years of experience building systems on top of React led us to intentionally maintain compatibility: the majority of InSpatial Kit’s default runtime markup and syntax were kept almost identical to React. This was done to ensure smooth portability and migration for both our internal projects and for anyone else coming from the React ecosystem.

While you can use other runtime markups, such as `jsxsfc` by changing the InSpatial Kit runtime flag, or even define your own, the default remains the familiar `jsx` your already know and love:

```jsx
// jsx
export function LooksLikeReactNative() {
  return <Text>Hello World</Text>;
}
```
And, of course, you can write switch to the new JSX-SFC with no return statement:

```jsx
// jsxsfc
export function SyntaxWithNoReturn() {
  <Text>Hello World</Text>;
}
```

This deliberate design ensures that both migration and onboarding are straightforward for developers already familiar with React or React Native with zero learning curve. 

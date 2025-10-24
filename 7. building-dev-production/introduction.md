# Building for Dev & Production

#### A friendly guide from “it works on my machine” to “it runs everywhere”

So, you’ve created your amazing InSpatial website, app, game or XR experience. Now it’s time to get it in front of real people. This page is your roadmap from everyday local development to fast, reliable deployments. Think of it like moving from rehearsal to opening night: you’ll polish performance, bundle what’s needed, and serve it from a place your audience can reach.

In development, you want speed and instant feedback. In production, you want stability, caching, and small, efficient files. We’ll show you both paths clearly—how to build quickly while you’re iterating, and how to produce optimized assets when you’re ready to host.

> **Terminology:**
>
> - “Dev build” prioritizes fast feedback (watch, hot reload, minimal configuration).
> - “Production build” prioritizes small file sizes, caching, and predictable output for hosting.

You have several options for building your app for development and production. InSpatial supports multiple tools through dedicated plugins and extensions:

- **InServe** (the default choice)
- **InVite** (for Vite integration)
- **InPack** (for Webpack, Rspack, or similar bundlers)

## Getting Started

### 🎯 Prerequisites

Before you start:

- Basic understanding of [JavaScript](https://web.dev/learn/javascript)
- Basic understanding of [TypesScript](https://www.typescriptlang.org/docs/handbook/intro.html)
- Basic understanding of [CLI](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Environment_setup/Command_line)
- Basic understanding of [Renderers](../1.%20graphical-user-interface/rendering/)
- Basic understanding of [Runtimes](../1.%20graphical-user-interface/runtime-markup🟡.md/)

### 🎮 Usage

#### Basic Import

```typescript
// ✅ Recommended: Direct import from kit
import { InServe, InVite, InPack } from "@inspatial/kit/build";

// ❌ Avoid: Package-level import
import { InServe, InVite, InPack } from "@inspatial/kit";
```

<details>
<summary>📦 Installation for Framework Authors</summary>

If you're building a framework or library that needs to include testing & specifications:

```bash
deno install jsr:@in/build
```

##

```bash
npx jsr add @in/build
```

##

```bash
yarn dlx jsr add @in/build
```

##

```bash
pnpm dlx jsr add @in/build
```

##

```bash
bunx jsr add @in/build
```

##

```bash
vlt install jsr:@in/build
```

</details>

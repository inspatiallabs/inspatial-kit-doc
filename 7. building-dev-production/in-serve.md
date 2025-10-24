## InServe (Founders Choice)

#### A simple, dev server and smart builder for your app

InSpatial Serve automatically monitor your files, rebuild anything that changes, and refreshes your website, app, game or XR experience so you always see the latest updates right away.

> **Note:** InServe is a focused, zero‑config dev server with sensible defaults and opt‑in controls when you need them.

#### Quick start

```typescript
// render.ts
import { createRenderer } from "@inspatial/kit/renderer";
import { InServe } from "@inspatial/kit/build";

createRenderer({
  extensions: [InServe()],
});
```

```json
// deno.json
{
  "tasks": {
    "serve": "deno run --allow-net --allow-read --allow-write --allow-env --allow-run src/config/render.ts"
  }
}
```

Run:

```bash
deno task serve
```

#### What it does for you

- **Serves files** from `./dist` on an available port (defaults: 6310 → 6311 → 6312)
- **Hot reloads** the browser via WebSocket (defaults: 8888 → 8889 → 8890)
- **Rebuilds CSS first** so Tailwind classes are always ready before JS
- **Watches sources** in `./src` (and optionally framework roots) to trigger smart rebuilds
- **Handles assets** under `/asset/` with a dev‑friendly cache policy

> **Terminology:** “Hot reload” here means an automatic full page reload when files change. It’s deliberately simple and reliable.

### Configuration

Think of InServe configuration like setting up your workspace. Just like you might adjust your desk height, lighting, and monitor position to work comfortably, you can customize InServe to match how you like to build.

You can adjust behavior in `deno.json` using the `inspatial.serve` block. Everything is optional—if you don't specify something, InServe uses sensible defaults that work for most projects.

> **Note:** All configuration goes inside your project's `deno.json` file under the `inspatial.serve` section. This keeps everything organized in one place.

#### Server Settings

These control how InServe runs your development server—like choosing which door your visitors use to enter your house.

```json
{
  "inspatial": {
    "serve": {
      "server": {
        "httpPorts": [6310, 6311, 6312],
        "wsPorts": [8888, 8889, 8890],
        "host": "localhost",
        "injectClient": true,
        "clientScriptUrl": "",
        "cacheControl": "no-store",
        "assetRouteBase": "/asset/"
      }
    }
  }
}
```

**httpPorts**: The doors your app uses to welcome visitors. InServe tries port 6310 first, then 6311, then 6312 if the previous ones are busy. It's like having backup entrances to your store.

**wsPorts**: The special communication channel for hot reload. Think of it as a direct phone line between your code editor and browser—when you save a file, this channel instantly tells your browser to refresh.

**host**: Where your server lives. `"localhost"` means only your computer can access it. Change to `"0.0.0.0"` if you want other devices on your network (like your phone) to see your app.

**injectClient**: Whether to automatically add the hot reload magic to your pages. It's like having a helpful assistant who automatically refreshes your browser when you make changes.

**clientScriptUrl**: If you want to use your own hot reload script instead of the built-in one. Most people never need to change this.

**cacheControl**: Tells browsers how to handle your files during development. `"no-store"` means "always get the fresh version"—perfect for development where you want to see changes immediately.

**assetRouteBase**: The web address where your images, fonts, and other assets live. Like organizing your files in a specific folder that everyone knows about.

#### File Paths

These tell InServe where to find your files and where to put the built versions—like giving someone directions to your kitchen and telling them where to put the clean dishes.

```json
{
  "paths": {
    "srcDir": "./src",
    "distDir": "./dist",
    "htmlEntry": "./index.html",
    "htmlDist": "./dist/index.html",
    "assetSrcDir": "./src/asset",
    "assetDistDir": "./dist/asset",
    "favicon": "./src/asset/favicon.png",
    "renderEntry": "./src/config/render.ts",
    "cssInput": "./src/config/app.css",
    "cssOutput": "./dist/kit.css",
    "jsOutput": "./dist/bundle.js"
  }
}
```

**srcDir**: Your source code folder—where you write your TypeScript, CSS, and other files. Think of it as your workshop where all the magic happens.

**distDir**: Where InServe puts the finished, ready-to-serve files. Like a display case for your completed work.

**htmlEntry**: Your main HTML file that visitors see first. It's the front door to your app.

**htmlDist**: Where the processed HTML file goes after InServe works on it.

**assetSrcDir** and **assetDistDir**: Where your images, fonts, and other assets live (source) and where they go after processing (distribution).

**favicon**: That little icon that appears in browser tabs. Make sure it exists, or InServe will politely mention it's missing.

**renderEntry**: The TypeScript file that starts your app—usually where you set up your renderer and extensions.

**cssInput** and **cssOutput**: Where your styles start and where they end up after processing (like Tailwind compilation).

**jsOutput**: The final JavaScript bundle that browsers actually run.

#### File Watching

This is how InServe knows when you've changed something and needs to rebuild. It's like having a vigilant security guard who notices every change in your building.

```json
{
  "watch": {
    "include": ["./src", "./index.html"],
    "includeFramework": ["@inspatial/kit", "../kit/src", "../kit/modules"],
    "exclude": ["node_modules", "dist", ".git"]
  }
}
```

**include**: Folders and files to watch for changes. When anything here changes, InServe springs into action.

**includeFramework**: Framework or library folders to watch. Perfect for when you're developing or contributing to InSpatial Kit itself or working with a local copy. Changes here trigger rebuilds too.

**exclude**: Folders to ignore completely. No point watching `node_modules` or `dist`—they're either external dependencies or generated files.

> **Note:** When you change a TypeScript file, InServe automatically rebuilds both JavaScript AND CSS.

#### Build Configuration

This is where the real magic happens... how InServe transforms your source code into something browsers can run.

##### JavaScript Building

```json
{
  "build": {
    "js": {
      "engine": "cli",
      "entrypoints": [],
      "outputDir": "./dist",
      "platform": "browser",
      "minify": false,
      "codeSplitting": false,
      "format": "esm",
      "inlineImports": false,
      "external": [],
      "packages": "bundle",
      "sourcemap": false,
      "write": true,
      "htmlEntrypoints": false
    }
  }
}
```

**engine**: How to bundle your JavaScript. `"cli"` uses Deno's command-line bundler (stable and reliable). `"inprocess"` uses Deno's internal API (faster but requires `--unstable-bundle`).

**entrypoints**: Which files to start bundling from. Empty means use `renderEntry`. Think of these as the main characters in your story—everything else gets pulled in as needed.

**outputDir**: Where to put the bundled files.

**platform**: What environment your code runs in. `"browser"` is perfect for web apps.

**minify**: Whether to make your code smaller by removing spaces and shortening variable names. `false` during development keeps code readable for debugging.

**codeSplitting**: Whether to break your code into multiple files for faster loading. Usually `false` for simple apps.

**format**: What JavaScript module format to use. `"esm"` (ES modules) is modern and widely supported.

**inlineImports**: Whether to include imported code directly instead of keeping separate files.

**external**: Packages to NOT bundle—they'll be loaded separately.

**packages**: Whether to `"bundle"` dependencies into your code or keep them `"external"`.

**sourcemap**: Whether to create debugging maps that connect your bundled code back to your original source. Helpful for debugging but makes files larger.

**write**: Whether to save files to disk (`true`) or keep them in memory (`false` for faster development).

**htmlEntrypoints**: Whether your entrypoints are HTML files instead of JavaScript files.

##### CSS Building

```json
{
  "css": {
    "engine": "iss",
    "input": "./src/config/app.css",
    "output": "./dist/kit.css",
    "contentGlobs": [
      "src/**/*.{ts,tsx,js,jsx}",
      "../kit/modules/**/*.{ts,tsx,js,jsx}"
    ]
  }
}
```

**engine**: How to process your CSS. `"iss"` is InSpatial's built-in processor, `"tailwind"` uses Tailwind CLI, `"none"` does minimal processing.

**input**: Your main CSS file—where you write your styles or import Tailwind.

**output**: Where the processed CSS goes.

**contentGlobs**: File patterns that Tailwind scans for class names. This is crucial—if Tailwind doesn't scan a file, it won't include the classes you use there.

> **Terminology:** "Globs" are file patterns using wildcards. `src/**/*.{ts,tsx}` means "all TypeScript files in src and its subfolders."

##### HTML Building

```json
{
  "html": { "enabled": true }
}
```

Simple but important—whether to copy and process your HTML files.

##### Timing Controls

```json
{
  "timing": {
    "debounceMs": 100,
    "waitAttempts": 50,
    "waitIntervalMs": 100,
    "cssStabilizeAttempts": 10,
    "cssStabilizeIntervalMs": 100
  }
}
```

These control how InServe handles the timing of rebuilds—like setting how long to wait before assuming someone is done talking.

**debounceMs**: How long to wait after a file change before starting a rebuild. Prevents rebuilding on every keystroke.

**waitAttempts** and **waitIntervalMs**: How many times to check if builds are done, and how long to wait between checks.

**cssStabilizeAttempts** and **cssStabilizeIntervalMs**: How to detect when CSS files are finished writing. Prevents reloading before Tailwind is done.

#### Discovery Settings

These help InServe find important files automatically, like having a smart assistant who knows where you usually keep things.

```json
{
  "discovery": {
    "renderSearch": [
      "./render.ts",
      "./src/render.ts",
      "./src/config/render.ts"
    ],
    "kitRoots": ["@inspatial/kit", "../kit/src", "../kit/modules"]
  }
}
```

**renderSearch**: Places to look for your main render file if it's not in the default location.

**kitRoots**: Framework folders that contain components and utilities. InServe includes these in Tailwind scanning and extension discovery.

#### App-level Trigger Types

You can declare custom extension trigger type files directly in your app's `deno.json`. This is perfect when you build your own extensions or want to include trigger types without modifying upstream modules.

```json
{
  "inspatial": {
    "triggerTypes": [
      "./src/types/my-custom-triggers.d.ts",
      "../shared/extensions/trigger-types.d.ts"
    ]
  }
}
```

> **What it does:** InServe will include these files when generating `src/types/extension-trigger-types.generated.d.ts`, merging them alongside any module-declared trigger types.

> **Note:** Paths are relative to your app root. Files must exist and export `interface ExtensionTriggerTypes { ... }`.

#### JS bundling modes

- **inprocess:** uses Deno’s runtime API `Deno.bundle()` in‑process.
  - Pros: supports HTML entrypoints, optional in‑memory outputs (`write=false`), fewer shell hops.
  - Requirements: Deno ≥ 2.5 with `--unstable-bundle`.
  - Set via `deno.json`: `inspatial.serve.build.js.engine = "runtime"` (alias for inprocess)
- **subprocess:** invokes the `deno bundle` CLI as a child process.
  - Pros: very stable, no unstable flags.
  - Behavior: writes to disk; HTML entrypoints supported via CLI flags.
  - Set via `deno.json`: `inspatial.serve.build.js.engine = "cli"` (alias for subprocess)

> **Note:** When `htmlEntrypoints` is true, the server skips manual HTML copying because the bundler rewrites your HTML with hashed asset names.

<details>
  <summary><strong>Genius</strong></summary>

- Build orchestration uses a debounced BuildQueue: if both CSS and JS are queued, CSS runs first to ensure Tailwind (when installed) classes exist before JS reloads. Timings (debounce, waits, stabilization) are configurable.
- Tailiwind CSS is built via Tailwind CLI with content globs from <code>build.css.contentGlobs</code> and optionally augmented by <code>discovery.kitRoots</code>. The server waits for <code>cssOutput</code> to stabilize (size unchanged) to avoid race conditions.
- JS bundling maps 1:1 to either <code>deno bundle</code> (subprocess) or <code>Deno.bundle()</code> (inprocess). Inprocess supports HTML entrypoints and optional in‑memory outputs (<code>write: false</code>) for fast dev serving; subprocess passes equivalent flags and falls back automatically on runtime errors.
- The WebSocket server binds the first available port from <code>server.wsPorts</code>. HTML responses get a tiny injected client (unless <code>injectClient</code> is false or a custom <code>clientScriptUrl</code> is provided) that reloads the page on change.
- The file server serves from <code>paths.distDir</code>, applies SPA fallback (serves <code>index.html</code> when no extension), and handles <code>/asset/</code> (or <code>assetRouteBase</code>) with <code>Cache-Control: no-store</code> in dev.
- Watchers are configured via <code>watch.include</code> and <code>watch.includeFramework</code>. Any TS/JS/TSX/JSX edit also queues a CSS rebuild to catch new Tailwind classes; pure asset edits trigger reload without rebuilding.

</details>

#### Hot reload client

- Toggle with `server.injectClient` (true by default)
- Provide a custom script with `server.clientScriptUrl` if you want to host your own client

#### Ports and environment overrides

- HTTP ports: `server.httpPorts` or `INSPATIAL_SERVE_HTTP_PORTS=6310,6311`
- WS ports: `server.wsPorts` or `INSPATIAL_SERVE_WS_PORTS=8888,8889`

#### Tailwind and CSS stability

- Styles build first to ensure new style classes exist before JS reloads
- `build.css.contentGlobs` controls where Tailwind scans for class names
- The server waits briefly for `cssOutput` to stabilize before reloading the page

### Real-World Examples

Here are some common scenarios you might encounter, explained like you're setting up your development environment with a friend.

#### Example 1: Ben's Lightning-Fast Development Setup

Ben works on InSpatial Kit itself and wants the fastest possible rebuilds. He uses in-memory bundling to skip writing files to disk.

```json
{
  "inspatial": {
    "serve": {
      "watch": {
        "includeFramework": ["../inspatial-kit/modules"]
      },
      "build": {
        "js": {
          "engine": "inprocess",
          "write": false,
          "sourcemap": "inline"
        }
      }
    }
  }
}
```

> **Note:** This setup requires `deno run --unstable-bundle` but gives you the fastest possible development experience.

#### Example 2: Charlotte's Multi-Device Testing

Charlotte needs to test her app on her phone and tablet, so she opens up the server to her local network.

```json
{
  "inspatial": {
    "serve": {
      "server": {
        "host": "0.0.0.0",
        "httpPorts": [3000]
      }
    }
  }
}
```

Now she can visit `http://192.168.1.100:3000` (her computer's IP) from any device on her network.

#### Example 3: Mike's Tailwind-Heavy Project

Mike uses lots of Tailwind classes and wants to make sure InServe catches them all, even in his component library.

```json
{
  "inspatial": {
    "serve": {
      "build": {
        "css": {
          "engine": "tailwindiss",
          "contentGlobs": [
            "src/**/*.{ts,tsx,js,jsx}",
            "../my-component-lib/**/*.{ts,tsx}",
            "../../shared-components/**/*.tsx"
          ]
        }
      },
      "discovery": {
        "kitRoots": ["../my-component-lib", "../../shared-components"]
      }
    }
  }
}
```

> **Note:** Tailwindiss is different from Tailwindcss. ISS works with or alongside the InSpatial Style Sheet Engine. Its the best of both worlds when you want to use ISS which is the recommended style engine with Tailwind. 

#### Example 4: Eli's Production-Ready Development

Eli likes to test his app in conditions closer to production, so he enables minification and source maps during development.

```json
{
  "inspatial": {
    "serve": {
      "build": {
        "js": {
          "minify": true,
          "sourcemap": "linked",
          "codeSplitting": true
        }
      }
    }
  }
}
```

#### Example 5: Ben Barrow's Custom Asset Setup

Ben Barrow has a complex asset structure with multiple folders and wants custom cache control.

```json
{
  "inspatial": {
    "serve": {
      "server": {
        "assetRouteBase": "/static/",
        "cacheControl": "max-age=3600"
      },
      "paths": {
        "assetSrcDir": "./assets",
        "assetDistDir": "./public/static"
      }
    }
  }
}
```

#### Example 6: HTML-First Development

Sometimes you want to start with an HTML file instead of a TypeScript entry point—perfect for simpler projects or when migrating existing sites.

```json
{
  "inspatial": {
    "serve": {
      "build": {
        "js": {
          "engine": "inprocess",
          "entrypoints": ["./index.html"],
          "htmlEntrypoints": true
        }
      }
    }
  }
}
```

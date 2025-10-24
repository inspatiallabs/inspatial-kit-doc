# Limitations

InSpatial Kit is currently at version v0.7 and is mostly stable. However, there are a few limitations to be aware of:

1. **Unstyled by Default:** Widgets and components are unstyled unless you manually include the InSpatial `kit.css` file/variables (The `kit.css` file is not currently public or open-source). (In the future, `inspatial-serve` will handle this automatically.) Tailwind is also supported. Our goal is to move toward eliminating external CSS dependencies entirely.

2. **No HMR with InServe:** If you use Vite/RSPack, Hot Module Replacement (HMR) works out of the box. However, with InSpatial's zero-config build tool, InServe, only live reload/refresh is currently supported. HMR support is planned for future updates.

3. **Large Bundle Size:** InSpatial Kit is ambitious it's built on a comprehensive infrastructure stack, even with zero dependencies. As a result, the entire stack is currently shipped with your application, leading to larger bundle sizes. From v1 onward, this will improve with the introduction of the InSpatial CLI to allow you to ship the modules you want and ongoing optimizations.

# Leaving InSpatial Land

InSpatial Kit provides compatibility with legacy integrations and concepts such as CommonJS, Vite, Webpack, RSPack, and SSR. However, we are not actively committed to maintaining feature parity or guaranteeing stable support for these approaches. Our primary focus is on building a modern, universal platform.

## CommonJS

Although you can switch InSpatial Serve from ESM (the default) to CommonJS with a single flag, neither InSpatial Kit nor its core modules are designed around CommonJS. We believe CommonJS is outdated and unlikely to remain relevant in the long term.

```json
// deno.json
{
  "inspatial": {
    "serve": {
      "build": {
        "js": {
          "format": "cjs"
        }
      }
    }
  }
}
```

For details, see [Building For Dev & Production](../7.%20building-dev-production🟢/introduction.md).

## Vite, Webpack, RSPack

InSpatial includes its own next-generation build tool, [InSpatial Serve](../7.%20building-dev-production🟢/in-serve.md), specifically designed for universal and cross-platform development not just the web. While we do provide out-of-the-box support for web-centric tools like [Vite](../7.%20building-dev-production🟢/in-vite.md) and [Webpack/RSPack](../7.%20building-dev-production🟢/in-pack.md), these third-party tools may not achieve full feature parity or receive the same level of support as InSpatial Serve.

## SSR

We approach SSR (server-side rendering) with caution. While SSR is supported, we fundamentally believe that, in many cases, it introduces unnecessary complexity and bloat. The drawbacks often outweigh the benefits. For more insight, see our [Philosophy](./philosophy.md).

If you choose to rely on these alternative tools and paradigms, you are **Leaving InSpatial Land**. This means you will be responsible for maintaining compatibility and resolving any issues that arise on your own.

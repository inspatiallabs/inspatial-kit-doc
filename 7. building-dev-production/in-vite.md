## InVite

Use `InVite` plugin for Vite. Create a `vite.config.ts` file and add the following conifguration

##### Pragma Config (Default)

```typescript
import { defineConfig } from "vite";
import { InVite } from "@inspatial/kit/build";

export default defineConfig({
  server: {
    port: 6310,
  },
  esbuild: {
    jsxFactory: "R.c",
    jsxFragment: "R.f",
  },
  plugins: [InVite()],
  build: {
    target: "esnext",
  },
});
```

##### Automatic Config

```typescript
import { defineConfig } from "vite";

export default defineConfig({
  esbuild: {
    jsx: "automatic",
    jsxImportSource: "@inspatial/kit", // This tells Vite where to find the runtime
  },
});
```

#### Babel (`.babelrc.json`)

```json
{
  "presets": [
    [
      "@babel/preset-react",
      {
        "runtime": "automatic",
        "importSource": "@inspatial/kit"
      }
    ]
  ]
}
```

See the [Vite](https://vite.dev/) for setup, usage, and API details.

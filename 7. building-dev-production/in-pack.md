## InPack

Use the `InPack` plugin for Webpack and/or RSPack/RSBuild

```javascript
import { defineConfig } from "@rsbuild/core";
import { InPack } from "@inspatial/kit/build";

export default defineConfig({
  plugins: [InPack()],
  // ... jsx config
});
```

See the [Webpack](https://webpack.js.org/) for setup, usage, and API details.
See the [Rspack](https://rspack.rs/) for setup, usage, and API details.
See the [Rsbuild](https://rsbuild.rs/) for setup, usage, and API details.

# Limitations

InSpatial Kit is currently at version v0.7 and is mostly stable. However, there are a few limitations to be aware of:

1. **Unstyled by Default:** Widgets and components are unstyled unless you manually include the InSpatial `kit.css` file/variables (The `kit.css` file is not currently public or open-source). (In the future, `inspatial-serve` will handle this automatically.) Tailwind is also supported. Our goal is to move toward eliminating external CSS dependencies entirely.

2. **No HMR with InServe:** If you use Vite/RSPack, Hot Module Replacement (HMR) works out of the box. However, with InSpatial's zero-config build tool, InServe, only live reload/refresh is currently supported. HMR support is planned for future updates.

3. **Large Bundle Size:** InSpatial Kit is ambitious it's built on a comprehensive infrastructure stack, even with zero dependencies. As a result, the entire stack is currently shipped with your application, leading to larger bundle sizes. From v1 onward, this will improve with the introduction of the InSpatial CLI to allow you to ship the modules you want and ongoing optimizations.
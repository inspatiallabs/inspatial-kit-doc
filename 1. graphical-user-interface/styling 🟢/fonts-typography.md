## Fonts & Typography

### Universal Font System

InSpatial provides a comprehensive typography system with two complementary font sources: **Primitive Fonts** (70+ premium curated fonts) and **Google Fonts** (1000+ web fonts). Both work seamlessly across all platforms with intelligent loading, fallbacks, and optimization.

```ascii
╔══════════════════════════════════════════════════════════════════════════════╗
║                         InSpatial Font Architecture                          ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  ┌─────────────────────┐              ┌─────────────────────┐                ║
║  │   PRIMITIVE FONTS   │              │   GOOGLE FONTS      │                ║
║  │   ┌─────────────┐   │              │   ┌─────────────┐   │                ║
║  │   │ 70+ Premium │   │              │   │ 1000+ Web   │   │                ║
║  │   │ Curated     │   │              │   │ Fonts       │   │                ║
║  │   │ Fonts       │   │              │   │             │   │                ║
║  │   └─────────────┘   │              │   └─────────────┘   │                ║
║  │                     │              │                     │                ║
║  │ ✅ CDN Optimized     │              │ ✅ Stub System      │              ║
║  │ ✅ Zero Setup        │              │ ✅ CLI Installation │              ║
║  │ ✅ Premium Quality   │              │ ✅ Type Safe        │              ║
║  │ ✅ Performance      │              │ ✅ Fallback Ready   │               ║
║  └─────────────────────┘              └─────────────────────┘                ║
║                          ╲            ╱                                      ║
║                           ╲          ╱                                       ║
║                            ╲        ╱                                        ║
║                             ╲      ╱                                         ║
║                              ╲    ╱                                          ║
║                               ╲  ╱                                           ║
║                                ╲╱                                            ║
║                    ┌─────────────────────┐                                   ║
║                    │   UNIFIED FONT API  │                                   ║
║                    │                     │                                   ║
║                    │ ┌─────────────────┐ │                                   ║
║                    │ │ Font Merging    │ │                                   ║
║                    │ │ Smart Loading   │ │                                   ║
║                    │ │ Fallback System │ │                                   ║
║                    │ │ Cross-Platform  │ │                                   ║
║                    │ └─────────────────┘ │                                   ║
║                    └─────────────────────┘                                   ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### 1. Primitive Fonts (Premium Built-in)

InSpatial Primitive Fonts are 70+ carefully curated premium fonts included by default. They're served via a highly optimized CDN with zero setup required.

#### Available Primitive Fonts

```ascii
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PRIMITIVE FONT LIBRARY                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SANS-SERIF                    SERIF                     DISPLAY            │
│  ┌─────────────┐              ┌─────────────┐           ┌─────────────┐     │
│  │ • inter     │              │ • bertha    │           │ • aerospace │     │
│  │ • poppins   │              │ • bernados  │           │ • attack    │     │
│  │ │ montserrat│              │ • carolin   │           │ • bionix    │     │
│  │ • lato      │              │ • gaoel     │           │ • brawls    │     │
│  │ • rubik     │              │ • heather   │           │ • engine    │     │
│  │ • euclid-   │              │ • logotype  │           │ • nanotech  │     │
│  │   circular  │              │ • morality  │           │ • polaris   │     │
│  │ • inder     │              │ • nafasyah  │           │ • qualux    │     │
│  │ • micro     │              │ • notche    │           │ • ransom    │     │
│  │ • monica    │              │ • parizaad  │           │ • safari    │     │
│  │ • naon      │              │ • polly     │           │ • slamdunk  │     │
│  │ • oklean    │              │ • quora     │           │ • viora     │     │
│  │ • trebuchet │              │ • remura    │           │ • zebrawood │     │
│  └─────────────┘              └─────────────┘           └─────────────┘     │
│                                                                             │
│  SCRIPT & CREATIVE             MONOSPACE                 SPECIALTY          │
│  ┌─────────────┐              ┌─────────────┐           ┌─────────────┐     │
│  │ • actual    │              │ • jls       │           │ • aeion     │     │
│  │ • along     │              │ • kimura    │           │ • alternox  │     │
│  │ • amithen   │              │ • lovelo    │           │ • amithen   │     │
│  │ • ankle     │              │ • moisses   │           │ • aperture  │     │
│  │ • anything  │              │ • numaposa  │           │ • aqum      │     │
│  │ • brighton  │              │ • rockley   │           │ • broad     │     │
│  │ • candace   │              │ • ronald    │           │ • candace   │     │
│  │ • congenial │              │ • sheylla   │           │ • dakar     │     │
│  │ • denson    │              │ • stampbor  │           │ • denson    │     │
│  │ • dumeh     │              │ • sweetsnow │           │ • dumeh     │     │
│  │ • editors-  │              └─────────────┘           │ • editors-  │     │
│  │   note      │                                        │   note      │     │
│  │ • elsone    │                                        │ • elsone    │     │
│  │ • enrique   │                                        │ • enrique   │     │
│  │ • folker    │                                        │ • folker    │     │
│  │ • fonzy     │                                        │ • fonzy     │     │
│  │ • foregen   │                                        │ • foregen   │     │
│  │ • goodly    │                                        │ • goodly    │     │
│  │ • hadeed    │                                        │ • hadeed    │     │
│  │ • queen-    │                                        │ • queen-    │     │
│  │   rogette   │                                        │   rogette   │     │
│  └─────────────┘                                        └─────────────┘     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Using Primitive Fonts

```typescript
import { PrimitiveFontProps } from "@inspatial/kit/style";

// Access any of the 70+ premium fonts
const { inter, poppins, montserrat, lato, rubik } = PrimitiveFontProps;

// Use fonts in your application
const myFont = inter({
  src: "/path/to/inter.woff2",
  weight: "400",
  style: "normal",
  fallback: ["system-ui", "sans-serif"],
  preload: true,
  variable: "--font-inter",
});

// Access font properties
console.log(myFont.className); // CSS class name
console.log(myFont.style); // Font style properties
```

> **Note:** Primitive Fonts are included by default with zero setup. They're served via CDN and optimized for performance with minimal payload (~15KB total).

### 2. Google Fonts Integration

InSpatial provides type-safe access to the entire Google Fonts library with an intelligent stub system for optimal bundle size.

#### How Google Font Stubs Work

```ascii
┏━━━━━━━━━━━━━━━━━━━━━━━ INITIAL PACKAGE ━━━━━━━━━━━━━━━━━━━━━━━┓
┃                                                               ┃
┃  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    ┃
┃  ┃                    LIGHTWEIGHT STUBS                  ┃    ┃
┃  ┃                                                       ┃    ┃
┃  ┃  ┏━━━━━━━━━━━━━━┓      ┏━━━━━━━━━━━━━━━━━━━━━━━┓      ┃    ┃
┃  ┃  ┃  Google Font ┃      ┃      stub.ts          ┃      ┃    ┃
┃  ┃  ┃  Interface   ┃━━━━━▶  ┏━━━━━━━━━━━━━━━┓         
┃  ┃  ┗━━━━━━━━━━━━━━┛      ┃  ┃ export const     
┃  ┃                        ┃  ┃ Roboto = ...          
┃  ┃                        ┃  ┃ Open_Sans = .           
┃  ┃        PROVIDES        ┃  ┃ Lato = ...              
┃  ┃      CONSISTENT        ┃  ┃ ...                    
┃  ┃         API            ┃  ┗━━━━━━━━━━━━━━━┛    ┃    
┃  ┃                        ┗━━━━━━━━━━━━━━━━━━━━━━━┛     
┃  ┃                                                     
┃  ┃  ⚠️ System fonts used as fallbacks                       
┃  ┃  ⚠️ Warning messages shown when used                     
┃  ┃  ⚠️ Small footprint (~15KB total)                        
┃  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  ┃
┃                                                             ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                               │
                               │
                               ▼
┏━━━━━━━━━━━━ AFTER RUNNING: deno task fonts:google:install ━━━━━━━━━━━┓
┃                                                                      
┃  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    ┏━━━━━━━━━━━━━━━━━┓  
┃  ┃         REAL IMPLEMENTATIONS           ┃    ┃                  
┃  ┃                                        ┃    ┃    FONT FILES    
┃  ┃  ┏━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━┓ ┃    ┃  ┏━━━━━━━━━━━┓  
┃  ┃  ┃  Google Font ┃     ┃   fonts.ts   ┃ ┃    ┃   roboto.woff
┃  ┃  ┃  Interface   ┃━━━━▶┃ ┏━━━━━━━━━━┓   =====  ┗━━━━━━━━━━━┛  
┃  ┃  ┗━━━━━━━━━━━━━━┛     ┃ ┃Roboto    ┃                         
┃  ┃                       ┃ ┃Open_Sans ┃◀┫─┼────╋─▶┏━━━━━━━━━━━┓  
┃  ┃                       ┃ ┃Lato      ┃◀┫─┼────╋─▶opensans.woff  
┃  ┃                       ┃ ┃...       ┃ ┃        ┗━━━━━━━━━━━┛  
┃  ┃  SAME API,            ┃ ┗━━━━━━━━━━┛ ┃                    
┃  ┃  REAL FUNCTIONALITY   ┗━━━━━━━━━━━━━━┛           ┏━━━━━━━━━━━┓ 
┃  ┃                                                    lato.woff  
┃  ┃  ✅ Actual font loading                      ┃   ┗━━━━━━━━━━━┛  
┃  ┃  ✅ CSS @font-face integration               ┃                  
┃  ┃  ✅ Optimized font display                   ┗━━━━━━━━━━━━━━━━━┛  
┃  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛                         
┃                                                                      
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

#### Using Google Fonts

```typescript
import { Roboto, Open_Sans, Lato } from "@inspatial/kit/style";

// Create a Google Font instance with options
const robotoFont = Roboto({
  weight: ["400", "700"],
  style: "normal",
  subsets: ["latin", "latin-ext"],
  display: "swap",
  preload: true,
  fallback: ["system-ui", "sans-serif"],
  adjustFontFallback: true,
  variable: "--font-roboto",
});

// Use the font in your styling
const styles = {
  fontFamily: robotoFont.style.fontFamily,
  className: robotoFont.className,
};
```

#### Installing Google Fonts

```bash
# Install all Google Fonts
deno task fonts:google:install

# Install only popular fonts (recommended)
deno task fonts:google:install -- --popular

# Install specific fonts
deno task fonts:google:install -- --families=Roboto,Open+Sans,Lato

# Uninstall Google Fonts (revert to stubs)
deno task fonts:google:uninstall
```

### 3. Font Merging & Unified API

InSpatial provides utilities to combine multiple font sources into a unified system:

```typescript
import {
  inspatialFontStyle,
  inspatialFontClass,
  inspatialFontVariable,
} from "@inspatial/kit/style";

// Combined font styles from all sources
const allFontStyles = inspatialFontStyle;

// Combined CSS classes
const allFontClasses = inspatialFontClass;

// Combined CSS variables
const allFontVariables = inspatialFontVariable;

// Usage in layout
function RootLayout() {
  return (
    <html className={allFontVariables}>
      <body className={allFontClasses} style={allFontStyles}>
        {/* Your app */}
      </body>
    </html>
  );
}
```

### 4. Advanced Font Configuration

#### Custom Font Loading

```typescript
import { primitiveFont } from "@inspatial/kit/style";

// Custom font with advanced options
const customFont = primitiveFont({
  src: [
    {
      path: "/fonts/custom-regular.woff2",
      weight: "400",
      style: "normal",
    },
    {
      path: "/fonts/custom-bold.woff2",
      weight: "700",
      style: "normal",
    },
  ],
  display: "swap",
  fallback: ["system-ui", "sans-serif"],
  preload: true,
  variable: "--font-custom",
  declarations: [
    { prop: "font-feature-settings", value: '"liga" 1, "kern" 1' },
    { prop: "font-optical-sizing", value: "auto" },
  ],
});
```

#### Typography Variants with Fonts

```typescript
import { createStyle } from "@inspatial/kit/style";
import { inter, poppins } from "@inspatial/kit/style";

const TypographyStyle = createStyle({
  settings: {
    variant: {
      heading: [
        poppins.className,
        { web: { fontFamily: poppins.style.fontFamily, fontWeight: 600 } },
      ],
      body: [
        inter.className,
        { web: { fontFamily: inter.style.fontFamily, fontWeight: 400 } },
      ],
      caption: [
        inter.className,
        {
          web: {
            fontFamily: inter.style.fontFamily,
            fontWeight: 300,
            fontSize: "0.875rem",
          },
        },
      ],
    },
  },
  defaultSettings: { variant: "body" },
});
```

### 5. Performance & Optimization

#### Font Loading Strategies

```ascii
┌─────────────────────────────────────────────────────────────────────────────┐
│                          FONT LOADING STRATEGIES                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  DISPLAY STRATEGY          BEHAVIOR                    BEST FOR             │
│  ┌─────────────────┐      ┌─────────────────┐        ┌─────────────────┐    │
│  │ auto (default)  │ ──── │ Browser decides │ ────── │ Most cases      │    │
│  │ block           │ ──── │ Block 3s, swap  │ ────── │ Critical text   │    │
│  │ swap            │ ──── │ Immediate swap  │ ────── │ Performance     │    │
│  │ fallback        │ ──── │ Block 100ms     │ ────── │ Progressive     │    │
│  │ optional        │ ──── │ No block, cache │ ────── │ Non-critical    │    │
│  └─────────────────┘      └─────────────────┘        └─────────────────┘    │
│                                                                             │
│  PRELOAD STRATEGY          WHEN TO USE                                      │
│  ┌─────────────────┐      ┌─────────────────────────────────────────────┐   │
│  │ preload: true   │ ──── │ Critical fonts, above-the-fold content      │   │
│  │ preload: false  │ ──── │ Secondary fonts, below-the-fold content     │   │
│  └─────────────────┘      └─────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Fallback Font Optimization

```typescript
// Automatic fallback adjustment for better CLS
const optimizedFont = Roboto({
  weight: "400",
  adjustFontFallback: true, // Automatically adjusts fallback metrics
  fallback: ["system-ui", "Arial", "sans-serif"],
});

// Manual fallback configuration
const manualFont = primitiveFont({
  src: "/fonts/custom.woff2",
  fallback: ["Inter", "system-ui", "sans-serif"],
  declarations: [
    { prop: "size-adjust", value: "95%" },
    { prop: "ascent-override", value: "90%" },
    { prop: "descent-override", value: "22%" },
  ],
});
```

### 6. Cross-Platform Considerations

```typescript
// Platform-specific font handling
const responsiveFont = createStyle({
  base: [
    {
      web: { fontFamily: "Inter, system-ui, sans-serif" },
      ios: { fontFamily: "SF Pro Display, system-ui, sans-serif" },
      android: { fontFamily: "Roboto, system-ui, sans-serif" },
      visionOS: { fontFamily: "SF Pro Display, system-ui, sans-serif" },
    },
  ],
});
```

> **Note:** InSpatial automatically handles platform-specific font optimizations, including font-display strategies, preloading, and fallback adjustments for optimal performance across all environments.

---

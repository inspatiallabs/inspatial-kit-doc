# Sidebar

#### The collapsible navigation menu for your app

The `Sidebar` a Navigation component.

Just like how a physical sidebar in a book helps you navigate chapters, this component helps users move through different sections of your application. It supports two main patterns: you can either add navigation items as JSX children (like building with blocks), or pass them as a structured widget tree (like handing over a complete blueprint).

> **Note:** Sidebar automatically handles active states for navigation items, showing users exactly where they are in your app. It also remembers whether it's minimized or expanded, so users don't lose their preference when moving around.

> **Terminology:**
>
> - **MPV (MultiPageView)**: Navigation items that take users to different routes/pages in your app
> - **SPV (SinglePageView)**: Selection items that work like radio buttons to switch views on the same page
> - **Minimized**: The collapsed state showing only icons (like a closed drawer)
> - **Expanded**: The full state showing icons and labels (like an open drawer)

<details>
  <summary><strong>Genius</strong></summary>

The Sidebar uses a per-instance state management system via `useSidebar` store. Each sidebar instance maintains its own minimized state in a shared `minimizedById` record, enabling multiple sidebars with independent collapse states.

**Key behaviors:**

- Controlled vs uncontrolled: `minimized` prop (controlled) overrides internal state
- Children resolution: Detects widget-tree shape by checking for `header`, `group`, `item`, `footer`, or `toggle` keys and absence of React element signatures (`$$typeof`, `type`, `props`)
- Array normalization: Uses `ensureArray` from `@in/vader` to handle single items or arrays uniformly
- Toggle visibility: Conditionally renders based on `showToggle` prop and hover state (always visible when minimized, hover-only when expanded)
- Active route tracking: `SidebarItem` internally uses `getActiveRoute` helper with fallbacks to global route systems and browser location

**Component tree structure:**

```
Sidebar (wrapper)
  ├─ children (JSX or mapped from widget-tree)
  │   ├─ SidebarHeader (optional)
  │   ├─ SidebarGroup (collapsible, optional)
  │   │   └─ SidebarItem[]
  │   ├─ SidebarItem (MPV or SPV)
  │   └─ SidebarFooter (optional)
  └─ SidebarToggle (conditional, gated by showToggle + hover/minimized)
```

</details>

## Getting Started

### 🎯 Prerequisites

Before you start:

- Basic understanding of [Routing](.././../../4.%20routing-navigation/routing-navigation🟡.md)

### 🎮 Usage

Here's how you might use Sidebar in real life:

#### Example 1: Simple Navigation Menu

Let's create a sidebar for a blog app where users can browse different sections:

```jsx
import { Sidebar } from "@inspatial/kit/navigation";
import { HomeIcon, ArticleIcon, AboutIcon } from "@inspatial/kit/icon";

export function BlogSidebar() {
  return (
    <Sidebar
      showToggle={true}
      defaultMinimized={false}
      children={{
        item: [
          {
            routeView: "MPV",
            to: "/home",
            icon: <HomeIcon scale="10xs" />,
            children: "Home",
          },
          {
            routeView: "MPV",
            to: "/articles",
            icon: <ArticleIcon scale="10xs" />,
            children: "Articles",
          },
          {
            routeView: "MPV",
            to: "/about",
            icon: <AboutIcon scale="10xs" />,
            children: "About",
          },
        ],
      }}
    />
  );
}
```

#### Example 2: Single Page View with SPV

Sometimes you want to switch views without changing routes. Here's a data explorer that switches between different panels:

```jsx
import { createState, $ } from "@inspatial/kit/state";
import { Sidebar } from "@inspatial/kit/navigation";
import { DBIcon, ChartIcon, APIIcon } from "@inspatial/kit/icon";

export function DataExplorerSidebar() {
  // Create local state for active view
  const activeView = createState("database");

  // Create reactive conditions for each view
  const isDatabase = $(() => activeView.get() === "database");
  const isCharts = $(() => activeView.get() === "charts");
  const isAPI = $(() => activeView.get() === "api");

  return (
    <Sidebar
      defaultMinimized={false}
      showToggle={true}
      style={{ web: { gap: "52px" } }}
      children={{
        item: [
          {
            routeView: "SPV",
            name: "data-view",
            value: "database",
            selected: isDatabase,
            "on:change": () => activeView.set("database"),
            icon: <DBIcon scale="10xs" />,
            children: "Database",
          },
          {
            routeView: "SPV",
            name: "data-view",
            value: "charts",
            selected: isCharts,
            "on:change": () => activeView.set("charts"),
            icon: <ChartIcon scale="10xs" />,
            children: "Analytics",
          },
          {
            routeView: "SPV",
            name: "data-view",
            value: "api",
            selected: isAPI,
            "on:change": () => activeView.set("api"),
            icon: <APIIcon scale="10xs" />,
            children: "API Explorer",
          },
        ],
      }}
    />
  );
}
```

#### Example 3: Adding a Header and Footer

Let's add branding at the top and utility links at the bottom:

```jsx
import { Sidebar } from "@inspatial/kit/navigation";
import {
  LogoIcon,
  HomeIcon,
  ArticleIcon,
  SettingsIcon,
  HelpIcon,
} from "@inspatial/kit/icon";

export function AppSidebar() {
  return (
    <Sidebar
      showToggle={true}
      defaultMinimized={false}
      children={{
        header: {
          logo: <LogoIcon size="md" />,
          title: "My Blog",
        },

        item: [
          {
            routeView: "MPV",
            to: "/home",
            icon: <HomeIcon scale="10xs" />,
            children: "Home",
          },
          {
            routeView: "MPV",
            to: "/articles",
            icon: <ArticleIcon scale="10xs" />,
            children: "Articles",
          },
        ],

        footer: {
          children: (
            <>
              <SidebarItem
                routeView="MPV"
                to="/settings"
                icon={<SettingsIcon scale="10xs" />}
              >
                Settings
              </SidebarItem>

              <SidebarItem
                routeView="MPV"
                to="/help"
                icon={<HelpIcon scale="10xs" />}
              >
                Help
              </SidebarItem>
            </>
          ),
        },
      }}
    />
  );
}
```

#### Example 4: Organizing with Groups

Here's a full-featured sidebar for a project management app with collapsible groups:

```jsx
import { Sidebar, SidebarItem } from "@inspatial/kit/navigation";
import {
  LogoIcon,
  ProjectIcon,
  TeamIcon,
  TaskIcon,
  SettingsIcon,
  HelpIcon,
} from "@inspatial/kit/icon";

export function ProjectSidebar() {
  return (
    <Sidebar
      showToggle={true}
      defaultMinimized={false}
      expandedSize="2xl"
      minimizedSize="sm"
      children={{
        header: {
          logo: <LogoIcon size="md" />,
          title: "ProjectHub",
        },

        group: [
          {
            id: "main-nav",
            title: "Projects",
            icon: <ProjectIcon scale="10xs" />,
            defaultExpanded: true,
            children: (
              <>
                <SidebarItem
                  routeView="MPV"
                  to="/projects/active"
                  icon={<TaskIcon scale="10xs" />}
                >
                  Active Projects
                </SidebarItem>

                <SidebarItem
                  routeView="MPV"
                  to="/projects/archived"
                  icon={<TaskIcon scale="10xs" />}
                >
                  Archived
                </SidebarItem>
              </>
            ),
          },
          {
            id: "team-nav",
            title: "Team",
            icon: <TeamIcon scale="10xs" />,
            defaultExpanded: false,
            children: (
              <>
                <SidebarItem routeView="MPV" to="/team/members">
                  Members
                </SidebarItem>

                <SidebarItem routeView="MPV" to="/team/roles">
                  Roles
                </SidebarItem>
              </>
            ),
          },
        ],

        footer: {
          children: (
            <>
              <SidebarItem
                routeView="MPV"
                to="/settings"
                icon={<SettingsIcon scale="10xs" />}
              >
                Settings
              </SidebarItem>

              <SidebarItem
                routeView="MPV"
                to="/help"
                icon={<HelpIcon scale="10xs" />}
              >
                Help
              </SidebarItem>
            </>
          ),
        },
      }}
    />
  );
}
```

#### Example 5: Using JSX Children (Alternative Pattern)

If you prefer writing JSX directly, you can nest `SidebarItem` components as children instead of using the widget-tree pattern:

```jsx
import {
  Sidebar,
  SidebarHeader,
  SidebarGroup,
  SidebarItem,
  SidebarFooter,
} from "@inspatial/kit/navigation";
import {
  LogoIcon,
  ProjectIcon,
  TeamIcon,
  TaskIcon,
  SettingsIcon,
  HelpIcon,
} from "@inspatial/kit/icon";

export function ProjectSidebar() {
  return (
    <Sidebar
      showToggle={true}
      defaultMinimized={false}
      expandedSize="2xl"
      minimizedSize="sm"
    >
      {/* Header with logo and title */}
      <SidebarHeader logo={<LogoIcon size="md" />} title="ProjectHub" />

      {/* Main navigation group */}
      <SidebarGroup
        id="main-nav"
        title="Projects"
        icon={<ProjectIcon scale="10xs" />}
        defaultExpanded={true}
      >
        <SidebarItem
          routeView="MPV"
          to="/projects/active"
          icon={<TaskIcon scale="10xs" />}
        >
          Active Projects
        </SidebarItem>

        <SidebarItem
          routeView="MPV"
          to="/projects/archived"
          icon={<TaskIcon scale="10xs" />}
        >
          Archived
        </SidebarItem>
      </SidebarGroup>

      {/* Team section */}
      <SidebarGroup
        id="team-nav"
        title="Team"
        icon={<TeamIcon scale="10xs" />}
        defaultExpanded={false}
      >
        <SidebarItem routeView="MPV" to="/team/members">
          Members
        </SidebarItem>

        <SidebarItem routeView="MPV" to="/team/roles">
          Roles
        </SidebarItem>
      </SidebarGroup>

      {/* Footer with settings */}
      <SidebarFooter>
        <SidebarItem
          routeView="MPV"
          to="/settings"
          icon={<SettingsIcon scale="10xs" />}
        >
          Settings
        </SidebarItem>

        <SidebarItem
          routeView="MPV"
          to="/help"
          icon={<HelpIcon scale="10xs" />}
        >
          Help
        </SidebarItem>
      </SidebarFooter>
    </Sidebar>
  );
}
```

### What You Get

The Sidebar gives you a complete navigation system that:

- **Automatically tracks active routes**: Highlights the current page so users always know where they are
- **Supports two view modes**: Use MPV for multi-page navigation or SPV for single-page view switching
- **Remembers its state**: Stays minimized or expanded as users navigate
- **Adapts to space**: Toggle between full labels and icon-only views
- **Groups related items**: Organize navigation into collapsible sections
- **Flexible styling**: Customize appearance with `className` and `style` props

### Common Mistakes to Avoid

- **Don't mix MPV and SPV in the same group**: Keep navigation items (MPV) separate from view switchers (SPV). They serve different purposes.
- **Don't forget the `name` prop for SPV items**: All SPV items in a group need the same `name` prop to work together like radio buttons.
- **Don't skip icons when minimized**: Icons are essential when the sidebar is collapsed, as they're the only visual identifier.
- **Don't use both JSX children and widget-tree pattern simultaneously**: Pick one approach and stick with it for consistency.

### Related Components

- [SidebarItem](#sidebaritem) - Individual navigation or selection items
- [SidebarGroup](#sidebargroup) - Collapsible sections to organize items
- [SidebarHeader](#sidebarheader) - Top section with logo and title
- [SidebarFooter](#sidebarfooter) - Bottom section for secondary actions
- [Link](./link.md) - Used internally by MPV items for routing

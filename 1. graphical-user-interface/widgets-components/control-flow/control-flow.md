# Control Flow

Control flows allow you to...

- **Control Flow:**
  _ `<Show when={signal}>`: Conditional rendering. Supports `true` and `else` props. For one-off static conditions, you can use inline typescript to return the desired branch directly just like in React(but will not have reactivity).
  _ `<List each={signalOfArray} track="key" indexed={true}>`: Efficiently renders lists with reconciliation by handling all list rendering scenarios. Automatically handles static arrays and signals, eliminates need for `derivedExtract`, provides direct item access in templates. Use `track` for stable keys when data changes completely. Exposes `getItem()`, `remove()`, `clear()` methods. View functions receive raw item data directly for clean, readable syntax.
  _ `<Async future={promise}>`: Manages asynchronous operations (pending, resolved, rejected states). `async` components automatically get `fallback` and `catch` props.
  _ **Error Handling in Asynchronous Components:** Implement robust error handling within `async` components. Utilize the `catch` prop of `<Async>` components or direct `fallback`/`catch` props on async components, and `try...catch` blocks for network requests to gracefully manage and display errors to the user.
  _ `<Dynamic is={componentOrTag}>`: Renders a component or HTML tag that can change dynamically.
  _ `<Fn ctx={value} catch={errorHandler}>`: Executes a function that returns a render function, useful for complex logic with error boundaries. \* **List Management:** Use `List` component's exposed methods (`getItem()`, `remove()`, `clear()`) for imperative list manipulation when needed.

- **`$ref`:** Special prop to get a reference to a DOM element or component instance (as a signal or function). **Critical for hot reload in dev mode:** always use `$ref` for component references, not `createComponent()` return values.
- **`expose()`:** Allows child components to expose properties/methods to their parent via `$ref`.
- **`capture()`:** Captures the current rendering context, useful for running functions (e.g., `expose()`) after `await` calls in async components.
- **Importing:** All built-in components can be imported directly from package `InSpatial`
- **`createComponent(template, props?, ...children)`:** Creates component instances
  **`renderer.render(container, component, props, ...children)`:** Renders a component into a container, with optional props and children.
  **`dispose(instance)`:** Cleans up component resources
  **`getCurrentSelf()`:** Gets current component instance
  **`snapshot()`:** Creates context snapshots for async operations
- **JSX Children in Control Flow Components:** When using components like `<Show>` and `<List>`, ensure their render function children return either a _single root element_ or a `Fragment` (`<>...</>`) if rendering multiple sibling elements. This prevents unexpected rendering issues.
- `**Fn`\*\* provides a render closure; inside it, you can read the table header groups and row model.
- **`R.c(tag, props, ...children)`** creates elements; `R.text(string)` creates text nodes.



### Conditional Classes

```javascript
 // ✅ The InSpatial Way
<Button className="btn" class:active={isActive}>
```

#### Conditionally Rendering Computed/Reactive Values

##### `Show`

```jsx
// ❌  DON'T DO THIS: It won't react properly
<Button on:tap={() => useTheme.action.handleToggle()}>
  {$(() =>
    useTheme.mode.get() === "dark" ? <LightModeIcon /> : <DarkModeIcon />
  )}
</Button>

// ✅ DO THIS: Use Show for reactive conditional rendering
<Button on:tap={() => useTheme.action.handleToggle()}>
  <Show
    when={$(() => useTheme.mode.get() === "dark")}
    otherwise={() => <DarkModeIcon />}
  >
    {() => <LightModeIcon />}
  </Show>
</Button>
```

##### `Choose` - The InSpatial Switch Statement

```jsx
// ❌  DON'T DO THIS: It won't react
export function InputField({ variant }) {
  return (
    <>
      {(() => {
        switch (variant.get()) {
          case "emailfield":
            return <EmailField />;
          case "searchfield":
            return <SearchField />;
          default:
            return <TextField />;
        }
      })()}
    </>
  );
}

// ✅ DO THIS: Use Choose for multi-branch reactive logic
import { Choose } from "@inspatial/kit/control-flow";

export function InputField({ variant }) {
  return (
    <Choose
      cases={[
        {
          when: $(() => variant.get() === "emailfield"),
          children: () => <EmailField />,
        },
        {
          when: $(() => variant.get() === "searchfield"),
          children: () => <SearchField />,
        },
      ]}
      otherwise={() => <TextField />}
    />
  );
}

// Or without JSX tags:
export function InputField({ variant }) {
  return Choose({
    cases: [
      {
        when: $(() => variant.get() === "emailfield"),
        children: () => <EmailField />,
      },
      {
        when: $(() => variant.get() === "searchfield"),
        children: () => <SearchField />,
      },
    ],
    otherwise: () => <TextField />,
  });
}
```

#### Difference Between Show & Choose

- **Show** and **Choose** are both used to react to conditional values
- Use `<Show>` for **binary conditions** (this or that):
  - Example: Show loading spinner OR content
  - Example: Show dark mode icon OR light mode icon
- Use `<Choose>` for **multi-branch logic** (this, that, or another thing):
  - Example: Show different input types (email, search, text, password)
  - Example: Show different UI states (loading, error, success, empty)

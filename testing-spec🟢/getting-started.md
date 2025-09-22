# Testing & Specifications

So, you've built something amazing with InSpatial. Now it's time to make sure it works perfectly, every time. This is your complete guide to testing with InSpatial Test - from writing your first test to mastering advanced testing patterns. Think of it like having a safety net that catches problems before your users ever see them and sometimes before you even create a feature for your App, website, game or XR experience.

InSpatial Test is designed to work everywhere your code runs. Whether you're building for the web, mobile, desktop, spatial computing etc..., you write your tests once and they run everywhere.

> **Terminology:**
>
> - **Test**: A piece of code that checks if another piece of code works correctly
> - **Assertion**: A statement that something should be true (like "this should equal 5")
> - **Mock**: A fake version of something (like pretending to call an API without actually calling it)
> - **Spy**: Watching what happens when code runs (like counting how many times a function gets called)

## What Makes InSpatial Test Special

InSpatial Test is like having the best parts of Jest, Vitest, and Deno's testing tools all rolled into one. But here's the magic: it automatically detects what environment you're running in and adapts accordingly. Zero Configuration universal testing that work at runtime with any runtime, environment or platform.

> **Note:** If you're new to testing, don't worry about these names! Jest is Facebook's popular testing framework, Vitest is a fast modern testing tool, and Chai is an assertion library. InSpatial Test combines the best features from all of them, so you get a powerful testing experience without needing to learn multiple tools.

You get the familiar `expect()` syntax you know and love, plus powerful features like time manipulation, snapshot testing, comprehensive mocking capabilities etc... And because it's built for the InSpatial ecosystem, it understands spatial computing, 3D environments, and cross-platform development out of the box.

> **Note:** InSpatial Test works seamlessly with Deno/InZero, Node.js, & Bun. You don't need to configure anything - it just works.

## 💡Core Concepts

Quick links to essential topics in this guide:

- [Writing Tests](#writing-tests)
- [Organizing with Describe](#organizing-tests-with-describe)
- [Syntactic Sugar: createTest](#syntactic-sugar-createtest)
- [Setup & Teardown](#setup-and-teardown)
- [Assertions](#assertions-checking-if-things-work)
- [Testing Async Code](#testing-async-code)
- [Mocking & Spying](#mocking-and-spying)
- [Time Control](#time-testing)
- [Snapshot Testing](#snapshot-testing)
- [Test Modifiers](#test-modifiers)
- [Running Tests](#running-tests)
- [Coverage Reports](#coverage-reports)
- [Performance Testing](#performance-testing)

## Getting Started

### 🎯 Prerequisites

Before you start:

- Basic understanding of [JavaScript](https://web.dev/learn/javascript)
- Basic understanding [TypesScript](https://www.typescriptlang.org/docs/handbook/intro.html)

### 🎮 Usage

#### Basic Import

```typescript
// ✅ Recommended: Direct import from kit
import { test } from "@inspatial/kit/test";

// ❌ Avoid: Package-level import
import { test } from "@inspatial/kit";
```

<details>
<summary>📦 Installation for Framework Authors</summary>

If you're building a framework or library that needs to include testing & specifications:

```bash
deno install jsr:@in/test
```

##

```bash
npx jsr add @in/test
```

##

```bash
yarn dlx jsr add @in/test
```

##

```bash
pnpm dlx jsr add @in/test
```

##

```bash
bunx jsr add @in/test
```

##

```bash
vlt install jsr:@in/test
```

</details>

### Your First Test

Let's start with something simple. Create a file called `math.test.ts`:

```typescript
import { test, expect } from "@inspatial/kit/test";

// Test a simple addition function
function add(a: number, b: number): number {
  return a + b;
}

test("adding numbers should work", () => {
  expect(add(2, 3)).toBe(5);
  expect(add(-1, 1)).toBe(0);
  expect(add(0, 0)).toBe(0);
});
```

Run it:

```bash
# Deno
deno test

# Node.js
node --test

# Bun
bun test
```

That's it! You've written and run your first InSpatial test.

## Test File Naming

InSpatial Test automatically finds your test files using these patterns:

- `*.test.ts` (founders choice)
- `*.test.js`
- `*_test.ts`
- `*_test.js`

```typescript
// user.test.ts ✅
// api_test.ts ✅
// calculator.test.js ✅
// my.spec.ts ✅ // ⚠️ Experimental
```

## Writing Tests

### Function Style (Simple and Quick)

```typescript
import { test, expect } from "@inspatial/kit/test";

test("user can log in", () => {
  const user = { name: "Ben", loggedIn: false };
  user.loggedIn = true;
  expect(user.loggedIn).toBe(true);
});
```

### Object Style (More Control)

```typescript
import { test, expect } from "@inspatial/kit/test";

test({
  name: "user can log in with detailed config",
  fn: () => {
    const user = { name: "Emma", loggedIn: false };
    user.loggedIn = true;
    expect(user.loggedIn).toBe(true);
  },
  options: {
    // Only run this test
    only: false,
    // Skip this test
    skip: false,
    // Mark as todo
    todo: false,
    // Test permissions (Deno)
    permissions: { read: true, write: false },
    // Check resource cleanup
    sanitizeResources: true,
  },
});
```

## Organizing Tests with Describe

Think of `describe` like folders for organizing your tests:

```typescript
import { describe, test, expect } from "@inspatial/kit/test";

describe("User Authentication", () => {
  test("should allow valid login", () => {
    // Test valid login
  });

  test("should reject invalid password", () => {
    // Test invalid login
  });

  describe("Password Reset", () => {
    test("should send reset email", () => {
      // Test password reset
    });
  });
});
```

You can also use `it()` instead of `test()` - they're exactly the same:

```typescript
import { describe, it, expect } from "@inspatial/kit/test";

describe("Shopping Cart", () => {
  it("should add items correctly", () => {
    // Test adding items
  });

  it("should calculate total price", () => {
    // Test price calculation
  });
});
```

### Syntactic Sugar: createTest

> **Terminology:**
>
> - **Syntactic Sugar**: An easier or more convenient way to write code that achieves the same result as a more verbose or complex approach. It doesn't add new functionality, but makes code simpler and more readable.

For quick, scoped test creation with an automatic suite/prefix, use `createTest`:

```typescript
import { createTest, expect } from "@inspatial/kit/test";

// Prefix all test names with a suite label
const t = createTest("Authentication");

t("should login with valid credentials", async () => {
  const user = await login("user@example.com", "password");
  expect(user.isAuthenticated).toBe(true);
});

// Modifiers (if supported by underlying runner)
t.only?.("should refresh token", async () => {
  const token = await refreshToken();
  expect(token).toBeDefined();
});

t.skip?.("should logout", () => {
  // pending
});

// You can also pass options instead of a string
const adminTests = createTest({ suite: "Admin" });
adminTests("should access admin panel", () => {
  expect(canAccessAdmin()).toBe(true);
});
```

## Setup and Teardown

Sometimes you need to prepare things before tests run, or clean up afterwards:

```typescript
import {
  describe,
  test,
  beforeEach,
  afterEach,
  beforeAll,
  afterAll,
} from "@inspatial/kit/test";

describe("Database Tests", () => {
  let database;

  // Run once before all tests in this describe block
  beforeAll(() => {
    database = createTestDatabase();
  });

  // Run before each individual test
  beforeEach(() => {
    database.clear();
    database.seed();
  });

  // Run after each individual test
  afterEach(() => {
    database.cleanup();
  });

  // Run once after all tests in this describe block
  afterAll(() => {
    database.destroy();
  });

  test("should save user", () => {
    // Your test here
  });
});
```

## Assertions: Checking if Things Work

### Expect Style (Founders Choice)

InSpatial Test uses the same `expect()` syntax as Jest and Vitest:

```typescript
import { expect } from "@inspatial/kit/test";

// Basic equality
expect(2 + 2).toBe(4); // Strict equality (===)
expect({ name: "Ben" }).toEqual({ name: "Ben" }); // Deep equality

// Truthiness
expect(true).toBeTruthy();
expect(false).toBeFalsy();
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect("hello").toBeDefined();

// Numbers
expect(10).toBeGreaterThan(5);
expect(3).toBeLessThan(10);
expect(5).toBeGreaterThanOrEqual(5);
expect(Math.PI).toBeCloseTo(3.14, 2);

// Strings and Arrays
expect("hello world").toContain("world");
expect([1, 2, 3]).toContain(2);
expect("test@example.com").toMatch(/\w+@\w+\.\w+/);

// Objects
expect({ name: "Ben", age: 24 }).toHaveProperty("name");
expect({ name: "Ben", age: 24 }).toHaveProperty("age", 24);

// Types
expect("hello").toBeType("string");
expect(42).toBeType("number");
expect([]).toBeType("object");

// Advanced matchers
expect("ben@inspatial.io").toBeEmail();
expect("https://inspatial.io").toBeUrl();
expect([]).toBeEmpty();
expect([1, 2, 3]).toBeSorted();
```

### Assert Style (Alternative)

If you prefer a more direct approach:

```typescript
import {
  assert,
  assertEquals,
  assertNotEquals,
  assertArrayIncludes,
  assertStringIncludes,
} from "@inspatial/kit/test";

assert(true); // Basic assertion
assertEquals(2 + 2, 4); // Values are equal
assertNotEquals("hello", "world"); // Values are different
assertArrayIncludes([1, 2, 3], [2]); // Array contains items
assertStringIncludes("hello", "ell"); // String contains substring
```

## Testing Async Code

InSpatial Test handles promises and async functions naturally:

```typescript
import { test, expect } from "@inspatial/kit/test";

// Async function testing
test("fetch user data", async () => {
  const userData = await fetchUser(123);
  expect(userData.id).toBe(123);
  expect(userData.name).toBeDefined();
});

// Promise resolution
test("promise resolves correctly", async () => {
  await expect(Promise.resolve("success")).resolves.toBe("success");
});

// Promise rejection
test("promise rejects correctly", async () => {
  await expect(Promise.reject("error")).rejects.toBe("error");
});

// Testing functions that should throw
test("function throws error", () => {
  expect(() => {
    throw new Error("Something went wrong");
  }).toThrow("Something went wrong");
});
```

## Mocking and Spying

### Spying (Watching Function Calls)

Use `spy()` when you want to watch what happens without changing behavior:

```typescript
import { test, spy, assertSpyCalls, assertSpyCall } from "@inspatial/kit/test";

test("tracking function calls", () => {
  const calculator = {
    add(a: number, b: number): number {
      return a + b;
    }
  };

  // Create a spy to watch the add method
  using addSpy = spy(calculator, "add");

  // Use the function normally
  const result = calculator.add(2, 3);
  expect(result).toBe(5);

  // Check that it was called
  assertSpyCalls(addSpy, 1);
  assertSpyCall(addSpy, 0, {
    args: [2, 3],
    returned: 5
  });
});
```

### Stubbing (Replacing Function Behavior)

Use `stub()` when you want to replace a function with controlled behavior:

```typescript
import { test, stub, returnsNext, assertSpyCalls } from "@inspatial/kit/test";

test("controlling function behavior", () => {
  const api = {
    fetchData(): string {
      return Math.random().toString(); // Unpredictable!
    }
  };

  // Replace with predictable behavior
  using fetchStub = stub(api, "fetchData", returnsNext(["data1", "data2"]));

  expect(api.fetchData()).toBe("data1");
  expect(api.fetchData()).toBe("data2");

  assertSpyCalls(fetchStub, 2);
});
```

### Mock Functions

Create standalone mock functions:

```typescript
import { test, mockFn, getMockCalls } from "@inspatial/kit/test";

test("mock function behavior", () => {
  const mockCallback = mockFn((x: number) => x * 2);

  mockCallback(5);
  mockCallback(10);

  const calls = getMockCalls(mockCallback);
  expect(calls).toHaveLength(2);
  expect(calls[0].args).toEqual([5]);
  expect(calls[1].args).toEqual([10]);
});
```

## Time Testing

Control time in your tests with `FakeTime`:

```typescript
import { test, FakeTime, spy, assertSpyCalls } from "@inspatial/kit/test";

test("testing timeouts and intervals", () => {
  using time = new FakeTime();

  const callback = spy();

  // Set up a timeout
  setTimeout(callback, 1000);

  // Initially, callback hasn't been called
  assertSpyCalls(callback, 0);

  // Advance time by 1 second
  time.tick(1000);

  // Now callback should have been called
  assertSpyCalls(callback, 1);
});

test("testing intervals", () => {
  using time = new FakeTime();

  const callback = spy();
  const intervalId = setInterval(callback, 500);

  // Advance time by 2 seconds (4 intervals)
  time.tick(2000);

  assertSpyCalls(callback, 4);

  clearInterval(intervalId);
});
```

## Snapshot Testing

Snapshot testing is like taking a "photo" of your data and comparing it later:

```typescript
import { test, assertSnapshot } from "@inspatial/kit/test";

test("user profile snapshot", async (t) => {
  const user = {
    name: "Charlotte",
    age: 34,
    email: "charlotte@inspatial.io",
    preferences: {
      theme: "dark",
      notifications: true,
    },
  };

  // This creates a snapshot file the first time
  // and compares against it on subsequent runs
  await assertSnapshot(t, user);
});
```

To update snapshots when you make intentional changes:

```bash
deno test -- --update
# or
deno test -- -u
```

## Test Modifiers

Control which tests run:

```typescript
import { test, describe } from "@inspatial/kit/test";

// Only run this test (skip all others without .only)
test.only("critical test", () => {
  // This will run
});

// Skip this test
test.skip("broken test", () => {
  // This won't run
});

// Mark as todo (reminder to implement later)
test.todo("implement user deletion");

// Works with describe too
describe.only("Critical Features", () => {
  test("important test", () => {
    // This will run
  });
});
```

## Running Tests

### Basic Commands

```bash
# Run all tests
deno test
node --test
bun test

# Run specific file
deno test user.test.ts
node --test user.test.ts
bun test user.test.ts

# Run tests in a directory
deno test tests/
node --test tests/
bun test tests/
```

### Watch Mode (Auto-rerun on changes)

```bash
deno test --watch
node --test --watch
bun test --watch
```

### Coverage Reports

```bash
# Deno
deno test --coverage=coverage
deno coverage coverage --html

# Node.js
node --test --experimental-test-coverage

# Bun
bun test --coverage
```

## Advanced Features

### File System Mocking

Mock file operations for testing:

```typescript
import { test, setupFsMocks } from "@inspatial/kit/test";

test("file operations", () => {
  const { fs } = setupFsMocks({
    virtualFiles: {
      "/test.txt": "Hello, World!",
    },
  });

  // Your code that reads files will now use the virtual file system
  // fs.readTextFile("/test.txt") returns "Hello, World!"
});
```

### Performance Testing

```typescript
import { test, expect, performance } from "@inspatial/kit/test";

test("performance test", () => {
  const start = performance.now();

  // Your code here
  const result = expensiveOperation();

  const end = performance.now();
  const duration = end - start;

  expect(duration).toBeLessThan(1000); // Should complete in under 1 second
  expect(result).toBeDefined();
});
```

## Best Practices

### 1. Write Descriptive Test Names

```typescript
// ❌ Bad
test("user test", () => {});

// ✅ Good
test("should create user with valid email and password", () => {});
```

### 2. Arrange, Act, Assert Pattern

```typescript
test("should calculate total price with tax", () => {
  // Arrange - Set up test data
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 1 },
  ];
  const taxRate = 0.1;

  // Act - Execute the code being tested
  const total = calculateTotal(items, taxRate);

  // Assert - Check the result
  expect(total).toBe(27.5); // (20 + 5) * 1.1
});
```

### 3. Test One Thing at a Time

```typescript
// ❌ Bad - Testing multiple things
test("user operations", () => {
  const user = createUser("Ben");
  expect(user.name).toBe("Ben");

  user.updateEmail("ben@inspatial.io");
  expect(user.email).toBe("ben@inspatial.io");

  user.delete();
  expect(user.isDeleted).toBe(true);
});

// ✅ Good - Separate tests
describe("User", () => {
  test("should create user with correct name", () => {
    const user = createUser("Ben");
    expect(user.name).toBe("Ben");
  });

  test("should update email correctly", () => {
    const user = createUser("Ben");
    user.updateEmail("ben@inspatial.io");
    expect(user.email).toBe("ben@inspatial.io");
  });

  test("should mark user as deleted when deleted", () => {
    const user = createUser("Ben");
    user.delete();
    expect(user.isDeleted).toBe(true);
  });
});
```

### 4. Use Meaningful Test Data

```typescript
// ❌ Bad
test("user creation", () => {
  const user = createUser("a", "b", 1);
  expect(user.name).toBe("a");
});

// ✅ Good
test("should create user with provided details", () => {
  const user = createUser("Ben Emma", "ben@inspatial.io", 24);
  expect(user.name).toBe("Ben Emma");
  expect(user.email).toBe("ben@inspatial.io");
  expect(user.age).toBe(24);
});
```

### 5. Clean Up Resources

```typescript
test("database operations", () => {
  using connection = createDatabaseConnection();
  using tempFile = createTempFile();

  // Test code here

  // Resources are automatically cleaned up due to 'using'
});
```

## Common Patterns

### Testing Error Conditions

```typescript
test("should handle invalid input gracefully", () => {
  expect(() => {
    processUserData(null);
  }).toThrow("User data cannot be null");

  expect(() => {
    processUserData({ name: "" });
  }).toThrow("Name cannot be empty");
});
```

### Testing API Calls

```typescript
test("should fetch user data from API", async () => {
  const api = {
    fetchUser: stub(
      returnsNext([{ id: 1, name: "Ben", email: "ben@inspatial.io" }])
    ),
  };

  const user = await getUserById(1);

  expect(user.name).toBe("Ben");
  expect(user.email).toBe("ben@inspatial.io");
});
```

### Testing Event Handlers

```typescript
test("should handle button click events", () => {
  const onClick = spy();
  const button = createButton({ onClick });

  button.click();

  assertSpyCalls(onClick, 1);
});
```

## Troubleshooting

### Tests Not Running

1. Check file naming: Use `.test.ts` or `_test.ts`
2. Check imports: Make sure you're importing from `@inspatial/kit/test`
3. Check syntax: Ensure your test functions are properly structured

### Assertion Failures

```typescript
// Get detailed error messages
test("debugging assertions", () => {
  const actual = { name: "Ben", age: 24 };
  const expected = { name: "Ben", age: 25 };

  // This will show a detailed diff of what's different
  expect(actual).toEqual(expected);
});
```

### Async Test Issues

```typescript
// ❌ Forgot to await
test("async test", () => {
  fetchData().then((data) => {
    expect(data).toBeDefined(); // This might not run!
  });
});

// ✅ Properly awaited
test("async test", async () => {
  const data = await fetchData();
  expect(data).toBeDefined();
});
```

## What's Next?

Now that you know the basics, you might want to explore:

- [Migration Guide](./migration.md) - Moving from Jest, Vitest, or Chai
- [Advanced Mocking](./advanced-mocking.md) - Complex mocking scenarios
- [CI/CD Integration](./ci-cd.md) - Running tests in continuous integration
<!-- - [Spatial Testing](./spatial-testing.md) - Testing 3D and AR/VR applications -->

Remember, good tests make your code more reliable, easier to refactor, and help you catch bugs before they reach production. Start simple, be consistent, and gradually add more sophisticated testing patterns as your application grows.

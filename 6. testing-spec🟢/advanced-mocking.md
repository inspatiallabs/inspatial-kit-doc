## Advanced Mocking

#### Master the art of creating realistic test doubles for complex scenarios

> Start with the [Core Concepts](./getting-started.md#core-concepts) if you're new to InSpatial Test, then come back here for advanced mocking techniques.

You've learned the basics of testing and simple mocking. Now it's time to dive deeper into the world of advanced mocking techniques. Think of this as upgrading from using cardboard cutouts in a play to having professional actors who can improvise and respond to any situation. Advanced mocking lets you simulate complex real-world scenarios, control unpredictable dependencies, and test edge cases that would be impossible to reproduce otherwise.

Whether you're testing API integrations, database operations, file systems, or complex business logic, advanced mocking gives you the power to create controlled, predictable test environments. You'll learn to mock entire modules, create dynamic responses, simulate network failures, and even mock time itself.

> **Terminology:**
>
> - **Test Double**: A generic term for any fake object used in testing (mocks, stubs, spies, fakes)
> - **Stub**: Replaces a function with predetermined behavior
> - **Spy**: Watches and records function calls without changing behavior
> - **Mock**: A stub that also verifies how it was called
> - **Fake**: A working implementation with shortcuts (like an in-memory database)

## Understanding Test Doubles

Before diving into advanced techniques, let's understand the different types of test doubles and when to use each:

### Spies: The Silent Observers

Spies watch what happens without interfering. Use them when you want to verify behavior but keep the original functionality:

```typescript
import { test, spy, assertSpyCalls, assertSpyCall } from "@inspatial/kit/test";

test("spy tracks calls without changing behavior", () => {
  const logger = {
    log(message: string) {
      console.log(`[LOG]: ${message}`);
    },
    error(message: string) {
      console.error(`[ERROR]: ${message}`);
    }
  };

  // Spy on the log method
  using logSpy = spy(logger, "log");
  using errorSpy = spy(logger, "error");

  // Use the logger normally
  logger.log("User logged in");
  logger.error("Database connection failed");

  // Verify the calls
  assertSpyCalls(logSpy, 1);
  assertSpyCall(logSpy, 0, { args: ["User logged in"] });
  
  assertSpyCalls(errorSpy, 1);
  assertSpyCall(errorSpy, 0, { args: ["Database connection failed"] });
});
```

### Stubs: The Controlled Actors

Stubs replace functions with controlled behavior. Use them when you need predictable responses:

```typescript
import { test, stub, returnsNext, resolvesNext } from "@inspatial/kit/test";

test("stub provides controlled responses", () => {
  const weatherService = {
    async getTemperature(city: string): Promise<number> {
      // This would normally make an API call
      throw new Error("Should not call real API in tests");
    }
  };

  // Replace with controlled behavior
  using tempStub = stub(
    weatherService, 
    "getTemperature", 
    resolvesNext([22, 25, 18]) // Returns these values in sequence
  );

  // Now we get predictable results
  expect(await weatherService.getTemperature("London")).toBe(22);
  expect(await weatherService.getTemperature("Paris")).toBe(25);
  expect(await weatherService.getTemperature("Berlin")).toBe(18);
});
```

### Mock Functions: The Versatile Performers

Mock functions are standalone test doubles that can be configured for any behavior:

```typescript
import { test, mockFn, getMockCalls } from "@inspatial/kit/test";

test("mock functions provide flexible behavior", () => {
  // Create a mock that simulates a callback
  const onSuccess = mockFn((data: any) => {
    return `Processed: ${data.name}`;
  });

  const onError = mockFn((error: Error) => {
    return `Failed: ${error.message}`;
  });

  // Use the mocks
  const result1 = onSuccess({ name: "Ben" });
  const result2 = onError(new Error("Network timeout"));

  // Verify behavior
  expect(result1).toBe("Processed: Ben");
  expect(result2).toBe("Failed: Network timeout");

  const successCalls = getMockCalls(onSuccess);
  expect(successCalls).toHaveLength(1);
  expect(successCalls[0].args).toEqual([{ name: "Ben" }]);
});
```

## Advanced Stubbing Techniques

### Sequential Responses

Create stubs that return different values on each call:

```typescript
import { test, stub, returnsNext } from "@inspatial/kit/test";

test("sequential responses simulate changing conditions", () => {
  const networkService = {
    checkConnection(): boolean {
      return navigator.onLine; // Real implementation
    }
  };

  // Simulate network going up and down
  using connectionStub = stub(
    networkService,
    "checkConnection",
    returnsNext([true, true, false, false, true])
  );

  expect(networkService.checkConnection()).toBe(true);  // First call
  expect(networkService.checkConnection()).toBe(true);  // Second call
  expect(networkService.checkConnection()).toBe(false); // Third call (network down)
  expect(networkService.checkConnection()).toBe(false); // Fourth call (still down)
  expect(networkService.checkConnection()).toBe(true);  // Fifth call (back up)
});
```

### Async Sequential Responses

Handle promises and async operations:

```typescript
import { test, stub, resolvesNext } from "@inspatial/kit/test";

test("async sequential responses for API calls", async () => {
  const apiService = {
    async fetchUserData(id: number): Promise<any> {
      throw new Error("Should not call real API");
    }
  };

  // Simulate different API responses
  using fetchStub = stub(
    apiService,
    "fetchUserData",
    resolvesNext([
      { id: 1, name: "Ben", status: "active" },
      { id: 2, name: "Emma", status: "inactive" },
      new Error("User not found"), // This will be thrown
      { id: 4, name: "Charlotte", status: "active" }
    ])
  );

  // First two calls succeed
  const user1 = await apiService.fetchUserData(1);
  expect(user1.name).toBe("Ben");

  const user2 = await apiService.fetchUserData(2);
  expect(user2.name).toBe("Emma");

  // Third call throws an error
  await expect(apiService.fetchUserData(3)).rejects.toThrow("User not found");

  // Fourth call succeeds again
  const user4 = await apiService.fetchUserData(4);
  expect(user4.name).toBe("Charlotte");
});
```

### Dynamic Responses Based on Arguments

Create stubs that respond differently based on input:

```typescript
import { test, stub } from "@inspatial/kit/test";

test("dynamic responses based on arguments", () => {
  const calculator = {
    divide(a: number, b: number): number {
      return a / b;
    }
  };

  // Create a stub with custom logic
  using divideStub = stub(calculator, "divide", (a: number, b: number) => {
    if (b === 0) {
      throw new Error("Division by zero");
    }
    if (a < 0 || b < 0) {
      throw new Error("Negative numbers not supported");
    }
    return a / b; // Normal calculation for positive numbers
  });

  expect(calculator.divide(10, 2)).toBe(5);
  expect(calculator.divide(15, 3)).toBe(5);
  
  expect(() => calculator.divide(10, 0)).toThrow("Division by zero");
  expect(() => calculator.divide(-5, 2)).toThrow("Negative numbers not supported");
});
```

## Module-Level Mocking

### Mocking Entire Modules

Sometimes you need to mock entire modules or libraries:

```typescript
import { test, stub, mockFn } from "@inspatial/kit/test";

// Mock a hypothetical HTTP client module
const mockHttpClient = {
  get: mockFn(),
  post: mockFn(),
  put: mockFn(),
  delete: mockFn()
};

test("mock entire HTTP client module", async () => {
  // Configure the mock HTTP client
  mockHttpClient.get.mockImplementation(async (url: string) => {
    if (url.includes("/users/1")) {
      return { data: { id: 1, name: "Ben" }, status: 200 };
    }
    if (url.includes("/users/2")) {
      return { data: { id: 2, name: "Emma" }, status: 200 };
    }
    throw new Error("Not found");
  });

  mockHttpClient.post.mockImplementation(async (url: string, data: any) => {
    return { data: { ...data, id: Date.now() }, status: 201 };
  });

  // Test code that uses the HTTP client
  const userService = {
    async getUser(id: number) {
      const response = await mockHttpClient.get(`/users/${id}`);
      return response.data;
    },
    
    async createUser(userData: any) {
      const response = await mockHttpClient.post("/users", userData);
      return response.data;
    }
  };

  const user = await userService.getUser(1);
  expect(user.name).toBe("Ben");

  const newUser = await userService.createUser({ name: "Mike", age: 36 });
  expect(newUser.name).toBe("Mike");
  expect(newUser.id).toBeDefined();
});
```

### Mocking External Dependencies

Mock external libraries and services:

```typescript
import { test, stub, mockFn } from "@inspatial/kit/test";

test("mock external payment service", async () => {
  // Mock a payment service
  const mockPaymentService = {
    processPayment: mockFn(),
    refundPayment: mockFn(),
    getPaymentStatus: mockFn()
  };

  // Configure payment processing behavior
  mockPaymentService.processPayment.mockImplementation(async (amount: number, cardToken: string) => {
    if (amount <= 0) {
      throw new Error("Invalid amount");
    }
    if (cardToken === "invalid_token") {
      throw new Error("Invalid card");
    }
    if (amount > 10000) {
      throw new Error("Amount too large");
    }
    
    return {
      transactionId: `txn_${Date.now()}`,
      status: "completed",
      amount
    };
  });

  // Test the payment flow
  const paymentProcessor = {
    async processOrder(order: any) {
      try {
        const payment = await mockPaymentService.processPayment(
          order.total, 
          order.cardToken
        );
        return { success: true, transactionId: payment.transactionId };
      } catch (error) {
        return { success: false, error: error.message };
      }
    }
  };

  // Test successful payment
  const successResult = await paymentProcessor.processOrder({
    total: 99.99,
    cardToken: "valid_token_123"
  });
  expect(successResult.success).toBe(true);
  expect(successResult.transactionId).toMatch(/^txn_/);

  // Test invalid amount
  const invalidAmountResult = await paymentProcessor.processOrder({
    total: -10,
    cardToken: "valid_token_123"
  });
  expect(invalidAmountResult.success).toBe(false);
  expect(invalidAmountResult.error).toBe("Invalid amount");
});
```

## File System and I/O Mocking

### Virtual File Systems

Create virtual file systems for testing file operations:

```typescript
import { test, setupFsMocks } from "@inspatial/kit/test";

test("virtual file system for file operations", async () => {
  const { fs } = setupFsMocks({
    virtualFiles: {
      "/config/app.json": JSON.stringify({
        name: "MyApp",
        version: "1.0.0",
        debug: true
      }),
      "/data/users.csv": "id,name,email\n1,Ben,ben@inspatial.io\n2,Emma,emma@inspatial.io",
      "/logs/app.log": "2024-01-01 10:00:00 - App started\n2024-01-01 10:01:00 - User logged in"
    }
  });

  // Test configuration loading
  const configService = {
    async loadConfig() {
      const configText = await fs.readTextFile("/config/app.json");
      return JSON.parse(configText);
    }
  };

  const config = await configService.loadConfig();
  expect(config.name).toBe("MyApp");
  expect(config.debug).toBe(true);

  // Test CSV parsing
  const dataService = {
    async loadUsers() {
      const csvText = await fs.readTextFile("/data/users.csv");
      const lines = csvText.split("\n");
      const headers = lines[0].split(",");
      
      return lines.slice(1).map(line => {
        const values = line.split(",");
        return headers.reduce((obj, header, index) => {
          obj[header] = values[index];
          return obj;
        }, {} as any);
      });
    }
  };

  const users = await dataService.loadUsers();
  expect(users).toHaveLength(2);
  expect(users[0].name).toBe("Ben");
  expect(users[1].email).toBe("emma@inspatial.io");
});
```

### Mocking File Operations with Errors

Simulate file system errors and edge cases:

```typescript
import { test, setupFsMocks } from "@inspatial/kit/test";

test("simulate file system errors", async () => {
  const { fs } = setupFsMocks({
    virtualFiles: {
      "/readonly/config.json": "{}",
    },
    mockExists: async (path: string) => {
      if (path.includes("nonexistent")) return false;
      return true;
    }
  });

  // Override specific methods to simulate errors
  const originalReadTextFile = fs.readTextFile;
  fs.readTextFile = async (path: string) => {
    if (path.includes("permission-denied")) {
      throw new Error("Permission denied");
    }
    if (path.includes("corrupted")) {
      throw new Error("File corrupted");
    }
    return originalReadTextFile(path);
  };

  const fileService = {
    async safeReadFile(path: string) {
      try {
        const exists = await fs.exists(path);
        if (!exists) {
          return { success: false, error: "File not found" };
        }
        
        const content = await fs.readTextFile(path);
        return { success: true, content };
      } catch (error) {
        return { success: false, error: error.message };
      }
    }
  };

  // Test successful read
  const successResult = await fileService.safeReadFile("/readonly/config.json");
  expect(successResult.success).toBe(true);

  // Test file not found
  const notFoundResult = await fileService.safeReadFile("/nonexistent/file.txt");
  expect(notFoundResult.success).toBe(false);
  expect(notFoundResult.error).toBe("File not found");

  // Test permission error
  const permissionResult = await fileService.safeReadFile("/permission-denied/file.txt");
  expect(permissionResult.success).toBe(false);
  expect(permissionResult.error).toBe("Permission denied");
});
```

## Time and Async Mocking

### Advanced Time Control

Control time for complex scenarios:

```typescript
import { test, FakeTime, spy, assertSpyCalls } from "@inspatial/kit/test";

test("complex time-based scenarios", () => {
  using time = new FakeTime(new Date("2024-01-01T00:00:00Z"));
  
  const eventLogger = {
    events: [] as Array<{ timestamp: Date; event: string }>,
    log(event: string) {
      this.events.push({ timestamp: new Date(), event });
    }
  };

  // Set up multiple timers
  const dailyBackup = setInterval(() => {
    eventLogger.log("Daily backup completed");
  }, 24 * 60 * 60 * 1000); // 24 hours

  const hourlyCheck = setInterval(() => {
    eventLogger.log("Hourly health check");
  }, 60 * 60 * 1000); // 1 hour

  const minutelyPing = setInterval(() => {
    eventLogger.log("Minute ping");
  }, 60 * 1000); // 1 minute

  // Advance time by 25 hours
  time.tick(25 * 60 * 60 * 1000);

  // Check what events were logged
  const backupEvents = eventLogger.events.filter(e => e.event.includes("backup"));
  const healthEvents = eventLogger.events.filter(e => e.event.includes("health"));
  const pingEvents = eventLogger.events.filter(e => e.event.includes("ping"));

  expect(backupEvents).toHaveLength(1); // 1 daily backup
  expect(healthEvents).toHaveLength(25); // 25 hourly checks
  expect(pingEvents).toHaveLength(25 * 60); // 25 hours * 60 minutes

  // Clean up
  clearInterval(dailyBackup);
  clearInterval(hourlyCheck);
  clearInterval(minutelyPing);
});
```

### Mocking Async Operations with Delays

Simulate real-world async delays and race conditions:

```typescript
import { test, FakeTime, mockFn } from "@inspatial/kit/test";

test("simulate async operations with realistic delays", async () => {
  using time = new FakeTime();
  
  const mockDatabase = {
    query: mockFn(),
    connect: mockFn(),
    disconnect: mockFn()
  };

  // Configure database operations with delays
  mockDatabase.connect.mockImplementation(async () => {
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1 second delay
    return { connected: true };
  });

  mockDatabase.query.mockImplementation(async (sql: string) => {
    const delay = sql.includes("SLOW") ? 5000 : 100; // Slow queries take 5 seconds
    await new Promise(resolve => setTimeout(resolve, delay));
    
    if (sql.includes("SELECT")) {
      return { rows: [{ id: 1, name: "Ben" }] };
    }
    return { affected: 1 };
  });

  // Test database service with timeouts
  const databaseService = {
    async queryWithTimeout(sql: string, timeoutMs: number = 3000) {
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error("Query timeout")), timeoutMs);
      });

      const queryPromise = mockDatabase.query(sql);
      
      return Promise.race([queryPromise, timeoutPromise]);
    }
  };

  // Start a slow query
  const slowQueryPromise = databaseService.queryWithTimeout("SELECT * FROM SLOW_TABLE", 3000);
  
  // Advance time to just before timeout
  time.tick(2999);
  
  // Query should still be running
  let queryCompleted = false;
  slowQueryPromise.catch(() => { queryCompleted = true; });
  
  // Advance past timeout
  time.tick(2);
  
  // Now it should timeout
  await expect(slowQueryPromise).rejects.toThrow("Query timeout");
});
```

## Network and API Mocking

### Simulating Network Conditions

Mock different network scenarios:

```typescript
import { test, stub, mockFn } from "@inspatial/kit/test";

test("simulate various network conditions", async () => {
  const mockFetch = mockFn();
  
  // Replace global fetch
  const originalFetch = globalThis.fetch;
  globalThis.fetch = mockFetch;

  // Configure different network scenarios
  mockFetch.mockImplementation(async (url: string, options?: any) => {
    // Simulate slow network
    if (url.includes("slow-endpoint")) {
      await new Promise(resolve => setTimeout(resolve, 5000));
      return new Response(JSON.stringify({ data: "slow response" }), { status: 200 });
    }
    
    // Simulate network error
    if (url.includes("network-error")) {
      throw new Error("Network error");
    }
    
    // Simulate server error
    if (url.includes("server-error")) {
      return new Response("Internal Server Error", { status: 500 });
    }
    
    // Simulate rate limiting
    if (url.includes("rate-limited")) {
      return new Response("Too Many Requests", { 
        status: 429,
        headers: { "Retry-After": "60" }
      });
    }
    
    // Default successful response
    return new Response(JSON.stringify({ data: "success" }), { status: 200 });
  });

  const apiClient = {
    async get(url: string, retries: number = 0): Promise<any> {
      try {
        const response = await fetch(url);
        
        if (response.status === 429 && retries > 0) {
          const retryAfter = parseInt(response.headers.get("Retry-After") || "1");
          await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
          return this.get(url, retries - 1);
        }
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return response.json();
      } catch (error) {
        if (retries > 0) {
          await new Promise(resolve => setTimeout(resolve, 1000));
          return this.get(url, retries - 1);
        }
        throw error;
      }
    }
  };

  // Test successful request
  const successResult = await apiClient.get("/api/data");
  expect(successResult.data).toBe("success");

  // Test network error with retry
  const networkErrorResult = await apiClient.get("/api/network-error", 2);
  // Should eventually succeed after retries (in a real scenario)

  // Test rate limiting with retry
  const rateLimitResult = await apiClient.get("/api/rate-limited", 1);
  // Should handle rate limiting gracefully

  // Restore original fetch
  globalThis.fetch = originalFetch;
});
```

### GraphQL API Mocking

Mock GraphQL APIs with complex queries:

```typescript
import { test, mockFn } from "@inspatial/kit/test";

test("mock GraphQL API responses", async () => {
  const mockGraphQLClient = {
    query: mockFn(),
    mutate: mockFn()
  };

  // Configure GraphQL responses
  mockGraphQLClient.query.mockImplementation(async (query: string, variables?: any) => {
    if (query.includes("getUser")) {
      const userId = variables?.id;
      if (userId === 1) {
        return {
          data: {
            user: {
              id: 1,
              name: "Ben Emma",
              email: "ben@inspatial.io",
              posts: [
                { id: 1, title: "First Post", content: "Hello World" },
                { id: 2, title: "Second Post", content: "GraphQL is awesome" }
              ]
            }
          }
        };
      }
      return { data: { user: null } };
    }
    
    if (query.includes("getAllUsers")) {
      return {
        data: {
          users: [
            { id: 1, name: "Ben Emma", email: "ben@inspatial.io" },
            { id: 2, name: "Michael Stephen Anderson", email: "mike@inspatial.io" }
          ]
        }
      };
    }
    
    throw new Error("Unknown query");
  });

  mockGraphQLClient.mutate.mockImplementation(async (mutation: string, variables?: any) => {
    if (mutation.includes("createUser")) {
      return {
        data: {
          createUser: {
            id: Date.now(),
            ...variables.input,
            createdAt: new Date().toISOString()
          }
        }
      };
    }
    
    throw new Error("Unknown mutation");
  });

  // Test GraphQL service
  const graphqlService = {
    async getUser(id: number) {
      const result = await mockGraphQLClient.query(`
        query GetUser($id: ID!) {
          user(id: $id) {
            id name email
            posts { id title content }
          }
        }
      `, { id });
      
      return result.data.user;
    },
    
    async createUser(userData: any) {
      const result = await mockGraphQLClient.mutate(`
        mutation CreateUser($input: UserInput!) {
          createUser(input: $input) {
            id name email createdAt
          }
        }
      `, { input: userData });
      
      return result.data.createUser;
    }
  };

  // Test user query
  const user = await graphqlService.getUser(1);
  expect(user.name).toBe("Ben Emma");
  expect(user.posts).toHaveLength(2);

  // Test user creation
  const newUser = await graphqlService.createUser({
    name: "Charlotte Rhodes",
    email: "charlotte@inspatial.io"
  });
  expect(newUser.name).toBe("Charlotte Rhodes");
  expect(newUser.id).toBeDefined();
  expect(newUser.createdAt).toBeDefined();
});
```

## State and Context Mocking

### Mocking Application State

Mock complex application state for testing:

```typescript
import { test, mockFn, spy } from "@inspatial/kit/test";

test("mock application state management", () => {
  // Mock state store
  const mockStore = {
    state: {
      user: null,
      cart: { items: [], total: 0 },
      ui: { loading: false, error: null }
    },
    
    dispatch: mockFn(),
    subscribe: mockFn(),
    getState: mockFn()
  };

  // Configure store behavior
  mockStore.getState.mockImplementation(() => mockStore.state);
  
  mockStore.dispatch.mockImplementation((action: any) => {
    switch (action.type) {
      case "SET_USER":
        mockStore.state.user = action.payload;
        break;
      case "ADD_TO_CART":
        mockStore.state.cart.items.push(action.payload);
        mockStore.state.cart.total += action.payload.price;
        break;
      case "SET_LOADING":
        mockStore.state.ui.loading = action.payload;
        break;
      case "SET_ERROR":
        mockStore.state.ui.error = action.payload;
        break;
    }
  });

  // Test component that uses the store
  const shoppingComponent = {
    store: mockStore,
    
    login(user: any) {
      this.store.dispatch({ type: "SET_USER", payload: user });
    },
    
    addToCart(item: any) {
      if (!this.store.getState().user) {
        this.store.dispatch({ 
          type: "SET_ERROR", 
          payload: "Must be logged in to add items" 
        });
        return;
      }
      
      this.store.dispatch({ type: "ADD_TO_CART", payload: item });
    },
    
    getCurrentUser() {
      return this.store.getState().user;
    },
    
    getCartTotal() {
      return this.store.getState().cart.total;
    }
  };

  // Test login
  shoppingComponent.login({ id: 1, name: "Ben" });
  expect(shoppingComponent.getCurrentUser().name).toBe("Ben");

  // Test adding to cart when logged in
  shoppingComponent.addToCart({ id: 1, name: "Widget", price: 29.99 });
  expect(shoppingComponent.getCartTotal()).toBe(29.99);

  // Test adding to cart when not logged in
  mockStore.state.user = null; // Simulate logout
  shoppingComponent.addToCart({ id: 2, name: "Gadget", price: 19.99 });
  expect(mockStore.state.ui.error).toBe("Must be logged in to add items");
});
```

## Testing Error Scenarios

### Simulating Complex Error Conditions

Test how your code handles various error scenarios:

```typescript
import { test, stub, mockFn } from "@inspatial/kit/test";

test("simulate complex error scenarios", async () => {
  const mockServices = {
    database: {
      connect: mockFn(),
      query: mockFn(),
      disconnect: mockFn()
    },
    cache: {
      get: mockFn(),
      set: mockFn(),
      delete: mockFn()
    },
    logger: {
      error: mockFn(),
      warn: mockFn(),
      info: mockFn()
    }
  };

  // Configure error scenarios
  let connectionAttempts = 0;
  mockServices.database.connect.mockImplementation(async () => {
    connectionAttempts++;
    if (connectionAttempts <= 2) {
      throw new Error("Connection timeout");
    }
    return { connected: true };
  });

  mockServices.database.query.mockImplementation(async (sql: string) => {
    if (sql.includes("DEADLOCK")) {
      throw new Error("Deadlock detected");
    }
    if (sql.includes("CONSTRAINT")) {
      throw new Error("Constraint violation");
    }
    return { rows: [] };
  });

  mockServices.cache.get.mockImplementation(async (key: string) => {
    if (key.includes("expired")) {
      return null; // Cache miss
    }
    throw new Error("Cache service unavailable");
  });

  // Test resilient service
  const resilientService = {
    async getUserWithRetry(userId: number, maxRetries: number = 3) {
      let lastError;
      
      for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
          // Try cache first
          try {
            const cached = await mockServices.cache.get(`user:${userId}`);
            if (cached) {
              mockServices.logger.info(`Cache hit for user ${userId}`);
              return cached;
            }
          } catch (cacheError) {
            mockServices.logger.warn(`Cache unavailable: ${cacheError.message}`);
          }
          
          // Connect to database
          await mockServices.database.connect();
          
          // Query database
          const result = await mockServices.database.query(
            `SELECT * FROM users WHERE id = ${userId}`
          );
          
          // Cache the result
          try {
            await mockServices.cache.set(`user:${userId}`, result.rows[0]);
          } catch (cacheError) {
            mockServices.logger.warn(`Failed to cache result: ${cacheError.message}`);
          }
          
          return result.rows[0];
          
        } catch (error) {
          lastError = error;
          mockServices.logger.error(`Attempt ${attempt} failed: ${error.message}`);
          
          if (attempt < maxRetries) {
            // Wait before retry (exponential backoff)
            await new Promise(resolve => 
              setTimeout(resolve, Math.pow(2, attempt) * 1000)
            );
          }
        }
      }
      
      throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
    }
  };

  // Test successful retry scenario
  const user = await resilientService.getUserWithRetry(1);
  expect(user).toBeDefined();
  
  // Verify retry behavior
  expect(mockServices.database.connect).toHaveBeenCalledTimes(3); // 2 failures + 1 success
  expect(mockServices.logger.error).toHaveBeenCalledTimes(2); // 2 failed attempts
  expect(mockServices.logger.warn).toHaveBeenCalled(); // Cache warnings
});
```

## Best Practices for Advanced Mocking

### 1. Keep Mocks Simple and Focused

```typescript
// ❌ Bad - Overly complex mock
const complexMock = mockFn((a, b, c, d, e) => {
  if (a > 0 && b < 10 && c === "test" && d.length > 5 && e.includes("x")) {
    return { status: "success", data: processComplexLogic(a, b, c, d, e) };
  }
  // ... 50 more lines of complex logic
});

// ✅ Good - Simple, focused mock
const simpleMock = mockFn((input: UserInput) => {
  if (input.isValid) {
    return { success: true, userId: 123 };
  }
  return { success: false, error: "Invalid input" };
});
```

### 2. Use Realistic Test Data

```typescript
// ❌ Bad - Unrealistic data
test("user service", () => {
  const user = { id: 1, name: "a", email: "b" };
  // ...
});

// ✅ Good - Realistic data
test("user service", () => {
  const user = {
    id: 1,
    name: "Ben Emma",
    email: "ben@inspatial.io",
    createdAt: "2024-01-15T10:30:00Z",
    preferences: {
      theme: "dark",
      notifications: true
    }
  };
  // ...
});
```

### 3. Clean Up Resources

```typescript
test("resource cleanup", () => {
  using time = new FakeTime();
  using apiStub = stub(api, "fetch", mockResponse);
  using dbConnection = mockDatabase.connect();
  
  // Test code here
  
  // Resources automatically cleaned up due to 'using'
});
```

### 4. Verify Mock Interactions

```typescript
test("verify mock interactions", () => {
  const emailService = mockFn();
  const smsService = mockFn();
  
  const notificationService = {
    async sendWelcome(user: User) {
      await emailService.send(user.email, "Welcome!");
      if (user.phone) {
        await smsService.send(user.phone, "Welcome to our app!");
      }
    }
  };
  
  // Test the service
  await notificationService.sendWelcome({
    email: "ben@inspatial.io",
    phone: "+1234567890"
  });
  
  // Verify interactions
  expect(emailService).toHaveBeenCalledWith("ben@inspatial.io", "Welcome!");
  expect(smsService).toHaveBeenCalledWith("+1234567890", "Welcome to our app!");
});
```

## Common Pitfalls and Solutions

### 1. Over-Mocking

**Problem**: Mocking too much makes tests brittle and unrealistic.

```typescript
// ❌ Bad - Over-mocked
test("over mocked test", () => {
  const mockEverything = {
    add: mockFn(() => 4),
    subtract: mockFn(() => 2),
    multiply: mockFn(() => 6),
    divide: mockFn(() => 3)
  };
  
  // This test doesn't actually test the real logic
});

// ✅ Good - Mock only external dependencies
test("properly mocked test", () => {
  using apiStub = stub(externalApi, "fetch", mockResponse);
  
  // Test real business logic with mocked external dependency
  const result = await businessLogic.processOrder(orderData);
  expect(result.total).toBe(calculateExpectedTotal(orderData));
});
```

### 2. Forgetting to Reset Mocks

**Problem**: Mocks from one test affecting another.

```typescript
// ✅ Good - Use 'using' for automatic cleanup
test("automatic cleanup", () => {
  using mockTimer = new FakeTime();
  using apiStub = stub(api, "call", mockResponse);
  
  // Test code here
  // Mocks automatically restored after test
});
```

### 3. Not Testing Error Paths

**Problem**: Only testing happy paths.

```typescript
// ✅ Good - Test both success and failure
test("comprehensive error testing", async () => {
  const service = {
    async processData(data: any) {
      if (!data) throw new Error("No data provided");
      if (data.invalid) throw new Error("Invalid data");
      return { processed: true };
    }
  };
  
  // Test success
  const result = await service.processData({ valid: true });
  expect(result.processed).toBe(true);
  
  // Test error cases
  await expect(service.processData(null)).rejects.toThrow("No data provided");
  await expect(service.processData({ invalid: true })).rejects.toThrow("Invalid data");
});
```

## What's Next?

Now that you've mastered advanced mocking, you might want to explore:

- [Performance Testing](./performance-testing.md) - Load testing and benchmarking
- [CI/CD Integration](./ci-cd.md) - Running tests in continuous integration

<!-- - [Integration Testing](./integration-testing.md) - Testing system interactions
- [Spatial Testing](./spatial-testing.md) - Testing 3D and AR/VR applications -->


Advanced mocking is a powerful tool that lets you test complex scenarios with confidence. Use it wisely to create comprehensive test suites that catch bugs before they reach production, while keeping your tests maintainable and realistic.

> **Note:** Remember that mocks are tools to help you test your code's behavior, not replace good design. Well-designed code with clear dependencies is easier to test and mock effectively.

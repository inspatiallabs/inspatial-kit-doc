## Performance Testing

#### Ensure your applications run fast and scale beautifully

> If you're just getting started, read the [Core Concepts](./getting-started.md#core-concepts) and basic testing flow before diving into performance strategies.

You've built something amazing, but how do you know it will perform well under real-world conditions? Performance testing is like taking your car for a test drive before a long road trip - you want to make sure everything runs smoothly when it matters most. Whether you're building a simple website or a complex XR experience, performance testing helps you identify bottlenecks, optimize resource usage, and ensure your users have a smooth experience.

Performance testing isn't just about speed - it's about understanding how your application behaves under different conditions. You'll learn to measure response times, memory usage, CPU utilization, and how your app scales when hundreds or thousands of users interact with it simultaneously. Think of it as your application's fitness test.

> **Terminology:**
>
> - **Benchmark**: A standardized test that measures performance against a baseline
> - **Load Testing**: Testing normal expected usage patterns
> - **Stress Testing**: Testing beyond normal capacity to find breaking points
> - **Spike Testing**: Testing sudden increases in load
> - **Volume Testing**: Testing with large amounts of data
> - **Endurance Testing**: Testing performance over extended periods

## Understanding Performance Metrics

Before diving into testing techniques, let's understand what we measure and why:

### Core Performance Metrics

```typescript
import { test, performance, expect } from "@inspatial/kit/test";

test("understanding performance metrics", async () => {
  const startTime = performance.now();

  // Simulate some work
  const result = await expensiveOperation();

  const endTime = performance.now();
  const duration = endTime - startTime;

  // Response Time: How long did it take?
  expect(duration).toBeLessThan(1000); // Should complete in under 1 second

  // Memory Usage: How much memory was used?
  const memoryUsage = performance.memory?.usedJSHeapSize || 0;
  expect(memoryUsage).toBeLessThan(50 * 1024 * 1024); // Less than 50MB

  // Throughput: How many operations per second?
  const operationsPerSecond = 1000 / duration;
  expect(operationsPerSecond).toBeGreaterThan(10); // At least 10 ops/sec
});

async function expensiveOperation() {
  // Simulate CPU-intensive work
  let result = 0;
  for (let i = 0; i < 100000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}
```

### Memory Performance Testing

Monitor memory usage and detect memory leaks:

```typescript
import { test, performance, expect } from "@inspatial/kit/test";

test("memory performance monitoring", async () => {
  const initialMemory = performance.memory?.usedJSHeapSize || 0;

  // Create and process large datasets
  const largeArray = new Array(100000).fill(0).map((_, i) => ({
    id: i,
    data: `Item ${i}`,
    timestamp: new Date(),
    metadata: { processed: false, priority: i % 3 },
  }));

  // Process the data
  const processed = largeArray.map((item) => ({
    ...item,
    metadata: { ...item.metadata, processed: true },
  }));

  const afterProcessingMemory = performance.memory?.usedJSHeapSize || 0;
  const memoryIncrease = afterProcessingMemory - initialMemory;

  // Memory should not increase excessively
  expect(memoryIncrease).toBeLessThan(100 * 1024 * 1024); // Less than 100MB increase

  // Clean up
  largeArray.length = 0;
  processed.length = 0;

  // Force garbage collection if available
  if (globalThis.gc) {
    globalThis.gc();
  }

  // Wait a bit for cleanup
  await new Promise((resolve) => setTimeout(resolve, 100));

  const afterCleanupMemory = performance.memory?.usedJSHeapSize || 0;
  const memoryLeaked = afterCleanupMemory - initialMemory;

  // Should not have significant memory leaks
  expect(memoryLeaked).toBeLessThan(10 * 1024 * 1024); // Less than 10MB leaked
});
```

## Performance Benchmarking

### Quick Benchmarking with InSpatial Test

For dedicated benchmarking and performance comparisons, InSpatial Test provides a powerful `bench()` API that works across all JavaScript runtimes:

```typescript
import { bench } from "@inspatial/kit/test";

// Simple benchmark
bench("Array operations", () => {
  const arr = [];
  for (let i = 0; i < 1000; i++) {
    arr.push(i);
  }
});

// Baseline comparison
bench("String concatenation", { baseline: true }, () => {
  let result = "";
  for (let i = 0; i < 100; i++) {
    result += `item-${i}`;
  }
});

bench("Array join", () => {
  const parts = [];
  for (let i = 0; i < 100; i++) {
    parts.push(`item-${i}`);
  }
  return parts.join("");
});
```

> **For comprehensive benchmarking:** See our complete [Benchmarking Guide](./benchmarking.md) for advanced techniques, grouping, statistical analysis, and CI/CD integration.

## Baseline Testing

### Regression Testing

Detect performance regressions over time:

```typescript
import { test, performance, expect } from "@inspatial/kit/test";

interface PerformanceBaseline {
  operation: string;
  averageDuration: number;
  maxDuration: number;
  memoryUsage: number;
  timestamp: string;
}

class PerformanceRegression {
  private baselines: Map<string, PerformanceBaseline> = new Map();

  setBaseline(operation: string, baseline: PerformanceBaseline) {
    this.baselines.set(operation, baseline);
  }

  async checkRegression(
    operation: string,
    testFunction: () => Promise<any> | any,
    iterations: number = 5
  ) {
    const baseline = this.baselines.get(operation);
    if (!baseline) {
      throw new Error(`No baseline found for operation: ${operation}`);
    }

    const measurements: number[] = [];
    const startMemory = performance.memory?.usedJSHeapSize || 0;

    // Run multiple iterations for accuracy
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      await testFunction();
      const end = performance.now();
      measurements.push(end - start);
    }

    const endMemory = performance.memory?.usedJSHeapSize || 0;
    const averageDuration =
      measurements.reduce((a, b) => a + b, 0) / measurements.length;
    const maxDuration = Math.max(...measurements);
    const memoryUsage = endMemory - startMemory;

    // Check for regressions (allow 20% variance)
    const durationRegression =
      (averageDuration - baseline.averageDuration) / baseline.averageDuration;
    const memoryRegression =
      (memoryUsage - baseline.memoryUsage) / baseline.memoryUsage;

    return {
      passed: durationRegression < 0.2 && memoryRegression < 0.2,
      averageDuration,
      maxDuration,
      memoryUsage,
      durationRegression: durationRegression * 100,
      memoryRegression: memoryRegression * 100,
      baseline,
    };
  }
}

test("performance regression detection", async () => {
  const regression = new PerformanceRegression();

  // Set baseline (this would typically be loaded from a file)
  regression.setBaseline("data_processing", {
    operation: "data_processing",
    averageDuration: 100, // 100ms baseline
    maxDuration: 150,
    memoryUsage: 1024 * 1024, // 1MB baseline
    timestamp: "2024-01-01T00:00:00Z",
  });

  // Test current performance
  const result = await regression.checkRegression(
    "data_processing",
    async () => {
      // Simulate data processing
      const data = new Array(10000)
        .fill(0)
        .map((_, i) => ({ id: i, value: Math.random() }));
      return data
        .filter((item) => item.value > 0.5)
        .map((item) => ({ ...item, processed: true }));
    }
  );

  expect(result.passed).toBe(true);
  expect(result.durationRegression).toBeLessThan(20); // Less than 20% regression
  expect(result.memoryRegression).toBeLessThan(20); // Less than 20% memory increase
});
```

## Load Testing

### Simulating Concurrent Users

Test how your application handles multiple simultaneous users:

```typescript
import { test, expect } from "@inspatial/kit/test";

class LoadTester {
  async simulateUsers(
    userCount: number,
    userAction: (userId: number) => Promise<any>,
    options: { rampUpTime?: number; duration?: number } = {}
  ) {
    const { rampUpTime = 0, duration = 1000 } = options;
    const results: Array<{
      userId: number;
      success: boolean;
      duration: number;
      error?: string;
    }> = [];

    // Create user simulation promises
    const userPromises = Array.from(
      { length: userCount },
      async (_, userId) => {
        // Stagger user start times for ramp-up
        if (rampUpTime > 0) {
          const delay = (rampUpTime / userCount) * userId;
          await new Promise((resolve) => setTimeout(resolve, delay));
        }

        const startTime = performance.now();

        try {
          await userAction(userId);
          const endTime = performance.now();

          results.push({
            userId,
            success: true,
            duration: endTime - startTime,
          });
        } catch (error) {
          const endTime = performance.now();

          results.push({
            userId,
            success: false,
            duration: endTime - startTime,
            error: error instanceof Error ? error.message : String(error),
          });
        }
      }
    );

    // Wait for all users to complete
    await Promise.all(userPromises);

    return this.analyzeResults(results);
  }

  private analyzeResults(
    results: Array<{
      userId: number;
      success: boolean;
      duration: number;
      error?: string;
    }>
  ) {
    const successful = results.filter((r) => r.success);
    const failed = results.filter((r) => !r.success);
    const durations = successful.map((r) => r.duration);

    return {
      totalUsers: results.length,
      successfulUsers: successful.length,
      failedUsers: failed.length,
      successRate: (successful.length / results.length) * 100,
      averageResponseTime:
        durations.length > 0
          ? durations.reduce((a, b) => a + b, 0) / durations.length
          : 0,
      minResponseTime: durations.length > 0 ? Math.min(...durations) : 0,
      maxResponseTime: durations.length > 0 ? Math.max(...durations) : 0,
      percentile95:
        durations.length > 0 ? this.calculatePercentile(durations, 95) : 0,
      errors: failed.map((f) => f.error).filter(Boolean),
    };
  }

  private calculatePercentile(values: number[], percentile: number): number {
    const sorted = values.sort((a, b) => a - b);
    const index = Math.ceil((percentile / 100) * sorted.length) - 1;
    return sorted[index] || 0;
  }
}

test("load testing with concurrent users", async () => {
  const loadTester = new LoadTester();

  // Mock API service
  const apiService = {
    async processRequest(userId: number) {
      // Simulate variable response times
      const delay = Math.random() * 200 + 50; // 50-250ms
      await new Promise((resolve) => setTimeout(resolve, delay));

      // Simulate occasional failures (5% failure rate)
      if (Math.random() < 0.05) {
        throw new Error(`Service temporarily unavailable for user ${userId}`);
      }

      return { userId, processed: true, timestamp: new Date().toISOString() };
    },
  };

  // Test with 50 concurrent users
  const results = await loadTester.simulateUsers(
    50,
    async (userId) => {
      return await apiService.processRequest(userId);
    },
    {
      rampUpTime: 1000, // Ramp up over 1 second
      duration: 5000, // Test for 5 seconds
    }
  );

  // Verify performance requirements
  expect(results.successRate).toBeGreaterThan(90); // At least 90% success rate
  expect(results.averageResponseTime).toBeLessThan(300); // Average under 300ms
  expect(results.percentile95).toBeLessThan(500); // 95% under 500ms
  expect(results.failedUsers).toBeLessThan(5); // Less than 5 failures
});
```

### API Endpoint Load Testing

Test specific API endpoints under load:

```typescript
import { test, expect, mockFn } from "@inspatial/kit/test";

class ApiLoadTester {
  private mockFetch = mockFn();

  constructor() {
    // Mock global fetch for testing
    globalThis.fetch = this.mockFetch;
  }

  setupMockResponses() {
    this.mockFetch.mockImplementation(async (url: string, options?: any) => {
      // Simulate realistic API response times
      const delay = Math.random() * 100 + 50; // 50-150ms
      await new Promise((resolve) => setTimeout(resolve, delay));

      // Simulate different endpoints
      if (url.includes("/api/users")) {
        return new Response(
          JSON.stringify({
            users: Array.from({ length: 10 }, (_, i) => ({
              id: i + 1,
              name: `User ${i + 1}`,
              email: `user${i + 1}@inspatial.io`,
            })),
          }),
          { status: 200 }
        );
      }

      if (url.includes("/api/data")) {
        // Simulate heavier payload
        const data = Array.from({ length: 100 }, (_, i) => ({
          id: i,
          value: Math.random(),
          metadata: { processed: true, timestamp: new Date().toISOString() },
        }));

        return new Response(JSON.stringify({ data }), { status: 200 });
      }

      return new Response("Not Found", { status: 404 });
    });
  }

  async testEndpointLoad(endpoint: string, concurrentRequests: number) {
    this.setupMockResponses();

    const startTime = performance.now();
    const promises = Array.from(
      { length: concurrentRequests },
      async (_, i) => {
        const requestStart = performance.now();

        try {
          const response = await fetch(endpoint);
          const data = await response.json();
          const requestEnd = performance.now();

          return {
            success: true,
            duration: requestEnd - requestStart,
            status: response.status,
            dataSize: JSON.stringify(data).length,
          };
        } catch (error) {
          const requestEnd = performance.now();

          return {
            success: false,
            duration: requestEnd - requestStart,
            error: error instanceof Error ? error.message : String(error),
          };
        }
      }
    );

    const results = await Promise.all(promises);
    const endTime = performance.now();

    const successful = results.filter((r) => r.success);
    const failed = results.filter((r) => !r.success);

    return {
      endpoint,
      totalRequests: concurrentRequests,
      successfulRequests: successful.length,
      failedRequests: failed.length,
      totalTime: endTime - startTime,
      averageResponseTime:
        successful.length > 0
          ? successful.reduce((sum, r) => sum + r.duration, 0) /
            successful.length
          : 0,
      requestsPerSecond: (successful.length / (endTime - startTime)) * 1000,
      averageDataSize:
        successful.length > 0
          ? successful.reduce((sum, r) => sum + (r.dataSize || 0), 0) /
            successful.length
          : 0,
    };
  }
}

test("API endpoint load testing", async () => {
  const apiTester = new ApiLoadTester();

  // Test users endpoint
  const usersResult = await apiTester.testEndpointLoad("/api/users", 20);

  expect(usersResult.successfulRequests).toBe(20);
  expect(usersResult.averageResponseTime).toBeLessThan(200); // Under 200ms average
  expect(usersResult.requestsPerSecond).toBeGreaterThan(50); // At least 50 RPS

  // Test data endpoint (heavier payload)
  const dataResult = await apiTester.testEndpointLoad("/api/data", 10);

  expect(dataResult.successfulRequests).toBe(10);
  expect(dataResult.averageResponseTime).toBeLessThan(300); // Under 300ms for heavier payload
  expect(dataResult.averageDataSize).toBeGreaterThan(1000); // Verify we're getting substantial data
});
```

## Stress Testing

### Finding Breaking Points

Test your application beyond normal capacity to find its limits:

```typescript
import { test, expect } from "@inspatial/kit/test";

class StressTester {
  async findBreakingPoint(
    testFunction: (
      load: number
    ) => Promise<{ success: boolean; responseTime: number }>,
    options: {
      startLoad: number;
      maxLoad: number;
      stepSize: number;
      failureThreshold: number; // Percentage of failures that indicates breaking point
      responseTimeThreshold: number; // Response time that indicates degradation
    }
  ) {
    const results: Array<{
      load: number;
      successRate: number;
      averageResponseTime: number;
      broken: boolean;
    }> = [];

    for (
      let load = options.startLoad;
      load <= options.maxLoad;
      load += options.stepSize
    ) {
      const testResults: Array<{ success: boolean; responseTime: number }> = [];

      // Run multiple tests at this load level
      for (let i = 0; i < 10; i++) {
        try {
          const result = await testFunction(load);
          testResults.push(result);
        } catch (error) {
          testResults.push({ success: false, responseTime: Infinity });
        }
      }

      const successCount = testResults.filter((r) => r.success).length;
      const successRate = (successCount / testResults.length) * 100;
      const validResponseTimes = testResults.filter(
        (r) => r.success && r.responseTime !== Infinity
      );
      const averageResponseTime =
        validResponseTimes.length > 0
          ? validResponseTimes.reduce((sum, r) => sum + r.responseTime, 0) /
            validResponseTimes.length
          : Infinity;

      const broken =
        successRate < 100 - options.failureThreshold ||
        averageResponseTime > options.responseTimeThreshold;

      results.push({
        load,
        successRate,
        averageResponseTime,
        broken,
      });

      // Stop if we've found the breaking point
      if (broken) {
        break;
      }
    }

    return results;
  }
}

test("stress testing to find breaking points", async () => {
  const stressTester = new StressTester();

  // Mock service that degrades under load
  const mockService = {
    async processLoad(load: number) {
      // Simulate degrading performance under load
      const baseResponseTime = 50;
      const loadFactor = Math.pow(load / 10, 2); // Exponential degradation
      const responseTime = baseResponseTime + loadFactor;

      await new Promise((resolve) => setTimeout(resolve, responseTime));

      // Simulate failures under high load
      const failureRate = Math.max(0, (load - 50) / 100); // Start failing after load 50
      const success = Math.random() > failureRate;

      return { success, responseTime };
    },
  };

  const results = await stressTester.findBreakingPoint(
    (load) => mockService.processLoad(load),
    {
      startLoad: 10,
      maxLoad: 100,
      stepSize: 10,
      failureThreshold: 20, // 20% failure rate indicates breaking point
      responseTimeThreshold: 1000, // 1 second response time threshold
    }
  );

  // Verify we found performance characteristics
  expect(results.length).toBeGreaterThan(0);

  const lastGoodResult = results.find((r) => !r.broken);
  const firstBrokenResult = results.find((r) => r.broken);

  if (lastGoodResult) {
    expect(lastGoodResult.successRate).toBeGreaterThan(80);
    expect(lastGoodResult.averageResponseTime).toBeLessThan(1000);
  }

  if (firstBrokenResult) {
    expect(firstBrokenResult.successRate).toBeLessThan(80);
  }
});
```

### Memory Stress Testing

Test memory usage under extreme conditions:

```typescript
import { test, expect } from "@inspatial/kit/test";

class MemoryStressTester {
  async testMemoryUsage(
    memoryIntensiveOperation: (size: number) => Promise<any>,
    maxMemoryMB: number = 100
  ) {
    const initialMemory = performance.memory?.usedJSHeapSize || 0;
    const maxMemoryBytes = maxMemoryMB * 1024 * 1024;

    const results: Array<{
      operationSize: number;
      memoryUsed: number;
      memoryLeaked: number;
      operationTime: number;
      success: boolean;
    }> = [];

    // Test with increasing memory loads
    for (let size = 1000; size <= 100000; size *= 2) {
      const beforeOperation = performance.memory?.usedJSHeapSize || 0;
      const startTime = performance.now();

      try {
        await memoryIntensiveOperation(size);

        const afterOperation = performance.memory?.usedJSHeapSize || 0;
        const operationTime = performance.now() - startTime;
        const memoryUsed = afterOperation - beforeOperation;

        // Force garbage collection if available
        if (globalThis.gc) {
          globalThis.gc();
        }

        // Wait for cleanup
        await new Promise((resolve) => setTimeout(resolve, 100));

        const afterCleanup = performance.memory?.usedJSHeapSize || 0;
        const memoryLeaked = afterCleanup - beforeOperation;

        results.push({
          operationSize: size,
          memoryUsed,
          memoryLeaked,
          operationTime,
          success: true,
        });

        // Stop if we're using too much memory
        if (memoryUsed > maxMemoryBytes) {
          break;
        }
      } catch (error) {
        const operationTime = performance.now() - startTime;

        results.push({
          operationSize: size,
          memoryUsed: 0,
          memoryLeaked: 0,
          operationTime,
          success: false,
        });

        break; // Stop on first failure
      }
    }

    return results;
  }
}

test("memory stress testing", async () => {
  const memoryTester = new MemoryStressTester();

  const results = await memoryTester.testMemoryUsage(async (size) => {
    // Create large data structures
    const largeArray = new Array(size).fill(0).map((_, i) => ({
      id: i,
      data: new Array(100).fill(`data-${i}`),
      metadata: {
        timestamp: new Date(),
        processed: false,
        tags: new Array(10).fill(`tag-${i}`),
      },
    }));

    // Process the data
    const processed = largeArray.map((item) => ({
      ...item,
      metadata: {
        ...item.metadata,
        processed: true,
        processedAt: new Date(),
      },
    }));

    // Simulate some work with the data
    const summary = processed.reduce(
      (acc, item) => {
        acc.totalItems++;
        acc.totalDataLength += item.data.length;
        return acc;
      },
      { totalItems: 0, totalDataLength: 0 }
    );

    return summary;
  }, 50); // Limit to 50MB

  // Verify memory behavior
  expect(results.length).toBeGreaterThan(0);

  const successfulResults = results.filter((r) => r.success);
  expect(successfulResults.length).toBeGreaterThan(0);

  // Memory usage should scale predictably
  for (let i = 1; i < successfulResults.length; i++) {
    const current = successfulResults[i];
    const previous = successfulResults[i - 1];

    // Memory usage should increase with operation size
    expect(current.memoryUsed).toBeGreaterThan(previous.memoryUsed);

    // Memory leaks should be minimal (less than 10% of used memory)
    expect(current.memoryLeaked).toBeLessThan(current.memoryUsed * 0.1);
  }
});
```

## Real-World Performance Scenarios

### Database Query Performance

Test database operations under various conditions:

```typescript
import { test, expect, mockFn } from "@inspatial/kit/test";

class DatabasePerformanceTester {
  private mockDb = {
    query: mockFn(),
    connect: mockFn(),
    disconnect: mockFn(),
  };

  setupMockDatabase() {
    this.mockDb.query.mockImplementation(
      async (sql: string, params?: any[]) => {
        // Simulate query complexity based on SQL
        let baseTime = 10; // Base 10ms

        if (sql.includes("JOIN")) baseTime += 20;
        if (sql.includes("ORDER BY")) baseTime += 15;
        if (sql.includes("GROUP BY")) baseTime += 25;
        if (sql.includes("COUNT")) baseTime += 10;

        // Simulate parameter impact
        const paramCount = params?.length || 0;
        baseTime += paramCount * 2;

        // Add some randomness
        const actualTime = baseTime + Math.random() * 10;
        await new Promise((resolve) => setTimeout(resolve, actualTime));

        // Simulate different result sizes
        const resultSize = sql.includes("LIMIT") ? 10 : 100;
        return {
          rows: Array.from({ length: resultSize }, (_, i) => ({
            id: i + 1,
            name: `Record ${i + 1}`,
            value: Math.random() * 1000,
          })),
          rowCount: resultSize,
          executionTime: actualTime,
        };
      }
    );
  }

  async testQueryPerformance(
    queries: Array<{ name: string; sql: string; params?: any[] }>
  ) {
    this.setupMockDatabase();

    const results = [];

    for (const query of queries) {
      const iterations = 5;
      const times: number[] = [];

      for (let i = 0; i < iterations; i++) {
        const start = performance.now();
        const result = await this.mockDb.query(query.sql, query.params);
        const end = performance.now();

        times.push(end - start);
      }

      const averageTime = times.reduce((a, b) => a + b, 0) / times.length;
      const minTime = Math.min(...times);
      const maxTime = Math.max(...times);

      results.push({
        name: query.name,
        sql: query.sql,
        averageTime,
        minTime,
        maxTime,
        consistency: (maxTime - minTime) / averageTime, // Lower is more consistent
      });
    }

    return results;
  }
}

test("database query performance testing", async () => {
  const dbTester = new DatabasePerformanceTester();

  const queries = [
    {
      name: "simple_select",
      sql: "SELECT * FROM users WHERE id = ?",
      params: [1],
    },
    {
      name: "complex_join",
      sql: "SELECT u.*, p.* FROM users u JOIN profiles p ON u.id = p.user_id WHERE u.active = ? ORDER BY u.created_at",
      params: [true],
    },
    {
      name: "aggregation",
      sql: "SELECT department, COUNT(*), AVG(salary) FROM employees GROUP BY department ORDER BY COUNT(*) DESC",
      params: [],
    },
    {
      name: "large_result",
      sql: "SELECT * FROM transactions WHERE date >= ? AND date <= ?",
      params: ["2024-01-01", "2024-12-31"],
    },
  ];

  const results = await dbTester.testQueryPerformance(queries);

  // Verify performance expectations
  const simpleSelect = results.find((r) => r.name === "simple_select")!;
  const complexJoin = results.find((r) => r.name === "complex_join")!;

  expect(simpleSelect.averageTime).toBeLessThan(50); // Simple queries under 50ms
  expect(complexJoin.averageTime).toBeLessThan(100); // Complex queries under 100ms

  // Consistency check (variance should be reasonable)
  results.forEach((result) => {
    expect(result.consistency).toBeLessThan(0.5); // Variance should be less than 50% of average
  });
});
```

### File I/O Performance

Test file operations performance:

```typescript
import { test, expect, setupFsMocks } from "@inspatial/kit/test";

class FilePerformanceTester {
  async testFileOperations(fileSizes: number[]) {
    const { fs } = setupFsMocks({
      virtualFiles: {},
    });

    const results = [];

    for (const size of fileSizes) {
      // Generate test data
      const testData = "x".repeat(size);
      const fileName = `/test-${size}.txt`;

      // Test write performance
      const writeStart = performance.now();
      await fs.writeTextFile(fileName, testData);
      const writeEnd = performance.now();
      const writeTime = writeEnd - writeStart;

      // Test read performance
      const readStart = performance.now();
      const readData = await fs.readTextFile(fileName);
      const readEnd = performance.now();
      const readTime = readEnd - readStart;

      // Test append performance
      const appendData = "y".repeat(size / 10);
      const appendStart = performance.now();
      await fs.writeTextFile(fileName, readData + appendData);
      const appendEnd = performance.now();
      const appendTime = appendEnd - appendStart;

      // Calculate throughput (bytes per second)
      const writeThroughput = size / (writeTime / 1000);
      const readThroughput = size / (readTime / 1000);

      results.push({
        fileSize: size,
        writeTime,
        readTime,
        appendTime,
        writeThroughput,
        readThroughput,
        dataIntegrity: readData === testData,
      });
    }

    return results;
  }
}

test("file I/O performance testing", async () => {
  const fileTester = new FilePerformanceTester();

  // Test with different file sizes (1KB, 10KB, 100KB, 1MB)
  const fileSizes = [1024, 10240, 102400, 1048576];

  const results = await fileTester.testFileOperations(fileSizes);

  // Verify all operations completed successfully
  results.forEach((result) => {
    expect(result.dataIntegrity).toBe(true);
  });

  // Performance should scale reasonably with file size
  for (let i = 1; i < results.length; i++) {
    const current = results[i];
    const previous = results[i - 1];

    // Larger files should take more time, but not exponentially more
    const sizeRatio = current.fileSize / previous.fileSize;
    const timeRatio = current.writeTime / previous.writeTime;

    expect(timeRatio).toBeLessThan(sizeRatio * 2); // Should scale sub-linearly
  }

  // Throughput should be reasonable (at least 1MB/s for reads)
  const largestFile = results[results.length - 1];
  expect(largestFile.readThroughput).toBeGreaterThan(1048576); // 1MB/s
});
```

## Performance Monitoring and Alerting

### Continuous Performance Monitoring

Set up automated performance monitoring:

```typescript
import { test, expect } from "@inspatial/kit/test";

class PerformanceMonitor {
  private metrics: Array<{
    timestamp: Date;
    operation: string;
    duration: number;
    memoryUsage: number;
    success: boolean;
  }> = [];

  async monitor<T>(operation: string, fn: () => Promise<T> | T): Promise<T> {
    const startTime = performance.now();
    const startMemory = performance.memory?.usedJSHeapSize || 0;

    try {
      const result = await fn();
      const endTime = performance.now();
      const endMemory = performance.memory?.usedJSHeapSize || 0;

      this.metrics.push({
        timestamp: new Date(),
        operation,
        duration: endTime - startTime,
        memoryUsage: endMemory - startMemory,
        success: true,
      });

      return result;
    } catch (error) {
      const endTime = performance.now();
      const endMemory = performance.memory?.usedJSHeapSize || 0;

      this.metrics.push({
        timestamp: new Date(),
        operation,
        duration: endTime - startTime,
        memoryUsage: endMemory - startMemory,
        success: false,
      });

      throw error;
    }
  }

  getMetrics(operation?: string) {
    return operation
      ? this.metrics.filter((m) => m.operation === operation)
      : this.metrics;
  }

  getAveragePerformance(operation: string, timeWindow?: number) {
    const now = new Date();
    const windowStart = timeWindow
      ? new Date(now.getTime() - timeWindow)
      : new Date(0);

    const relevantMetrics = this.metrics.filter(
      (m) =>
        m.operation === operation && m.timestamp >= windowStart && m.success
    );

    if (relevantMetrics.length === 0) {
      return null;
    }

    const totalDuration = relevantMetrics.reduce(
      (sum, m) => sum + m.duration,
      0
    );
    const totalMemory = relevantMetrics.reduce(
      (sum, m) => sum + m.memoryUsage,
      0
    );

    return {
      count: relevantMetrics.length,
      averageDuration: totalDuration / relevantMetrics.length,
      averageMemoryUsage: totalMemory / relevantMetrics.length,
      successRate:
        (relevantMetrics.length /
          this.metrics.filter((m) => m.operation === operation).length) *
        100,
    };
  }

  checkPerformanceAlerts(thresholds: {
    [operation: string]: {
      maxDuration: number;
      maxMemoryUsage: number;
      minSuccessRate: number;
    };
  }) {
    const alerts = [];

    for (const [operation, threshold] of Object.entries(thresholds)) {
      const performance = this.getAveragePerformance(operation, 60000); // Last minute

      if (!performance) continue;

      if (performance.averageDuration > threshold.maxDuration) {
        alerts.push({
          type: "duration",
          operation,
          actual: performance.averageDuration,
          threshold: threshold.maxDuration,
          message: `${operation} average duration (${performance.averageDuration.toFixed(
            2
          )}ms) exceeds threshold (${threshold.maxDuration}ms)`,
        });
      }

      if (performance.averageMemoryUsage > threshold.maxMemoryUsage) {
        alerts.push({
          type: "memory",
          operation,
          actual: performance.averageMemoryUsage,
          threshold: threshold.maxMemoryUsage,
          message: `${operation} average memory usage (${(
            performance.averageMemoryUsage /
            1024 /
            1024
          ).toFixed(2)}MB) exceeds threshold (${(
            threshold.maxMemoryUsage /
            1024 /
            1024
          ).toFixed(2)}MB)`,
        });
      }

      if (performance.successRate < threshold.minSuccessRate) {
        alerts.push({
          type: "success_rate",
          operation,
          actual: performance.successRate,
          threshold: threshold.minSuccessRate,
          message: `${operation} success rate (${performance.successRate.toFixed(
            2
          )}%) below threshold (${threshold.minSuccessRate}%)`,
        });
      }
    }

    return alerts;
  }
}

test("performance monitoring and alerting", async () => {
  const monitor = new PerformanceMonitor();

  // Simulate various operations
  await monitor.monitor("user_login", async () => {
    await new Promise((resolve) => setTimeout(resolve, 150)); // 150ms
    return { userId: 1, token: "abc123" };
  });

  await monitor.monitor("data_processing", async () => {
    await new Promise((resolve) => setTimeout(resolve, 300)); // 300ms
    // Simulate memory usage
    const data = new Array(10000).fill(0);
    return data.length;
  });

  await monitor.monitor("user_login", async () => {
    await new Promise((resolve) => setTimeout(resolve, 100)); // 100ms
    return { userId: 2, token: "def456" };
  });

  // Check performance metrics
  const loginPerformance = monitor.getAveragePerformance("user_login");
  expect(loginPerformance).toBeDefined();
  expect(loginPerformance!.count).toBe(2);
  expect(loginPerformance!.averageDuration).toBeCloseTo(125, 0); // Average of 150ms and 100ms

  // Set up performance thresholds
  const thresholds = {
    user_login: {
      maxDuration: 200,
      maxMemoryUsage: 1024 * 1024, // 1MB
      minSuccessRate: 95,
    },
    data_processing: {
      maxDuration: 500,
      maxMemoryUsage: 10 * 1024 * 1024, // 10MB
      minSuccessRate: 90,
    },
  };

  // Check for alerts
  const alerts = monitor.checkPerformanceAlerts(thresholds);

  // Should not have any alerts with current performance
  expect(alerts).toHaveLength(0);

  // Simulate a slow operation that should trigger an alert
  await monitor.monitor("user_login", async () => {
    await new Promise((resolve) => setTimeout(resolve, 500)); // 500ms - exceeds threshold
    return { userId: 3, token: "ghi789" };
  });

  const newAlerts = monitor.checkPerformanceAlerts(thresholds);
  expect(newAlerts.length).toBeGreaterThan(0);

  const durationAlert = newAlerts.find(
    (a) => a.type === "duration" && a.operation === "user_login"
  );
  expect(durationAlert).toBeDefined();
  expect(durationAlert!.actual).toBeGreaterThan(
    thresholds.user_login.maxDuration
  );
});
```

## Best Practices for Performance Testing

### 1. Establish Baselines Early

```typescript
// ✅ Good - Establish performance baselines
test("establish performance baseline", async () => {
  const baseline = {
    operation: "user_authentication",
    averageResponseTime: 150, // ms
    maxResponseTime: 300,
    memoryUsage: 2 * 1024 * 1024, // 2MB
    throughput: 100, // requests per second
  };

  // Save baseline for future comparisons
  // In real scenarios, this would be saved to a file or database
  expect(baseline.averageResponseTime).toBeLessThan(200);
});
```

### 2. Test Realistic Scenarios

```typescript
// ✅ Good - Test realistic user patterns
test("realistic user behavior patterns", async () => {
  const userBehaviors = [
    { action: "browse", frequency: 0.4, duration: 2000 },
    { action: "search", frequency: 0.3, duration: 1000 },
    { action: "purchase", frequency: 0.2, duration: 5000 },
    { action: "support", frequency: 0.1, duration: 10000 },
  ];

  // Simulate realistic user mix
  for (const behavior of userBehaviors) {
    const userCount = Math.floor(100 * behavior.frequency);
    // Test with realistic user distribution
  }
});
```

### 3. Monitor Resource Usage

```typescript
// ✅ Good - Comprehensive resource monitoring
test("comprehensive resource monitoring", async () => {
  const resourceMonitor = {
    cpu: performance.now(),
    memory: performance.memory?.usedJSHeapSize || 0,
    network: 0, // Would track network calls in real scenario
    storage: 0, // Would track storage usage in real scenario
  };

  // Monitor throughout test execution
  expect(resourceMonitor.memory).toBeLessThan(100 * 1024 * 1024); // 100MB limit
});
```

### 4. Test Edge Cases

```typescript
// ✅ Good - Test performance edge cases
test("performance edge cases", async () => {
  const edgeCases = [
    { scenario: "empty_dataset", dataSize: 0 },
    { scenario: "single_item", dataSize: 1 },
    { scenario: "large_dataset", dataSize: 100000 },
    { scenario: "maximum_dataset", dataSize: 1000000 },
  ];

  for (const edgeCase of edgeCases) {
    const performance = await testWithDataSize(edgeCase.dataSize);
    expect(performance.responseTime).toBeLessThan(5000); // 5 second max
  }
});

async function testWithDataSize(size: number) {
  const start = performance.now();
  // Simulate processing data of given size
  await new Promise((resolve) => setTimeout(resolve, Math.log(size + 1) * 10));
  const end = performance.now();

  return { responseTime: end - start };
}
```

## What's Next?

Now that you've mastered performance testing, you might want to explore:

- [CI/CD Integration](./ci-cd.md) - Running tests in continuous integration
- [Advanced Mocking](./advanced-mocking.md) - Complex test scenarios

<!-- - [Integration Testing](./integration-testing.md) - Testing system interactions
- [Spatial Testing](./spatial-testing.md) - Testing 3D and AR/VR applications -->


Performance testing is crucial for delivering applications that scale and provide excellent user experiences. Start with simple benchmarks, establish baselines early, and gradually build more sophisticated performance test suites as your application grows.

> **Note:** Performance testing should be an ongoing process, not a one-time activity. Regular performance monitoring helps catch regressions early and ensures your application continues to perform well as it evolves.

## Benchmarking

#### Measure performance and optimize your applications with precision

> New to InSpatial Test? Review the [Core Concepts](./getting-started.md#core-concepts) first, then use this guide for performance benchmarking.

You've built something amazing, but how fast is it really? Benchmarking is like having a stopwatch for your code - it tells you exactly how long operations take and helps you identify performance bottlenecks before they become problems. Whether you're optimizing a critical algorithm, comparing different approaches, or ensuring your app meets performance requirements, benchmarking gives you the data you need to make informed decisions.

InSpatial Test provides comprehensive benchmarking capabilities that work seamlessly across all JavaScript runtimes. You can measure execution time, memory usage, and compare different implementations side-by-side. Think of it as your performance laboratory where you can experiment, measure, and optimize with confidence.

> **Terminology:**
>
> - **Benchmark**: A test that measures how long code takes to execute
> - **Baseline**: A reference benchmark used for comparison
> - **Iteration**: A single execution of the benchmarked code
> - **Throughput**: How many operations can be performed per second
> - **Latency**: How long a single operation takes to complete

## Quick Start

Let's start with a simple benchmark to see how fast different array operations are:

```typescript
// math-operations.bench.ts
import { bench } from "@inspatial/kit/test";

// Using InSpatial Test benchmarking (works across all runtimes)
bench("Array.push vs Array.concat", () => {
  const arr = [];
  for (let i = 0; i < 1000; i++) {
    arr.push(i);
  }
});

bench("Array.concat", () => {
  let arr = [];
  for (let i = 0; i < 1000; i++) {
    arr = arr.concat([i]);
  }
});
```

Run your benchmarks:

```bash
# Deno
deno test math-operations.bench.ts

# Node.js
node --test math-operations.bench.ts

# Bun
bun test math-operations.bench.ts
```

You'll see output showing execution time, iterations per second, and performance statistics for each benchmark.

### Syntactic Sugar: createBenchmark

> **Terminology:**
>
> - **Syntactic Sugar**: An easier or more convenient way to write code that achieves the same result as a more verbose or complex approach. It doesn't add new functionality, but makes code simpler and more readable.


If you prefer to group and execute benchmarks programmatically with defaults, use `createBenchmark`:

```typescript
import { createBenchmark } from "@inspatial/kit/test";

// Provide a default group and default options applied to each benchmark
const b = createBenchmark({
  group: "algorithms",
  defaultOptions: { iterations: 50 },
});

b.run("quick sort", () => quickSort(testData));
b.bench("merge sort", () => mergeSort(testData), { baseline: true });

// Execute and inspect results
const results = await b.execute();
console.log(results);
```

## Understanding Benchmark Results

When you run benchmarks, you'll see several key metrics:

- **Time per iteration**: How long each execution takes (in nanoseconds)
- **Iterations per second**: How many times the code can run in one second
- **Min/Max/Average**: Statistical distribution of execution times
- **Percentiles (p75, p99)**: Performance at different percentile levels

```typescript
// Example output:
// benchmark      time (avg)        (min … max)       p75       p99      p995
// ------------- ----------------- ----------------- --------- --------- ---------
// Array.push       2.1 µs/iter   (1.8 µs … 15.2 µs)   2.0 µs   3.2 µs   15.2 µs
// Array.concat   127.3 µs/iter  (98.1 µs … 2.1 ms)  134.2 µs  890.1 µs   2.1 ms
```

## Baseline Comparisons

Compare different approaches by setting one as a baseline:

```typescript
// string-operations.bench.ts
import { bench } from "@inspatial/kit/test";

// Set this as our baseline for comparison
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

bench("Template literals", () => {
  const parts = [];
  for (let i = 0; i < 100; i++) {
    parts.push(`item-${i}`);
  }
  return parts.reduce((acc, part) => `${acc}${part}`, "");
});
```

The baseline comparison will show you exactly how much faster or slower each approach is:

```
Summary
  Array join
   2.15x faster than String concatenation

  Template literals
   1.23x slower than String concatenation
```

## Grouping Benchmarks

Organize related benchmarks using groups:

```typescript
// data-structures.bench.ts
import { bench } from "@inspatial/kit/test";

// Array operations group
bench("Array push", { group: "arrays", baseline: true }, () => {
  const arr = [];
  for (let i = 0; i < 1000; i++) {
    arr.push(i);
  }
});

bench("Array unshift", { group: "arrays" }, () => {
  const arr = [];
  for (let i = 0; i < 1000; i++) {
    arr.unshift(i);
  }
});

// Object operations group
bench("Object property access", { group: "objects", baseline: true }, () => {
  const obj = { a: 1, b: 2, c: 3 };
  for (let i = 0; i < 1000; i++) {
    const value = obj.a + obj.b + obj.c;
  }
});

bench("Object destructuring", { group: "objects" }, () => {
  const obj = { a: 1, b: 2, c: 3 };
  for (let i = 0; i < 1000; i++) {
    const { a, b, c } = obj;
    const value = a + b + c;
  }
});
```

## Precise Timing with start() and end()

Sometimes you need to measure only specific parts of your code:

```typescript
// file-operations.bench.ts
import { bench } from "@inspatial/kit/test";

bench("File processing", (b) => {
  // Setup (not measured)
  const filename = "./large-dataset.json";

  // Start measuring here
  b.start();
  const data = Deno.readTextFileSync(filename);
  const parsed = JSON.parse(data);
  const processed = parsed.map((item) => item.value * 2);
  b.end();
  // Stop measuring here

  // Cleanup (not measured)
  // Any cleanup code here won't affect the benchmark
});
```

## Memory Benchmarking

Use InSpatial Test's performance API to measure memory usage:

```typescript
// memory-usage.bench.ts
import { bench, performance } from "@inspatial/kit/test";

bench("Large array creation", (b) => {
  const startMemory = performance.memory?.usedJSHeapSize || 0;

  b.start();
  const result = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    value: Math.random(),
  }));
  b.end();

  const endMemory = performance.memory?.usedJSHeapSize || 0;
  console.log(
    `Memory used: ${((endMemory - startMemory) / 1024 / 1024).toFixed(2)}MB`
  );

  return result;
});

bench("Object creation loop", (b) => {
  const startMemory = performance.memory?.usedJSHeapSize || 0;

  b.start();
  const objects = [];
  for (let i = 0; i < 100000; i++) {
    objects.push({ id: i, value: Math.random() });
  }
  b.end();

  const endMemory = performance.memory?.usedJSHeapSize || 0;
  console.log(
    `Memory used: ${((endMemory - startMemory) / 1024 / 1024).toFixed(2)}MB`
  );

  return objects;
});
```

## Advanced Performance Measurement

Combine benchmarking with testing for comprehensive performance validation:

```typescript
// api-performance.bench.ts
import { bench, performance } from "@inspatial/kit/test";

// Benchmark different HTTP clients
bench("Fetch API", async () => {
  const response = await fetch("https://httpbin.org/json");
  await response.json();
});

bench("Custom HTTP client", async () => {
  // Your custom HTTP implementation
  const data = await customHttpGet("https://httpbin.org/json");
});

// Benchmark with timing control
bench("API with precise timing", async (b) => {
  // Setup (not measured)
  const apiUrl = "https://api.example.com/users";

  // Start measuring only the actual request
  b.start();
  const response = await fetch(apiUrl);
  const data = await response.json();
  b.end();

  // Validation (not measured)
  console.log(`Received ${data.users?.length || 0} users`);
});
```

## Performance Marks and Measures

Use performance marks for detailed timing analysis:

```typescript
// complex-operation.bench.ts
import { bench, performance } from "@inspatial/kit/test";

bench("Complex operation with phases", async (b) => {
  // Setup (not measured)
  performance.mark("operation-start");

  b.start();

  // Phase 1: Data loading
  performance.mark("loading-start");
  const data = await loadLargeDataset();
  performance.mark("loading-end");

  // Phase 2: Processing
  performance.mark("processing-start");
  const processed = await processData(data);
  performance.mark("processing-end");

  // Phase 3: Saving
  performance.mark("saving-start");
  await saveResults(processed);
  performance.mark("saving-end");

  b.end();

  performance.mark("operation-end");

  // Measure each phase (not included in benchmark timing)
  const loadingTime = performance.measure(
    "loading",
    "loading-start",
    "loading-end"
  );
  const processingTime = performance.measure(
    "processing",
    "processing-start",
    "processing-end"
  );
  const savingTime = performance.measure(
    "saving",
    "saving-start",
    "saving-end"
  );

  console.log(`Loading: ${loadingTime.duration}ms`);
  console.log(`Processing: ${processingTime.duration}ms`);
  console.log(`Saving: ${savingTime.duration}ms`);
});
```

## Async Function Benchmarking

Benchmark asynchronous operations properly:

```typescript
// async-operations.bench.ts
import { bench } from "@inspatial/kit/test";

// Benchmark async database operations
bench("Database query", async () => {
  const result = await db.query("SELECT * FROM users LIMIT 100");
  return result;
});

// Benchmark Promise-based operations
bench("Parallel API calls", async () => {
  const promises = [
    fetch("https://api.example.com/users"),
    fetch("https://api.example.com/posts"),
    fetch("https://api.example.com/comments"),
  ];

  const responses = await Promise.all(promises);
  return await Promise.all(responses.map((r) => r.json()));
});

// Benchmark sequential vs parallel operations
bench(
  "Sequential processing",
  { group: "processing", baseline: true },
  async () => {
    const results = [];
    for (let i = 0; i < 10; i++) {
      const result = await processItem(i);
      results.push(result);
    }
    return results;
  }
);

bench("Parallel processing", { group: "processing" }, async () => {
  const promises = Array.from({ length: 10 }, (_, i) => processItem(i));
  return await Promise.all(promises);
});
```

## Filtering and Focusing Benchmarks

Run specific benchmarks during development:

```typescript
// focused-benchmarks.bench.ts
import { bench } from "@inspatial/kit/test";

// Only run this benchmark (useful during optimization)
bench.only("Optimized algorithm", () => {
  // Your optimized code here
  return optimizedSort(largeArray);
});

// Ignore this benchmark for now
bench.ignore("Slow legacy algorithm", () => {
  return legacySort(largeArray);
});

// Regular benchmark
bench("Standard algorithm", () => {
  return standardSort(largeArray);
});
```

Run with filtering:

```bash
# Run only benchmarks matching a pattern
deno test --filter="algorithm" benchmarks/

# Run benchmarks and output JSON for analysis
deno test --reporter=json benchmarks/ > results.json
```

## Real-World Benchmarking Examples

### Database Performance

```typescript
// database-performance.bench.ts
import { bench } from "@inspatial/kit/test";

// Benchmark different query approaches
bench("Indexed query", { group: "database", baseline: true }, async () => {
  return await db.query("SELECT * FROM users WHERE id = ?", [123]);
});

bench("Full table scan", { group: "database" }, async () => {
  return await db.query("SELECT * FROM users WHERE name = ?", ["Ben"]);
});

// Benchmark connection pooling
bench(
  "Single connection",
  { group: "connections", baseline: true },
  async () => {
    const connection = await createConnection();
    const result = await connection.query("SELECT COUNT(*) FROM users");
    await connection.close();
    return result;
  }
);

bench("Connection pool", { group: "connections" }, async () => {
  const connection = await pool.getConnection();
  const result = await connection.query("SELECT COUNT(*) FROM users");
  pool.releaseConnection(connection);
  return result;
});
```

### Algorithm Comparison

```typescript
// sorting-algorithms.bench.ts
import { bench } from "@inspatial/kit/test";

// Generate test data
const smallArray = Array.from({ length: 100 }, () => Math.random());
const largeArray = Array.from({ length: 10000 }, () => Math.random());

// Small array benchmarks
bench("Bubble sort (small)", { group: "small-arrays", baseline: true }, () => {
  return bubbleSort([...smallArray]);
});

bench("Quick sort (small)", { group: "small-arrays" }, () => {
  return quickSort([...smallArray]);
});

bench("Native sort (small)", { group: "small-arrays" }, () => {
  return [...smallArray].sort((a, b) => a - b);
});

// Large array benchmarks
bench("Quick sort (large)", { group: "large-arrays", baseline: true }, () => {
  return quickSort([...largeArray]);
});

bench("Merge sort (large)", { group: "large-arrays" }, () => {
  return mergeSort([...largeArray]);
});

bench("Native sort (large)", { group: "large-arrays" }, () => {
  return [...largeArray].sort((a, b) => a - b);
});
```

### Web Performance

```typescript
// web-performance.bench.ts
import { bench } from "@inspatial/kit/test";

bench("Vanilla DOM manipulation", { group: "dom", baseline: true }, () => {
  const container = document.createElement("div");
  document.body.appendChild(container);

  // Vanilla DOM manipulation
  for (let i = 0; i < 1000; i++) {
    const element = document.createElement("div");
    element.textContent = `Item ${i}`;
    container.appendChild(element);
  }

  // Cleanup
  document.body.removeChild(container);
});

bench("DocumentFragment manipulation", { group: "dom" }, () => {
  const container = document.createElement("div");
  document.body.appendChild(container);

  // Using DocumentFragment (should be faster)
  const fragment = document.createDocumentFragment();
  for (let i = 0; i < 1000; i++) {
    const element = document.createElement("div");
    element.textContent = `Item ${i}`;
    fragment.appendChild(element);
  }
  container.appendChild(fragment);

  // Cleanup
  document.body.removeChild(container);
});
```

## Continuous Performance Monitoring

Integrate benchmarks into your CI/CD pipeline:

```typescript
// performance-regression.bench.ts
import { bench, runBenchmarks, getBenchmarks } from "@inspatial/kit/test";

bench("Critical algorithm", { baseline: true }, () => {
  return criticalAlgorithm(testData);
});

bench("Optimized algorithm", () => {
  return optimizedAlgorithm(testData);
});

// Run benchmarks and check for regressions
async function checkPerformanceRegression() {
  const results = await runBenchmarks();

  // Define performance thresholds
  const PERFORMANCE_THRESHOLD = 100; // milliseconds
  const REGRESSION_THRESHOLD = 1.2; // 20% slower is considered regression

  for (const result of results) {
    // Check absolute performance
    if (result.duration > PERFORMANCE_THRESHOLD) {
      console.warn(
        `⚠️ ${result.name} exceeded threshold: ${result.duration}ms > ${PERFORMANCE_THRESHOLD}ms`
      );
    }

    // Check against baseline (you might load this from a file)
    const baselineDuration = loadBaselinePerformance(result.name);
    if (baselineDuration) {
      const regressionRatio = result.duration / baselineDuration;
      if (regressionRatio > REGRESSION_THRESHOLD) {
        console.error(
          `❌ ${result.name} performance regression: ${(
            regressionRatio * 100 -
            100
          ).toFixed(1)}% slower`
        );
      }
    }

    // Save current performance as new baseline
    saveBaselinePerformance(result.name, result.duration);
  }
}
```

## Best Practices

### 1. Warm-up Iterations

```typescript
// Always warm up before benchmarking
bench("Warmed up benchmark", (b) => {
  // Warm-up phase (not measured)
  for (let i = 0; i < 100; i++) {
    expensiveOperation();
  }

  // Actual benchmark
  b.start();
  expensiveOperation();
  b.end();
});
```

### 2. Consistent Test Data

```typescript
// Use consistent, realistic test data
const TEST_DATA = {
  smallDataset: generateTestData(100),
  mediumDataset: generateTestData(1000),
  largeDataset: generateTestData(10000),
};

bench("Process small dataset", () => {
  return processData([...TEST_DATA.smallDataset]); // Copy to avoid mutation
});
```

### 3. Memory Management

```typescript
// Be mindful of memory usage in benchmarks
bench("Memory-conscious benchmark", () => {
  const result = heavyComputation();

  // Clean up large objects if needed
  if (typeof result === "object" && result !== null) {
    // Process result immediately
    const summary = summarizeResult(result);
    return summary; // Return smaller object
  }

  return result;
});
```

### 4. Statistical Significance

```typescript
// Run multiple iterations for statistical significance
bench("Statistical benchmark", { iterations: 10 }, () => {
  return algorithmUnderTest(testData);
});

// Or manually track multiple runs
const benchmarkResults = [];
for (let i = 0; i < 10; i++) {
  bench(`Algorithm run ${i}`, () => {
    const start = performance.now();
    const result = algorithmUnderTest(testData);
    const end = performance.now();
    benchmarkResults.push(end - start);
    return result;
  });
}

// Analyze results after benchmarks complete
function analyzeStatistics() {
  const average =
    benchmarkResults.reduce((a, b) => a + b) / benchmarkResults.length;
  const variance =
    benchmarkResults.reduce((acc, val) => acc + Math.pow(val - average, 2), 0) /
    benchmarkResults.length;
  const standardDeviation = Math.sqrt(variance);

  console.log(`Average: ${average}ms`);
  console.log(`Standard Deviation: ${standardDeviation}ms`);
  console.log(
    `Coefficient of Variation: ${((standardDeviation / average) * 100).toFixed(
      2
    )}%`
  );
}
```

## Troubleshooting Performance Issues

### Identifying Bottlenecks

```typescript
// Profile different parts of your application
test("Performance profiling", async () => {
  mark("app-start");

  mark("init-start");
  await initializeApp();
  mark("init-end");

  mark("load-data-start");
  const data = await loadData();
  mark("load-data-end");

  mark("process-start");
  const result = await processData(data);
  mark("process-end");

  mark("render-start");
  await renderResult(result);
  mark("render-end");

  mark("app-end");

  // Measure each phase
  const phases = [
    { name: "Initialization", start: "init-start", end: "init-end" },
    { name: "Data Loading", start: "load-data-start", end: "load-data-end" },
    { name: "Processing", start: "process-start", end: "process-end" },
    { name: "Rendering", start: "render-start", end: "render-end" },
  ];

  phases.forEach((phase) => {
    const duration = measure(phase.name, phase.start, phase.end);
    console.log(`${phase.name}: ${duration}ms`);
  });

  const totalTime = measure("Total", "app-start", "app-end");
  console.log(`Total application time: ${totalTime}ms`);
});
```

### Memory Leak Detection

```typescript
// Detect memory leaks in long-running operations
test("Memory leak detection", async () => {
  const initialMemory = performance.memory?.usedJSHeapSize || 0;

  // Simulate long-running operation
  for (let i = 0; i < 1000; i++) {
    await simulateUserInteraction();

    // Force garbage collection periodically (if available)
    if (global.gc) {
      global.gc();
    }
  }

  const finalMemory = performance.memory?.usedJSHeapSize || 0;
  const memoryIncrease = finalMemory - initialMemory;

  console.log(`Memory increase: ${memoryIncrease} bytes`);

  // Memory increase should be minimal after GC
  expect(memoryIncrease).toBeLessThan(1024 * 1024); // Less than 1MB increase
});
```

## Integration with CI/CD

Create a performance monitoring script:

```typescript
// scripts/performance-check.ts
import { bench, runBenchmarks } from "@inspatial/kit/test";

// Define critical operations as benchmarks
bench("user_authentication", async () => {
  return await authenticateUser("test@example.com", "password");
});

bench("data_processing", () => processLargeDataset(testDataset));

bench("api_response", () => handleApiRequest(mockRequest));

async function runPerformanceChecks() {
  const results = await runBenchmarks();

  // Check against performance budgets
  const performanceBudgets: Record<string, number> = {
    user_authentication: 200, // 200ms max
    data_processing: 1000, // 1s max
    api_response: 100, // 100ms max
  };

  let allPassed = true;

  for (const result of results) {
    const budget = performanceBudgets[result.name];
    if (budget && result.duration > budget) {
      console.error(
        `❌ ${result.name}: ${result.duration.toFixed(
          2
        )}ms (budget: ${budget}ms)`
      );
      allPassed = false;
    } else {
      console.log(`✅ ${result.name}: ${result.duration.toFixed(2)}ms`);
    }
  }

  if (!allPassed) {
    // Use environment-aware exit if available; node shown for brevity
    // deno-lint-ignore no-explicit-any
    const g: any = globalThis as any;
    if (g.InZero?.exit) g.InZero.exit(1);
    else if (typeof process !== "undefined" && process.exit) process.exit(1);
    else throw new Error("Performance checks failed");
  }
}

if (import.meta.main) {
  await runPerformanceChecks();
}
```

Add to your CI pipeline:

```yaml
# .github/workflows/performance.yml
name: Performance Tests
on: [push, pull_request]

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Run performance benchmarks
        run: deno run --allow-all scripts/performance-check.ts

      - name: Run benchmark suite
        run: deno bench --json benchmarks/ > performance-results.json

      - name: Upload performance results
        uses: actions/upload-artifact@v3
        with:
          name: performance-results
          path: performance-results.json
```

Benchmarking is essential for building high-performance applications. Use these tools and techniques to measure, optimize, and maintain the performance of your InSpatial applications. Remember: you can't optimize what you don't measure!

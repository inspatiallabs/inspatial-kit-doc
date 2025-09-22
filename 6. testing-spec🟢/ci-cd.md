## CI/CD Integration

#### Automate your testing pipeline for reliable, continuous delivery

> New to the test APIs? See the [Core Concepts](./getting-started.md#core-concepts) to understand the test primitives used throughout these workflows.

You've built an amazing application and written comprehensive tests. Now it's time to automate everything so your tests run automatically whenever you make changes. CI/CD (Continuous Integration/Continuous Deployment) is like having a tireless assistant that checks your code, runs your tests, and deploys your application - all without you having to remember to do it manually.

InSpatial Test is designed to work seamlessly with all major CI/CD platforms. Whether you're using GitHub Actions, GitLab CI, Jenkins, or any other platform, you can set up automated testing that runs your entire test suite, generates reports, and even prevents broken code from reaching production.

> **Terminology:**
>
> - **CI (Continuous Integration)**: Automatically testing code changes as they're made
> - **CD (Continuous Deployment)**: Automatically deploying tested code to production
> - **Pipeline**: A series of automated steps that run when code changes
> - **Workflow**: The configuration that defines what happens in your pipeline
> - **Artifact**: Files generated during the pipeline (like test reports or builds)
> - **Matrix Testing**: Running tests across multiple environments simultaneously

## Quick Start

Let's start with a simple GitHub Actions workflow that runs your InSpatial tests:

```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Run tests
        run: deno test --allow-all

      - name: Run benchmarks
        run: deno test --allow-all **/*.bench.ts
```

This basic workflow will:

- Run on every push to main/develop branches
- Run on every pull request to main
- Set up Deno environment
- Run all your tests and benchmarks

## Platform-Specific Configurations

### GitHub Actions

GitHub Actions is one of the most popular CI/CD platforms. Here's a comprehensive setup:

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    # Run tests daily at 2 AM UTC
    - cron: "0 2 * * *"

env:
  DENO_VERSION: v1.x
  NODE_VERSION: "20"

jobs:
  # Test across multiple runtimes
  test-matrix:
    name: Test on ${{ matrix.runtime }}
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        runtime: [deno, node, bun]
        os: [ubuntu-latest, windows-latest, macos-latest]
        exclude:
          # Bun doesn't support Windows yet
          - runtime: bun
            os: windows-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Deno
        if: matrix.runtime == 'deno'
        uses: denoland/setup-deno@v1
        with:
          deno-version: ${{ env.DENO_VERSION }}

      - name: Setup Node.js
        if: matrix.runtime == 'node'
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Setup Bun
        if: matrix.runtime == 'bun'
        uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest

      - name: Install dependencies (Node.js)
        if: matrix.runtime == 'node'
        run: npm ci

      - name: Install dependencies (Bun)
        if: matrix.runtime == 'bun'
        run: bun install

      - name: Run unit tests
        run: |
          if [ "${{ matrix.runtime }}" = "deno" ]; then
            deno test --allow-all --coverage=coverage
          elif [ "${{ matrix.runtime }}" = "node" ]; then
            npm test
          else
            bun test
          fi
        shell: bash

      - name: Run integration tests
        run: |
          if [ "${{ matrix.runtime }}" = "deno" ]; then
            deno test --allow-all tests/integration/
          elif [ "${{ matrix.runtime }}" = "node" ]; then
            npm run test:integration
          else
            bun run test:integration
          fi
        shell: bash

      - name: Generate coverage report (Deno)
        if: matrix.runtime == 'deno'
        run: |
          deno coverage coverage --lcov --output=coverage.lcov

      - name: Upload coverage to Codecov
        if: matrix.runtime == 'deno' && matrix.os == 'ubuntu-latest'
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.lcov
          flags: unittests
          name: codecov-umbrella

  # Performance testing
  performance:
    name: Performance Tests
    runs-on: ubuntu-latest
    needs: test-matrix

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: ${{ env.DENO_VERSION }}

      - name: Run performance benchmarks
        run: |
          deno test --allow-all **/*.bench.ts --reporter=json > benchmark-results.json

      - name: Check performance regressions
        run: |
          deno run --allow-all scripts/check-performance.ts benchmark-results.json

      - name: Upload benchmark results
        uses: actions/upload-artifact@v3
        with:
          name: benchmark-results
          path: benchmark-results.json
          retention-days: 30

  # Security and quality checks
  quality:
    name: Code Quality
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: ${{ env.DENO_VERSION }}

      - name: Lint code
        run: deno lint

      - name: Format check
        run: deno fmt --check

      - name: Type check
        run: deno check **/*.ts

      - name: Security audit
        run: deno task audit
        continue-on-error: true

  # Build and deploy
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: [test-matrix, performance, quality]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    environment:
      name: production
      url: https://your-app.com

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: ${{ env.DENO_VERSION }}

      - name: Build application
        run: deno task build

      - name: Deploy to production
        run: deno task deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}

      - name: Run smoke tests
        run: deno test --allow-all tests/smoke/
```

### GitLab CI

GitLab CI uses a `.gitlab-ci.yml` file in your repository root:

```yaml
# .gitlab-ci.yml
stages:
  - test
  - quality
  - performance
  - deploy

variables:
  DENO_VERSION: "v1.x"

# Test across multiple runtimes
.test_template: &test_template
  stage: test
  before_script:
    - curl -fsSL https://deno.land/install.sh | sh
    - export PATH="$HOME/.deno/bin:$PATH"
  script:
    - deno test --allow-all --coverage=coverage
    - deno coverage coverage --lcov --output=coverage.lcov
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    paths:
      - coverage.lcov
    expire_in: 1 week
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

test:deno:
  <<: *test_template
  image: denoland/deno:latest

test:node:
  stage: test
  image: node:20
  before_script:
    - npm ci
  script:
    - npm test
    - npm run test:coverage
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Code quality checks
lint:
  stage: quality
  image: denoland/deno:latest
  script:
    - deno lint
    - deno fmt --check
    - deno check **/*.ts
  allow_failure: false

# Performance testing
performance:
  stage: performance
  image: denoland/deno:latest
  script:
    - deno test --allow-all **/*.bench.ts --reporter=json > benchmark-results.json
    - deno run --allow-all scripts/check-performance.ts benchmark-results.json
  artifacts:
    paths:
      - benchmark-results.json
    expire_in: 1 month
  only:
    - main
    - develop

# Deploy to staging
deploy:staging:
  stage: deploy
  image: denoland/deno:latest
  script:
    - deno task build
    - deno task deploy:staging
  environment:
    name: staging
    url: https://staging.your-app.com
  only:
    - develop

# Deploy to production
deploy:production:
  stage: deploy
  image: denoland/deno:latest
  script:
    - deno task build
    - deno task deploy:production
    - deno test --allow-all tests/smoke/
  environment:
    name: production
    url: https://your-app.com
  when: manual
  only:
    - main
```

### Jenkins Pipeline

For Jenkins, create a `Jenkinsfile` in your repository:

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        DENO_VERSION = 'v1.x'
        NODE_VERSION = '20'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup') {
            parallel {
                stage('Setup Deno') {
                    steps {
                        sh '''
                            curl -fsSL https://deno.land/install.sh | sh
                            export PATH="$HOME/.deno/bin:$PATH"
                            deno --version
                        '''
                    }
                }

                stage('Setup Node.js') {
                    steps {
                        sh '''
                            nvm install ${NODE_VERSION}
                            nvm use ${NODE_VERSION}
                            npm ci
                        '''
                    }
                }
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests (Deno)') {
                    steps {
                        sh '''
                            export PATH="$HOME/.deno/bin:$PATH"
                            deno test --allow-all --coverage=coverage
                            deno coverage coverage --lcov --output=coverage.lcov
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'coverage',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report'
                            ])
                        }
                    }
                }

                stage('Unit Tests (Node.js)') {
                    steps {
                        sh '''
                            nvm use ${NODE_VERSION}
                            npm test
                        '''
                    }
                }

                stage('Integration Tests') {
                    steps {
                        sh '''
                            export PATH="$HOME/.deno/bin:$PATH"
                            deno test --allow-all tests/integration/
                        '''
                    }
                }
            }
        }

        stage('Quality Checks') {
            parallel {
                stage('Lint') {
                    steps {
                        sh '''
                            export PATH="$HOME/.deno/bin:$PATH"
                            deno lint
                        '''
                    }
                }

                stage('Format Check') {
                    steps {
                        sh '''
                            export PATH="$HOME/.deno/bin:$PATH"
                            deno fmt --check
                        '''
                    }
                }

                stage('Type Check') {
                    steps {
                        sh '''
                            export PATH="$HOME/.deno/bin:$PATH"
                            deno check **/*.ts
                        '''
                    }
                }
            }
        }

        stage('Performance Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                sh '''
                    export PATH="$HOME/.deno/bin:$PATH"
                    deno test --allow-all **/*.bench.ts --reporter=json > benchmark-results.json
                    deno run --allow-all scripts/check-performance.ts benchmark-results.json
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'benchmark-results.json', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    export PATH="$HOME/.deno/bin:$PATH"
                    deno task build
                    deno task deploy
                '''
            }
            post {
                success {
                    sh '''
                        export PATH="$HOME/.deno/bin:$PATH"
                        deno test --allow-all tests/smoke/
                    '''
                }
            }
        }
    }

    post {
        always {
            // Clean up
            sh 'rm -rf coverage/ benchmark-results.json'
        }

        failure {
            // Notify team of failure
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. Check console output at ${env.BUILD_URL}",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }

        success {
            // Notify team of success
            slackSend (
                color: 'good',
                message: "✅ Build successful: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

### Azure DevOps

Create an `azure-pipelines.yml` file:

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - README.md
      - docs/*

pr:
  branches:
    include:
      - main

variables:
  DENO_VERSION: "v1.x"
  NODE_VERSION: "20.x"

stages:
  - stage: Test
    displayName: "Test Stage"
    jobs:
      - job: TestMatrix
        displayName: "Test Matrix"
        strategy:
          matrix:
            Deno_Ubuntu:
              imageName: "ubuntu-latest"
              runtime: "deno"
            Deno_Windows:
              imageName: "windows-latest"
              runtime: "deno"
            Deno_macOS:
              imageName: "macOS-latest"
              runtime: "deno"
            Node_Ubuntu:
              imageName: "ubuntu-latest"
              runtime: "node"

        pool:
          vmImage: $(imageName)

        steps:
          - task: NodeTool@0
            condition: eq(variables['runtime'], 'node')
            inputs:
              versionSpec: $(NODE_VERSION)
            displayName: "Setup Node.js"

          - bash: |
              curl -fsSL https://deno.land/install.sh | sh
              echo "##vso[task.prependpath]$HOME/.deno/bin"
            condition: eq(variables['runtime'], 'deno')
            displayName: "Setup Deno"

          - bash: npm ci
            condition: eq(variables['runtime'], 'node')
            displayName: "Install Node.js dependencies"

          - bash: |
              if [ "$(runtime)" = "deno" ]; then
                deno test --allow-all --coverage=coverage
                deno coverage coverage --lcov --output=coverage.lcov
              else
                npm test
              fi
            displayName: "Run tests"

          - task: PublishCodeCoverageResults@1
            condition: eq(variables['runtime'], 'deno')
            inputs:
              codeCoverageTool: "Cobertura"
              summaryFileLocation: "coverage.lcov"
            displayName: "Publish coverage results"

  - stage: Quality
    displayName: "Quality Checks"
    dependsOn: Test
    jobs:
      - job: QualityChecks
        displayName: "Code Quality"
        pool:
          vmImage: "ubuntu-latest"

        steps:
          - bash: |
              curl -fsSL https://deno.land/install.sh | sh
              echo "##vso[task.prependpath]$HOME/.deno/bin"
            displayName: "Setup Deno"

          - bash: deno lint
            displayName: "Lint code"

          - bash: deno fmt --check
            displayName: "Check formatting"

          - bash: deno check **/*.ts
            displayName: "Type check"

  - stage: Performance
    displayName: "Performance Tests"
    dependsOn: Quality
    condition: and(succeeded(), in(variables['Build.SourceBranch'], 'refs/heads/main', 'refs/heads/develop'))
    jobs:
      - job: PerformanceTests
        displayName: "Run Performance Tests"
        pool:
          vmImage: "ubuntu-latest"

        steps:
          - bash: |
              curl -fsSL https://deno.land/install.sh | sh
              echo "##vso[task.prependpath]$HOME/.deno/bin"
            displayName: "Setup Deno"

          - bash: |
              deno test --allow-all **/*.bench.ts --reporter=json > benchmark-results.json
              deno run --allow-all scripts/check-performance.ts benchmark-results.json
            displayName: "Run benchmarks"

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: "benchmark-results.json"
              artifactName: "benchmark-results"
            displayName: "Publish benchmark results"

  - stage: Deploy
    displayName: "Deploy"
    dependsOn: Performance
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProduction
        displayName: "Deploy to Production"
        environment: "production"
        pool:
          vmImage: "ubuntu-latest"

        strategy:
          runOnce:
            deploy:
              steps:
                - bash: |
                    curl -fsSL https://deno.land/install.sh | sh
                    echo "##vso[task.prependpath]$HOME/.deno/bin"
                  displayName: "Setup Deno"

                - bash: deno task build
                  displayName: "Build application"

                - bash: deno task deploy
                  displayName: "Deploy application"
                  env:
                    DEPLOY_TOKEN: $(DEPLOY_TOKEN)

                - bash: deno test --allow-all tests/smoke/
                  displayName: "Run smoke tests"
```

## Advanced CI/CD Patterns

### Parallel Testing Strategy

Run different types of tests in parallel to speed up your pipeline:

```yaml
# .github/workflows/parallel-testing.yml
name: Parallel Testing

on: [push, pull_request]

jobs:
  # Unit tests - fast and run first
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Run unit tests
        run: deno test --allow-all tests/unit/ --parallel
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: unit-test-results
          path: test-results.xml

  # Integration tests - slower but comprehensive
  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Run integration tests
        run: deno test --allow-all tests/integration/
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

  # End-to-end tests - slowest but most realistic
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Start application
        run: deno task start &
      - name: Wait for application
        run: |
          timeout 60 bash -c 'until curl -f http://localhost:8000/health; do sleep 2; done'
      - name: Run E2E tests
        run: deno test --allow-all tests/e2e/

  # Performance tests - run on specific conditions
  performance-tests:
    name: Performance Tests
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Run performance benchmarks
        run: deno test --allow-all **/*.bench.ts
      - name: Check performance regressions
        run: deno run --allow-all scripts/performance-check.ts

  # Combine results and proceed
  test-summary:
    name: Test Summary
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests, e2e-tests]
    if: always()
    steps:
      - name: Check test results
        run: |
          if [[ "${{ needs.unit-tests.result }}" == "failure" || 
                "${{ needs.integration-tests.result }}" == "failure" || 
                "${{ needs.e2e-tests.result }}" == "failure" ]]; then
            echo "❌ Some tests failed"
            exit 1
          else
            echo "✅ All tests passed"
          fi
```

### Multi-Environment Testing

Test your application across different environments:

```yaml
# .github/workflows/multi-environment.yml
name: Multi-Environment Testing

on: [push, pull_request]

jobs:
  test-environments:
    name: Test on ${{ matrix.environment }}
    runs-on: ubuntu-latest

    strategy:
      matrix:
        environment: [development, staging, production]
        include:
          - environment: development
            config_file: config/dev.json
            database_url: postgresql://localhost:5432/dev
          - environment: staging
            config_file: config/staging.json
            database_url: postgresql://staging-db:5432/staging
          - environment: production
            config_file: config/prod.json
            database_url: postgresql://prod-db:5432/prod

    steps:
      - uses: actions/checkout@v4

      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Setup environment configuration
        run: |
          cp ${{ matrix.config_file }} config/current.json
          echo "Environment: ${{ matrix.environment }}"

      - name: Run environment-specific tests
        run: |
          deno test --allow-all tests/environments/${{ matrix.environment }}/
        env:
          ENVIRONMENT: ${{ matrix.environment }}
          DATABASE_URL: ${{ matrix.database_url }}
          CONFIG_FILE: config/current.json

      - name: Validate environment health
        run: |
          deno run --allow-all scripts/health-check.ts --env ${{ matrix.environment }}
```

### Conditional Testing

Run different tests based on what changed:

```yaml
# .github/workflows/conditional-testing.yml
name: Conditional Testing

on: [push, pull_request]

jobs:
  detect-changes:
    name: Detect Changes
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.changes.outputs.backend }}
      frontend: ${{ steps.changes.outputs.frontend }}
      docs: ${{ steps.changes.outputs.docs }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            backend:
              - 'src/backend/**'
              - 'tests/backend/**'
              - 'deno.json'
            frontend:
              - 'src/frontend/**'
              - 'tests/frontend/**'
              - 'package.json'
            docs:
              - 'docs/**'
              - '*.md'

  test-backend:
    name: Backend Tests
    runs-on: ubuntu-latest
    needs: detect-changes
    if: needs.detect-changes.outputs.backend == 'true'
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Run backend tests
        run: deno test --allow-all tests/backend/

  test-frontend:
    name: Frontend Tests
    runs-on: ubuntu-latest
    needs: detect-changes
    if: needs.detect-changes.outputs.frontend == 'true'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - name: Install dependencies
        run: npm ci
      - name: Run frontend tests
        run: npm run test:frontend

  validate-docs:
    name: Validate Documentation
    runs-on: ubuntu-latest
    needs: detect-changes
    if: needs.detect-changes.outputs.docs == 'true'
    steps:
      - uses: actions/checkout@v4
      - name: Check markdown links
        run: |
          npm install -g markdown-link-check
          find docs -name "*.md" -exec markdown-link-check {} \;
```

## Test Reporting and Artifacts

### Generating Test Reports

InSpatial Test provides built-in test reporting utilities for comprehensive CI/CD integration:

```typescript
// scripts/generate-test-report.ts
import { TestReporter } from "@inspatial/kit/test";

// Usage in CI/CD
if (import.meta.main) {
  const reporter = new TestReporter();

  // Add test results (this would typically be integrated with your test runner)
  reporter.addResult({
    name: "User authentication should work",
    status: "passed",
    duration: 45.2,
    suite: "Authentication",
    tags: ["auth", "critical"],
  });

  reporter.addResult({
    name: "Database connection should handle errors",
    status: "failed",
    duration: 123.7,
    error: "Connection timeout after 5000ms",
    suite: "Database",
    tags: ["db", "integration"],
  });

  reporter.addResult({
    name: "Optional feature test",
    status: "skipped",
    duration: 0,
    suite: "Features",
  });

  // Set coverage information (optional)
  reporter.setCoverage({
    lines: 850,
    functions: 120,
    branches: 200,
    statements: 900,
    linesPercent: 85.0,
    functionsPercent: 92.3,
    branchesPercent: 78.5,
    statementsPercent: 88.9,
  });

  // Add metadata
  reporter.setMetadata({
    buildNumber: "123",
    commitHash: "abc123def",
    branch: "main",
  });

  // Generate comprehensive report
  const report = reporter.generateReport();

  // Save in multiple formats
  await reporter.saveReport(report, "html"); // Beautiful HTML report
  await reporter.saveReport(report, "json"); // Machine-readable JSON
  await reporter.saveReport(report, "junit"); // JUnit XML for CI/CD tools

  console.log(`📊 Generated reports with ${report.summary.total} tests`);
  console.log(`✅ Success rate: ${report.summary.successRate.toFixed(1)}%`);
}
```

The built-in `TestReporter` provides:

- **Multiple output formats**: HTML, JSON, and JUnit XML
- **Rich test metadata**: Suites, tags, error details, and custom metadata
- **Coverage integration**: Automatic coverage reporting
- **Environment detection**: Runtime, platform, and CI information
- **Beautiful HTML reports**: Professional-looking reports with charts and filtering
- **CI/CD integration**: JUnit XML format for seamless CI/CD tool integration

### Coverage Reporting

Set up comprehensive coverage reporting:

```yaml
# .github/workflows/coverage.yml
name: Coverage Report

on: [push, pull_request]

jobs:
  coverage:
    name: Generate Coverage Report
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Run tests with coverage
        run: |
          deno test --allow-all --coverage=coverage_profile
          deno coverage coverage_profile --lcov --output=coverage.lcov
          deno coverage coverage_profile --html --output=coverage_html

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.lcov
          flags: unittests
          name: codecov-umbrella
          fail_ci_if_error: true

      - name: Upload coverage HTML report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: coverage_html/

      - name: Comment coverage on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const coverage = fs.readFileSync('coverage.lcov', 'utf8');
            const lines = coverage.split('\n');
            const linesCovered = lines.filter(line => line.startsWith('LH:')).length;
            const linesTotal = lines.filter(line => line.startsWith('LF:')).length;
            const coveragePercent = ((linesCovered / linesTotal) * 100).toFixed(2);

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## 📊 Coverage Report\n\n**Line Coverage:** ${coveragePercent}%\n\n[View detailed report](https://github.com/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId})`
            });
```

## Performance Monitoring in CI/CD

### Automated Performance Regression Detection

```typescript
// scripts/performance-check.ts
interface BenchmarkResult {
  name: string;
  duration: number;
  iterations: number;
  throughput: number;
}

interface PerformanceBaseline {
  [benchmarkName: string]: {
    duration: number;
    throughput: number;
    timestamp: string;
  };
}

class PerformanceMonitor {
  private baselinePath = "performance-baseline.json";
  private regressionThreshold = 0.2; // 20% slower is considered regression

  async loadBaseline(): Promise<PerformanceBaseline> {
    try {
      const content = await Deno.readTextFile(this.baselinePath);
      return JSON.parse(content);
    } catch {
      return {};
    }
  }

  async saveBaseline(baseline: PerformanceBaseline): Promise<void> {
    await Deno.writeTextFile(
      this.baselinePath,
      JSON.stringify(baseline, null, 2)
    );
  }

  async checkPerformance(results: BenchmarkResult[]): Promise<{
    passed: boolean;
    regressions: Array<{
      name: string;
      currentDuration: number;
      baselineDuration: number;
      regressionPercent: number;
    }>;
  }> {
    const baseline = await this.loadBaseline();
    const regressions = [];

    for (const result of results) {
      const baselineResult = baseline[result.name];

      if (baselineResult) {
        const regressionRatio = result.duration / baselineResult.duration;

        if (regressionRatio > 1 + this.regressionThreshold) {
          const regressionPercent = (regressionRatio - 1) * 100;

          regressions.push({
            name: result.name,
            currentDuration: result.duration,
            baselineDuration: baselineResult.duration,
            regressionPercent,
          });
        }
      }

      // Update baseline with current results
      baseline[result.name] = {
        duration: result.duration,
        throughput: result.throughput,
        timestamp: new Date().toISOString(),
      };
    }

    await this.saveBaseline(baseline);

    return {
      passed: regressions.length === 0,
      regressions,
    };
  }

  generateReport(results: BenchmarkResult[], regressions: any[]): string {
    let report = "# Performance Report\n\n";

    if (regressions.length === 0) {
      report += "✅ **No performance regressions detected**\n\n";
    } else {
      report += `❌ **${regressions.length} performance regression(s) detected**\n\n`;

      for (const regression of regressions) {
        report += `- **${
          regression.name
        }**: ${regression.regressionPercent.toFixed(1)}% slower\n`;
        report += `  - Current: ${regression.currentDuration.toFixed(2)}ms\n`;
        report += `  - Baseline: ${regression.baselineDuration.toFixed(
          2
        )}ms\n\n`;
      }
    }

    report += "## Benchmark Results\n\n";
    report += "| Benchmark | Duration | Throughput |\n";
    report += "|-----------|----------|------------|\n";

    for (const result of results) {
      report += `| ${result.name} | ${result.duration.toFixed(
        2
      )}ms | ${result.throughput.toFixed(0)} ops/sec |\n`;
    }

    return report;
  }
}

// Usage in CI/CD
if (import.meta.main) {
  const monitor = new PerformanceMonitor();

  // This would typically read from benchmark results file
  const results: BenchmarkResult[] = [
    {
      name: "Array operations",
      duration: 45.2,
      iterations: 1000,
      throughput: 22123,
    },
    {
      name: "String processing",
      duration: 78.9,
      iterations: 500,
      throughput: 6340,
    },
    {
      name: "Database queries",
      duration: 234.5,
      iterations: 100,
      throughput: 426,
    },
  ];

  const { passed, regressions } = await monitor.checkPerformance(results);
  const report = monitor.generateReport(results, regressions);

  console.log(report);

  // Write report for CI/CD artifacts
  await Deno.writeTextFile("performance-report.md", report);

  if (!passed) {
    console.error("❌ Performance regressions detected!");
    Deno.exit(1);
  } else {
    console.log("✅ Performance check passed!");
  }
}
```

## Best Practices for CI/CD

### 1. Fast Feedback Loops

```yaml
# Optimize for speed - run fastest tests first
jobs:
  quick-checks:
    name: Quick Checks (< 2 min)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
      - name: Lint and format check
        run: |
          deno lint
          deno fmt --check
      - name: Type check
        run: deno check **/*.ts
      - name: Unit tests
        run: deno test --allow-all tests/unit/ --parallel

  comprehensive-tests:
    name: Comprehensive Tests
    runs-on: ubuntu-latest
    needs: quick-checks # Only run if quick checks pass
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
      - name: Integration tests
        run: deno test --allow-all tests/integration/
      - name: E2E tests
        run: deno test --allow-all tests/e2e/
```

### 2. Fail Fast Strategy

```yaml
# Stop early if critical tests fail
jobs:
  critical-tests:
    name: Critical Path Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
      - name: Security tests
        run: deno test --allow-all tests/security/
      - name: Core functionality tests
        run: deno test --allow-all tests/core/
      # If these fail, don't waste resources on other tests

  extended-tests:
    name: Extended Test Suite
    runs-on: ubuntu-latest
    needs: critical-tests
    if: success() # Only run if critical tests pass
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1
      - name: Feature tests
        run: deno test --allow-all tests/features/
      - name: Performance tests
        run: deno test --allow-all **/*.bench.ts
```

### 3. Environment Parity

```yaml
# Ensure CI environment matches production
jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: denoland/deno:1.40.0 # Pin exact version

    services:
      postgres:
        image: postgres:15.2 # Same version as production
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7.0.8 # Same version as production
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - name: Run tests with production-like environment
        run: deno test --allow-all
        env:
          DATABASE_URL: postgresql://postgres:postgres@postgres:5432/test
          REDIS_URL: redis://redis:6379
          NODE_ENV: test
```

### 4. Artifact Management

```yaml
# Manage test artifacts effectively
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v1

      - name: Run tests
        run: |
          deno test --allow-all --reporter=json > test-results.json
          deno test --allow-all --coverage=coverage
          deno coverage coverage --html --output=coverage-html

      - name: Upload test results
        if: always() # Upload even if tests fail
        uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: test-results.json
          retention-days: 30

      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: coverage-html/
          retention-days: 7

      - name: Upload logs on failure
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: failure-logs
          path: |
            logs/
            *.log
          retention-days: 14
```

## Troubleshooting CI/CD Issues

### Common Problems and Solutions

#### 1. Flaky Tests

InSpatial provides built-in retry utilities for handling flaky tests:

```typescript
import {
  retryTest,
  retry,
  retryTestConditional,
  retryTestWithBackoff,
} from "@inspatial/kit/test";

// Simple retry with fixed delay
test("flaky network operation", async () => {
  await retryTest(
    async () => {
      const response = await fetch("https://api.example.com/data");
      expect(response.ok).toBe(true);
    },
    3,
    2000
  ); // 3 retries, 2 second delay
});

// Advanced retry with exponential backoff
test("database connection", async () => {
  await retry(
    async () => {
      const db = await connectToDatabase();
      expect(db.isConnected()).toBe(true);
    },
    {
      maxRetries: 5,
      exponentialBackoff: true,
      shouldRetry: (error) => error.message.includes("connection"),
      calculateDelay: (attempt, baseDelay) =>
        baseDelay * Math.pow(2, attempt - 1),
    }
  );
});

// Conditional retry based on error type
test("API with specific retry conditions", async () => {
  await retryTestConditional(
    async () => {
      const response = await fetch("https://api.example.com/data");
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    },
    (error) => {
      // Only retry on network errors or 5xx status codes
      return (
        error.message.includes("HTTP 5") || error.message.includes("network")
      );
    },
    3,
    2000
  );
});
```

#### 2. Environment-Specific Issues

InSpatial Test provides built-in environment utilities for conditional test execution:

```typescript
import {
  CI,
  skipInCI,
  onlyInCI,
  skipInEnvironments,
  onlyInEnvironments,
  skipOnPlatforms,
  onlyOnPlatforms,
  skipOnRuntimes,
  onlyOnRuntimes,
  skipWithoutEnvVars,
  skipInCIProviders,
  onlyInCIProviders,
} from "@inspatial/kit/test";

// Low-level env detection (if needed)
import { env, detectEnvironment } from "@in/vader";

// Skip tests in CI environment
test("local file system test", () => {
  if (skipInCI("File system not available in CI")) return;

  // Test local file operations
  const file = Deno.readTextFileSync("./local-file.txt");
  expect(file).toContain("expected content");
});

// Only run in CI environment
test("CI-specific integration test", () => {
  if (onlyInCI("Requires CI environment variables")) return;

  // Test CI-specific functionality
  const deployKey = env.get("DEPLOY_KEY");
  expect(deployKey).toBeDefined();
});

// Skip in specific environments
test("dangerous operation", () => {
  if (skipInEnvironments(["production", "staging"], "Not safe in production"))
    return;

  // Test that should only run in development
  performDangerousOperation();
});

// Platform-specific tests
test("unix-specific test", () => {
  if (skipOnPlatforms(["windows"], "Windows doesn't support this feature"))
    return;

  // Test Unix-specific functionality
  const result = execSync("ls -la");
  expect(result).toBeDefined();
});

// Require environment variables
test("API integration test", () => {
  if (skipWithoutEnvVars(["API_KEY", "API_SECRET"], "API credentials required"))
    return;

  // Test API integration
  const api = new ApiClient(
    Environment.getEnvVar("API_KEY"),
    Environment.getEnvVar("API_SECRET")
  );
  expect(api.isAuthenticated()).toBe(true);
});

// Enhanced CI detection
test("CI provider-specific test", () => {
  if (CI.isCI()) {
    const ciInfo = CI.getInfo();
    console.log(`Running in ${ciInfo.provider} CI`);

    // Provider-specific logic
    if (CI.isProvider("GitHub Actions")) {
      expect(ciInfo.buildNumber).toBeDefined();
    } else if (CI.isProvider("GitLab CI")) {
      expect(ciInfo.jobId).toBeDefined();
    }
  }
});

// Skip tests in specific CI providers
test("provider-specific test", () => {
  if (
    skipInCIProviders(
      ["Travis CI", "CircleCI"],
      "Not supported in these providers"
    )
  )
    return;

  // Test that only runs in GitHub Actions, GitLab CI, etc.
  performAdvancedOperation();
});

// Runtime-specific tests
test("runtime-aware test", () => {
  if (skipOnRuntimes(["webworker"], "DOM not available in web workers")) return;

  // Test that requires DOM
  expect(document).toBeDefined();
});

// Environment detection
test("environment-aware test", () => {
  if (env.isProduction()) {
    // Production-specific assertions
    expect(env.get("DEBUG")).toBe("false");
  } else if (env.isDevelopment()) {
    // Development-specific assertions
    expect(env.get("DEBUG")).toBe("true");
  }

  // Runtime detection using InSpatial's comprehensive environment detection
  const info = detectEnvironment();
  expect(["deno", "node", "bun", "dom", "ssr"]).toContain(info.type);

  // Feature detection
  if (info.features.hasDOM) {
    expect(document).toBeDefined();
  }

  if (info.features.isXR) {
    // XR-specific tests
    expect(info.features.isXR).toBe(true);
  }
});
```

#### 3. Resource Cleanup

InSpatial Test provides comprehensive resource cleanup utilities:

```typescript
import {
  TestCleanup,
  withCleanup,
  createTempDir,
  createTempFile,
  ResourceManager,
} from "@inspatial/kit/test";

// Manual cleanup management
test("database operations", async () => {
  const cleanup = new TestCleanup();

  try {
    const testDb = await createTestDatabase();
    cleanup.addCleanup(() => testDb.close());

    const testUser = await createTestUser(testDb);
    cleanup.addCleanup(() => deleteTestUser(testUser.id));

    // Run your tests
    expect(testUser.name).toBe("Test User");
  } finally {
    await cleanup.cleanup();
  }
});

// Automatic cleanup with decorator
test(
  "automatic cleanup",
  withCleanup(async (cleanup) => {
    const resource = await createResource();
    cleanup.addCleanup(() => resource.dispose());

    // Test logic here
    expect(resource.isActive()).toBe(true);

    // Cleanup happens automatically
  })
);

// Temporary file/directory management
test("file operations", async () => {
  const { path: tempDir, cleanup: cleanupDir } = await createTempDir("test-");
  const { path: tempFile, cleanup: cleanupFile } = await createTempFile(
    "initial content",
    ".txt"
  );

  try {
    // Use temporary resources
    await Deno.writeTextFile(`${tempDir}/test.txt`, "content");
    const content = await Deno.readTextFile(tempFile);
    expect(content).toBe("initial content");
  } finally {
    await cleanupDir();
    await cleanupFile();
  }
});

// Resource manager for complex scenarios
test("resource management", async () => {
  const resources = new ResourceManager();

  try {
    const db = await resources.manage(createDatabase(), (db) => db.close());
    const server = await resources.manage(startServer(), (server) =>
      server.stop()
    );
    const cache = await resources.manage(createCache(), (cache) =>
      cache.clear()
    );

    // Use resources
    expect(db.isConnected()).toBe(true);
    expect(server.isRunning()).toBe(true);
    expect(cache.size()).toBe(0);
  } finally {
    await resources.cleanup(); // Cleans up all managed resources
  }
});
```

## What's Next?

Now that you've mastered CI/CD integration with InSpatial Test, you might want to explore:

- [Performance Testing](./performance-testing.md) - Automated performance monitoring
- [Advanced Mocking](./advanced-mocking.md) - Complex test scenarios
- [Benchmarking](./benchmarking.md) - Comprehensive performance analysis
- [Getting Started](./getting-started.md) - Back to basics

CI/CD is essential for maintaining code quality and delivering reliable software. Start with simple workflows and gradually add more sophisticated testing strategies as your project grows. Remember: the best CI/CD pipeline is one that gives you confidence to deploy at any time!

> **Pro Tip:** Always test your CI/CD configuration locally first using tools like `act` for GitHub Actions or `gitlab-runner` for GitLab CI. This saves time and prevents broken pipelines from blocking your team.

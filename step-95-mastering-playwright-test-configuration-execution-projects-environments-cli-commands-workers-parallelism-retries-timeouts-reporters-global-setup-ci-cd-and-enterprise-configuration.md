# Playwright with TypeScript Mastery Bootcamp

# Step 95 — Mastering Playwright Test Configuration & Execution: Projects, Environments, CLI Commands, Workers, Parallelism, Retries, Timeouts, Reporters, Global Setup, CI/CD & Enterprise Configuration

## 1. Introduction

Playwright configuration controls how the complete automation suite runs.

A production-quality framework should define:

```text
Browser
Projects
Base URL
Test Directory
Workers
Parallelism
Retries
Timeouts
Reporters
Authentication
Environment
Artifacts
CI behavior
```

A typical architecture is:

```text
playwright.config.ts
        ↓
Environment
        ↓
Projects
        ↓
Workers
        ↓
Tests
        ↓
Reports / Artifacts
```

A good configuration should be:

- Predictable
- Maintainable
- Environment-aware
- CI-friendly
- Parallel-safe
- Secure
- Easy for the team to understand

---

## 2. Why Configuration Matters

Without centralized configuration, test files may contain:

```text
Hard-coded URLs
Timeouts
Browser settings
Credentials
Reporter settings
Retry settings
```

This creates maintenance problems.

Instead:

```text
Configuration
      ↓
Reusable Test Behavior
```

---

## 3. `playwright.config.ts`

A typical configuration starts with:

```ts
import {
  defineConfig,
  devices
} from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  use: {
    baseURL:
      'https://example.test',
    headless: true
  },

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices[
          'Desktop Chrome'
        ]
      }
    }
  ]
});
```

---

## 4. `defineConfig()`

Use:

```ts
defineConfig({})
```

to define Playwright configuration with TypeScript support and editor assistance.

It makes configuration easier to maintain.

---

## 5. `testDir`

Example:

```ts
testDir:
  './tests'
```

This tells Playwright where to find test files.

Possible structure:

```text
tests/
├── login.spec.ts
├── dashboard.spec.ts
└── users.spec.ts
```

---

## 6. `testMatch`

You can control which files are treated as tests.

Example:

```ts
testMatch:
  '**/*.spec.ts'
```

Possible naming conventions:

```text
*.spec.ts
*.test.ts
```

Use one consistent convention across the project.

---

## 7. `testIgnore`

Example:

```ts
testIgnore:
  '**/experimental/**'
```

This can exclude temporary or experimental tests.

Avoid using `testIgnore` to hide permanently failing tests.

---

## 8. `baseURL`

Example:

```ts
use: {
  baseURL:
    'https://example.test'
}
```

Then:

```ts
await page.goto(
  '/login'
);
```

instead of:

```ts
await page.goto(
  'https://example.test/login'
);
```

---

## 9. Environment-Based Base URL

Use:

```ts
const baseURL =
  process.env.BASE_URL;
```

Then:

```ts
use: {
  baseURL
}
```

This supports:

```text
LOCAL
DEV
QA
STAGING
```

without changing test files.

---

## 10. Environment Variables

Typical variables:

```text
BASE_URL
API_BASE_URL
TEST_USERNAME
TEST_PASSWORD
ADMIN_USERNAME
ADMIN_PASSWORD
CI
```

Never commit secrets.

Use environment variables or CI secret management.

---

## 11. `.env` Files

A local project may use:

```text
.env
.env.qa
.env.staging
```

Example:

```text
BASE_URL=https://qa.example.test
API_BASE_URL=https://qa-api.example.test
```

Do not commit sensitive `.env` files.

Use:

```gitignore
.env
.env.*
```

when appropriate for the project.

---

## 12. Loading Environment Variables

If using `dotenv`:

```ts
import dotenv from 'dotenv';

dotenv.config();
```

Then:

```ts
process.env.BASE_URL
```

can be used.

In CI, environment variables can instead be provided directly by the CI platform.

---

## 13. Headless Mode

Example:

```ts
use: {
  headless: true
}
```

Headless mode means the browser UI is not displayed.

Recommended for:

```text
CI
Regression
Scheduled automation
Large suites
```

---

## 14. Headed Mode

Example:

```ts
use: {
  headless: false
}
```

Useful for:

```text
Local debugging
Learning
Demo
Investigating UI behavior
```

Do not require headed mode in CI unless there is a specific reason.

---

## 15. `slowMo`

Example:

```ts
use: {
  launchOptions: {
    slowMo: 500
  }
}
```

This slows browser actions.

Use primarily for debugging.

Avoid using large `slowMo` values in CI.

---

## 16. Browser Projects

Projects allow the same tests to run with different configurations.

Example:

```ts
projects: [
  {
    name: 'chromium',
    use: {
      ...devices[
        'Desktop Chrome'
      ]
    }
  },
  {
    name: 'firefox',
    use: {
      ...devices[
        'Desktop Firefox'
      ]
    }
  },
  {
    name: 'webkit',
    use: {
      ...devices[
        'Desktop Safari'
      ]
    }
  }
]
```

---

## 17. Why Use Projects?

Projects are useful for:

```text
Different browsers
Different devices
Different environments
Different roles
Different authentication states
Setup dependencies
```

---

## 18. Browser Matrix

A common matrix:

```text
Chromium
Firefox
WebKit
```

Enterprise teams may choose a smaller matrix based on actual supported browsers.

Do not run every browser combination simply because it is available.

---

## 19. Device Projects

Playwright provides device descriptors.

Example:

```ts
{
  name: 'mobile',
  use: {
    ...devices[
      'Pixel 5'
    ]
  }
}
```

Useful for responsive UI testing.

---

## 20. Desktop vs Mobile Projects

Example:

```text
chromium-desktop
chromium-mobile
```

This allows the same test suite to validate responsive behavior.

However, mobile tests should focus on meaningful mobile-specific behavior rather than duplicating every desktop test.

---

## 21. Project Dependencies

Projects can depend on another project.

Example:

```text
setup
  ↓
chromium
firefox
webkit
```

Authentication setup can run first.

Then browser projects reuse the generated state.

---

## 22. Setup Project

Conceptually:

```ts
{
  name: 'setup',
  testMatch:
    /.*\.setup\.ts/
}
```

Then:

```ts
{
  name: 'chromium',
  dependencies: [
    'setup'
  ]
}
```

This creates a clean authentication/setup workflow.

---

## 23. Project-Specific Configuration

Example:

```ts
{
  name: 'chromium',
  use: {
    ...devices[
      'Desktop Chrome'
    ],
    browserName:
      'chromium'
  }
}
```

Each project can override configuration.

---

## 24. Project-Specific Base URL

Projects can target different environments.

Example concept:

```text
qa-chromium
staging-chromium
```

Each can use a different base URL.

Avoid multiplying projects unnecessarily.

---

## 25. Test Timeout

Example:

```ts
timeout:
  30_000
```

This controls the maximum time allowed for a test.

Timeouts should reflect realistic application behavior.

---

## 26. Expect Timeout

Example:

```ts
expect: {
  timeout:
    5_000
}
```

This controls assertion waiting time.

Example:

```ts
await expect(
  page.getByText(
    'Dashboard'
  )
).toBeVisible();
```

The assertion waits according to the configured expectation timeout.

---

## 27. Navigation Timeout

Example:

```ts
use: {
  navigationTimeout:
    30_000
}
```

This controls navigation-related waiting.

Do not solve slow application problems by endlessly increasing timeouts.

---

## 28. Action Timeout

Example:

```ts
use: {
  actionTimeout:
    10_000
}
```

This controls actions such as:

```text
click
fill
check
select
```

Use reasonable values.

---

## 29. Timeout Hierarchy

Think about:

```text
Test Timeout
   ├── Actions
   ├── Assertions
   └── Navigation
```

Different timeout settings serve different purposes.

Avoid configuring every timeout to extremely high values.

---

## 30. Global Timeout

A global timeout can limit the complete test run.

Use it carefully.

Example concept:

```ts
globalTimeout:
  60 * 60 * 1000
```

This can prevent a CI run from continuing indefinitely.

---

## 31. Retries

Example:

```ts
retries:
  process.env.CI
    ? 2
    : 0
```

A common strategy is:

```text
Local → 0 retries
CI → limited retries
```

Retries should not hide flaky tests.

---

## 32. Why Retries Can Be Dangerous

Suppose:

```text
Test fails
 ↓
Retry
 ↓
Pass
```

The build may appear green even though the test is unstable.

Track retries and investigate repeated failures.

---

## 33. Flaky Test Detection

A mature CI system should monitor:

```text
Passed
Failed
Retried
Flaky
Skipped
```

A test that passes only after retry should receive attention.

---

## 34. Workers

Workers control parallel test execution.

Example:

```ts
workers:
  4
```

The suite can run multiple tests concurrently.

---

## 35. Workers and CPU

Too many workers can cause:

```text
CPU saturation
Memory pressure
Database overload
API rate limits
Browser instability
```

More workers do not always mean faster execution.

---

## 36. CI Workers

A common approach:

```ts
workers:
  process.env.CI
    ? 2
    : undefined
```

The exact value should be based on CI resources.

---

## 37. Fully Parallel

Example:

```ts
fullyParallel:
  true
```

This allows tests to be scheduled more aggressively in parallel.

Only enable broad parallelism when tests are properly isolated.

---

## 38. Parallel Test Isolation

Tests should not depend on:

```text
Shared users
Shared orders
Shared browser state
Shared mutable files
Shared global variables
```

Use:

```text
Unique data
Isolated contexts
Fixtures
Worker-safe resources
```

---

## 39. Serial Tests

Some workflows may require serial execution.

However, serializing tests is usually not the best solution to state-sharing problems.

Prefer redesigning tests for isolation.

---

## 40. Sharding

Large suites can be split across CI machines.

Conceptually:

```text
Shard 1 → Tests 1-100
Shard 2 → Tests 101-200
Shard 3 → Tests 201-300
```

Playwright supports sharding through CLI options.

Example:

```bash
npx playwright test   --shard=1/3
```

Another machine:

```bash
npx playwright test   --shard=2/3
```

---

## 41. Why Sharding Matters

Sharding reduces total CI wall-clock time.

Example:

```text
One machine
 ↓
60 minutes

Three shards
 ↓
Approximately 20+ minutes
```

Actual performance depends on test balance and infrastructure.

---

## 42. CLI: Run All Tests

```bash
npx playwright test
```

Runs the configured test suite.

---

## 43. CLI: Run a Specific File

```bash
npx playwright test   tests/login.spec.ts
```

Useful for focused debugging.

---

## 44. CLI: Run a Test by Name

```bash
npx playwright test   -g "login successfully"
```

This runs tests matching the provided expression.

---

## 45. CLI: Run a Project

```bash
npx playwright test   --project=chromium
```

Useful when debugging a specific browser configuration.

---

## 46. CLI: Headed Mode

```bash
npx playwright test   --headed
```

Useful for local debugging.

---

## 47. CLI: Debug Mode

```bash
npx playwright test   --debug
```

This enables Playwright's debugging workflow.

---

## 48. CLI: UI Mode

```bash
npx playwright test   --ui
```

UI mode is useful for interactive local test exploration and debugging.

---

## 49. CLI: Trace

You can configure trace collection or use command-line options depending on the desired workflow.

A common configuration:

```ts
use: {
  trace:
    'on-first-retry'
}
```

This keeps artifacts focused on failed/retried tests.

---

## 50. Screenshots

Example:

```ts
use: {
  screenshot:
    'only-on-failure'
}
```

This reduces unnecessary artifact generation.

---

## 51. Video

Example:

```ts
use: {
  video:
    'retain-on-failure'
}
```

Video can be useful for debugging but consumes storage.

Use selectively in CI.

---

## 52. Trace Strategy

Common strategy:

```text
Local
 → off/on-demand

CI
 → on-first-retry
```

This balances debugging capability and storage cost.

---

## 53. Reporters

Playwright supports multiple reporter formats.

Examples include:

```text
list
line
dot
html
json
junit
blob
```

Use one or more based on your reporting pipeline.

---

## 54. HTML Reporter

Example:

```ts
reporter: [
  [
    'html',
    {
      open:
        'never'
    }
  ]
]
```

HTML reports are useful for human investigation.

---

## 55. JSON Reporter

Example:

```ts
reporter: [
  [
    'json',
    {
      outputFile:
        'reports/results.json'
    }
  ]
]
```

JSON is useful for integrations and custom reporting.

---

## 56. JUnit Reporter

Example:

```ts
reporter: [
  [
    'junit',
    {
      outputFile:
        'reports/results.xml'
    }
  ]
]
```

JUnit XML is commonly consumed by CI systems.

---

## 57. Multiple Reporters

Example:

```ts
reporter: [
  [
    'html',
    {
      open:
        'never'
    }
  ],
  [
    'junit',
    {
      outputFile:
        'reports/results.xml'
    }
  ]
]
```

This gives:

```text
Human Report
+
CI Report
```

---

## 58. Reporter Strategy

A practical enterprise setup:

```text
HTML
 ↓
Developers / QA

JUnit
 ↓
CI Dashboard

JSON
 ↓
Custom Analytics
```

Only generate what the organization actually uses.

---

## 59. Output Directory

Example:

```ts
outputDir:
  'test-results'
```

Artifacts can include:

```text
screenshots
videos
traces
attachments
```

Clean old results as part of normal test execution or CI workspace management.

---

## 60. Report Directory

Keep reports organized:

```text
reports/
├── html/
├── junit/
└── json/
```

Avoid mixing test artifacts with source files.

---

## 61. Global Setup

Playwright can use global setup for suite-level preparation.

Possible use cases:

```text
Environment validation
Database preparation
Global authentication preparation
External service checks
```

However, prefer fixtures or setup projects when they provide better isolation and dependency management.

---

## 62. Global Teardown

Global teardown can clean suite-level resources.

Use it carefully.

A global teardown should not become responsible for cleaning up test-specific state that should be owned by fixtures.

---

## 63. Setup Project vs Global Setup

### Setup Project

Good for:

```text
Authentication state
Setup tests
Dependencies
Visible setup flow
```

### Global Setup

Good for:

```text
Suite-wide initialization
Infrastructure checks
Very global resources
```

Prefer the mechanism that provides the clearest lifecycle and reporting.

---

## 64. Project Dependencies vs Global Setup

Project dependencies have advantages when setup itself should appear as part of the Playwright project graph.

Example:

```text
setup
 ↓
chromium
firefox
webkit
```

This makes dependencies explicit.

---

## 65. `use` Configuration

Common settings:

```ts
use: {
  baseURL,
  headless,
  viewport,
  screenshot,
  video,
  trace,
  storageState
}
```

Keep common configuration centralized.

---

## 66. Viewport

Example:

```ts
use: {
  viewport: {
    width: 1440,
    height: 900
  }
}
```

Useful for consistent desktop tests.

---

## 67. Color Scheme

Example:

```ts
use: {
  colorScheme:
    'dark'
}
```

This can test theme-specific UI behavior.

---

## 68. Locale

Example:

```ts
use: {
  locale:
    'en-IN'
}
```

Useful for internationalization tests.

---

## 69. Time Zone

Example:

```ts
use: {
  timezoneId:
    'Asia/Kolkata'
}
```

Useful when application behavior depends on local time.

---

## 70. Geolocation

If required:

```ts
use: {
  geolocation: {
    latitude: 22.7196,
    longitude: 75.8577
  },
  permissions: [
    'geolocation'
  ]
}
```

Use only synthetic or intentionally configured test locations.

---

## 71. User Agent

A project can override user-agent behavior when testing compatibility.

Avoid unnecessary user-agent manipulation.

Prefer device descriptors when possible.

---

## 72. Browser Launch Options

Example:

```ts
use: {
  launchOptions: {
    slowMo: 100
  }
}
```

Other launch options can be configured where required.

Do not add browser flags without understanding their impact.

---

## 73. Ignore HTTPS Errors

Example:

```ts
use: {
  ignoreHTTPSErrors:
    true
}
```

This may be useful for local test environments with self-signed certificates.

Avoid disabling certificate verification in production-like environments unless explicitly required.

---

## 74. Extra HTTP Headers

Example:

```ts
use: {
  extraHTTPHeaders: {
    'x-test-mode':
      'true'
  }
}
```

Useful for controlled test environments.

Do not send test-only headers to production accidentally.

---

## 75. Browser Context Options

Playwright's `use` configuration can define context behavior.

Examples:

```text
storageState
viewport
locale
timezoneId
permissions
geolocation
colorScheme
extraHTTPHeaders
```

Choose only the settings required by the tests.

---

## 76. Test Tagging

Tests can be categorized using tags or naming conventions.

Conceptually:

```text
@smoke
@regression
@api
@admin
@critical
```

Then CLI filtering can run targeted groups depending on the tagging approach used by the project.

---

## 77. Smoke Test Project

A project can target smoke tests:

```text
Smoke
 ↓
Critical Login
Critical Dashboard
Critical API
```

This can run quickly after every deployment.

---

## 78. Regression Project

Regression can include:

```text
All functional tests
Cross-browser tests
Role-based tests
Integration tests
```

Run it on a suitable schedule.

---

## 79. Environment + Project Matrix

A mature suite might use:

```text
QA
 ├── chromium
 ├── firefox
 └── webkit

STAGING
 └── chromium
```

Avoid a huge matrix unless business requirements justify it.

---

## 80. CI Configuration Strategy

A good CI pipeline can be:

```text
Install
 ↓
Lint / Type Check
 ↓
Setup Environment
 ↓
Authentication Setup
 ↓
Smoke Tests
 ↓
Full Tests
 ↓
Reports
 ↓
Artifacts
```

---

## 81. CI Environment Variables

Example:

```text
BASE_URL
API_BASE_URL
TEST_USERNAME
TEST_PASSWORD
CI
```

Configure them through the CI platform.

Never commit:

```text
password
token
private key
session cookie
```

---

## 82. CI Failure Artifacts

When tests fail, preserve:

```text
HTML Report
Trace
Screenshot
Video
JUnit XML
```

Use artifact retention policies appropriate to the organization.

---

## 83. CI Retry Strategy

Example:

```ts
retries:
  process.env.CI
    ? 2
    : 0
```

Combine retries with trace collection:

```ts
trace:
  'on-first-retry'
```

This gives useful debugging evidence.

---

## 84. CI Parallelism

Use:

```text
Workers
+
Shards
```

For example:

```text
CI Machine 1
 ├── Worker 1
 └── Worker 2

CI Machine 2
 ├── Worker 1
 └── Worker 2
```

Test data must remain isolated.

---

## 85. CI Resource Planning

Consider:

```text
CPU
Memory
Browser count
Database capacity
API capacity
Parallel tests
```

Too much parallelism can make the entire system slower.

---

## 86. Enterprise Configuration Layers

A useful configuration structure:

```text
Base Configuration
        ↓
Environment Configuration
        ↓
Project Configuration
        ↓
Test-Specific Overrides
```

Keep overrides minimal.

---

## 87. Avoid Configuration Duplication

Bad:

```text
Same baseURL in 10 files
Same timeout in 20 files
Same reporter in 5 scripts
```

Better:

```text
playwright.config.ts
```

contains shared configuration.

---

## 88. Config Helper Functions

If configuration becomes complex, use helper functions.

Example:

```ts
function getBaseURL() {
  return (
    process.env.BASE_URL ??
    'http://localhost:3000'
  );
}
```

Keep helpers small and predictable.

---

## 89. Configuration Validation

Validate required environment values.

Example:

```ts
if (
  !process.env.BASE_URL
) {
  throw new Error(
    'BASE_URL is required'
  );
}
```

Fail early rather than producing confusing navigation failures.

---

## 90. Configuration for Local vs CI

Example:

```ts
const isCI =
  !!process.env.CI;

export default defineConfig({
  workers:
    isCI ? 2 : undefined,

  retries:
    isCI ? 2 : 0,

  reporter:
    isCI
      ? 'junit'
      : 'list'
});
```

In real projects, combine CI reporting with human-friendly reports when needed.

---

## 91. Configuration for Debugging

Local debugging may use:

```text
headed
slowMo
trace
single worker
specific project
```

Example CLI:

```bash
npx playwright test   tests/login.spec.ts   --headed   --workers=1
```

---

## 92. Configuration for CI

CI usually prefers:

```text
headless
limited workers
retries
artifacts on failure
machine-readable reports
```

---

## 93. Configuration for Nightly Regression

Nightly runs can use:

```text
More browsers
More tests
More shards
Full reports
Extended artifact retention
```

This is different from a fast pull-request suite.

---

## 94. Configuration for Pull Requests

A PR pipeline should prioritize:

```text
Fast feedback
Smoke tests
Critical regression
Limited browser matrix
```

Then deeper tests can run after merge or on schedule.

---

## 95. Configuration for Release Validation

Release validation may use:

```text
Full regression
Cross-browser
Critical role tests
API integration
E2E workflows
```

---

## 96. Test Execution Strategy

A mature organization can use:

```text
Pull Request
 ↓
Smoke

Merge
 ↓
Regression

Nightly
 ↓
Full Cross-Browser

Release
 ↓
Complete Validation
```

---

## 97. Common CLI Commands

### Run all

```bash
npx playwright test
```

### Specific file

```bash
npx playwright test   tests/login.spec.ts
```

### Specific project

```bash
npx playwright test   --project=chromium
```

### Specific test

```bash
npx playwright test   -g "login"
```

### Headed

```bash
npx playwright test   --headed
```

### Debug

```bash
npx playwright test   --debug
```

### UI mode

```bash
npx playwright test   --ui
```

### Shard

```bash
npx playwright test   --shard=1/3
```

---

## 98. HTML Report

After a test run:

```bash
npx playwright show-report
```

This opens the HTML report.

Use it to inspect:

```text
Tests
Steps
Errors
Screenshots
Videos
Traces
```

---

## 99. Trace Viewer

A trace can help inspect:

```text
Actions
DOM snapshots
Network
Console
Screenshots
Timing
```

This is one of the most useful debugging tools for difficult failures.

---

## 100. Enterprise Configuration Example

```ts
import {
  defineConfig,
  devices
} from '@playwright/test';

const isCI =
  !!process.env.CI;

const baseURL =
  process.env.BASE_URL;

if (!baseURL) {
  throw new Error(
    'BASE_URL is required'
  );
}

export default defineConfig({

  testDir:
    './tests',

  fullyParallel:
    true,

  forbidOnly:
    isCI,

  retries:
    isCI ? 2 : 0,

  workers:
    isCI ? 2 : undefined,

  reporter: [
    [
      'html',
      {
        open:
          'never'
      }
    ],
    [
      'junit',
      {
        outputFile:
          'reports/results.xml'
      }
    ]
  ],

  use: {
    baseURL,

    headless:
      isCI,

    screenshot:
      'only-on-failure',

    video:
      'retain-on-failure',

    trace:
      'on-first-retry'
  },

  projects: [
    {
      name:
        'setup',
      testMatch:
        /.*\.setup\.ts/
    },

    {
      name:
        'chromium',
      use: {
        ...devices[
          'Desktop Chrome'
        ]
      },
      dependencies: [
        'setup'
      ]
    },

    {
      name:
        'firefox',
      use: {
        ...devices[
          'Desktop Firefox'
        ]
      },
      dependencies: [
        'setup'
      ]
    }
  ]
});
```

---

## 101. Common Configuration Mistakes

### Mistake 1

Hard-coded credentials.

### Mistake 2

Hard-coded environment URLs.

### Mistake 3

Unlimited retries.

### Mistake 4

Too many workers.

### Mistake 5

Running every browser for every PR.

### Mistake 6

Keeping video for every passing test.

### Mistake 7

No CI artifacts.

### Mistake 8

Using extremely large timeouts.

### Mistake 9

Using global setup for test-specific data.

### Mistake 10

Allowing `.only` to reach CI.

---

## 102. `forbidOnly`

Example:

```ts
forbidOnly:
  !!process.env.CI
```

This prevents accidental:

```ts
test.only(...)
```

from passing unnoticed in CI.

This is an important safety mechanism.

---

## 103. `fullyParallel`

Use:

```ts
fullyParallel:
  true
```

when the suite is designed for parallel execution.

If tests rely on shared mutable state, fix the isolation problem rather than simply disabling parallelism.

---

## 104. `maxFailures`

A suite can stop after a certain number of failures.

Conceptually:

```ts
maxFailures:
  process.env.CI
    ? 20
    : undefined
```

Useful when a large suite becomes clearly unhealthy.

Use carefully because early stopping can reduce failure coverage.

---

## 105. `preserveOutput`

Reporter and output behavior can be configured according to project needs.

Keep CI logs readable and artifacts organized.

---

## 106. Timeout Strategy

Recommended principle:

```text
Default timeout
 ↓
Reasonable

Specific slow operation
 ↓
Explicit override
```

Avoid:

```ts
timeout:
  10 * 60 * 1000
```

for every test.

---

## 107. Project-Specific Timeouts

If a particular project genuinely needs different settings:

```ts
{
  name: 'slow-report-tests',
  timeout: 60_000
}
```

Keep such exceptions explicit.

---

## 108. API and UI Projects

A large suite can separate projects:

```text
api
ui
integration
```

Example:

```text
api
 ↓
Fast API validation

ui
 ↓
Browser tests

integration
 ↓
API + UI workflows
```

---

## 109. Smoke and Regression Projects

Possible project organization:

```text
smoke
regression
```

The test selection strategy should be clear and maintainable.

Avoid creating dozens of tiny projects.

---

## 110. Enterprise Project Dependency Graph

```text
Environment Setup
       ↓
Authentication Setup
       ↓
API Tests
       ↓
UI Smoke
       ↓
UI Regression
       ↓
Cross-Browser
```

Not every project must depend on every other project.

Use dependencies only when a real dependency exists.

---

## 111. Configuration Review Checklist

Before merging configuration changes, check:

```text
[ ] URLs are environment-driven
[ ] Secrets are not committed
[ ] CI behavior is defined
[ ] Workers are reasonable
[ ] Retries are limited
[ ] Timeouts are reasonable
[ ] Artifacts are configured
[ ] Reports are useful
[ ] Projects have clear purposes
[ ] Setup dependencies are explicit
[ ] Parallelism is safe
[ ] `.only` is blocked in CI
```

---

## 112. Interview Questions

### Q1. What is `playwright.config.ts`?

The central configuration file that defines how Playwright tests are discovered and executed.

### Q2. What are projects?

Projects allow the same tests to run under different configurations such as browsers, devices, environments, or roles.

### Q3. What are workers?

Workers are parallel processes used to execute tests concurrently.

### Q4. What is sharding?

Sharding distributes a test suite across multiple CI machines or jobs.

### Q5. Why use retries?

To provide limited resilience against transient failures, especially in CI. Repeated retries should not be used to hide flaky tests.

### Q6. What is `forbidOnly`?

It prevents accidental focused tests such as `test.only()` from running in CI.

### Q7. What is `fullyParallel`?

It enables more aggressive parallel scheduling when the suite is properly isolated.

### Q8. What is the difference between test timeout and expect timeout?

Test timeout limits the complete test duration; expect timeout controls how long an assertion waits.

### Q9. Why use multiple reporters?

Different consumers may need different formats, such as human-readable HTML and CI-friendly JUnit.

### Q10. Why are project dependencies useful?

They make setup relationships explicit, such as authentication setup running before browser tests.

---

## 113. Hands-On Exercises

### Exercise 1 — Basic Configuration

Create a clean:

```text
playwright.config.ts
```

with:

```text
testDir
baseURL
browser project
reporter
timeouts
```

### Exercise 2 — Environment Configuration

Support:

```text
DEV
QA
STAGING
```

using environment variables.

### Exercise 3 — Browser Projects

Configure:

```text
Chromium
Firefox
WebKit
```

### Exercise 4 — Setup Project

Create an authentication setup project.

### Exercise 5 — Parallel Execution

Configure workers and verify that tests remain isolated.

### Exercise 6 — Retry Strategy

Configure:

```text
Local → 0
CI → 2
```

and analyze retry results.

### Exercise 7 — Artifact Strategy

Configure:

```text
Screenshot on failure
Video on failure
Trace on first retry
```

### Exercise 8 — Reporting

Configure:

```text
HTML
JUnit
```

### Exercise 9 — Sharding

Run the suite across:

```text
3 shards
```

### Exercise 10 — Enterprise Configuration

Create a complete configuration supporting:

```text
Environment
Projects
Authentication Setup
Browsers
Workers
Retries
Timeouts
Reports
Artifacts
CI/CD
```

---

## 114. Enterprise Configuration Challenge

Build a production-ready Playwright configuration supporting:

```text
LOCAL
QA
STAGING
```

and:

```text
Chromium
Firefox
WebKit
Mobile
```

Requirements:

1. Environment-based URLs.
2. Secure credentials.
3. Authentication setup project.
4. Project dependencies.
5. Controlled workers.
6. CI-specific retries.
7. Reasonable timeouts.
8. HTML reporter.
9. JUnit reporter.
10. Screenshots on failure.
11. Videos on failure.
12. Traces on first retry.
13. CI `.only` protection.
14. Test sharding.
15. Parallel-safe execution.
16. Smoke test strategy.
17. Regression test strategy.
18. API test project.
19. UI test project.
20. Clear configuration documentation.

---

## 115. Final Enterprise Architecture

```text
                         playwright.config.ts
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
              Environment                    Projects
                    │                           │
          ┌─────────┼─────────┐          ┌──────┼──────┐
          │         │         │          │      │      │
         DEV        QA      STAGING     API     UI   Mobile
                                              │
                                      ┌───────┴───────┐
                                      │               │
                                   Workers         Shards
                                      │               │
                                      └───────┬───────┘
                                              │
                                            Tests
                                              │
                                ┌─────────────┼─────────────┐
                                │             │             │
                              Trace       Screenshot      Video
                                │             │             │
                                └─────────────┼─────────────┘
                                              │
                                           Reports
                                              │
                                   HTML / JUnit / JSON
                                              │
                                             CI
```

This architecture provides centralized control over test execution while supporting multiple environments, browsers, workers, artifacts, reporting, and CI pipelines.

---

## 116. Final Checklist

```text
[ ] playwright.config.ts created
[ ] testDir configured
[ ] testMatch understood
[ ] baseURL environment-driven
[ ] Environment variables configured
[ ] Headless/headed understood
[ ] Browser projects configured
[ ] Device projects understood
[ ] Project dependencies configured
[ ] Setup project configured
[ ] Test timeout configured
[ ] Expect timeout configured
[ ] Action timeout understood
[ ] Navigation timeout understood
[ ] Retries configured
[ ] Workers configured
[ ] Parallelism validated
[ ] Sharding understood
[ ] HTML reporter configured
[ ] JUnit reporter configured
[ ] Screenshot strategy configured
[ ] Video strategy configured
[ ] Trace strategy configured
[ ] CI `.only` protection enabled
[ ] Global setup understood
[ ] Global teardown understood
[ ] CI environment configured
[ ] Smoke strategy configured
[ ] Regression strategy configured
[ ] API/UI projects organized
[ ] Configuration documented
```

---

## 117. Key Takeaways

- `playwright.config.ts` is the central control point for Playwright execution.
- Projects allow multiple browsers, devices, environments, roles, and test configurations.
- Environment variables keep URLs and credentials outside test code.
- Workers enable parallel execution.
- Sharding distributes tests across CI machines.
- Retries should be limited and monitored.
- Test, action, navigation, and expectation timeouts serve different purposes.
- HTML reports help humans investigate failures.
- JUnit and JSON reports help CI and automation systems consume results.
- Screenshots, videos, and traces should be collected strategically.
- Setup projects provide explicit dependency management.
- `forbidOnly` protects CI from accidental focused tests.
- Parallel execution requires strong test isolation.
- CI and local execution should have appropriate configuration differences.
- A mature enterprise suite separates smoke, regression, API, UI, and integration execution.
- Good configuration improves speed, reliability, security, and maintainability.

---

## 118. Next Step

**Step 96 — Mastering Playwright Debugging & Diagnostics: Trace Viewer, Screenshots, Video, Console Logs, Network Logs, Test Attachments, Error Analysis, VS Code Debugging & Enterprise Failure Investigation**

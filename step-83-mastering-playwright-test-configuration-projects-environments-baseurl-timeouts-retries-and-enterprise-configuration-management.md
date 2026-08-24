# Playwright with TypeScript Mastery Bootcamp

# Step 83 — Mastering Playwright Test Configuration, Projects, Environments, BaseURL, Timeouts, Retries & Enterprise Configuration Management

## 1. Introduction

A scalable Playwright framework needs a well-designed configuration strategy.

The `playwright.config.ts` file controls many important aspects of test execution, including:

- Test directory
- Base URL
- Browser projects
- Retries
- Workers
- Timeouts
- Reporters
- Screenshots
- Videos
- Traces
- Authentication state
- Parallel execution
- Environment-specific settings

A good configuration makes tests easier to run locally and in CI/CD.

---

## 2. The Playwright Configuration File

A basic configuration looks like:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'https://example.com'
  }
});
```

The configuration is normally stored as:

```text
playwright.config.ts
```

---

## 3. Why Configuration Matters

Without centralized configuration, tests may contain repeated values:

```ts
await page.goto('https://example.com/login');
await page.goto('https://example.com/dashboard');
```

With `baseURL`:

```ts
await page.goto('/login');
await page.goto('/dashboard');
```

Configuration centralizes environment-specific behavior.

---

## 4. `testDir`

`testDir` specifies where Playwright should find tests.

```ts
export default defineConfig({
  testDir: './tests'
});
```

Example structure:

```text
project/
├── tests/
│   ├── login.spec.ts
│   ├── admin.spec.ts
│   └── dashboard.spec.ts
└── playwright.config.ts
```

---

## 5. `testMatch`

`testMatch` controls which files are recognized as tests.

Example:

```ts
testMatch: '**/*.spec.ts'
```

Another example:

```ts
testMatch: /.*\.spec\.ts/
```

This is useful when a project contains different test file types.

---

## 6. `testIgnore`

Use `testIgnore` to exclude files or directories.

```ts
testIgnore: '**/experimental/**'
```

This can prevent unfinished or special-purpose tests from running in normal execution.

---

## 7. `baseURL`

Example:

```ts
use: {
  baseURL: 'https://example.com'
}
```

Now:

```ts
await page.goto('/login');
```

is resolved against the configured base URL.

This is especially useful for environment switching.

---

## 8. Environment-Based Base URL

A common enterprise approach is:

```ts
const baseURL =
  process.env.BASE_URL ??
  'http://localhost:3000';
```

Then:

```ts
export default defineConfig({
  use: {
    baseURL
  }
});
```

Run with:

```bash
BASE_URL=https://qa.example.com npx playwright test
```

On Windows PowerShell:

```powershell
$env:BASE_URL="https://qa.example.com"
npx playwright test
```

---

## 9. Environment Variables

Environment variables can control:

```text
BASE_URL
API_BASE_URL
TEST_USERNAME
TEST_PASSWORD
ENVIRONMENT
```

Example:

```ts
const environment =
  process.env.ENVIRONMENT ?? 'local';
```

Use environment variables for deployment-specific values.

Do not store secrets directly in source code.

---

## 10. Environment Configuration Strategy

A project may have:

```text
local
dev
qa
staging
```

Example:

```text
local  → http://localhost:3000
dev    → https://dev.example.com
qa     → https://qa.example.com
stage  → https://staging.example.com
```

The same tests should ideally run against different environments without code changes.

---

## 11. `use` Configuration

The `use` object contains browser-level defaults.

Example:

```ts
use: {
  baseURL,
  headless: true,
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'retain-on-first-retry'
}
```

These defaults apply to tests unless overridden.

---

## 12. Browser Projects

Playwright projects allow multiple configurations in one test run.

Example:

```ts
projects: [
  {
    name: 'chromium'
  },
  {
    name: 'firefox'
  },
  {
    name: 'webkit'
  }
]
```

This allows the same test suite to run across multiple browser engines.

---

## 13. Browser Device Configuration

Example:

```ts
import { devices } from '@playwright/test';

projects: [
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome']
    }
  }
]
```

For mobile:

```ts
{
  name: 'mobile-chrome',
  use: {
    ...devices['Pixel 5']
  }
}
```

---

## 14. Browser-Specific Projects

A common configuration:

```ts
projects: [
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome']
    }
  },
  {
    name: 'firefox',
    use: {
      ...devices['Desktop Firefox']
    }
  },
  {
    name: 'webkit',
    use: {
      ...devices['Desktop Safari']
    }
  }
]
```

This provides cross-browser coverage.

---

## 15. Project-Specific Configuration

Each project can have different settings.

Example:

```ts
projects: [
  {
    name: 'chromium',
    use: {
      browserName: 'chromium',
      headless: true
    }
  },
  {
    name: 'firefox',
    use: {
      browserName: 'firefox',
      headless: true
    }
  }
]
```

---

## 16. Role-Based Projects

Projects can also represent user roles.

Example:

```ts
projects: [
  {
    name: 'admin',
    use: {
      storageState:
        'playwright/.auth/admin.json'
    }
  },
  {
    name: 'employee',
    use: {
      storageState:
        'playwright/.auth/employee.json'
    }
  }
]
```

This allows role-based test execution.

---

## 17. Combining Browser and Role Projects

A larger framework may have:

```text
chromium-admin
chromium-employee
firefox-admin
firefox-employee
```

This creates a matrix:

```text
Browser
   ×
Role
```

Be careful with project explosion because the number of executions can grow quickly.

---

## 18. Test Timeout

The test timeout controls how long a test may run.

Example:

```ts
timeout: 30_000
```

This means 30 seconds.

Use:

```ts
timeout: 60_000
```

for workflows that genuinely require more time.

Avoid increasing the timeout globally just to hide slow tests.

---

## 19. Expect Timeout

Assertions can have their own timeout:

```ts
expect: {
  timeout: 5_000
}
```

Example:

```ts
await expect(
  page.getByText('Dashboard')
).toBeVisible();
```

If the assertion does not become true within the configured period, it fails.

---

## 20. Action Timeout

Action timeout controls operations such as:

```ts
click()
fill()
check()
selectOption()
```

Example:

```ts
use: {
  actionTimeout: 10_000
}
```

Use this carefully.

Playwright's built-in auto-waiting often means large action timeouts are unnecessary.

---

## 21. Navigation Timeout

Navigation can have a separate timeout:

```ts
use: {
  navigationTimeout: 30_000
}
```

This controls operations such as:

```ts
page.goto()
page.reload()
page.goBack()
```

---

## 22. Timeout Hierarchy

Timeouts can be configured at different levels:

```text
Global Test Timeout
       ↓
Test-Level Timeout
       ↓
Action Timeout
       ↓
Assertion Timeout
```

Use the narrowest timeout that represents the actual requirement.

---

## 23. Test-Level Timeout

A specific test can override the global timeout:

```ts
test(
  'long workflow',
  async ({ page }) => {
    // test
  },
  {
    timeout: 60_000
  }
);
```

This is better than increasing the timeout for the entire suite.

---

## 24. `test.setTimeout()`

You can also configure timeout inside a test:

```ts
test('long workflow', async ({ page }) => {
  test.setTimeout(60_000);

  // test steps
});
```

Use this when a specific test legitimately needs more time.

---

## 25. Retries

Retries allow failed tests to be executed again.

Example:

```ts
retries: 2
```

This means Playwright may retry a failed test up to two times.

Retries should not be used to hide flaky tests.

---

## 26. CI-Specific Retries

A common strategy:

```ts
retries: process.env.CI ? 2 : 0
```

Local development:

```text
No retries
```

CI:

```text
Limited retries
```

This provides faster local feedback while allowing some resilience in CI.

---

## 27. Detecting CI

Playwright projects commonly use:

```ts
process.env.CI
```

Example:

```ts
const isCI = !!process.env.CI;
```

Then:

```ts
retries: isCI ? 2 : 0
```

---

## 28. Workers

Workers control parallel test execution.

Example:

```ts
workers: 4
```

You can also use:

```ts
workers: process.env.CI ? 2 : undefined
```

The ideal number depends on:

- CPU
- Memory
- Application capacity
- CI environment
- Test isolation

---

## 29. `fullyParallel`

Example:

```ts
fullyParallel: true
```

This allows tests in files to run in parallel where Playwright's execution model permits it.

Parallel execution can significantly reduce runtime.

However, tests must be designed for isolation.

---

## 30. Parallel Execution Risks

Parallel tests can conflict through:

- Shared users
- Shared database records
- Shared files
- Fixed download paths
- Shared authentication state
- Global application state

A scalable framework should use unique data and isolated resources.

---

## 31. Reporter Configuration

Example:

```ts
reporter: [
  ['html'],
  ['list']
]
```

Common reporters include:

```text
list
line
dot
html
json
junit
```

Use reporters according to local and CI reporting requirements.

---

## 32. HTML Reporter

Example:

```ts
reporter: [['html']]
```

Generate and view the report with:

```bash
npx playwright show-report
```

HTML reports are useful for local debugging.

---

## 33. CI Reporters

CI pipelines may benefit from:

```ts
reporter: [
  ['list'],
  ['junit', {
    outputFile:
      'test-results/results.xml'
  }]
]
```

JUnit results can often be consumed by CI systems.

---

## 34. Screenshot Configuration

Example:

```ts
use: {
  screenshot: 'only-on-failure'
}
```

Possible strategies include:

```text
off
on
only-on-failure
```

Capturing screenshots only on failure can reduce unnecessary artifacts.

---

## 35. Video Configuration

Example:

```ts
use: {
  video: 'retain-on-failure'
}
```

This can help diagnose difficult failures.

Video artifacts can increase storage requirements, so use them intentionally.

---

## 36. Trace Configuration

A common setting is:

```ts
use: {
  trace: 'retain-on-first-retry'
}
```

This provides detailed diagnostics when a test fails and is retried.

Trace data can include sensitive application information, so protect CI artifacts.

---

## 37. Headless vs Headed

Headless:

```ts
use: {
  headless: true
}
```

Headed:

```ts
use: {
  headless: false
}
```

Headless mode is normally preferred in CI.

Headed mode is useful for debugging locally.

---

## 38. Slow Motion

For debugging:

```ts
use: {
  launchOptions: {
    slowMo: 500
  }
}
```

Slow motion should generally be used only during debugging.

Do not use artificial delays as a synchronization strategy.

---

## 39. Browser Launch Options

Example:

```ts
use: {
  launchOptions: {
    headless: true
  }
}
```

Launch options affect browser startup behavior.

Use them only when a genuine browser-level configuration is required.

---

## 40. Global Setup vs Project Dependencies

Playwright supports global setup patterns, but project dependencies are often easier to reason about when setup should be represented as a test-like project.

Conceptually:

```text
Setup Project
      ↓
Dependent Projects
```

This can make setup artifacts and execution order easier to inspect.

---

## 41. Environment-Specific Projects

You can define projects for environments:

```ts
projects: [
  {
    name: 'qa',
    use: {
      baseURL:
        'https://qa.example.com'
    }
  },
  {
    name: 'staging',
    use: {
      baseURL:
        'https://staging.example.com'
    }
  }
]
```

This allows the same tests to run against different environments.

---

## 42. Environment Selection Through Variables

Instead of hard-coding environments:

```ts
const environment =
  process.env.ENVIRONMENT ?? 'qa';

const baseURLs = {
  qa: 'https://qa.example.com',
  staging: 'https://staging.example.com'
};

export default defineConfig({
  use: {
    baseURL: baseURLs[environment]
  }
});
```

This is a common enterprise configuration pattern.

---

## 43. Configuration Validation

Configuration should fail early when required values are missing.

Example:

```ts
if (!process.env.BASE_URL) {
  throw new Error(
    'BASE_URL is required'
  );
}
```

Failing early is better than allowing dozens of tests to fail because the environment is incorrectly configured.

---

## 44. Environment Configuration Object

A cleaner approach is:

```ts
const config = {
  qa: {
    baseURL: 'https://qa.example.com',
    apiURL: 'https://qa-api.example.com'
  },
  staging: {
    baseURL: 'https://staging.example.com',
    apiURL: 'https://staging-api.example.com'
  }
};
```

Then:

```ts
const environment =
  process.env.ENVIRONMENT ?? 'qa';

const envConfig =
  config[environment];
```

This keeps environment information centralized.

---

## 45. Secrets vs Configuration

Not every environment value is a secret.

### Configuration

Examples:

```text
BASE_URL
API_BASE_URL
ENVIRONMENT
```

### Secrets

Examples:

```text
PASSWORD
API_KEY
ACCESS_TOKEN
CLIENT_SECRET
```

Secrets should be stored in secure secret-management systems or CI variables.

---

## 46. `use` Overrides

Global configuration:

```ts
use: {
  headless: true
}
```

A project can override:

```ts
projects: [
  {
    name: 'debug',
    use: {
      headless: false
    }
  }
]
```

A test can also override selected behavior where appropriate.

---

## 47. Project Dependencies

Example:

```ts
projects: [
  {
    name: 'setup',
    testMatch: /.*\.setup\.ts/
  },
  {
    name: 'chromium',
    dependencies: ['setup']
  }
]
```

This creates an explicit dependency relationship.

---

## 48. Project Selection

Run a specific project:

```bash
npx playwright test --project=chromium
```

Run another project:

```bash
npx playwright test --project=firefox
```

This is useful for targeted execution.

---

## 49. Test Tags and Configuration

Tags can help organize tests.

Example:

```ts
test(
  'critical login @smoke',
  async ({ page }) => {
    // test
  }
);
```

Then run matching tests using Playwright's filtering capabilities.

Configuration and tags can work together to create efficient execution strategies.

---

## 50. Global Test Setup Considerations

Before adding global setup, ask:

- Does every test require it?
- Can it be represented as a fixture?
- Can it be represented as a project dependency?
- Does it create shared mutable state?
- Does it make debugging harder?

Use the simplest lifecycle mechanism that satisfies the requirement.

---

## 51. Configuration for Local Development

A practical local configuration might use:

```ts
use: {
  headless: false,
  screenshot: 'only-on-failure',
  trace: 'on-first-retry'
}
```

This provides useful debugging without recording excessive artifacts.

---

## 52. Configuration for CI

A CI configuration may use:

```ts
use: {
  headless: true,
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'retain-on-first-retry'
},
retries: 2
```

Workers should be tuned according to CI resources.

---

## 53. Enterprise Configuration Architecture

A large project can use:

```text
config/
├── environments.ts
├── test-config.ts
├── browser-config.ts
└── secrets.ts
```

Then:

```text
playwright.config.ts
        ↓
Environment Config
        ↓
Browser Config
        ↓
Test Execution
```

Avoid duplicating configuration across many files.

---

## 54. Configuration and TypeScript

TypeScript can help catch configuration mistakes.

Example:

```ts
type Environment =
  | 'qa'
  | 'staging';

const environment:
  Environment =
  process.env.ENVIRONMENT as Environment;
```

For production-quality frameworks, validate external environment values at runtime rather than relying only on TypeScript casts.

---

## 55. Configuration and CI Matrix

CI can run a matrix such as:

```text
Chromium + QA
Firefox + QA
Chromium + Staging
```

This can be useful for release validation.

However, running every combination on every pull request may be unnecessarily expensive.

A staged strategy is often better.

---

## 56. Recommended Execution Strategy

### Pull Request

```text
Chromium
+
Smoke Tests
```

### Main Branch

```text
Chromium
+
Firefox
+
Regression
```

### Release

```text
Chromium
+
Firefox
+
WebKit
+
Critical Roles
+
Full Regression
```

This balances speed and coverage.

---

## 57. Configuration Debugging

To understand the active environment, print safe configuration information:

```ts
console.log({
  environment:
    process.env.ENVIRONMENT,
  baseURL:
    process.env.BASE_URL
});
```

Do not print secrets.

---

## 58. Common Configuration Mistakes

### Mistake 1 — Hard-coding environment URLs everywhere

Use `baseURL` and environment configuration.

### Mistake 2 — Increasing all timeouts

This hides performance and synchronization problems.

### Mistake 3 — Too many retries

Retries can hide flaky tests.

### Mistake 4 — Too many workers

This can overload the application or CI machine.

### Mistake 5 — Recording videos for every test

This increases storage and runtime overhead.

### Mistake 6 — Exposing secrets

Never print credentials or tokens.

### Mistake 7 — Creating too many projects

A large browser × role × environment matrix can become expensive.

---

## 59. Production-Quality Configuration Example

```ts
import {
  defineConfig,
  devices
} from '@playwright/test';

const isCI = !!process.env.CI;

const environment =
  process.env.ENVIRONMENT ?? 'qa';

const baseURLs = {
  qa: 'https://qa.example.com',
  staging: 'https://staging.example.com'
};

const baseURL =
  baseURLs[
    environment as keyof typeof baseURLs
  ];

if (!baseURL) {
  throw new Error(
    `Unsupported environment: ${environment}`
  );
}

export default defineConfig({
  testDir: './tests',

  fullyParallel: true,

  forbidOnly: isCI,

  retries: isCI ? 2 : 0,

  workers: isCI ? 2 : undefined,

  timeout: 30_000,

  expect: {
    timeout: 5_000
  },

  reporter: [
    ['html'],
    ['list']
  ],

  use: {
    baseURL,

    headless: isCI,

    screenshot: 'only-on-failure',

    video: 'retain-on-failure',

    trace: 'retain-on-first-retry'
  },

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome']
      }
    },
    {
      name: 'firefox',
      use: {
        ...devices['Desktop Firefox']
      }
    }
  ]
});
```

---

## 60. Interview Questions

### Q1. What is `playwright.config.ts`?

It is the central Playwright Test configuration file used to define test execution settings.

### Q2. What is `baseURL`?

It allows relative navigation paths such as:

```ts
page.goto('/login')
```

to be resolved against a configured base URL.

### Q3. What are Playwright projects?

Projects allow different test configurations, such as browsers, devices, roles, or environments, within the same configuration.

### Q4. How do you configure retries?

```ts
retries: 2
```

### Q5. How do you configure different retry behavior for CI?

```ts
retries: process.env.CI ? 2 : 0
```

### Q6. What is the difference between test timeout and expect timeout?

Test timeout limits the overall test duration. Expect timeout controls how long an assertion waits for its expected condition.

### Q7. What is `fullyParallel`?

It enables broader parallel execution of tests where Playwright's execution model permits it.

### Q8. Why use environment variables?

They allow environment-specific configuration and secure handling of deployment values without modifying test source code.

### Q9. Why should retries not be excessive?

Too many retries can hide genuine flaky-test problems.

### Q10. How do projects help multi-browser testing?

A project can define browser-specific settings, allowing the same test suite to run across Chromium, Firefox, and WebKit.

---

## 61. Hands-On Exercises

### Exercise 1 — Base Configuration

Create a `playwright.config.ts` with:

```text
testDir
baseURL
timeout
expect timeout
reporter
```

### Exercise 2 — Environment Switching

Support:

```text
qa
staging
```

through an `ENVIRONMENT` variable.

### Exercise 3 — Browser Projects

Configure:

```text
chromium
firefox
webkit
```

### Exercise 4 — CI Configuration

Configure:

```text
headless
retries
workers
trace
```

based on `CI`.

### Exercise 5 — Role Projects

Create:

```text
admin
employee
```

projects with different storage states.

### Exercise 6 — Project Dependencies

Create an authentication setup project and make browser projects depend on it.

### Exercise 7 — Failure Artifacts

Configure:

```text
screenshots
videos
traces
```

for useful failure diagnostics.

### Exercise 8 — Configuration Validation

Fail immediately when an unsupported environment is selected.

### Exercise 9 — Parallel Execution

Run tests with multiple workers and verify that test data remains isolated.

### Exercise 10 — Enterprise Configuration

Separate:

```text
environment configuration
browser configuration
test configuration
authentication configuration
```

while keeping the main Playwright configuration easy to understand.

---

## 62. Key Takeaways

- `playwright.config.ts` centralizes Playwright Test configuration.
- `testDir` defines where tests are located.
- `testMatch` controls which files are recognized.
- `baseURL` simplifies navigation and environment switching.
- Environment variables are useful for deployment-specific configuration.
- Projects support browsers, devices, roles, and environments.
- Timeouts should be realistic rather than excessively large.
- Retries should be limited and should not hide flaky tests.
- Workers control parallel execution.
- `fullyParallel` can reduce execution time when tests are isolated.
- Screenshots, videos, and traces provide valuable failure diagnostics.
- CI and local execution can use different settings.
- Project dependencies can manage setup workflows.
- Secrets must never be hard-coded or logged.
- Configuration should fail early when required values are invalid.
- Enterprise configuration should remain centralized, predictable, and maintainable.

---

## 63. Enterprise Configuration Challenge

Build a configuration system supporting:

```text
Environment:
  QA
  Staging

Browsers:
  Chromium
  Firefox
  WebKit

Roles:
  Admin
  Employee

Execution:
  Local
  CI
```

The final framework should be able to select the required environment, browser, role, retries, workers, authentication state, and failure artifacts without modifying test source code.

---

## 64. Next Step

**Step 84 — Mastering Playwright Test Hooks, BeforeAll, AfterAll, BeforeEach, AfterEach, Setup/Teardown & Enterprise Lifecycle Management**

# Playwright with TypeScript Mastery Bootcamp

# Step 94 — Mastering Playwright Test Fixtures & Advanced Dependency Injection: Built-in Fixtures, Custom Fixtures, Worker Fixtures, Test Fixtures, Fixture Composition, Scope & Enterprise Framework Design

## 1. Introduction

Fixtures are one of the most important features for building a scalable Playwright framework.

A fixture provides reusable setup and teardown logic to tests.

Instead of repeating:

```text
Create Page
Login
Create Test Data
Configure API
Open Dashboard
```

inside every test, fixtures can provide these dependencies automatically.

A typical enterprise fixture architecture looks like:

```text
Test
 ↓
Custom Fixture
 ↓
Page Object / API Client
 ↓
Application
```

Fixtures help with:

- Reusable setup
- Reusable teardown
- Dependency injection
- Authentication
- Page Objects
- API clients
- Test data
- Database utilities
- Multi-role testing
- Worker-level resources
- Parallel execution

---

## 2. What Is a Fixture?

A fixture is a reusable object or setup mechanism provided to a test.

Example:

```ts
test(
  'dashboard test',
  async ({ page }) => {
    await page.goto(
      '/dashboard'
    );
  }
);
```

Here:

```text
page
```

is a built-in Playwright fixture.

Playwright creates it and provides it to the test.

---

## 3. Dependency Injection

Fixtures are a form of dependency injection.

Instead of manually creating dependencies:

```ts
const page =
  await browser.newPage();
```

the test receives:

```ts
async ({ page }) => {}
```

The framework supplies the dependency.

Conceptually:

```text
Test
 ↓
Requests Dependency
 ↓
Fixture System
 ↓
Creates Dependency
 ↓
Injects Dependency
```

---

## 4. Why Dependency Injection Matters

Without dependency injection:

```ts
const browser =
  await chromium.launch();

const context =
  await browser.newContext();

const page =
  await context.newPage();
```

Tests become repetitive.

With fixtures:

```ts
test(
  'example',
  async ({ page }) => {
    // use page
  }
);
```

The framework handles setup and lifecycle.

---

## 5. Built-in Playwright Fixtures

Common built-in fixtures include:

```text
page
context
browser
browserName
request
playwright
```

Depending on the Playwright version and test configuration, additional fixtures may be available.

---

## 6. `page` Fixture

The `page` fixture provides a browser page.

Example:

```ts
test(
  'open application',
  async ({ page }) => {

    await page.goto('/');

    await expect(
      page
    ).toHaveTitle(
      /Application/
    );
  }
);
```

Each test normally receives an isolated page.

---

## 7. `context` Fixture

The browser context represents an isolated browser session.

Example:

```ts
test(
  'context example',
  async ({ context }) => {

    const cookies =
      await context.cookies();

    console.log(
      cookies.map(
        cookie =>
          cookie.name
      )
    );
  }
);
```

The context controls:

```text
Cookies
Storage
Permissions
Pages
Session
```

---

## 8. `browser` Fixture

The browser fixture represents the browser instance.

Example:

```ts
test(
  'browser example',
  async ({ browser }) => {

    const context =
      await browser.newContext();

    const page =
      await context.newPage();

    await page.goto('/');

    await context.close();
  }
);
```

Prefer normal `page` and `context` fixtures unless you specifically need additional contexts.

---

## 9. `request` Fixture

The request fixture provides API testing capability.

Example:

```ts
test(
  'API request',
  async ({ request }) => {

    const response =
      await request.get(
        '/api/users'
      );

    expect(
      response.status()
    ).toBe(200);
  }
);
```

---

## 10. Fixture Lifecycle

A fixture generally follows:

```text
Setup
 ↓
Test Uses Fixture
 ↓
Teardown
```

For example:

```text
Create Test Data
 ↓
Run Test
 ↓
Delete Test Data
```

The fixture system can manage both setup and teardown.

---

## 11. Basic Custom Fixture

Create a custom test object:

```ts
import {
  test as base
} from '@playwright/test';

type Fixtures = {
  testMessage: string;
};

export const test =
  base.extend<Fixtures>({
    testMessage: async (
      {},
      use
    ) => {

      await use(
        'Hello Playwright'
      );
    }
  });
```

Then:

```ts
test(
  'custom fixture',
  async ({
    testMessage
  }) => {

    console.log(
      testMessage
    );
  }
);
```

---

## 12. `use()` Function

The `use()` callback controls fixture execution.

Conceptually:

```ts
async ({ dependency }, use) => {

  // setup

  await use(
    fixtureValue
  );

  // teardown
}
```

Everything before `use()` is setup.

Everything after `use()` is teardown.

---

## 13. Fixture Teardown

Example:

```ts
const test =
  base.extend<Fixtures>({
    testMessage: async (
      {},
      use
    ) => {

      console.log(
        'Setup'
      );

      await use(
        'Hello'
      );

      console.log(
        'Teardown'
      );
    }
  });
```

Lifecycle:

```text
Setup
 ↓
Test
 ↓
Teardown
```

---

## 14. Fixture for Page Object

Example:

```ts
type Fixtures = {
  loginPage: LoginPage;
};

export const test =
  base.extend<Fixtures>({
    loginPage: async (
      { page },
      use
    ) => {

      const loginPage =
        new LoginPage(page);

      await use(
        loginPage
      );
    }
  });
```

Now tests can use:

```ts
test(
  'login',
  async ({
    loginPage
  }) => {

    await loginPage.login();
  }
);
```

---

## 15. Fixture Dependency

A custom fixture can depend on another fixture.

Example:

```ts
loginPage: async (
  { page },
  use
) => {
  const loginPage =
    new LoginPage(page);

  await use(
    loginPage
  );
}
```

Here:

```text
page
 ↓
LoginPage
 ↓
Test
```

The fixture system resolves the dependency.

---

## 16. Page Object Fixture Architecture

Recommended:

```text
Base Test
    ↓
Fixtures
    ↓
Page Objects
    ↓
Tests
```

Example fixtures:

```text
loginPage
dashboardPage
adminPage
profilePage
```

This makes tests cleaner.

---

## 17. Base Test Pattern

Example:

```ts
import {
  test as base
} from '@playwright/test';

type Fixtures = {
  loginPage:
    LoginPage;
  dashboardPage:
    DashboardPage;
};

export const test =
  base.extend<Fixtures>({
    loginPage:
      async (
        { page },
        use
      ) => {

        await use(
          new LoginPage(page)
        );
      },

    dashboardPage:
      async (
        { page },
        use
      ) => {

        await use(
          new DashboardPage(page)
        );
      }
  });

export {
  expect
} from '@playwright/test';
```

---

## 18. Using Custom Test

Import your custom test:

```ts
import {
  test,
  expect
} from '../fixtures/baseTest';
```

Then:

```ts
test(
  'dashboard',
  async ({
    dashboardPage
  }) => {

    await dashboardPage
      .open();

  }
);
```

---

## 19. Why Use a Base Test?

A base test centralizes:

```text
Fixtures
Authentication
Page Objects
API Clients
Common Utilities
```

This reduces duplicated setup across test files.

---

## 20. Fixture Composition

Fixtures can depend on other fixtures.

Example:

```text
page
 ↓
loginPage
 ↓
authenticatedPage
 ↓
dashboardPage
 ↓
test
```

This is fixture composition.

It allows complex dependencies to be assembled from smaller components.

---

## 21. Authenticated Page Fixture

Example:

```ts
authenticatedPage:
  async (
    { page },
    use
  ) => {

    await page.goto(
      '/login'
    );

    // perform login

    await use(page);
  }
```

Then:

```ts
test(
  'authenticated page',
  async ({
    authenticatedPage
  }) => {

    await authenticatedPage
      .goto('/dashboard');

  }
);
```

For large suites, storage state is often preferable to logging in inside every fixture.

---

## 22. Authentication Fixture with Storage State

A cleaner enterprise approach:

```text
Authentication Setup
 ↓
.auth/user.json
 ↓
Browser Context
 ↓
Custom Fixture
 ↓
Test
```

The fixture does not need to repeat login.

---

## 23. Worker Fixtures

Fixtures can have different scopes.

A worker-scoped fixture is shared within a worker.

Example:

```ts
type WorkerFixtures = {
  workerId:
    string;
};

export const test =
  base.extend<
    {},
    WorkerFixtures
  >({
    workerId: [
      async (
        {},
        use
      ) => {

        await use(
          `worker-${process.pid}`
        );
      },
      {
        scope:
          'worker'
      }
    ]
  });
```

Use worker fixtures for resources that can safely be shared by tests within the same worker.

---

## 24. Test-Scoped Fixtures

Test-scoped fixtures are created for each test.

Default fixture scope is generally:

```text
test
```

Example:

```ts
testData:
  async (
    {},
    use
  ) => {

    const data =
      createTestData();

    await use(data);
  }
```

Each test gets its own data.

---

## 25. Test Scope vs Worker Scope

### Test Scope

```text
Test A → Fixture A
Test B → Fixture B
Test C → Fixture C
```

### Worker Scope

```text
Worker
 ↓
Shared Fixture
 ↓
Test A
Test B
Test C
```

Worker scope improves performance but requires careful isolation design.

---

## 26. When to Use Worker Fixtures

Good candidates:

```text
Database connection pool
Expensive server setup
Static configuration
Worker-specific test account
Shared read-only resource
```

Avoid worker scope for mutable test data unless it is intentionally isolated.

---

## 27. Worker Fixture Risk

If tests modify shared worker state:

```text
Test A changes state
 ↓
Test B sees changed state
 ↓
Unexpected failure
```

This creates hidden dependencies.

Prefer test-scoped isolation for mutable resources.

---

## 28. Fixture Auto-Execution

Fixtures can be configured to run automatically.

Conceptually:

```ts
{
  auto: true
}
```

An auto fixture executes even if the test does not explicitly request it.

Use auto fixtures carefully.

---

## 29. When to Use Auto Fixtures

Useful for:

```text
Logging
Tracing
Environment validation
Global test diagnostics
Test metadata
Common cleanup
```

Avoid putting expensive browser operations into auto fixtures unless necessary.

---

## 30. Fixture Dependencies

Fixtures can form a dependency graph:

```text
browser
   ↓
context
   ↓
page
   ↓
loginPage
   ↓
dashboardPage
```

The framework resolves dependencies based on the fixture definitions.

---

## 31. Fixture Dependency Injection Diagram

```text
                Test
                 │
        ┌────────┼─────────┐
        │        │         │
      page     request   testData
        │
    loginPage
        │
 dashboardPage
```

The test requests only what it needs.

---

## 32. Fixture Teardown Order

When fixtures depend on one another:

```text
Setup:
A
 ↓
B
 ↓
C

Teardown:
C
 ↓
B
 ↓
A
```

Dependent resources should be cleaned up before their dependencies.

---

## 33. Fixture Error Handling

If fixture setup fails:

```text
Fixture Setup
 ↓
Error
 ↓
Test Does Not Continue
```

Make errors descriptive.

Example:

```ts
throw new Error(
  'Unable to create test user'
);
```

---

## 34. Fixture for Test Data

Example:

```ts
type Fixtures = {
  userData:
    UserData;
};

export const test =
  base.extend<Fixtures>({
    userData:
      async (
        {},
        use
      ) => {

        const data =
          buildUser();

        await use(data);
      }
  });
```

---

## 35. Fixture for API Client

Example:

```ts
type Fixtures = {
  usersApi:
    UsersApi;
};

export const test =
  base.extend<Fixtures>({
    usersApi:
      async (
        { request },
        use
      ) => {

        await use(
          new UsersApi(
            request
          )
        );
      }
  });
```

Then:

```ts
test(
  'create user',
  async ({
    usersApi
  }) => {

    const response =
      await usersApi
        .createUser({
          name:
            'Automation User'
        });

  }
);
```

---

## 36. API + Page Object Fixtures

An enterprise fixture set may contain:

```text
loginPage
dashboardPage
usersPage
usersApi
reportsApi
testData
```

This allows:

```text
API Setup
 ↓
UI Action
 ↓
API Validation
```

---

## 37. Fixture for Database Access

A database fixture can expose a test utility.

Conceptually:

```ts
db:
  async (
    {},
    use
  ) => {

    const connection =
      await createConnection();

    await use(
      connection
    );

    await connection.close();
  }
```

Do not create a database connection for every test if that is unnecessarily expensive.

A worker-scoped pool may be more appropriate when safely designed.

---

## 38. Fixture Cleanup

Example:

```ts
testData:
  async (
    {},
    use
  ) => {

    const data =
      await createData();

    try {

      await use(data);

    } finally {

      await deleteData(
        data.id
      );
    }
  }
```

This guarantees cleanup when possible.

---

## 39. Fixture and `try/finally`

For critical cleanup:

```ts
await use(data);
```

followed by:

```ts
try {
  await use(data);
} finally {
  await cleanup();
}
```

The cleanup code should be designed to execute even when the test fails.

---

## 40. Fixture Composition Example

```text
API Request
    ↓
Users API
    ↓
Test User
    ↓
Authenticated Page
    ↓
Dashboard Page
    ↓
Test
```

Each layer can be represented by a fixture.

---

## 41. Avoid Fixture Overengineering

Do not create fixtures for every small variable.

Bad:

```text
firstNameFixture
lastNameFixture
roleFixture
emailFixture
```

when the test can simply use a data factory.

Better:

```text
userData
```

containing all related fields.

---

## 42. Fixture Naming

Use clear names:

```text
loginPage
dashboardPage
adminPage
usersApi
testUser
authenticatedPage
adminContext
```

Avoid vague names:

```text
helper
thing
data1
object
common
```

---

## 43. Fixture File Organization

Recommended:

```text
fixtures/
├── base.fixture.ts
├── auth.fixture.ts
├── api.fixture.ts
├── data.fixture.ts
└── page.fixture.ts
```

For smaller projects, one base fixture file may be sufficient.

---

## 44. Enterprise Fixture Layer

Example:

```text
fixtures/
│
├── base.fixture.ts
│
├── auth.fixture.ts
│
├── page.fixture.ts
│
├── api.fixture.ts
│
└── data.fixture.ts
```

Dependencies:

```text
base
 ↓
auth
 ↓
page
 ↓
api
 ↓
data
```

Avoid unnecessary dependency chains.

---

## 45. Custom Test Types

TypeScript can strongly type custom fixtures.

Example:

```ts
type Fixtures = {
  usersPage:
    UsersPage;
  usersApi:
    UsersApi;
};
```

Then:

```ts
export const test =
  base.extend<Fixtures>({
    // ...
  });
```

This provides autocomplete and compile-time checking.

---

## 46. Fixture Type Safety

Without proper typing:

```ts
fixture.someUnknownProperty
```

may not be detected correctly.

With TypeScript:

```ts
type Fixtures = {
  usersPage:
    UsersPage;
};
```

the compiler understands the fixture API.

---

## 47. Extending Existing Fixtures

Use:

```ts
const test =
  base.extend<Fixtures>({
    // custom fixtures
  });
```

This preserves built-in Playwright fixtures.

For example:

```text
page
context
request
browser
```

remain available.

---

## 48. Re-Exporting `expect`

Common pattern:

```ts
export {
  expect
} from '@playwright/test';
```

Then test files can import both:

```ts
import {
  test,
  expect
} from '../fixtures/base.fixture';
```

This keeps test imports consistent.

---

## 49. Fixture-Based Page Object Example

```ts
type Fixtures = {
  usersPage:
    UsersPage;
};

export const test =
  base.extend<Fixtures>({
    usersPage:
      async (
        { page },
        use
      ) => {

        const usersPage =
          new UsersPage(page);

        await use(
          usersPage
        );
      }
  });
```

Test:

```ts
test(
  'users page',
  async ({
    usersPage
  }) => {

    await usersPage.open();

    await usersPage
      .verifyLoaded();

  }
);
```

---

## 50. Fixture-Based API Example

```ts
type Fixtures = {
  usersApi:
    UsersApi;
};

export const test =
  base.extend<Fixtures>({
    usersApi:
      async (
        { request },
        use
      ) => {

        const usersApi =
          new UsersApi(
            request
          );

        await use(
          usersApi
        );
      }
  });
```

---

## 51. Fixture-Based Authentication Example

```ts
type Fixtures = {
  authenticatedPage:
    Page;
};

export const test =
  base.extend<Fixtures>({
    authenticatedPage:
      async (
        { page },
        use
      ) => {

        await page.goto(
          '/dashboard'
        );

        await use(page);
      }
  });
```

In a real enterprise project, use the appropriate storage state rather than repeating UI login unnecessarily.

---

## 52. Role-Based Fixtures

Possible fixture structure:

```text
adminPage
managerPage
employeePage
```

Example:

```ts
adminPage:
  async (
    { browser },
    use
  ) => {

    const context =
      await browser.newContext({
        storageState:
          '.auth/admin.json'
      });

    const page =
      await context.newPage();

    await use(page);

    await context.close();
  }
```

This is useful for multi-role tests.

---

## 53. Multi-User Fixtures

For workflows requiring two users:

```text
employeePage
managerPage
```

Each can have a separate context.

Example:

```text
Employee Context
      ↓
Employee Page

Manager Context
      ↓
Manager Page
```

This prevents cookie/session interference.

---

## 54. Fixture Scope Strategy

Use:

```text
Test scope
```

for:

```text
Mutable test data
Page-specific objects
Test-specific resources
```

Use:

```text
Worker scope
```

for:

```text
Worker-specific resources
Expensive shared setup
Read-only configuration
```

Use:

```text
Auto fixtures
```

for:

```text
Cross-cutting diagnostics
```

---

## 55. Fixture Performance

Poor fixture:

```text
Every Test
 ↓
Create Database
 ↓
Create Browser
 ↓
Start Service
 ↓
Login
```

Better:

```text
Worker
 ↓
Shared Expensive Infrastructure

Test
 ↓
Isolated Test Data
```

Balance speed against isolation.

---

## 56. Fixture Isolation

The strongest enterprise rule is:

```text
Shared Infrastructure
+
Isolated Test State
```

Do not confuse shared infrastructure with shared mutable test data.

---

## 57. Fixture Parallelism

With parallel execution:

```text
Worker 1
 ├── Test A
 └── Test B

Worker 2
 ├── Test C
 └── Test D
```

Worker-scoped fixtures are instantiated per worker.

This can improve performance.

---

## 58. Worker-Specific Test Accounts

For systems that prevent concurrent login with the same account, a worker fixture can provide:

```text
Worker 1 → Account 1
Worker 2 → Account 2
Worker 3 → Account 3
```

This requires deliberate account allocation and cleanup.

---

## 59. Fixture Dependency Graph Design

Before creating many fixtures, draw:

```text
                Test
                 │
       ┌─────────┼─────────┐
       │         │         │
      Page      API      Data
       │         │
    AuthPage  UsersApi
```

Keep the graph understandable.

---

## 60. Circular Dependencies

Avoid:

```text
Fixture A
 ↓
Fixture B
 ↓
Fixture A
```

Circular dependencies make the framework difficult to reason about.

If this occurs, refactor the shared responsibility into another utility or fixture.

---

## 61. Fixture vs Utility

Use a fixture when the dependency needs:

```text
Lifecycle
Injection
Setup
Teardown
Other fixtures
```

Use a utility when it is simply:

```text
Pure transformation
Formatting
Calculation
Static helper
```

---

## 62. Fixture vs Page Object

Use Page Object for:

```text
UI behavior
Locators
Page actions
Page assertions
```

Use Fixture for:

```text
Creating
Configuring
Injecting
Managing lifecycle
```

They work together.

---

## 63. Fixture vs API Client

Use API Client for:

```text
HTTP endpoints
Request payloads
Response handling
```

Use Fixture for:

```text
Creating and injecting API client
Authentication
Lifecycle
```

---

## 64. Fixture vs Test Data Factory

Use a factory for:

```text
Generating data
```

Use a fixture when you need:

```text
Create data
 ↓
Use in test
 ↓
Cleanup data
```

---

## 65. Enterprise Dependency Injection Pattern

Recommended:

```text
Test
 ↓
Fixture
 ↓
Dependency
 ↓
Implementation
```

Example:

```text
Test
 ↓
usersApi fixture
 ↓
UsersApi
 ↓
APIRequestContext
```

---

## 66. Advanced Fixture Composition

A mature framework might use:

```text
API Request
    ↓
Authenticated API
    ↓
Users API
    ↓
Test User
    ↓
Authenticated Browser
    ↓
Users Page
    ↓
Test
```

Each dependency should have a single clear responsibility.

---

## 67. Fixture Configuration

Fixture options can control behavior.

Conceptually:

```ts
{
  scope: 'worker',
  auto: true
}
```

Other Playwright fixture configuration options can be used where supported.

Always consult the version of Playwright used by the project.

---

## 68. Fixture Options

Fixture options can expose configurable values.

Conceptually:

```ts
type Options = {
  environment:
    string;
};
```

Then tests can be configured for:

```text
QA
STAGING
DEV
```

without duplicating fixture logic.

---

## 69. Environment-Aware Fixtures

Example concept:

```ts
environment:
  async (
    {},
    use
  ) => {

    await use(
      process.env.TEST_ENV ||
      'qa'
    );
  }
```

Then:

```ts
test(
  'environment',
  async ({
    environment
  }) => {

    console.log(
      environment
    );
  }
);
```

---

## 70. Fixture Logging

Fixtures can log lifecycle events:

```ts
console.log(
  'Creating test user'
);
```

and:

```ts
console.log(
  'Deleting test user'
);
```

Avoid excessive logs in normal CI runs.

Use structured or debug-level logging when possible.

---

## 71. Fixture Diagnostics

Useful diagnostics include:

```text
Fixture name
Test name
Worker index
Environment
Resource ID
```

Never log:

```text
Password
Token
Cookie value
Secret
```

---

## 72. Fixture Error Messages

Bad:

```text
Error occurred
```

Better:

```text
Failed to create test user
Environment: QA
Role: Employee
```

Do not include sensitive information.

---

## 73. Fixture Cleanup Failures

Cleanup can fail because:

```text
API unavailable
Resource already deleted
Permission changed
Test environment unavailable
```

Decide whether cleanup failure should:

```text
Fail the test
Warn
Retry
Record diagnostics
```

based on the project's requirements.

---

## 74. Fixture Retry Considerations

Avoid blindly retrying fixture setup.

If:

```text
Create User
```

is not idempotent, retrying could create duplicates.

Prefer deterministic setup strategies.

---

## 75. Fixture and Test Retries

When Playwright retries a failed test:

```text
Attempt 1
 ↓
Fixture Setup
 ↓
Test Failure
 ↓
Teardown
 ↓
Attempt 2
 ↓
Fixture Setup Again
```

Fixtures should therefore be designed to work correctly across retries.

---

## 76. Fixture and Test Data Isolation

A retry must not accidentally reuse invalid state.

Good:

```text
Retry
 ↓
Fresh Test Data
```

Potentially risky:

```text
Retry
 ↓
Reuse Already Modified Data
```

Use unique or resettable test data.

---

## 77. Enterprise Fixture Anti-Patterns

Avoid:

```text
God Fixture
```

that creates everything:

```text
Login
Database
API
Users
Reports
Pages
Payments
```

This becomes hard to maintain.

Instead:

```text
Auth Fixture
API Fixture
Data Fixture
Page Fixture
```

compose only what is needed.

---

## 78. Another Anti-Pattern: Hidden Navigation

Avoid a fixture that secretly does:

```text
Create Page
 ↓
Login
 ↓
Navigate
 ↓
Open Dashboard
```

when tests are not expecting that behavior.

Fixtures should have predictable responsibilities.

---

## 79. Another Anti-Pattern: Global Mutable State

Avoid:

```ts
let currentUser;
let currentOrder;
let currentPage;
```

shared across tests.

Parallel tests can overwrite these values.

Use fixtures and test-scoped data instead.

---

## 80. Production-Quality Base Fixture

```ts
import {
  test as base,
  expect
} from '@playwright/test';

import {
  UsersPage
} from '../pages/UsersPage';

import {
  UsersApi
} from '../api/UsersApi';

type Fixtures = {
  usersPage:
    UsersPage;

  usersApi:
    UsersApi;
};

export const test =
  base.extend<Fixtures>({
    usersPage:
      async (
        { page },
        use
      ) => {

        const usersPage =
          new UsersPage(page);

        await use(
          usersPage
        );
      },

    usersApi:
      async (
        { request },
        use
      ) => {

        const usersApi =
          new UsersApi(
            request
          );

        await use(
          usersApi
        );
      }
  });

export { expect };
```

---

## 81. Production-Quality Test

```ts
import {
  test,
  expect
} from '../fixtures/base.fixture';

test(
  'users page displays data',
  async ({
    usersPage
  }) => {

    await usersPage.open();

    await expect(
      usersPage.title
    ).toBeVisible();
  }
);
```

The test focuses on business behavior instead of setup details.

---

## 82. Production-Quality Data Fixture

```ts
type Fixtures = {
  testUser:
    User;
};

export const test =
  base.extend<Fixtures>({
    testUser:
      async (
        { usersApi },
        use
      ) => {

        const response =
          await usersApi
            .createUser({
              name:
                'Automation User',
              role:
                'QA'
            });

        const user =
          await response.json();

        try {

          await use(user);

        } finally {

          await usersApi
            .deleteUser(
              user.id
            );
        }
      }
  });
```

This pattern combines:

```text
Fixture
 ↓
API Client
 ↓
Create Data
 ↓
Test
 ↓
Cleanup
```

---

## 83. Production-Quality Worker Fixture

Use worker scope for resources intentionally shared by tests in one worker.

Example concept:

```ts
type WorkerFixtures = {
  workerConfig:
    WorkerConfig;
};

export const test =
  base.extend<
    {},
    WorkerFixtures
  >({
    workerConfig: [
      async (
        {},
        use
      ) => {

        const config =
          createWorkerConfig();

        await use(config);

        await cleanupWorkerConfig(
          config
        );
      },
      {
        scope:
          'worker'
      }
    ]
  });
```

---

## 84. Multi-Role Fixture Example

```ts
type Fixtures = {
  adminPage:
    Page;
  employeePage:
    Page;
};

export const test =
  base.extend<Fixtures>({
    adminPage:
      async (
        { browser },
        use
      ) => {

        const context =
          await browser.newContext({
            storageState:
              '.auth/admin.json'
          });

        const page =
          await context.newPage();

        await use(page);

        await context.close();
      },

    employeePage:
      async (
        { browser },
        use
      ) => {

        const context =
          await browser.newContext({
            storageState:
              '.auth/employee.json'
          });

        const page =
          await context.newPage();

        await use(page);

        await context.close();
      }
  });
```

---

## 85. Fixture Naming Convention

Recommended:

```text
loginPage
dashboardPage
usersPage
usersApi
testUser
authenticatedPage
adminPage
employeePage
workerConfig
```

Keep names descriptive and consistent.

---

## 86. Fixture Directory Convention

A scalable project can use:

```text
fixtures/
├── base.fixture.ts
├── auth.fixture.ts
├── page.fixture.ts
├── api.fixture.ts
├── data.fixture.ts
└── worker.fixture.ts
```

The exact structure can be simplified for smaller projects.

---

## 87. Enterprise Fixture Dependency Map

```text
                         Test
                           │
              ┌────────────┼────────────┐
              │            │            │
          Page Layer    API Layer    Data Layer
              │            │            │
         Page Objects   API Clients   Factories
              │            │            │
              └────────────┼────────────┘
                           │
                     Auth Fixture
                           │
                    Storage State
                           │
                       Browser
```

This keeps responsibilities separated.

---

## 88. Testing Fixture Behavior

Fixtures themselves should be reliable.

Test scenarios such as:

```text
Fixture setup success
Fixture setup failure
Cleanup success
Cleanup failure
Parallel execution
Retry behavior
Authentication expiry
```

For critical enterprise fixtures, treat fixture infrastructure as production code.

---

## 89. Fixture Performance Measurement

Track:

```text
Fixture setup time
Fixture teardown time
Test execution time
Worker startup time
```

If a fixture consumes several seconds for every test, investigate whether it can be:

```text
Worker-scoped
API-based
Cached safely
Moved to setup project
```

---

## 90. Fixture Design Rules

A good fixture should:

```text
Have one clear responsibility
Be reusable
Have predictable lifecycle
Be strongly typed
Handle cleanup
Avoid hidden behavior
Support parallel execution
Provide useful errors
Protect sensitive data
```

---

## 91. Interview Questions

### Q1. What is a Playwright fixture?

A reusable dependency that provides setup, teardown, and resources to tests.

### Q2. What is dependency injection?

The framework supplies required dependencies instead of tests manually creating them.

### Q3. What is `use()`?

It passes the fixture value to the test and controls the point between setup and teardown.

### Q4. What is a worker fixture?

A fixture with worker-level scope that can be shared by tests running within the same worker.

### Q5. When should you use worker scope?

For safely shareable worker-specific resources, especially expensive setup that does not contain mutable per-test state.

### Q6. What is fixture composition?

Building a fixture from other fixtures and dependencies.

### Q7. Fixture vs Page Object?

A fixture manages dependency lifecycle and injection; a Page Object encapsulates UI behavior.

### Q8. Fixture vs utility?

A fixture participates in dependency injection and lifecycle; a utility generally performs standalone operations.

### Q9. Why is cleanup important?

It prevents test data and resources from leaking into later tests.

### Q10. What is a fixture anti-pattern?

A huge “God Fixture” that performs unrelated setup for every test.

---

## 92. Hands-On Exercises

### Exercise 1 — Custom Fixture

Create:

```text
testMessage
```

and inject it into a test.

### Exercise 2 — Page Object Fixture

Create:

```text
loginPage
dashboardPage
```

fixtures.

### Exercise 3 — API Fixture

Create:

```text
usersApi
```

fixture.

### Exercise 4 — Data Fixture

Create:

```text
testUser
```

with automatic cleanup.

### Exercise 5 — Worker Fixture

Create a worker-scoped configuration object.

### Exercise 6 — Role Fixtures

Create:

```text
adminPage
employeePage
```

### Exercise 7 — Multi-User Workflow

Use separate role fixtures to implement:

```text
Employee submits
 ↓
Manager approves
```

### Exercise 8 — Fixture Composition

Build:

```text
usersApi
 ↓
testUser
 ↓
usersPage
```

### Exercise 9 — Retry Safety

Verify that test retries create isolated test data.

### Exercise 10 — Enterprise Fixture Framework

Create a complete fixture architecture supporting:

```text
Authentication
Page Objects
API Clients
Test Data
Worker Resources
Multi-Role Contexts
Cleanup
```

---

## 93. Enterprise Fixture Challenge

Build a reusable Playwright fixture framework for an enterprise application.

Required fixtures:

```text
loginPage
dashboardPage
usersPage
usersApi
reportsApi
testUser
authenticatedPage
adminPage
managerPage
employeePage
workerConfig
```

Requirements:

1. Use TypeScript types for all fixtures.
2. Extend Playwright's base test.
3. Preserve built-in fixtures.
4. Use storage state for reusable authentication.
5. Support multiple roles.
6. Support API clients.
7. Support test-data creation.
8. Automatically clean up test data.
9. Use worker scope only when appropriate.
10. Keep mutable state test-scoped.
11. Support parallel execution.
12. Support retries.
13. Provide clear fixture errors.
14. Protect authentication secrets.
15. Avoid hidden navigation.
16. Avoid global mutable state.
17. Keep fixture files modular.
18. Measure fixture performance.
19. Make the framework CI-friendly.
20. Document fixture responsibilities.

---

## 94. Final Enterprise Architecture

```text
                         Playwright Test
                                │
                    ┌───────────┴───────────┐
                    │                       │
                 Fixtures              Test Data
                    │                       │
          ┌─────────┼─────────┐             │
          │         │         │             │
        Auth       Page       API         Factory
          │         │         │             │
      Storage     Objects   Clients         │
       State        │         │              │
          │         └────┬────┘              │
          │              │                   │
          └──────────────┼───────────────────┘
                         │
                    Test Workflow
                         │
                 ┌───────┴───────┐
                 │               │
              Browser           API
                 │               │
                 └───────┬───────┘
                         │
                      Cleanup
```

This architecture keeps setup, dependencies, authentication, test data, API clients, Page Objects, and cleanup modular and reusable.

---

## 95. Final Checklist

```text
[ ] Built-in fixtures understood
[ ] Custom fixtures implemented
[ ] Dependency injection understood
[ ] use() lifecycle understood
[ ] Setup and teardown implemented
[ ] Page Object fixtures implemented
[ ] API fixtures implemented
[ ] Data fixtures implemented
[ ] Fixture composition understood
[ ] Test scope understood
[ ] Worker scope understood
[ ] Auto fixtures understood
[ ] Fixture dependencies understood
[ ] Cleanup implemented
[ ] Retry behavior considered
[ ] Parallel execution considered
[ ] Multi-role fixtures implemented
[ ] Authentication state integrated
[ ] Fixture types defined
[ ] Fixture files organized
[ ] Global mutable state avoided
[ ] God fixtures avoided
[ ] Fixture performance considered
[ ] CI execution supported
```

---

## 96. Key Takeaways

- Fixtures are the foundation of a scalable Playwright framework.
- They provide reusable dependencies through dependency injection.
- Built-in fixtures such as `page`, `context`, `browser`, and `request` simplify test setup.
- Custom fixtures allow Page Objects, API clients, test data, authentication, and other resources to be injected.
- `use()` separates fixture setup from teardown.
- Test-scoped fixtures provide strong isolation.
- Worker-scoped fixtures can improve performance when resources are safely shareable.
- Fixture composition allows complex dependencies to be built from simpler ones.
- Page Objects and fixtures have different responsibilities.
- API clients and test-data factories should remain separate from fixture lifecycle management.
- Cleanup is essential for reliable automation.
- Fixtures must be safe for retries and parallel execution.
- Avoid global mutable state and oversized “God Fixtures.”
- Strong TypeScript typing makes custom fixture systems easier to maintain.
- Enterprise fixtures should be modular, predictable, secure, and CI-friendly.

---

## 97. Next Step

**Step 95 — Mastering Playwright Test Configuration & Execution: Projects, Environments, CLI Commands, Workers, Parallelism, Retries, Timeouts, Reporters, Global Setup, CI/CD & Enterprise Configuration**

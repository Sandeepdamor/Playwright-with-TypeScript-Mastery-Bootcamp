# Playwright with TypeScript Mastery Bootcamp

# Step 82 — Mastering Playwright Advanced Fixtures, Test Fixtures, Worker Fixtures, Fixture Composition & Enterprise Dependency Injection

## 1. Introduction

Fixtures are one of the most important concepts for building a scalable Playwright automation framework.

A fixture provides reusable setup and teardown logic to tests.

Instead of repeating:

```ts
const page = await browser.newPage();
await login();
await createTestData();
```

in every test, fixtures can provide ready-to-use dependencies.

A mature Playwright framework can use fixtures for:

- Authenticated pages
- Page Objects
- API clients
- Test data
- Database connections
- Browser contexts
- Role-based users
- Environment configuration
- Cleanup
- Reporting helpers

---

## 2. What Is a Fixture?

A fixture is a reusable test dependency.

Conceptually:

```text
Fixture
   ↓
Setup
   ↓
Test
   ↓
Teardown
```

Example:

```ts
test('dashboard', async ({ page }) => {
  await page.goto('/dashboard');
});
```

Here, `page` is a built-in Playwright fixture.

---

## 3. Built-In Playwright Fixtures

Playwright provides several built-in fixtures.

Common examples include:

```text
page
context
browser
browserName
request
baseURL
```

These fixtures are automatically provided to tests.

Example:

```ts
test('example', async ({
  page,
  context,
  browser,
  request
}) => {
  // Test implementation
});
```

---

## 4. Why Fixtures Are Important

Without fixtures:

```text
Test 1 → setup → test → cleanup
Test 2 → setup → test → cleanup
Test 3 → setup → test → cleanup
```

With fixtures:

```text
Reusable Fixture
       ↓
Test 1
Test 2
Test 3
```

Benefits include:

- Less duplicate code
- Better test isolation
- Centralized setup
- Centralized teardown
- Easier maintenance
- Better scalability

---

## 5. Test-Scoped vs Worker-Scoped Fixtures

Playwright fixtures can have different scopes.

### Test-scoped fixture

Created for each test.

```text
Test 1 → Fixture
Test 2 → Fixture
Test 3 → Fixture
```

### Worker-scoped fixture

Created once per worker.

```text
Worker
  ↓
Fixture
  ├── Test 1
  ├── Test 2
  └── Test 3
```

Choosing the correct scope is important for performance and isolation.

---

## 6. Creating a Custom Fixture

Use `test.extend()`.

Example:

```ts
import { test as base } from '@playwright/test';

type Fixtures = {
  testMessage: string;
};

export const test = base.extend<Fixtures>({
  testMessage: async ({}, use) => {
    await use('Hello Playwright');
  }
});
```

Test:

```ts
test('custom fixture', async ({
  testMessage
}) => {
  console.log(testMessage);
});
```

---

## 7. Understanding `test.extend()`

The basic pattern is:

```ts
const test = base.extend<Fixtures>({
  fixtureName: async ({}, use) => {
    // Setup

    await use(value);

    // Teardown
  }
});
```

The `use()` function separates setup from teardown.

---

## 8. Fixture Lifecycle

A fixture typically follows:

```text
Fixture Created
      ↓
Setup
      ↓
use(value)
      ↓
Test Executes
      ↓
Teardown
      ↓
Fixture Destroyed
```

Example:

```ts
testData: async ({}, use) => {
  const data = createData();

  await use(data);

  deleteData(data);
}
```

---

## 9. Setup and Teardown

Example:

```ts
const test = base.extend({
  resource: async ({}, use) => {
    const resource = await createResource();

    await use(resource);

    await resource.cleanup();
  }
});
```

This ensures cleanup happens after the test completes.

---

## 10. Page Object Fixture

A common enterprise pattern is exposing Page Objects through fixtures.

Example:

```ts
import {
  test as base
} from '@playwright/test';

import { LoginPage } from '../pages/LoginPage';

type Fixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  }
});
```

Test:

```ts
test('login', async ({
  loginPage
}) => {
  await loginPage.navigate();
});
```

---

## 11. Multiple Page Object Fixtures

A larger framework can expose:

```text
loginPage
dashboardPage
adminPage
userPage
reportsPage
```

Example:

```ts
type Fixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
  adminPage: AdminPage;
};
```

This keeps test code concise.

---

## 12. Fixture Composition

Fixtures can depend on other fixtures.

Example:

```ts
const test = base.extend({
  authenticatedPage: async ({
    page
  }, use) => {
    await login(page);

    await use(page);
  }
});
```

Another fixture can depend on it:

```ts
dashboardPage: async ({
  authenticatedPage
}, use) => {
  await use(
    new DashboardPage(authenticatedPage)
  );
}
```

This creates a dependency chain:

```text
page
 ↓
authenticatedPage
 ↓
dashboardPage
```

---

## 13. Fixture Dependency Injection

Playwright automatically injects fixture dependencies.

Example:

```ts
myFixture: async ({
  page,
  request
}, use) => {
  // page and request are injected
}
```

The fixture does not manually create them.

This is similar to dependency injection concepts used in enterprise frameworks.

---

## 14. Fixture Dependency Graph

A complex framework may look like:

```text
browser
   ↓
context
   ↓
page
   ↓
authenticatedPage
   ↓
dashboardPage
```

Another branch may be:

```text
request
   ↓
usersApi
   ↓
testData
```

Fixtures form a dependency graph.

---

## 15. API Client Fixture

Example:

```ts
import { APIRequestContext } from '@playwright/test';

class UsersApi {
  constructor(
    private readonly request: APIRequestContext
  ) {}

  async getUsers() {
    return this.request.get('/api/users');
  }
}
```

Fixture:

```ts
const test = base.extend<{
  usersApi: UsersApi;
}>({
  usersApi: async ({ request }, use) => {
    await use(new UsersApi(request));
  }
});
```

Test:

```ts
test('users API', async ({
  usersApi
}) => {
  const response =
    await usersApi.getUsers();

  expect(response.ok()).toBeTruthy();
});
```

---

## 16. Authentication Fixture

A reusable authenticated page can be provided:

```ts
authenticatedPage: async ({
  browser
}, use) => {
  const context = await browser.newContext({
    storageState:
      'playwright/.auth/user.json'
  });

  const page = await context.newPage();

  await use(page);

  await context.close();
}
```

This keeps authentication logic out of individual tests.

---

## 17. Role-Based Fixtures

Example:

```text
adminPage
managerPage
employeePage
```

Each can use a different storage state.

Conceptually:

```text
adminPage
   ↓
admin.json

managerPage
   ↓
manager.json

employeePage
   ↓
employee.json
```

This supports role-based testing.

---

## 18. Worker Fixtures

Worker fixtures are useful when setup should happen once per worker.

Example:

```ts
type WorkerFixtures = {
  workerData: string;
};

const test = base.extend<
  {},
  WorkerFixtures
>({
  workerData: [
    async ({}, use) => {
      const data =
        `worker-${process.pid}`;

      await use(data);
    },
    { scope: 'worker' }
  ]
});
```

The fixture is shared by tests executed within that worker.

---

## 19. Why Use Worker Fixtures?

Worker fixtures are useful for expensive setup such as:

- Database connections
- Service clients
- Environment initialization
- Test servers
- Worker-specific accounts
- Large static data setup

They should not be used for mutable state that must remain isolated between tests.

---

## 20. Test Fixtures vs Worker Fixtures

### Test Fixture

```text
Test 1 → Fixture
Test 2 → Fixture
Test 3 → Fixture
```

Best for:

- Page objects
- Test-specific data
- Authenticated page
- Mutable resources

### Worker Fixture

```text
Worker → Fixture
           ↓
        Test 1
        Test 2
        Test 3
```

Best for:

- Expensive shared resources
- Worker-specific setup
- Database connections
- Servers

---

## 21. Worker Isolation

Playwright workers are independent processes.

Conceptually:

```text
Worker 1
 ├── Test A
 ├── Test B
 └── Test C

Worker 2
 ├── Test D
 ├── Test E
 └── Test F
```

A worker-scoped fixture exists separately in each worker.

---

## 22. Worker Fixture with Unique Data

You can use the worker index:

```ts
workerData: [
  async ({}, use, workerInfo) => {
    const data = {
      worker: workerInfo.workerIndex
    };

    await use(data);
  },
  { scope: 'worker' }
]
```

This can help create worker-specific resources.

---

## 23. Test Fixtures with `testInfo`

Fixtures can access test metadata:

```ts
myFixture: async ({
  page
}, use, testInfo) => {
  console.log(
    testInfo.title
  );

  await use(page);
}
```

This can be useful for:

- Logging
- Dynamic test data
- Artifact paths
- Diagnostics

---

## 24. Using `workerInfo`

Worker fixtures can access worker metadata:

```ts
workerFixture: [
  async ({}, use, workerInfo) => {
    console.log(
      workerInfo.workerIndex
    );

    await use();
  },
  { scope: 'worker' }
]
```

---

## 25. Fixture Auto-Execution

Fixtures can be configured with:

```ts
{ auto: true }
```

Example:

```ts
setupEnvironment: [
  async ({}, use) => {
    await setupEnvironment();

    await use();
  },
  { auto: true }
]
```

An automatic fixture runs without needing to be explicitly requested by the test.

Use automatic fixtures carefully because they can hide setup dependencies.

---

## 26. Optional Fixtures

A fixture can be designed so that tests only use it when needed.

This keeps setup efficient.

Prefer explicit dependencies for major resources unless the fixture genuinely belongs to every test.

---

## 27. Fixture Timeout

Fixtures can have their own timeout configuration.

Example:

```ts
slowFixture: [
  async ({}, use) => {
    await setupSlowResource();

    await use();
  },
  {
    timeout: 60_000
  }
]
```

This is useful when setup is intentionally longer than normal test operations.

---

## 28. Fixture Teardown on Failure

Fixture teardown should be designed to run even when a test fails.

Example:

```ts
resource: async ({}, use) => {
  const resource =
    await createResource();

  try {
    await use(resource);
  } finally {
    await deleteResource(resource);
  }
}
```

This makes cleanup more robust.

---

## 29. Database Fixture

A database fixture can provide test data.

Conceptually:

```ts
db: async ({}, use) => {
  const connection =
    await createDatabaseConnection();

  await use(connection);

  await connection.close();
}
```

The actual implementation depends on the database and client library.

---

## 30. Test Data Fixture

Example:

```ts
userData: async ({}, use) => {
  const user = {
    name: `Test User ${Date.now()}`,
    role: 'employee'
  };

  await use(user);
}
```

Tests can then use:

```ts
test('create user', async ({
  userData
}) => {
  console.log(userData.name);
});
```

---

## 31. Fixture for API-Created Test Data

A powerful pattern is:

```text
API
 ↓
Create Test User
 ↓
Fixture
 ↓
UI Test
 ↓
Cleanup
```

Example:

```ts
testUser: async ({
  request
}, use) => {
  const response =
    await request.post('/api/users', {
      data: {
        name: 'Test User'
      }
    });

  const user =
    await response.json();

  await use(user);

  await request.delete(
    `/api/users/${user.id}`
  );
}
```

This provides automatic test-data cleanup.

---

## 32. Fixture Composition Example

A mature framework may have:

```text
request
 ↓
usersApi
 ↓
testUser
 ↓
authenticatedPage
 ↓
userPage
```

Each layer provides a higher-level dependency.

This is one of the strongest reasons to use fixtures.

---

## 33. Fixture File Organization

A scalable project can use:

```text
fixtures/
├── base.fixture.ts
├── auth.fixture.ts
├── api.fixture.ts
├── page.fixture.ts
├── data.fixture.ts
└── index.ts
```

For very large frameworks, fixtures can be organized by domain.

---

## 34. Base Test Pattern

Example:

```ts
import {
  test as base,
  expect
} from '@playwright/test';

export const test = base.extend({
  // Custom fixtures
});

export { expect };
```

Then:

```ts
import {
  test,
  expect
} from '../fixtures/base.fixture';
```

This creates a single custom test entry point.

---

## 35. Avoiding Fixture Overengineering

Not every helper should become a fixture.

Use a fixture when:

- Setup/teardown is required
- A dependency is reused
- Scope matters
- Dependency injection improves readability

Use a utility when:

- The function is stateless
- No lifecycle is required
- It does not need fixture dependencies

Example utility:

```ts
export function generateEmail() {
  return `user-${Date.now()}@example.com`;
}
```

---

## 36. Fixture vs Utility

### Fixture

```ts
testUser: async ({
  request
}, use) => {
  // Setup
  // Test
  // Cleanup
}
```

### Utility

```ts
function generateUserName() {
  return `user-${Date.now()}`;
}
```

Fixtures are lifecycle-aware. Utilities are usually simple reusable functions.

---

## 37. Fixture Naming Best Practices

Use names that describe the dependency:

```text
loginPage
dashboardPage
adminPage
usersApi
testUser
authenticatedPage
database
```

Avoid vague names:

```text
helper
thing
data
common
object
```

Clear names make tests self-documenting.

---

## 38. Fixture Dependency Direction

Keep dependencies flowing in a clear direction.

Good:

```text
browser
 ↓
page
 ↓
authenticatedPage
 ↓
dashboardPage
```

Avoid circular dependencies such as:

```text
Fixture A
 ↓
Fixture B
 ↓
Fixture A
```

The dependency graph should remain understandable.

---

## 39. Enterprise Fixture Architecture

A large framework can look like:

```text
Base Playwright Fixtures
          ↓
Environment Fixtures
          ↓
Authentication Fixtures
          ↓
API Fixtures
          ↓
Test Data Fixtures
          ↓
Page Object Fixtures
          ↓
Business Workflow Fixtures
          ↓
Tests
```

This provides layered dependency injection.

---

## 40. Fixture Lifecycle Visualization

```text
Worker Starts
     ↓
Worker Fixture Setup
     ↓
Test Starts
     ↓
Test Fixture Setup
     ↓
Page Object Setup
     ↓
Test Executes
     ↓
Test Fixture Teardown
     ↓
Next Test
     ↓
Worker Ends
     ↓
Worker Fixture Teardown
```

Understanding this lifecycle is critical for debugging setup and cleanup issues.

---

## 41. Fixture Execution Order

When a fixture depends on another fixture, Playwright resolves dependencies first.

Example:

```text
page
 ↓
authPage
 ↓
dashboardPage
```

The lifecycle follows the dependency graph.

This allows complex fixtures to be composed safely.

---

## 42. Cleanup Ordering

Cleanup happens in reverse dependency order.

Conceptually:

```text
dashboardPage cleanup
        ↓
authPage cleanup
        ↓
page/context cleanup
```

This ensures dependent resources are cleaned up before their dependencies.

---

## 43. Fixture Error Handling

Fixtures should provide meaningful errors.

Example:

```ts
testUser: async ({
  request
}, use) => {
  const response =
    await request.post('/api/users', {
      data: {
        name: 'Test User'
      }
    });

  if (!response.ok()) {
    throw new Error(
      `Failed to create test user: ${
        response.status()
      }`
    );
  }

  const user =
    await response.json();

  await use(user);

  await request.delete(
    `/api/users/${user.id}`
  );
}
```

Avoid exposing sensitive response data in errors.

---

## 44. Fixture-Based Authentication + Page Object

Example dependency chain:

```text
browser
 ↓
authenticatedPage
 ↓
dashboardPage
```

Fixture:

```ts
dashboardPage: async ({
  authenticatedPage
}, use) => {
  await use(
    new DashboardPage(
      authenticatedPage
    )
  );
}
```

Test:

```ts
test('dashboard', async ({
  dashboardPage
}) => {
  await dashboardPage.verifyDashboard();
});
```

The test is now focused entirely on business behavior.

---

## 45. Fixture-Based API + Test Data

Example:

```text
request
 ↓
usersApi
 ↓
testUser
 ↓
test
```

The test only needs:

```ts
test('user workflow', async ({
  testUser
}) => {
  // Use prepared user
});
```

---

## 46. Fixture-Based Multi-Role Testing

Example:

```text
adminPage
managerPage
employeePage
```

Each fixture can create a browser context using the correct storage state.

This makes role-specific tests extremely readable.

---

## 47. Fixture Scope Selection Guide

Use **test scope** when:

```text
State is mutable
State must be isolated
Resource belongs to one test
```

Use **worker scope** when:

```text
Setup is expensive
Resource can safely be shared
Resource is worker-specific
```

Never choose worker scope simply because it is faster if doing so breaks test isolation.

---

## 48. Parallel Execution and Fixtures

With:

```ts
fullyParallel: true
```

tests may execute concurrently.

Fixtures should therefore:

- Avoid shared mutable state
- Use unique test data
- Use worker-aware resources
- Clean up reliably
- Avoid fixed filenames
- Avoid shared accounts when mutations conflict

---

## 49. Fixture and Test Isolation

A good framework ensures:

```text
Test A
 ↓
Own Data
 ↓
Own State
 ↓
Own Cleanup
```

and:

```text
Test B
 ↓
Own Data
 ↓
Own State
 ↓
Own Cleanup
```

Isolation is more important than blindly maximizing reuse.

---

## 50. Common Fixture Mistakes

### Mistake 1 — Making everything a fixture

This creates unnecessary complexity.

### Mistake 2 — Using worker scope for mutable data

This can cause tests to influence one another.

### Mistake 3 — Forgetting teardown

Resources can leak across tests.

### Mistake 4 — Circular dependencies

Fixtures should form a clear dependency graph.

### Mistake 5 — Hiding important setup in auto fixtures

Tests become harder to understand.

### Mistake 6 — Sharing authentication state incorrectly

Role and session changes can leak between tests.

---

## 51. Production-Quality Fixture Example

```ts
import {
  test as base,
  expect
} from '@playwright/test';

import { DashboardPage } from '../pages/DashboardPage';
import { UsersApi } from '../api/UsersApi';

type Fixtures = {
  dashboardPage: DashboardPage;
  usersApi: UsersApi;
};

export const test = base.extend<Fixtures>({
  dashboardPage: async ({
    page
  }, use) => {
    await use(
      new DashboardPage(page)
    );
  },

  usersApi: async ({
    request
  }, use) => {
    await use(
      new UsersApi(request)
    );
  }
});

export { expect };
```

Test:

```ts
import {
  test,
  expect
} from '../fixtures/base.fixture';

test('dashboard and API', async ({
  dashboardPage,
  usersApi
}) => {
  await dashboardPage.verifyDashboard();

  const response =
    await usersApi.getUsers();

  expect(response.ok()).toBeTruthy();
});
```

---

## 52. Interview Questions

### Q1. What is a fixture in Playwright?

A fixture is a reusable dependency that provides setup, a resource for the test, and optional teardown.

### Q2. How do you create a custom fixture?

Use:

```ts
test.extend()
```

### Q3. What is the difference between test-scoped and worker-scoped fixtures?

Test-scoped fixtures are created for each test, while worker-scoped fixtures are created once per worker.

### Q4. When should you use a worker fixture?

For expensive resources that can safely be shared within a worker.

### Q5. What is fixture composition?

It is the ability for one fixture to depend on another fixture.

### Q6. What is dependency injection in Playwright fixtures?

Playwright automatically provides fixture dependencies through the fixture function arguments.

### Q7. What does `use()` do?

It provides the fixture value to the test and separates setup from teardown.

### Q8. What is `scope: 'worker'`?

It makes a custom fixture worker-scoped instead of test-scoped.

### Q9. What is an automatic fixture?

A fixture configured with:

```ts
{ auto: true }
```

which runs automatically without being explicitly requested by the test.

### Q10. Should every helper be a fixture?

No. Stateless reusable functions are often better implemented as utilities.

---

## 53. Hands-On Exercises

### Exercise 1 — Custom Fixture

Create a fixture that provides:

```ts
testMessage
```

### Exercise 2 — Page Object Fixture

Create:

```text
loginPage
dashboardPage
adminPage
```

fixtures.

### Exercise 3 — API Fixture

Create:

```text
usersApi
```

using `APIRequestContext`.

### Exercise 4 — Test Data Fixture

Create a user through an API and automatically delete it during teardown.

### Exercise 5 — Worker Fixture

Create a worker-scoped fixture that provides a unique worker identifier.

### Exercise 6 — Authentication Fixture

Create:

```text
authenticatedPage
```

using storage state.

### Exercise 7 — Role Fixtures

Create:

```text
adminPage
employeePage
```

using separate authentication states.

### Exercise 8 — Fixture Composition

Build:

```text
request
 ↓
usersApi
 ↓
testUser
```

### Exercise 9 — Cleanup

Add reliable teardown for every created resource.

### Exercise 10 — Parallel Execution

Run the fixture-based tests with multiple workers and verify that test data remains isolated.

---

## 54. Key Takeaways

- Fixtures provide reusable setup and teardown.
- Playwright includes built-in fixtures such as `page`, `context`, `browser`, and `request`.
- Custom fixtures are created with `test.extend()`.
- `use()` provides the fixture value to the test.
- Test-scoped fixtures provide strong isolation.
- Worker-scoped fixtures are useful for expensive worker-level resources.
- Fixtures can depend on other fixtures.
- Fixture composition creates a dependency graph.
- Page Objects can be exposed through fixtures.
- API clients can be exposed through fixtures.
- Test data can be created and cleaned up through fixtures.
- Authentication can be encapsulated in fixtures.
- Role-specific fixtures support multi-role testing.
- Avoid making every helper a fixture.
- Keep fixture dependencies simple and directional.
- Use teardown to prevent resource leaks.
- Protect test isolation during parallel execution.
- Avoid unnecessary automatic fixtures.
- Worker scope should be used for safe shared resources, not mutable test state.

---

## 55. Enterprise Fixture Architecture Challenge

Build a framework with this dependency structure:

```text
browser
   ↓
authenticatedPage
   ↓
dashboardPage

request
   ↓
usersApi
   ↓
testUser
   ↓
userPage
```

Then add:

```text
adminPage
managerPage
employeePage
```

as role-specific fixtures.

The final tests should require minimal setup code and should focus primarily on business behavior.

---

## 56. Next Step

**Step 83 — Mastering Playwright Test Configuration, Projects, Environments, BaseURL, Timeouts, Retries & Enterprise Configuration Management**

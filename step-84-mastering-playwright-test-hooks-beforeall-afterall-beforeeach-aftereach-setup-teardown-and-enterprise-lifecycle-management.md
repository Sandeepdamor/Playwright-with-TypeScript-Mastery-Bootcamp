# Playwright with TypeScript Mastery Bootcamp

# Step 84 — Mastering Playwright Test Hooks, BeforeAll, AfterAll, BeforeEach, AfterEach, Setup/Teardown & Enterprise Lifecycle Management

## 1. Introduction

Test hooks are used to control setup and cleanup around Playwright tests.

Hooks help manage the test lifecycle:

```text
Before All
    ↓
Before Each
    ↓
Test
    ↓
After Each
    ↓
After All
```

They are useful for:

- Authentication setup
- Test data preparation
- Database cleanup
- Navigation
- Logging
- Resource initialization
- Resource cleanup
- Environment preparation

Hooks should be used carefully. Fixtures are often a better choice when setup requires reusable dependencies, lifecycle control, or dependency injection.

---

## 2. What Are Test Hooks?

Hooks are lifecycle functions that run automatically at specific points during test execution.

Main Playwright hooks:

```ts
test.beforeAll()
test.afterAll()
test.beforeEach()
test.afterEach()
```

They allow common setup and teardown logic to be centralized.

---

## 3. Basic Hook Lifecycle

A simplified test lifecycle is:

```text
beforeAll
   ↓
beforeEach
   ↓
Test 1
   ↓
afterEach
   ↓
beforeEach
   ↓
Test 2
   ↓
afterEach
   ↓
afterAll
```

The exact lifecycle also involves fixture setup and teardown.

---

## 4. `beforeEach`

`beforeEach` runs before every test in its scope.

Example:

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/login');
});
```

If there are three tests:

```text
beforeEach
Test 1

beforeEach
Test 2

beforeEach
Test 3
```

---

## 5. Using `beforeEach` for Navigation

Example:

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/dashboard');
});

test('profile', async ({ page }) => {
  await page.getByRole('link', {
    name: 'Profile'
  }).click();
});

test('settings', async ({ page }) => {
  await page.getByRole('link', {
    name: 'Settings'
  }).click();
});
```

Each test starts from the dashboard.

---

## 6. `afterEach`

`afterEach` runs after every test in its scope.

Example:

```ts
test.afterEach(async ({ page }) => {
  console.log('Test completed');
});
```

It can be used for:

- Cleanup
- Logging
- Diagnostics
- Resetting state

---

## 7. `beforeAll`

`beforeAll` runs once before the tests in its scope.

Example:

```ts
test.beforeAll(async () => {
  console.log('Suite setup');
});
```

Conceptually:

```text
beforeAll
 ↓
Test 1
Test 2
Test 3
 ↓
afterAll
```

---

## 8. `afterAll`

`afterAll` runs once after all tests in its scope finish.

Example:

```ts
test.afterAll(async () => {
  console.log('Suite cleanup');
});
```

It is useful for cleaning resources created for the whole suite.

---

## 9. `beforeAll` vs `beforeEach`

### `beforeAll`

Runs once:

```text
beforeAll
 ↓
Test 1
Test 2
Test 3
```

### `beforeEach`

Runs before every test:

```text
beforeEach
Test 1

beforeEach
Test 2

beforeEach
Test 3
```

Use `beforeAll` for shared setup that is safe to reuse.

Use `beforeEach` when every test needs fresh setup.

---

## 10. `afterAll` vs `afterEach`

### `afterAll`

Runs once after the group:

```text
Test 1
Test 2
Test 3
 ↓
afterAll
```

### `afterEach`

Runs after every test:

```text
Test 1
 ↓
afterEach

Test 2
 ↓
afterEach
```

Choose based on the required isolation level.

---

## 11. Hooks Inside a Test Group

Hooks can be scoped to a `test.describe()` block.

Example:

```ts
test.describe('Admin Tests', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('/admin');
  });

  test('users', async ({ page }) => {
    // test
  });

  test('roles', async ({ page }) => {
    // test
  });

});
```

The hook applies only to tests inside the group.

---

## 12. Describe-Level Organization

Example:

```ts
test.describe('User Management', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('/users');
  });

  test('create user', async ({ page }) => {
    // ...
  });

  test('search user', async ({ page }) => {
    // ...
  });

});
```

This is useful when a group of tests shares setup.

---

## 13. Nested Describe Blocks

Hooks can be used with nested test groups.

Example:

```ts
test.describe('Admin', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('/admin');
  });

  test.describe('Users', () => {

    test.beforeEach(async ({ page }) => {
      await page.goto('/admin/users');
    });

    test('create user', async ({ page }) => {
      // ...
    });

  });

});
```

The nested structure should remain easy to understand.

---

## 14. Hook Execution Order

Suppose:

```ts
test.beforeAll(...);

test.beforeEach(...);

test('test', async () => {});

test.afterEach(...);

test.afterAll(...);
```

The lifecycle is:

```text
beforeAll
   ↓
beforeEach
   ↓
test
   ↓
afterEach
   ↓
afterAll
```

For multiple tests:

```text
beforeAll
 ↓
beforeEach
 ↓
Test 1
 ↓
afterEach
 ↓
beforeEach
 ↓
Test 2
 ↓
afterEach
 ↓
afterAll
```

---

## 15. Hooks and Fixtures

Hooks can request fixtures.

Example:

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/dashboard');
});
```

Here Playwright provides the `page` fixture.

This is one reason hooks integrate naturally with Playwright's fixture system.

---

## 16. Hooks with `request`

API setup can also happen in hooks.

Example:

```ts
test.beforeEach(async ({ request }) => {
  const response = await request.post('/api/test-data', {
    data: {
      name: 'Test User'
    }
  });

  expect(response.ok()).toBeTruthy();
});
```

However, if the setup needs reusable data and cleanup, a custom fixture may be a better design.

---

## 17. Hook Setup and Cleanup

Example:

```ts
test.beforeEach(async ({ request }) => {
  await request.post('/api/setup');
});

test.afterEach(async ({ request }) => {
  await request.post('/api/cleanup');
});
```

This can work for simple stateless setup.

For resource-specific lifecycle, fixtures are usually more expressive.

---

## 18. `beforeAll` with Browser

A common misconception is that `beforeAll` should manually create everything.

For example:

```ts
test.beforeAll(async ({ browser }) => {
  // suite-level setup
});
```

The browser fixture can be used for suite-level setup when appropriate.

Avoid creating unnecessary shared browser state that causes tests to depend on one another.

---

## 19. Shared Page Anti-Pattern

Avoid using one shared page for independent tests:

```text
Shared Page
   ↓
Test 1
   ↓
Test 2
   ↓
Test 3
```

This can create state leakage.

Prefer Playwright's normal test-scoped `page` fixture:

```text
Test 1 → own page
Test 2 → own page
Test 3 → own page
```

---

## 20. Test Isolation

A strong framework should aim for:

```text
Test A
 ↓
Setup
 ↓
Test
 ↓
Cleanup

Test B
 ↓
Setup
 ↓
Test
 ↓
Cleanup
```

One test should not depend on another test's result.

---

## 21. Hooks and Authentication

A simple suite may use:

```ts
test.beforeEach(async ({ page }) => {
  await login(page);
});
```

But repeatedly logging in through the UI can be slow.

For large suites, prefer:

```text
Authentication Setup
        ↓
Storage State
        ↓
Authenticated Tests
```

Hooks should not become a workaround for missing authentication architecture.

---

## 22. Hooks and Page Objects

Hooks can initialize navigation while Page Objects handle business actions.

Example:

```ts
test.beforeEach(async ({
  dashboardPage
}) => {
  await dashboardPage.navigate();
});
```

Test:

```ts
test('verify dashboard', async ({
  dashboardPage
}) => {
  await dashboardPage.verifyDashboard();
});
```

This creates readable tests.

---

## 23. Hooks and Test Data

Example:

```ts
test.beforeEach(async ({
  usersApi
}) => {
  await usersApi.createTestUser();
});
```

But if the test needs the created user's ID for cleanup, a fixture is usually better:

```text
usersApi
   ↓
testUser fixture
   ↓
test
   ↓
cleanup
```

---

## 24. When to Use Hooks

Use hooks for simple, shared lifecycle operations such as:

- Navigation
- Lightweight logging
- Common setup
- Suite-level initialization
- Simple cleanup
- Test metadata preparation

---

## 25. When to Use Fixtures Instead

Prefer fixtures when setup involves:

- Dependency injection
- Resource lifecycle
- Automatic cleanup
- Test data creation
- Authentication contexts
- API clients
- Page Objects
- Scope management

Example:

```text
Fixture
  ↓
Create Resource
  ↓
Use Resource
  ↓
Cleanup
```

---

## 26. Hooks vs Fixtures

### Hooks

```text
beforeEach
afterEach
beforeAll
afterAll
```

Good for procedural lifecycle logic.

### Fixtures

```text
fixture
 ↓
setup
 ↓
use()
 ↓
teardown
```

Good for reusable dependencies and resource lifecycle.

---

## 27. `beforeEach` Failure

If a `beforeEach` hook fails, the test may not execute.

Example:

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/dashboard');

  await expect(
    page.getByText('Dashboard')
  ).toBeVisible();
});
```

If the dashboard cannot be reached, the test setup fails.

This can be useful because the failure clearly identifies a shared prerequisite.

---

## 28. `afterEach` Failure

Cleanup can also fail.

Example:

```ts
test.afterEach(async ({ request }) => {
  const response =
    await request.delete('/api/test-data');

  expect(response.ok()).toBeTruthy();
});
```

A failing cleanup can affect test results and should be handled carefully.

For critical cleanup, make the cleanup reliable and report useful diagnostics without hiding the original test failure.

---

## 29. Using `try/finally`

For explicit resource cleanup:

```ts
test('resource workflow', async ({
  request
}) => {
  const resource =
    await createResource(request);

  try {
    // test
  } finally {
    await deleteResource(
      request,
      resource.id
    );
  }
});
```

Fixtures can often provide an even cleaner lifecycle abstraction.

---

## 30. Hook Timeouts

Hooks execute as part of the test lifecycle and can be affected by timeout configuration.

If setup is genuinely slow, configure the appropriate timeout rather than adding arbitrary waits.

Avoid:

```ts
await page.waitForTimeout(5000);
```

as a general synchronization strategy.

---

## 31. Hooks and Parallel Execution

With multiple workers:

```text
Worker 1
 ↓
beforeAll
 ↓
Tests

Worker 2
 ↓
beforeAll
 ↓
Tests
```

A worker-scoped `beforeAll` runs once per worker rather than once globally across the entire test process.

This is important when creating shared resources.

---

## 32. `beforeAll` and Parallel Workers

Suppose there are four workers:

```text
Worker 1 → beforeAll
Worker 2 → beforeAll
Worker 3 → beforeAll
Worker 4 → beforeAll
```

Therefore, do not assume `beforeAll` is a global singleton across all workers.

For globally unique setup, use an appropriate external coordination mechanism or redesign the setup.

---

## 33. Hook Scope

Hooks can exist at:

```text
File level
Describe level
Nested describe level
```

The narrower the scope, the fewer tests are affected.

Prefer the smallest scope that makes sense.

---

## 34. File-Level Hook

Example:

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/dashboard');
});
```

This applies to tests in that file.

---

## 35. Describe-Level Hook

Example:

```ts
test.describe('Reports', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('/reports');
  });

  test('monthly report', async () => {});
  test('yearly report', async () => {});

});
```

Only report tests use this setup.

---

## 36. Hook Reuse

If many files need exactly the same setup, avoid copying hooks everywhere.

Consider:

```text
Custom Fixture
```

or:

```text
Reusable helper
```

This keeps behavior centralized.

---

## 37. Hook and Page Navigation

Example:

```ts
test.beforeEach(async ({
  page
}) => {
  await page.goto('/users');
});
```

This is acceptable when every test in the group genuinely starts from `/users`.

Do not use it when different tests require different starting states.

---

## 38. Hook Overuse

Bad design:

```ts
beforeEach → login
beforeEach → create data
beforeEach → navigate
beforeEach → configure app
beforeEach → create files
beforeEach → seed database
```

This can make every test expensive and difficult to debug.

A better architecture separates responsibilities:

```text
Authentication Fixture
Test Data Fixture
Page Object
Test
```

---

## 39. Hidden Dependencies

Avoid hooks that silently modify application state.

For example:

```ts
beforeEach(async () => {
  await deleteAllUsers();
});
```

This can make tests difficult to understand and dangerous if multiple tests or workers use the same environment.

Prefer explicit, isolated test data.

---

## 40. Suite-Level Database Setup

A suite-level setup might be:

```ts
test.beforeAll(async () => {
  await seedReferenceData();
});
```

Then:

```ts
test.afterAll(async () => {
  await removeReferenceData();
});
```

This is appropriate only if the data is safe to share and does not create test-order dependencies.

---

## 41. Test Data Isolation

Instead of:

```text
beforeAll
 ↓
Create User A
 ↓
All tests modify User A
```

prefer:

```text
Test 1 → User 1
Test 2 → User 2
Test 3 → User 3
```

when the tests mutate the data.

This supports reliable parallel execution.

---

## 42. Hook-Based Logging

Hooks can provide useful logs:

```ts
test.beforeEach(async ({}, testInfo) => {
  console.log(
    `Starting: ${testInfo.title}`
  );
});

test.afterEach(async ({}, testInfo) => {
  console.log(
    `Finished: ${testInfo.title}`
  );
});
```

Do not log secrets or sensitive application data.

---

## 43. Using `testInfo`

`testInfo` provides information about the current test.

Example:

```ts
test.afterEach(async ({}, testInfo) => {
  console.log({
    title: testInfo.title,
    status: testInfo.status
  });
});
```

This is useful for diagnostics and custom reporting.

---

## 44. Conditional Cleanup

Sometimes cleanup should depend on what was created.

Example:

```ts
test.afterEach(async ({
  request
}, testInfo) => {
  if (testInfo.status === 'failed') {
    // Collect safe diagnostics
  }

  // Perform required cleanup
});
```

Do not skip required cleanup simply because a test failed.

---

## 45. Screenshot on Failure

Playwright configuration is usually better for generic screenshots:

```ts
use: {
  screenshot: 'only-on-failure'
}
```

Hooks can be used for custom diagnostics when necessary.

Avoid duplicating built-in artifact behavior unnecessarily.

---

## 46. Trace on Failure

Similarly:

```ts
use: {
  trace: 'retain-on-first-retry'
}
```

is often preferable to manually starting and stopping traces in every hook.

Use hooks for application-specific diagnostics rather than rebuilding Playwright's built-in reporting features.

---

## 47. Cleanup Strategies

Possible cleanup levels:

```text
Test-level cleanup
 ↓
afterEach / fixture teardown

Suite-level cleanup
 ↓
afterAll

Environment-level cleanup
 ↓
Dedicated setup/cleanup project
```

Choose the smallest reliable scope.

---

## 48. Enterprise Lifecycle Architecture

A scalable architecture can look like:

```text
Environment Setup
       ↓
Authentication Setup
       ↓
Worker Initialization
       ↓
Test Fixture Setup
       ↓
beforeEach
       ↓
Test
       ↓
afterEach
       ↓
Fixture Teardown
       ↓
afterAll
       ↓
Environment Cleanup
```

Each layer should have a clear responsibility.

---

## 49. Recommended Hook Strategy

### Use `beforeAll` for:

- Suite-level static setup
- Safe reference data
- Expensive initialization that can be shared

### Use `beforeEach` for:

- Navigation
- Lightweight test preparation
- Independent test prerequisites

### Use `afterEach` for:

- Test-specific cleanup
- Logging
- Diagnostics

### Use `afterAll` for:

- Suite-level cleanup
- Shared resource shutdown

### Use fixtures for:

- Reusable dependencies
- Authentication
- API clients
- Test data
- Page Objects
- Automatic setup/teardown

---

## 50. Production-Quality Example

```ts
import {
  test,
  expect
} from '../fixtures/base.fixture';

test.describe('User Management', () => {

  test.beforeEach(async ({
    adminPage
  }) => {
    await adminPage.navigate();
  });

  test('create user', async ({
    adminPage,
    testUser
  }) => {
    await adminPage.createUser(
      testUser
    );

    await expect(
      adminPage.user(testUser.name)
    ).toBeVisible();
  });

  test('search user', async ({
    adminPage,
    testUser
  }) => {
    await adminPage.searchUser(
      testUser.name
    );

    await expect(
      adminPage.user(testUser.name)
    ).toBeVisible();
  });

});
```

Here:

```text
Fixture → authentication
Fixture → testUser
Hook → navigation
Page Object → business actions
Test → assertions
```

This is a clean separation of responsibility.

---

## 51. Common Mistakes

### Mistake 1 — Logging in through `beforeEach` everywhere

This makes tests slow.

### Mistake 2 — Using `beforeAll` for mutable test data

Tests can become dependent on one another.

### Mistake 3 — Sharing a page across tests

This breaks isolation.

### Mistake 4 — Putting all setup into hooks

Fixtures may be a better solution.

### Mistake 5 — Using arbitrary sleeps

Avoid:

```ts
await page.waitForTimeout(5000);
```

### Mistake 6 — Assuming `beforeAll` runs once globally

With workers, it can run once per worker.

### Mistake 7 — Skipping cleanup after failures

Always design cleanup carefully.

---

## 52. Interview Questions

### Q1. What are Playwright hooks?

Hooks are lifecycle functions such as `beforeAll`, `afterAll`, `beforeEach`, and `afterEach` used for test setup and cleanup.

### Q2. What is the difference between `beforeAll` and `beforeEach`?

`beforeAll` runs once for the relevant scope, while `beforeEach` runs before every test in that scope.

### Q3. What is the difference between `afterAll` and `afterEach`?

`afterAll` runs once after the scope finishes, while `afterEach` runs after every test.

### Q4. Can hooks use fixtures?

Yes. Hook callbacks can request Playwright fixtures.

### Q5. Can hooks be scoped to `describe`?

Yes. Hooks inside `test.describe()` apply to tests in that group.

### Q6. Should authentication always happen in `beforeEach`?

No. Reusable storage state or authentication fixtures are often better for large suites.

### Q7. Why should shared mutable data be avoided in `beforeAll`?

Because tests can influence each other and parallel execution becomes unreliable.

### Q8. What is the difference between hooks and fixtures?

Hooks are lifecycle callbacks. Fixtures provide reusable dependencies with setup and teardown and support dependency injection.

### Q9. Does `beforeAll` run once for the entire test suite?

Not necessarily. With multiple workers, it can execute once in each worker's process for the relevant scope.

### Q10. What should be used for complex test-data lifecycle?

A fixture is usually preferable because it can create, expose, and clean up the resource automatically.

---

## 53. Hands-On Exercises

### Exercise 1 — beforeEach

Create a suite where every test starts at:

```text
/dashboard
```

### Exercise 2 — afterEach

Add cleanup after every test.

### Exercise 3 — beforeAll

Create safe suite-level reference data.

### Exercise 4 — afterAll

Remove the suite-level data.

### Exercise 5 — Describe Hooks

Create an Admin test group with its own setup.

### Exercise 6 — Hook + Fixture

Use an authenticated Page Object fixture and a `beforeEach` navigation hook.

### Exercise 7 — API Setup

Create test data before a test and clean it afterward.

### Exercise 8 — Parallel Workers

Run the suite with multiple workers and verify that `beforeAll` behavior is worker-scoped.

### Exercise 9 — Failure Cleanup

Force a test failure and verify that cleanup still happens.

### Exercise 10 — Refactor

Take a suite containing repeated setup code and refactor it into:

```text
Fixtures
+
Hooks
+
Page Objects
+
Utilities
```

---

## 54. Key Takeaways

- Hooks control Playwright test lifecycle operations.
- `beforeEach` runs before every test in its scope.
- `afterEach` runs after every test in its scope.
- `beforeAll` runs once per relevant worker/scope.
- `afterAll` runs after the relevant tests complete.
- Hooks can be scoped to `test.describe()`.
- Hooks can consume Playwright fixtures.
- Avoid sharing mutable browser state across tests.
- Avoid putting all setup logic into hooks.
- Use fixtures for reusable dependencies and resource lifecycle.
- Use storage state or authentication fixtures instead of repeated UI login where appropriate.
- Design test data for isolation.
- Remember that parallel workers affect `beforeAll` and `afterAll` behavior.
- Avoid arbitrary waits in hooks.
- Keep setup and cleanup responsibilities clear.
- Always consider teardown when a test fails.
- A good framework combines hooks, fixtures, Page Objects, APIs, and configuration rather than relying on one mechanism.

---

## 55. Enterprise Lifecycle Challenge

Build a complete test lifecycle:

```text
Environment
    ↓
Authentication
    ↓
Worker Setup
    ↓
Test Data Fixture
    ↓
beforeEach
    ↓
Test
    ↓
afterEach
    ↓
Fixture Cleanup
    ↓
afterAll
```

Requirements:

1. Use authentication storage state.
2. Create isolated test data.
3. Use a Page Object fixture.
4. Use `beforeEach` for navigation.
5. Use `afterEach` for safe diagnostics.
6. Use `beforeAll` only for safe shared setup.
7. Use `afterAll` for suite-level cleanup.
8. Run with multiple workers.
9. Verify tests remain independent.
10. Ensure cleanup works after failures.

---

## 56. Next Step

**Step 85 — Mastering Playwright Assertions, Auto-Waiting, Web-First Assertions, Soft Assertions, Custom Matchers & Advanced Validation**

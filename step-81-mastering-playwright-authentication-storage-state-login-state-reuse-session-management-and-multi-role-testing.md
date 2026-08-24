# Playwright with TypeScript Mastery Bootcamp

# Step 81 — Mastering Playwright Authentication, Storage State, Login State Reuse, Session Management & Multi-Role Testing

## 1. Introduction

Authentication is one of the most important parts of enterprise web automation.

Real applications commonly contain:

- Login pages
- Username and password authentication
- JWT authentication
- Session cookies
- Access tokens
- Refresh tokens
- OAuth authentication
- Multi-factor authentication
- Role-based access
- Admin and regular-user accounts
- Session expiration

Playwright provides several features for handling authenticated applications efficiently.

Important concepts include:

```text
Authentication
Storage State
Cookies
Local Storage
Login State Reuse
Authentication Fixtures
Multiple Roles
Session Management
```

---

## 2. Why Authentication Matters in Test Automation

A naive test suite may perform login repeatedly:

```text
Test 1 → Open browser → Login → Test
Test 2 → Open browser → Login → Test
Test 3 → Open browser → Login → Test
```

A better approach is:

```text
Authenticate Once
      ↓
Save Authentication State
      ↓
Reuse State
      ↓
Run Tests
```

This can make the suite faster and reduce dependency on the login UI.

---

## 3. Understanding Browser Authentication State

A logged-in browser session may contain:

```text
Cookies
Local Storage
Session Information
Authentication Tokens
```

Playwright can capture reusable browser storage state:

```ts
await page.context().storageState({
  path: 'playwright/.auth/user.json'
});
```

The saved state can later be reused by another browser context.

---

## 4. What Is Storage State?

Storage state represents persisted browser state that can be restored when creating a new context.

It primarily contains:

- Cookies
- Local storage entries

Example:

```ts
await context.storageState({
  path: 'playwright/.auth/user.json'
});
```

---

## 5. Why Use Storage State?

Without storage state:

```text
Test
 ↓
Login UI
 ↓
Dashboard
```

With storage state:

```text
Stored Authentication
 ↓
Authenticated Context
 ↓
Dashboard
```

This avoids unnecessary repeated login actions.

---

## 6. Authentication Setup Project

A common project structure is:

```text
playwright/
├── tests/
├── pages/
├── fixtures/
├── test-data/
├── utils/
├── .auth/
├── playwright.config.ts
└── package.json
```

Authentication state should not be committed when it contains active credentials or sessions.

---

## 7. Login Setup Test

Example:

```ts
import { test as setup, expect } from '@playwright/test';

setup('authenticate user', async ({ page }) => {
  await page.goto('/login');

  await page.getByLabel('Username')
    .fill(process.env.TEST_USERNAME!);

  await page.getByLabel('Password')
    .fill(process.env.TEST_PASSWORD!);

  await page.getByRole('button', {
    name: 'Login'
  }).click();

  await expect(
    page.getByRole('heading', {
      name: 'Dashboard'
    })
  ).toBeVisible();

  await page.context().storageState({
    path: 'playwright/.auth/user.json'
  });
});
```

---

## 8. Reusing Storage State

Configure the project:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    storageState: 'playwright/.auth/user.json'
  }
});
```

Tests can now begin with the stored authentication state.

---

## 9. Authenticated Test

```ts
import { test, expect } from '@playwright/test';

test('authenticated dashboard', async ({ page }) => {
  await page.goto('/dashboard');

  await expect(
    page.getByRole('heading', {
      name: 'Dashboard'
    })
  ).toBeVisible();
});
```

No login UI interaction is necessary.

---

## 10. Authentication Project Dependency

A setup project can run before authenticated tests:

```ts
projects: [
  {
    name: 'setup',
    testMatch: /.*\.setup\.ts/
  },
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome'],
      storageState: 'playwright/.auth/user.json'
    },
    dependencies: ['setup']
  }
]
```

This creates a workflow:

```text
Authentication Setup
        ↓
Authenticated Tests
```

---

## 11. API Login

Some applications expose an authentication API.

Example:

```ts
const response = await request.post('/api/login', {
  data: {
    username: process.env.TEST_USERNAME,
    password: process.env.TEST_PASSWORD
  }
});

expect(response.ok()).toBeTruthy();
```

API authentication can sometimes be faster than UI login.

---

## 12. API Login + Browser Authentication

A possible enterprise architecture is:

```text
API Login
   ↓
Authentication Cookie / Token
   ↓
Browser Context
   ↓
Authenticated UI
```

The exact implementation depends on whether the application uses cookies, local storage, headers, or another mechanism.

---

## 13. Inspecting Cookies

Use:

```ts
const cookies = await page.context().cookies();

console.log(
  cookies.map(cookie => cookie.name)
);
```

Avoid logging cookie values because they may contain active session credentials.

---

## 14. Adding Cookies

Example:

```ts
await context.addCookies([
  {
    name: 'session',
    value: 'test-session',
    domain: 'example.com',
    path: '/'
  }
]);
```

Use controlled test values only.

---

## 15. Local Storage Authentication

Some applications store authentication information in local storage.

Example:

```ts
const token = await page.evaluate(() => {
  return localStorage.getItem('accessToken');
});
```

Never print real access tokens into test reports or CI logs.

---

## 16. Setting Local Storage

A page can initialize test data before navigation:

```ts
await page.addInitScript(() => {
  localStorage.setItem(
    'testMode',
    'true'
  );
});
```

For authentication, prefer the application's supported authentication flow unless the test specifically targets token injection.

---

## 17. Session Management

Authentication state can expire because of:

- Session timeout
- Token expiration
- Logout
- Server-side invalidation
- Refresh-token expiration
- Password changes
- Security policies

Tests should distinguish between:

```text
Valid Session
Expired Session
Invalid Session
Logged-Out Session
```

---

## 18. Testing Session Expiration

A session-expiration scenario can be simulated by mocking a protected API:

```ts
await page.route('**/api/profile', async route => {
  await route.fulfill({
    status: 401,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'Session expired'
    })
  });
});
```

Then verify the expected redirect or message.

---

## 19. Testing Logout

```ts
await page.getByRole('button', {
  name: 'Logout'
}).click();

await expect(
  page.getByRole('heading', {
    name: 'Login'
  })
).toBeVisible();
```

The exact assertion depends on the application's logout behavior.

---

## 20. Protected Routes

Example:

```ts
await page.goto('/admin');

await expect(
  page.getByRole('heading', {
    name: 'Login'
  })
).toBeVisible();
```

Protected routes should prevent unauthorized access.

---

## 21. Multi-Role Testing

Enterprise applications often contain roles such as:

```text
Admin
Manager
Employee
Viewer
Auditor
```

Different roles may have different permissions.

A framework can maintain separate authentication states:

```text
playwright/.auth/
├── admin.json
├── manager.json
└── employee.json
```

---

## 22. Role-Based Storage State

Admin tests:

```ts
storageState:
  'playwright/.auth/admin.json'
```

Employee tests:

```ts
storageState:
  'playwright/.auth/employee.json'
```

This keeps role-specific authentication isolated.

---

## 23. Role-Based Projects

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

---

## 24. Authentication vs Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to do?
```

For example:

```text
Admin
 ├── Create User
 ├── Edit User
 └── Delete User

Employee
 ├── View Profile
 └── Edit Own Profile
```

Both areas should be tested.

---

## 25. Admin Authorization Test

```ts
test('admin can access user management', async ({ page }) => {
  await page.goto('/admin/users');

  await expect(
    page.getByRole('heading', {
      name: 'User Management'
    })
  ).toBeVisible();
});
```

---

## 26. Employee Authorization Test

```ts
test('employee cannot access user management', async ({
  page
}) => {
  await page.goto('/admin/users');

  await expect(
    page.getByText('Access denied')
  ).toBeVisible();
});
```

The expected result should match the application's security behavior.

---

## 27. Separate Browser Contexts for Multiple Users

For workflows involving multiple users:

```ts
const adminContext =
  await browser.newContext({
    storageState:
      'playwright/.auth/admin.json'
  });

const employeeContext =
  await browser.newContext({
    storageState:
      'playwright/.auth/employee.json'
  });

const adminPage =
  await adminContext.newPage();

const employeePage =
  await employeeContext.newPage();
```

Each context maintains an independent session.

---

## 28. Multi-User Collaboration Testing

Example workflow:

```text
Admin
 ↓
Creates Approval Request

Employee
 ↓
Receives Request

Manager
 ↓
Approves Request

Employee
 ↓
Verifies Approved Status
```

Separate contexts allow each user to interact independently.

---

## 29. Parallel Authentication State

Parallel tests should avoid modifying the same shared state.

Prefer separate files such as:

```text
admin.json
manager.json
employee.json
```

For highly isolated scenarios, generate state specifically for each worker or test.

---

## 30. Authentication State Security

Authentication state can contain sensitive information.

Therefore:

- Do not commit `.auth` files.
- Add authentication directories to `.gitignore`.
- Do not upload them as public CI artifacts.
- Do not log tokens.
- Use dedicated test accounts.
- Prefer short-lived credentials.

Example:

```text
playwright/.auth/
```

in `.gitignore`.

---

## 31. CI/CD Credentials

Use environment variables:

```ts
process.env.TEST_USERNAME
process.env.TEST_PASSWORD
```

CI systems should store credentials as protected secrets.

Avoid:

```ts
const username = 'admin';
const password = 'admin123';
```

---

## 32. Login Once vs Login Per Test

### Login per test

Advantages:

- Strong isolation
- Fresh authentication state

Disadvantages:

- Slower
- Repeated UI work
- More dependency on login functionality

### Reused storage state

Advantages:

- Faster
- Less repetition
- Cleaner business-focused tests

Disadvantages:

- State can become stale
- Shared state can cause coupling if tests mutate it

---

## 33. When Fresh Authentication Is Better

Use fresh authentication for:

- Login tests
- Logout tests
- Password changes
- MFA tests
- Account lockout
- Session expiration
- Authentication failures
- Security-control tests

These scenarios should directly test authentication behavior.

---

## 34. When Storage State Is Better

Use reusable authentication state for:

- Dashboards
- CRUD workflows
- Reports
- Search
- Business workflows
- Admin features

This keeps tests focused on business functionality.

---

## 35. Authentication Fixture

A reusable authenticated-page fixture can look like:

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState:
        'playwright/.auth/user.json'
    });

    const page = await context.newPage();

    await use(page);

    await context.close();
  }
});
```

Then:

```ts
test('dashboard', async ({
  authenticatedPage
}) => {
  await authenticatedPage.goto('/dashboard');
});
```

---

## 36. Role-Based Fixtures

A framework can expose fixtures such as:

```text
adminPage
managerPage
employeePage
```

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

This makes tests easy to understand.

---

## 37. Authentication State Refresh

Stored sessions may eventually become invalid.

A setup project can regenerate authentication state:

```text
Auth Setup
    ↓
Generate State
    ↓
Dependent Tests
```

This is generally easier to maintain than manually refreshing state throughout tests.

---

## 38. Authentication Failure Diagnostics

When authentication fails, useful evidence includes:

- Screenshot
- Trace
- Safe response status
- Redirect URL
- Non-sensitive logs

Avoid collecting:

- Passwords
- Access tokens
- Session cookies
- Secret headers

---

## 39. Authentication with Trace

Tracing can help diagnose authentication workflows:

```bash
npx playwright test --trace on
```

You can inspect:

```text
Login
 ↓
Redirect
 ↓
Dashboard
```

Sensitive information should not be shared through trace artifacts.

---

## 40. Common Mistakes

### Mistake 1 — Committing authentication files

Never commit active authentication state.

### Mistake 2 — Hard-coding credentials

Use secure environment variables.

### Mistake 3 — Using one role for every test

Authorization defects may be missed.

### Mistake 4 — Logging in on every test

This unnecessarily slows large suites.

### Mistake 5 — Sharing mutable state blindly

A test that changes the session can affect other tests.

### Mistake 6 — Logging tokens

Never print secrets.

---

## 41. Production Authentication Architecture

A scalable project can use:

```text
playwright/
├── auth/
│   ├── admin.setup.ts
│   ├── employee.setup.ts
│   └── manager.setup.ts
├── .auth/
│   ├── admin.json
│   ├── employee.json
│   └── manager.json
├── fixtures/
│   └── auth.fixture.ts
├── pages/
├── tests/
├── utils/
└── playwright.config.ts
```

The exact structure can be adapted to the project.

---

## 42. Authentication Flow Architecture

```text
                Authentication
                       │
             ┌─────────┴─────────┐
             │                   │
          UI Login            API Login
             │                   │
             └─────────┬─────────┘
                       ↓
                Authenticated State
                       ↓
                 Storage State
                       ↓
              ┌────────┼────────┐
              ↓        ↓        ↓
            Admin    Manager  Employee
              ↓        ↓        ↓
            Tests     Tests    Tests
```

---

## 43. End-to-End Multi-Role Example

```text
Admin
 ↓
Creates Expense Policy

Employee
 ↓
Creates Expense

Manager
 ↓
Approves Expense

Employee
 ↓
Verifies Approved Status
```

Each role can operate in an independent browser context.

---

## 44. Interview Questions

### Q1. What is storage state in Playwright?

Storage state is persisted browser authentication-related state, primarily cookies and local storage, that can be reused when creating browser contexts.

### Q2. Why use storage state?

It avoids repeating UI login and can speed up authenticated test suites.

### Q3. Where should authentication state be stored?

A common location is:

```text
playwright/.auth/
```

These files should not be committed.

### Q4. How do you configure storage state?

```ts
use: {
  storageState: 'playwright/.auth/user.json'
}
```

### Q5. Can different roles have different authentication states?

Yes.

```text
admin.json
manager.json
employee.json
```

### Q6. How do you test multiple users simultaneously?

Create separate browser contexts with separate storage states.

### Q7. What is the difference between authentication and authorization?

Authentication identifies the user. Authorization determines what the authenticated user is allowed to do.

### Q8. Should every test perform login?

No. Reuse authentication state when login itself is not the subject under test.

### Q9. When should you perform a fresh login?

For login, logout, MFA, account lockout, session expiration, and other authentication-specific scenarios.

### Q10. How should credentials be managed?

Use secure environment variables or CI/CD secrets rather than hard-coded credentials.

---

## 45. Hands-On Exercises

### Exercise 1 — Login Setup

Create an authentication setup test that saves:

```text
playwright/.auth/user.json
```

### Exercise 2 — Reuse Storage State

Configure the project to use the saved state and open a protected dashboard without logging in.

### Exercise 3 — Admin Role

Create:

```text
playwright/.auth/admin.json
```

and verify admin-only functionality.

### Exercise 4 — Employee Role

Create:

```text
playwright/.auth/employee.json
```

and verify restricted functionality.

### Exercise 5 — Authorization

Verify that an employee cannot access an admin endpoint.

### Exercise 6 — Multi-User Workflow

Create separate contexts for two users and test a collaboration workflow.

### Exercise 7 — Logout

Verify that logout invalidates the authenticated experience.

### Exercise 8 — Session Expiration

Simulate an expired session and verify that the application redirects to login.

### Exercise 9 — Authentication Fixture

Create reusable fixtures for:

```text
adminPage
managerPage
employeePage
```

### Exercise 10 — CI Credentials

Move credentials from source code into environment variables.

---

## 46. Key Takeaways

- Authentication is a core part of enterprise Playwright automation.
- Storage state allows authentication state to be reused.
- `storageState` can contain cookies and local storage.
- Authentication setup projects can generate reusable state.
- API login can sometimes be faster than UI login.
- Separate storage states should be used for different roles.
- Separate browser contexts are useful for multi-user workflows.
- Authentication and authorization are different testing concerns.
- Fresh login should be used for authentication-specific tests.
- Reusable storage state is ideal for most authenticated business tests.
- Authentication files must never be committed with active credentials.
- Credentials should come from secure environment variables or CI secrets.
- Never log passwords, access tokens, or session cookies.
- Multi-role testing is essential for applications with role-based permissions.
- Authentication fixtures can make enterprise test suites cleaner and more maintainable.

---

## 47. Hands-On Challenge

Build an authentication framework that supports:

```text
Admin
Manager
Employee
```

Requirements:

1. Create authentication setup tests.
2. Generate separate storage-state files.
3. Configure role-based projects.
4. Create role-specific fixtures.
5. Test protected routes.
6. Test authorization boundaries.
7. Test logout.
8. Test session expiration.
9. Run selected roles independently.
10. Keep credentials and authentication state outside source control.

---

## 48. Next Step

**Step 82 — Mastering Playwright Advanced Fixtures, Test Fixtures, Worker Fixtures, Fixture Composition & Enterprise Dependency Injection**

# Playwright with TypeScript Mastery Bootcamp

# Step 93 — Mastering Playwright Authentication & Session Management: Storage State, Cookies, Local Storage, Session Storage, JWT, OAuth2, Multi-Role Login & Enterprise Auth Fixtures

## 1. Introduction

Authentication is one of the most important areas of enterprise Playwright automation.

Modern applications may use:

```text
Username + Password
Session Cookies
JWT
OAuth2
OpenID Connect
SSO
Multi-Factor Authentication
Role-Based Access
Refresh Tokens
```

A test framework should avoid repeating expensive login steps in every test.

A scalable authentication architecture looks like:

```text
Authenticate Once
      ↓
Save Authentication State
      ↓
Reuse State
      ↓
Run Tests
```

Playwright provides powerful support for:

- Browser storage state
- Cookies
- Local Storage
- Session Storage
- Authentication fixtures
- Multiple browser contexts
- API-based authentication
- Multi-role sessions

---

## 2. Why Authentication Architecture Matters

A poorly designed framework may do this for every test:

```text
Open Browser
 ↓
Open Login Page
 ↓
Enter Username
 ↓
Enter Password
 ↓
Click Login
 ↓
Wait for Dashboard
 ↓
Run Test
```

If there are 500 tests, authentication can become expensive.

A better strategy is:

```text
Login Once
 ↓
Save State
 ↓
Reuse State
 ↓
Run 500 Tests
```

This makes tests faster and more stable.

---

## 3. Authentication vs Authorization

These concepts are different.

### Authentication

Answers:

```text
Who are you?
```

Example:

```text
Username + Password
```

### Authorization

Answers:

```text
What are you allowed to do?
```

Example:

```text
Admin → Can manage users
Employee → Cannot manage users
```

Both must be tested.

---

## 4. Common Authentication Mechanisms

Enterprise applications commonly use:

```text
Session Cookie
JWT
OAuth2
OpenID Connect
SAML / SSO
API Key
Basic Authentication
```

Playwright can automate many of these through browser and API capabilities.

---

## 5. Playwright Authentication State

Playwright supports storing browser authentication state.

Typical configuration:

```ts
use: {
  storageState:
    '.auth/user.json'
}
```

This state can contain authentication information such as cookies and origin storage.

---

## 6. Storage State File

A common structure:

```text
playwright/
├── .auth/
│   ├── user.json
│   ├── admin.json
│   └── manager.json
```

Example:

```text
.auth/user.json
```

This file should normally be treated as sensitive.

Do not commit real authenticated state files to source control.

---

## 7. Authentication Setup Project

A scalable Playwright project can use a setup project:

```text
projects:
  setup
      ↓
  chromium
  firefox
  webkit
```

The setup project authenticates first.

Then browser projects reuse the generated authentication state.

---

## 8. Basic Authentication Setup

Example:

```ts
import {
  test as setup,
  expect
} from '@playwright/test';

const authFile =
  '.auth/user.json';

setup(
  'authenticate',
  async ({ page }) => {

    await page.goto(
      '/login'
    );

    await page
      .getByLabel(
        'Username'
      )
      .fill(
        process.env.TEST_USERNAME!
      );

    await page
      .getByLabel(
        'Password'
      )
      .fill(
        process.env.TEST_PASSWORD!
      );

    await page.getByRole(
      'button',
      { name: 'Login' }
    ).click();

    await expect(
      page.getByText(
        'Dashboard'
      )
    ).toBeVisible();

    await page.context()
      .storageState({
        path: authFile
      });
  }
);
```

---

## 9. Why Use `storageState()`?

It captures the browser authentication state so later tests can reuse it.

Conceptually:

```text
Login
 ↓
Cookies / Storage
 ↓
storageState()
 ↓
.auth/user.json
```

Later:

```text
.auth/user.json
 ↓
Browser Context
 ↓
Already Authenticated
```

---

## 10. Reusing Storage State

In configuration:

```ts
use: {
  storageState:
    '.auth/user.json'
}
```

Tests can then start with the stored authentication state.

---

## 11. Storage State and Cookies

Session-based applications often store authentication in cookies.

Example conceptual cookie:

```text
sessionid=abc123
```

The storage state captures the cookie so the next context can use the authenticated session.

---

## 12. Inspecting Cookies

Playwright can inspect cookies through the browser context.

Example:

```ts
const cookies =
  await page.context()
    .cookies();

console.log(
  cookies.map(
    cookie => cookie.name
  )
);
```

Avoid logging cookie values.

---

## 13. Cookie Properties

Cookies can contain:

```text
name
value
domain
path
expires
httpOnly
secure
sameSite
```

Security-sensitive attributes include:

```text
HttpOnly
Secure
SameSite
```

---

## 14. HttpOnly Cookies

An HttpOnly cookie cannot normally be accessed by client-side JavaScript.

This is often used for session authentication.

Playwright context APIs can still work with the browser's cookie state.

---

## 15. Secure Cookies

A secure cookie is intended to be transmitted over secure connections.

Production authentication should normally use HTTPS.

Tests should match the actual deployment security configuration.

---

## 16. SameSite Cookies

SameSite controls when cookies are sent in cross-site scenarios.

Common values:

```text
Strict
Lax
None
```

Cross-site authentication flows can depend on this behavior.

---

## 17. Adding Cookies Manually

Playwright can add cookies to a browser context:

```ts
await context.addCookies([
  {
    name: 'session',
    value: 'test-session',
    domain:
      'example.test',
    path: '/'
  }
]);
```

Use synthetic test values.

Do not manually inject real production session cookies.

---

## 18. Clearing Cookies

Use:

```ts
await context.clearCookies();
```

This is useful for testing logged-out behavior.

Example:

```text
Authenticated
 ↓
Clear Cookies
 ↓
Refresh
 ↓
Login Required
```

---

## 19. Testing Logout

A logout test should verify:

```text
User Authenticated
 ↓
Click Logout
 ↓
Session Removed
 ↓
Protected Page
 ↓
Redirect/Login Required
```

Example:

```ts
await page.getByRole(
  'button',
  { name: 'Logout' }
).click();

await expect(
  page
).toHaveURL(
  /login/
);
```

The exact URL should match the application.

---

## 20. Local Storage

Applications may store authentication-related information in local storage.

Example:

```text
localStorage
 ├── accessToken
 ├── user
 └── preferences
```

Playwright storage state can preserve origin storage.

---

## 21. Inspecting Local Storage

Browser JavaScript can read local storage:

```ts
const token =
  await page.evaluate(
    () =>
      localStorage.getItem(
        'accessToken'
      )
  );
```

Never print real access tokens.

---

## 22. Setting Local Storage

Example:

```ts
await page.evaluate(() => {
  localStorage.setItem(
    'theme',
    'dark'
  );
});
```

For authentication state, prefer the application's real authentication flow or carefully controlled test fixtures rather than bypassing important security behavior without a reason.

---

## 23. Session Storage

Session storage is scoped to a page/tab session.

Example:

```ts
await page.evaluate(() => {
  sessionStorage.setItem(
    'testMode',
    'true'
  );
});
```

It is different from local storage.

---

## 24. Local Storage vs Session Storage

| Feature | Local Storage | Session Storage |
|---|---|---|
| Persistence | Longer-lived | Page/session scoped |
| Shared across tabs | Origin-dependent behavior | Generally isolated per tab |
| Typical use | Preferences/tokens | Temporary session data |
| Browser restart | Usually persists | Usually does not |

Always follow the application's actual storage behavior.

---

## 25. JWT Authentication

JWT commonly follows:

```text
Login
 ↓
Access Token
 ↓
Authorization Header
 ↓
Protected API
```

Header:

```ts
Authorization:
  `Bearer ${token}`
```

A JWT usually contains:

```text
Header
Payload
Signature
```

Do not place real production JWTs in source code.

---

## 26. JWT Access Token Testing

Test scenarios:

```text
Valid token
Invalid token
Expired token
Missing token
Insufficient permissions
```

Expected behavior depends on the API contract.

---

## 27. Refresh Token

A common architecture:

```text
Access Token
      ↓
Expires
      ↓
Refresh Token
      ↓
New Access Token
```

Test the refresh workflow separately.

---

## 28. Refresh Token Test Strategy

A robust test can validate:

```text
Authenticated Request
 ↓
Access Token Expired
 ↓
Refresh Endpoint
 ↓
New Access Token
 ↓
Original Request Succeeds
```

Do not rely on artificial token manipulation if the application has a safer test mechanism for expiry.

---

## 29. OAuth2

OAuth2 commonly involves:

```text
Client
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Token Exchange
 ↓
Access Token
 ↓
Resource Server
```

The exact flow depends on the provider and application.

---

## 30. Authorization Code Flow

Typical flow:

```text
Application
 ↓
Authorization Server
 ↓
Login
 ↓
Consent
 ↓
Authorization Code
 ↓
Application
 ↓
Token Exchange
 ↓
Access Token
```

Browser automation may be required for the interactive login portion.

---

## 31. OpenID Connect

OpenID Connect builds identity functionality on top of OAuth2.

Typical concepts:

```text
ID Token
Access Token
UserInfo
Issuer
Client ID
Redirect URI
```

Enterprise SSO systems may use OpenID Connect.

---

## 32. SSO Testing

SSO flows may involve:

```text
Application
 ↓
Identity Provider
 ↓
Login
 ↓
Redirect
 ↓
Application
```

Test:

```text
Successful SSO
Failed login
Unauthorized user
Expired session
Logout
Redirect behavior
```

Avoid automating external identity providers in every test if a supported test authentication strategy exists.

---

## 33. Authentication State Strategy

A scalable framework can use:

```text
Setup Project
      ↓
Authenticate
      ↓
Save State
      ↓
Browser Projects
      ↓
Reuse State
```

This reduces repeated login operations.

---

## 34. Multiple User Roles

Enterprise applications often have:

```text
Admin
Manager
Employee
Viewer
Auditor
```

Each role may require different permissions.

Create separate states:

```text
.auth/
├── admin.json
├── manager.json
├── employee.json
└── viewer.json
```

Use synthetic test accounts.

---

## 35. Multi-Role Test Architecture

Example:

```text
Setup
 ├── Admin Login
 ├── Manager Login
 └── Employee Login
       ↓
Save States
       ↓
Role-Based Tests
```

This makes authorization testing much easier.

---

## 36. Role-Based Configuration

Projects can use different storage states.

Conceptually:

```ts
projects: [
  {
    name: 'admin',
    use: {
      storageState:
        '.auth/admin.json'
    }
  },
  {
    name: 'employee',
    use: {
      storageState:
        '.auth/employee.json'
    }
  }
]
```

Use the exact Playwright configuration structure appropriate to the project.

---

## 37. Admin Test

An admin test might verify:

```text
Admin
 ↓
Open Admin Panel
 ↓
User Management Visible
```

Example:

```ts
await expect(
  page.getByText(
    'User Management'
  )
).toBeVisible();
```

---

## 38. Employee Test

An employee test might verify:

```text
Employee
 ↓
Open Admin URL
 ↓
Access Denied
```

Example:

```ts
await page.goto(
  '/admin'
);

await expect(
  page.getByRole(
    'alert'
  )
).toContainText(
  'Access denied'
);
```

Use the application's actual behavior.

---

## 39. Permission Matrix

Create a matrix:

| Role | Dashboard | Users | Reports | Settings |
|---|---|---|---|---|
| Admin | Yes | Yes | Yes | Yes |
| Manager | Yes | Limited | Yes | Limited |
| Employee | Yes | No | Own | No |
| Viewer | Yes | No | Read | No |

The actual permissions should come from the application's requirements.

---

## 40. Authentication Fixture

A reusable fixture can expose:

```text
adminPage
managerPage
employeePage
```

or authentication states.

Example concept:

```ts
type AuthFixtures = {
  adminPage: Page;
  employeePage: Page;
};
```

The fixture can create contexts using the appropriate storage state.

---

## 41. Authentication Fixture Benefits

Fixtures can centralize:

```text
Login
Storage State
Role
Browser Context
Cleanup
```

Tests become easier to read:

```ts
test(
  'admin can manage users',
  async ({
    adminPage
  }) => {

    await adminPage.goto(
      '/admin/users'
    );

  }
);
```

---

## 42. Role-Specific Contexts

For tests involving multiple users simultaneously:

```text
Admin Context
 └── Admin Page

Employee Context
 └── Employee Page
```

Each context maintains separate cookies and storage.

This is useful for:

```text
Approval workflows
Chat
Collaboration
Permission changes
Multi-user transactions
```

---

## 43. Multiple Browser Contexts

Example:

```ts
const adminContext =
  await browser.newContext({
    storageState:
      '.auth/admin.json'
  });

const employeeContext =
  await browser.newContext({
    storageState:
      '.auth/employee.json'
  });

const adminPage =
  await adminContext.newPage();

const employeePage =
  await employeeContext.newPage();
```

Remember to close custom contexts.

---

## 44. Multi-User Workflow

Example:

```text
Employee
 ↓
Submit Expense
 ↓
Manager
 ↓
Approve Expense
 ↓
Employee
 ↓
Verify Approved
```

Use separate contexts to represent separate users.

---

## 45. Authentication and API Setup

API authentication can also create test state.

Example:

```text
API Login
 ↓
Create Data
 ↓
Browser Context
 ↓
UI Verification
```

This is often faster than performing setup entirely through the UI.

---

## 46. Authentication State Security

Authentication state files can contain sensitive session information.

Treat them as secrets.

Recommended:

```text
.auth/
```

should generally be excluded from Git:

```gitignore
.auth/
```

Do not upload real authenticated state files to public repositories.

---

## 47. CI Authentication

CI should obtain credentials securely through:

```text
CI Secret Variables
Environment Variables
Secret Managers
```

Never:

```text
Hard-code passwords
Commit tokens
Print credentials
Store production cookies
```

---

## 48. Environment Configuration

Example:

```text
TEST_USERNAME
TEST_PASSWORD
ADMIN_USERNAME
ADMIN_PASSWORD
API_BASE_URL
BASE_URL
```

Use environment-specific configuration.

---

## 49. Authentication Setup Failure

If authentication fails, the setup project should fail clearly.

Example:

```ts
if (
  !username ||
  !password
) {
  throw new Error(
    'Authentication credentials are missing'
  );
}
```

Do not continue with an invalid authentication state.

---

## 50. Verify Authentication State

After login, verify a reliable application state:

```ts
await expect(
  page.getByText(
    'Dashboard'
  )
).toBeVisible();
```

Do not assume that clicking Login means authentication succeeded.

---

## 51. Verify Storage State

After saving:

```ts
await page.context()
  .storageState({
    path: authFile
  });
```

You can inspect the resulting state during debugging.

Avoid printing sensitive values.

---

## 52. Authentication Expiry

Stored authentication state may expire.

Symptoms:

```text
401
Redirect to login
Dashboard unavailable
Tests suddenly fail
```

The setup project should regenerate state when needed.

---

## 53. Authentication State Lifecycle

Recommended:

```text
Start Test Run
 ↓
Check / Generate Auth State
 ↓
Run Tests
 ↓
State Used
 ↓
End Test Run
```

Do not assume authentication state remains valid forever.

---

## 54. Re-Authentication

If the application has short-lived sessions, consider:

```text
Fresh setup
Per suite setup
Per role setup
```

Choose based on session lifetime and test-suite performance.

---

## 55. Authentication and Parallelism

Parallel tests should use isolated browser contexts.

Avoid:

```text
Shared mutable cookies
Shared global page
Shared session state
```

Each test should receive its own context through Playwright fixtures.

---

## 56. Authentication and Test Isolation

A test should not depend on another test logging in.

Correct:

```text
Test A → Own Context
Test B → Own Context
Test C → Own Context
```

The authentication state can be reused as an immutable starting point.

---

## 57. Storage State vs Manual Login

### Storage State

Advantages:

- Fast
- Reusable
- Scalable
- Less repetitive

### Manual Login

Advantages:

- Tests actual login UI
- Validates login functionality
- Useful for authentication-specific tests

A mature suite uses both.

---

## 58. Authentication Test Layers

Use:

```text
Login UI Tests
       ↓
API Authentication Tests
       ↓
Storage State Reuse
       ↓
Authenticated Feature Tests
       ↓
Authorization Tests
```

Do not remove login tests just because other tests reuse storage state.

---

## 59. Testing Invalid Credentials

Example:

```ts
await page
  .getByLabel(
    'Username'
  )
  .fill(
    'invalid-user'
  );

await page
  .getByLabel(
    'Password'
  )
  .fill(
    'invalid-password'
  );

await page.getByRole(
  'button',
  { name: 'Login' }
).click();

await expect(
  page.getByRole(
    'alert'
  )
).toBeVisible();
```

Validate the actual error message where appropriate.

---

## 60. Testing Logout and Session Invalidation

After logout:

```text
Session Active
 ↓
Logout
 ↓
Session Invalidated
 ↓
Protected URL
 ↓
Login Required
```

Also consider validating that an API request using the old session is rejected when the application's architecture supports such a check.

---

## 61. Testing Session Timeout

Possible workflow:

```text
Login
 ↓
Session expires
 ↓
User performs action
 ↓
Session-expired behavior
```

The safest implementation depends on how the application exposes testable session expiration.

---

## 62. Testing Unauthorized URLs

Example:

```ts
await page.goto(
  '/admin/users'
);
```

As an employee, verify:

```text
403
or
Redirect
or
Access Denied
```

Use the application's expected behavior.

---

## 63. Testing Direct URL Access

Authorization should not depend only on menu visibility.

Test:

```text
Role
 ↓
Direct Protected URL
 ↓
Expected Authorization Result
```

This catches UI-only permission hiding.

---

## 64. Testing API Authorization

Combine browser role testing with API checks:

```text
Employee Token
 ↓
GET /api/admin/users
 ↓
403
```

This validates backend authorization.

---

## 65. Multi-Role Authentication Matrix

A useful test matrix:

| Scenario | Admin | Manager | Employee |
|---|---|---|---|
| Login | ✓ | ✓ | ✓ |
| Users | ✓ | Limited | ✗ |
| Reports | ✓ | ✓ | Limited |
| Settings | ✓ | Limited | ✗ |
| Admin API | ✓ | Depends | ✗ |

Use actual application requirements.

---

## 66. OAuth2 Testing Best Practices

Avoid repeating full external OAuth login in every test.

Prefer:

```text
Dedicated Auth Test
       ↓
Validate OAuth Flow
       ↓
Controlled Auth State
       ↓
Feature Tests
```

This reduces external dependency and test flakiness.

---

## 67. SSO Testing Best Practices

For enterprise SSO:

```text
Smoke SSO Tests
       ↓
Full Authentication Validation

Feature Tests
       ↓
Reuse Controlled Auth State
```

Use a test identity provider or supported test environment where possible.

---

## 68. MFA Testing

Multi-factor authentication may involve:

```text
Password
 ↓
OTP
 ↓
Verification
 ↓
Session
```

Do not put real personal MFA information into automation.

Use a controlled test MFA mechanism where available.

---

## 69. Authentication Test Data

Use dedicated accounts:

```text
automation-admin
automation-manager
automation-employee
```

Avoid using real employee accounts.

Keep credentials in secure configuration.

---

## 70. Authentication Utilities

A framework may include:

```text
utils/
└── authUtils.ts
```

Possible responsibilities:

```text
loginViaApi()
createAuthState()
validateAuthState()
clearAuthState()
```

Keep utilities focused and avoid hiding business assertions.

---

## 71. Authentication Project Structure

Recommended:

```text
playwright/
├── .auth/
│   ├── admin.json
│   ├── manager.json
│   └── employee.json
│
├── auth/
│   ├── auth.setup.ts
│   └── auth.utils.ts
│
├── fixtures/
│   └── auth.fixture.ts
│
├── pages/
│   └── LoginPage.ts
│
├── tests/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   ├── logout.spec.ts
│   │   └── authorization.spec.ts
│   │
│   └── admin/
│       └── users.spec.ts
│
└── playwright.config.ts
```

---

## 72. Production-Quality Authentication Setup

```ts
import {
  test as setup,
  expect
} from '@playwright/test';

const authFile =
  '.auth/user.json';

setup(
  'authenticate user',
  async ({ page }) => {

    const username =
      process.env.TEST_USERNAME;

    const password =
      process.env.TEST_PASSWORD;

    if (
      !username ||
      !password
    ) {
      throw new Error(
        'Authentication credentials are missing'
      );
    }

    await page.goto(
      '/login'
    );

    await page
      .getByLabel(
        'Username'
      )
      .fill(username);

    await page
      .getByLabel(
        'Password'
      )
      .fill(password);

    await page.getByRole(
      'button',
      { name: 'Login' }
    ).click();

    await expect(
      page.getByText(
        'Dashboard'
      )
    ).toBeVisible();

    await page.context()
      .storageState({
        path: authFile
      });
  }
);
```

---

## 73. Production-Quality Role States

Conceptually:

```text
.auth/
├── admin.json
├── manager.json
└── employee.json
```

Each setup flow should:

```text
Login
 ↓
Verify Correct Role
 ↓
Save State
```

Do not save a state until the correct identity is confirmed.

---

## 74. Production-Quality Multi-User Example

```ts
test(
  'employee submits and manager approves',
  async ({
    browser
  }) => {

    const employeeContext =
      await browser.newContext({
        storageState:
          '.auth/employee.json'
      });

    const managerContext =
      await browser.newContext({
        storageState:
          '.auth/manager.json'
      });

    const employeePage =
      await employeeContext
        .newPage();

    const managerPage =
      await managerContext
        .newPage();

    try {

      await employeePage.goto(
        '/expenses'
      );

      // Employee submits expense

      await managerPage.goto(
        '/approvals'
      );

      // Manager approves expense

    } finally {

      await employeeContext.close();
      await managerContext.close();
    }
  }
);
```

The actual business actions should be implemented through Page Objects.

---

## 75. Common Authentication Mistakes

### Mistake 1

Logging in through the UI for every test.

### Mistake 2

Committing `.auth/*.json`.

### Mistake 3

Printing passwords or tokens.

### Mistake 4

Sharing mutable browser context between tests.

### Mistake 5

Assuming login succeeded without verification.

### Mistake 6

Testing only menu visibility for authorization.

### Mistake 7

Ignoring direct URL authorization.

### Mistake 8

Ignoring API authorization.

### Mistake 9

Using real employee accounts.

### Mistake 10

Assuming authentication state never expires.

---

## 76. Best Practices

1. Use setup projects for reusable authentication state.
2. Keep authentication-specific tests separate from feature tests.
3. Store credentials securely.
4. Never commit real authentication state.
5. Use dedicated automation accounts.
6. Create separate states for important roles.
7. Verify the authenticated identity before saving state.
8. Use isolated browser contexts.
9. Test both authentication and authorization.
10. Test direct protected URLs.
11. Test API authorization where relevant.
12. Handle session expiration.
13. Keep SSO/OAuth tests controlled and deterministic.
14. Use API authentication where appropriate for fast setup.
15. Keep authentication fixtures reusable but transparent.

---

## 77. Interview Questions

### Q1. What is `storageState`?

It is Playwright's mechanism for saving and reusing browser authentication/storage state.

### Q2. Why use storage state?

To avoid repeating expensive login flows in every test.

### Q3. What can authentication state contain?

It can include browser cookies and origin storage used by the application.

### Q4. Should `.auth/*.json` be committed?

No, authenticated state may contain sensitive session information.

### Q5. What is the difference between authentication and authorization?

Authentication verifies identity; authorization verifies permissions.

### Q6. How do you test multiple roles?

Create separate authentication states or isolated browser contexts for each role.

### Q7. How can API authentication help UI tests?

API login or setup can create authenticated state or test data more quickly than repeating UI interactions.

### Q8. What is the difference between local storage and session storage?

Local storage generally persists longer, while session storage is associated with the page/session lifecycle.

### Q9. How do you test authorization properly?

Test UI visibility, direct protected URLs, and API permissions where applicable.

### Q10. Why should authentication state not be shared as mutable state?

Shared mutable state can cause tests to interfere with each other and create parallel-execution failures.

---

## 78. Hands-On Exercises

### Exercise 1 — Login State

Create a setup project that logs in and saves:

```text
.auth/user.json
```

### Exercise 2 — Reuse State

Configure browser tests to reuse the authentication state.

### Exercise 3 — Logout

Verify that logout removes access to protected pages.

### Exercise 4 — Admin State

Create:

```text
.auth/admin.json
```

### Exercise 5 — Employee State

Create:

```text
.auth/employee.json
```

### Exercise 6 — Authorization

Verify that the employee cannot access the admin area.

### Exercise 7 — Direct URL

Open the admin URL directly as an employee.

### Exercise 8 — Multi-User Workflow

Use separate contexts for:

```text
Employee
Manager
```

and automate an approval workflow.

### Exercise 9 — API Authentication

Authenticate through an API and use the resulting session/token where supported by the application.

### Exercise 10 — Enterprise Auth Fixture

Create reusable authentication fixtures for:

```text
adminPage
managerPage
employeePage
```

---

## 79. Enterprise Authentication Challenge

Build an authentication framework supporting:

```text
Admin
Manager
Employee
Viewer
```

Implement:

```text
Login
Logout
Storage State
Cookies
Local Storage
Session Storage
JWT
Refresh Token
OAuth2/SSO strategy
Multi-Role Authentication
Authorization
API Authorization
Session Expiration
```

Requirements:

1. Create an authentication setup project.
2. Generate role-specific authentication states.
3. Keep credentials in environment/CI secrets.
4. Exclude authentication files from Git.
5. Create reusable auth fixtures.
6. Support multiple browser contexts.
7. Test direct protected URLs.
8. Test role-based UI access.
9. Test API authorization.
10. Test logout.
11. Test invalid credentials.
12. Test expired sessions.
13. Support API + UI authentication workflows.
14. Keep authentication tests independent.
15. Make the framework parallel-safe and CI-friendly.

---

## 80. Final Enterprise Architecture

```text
                         Authentication Layer
                                  │
                 ┌────────────────┼────────────────┐
                 │                │                │
              UI Login         API Login        SSO/OAuth
                 │                │                │
                 └────────────────┼────────────────┘
                                  │
                          Authentication State
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
         Admin State          Manager State       Employee State
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
                           Browser Contexts
                                  │
                         ┌────────┴────────┐
                         │                 │
                       Pages          API Requests
                         │                 │
                         └────────┬────────┘
                                  │
                         Authorization Tests
                                  │
                           Business Workflows
```

This architecture separates authentication setup, session state, role management, API access, and authorization validation while keeping the framework scalable.

---

## 81. Final Checklist

```text
[ ] Login UI tested
[ ] API authentication tested
[ ] Storage state implemented
[ ] Cookies understood
[ ] Local storage understood
[ ] Session storage understood
[ ] JWT scenarios covered
[ ] Refresh token behavior covered
[ ] OAuth2/SSO strategy defined
[ ] Logout tested
[ ] Invalid credentials tested
[ ] Session expiry tested
[ ] Admin state created
[ ] Manager state created
[ ] Employee state created
[ ] Authorization tested
[ ] Direct URL access tested
[ ] API authorization tested
[ ] Multi-user contexts supported
[ ] Auth files protected
[ ] Secrets protected
[ ] Parallel execution supported
[ ] CI authentication configured
```

---

## 82. Key Takeaways

- Authentication and authorization are different concerns.
- Playwright `storageState` is essential for scalable authenticated testing.
- Login once and reuse authentication state when appropriate.
- Keep authentication state files secure.
- Use separate states for different roles.
- Use isolated browser contexts for multi-user workflows.
- Test login, logout, session expiration, and authorization.
- Test direct protected URLs, not only visible menus.
- Use API authentication for fast setup where appropriate.
- JWT and OAuth2 require testing both successful and failure scenarios.
- Local storage, session storage, and cookies behave differently.
- Enterprise frameworks should combine authentication tests with reusable role fixtures.
- Never hard-code credentials, tokens, or production session data.
- Authentication architecture should support parallel execution and CI.

---

## 83. Next Step

**Step 94 — Mastering Playwright Test Fixtures & Advanced Dependency Injection: Built-in Fixtures, Custom Fixtures, Worker Fixtures, Test Fixtures, Fixture Composition, Scope & Enterprise Framework Design**

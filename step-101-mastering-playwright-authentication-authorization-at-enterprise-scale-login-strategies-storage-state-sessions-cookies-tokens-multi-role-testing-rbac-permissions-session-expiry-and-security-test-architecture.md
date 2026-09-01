# Playwright with TypeScript Mastery Bootcamp

# Step 101 — Mastering Playwright Authentication & Authorization at Enterprise Scale

## 1. Introduction

Authentication and authorization are critical parts of enterprise Playwright automation.

A mature framework should test not only:

- Can the user log in?
- Is the session created correctly?
- Can the user access protected pages?
- Can the user access protected APIs?
- Does each role have the correct permissions?
- What happens when the session expires?
- What happens after logout?
- Can one user access another user's resources?
- Can one tenant access another tenant's data?

The security model is:

```text
Authentication
      ↓
Identity
      ↓
Session
      ↓
Authorization
      ↓
Permissions
      ↓
Application Behavior
```

---

## 2. Authentication vs Authorization

### Authentication

Authentication answers:

```text
Who are you?
```

Common mechanisms include:

```text
Username + Password
SSO
OAuth
OIDC
MFA
Session Cookies
Access Tokens
Refresh Tokens
```

### Authorization

Authorization answers:

```text
What are you allowed to do?
```

Examples:

```text
Admin
Manager
Employee
Auditor
Read
Write
Delete
Approve
Export
```

Authentication and authorization must be tested separately.

---

## 3. Typical Authentication Flow

```text
User
 ↓
Login Page
 ↓
Credentials
 ↓
Authentication API
 ↓
Identity Provider
 ↓
Session / Token
 ↓
Application
```

A Playwright suite should verify the important parts of this flow.

---

## 4. Typical Authorization Flow

```text
Authenticated User
       ↓
Role / Permission
       ↓
Authorization Check
       ↓
Allowed / Denied
       ↓
UI + API Behavior
```

Being authenticated does not mean the user can access every feature.

---

## 5. Why Repeated Login Is Expensive

If every test performs:

```text
Open Login
 ↓
Fill Username
 ↓
Fill Password
 ↓
Click Login
 ↓
Wait Dashboard
```

the suite becomes:

```text
Slow
Repetitive
Harder to maintain
More dependent on login stability
```

Playwright's storage state can reduce this cost.

---

## 6. Storage State

Playwright can save authentication-related browser state.

Example:

```ts
await page.context()
  .storageState({
    path:
      '.auth/user.json'
  });
```

This state can include cookies and local-storage data needed by the application.

Treat the file as sensitive.

---

## 7. Reusing Storage State

A project can reuse saved state:

```ts
use: {
  storageState:
    '.auth/user.json'
}
```

Architecture:

```text
Authentication Setup
        ↓
Saved State
        ↓
Dependent Tests
```

This allows tests to begin in an authenticated state.

---

## 8. Authentication Setup Project

A common project structure is:

```text
tests/
├── auth/
│   └── user.setup.ts
├── login/
├── dashboard/
├── admin/
└── reports/
```

The setup project creates authentication state before dependent tests run.

---

## 9. Example Authentication Setup

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
      '/web/index.php/auth/login'
    );

    await page
      .getByPlaceholder(
        'Username'
      )
      .fill(
        process.env.TEST_USERNAME!
      );

    await page
      .getByPlaceholder(
        'Password'
      )
      .fill(
        process.env.TEST_PASSWORD!
      );

    await page
      .getByRole(
        'button',
        {
          name: 'Login'
        }
      )
      .click();

    await expect(page)
      .toHaveURL(
        /dashboard/
      );

    await page.context()
      .storageState({
        path: authFile
      });
  }
);
```

---

## 10. Never Hard-Code Credentials

Avoid:

```ts
.fill('Admin')
.fill('admin123')
```

in production test code.

Prefer:

```ts
process.env.TEST_USERNAME
process.env.TEST_PASSWORD
```

and secure CI secret management.

---

## 11. Protect Authentication State

An authentication state file may contain:

```text
Cookies
Tokens
Local storage
Session information
```

Never:

```text
Commit it to Git
Print it in logs
Upload it publicly
Share it unnecessarily
```

Add the authentication directory to `.gitignore`.

Example:

```gitignore
.auth/
playwright/.auth/
```

---

## 12. Multiple Roles

Enterprise applications often contain:

```text
Admin
Manager
Employee
Auditor
Finance
HR
ReadOnly
```

Each role may require different authentication state.

Example:

```text
.auth/
├── admin.json
├── manager.json
├── employee.json
└── auditor.json
```

---

## 13. Role-Based Projects

A Playwright configuration can define projects such as:

```text
setup
admin
manager
employee
```

Conceptually:

```ts
{
  name:
    'admin',
  use: {
    storageState:
      '.auth/admin.json'
  }
}
```

The same approach can be used for manager and employee states.

---

## 14. Role Permission Matrix

Example:

| Feature | Admin | Manager | Employee |
|---|---:|---:|---:|
| Dashboard | Yes | Yes | Yes |
| User Management | Yes | Limited | No |
| Create Expense | Yes | Yes | Yes |
| Approve Expense | Yes | Yes | No |
| Delete User | Yes | No | No |
| Reports | Yes | Yes | Limited |

The application's security policy should be the source of truth.

---

## 15. RBAC

RBAC means:

```text
Role-Based Access Control
```

Example:

```text
Admin
 ├── USER_READ
 ├── USER_CREATE
 ├── USER_UPDATE
 └── USER_DELETE
```

Testing should verify both allowed and denied actions.

---

## 16. Permission-Based Authorization

Some applications use direct permissions:

```text
USER_READ
USER_CREATE
USER_UPDATE
USER_DELETE
EXPENSE_APPROVE
REPORT_EXPORT
```

A role can contain multiple permissions.

Tests should validate the effective permissions.

---

## 17. UI Authorization

Example:

```ts
await expect(
  page.getByRole(
    'link',
    {
      name: 'Admin'
    }
  )
).not.toBeVisible();
```

This validates UI visibility.

However:

```text
Hidden UI ≠ Complete Security
```

Server-side authorization must also be tested.

---

## 18. API Authorization

For a restricted role:

```text
GET /api/admin/users
```

may be expected to return:

```text
403 Forbidden
```

or another documented authorization response.

This verifies the server-side security boundary.

---

## 19. UI + API Authorization

A strong authorization test strategy is:

```text
Role
 ↓
Login
 ↓
UI Access
 ↓
API Access
 ↓
Business Action
```

This gives stronger coverage than testing only menus and buttons.

---

## 20. Cookies

Cookie-based authentication commonly works as:

```text
Login
 ↓
Server
 ↓
Set-Cookie
 ↓
Browser
 ↓
Authenticated Requests
```

Storage state can preserve relevant cookies.

If cookie behavior itself is a requirement, inspect security attributes rather than asserting actual token values.

---

## 21. Cookie Security Attributes

Important attributes include:

```text
HttpOnly
Secure
SameSite
Domain
Path
Expiration
```

Validate them only when required by the application's security contract.

---

## 22. HttpOnly

An HttpOnly cookie is not normally readable by client-side JavaScript.

Security tests can verify this property when required.

Do not expose cookie values in test logs.

---

## 23. Secure

The `Secure` attribute limits cookie transmission to secure connections.

For HTTPS environments, validate the expected configuration where appropriate.

---

## 24. SameSite

Common values:

```text
Strict
Lax
None
```

The correct value depends on the application's architecture and cross-site requirements.

---

## 25. Local Storage Authentication

Some SPAs store authentication data in local storage.

Example inspection:

```ts
await page.evaluate(() =>
  localStorage.getItem(
    'accessToken'
  )
);
```

Do not print real tokens.

Prefer behavior-based assertions.

---

## 26. Access Tokens

Token-based applications may use:

```text
Access Token
Refresh Token
```

Typical flow:

```text
Login
 ↓
Access Token
 ↓
API Requests
 ↓
Token Expires
 ↓
Refresh Token
 ↓
New Access Token
```

Test the behavior rather than exact token contents.

---

## 27. Refresh Token Testing

Validate:

```text
Access token expires
 ↓
Refresh endpoint
 ↓
New access token
 ↓
Authenticated request continues
```

The exact behavior depends on the client and server implementation.

---

## 28. Expired Tokens

A controlled test can verify:

```text
Expired Access Token
 ↓
Protected Request
 ↓
401
 ↓
Refresh
 ↓
Retry
```

If refresh also fails:

```text
Session Ends
 ↓
Login Required
```

---

## 29. Session Expiry

Sessions can expire due to:

```text
Idle timeout
Absolute timeout
Token expiration
Server-side invalidation
Password change
Administrative logout
```

Tests should validate the documented behavior.

---

## 30. Logout

A complete logout test should verify:

```text
Logout
 ↓
Session removed / invalidated
 ↓
Protected resource unavailable
 ↓
Login required
```

Example:

```ts
await page
  .getByRole(
    'button',
    {
      name: 'Logout'
    }
  )
  .click();

await expect(page)
  .toHaveURL(
    /login/
  );
```

Then verify protected access is no longer available.

---

## 31. Browser Back After Logout

A security-sensitive scenario:

```text
Login
 ↓
Dashboard
 ↓
Logout
 ↓
Browser Back
```

The application should not allow unauthorized access to protected data.

Validate this according to the application's caching and session policy.

---

## 32. Browser Context Isolation

Different users should use separate browser contexts when simultaneous identities are required.

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
```

Each context has independent browser state.

---

## 33. Multi-User Workflows

Enterprise workflows may require multiple users.

Example:

```text
Employee
 ↓
Submit Expense
 ↓
Pending
 ↓
Manager
 ↓
Approve Expense
```

Use separate contexts.

---

## 34. Multi-User Example

```ts
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
  await employeeContext.newPage();

const managerPage =
  await managerContext.newPage();
```

Avoid switching identities by manually changing tokens in one page.

---

## 35. Tenant Authentication

Multi-tenant systems may contain:

```text
Tenant A
Tenant B
Tenant C
```

Users should normally be restricted to their authorized tenant.

---

## 36. Tenant Isolation

Example:

```text
Tenant A User
 ↓
Tenant A Expense
```

must not access:

```text
Tenant B Expense
```

Test this through the appropriate UI and API boundaries.

---

## 37. Horizontal Authorization

Horizontal authorization tests users with similar privilege levels.

Example:

```text
Employee A
 ↓
Employee B's Expense
 ↓
Denied
```

This helps detect cross-user access problems.

---

## 38. Vertical Authorization

Vertical authorization tests privilege boundaries.

Example:

```text
Employee
 ↓
Admin Endpoint
 ↓
Denied
```

This validates role escalation boundaries.

---

## 39. Direct URL Authorization

A restricted menu item might be hidden, but the user could manually navigate to its URL.

Test:

```text
Login
 ↓
Open Restricted URL Directly
 ↓
Access Denied
```

This catches missing route protection.

---

## 40. Direct API Authorization

Similarly, test the server directly:

```text
Restricted Role
 ↓
Restricted API
 ↓
Expected Denial
```

Do not rely only on frontend restrictions.

---

## 41. IDOR Testing

IDOR means insecure direct object reference.

A basic test concept:

```text
User A owns Record A
 ↓
User A requests Record B
 ↓
Access Denied
```

Important resources include:

```text
Users
Expenses
Documents
Reports
Invoices
Orders
```

Use synthetic records.

---

## 42. Authentication Test Categories

Create separate coverage for:

```text
Valid Login
Invalid Login
Logout
Session Expiry
Token Refresh
Role Authorization
Permission Authorization
Tenant Isolation
API Authorization
Direct URL Authorization
Multi-User Workflows
```

---

## 43. Invalid Credentials

Example:

```ts
await loginPage.login(
  'wrong-user',
  'wrong-password'
);

await expect(
  loginPage.errorMessage
).toBeVisible();
```

Validate the application's documented error behavior.

---

## 44. Account Lockout

If supported:

```text
Repeated controlled failures
 ↓
Account Locked
 ↓
Further Login Denied
```

Validate the lockout behavior in a dedicated test environment.

Do not perform uncontrolled attempts against shared systems.

---

## 45. Password Policy

Test safe non-production accounts for:

```text
Minimum length
Complexity
Invalid format
Password reuse
Expiration
```

Only test policies that are part of the application's requirements.

---

## 46. MFA

MFA automation may require:

```text
Dedicated test MFA environment
Controlled OTP provider
Mocked identity provider
Non-production test bypass
```

Do not rely on personal MFA devices for normal automated testing.

---

## 47. SSO

Enterprise applications may use:

```text
SAML
OAuth 2.0
OIDC
Corporate Identity Provider
```

Separate application authentication tests from the external identity provider itself where practical.

---

## 48. SSO Test Strategy

Validate:

```text
Redirect
Authentication
Callback
Identity mapping
Application session
Logout
```

Use a controlled test identity provider.

---

## 49. Role Mapping

External identity information may map to application roles.

Example:

```text
IdP Group
 ↓
Application Role
 ↓
Permissions
```

Test that the expected mapping occurs.

---

## 50. Permission Boundary Testing

For each sensitive operation:

```text
Admin → Allowed
Manager → Allowed / Denied
Employee → Denied
```

Create an explicit matrix.

---

## 51. Authentication Setup Architecture

A mature project may use:

```text
tests/
├── auth/
│   ├── admin.setup.ts
│   ├── manager.setup.ts
│   └── employee.setup.ts
├── admin/
├── manager/
└── employee/
```

Each setup creates the corresponding authentication state.

---

## 52. Role-Specific Fixtures

Possible fixtures:

```text
adminPage
managerPage
employeePage
```

Each fixture can create a pre-authenticated context.

This makes test intent explicit.

---

## 53. Example Authenticated Fixture

```ts
export const test =
  base.extend({
    adminPage:
      async ({
        browser
      }, use) => {

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
  });
```

Use this when a dedicated authenticated context is useful.

---

## 54. One Account vs Multiple Accounts

### One Account

Advantages:

```text
Simple
Fast
Less setup
```

Disadvantages:

```text
Shared state
Parallel conflicts
Limited role coverage
```

### Multiple Accounts

Advantages:

```text
Isolation
Role coverage
Parallel safety
```

Disadvantages:

```text
More setup
Credential management
Maintenance
```

---

## 55. Account Pool

Large suites can maintain dedicated synthetic accounts:

```text
Admin-01
Admin-02
Manager-01
Manager-02
Employee-01
Employee-02
```

Workers can receive separate accounts when mutable state requires isolation.

---

## 56. Parallel Authentication

Example:

```text
Worker 0 → Employee-01
Worker 1 → Employee-02
Worker 2 → Employee-03
```

This reduces account-state collisions.

Use isolated accounts only where shared account mutation would create conflicts.

---

## 57. Stale Authentication State

Authentication state can become invalid because of:

```text
Expired session
Changed password
Environment reset
Server invalidation
Token expiration
```

Unexpected 401 errors should trigger an authentication-state investigation.

---

## 58. Refreshing Authentication State

Create fresh state when:

```text
Credentials change
Environment is reset
Session expires
Authentication behavior changes
```

CI should generally generate authentication state as part of the job instead of relying on old shared artifacts.

---

## 59. Authentication Debugging

When login or state reuse fails, inspect:

```text
URL
Redirects
Cookies
Storage state
Network responses
Console errors
Authentication provider
Environment
```

Never log token values.

---

## 60. Trace-Based Debugging

Playwright traces can help identify:

```text
Login redirects
Failed requests
Unexpected pages
Missing state
Navigation problems
```

Use controlled artifacts and avoid exposing sensitive authentication information.

---

## 61. Authentication Assertions

After login:

```ts
await expect(page)
  .toHaveURL(
    /dashboard/
  );

await expect(
  page.getByRole(
    'heading',
    {
      name:
        'Dashboard'
    }
  )
).toBeVisible();
```

Validate meaningful authenticated state.

---

## 62. API Authentication Assertions

Example:

```ts
const response =
  await request.get(
    '/api/profile'
  );

expect(
  response.status()
).toBe(200);

const body =
  await response.json();

expect(
  body.username
).toBe(
  expectedUsername
);
```

Validate identity behavior rather than token contents.

---

## 63. Authorization API Assertion

Example:

```ts
const response =
  await employeeRequest.delete(
    '/api/users/123'
  );

expect(
  response.status()
).toBe(403);
```

The expected status must match the application's API contract.

---

## 64. Session Expiry API Test

Conceptually:

```text
Authenticated Request
 ↓
Invalidate Session
 ↓
Protected Request
 ↓
401
```

Validate the expected behavior.

---

## 65. Security Headers

Depending on the application's security requirements, validate headers such as:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
```

Do not create assertions for headers that are irrelevant to the application contract.

---

## 66. CSRF

Cookie-based authentication may require CSRF protection.

A controlled non-production test can verify that a state-changing request without the required CSRF mechanism is rejected.

Coordinate security testing with the application's threat model.

---

## 67. Open Redirect

Authentication flows may contain:

```text
returnUrl
redirect
next
continue
```

Test that untrusted redirect destinations cannot create unsafe redirects when protection is required.

---

## 68. Authentication Rate Limiting

If rate limiting exists:

```text
Controlled Attempts
 ↓
Rate Limit
 ↓
Expected Response
```

Run this only in a dedicated environment so shared systems are not disrupted.

---

## 69. Authentication Architecture

A large Playwright framework can use:

```text
auth/
├── setup/
├── states/
├── users/
├── roleMatrix/
└── helpers/
```

and:

```text
fixtures/
├── authenticatedTest.ts
├── adminTest.ts
├── managerTest.ts
└── employeeTest.ts
```

---

## 70. Layered Security Test Strategy

A complete strategy can contain:

```text
Layer 1 → Login UI
Layer 2 → Authentication API
Layer 3 → Session State
Layer 4 → Role Mapping
Layer 5 → UI Authorization
Layer 6 → API Authorization
Layer 7 → Business Authorization
Layer 8 → Session Expiry
Layer 9 → Logout
Layer 10 → Tenant Isolation
```

---

## 71. UI, API, and Security Distribution

Do not test every security rule only through the UI.

A balanced approach is:

```text
UI Tests
 ↓
Critical user journeys

API Tests
 ↓
Authorization matrix

Integration Tests
 ↓
Identity provider and role mapping

Security Tests
 ↓
Boundary and abuse scenarios
```

This keeps execution efficient.

---

## 72. Authentication Anti-Patterns

Avoid:

```text
Hard-coded passwords
Committed storage state
Shared mutable accounts
Tokens in logs
One account for every role
UI login in every test
Stale authentication state
Testing only menu visibility
Ignoring API authorization
Testing destructive security scenarios against production
```

---

## 73. Authorization Anti-Patterns

Do not assume:

```text
Hidden button = secure
Hidden menu = secure
Unavailable route = secure
```

Sensitive authorization must be enforced server-side.

Test the server independently.

---

## 74. Enterprise Best Practices

1. Keep credentials outside source control.
2. Treat storage state as sensitive.
3. Use dedicated synthetic accounts.
4. Reuse authentication state where safe.
5. Create separate states for important roles.
6. Use separate contexts for simultaneous users.
7. Test UI and API authorization.
8. Maintain a role-permission matrix.
9. Test horizontal and vertical boundaries.
10. Test session expiry.
11. Test logout.
12. Test tenant isolation.
13. Never log secrets.
14. Keep authentication setup separate from business tests.
15. Refresh stale state.
16. Design authentication for parallel execution.
17. Use controlled MFA/SSO environments.
18. Run security tests in non-production environments.
19. Keep tests deterministic.
20. Document the authentication architecture.

---

## 75. Hands-On Exercise 1 — Login State

Create:

```text
tests/auth/user.setup.ts
```

and save:

```text
.auth/user.json
```

Then configure another project to reuse the state.

---

## 76. Hands-On Exercise 2 — Multiple Roles

Create:

```text
admin.json
manager.json
employee.json
```

and configure role-specific projects.

---

## 77. Hands-On Exercise 3 — RBAC Matrix

Create a matrix for:

```text
Admin
Manager
Employee
```

with:

```text
Read
Create
Update
Delete
Approve
Export
```

---

## 78. Hands-On Exercise 4 — API Authorization

Use separate authenticated contexts to test:

```text
Allowed API
Denied API
```

---

## 79. Hands-On Exercise 5 — Multi-User Workflow

Implement:

```text
Employee
 ↓
Submit Expense
 ↓
Manager
 ↓
Approve Expense
```

using separate browser contexts.

---

## 80. Hands-On Exercise 6 — Logout

Verify:

```text
Login
 ↓
Dashboard
 ↓
Logout
 ↓
Protected Page
 ↓
Access Denied / Login Required
```

---

## 81. Hands-On Exercise 7 — Session Expiry

In a controlled environment, simulate or configure session expiration and verify the expected redirect or authentication response.

---

## 82. Hands-On Exercise 8 — Tenant Isolation

Create two synthetic tenants:

```text
Tenant A
Tenant B
```

Verify that Tenant A cannot access Tenant B resources.

---

## 83. Enterprise Authentication Challenge

Build a complete authentication and authorization framework supporting:

```text
Login
Storage State
Multiple Roles
RBAC
Permissions
Cookies
Tokens
Session Expiry
Logout
Multi-User Workflows
Tenant Isolation
API Authorization
UI Authorization
```

Requirements:

1. Create authentication setup projects.
2. Store authentication state securely.
3. Create admin state.
4. Create manager state.
5. Create employee state.
6. Configure role-specific projects.
7. Create a role-permission matrix.
8. Validate protected routes.
9. Validate protected APIs.
10. Test unauthorized actions.
11. Test logout.
12. Test session expiration.
13. Test refresh behavior where applicable.
14. Test tenant isolation.
15. Use separate contexts for simultaneous users.
16. Protect credentials and tokens.
17. Design for parallel execution.
18. Add useful authentication diagnostics.
19. Avoid production authentication testing.
20. Document the security test architecture.

---

## 84. Final Enterprise Architecture

```text
                         Playwright
                             │
                       Authentication
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
       Login Flow        Storage State      API Auth
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                       Identity / Role
                             │
                  ┌──────────┼──────────┐
                  │          │          │
                Admin      Manager    Employee
                  │          │          │
                  └──────────┼──────────┘
                             │
                         Permissions
                             │
              ┌──────────────┼──────────────┐
              │              │              │
             UI             API          Business
              │              │              │
              └──────────────┼──────────────┘
                             │
                      Security Validation
```

---

## 85. Final Checklist

```text
[ ] Authentication understood
[ ] Authorization understood
[ ] Authentication vs authorization understood
[ ] Storage state understood
[ ] Authentication setup project understood
[ ] Authentication state security understood
[ ] Multiple roles understood
[ ] RBAC understood
[ ] Permission-based authorization understood
[ ] Role matrix created
[ ] Cookie authentication understood
[ ] Cookie security attributes understood
[ ] Local storage authentication understood
[ ] Access tokens understood
[ ] Refresh tokens understood
[ ] Session expiry understood
[ ] Logout tested
[ ] Multi-user contexts understood
[ ] Tenant isolation understood
[ ] Horizontal authorization understood
[ ] Vertical authorization understood
[ ] Direct URL authorization understood
[ ] API authorization understood
[ ] IDOR concept understood
[ ] SSO strategy understood
[ ] OAuth/OIDC concepts understood
[ ] MFA test strategy understood
[ ] Security headers understood
[ ] CSRF concept understood
[ ] Authentication debugging understood
[ ] Role-specific fixtures understood
[ ] Parallel authentication strategy understood
[ ] Authentication anti-patterns understood
[ ] Enterprise authentication architecture understood
```

---

## 86. Key Takeaways

- Authentication answers **who the user is**.
- Authorization answers **what the user is allowed to do**.
- Storage state can eliminate repeated login work.
- Authentication state must be treated as sensitive.
- Different roles should have isolated authentication states when required.
- Separate browser contexts are useful for multi-user workflows.
- RBAC testing should verify both allowed and denied operations.
- UI restrictions are not sufficient proof of server-side security.
- Sensitive APIs should be tested directly with different roles.
- Session expiry and logout are important security workflows.
- Multi-tenant systems require explicit tenant-isolation tests.
- Synthetic test accounts are safer than real accounts.
- Credentials, tokens, and authentication artifacts should never appear in source control or logs.
- Authentication setup should remain separate from business behavior tests.
- Enterprise security testing works best when UI, API, integration, and security layers complement each other.
- A strong Playwright authentication architecture improves speed, reliability, security coverage, and maintainability.

---

## 87. Next Step

**Step 102 — Mastering Playwright API Testing & Network Automation: APIRequestContext, REST Methods, Authentication, Headers, Query Parameters, Request Bodies, Response Validation, API Fixtures, CRUD Workflows, API/UI Integration & Enterprise API Test Architecture**

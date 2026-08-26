# Playwright with TypeScript Mastery Bootcamp

# Step 99 — Mastering Playwright Advanced Assertions & Validation: Auto-Retrying Assertions, Soft Assertions, Polling, Custom Expectations, API Assertions, UI State Validation, Data Validation & Enterprise Assertion Strategy

## 1. Introduction

Assertions are the verification layer of Playwright automation.

A test is not complete when it performs an action.

It is complete when it verifies that the expected outcome occurred.

The basic flow is:

```text
Arrange
  ↓
Act
  ↓
Assert
```

Example:

```ts
await page
  .getByRole(
    'button',
    {
      name: 'Login'
    }
  )
  .click();

await expect(
  page.getByRole(
    'heading',
    {
      name: 'Dashboard'
    }
  )
).toBeVisible();
```

Playwright provides powerful assertion capabilities including:

```text
Auto-Retrying Assertions
Soft Assertions
Locator Assertions
Page Assertions
API Assertions
Polling
Custom Matchers
State Validation
URL Validation
Text Validation
Attribute Validation
Screenshot Assertions
Enterprise Validation Strategies
```

---

## 2. Why Assertions Matter

Without assertions:

```ts
await loginPage.login(
  username,
  password
);
```

The test only proves that an action was attempted.

With an assertion:

```ts
await expect(
  dashboardPage.heading
).toBeVisible();
```

the test verifies the expected application state.

---

## 3. Assertion Types

Playwright assertions can be grouped into:

```text
Locator Assertions
Page Assertions
APIResponse Assertions
Generic Assertions
Soft Assertions
Polling Assertions
Screenshot Assertions
Custom Expectations
```

Each type serves a different validation purpose.

---

## 4. `expect()`

Playwright's main assertion API is:

```ts
import {
  test,
  expect
} from '@playwright/test';
```

Example:

```ts
await expect(
  page.getByText(
    'Welcome'
  )
).toBeVisible();
```

---

## 5. Auto-Retrying Assertions

One of Playwright's most important features is automatic retrying.

Example:

```ts
await expect(
  page.getByText(
    'Success'
  )
).toBeVisible();
```

If the element is not immediately visible, Playwright waits and retries until the assertion timeout is reached.

This is better than:

```ts
await page.waitForTimeout(
  3000
);

expect(
  await locator.isVisible()
).toBe(true);
```

---

## 6. Why Auto-Retrying Assertions Matter

Modern applications are asynchronous.

For example:

```text
Click Save
   ↓
API Request
   ↓
Backend Processing
   ↓
UI Update
   ↓
Success Message
```

The assertion should wait for the actual expected state.

---

## 7. Assertion Timeout

Playwright assertions have their own timeout configuration.

Example:

```ts
await expect(
  locator
).toBeVisible({
  timeout: 10000
});
```

Use longer timeouts only when the application genuinely needs them.

Do not increase timeouts to hide synchronization problems.

---

## 8. Global Expect Timeout

Configuration can define a default expectation timeout.

Example:

```ts
export default defineConfig({
  expect: {
    timeout:
      5000
  }
});
```

This applies to Playwright expectations unless overridden.

---

## 9. `toBeVisible()`

Example:

```ts
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

Useful for:

```text
Pages
Dialogs
Messages
Buttons
Forms
Sections
```

---

## 10. `not.toBeVisible()`

Example:

```ts
await expect(
  page.getByText(
    'Loading...'
  )
).not.toBeVisible();
```

Useful when a loading indicator should disappear.

---

## 11. `toBeHidden()`

Example:

```ts
await expect(
  page.getByTestId(
    'loading-spinner'
  )
).toBeHidden();
```

This verifies the element is hidden.

---

## 12. `toBeAttached()`

Example:

```ts
await expect(
  locator
).toBeAttached();
```

Useful when the test needs to verify DOM attachment.

Remember that attached does not necessarily mean visible.

---

## 13. `toBeEnabled()`

Example:

```ts
await expect(
  page.getByRole(
    'button',
    {
      name:
        'Submit'
    }
  )
).toBeEnabled();
```

Useful for form and workflow validation.

---

## 14. `toBeDisabled()`

Example:

```ts
await expect(
  page.getByRole(
    'button',
    {
      name:
        'Submit'
    }
  )
).toBeDisabled();
```

Useful for validating UI restrictions.

---

## 15. `toBeEditable()`

Example:

```ts
await expect(
  page.getByLabel(
    'Username'
  )
).toBeEditable();
```

Useful when validating form state.

---

## 16. `toBeEmpty()`

Example:

```ts
await expect(
  page.getByLabel(
    'Search'
  )
).toBeEmpty();
```

Useful for input state validation.

---

## 17. `toBeFocused()`

Example:

```ts
await expect(
  page.getByLabel(
    'Search'
  )
).toBeFocused();
```

Useful for keyboard and accessibility workflows.

---

## 18. `toBeChecked()`

Example:

```ts
await expect(
  page.getByRole(
    'checkbox',
    {
      name:
        'Remember me'
    }
  )
).toBeChecked();
```

---

## 19. `toBeUnchecked()`

Example:

```ts
await expect(
  page.getByRole(
    'checkbox',
    {
      name:
        'Remember me'
    }
  )
).not.toBeChecked();
```

---

## 20. `toHaveText()`

Example:

```ts
await expect(
  page.getByTestId(
    'status'
  )
).toHaveText(
  'Active'
);
```

This retries until the expected text appears.

---

## 21. `toContainText()`

Example:

```ts
await expect(
  page.getByTestId(
    'message'
  )
).toContainText(
  'success'
);
```

Useful when the element contains additional dynamic text.

---

## 22. Exact Text vs Partial Text

Exact:

```ts
await expect(
  locator
).toHaveText(
  'Active'
);
```

Partial:

```ts
await expect(
  locator
).toContainText(
  'Active'
);
```

Choose based on the requirement.

---

## 23. `toHaveValue()`

Example:

```ts
await expect(
  page.getByLabel(
    'Username'
  )
).toHaveValue(
  'Admin'
);
```

Useful for input validation.

---

## 24. `toHaveValues()`

For multi-select controls where appropriate:

```ts
await expect(
  locator
).toHaveValues([
  'admin',
  'user'
]);
```

Use according to the control type.

---

## 25. `toHaveAttribute()`

Example:

```ts
await expect(
  page.getByRole(
    'link',
    {
      name:
        'Dashboard'
    }
  )
).toHaveAttribute(
  'href',
  '/dashboard'
);
```

Useful when an attribute is part of the behavior being validated.

---

## 26. `toHaveClass()`

Example:

```ts
await expect(
  locator
).toHaveClass(
  /active/
);
```

Use class assertions carefully.

Prefer user-visible state when possible.

---

## 27. `toHaveCSS()`

Example:

```ts
await expect(
  locator
).toHaveCSS(
  'display',
  'block'
);
```

Use CSS assertions only when styling itself is part of the requirement.

---

## 28. `toHaveId()`

Example:

```ts
await expect(
  locator
).toHaveId(
  'username'
);
```

Useful when a stable DOM ID is part of an accessibility or integration contract.

---

## 29. `toHaveCount()`

Example:

```ts
await expect(
  page.getByRole(
    'row'
  )
).toHaveCount(5);
```

Excellent for validating:

```text
Tables
Lists
Cards
Search results
Notifications
```

---

## 30. `toHaveURL()`

Example:

```ts
await expect(
  page
).toHaveURL(
  /dashboard/
);
```

This is better than immediately reading the URL and comparing it manually.

---

## 31. Exact URL Assertion

Example:

```ts
await expect(
  page
).toHaveURL(
  'https://example.test/dashboard'
);
```

Use exact URLs when the complete URL is part of the requirement.

---

## 32. URL Regex

Example:

```ts
await expect(
  page
).toHaveURL(
  /\/users\/\d+$/
);
```

Useful for dynamic IDs.

---

## 33. Page Title

Example:

```ts
await expect(
  page
).toHaveTitle(
  'Dashboard'
);
```

Useful for browser title validation.

---

## 34. `toHaveScreenshot()`

Playwright supports visual assertions.

Example:

```ts
await expect(
  page
).toHaveScreenshot(
  'dashboard.png'
);
```

This compares the current rendering against a baseline.

---

## 35. Visual Regression

Visual testing can detect:

```text
Layout changes
Missing elements
Unexpected colors
Spacing changes
Font changes
Responsive issues
```

Use visual assertions selectively.

---

## 36. Screenshot Assertion Stability

Visual tests can be affected by:

```text
Browser version
Operating system
Fonts
Animations
Time-dependent content
Random data
Network images
```

Control these sources of nondeterminism.

---

## 37. Generic Assertions

Playwright also provides generic expectations.

Example:

```ts
expect(
  user.name
).toBe(
  'Sandeep'
);
```

These are useful for:

```text
Variables
API data
Computed values
Business rules
```

---

## 38. Generic `toEqual()`

Example:

```ts
expect(
  responseBody
).toEqual({
  status:
    'success'
});
```

Use when complete structural equality is required.

---

## 39. `toMatchObject()`

Example:

```ts
expect(
  responseBody
).toMatchObject({
  status:
    'success'
});
```

Useful when the response contains many fields but only some matter to the test.

---

## 40. `toContain()`

Example:

```ts
expect(
  roles
).toContain(
  'ADMIN'
);
```

Useful for arrays and strings.

---

## 41. `toHaveLength()`

Example:

```ts
expect(
  users
).toHaveLength(
  10
);
```

Useful for API data and business calculations.

---

## 42. `toBeTruthy()` and `toBeFalsy()`

Example:

```ts
expect(
  isAuthenticated
).toBeTruthy();
```

Use specific assertions where possible.

Instead of:

```ts
toBeTruthy()
```

prefer:

```ts
toBe(true)
```

when the exact boolean state is required.

---

## 43. `toBeNull()`

Example:

```ts
expect(
  deletedUser
).toBeNull();
```

Useful for API and service-layer validation.

---

## 44. `toBeDefined()`

Example:

```ts
expect(
  response.id
).toBeDefined();
```

Useful for generated IDs and response properties.

---

## 45. Soft Assertions

A normal assertion fails the test immediately.

Example:

```ts
await expect(
  page.getByText(
    'Name'
  )
).toBeVisible();

await expect(
  page.getByText(
    'Role'
  )
).toBeVisible();
```

If the first assertion fails, the second may not execute.

---

## 46. `expect.soft()`

Example:

```ts
await expect.soft(
  page.getByText(
    'Name'
  )
).toBeVisible();

await expect.soft(
  page.getByText(
    'Role'
  )
).toBeVisible();

await expect.soft(
  page.getByText(
    'Status'
  )
).toBeVisible();
```

The test can continue collecting failures.

---

## 47. When to Use Soft Assertions

Good use cases:

```text
Dashboard field validation
Multiple independent UI checks
Table column validation
Summary page verification
```

Avoid soft assertions when a failure makes subsequent actions meaningless.

---

## 48. Hard vs Soft Assertions

### Hard

```text
Failure
 ↓
Stop Test
```

### Soft

```text
Failure
 ↓
Record Failure
 ↓
Continue
 ↓
Report Multiple Failures
```

Choose based on dependency between validations.

---

## 49. Soft Assertion Strategy

Good:

```text
Verify independent dashboard widgets
```

Poor:

```text
Login failed
 ↓
Continue testing authenticated dashboard
```

Authentication failure should usually stop the workflow.

---

## 50. Soft Assertion Error Check

Advanced scenarios can inspect test information after soft assertions.

Use this to make decisions only when necessary.

Keep the test logic simple.

---

## 51. Polling

Some systems update asynchronously.

Example:

```text
Create Job
 ↓
Processing
 ↓
Completed
```

A simple immediate assertion may fail.

Polling allows repeated evaluation until the expected condition becomes true.

---

## 52. `expect.poll()`

Conceptual example:

```ts
await expect.poll(
  async () => {
    const response =
      await request.get(
        '/api/job/123'
      );

    return response
      .status();
  }
).toBe(200);
```

This is useful for asynchronous backend state.

---

## 53. Polling a Business Value

Example:

```ts
await expect.poll(
  async () => {
    const response =
      await request.get(
        '/api/job/123'
      );

    const body =
      await response.json();

    return body.status;
  }
).toBe(
  'COMPLETED'
);
```

This avoids arbitrary sleeps.

---

## 54. Polling Options

Polling can be configured for:

```text
Timeout
Intervals
Message
```

Conceptually:

```ts
await expect.poll(
  async () =>
    getStatus(),
  {
    timeout:
      30000,
    intervals: [
      1000,
      2000,
      5000
    ]
  }
).toBe(
  'COMPLETED'
);
```

Use intervals appropriate for the system.

---

## 55. Polling vs `waitForTimeout()`

Bad:

```ts
await page.waitForTimeout(
  10000
);

await expect(
  status
).toHaveText(
  'Completed'
);
```

Better:

```ts
await expect.poll(
  async () =>
    getStatus()
).toBe(
  'COMPLETED'
);
```

The test stops as soon as the condition succeeds.

---

## 56. Polling UI State

You can poll a function that reads application state.

Example concept:

```ts
await expect.poll(
  async () =>
    await page
      .getByTestId(
        'status'
      )
      .textContent()
).toContain(
  'Completed'
);
```

For normal UI synchronization, prefer locator assertions first.

Use polling when the state source is not directly represented by a suitable locator assertion.

---

## 57. Polling API State

Common enterprise scenario:

```text
POST /jobs
 ↓
Job ID
 ↓
GET /jobs/{id}
 ↓
Status
```

Use polling:

```text
QUEUED
 ↓
RUNNING
 ↓
COMPLETED
```

This is more reliable than fixed sleeps.

---

## 58. Polling Database State

If the test environment permits database access:

```text
Create record
 ↓
Background process
 ↓
Database state changes
```

A controlled polling helper can wait for the expected state.

Keep database polling outside UI tests unless it directly supports the validation strategy.

---

## 59. API Response Assertions

Example:

```ts
const response =
  await request.get(
    '/api/users'
  );

expect(
  response.ok()
).toBe(true);

expect(
  response.status()
).toBe(200);
```

This validates backend behavior directly.

---

## 60. API Body Validation

Example:

```ts
const body =
  await response.json();

expect(
  body.users
).toHaveLength(3);
```

---

## 61. API Schema-Like Validation

For complex APIs, validate important fields:

```ts
expect(
  body
).toMatchObject({
  status:
    'success',
  data: {
    id:
      expect.any(Number)
  }
});
```

This provides flexible structural validation.

---

## 62. API Headers

Example:

```ts
expect(
  response.headers()[
    'content-type'
  ]
).toContain(
  'application/json'
);
```

Only validate headers that matter to the contract.

---

## 63. Status Code Assertions

Examples:

```text
200 → Successful request
201 → Resource created
204 → No content
400 → Invalid request
401 → Authentication failure
403 → Authorization failure
404 → Resource missing
409 → Conflict
429 → Rate limit
500 → Server error
```

Tests should assert the status appropriate to the scenario.

---

## 64. Positive API Test

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {
        name:
          'Automation User'
      }
    }
  );

expect(
  response.status()
).toBe(201);
```

---

## 65. Negative API Test

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {}
    }
  );

expect(
  response.status()
).toBe(400);
```

Negative tests are essential for API validation.

---

## 66. UI + API Assertion

Example:

```text
UI creates user
 ↓
API confirms creation
 ↓
UI shows success
```

Assertions:

```ts
await expect(
  successMessage
).toBeVisible();

expect(
  response.status()
).toBe(201);
```

This provides stronger evidence than only checking the UI.

---

## 67. Data Validation

Suppose a form contains:

```text
Amount
Tax
Total
```

Instead of only checking text visually, validate the business rule.

Example:

```ts
expect(
  total
).toBe(
  amount + tax
);
```

This is a business assertion.

---

## 68. Business Rule Assertions

Example:

```ts
expect(
  expense.total
).toBe(
  expense.amount +
  expense.tax
);
```

This catches logic errors that visual checks may miss.

---

## 69. Validation of Calculated Fields

Example:

```ts
const expectedTotal =
  amount + tax;

await expect(
  page.getByLabel(
    'Total'
  )
).toHaveValue(
  String(
    expectedTotal
  )
);
```

---

## 70. Data Normalization

Before comparing data, normalize only when the representation is intentionally different.

Example:

```ts
const actual =
  actualValue.trim();

const expected =
  expectedValue.trim();

expect(
  actual
).toBe(
  expected
);
```

Avoid excessive normalization that can hide defects.

---

## 71. Date Assertions

Dates can vary by:

```text
Timezone
Locale
Format
Clock
```

Prefer controlled date inputs and deterministic formatting.

Example:

```ts
expect(
  actualDate
).toBe(
  expectedDate
);
```

Normalize only when the requirement allows it.

---

## 72. Time-Dependent Assertions

Avoid:

```ts
expect(
  page.getByText(
    new Date().toString()
  )
).toBeVisible();
```

This can be unstable.

Use a controlled clock or assert stable date components when appropriate.

---

## 73. Currency Assertions

Avoid comparing formatted strings when the business requirement is numeric.

Instead of:

```text
₹1,000.00
```

parse or retrieve the numeric value appropriately and validate:

```ts
expect(
  amount
).toBe(1000);
```

Then separately test formatting if formatting itself matters.

---

## 74. Table Validation

Example:

```ts
const rows =
  page.getByRole(
    'row'
  );

await expect(
  rows
).toHaveCount(6);
```

Then validate important rows:

```ts
await expect(
  page.getByRole(
    'row'
  ).filter({
    hasText:
      'Automation User'
  })
).toContainText(
  'Active'
);
```

---

## 75. List Ordering

If order matters:

```ts
const names =
  await page
    .getByTestId(
      'user-name'
    )
    .allTextContents();

expect(
  names
).toEqual([
  'Alice',
  'Bob',
  'Charlie'
]);
```

If order does not matter, do not make the test depend on order.

---

## 76. Set-Like Validation

For unordered data, normalize intentionally.

Example:

```ts
expect(
  [...actual].sort()
).toEqual(
  [...expected].sort()
);
```

This validates membership without requiring ordering.

---

## 77. Validation of Notifications

Example:

```ts
await expect(
  page.getByRole(
    'alert'
  )
).toContainText(
  'User created successfully'
);
```

Alerts are useful semantic targets for feedback messages.

---

## 78. Dialog Assertions

Example:

```ts
const dialog =
  page.getByRole(
    'dialog'
  );

await expect(
  dialog
).toBeVisible();

await expect(
  dialog.getByRole(
    'heading'
  )
).toHaveText(
  'Delete User'
);
```

---

## 79. Navigation Assertions

After an action:

```ts
await page.getByRole(
  'link',
  {
    name:
      'Admin'
  }
).click();

await expect(
  page
).toHaveURL(
  /\/admin/
);
```

Use URL assertions when navigation is part of the requirement.

---

## 80. Redirect Assertions

Example:

```ts
await expect(
  page
).toHaveURL(
  /\/login/
);
```

Useful for:

```text
Session expiry
Unauthorized access
Logout
Protected route
```

---

## 81. Authentication Assertions

After login:

```ts
await expect(
  page
).not.toHaveURL(
  /\/login/
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

Validate both navigation and application state when useful.

---

## 82. Authorization Assertions

For a restricted page:

```ts
await expect(
  page.getByRole(
    'heading',
    {
      name:
        'Access Denied'
    }
  )
).toBeVisible();
```

Or validate:

```text
403
```

through an API-level test.

---

## 83. Custom Expectations

Large frameworks may need domain-specific expectations.

Example:

```ts
expect.extend({
  async toBeActive(
    locator
  ) {
    const text =
      await locator.textContent();

    const pass =
      text?.trim() ===
      'Active';

    return {
      pass,
      message: () =>
        pass
          ? 'Expected not active'
          : 'Expected active'
    };
  }
});
```

Custom matchers should remain simple and well documented.

---

## 84. Why Custom Matchers?

Instead of repeatedly writing:

```ts
await expect(
  row
).toContainText(
  'Active'
);
```

a domain-specific matcher could express:

```ts
await expect(
  row
).toBeActive();
```

This can improve readability in large suites.

---

## 85. Custom Matcher Risks

Do not create custom matchers for every assertion.

Too many custom matchers can:

```text
Hide actual behavior
Increase framework complexity
Make debugging harder
Require additional documentation
```

Create them when they provide meaningful domain value.

---

## 86. Domain Assertion Helpers

An alternative to custom matchers is helper functions.

Example:

```ts
async function expectUserActive(
  row: Locator
) {
  await expect(
    row
  ).toContainText(
    'Active'
  );
}
```

This is often simpler.

---

## 87. Assertion Helper Design

A good assertion helper should:

```text
Have a clear name
Express business meaning
Use Playwright expectations
Produce useful failures
Avoid hidden side effects
```

---

## 88. Assertion Messages

Useful assertion messages explain:

```text
What was expected
Why it matters
Which entity failed
```

Example:

```ts
await expect(
  status
).toHaveText(
  'Active',
  {
    timeout:
      5000
  }
);
```

Keep messages meaningful without excessive verbosity.

---

## 89. Negative Assertions

Negative scenarios are essential.

Examples:

```text
User cannot access admin
Invalid login shows error
Deleted record disappears
Unauthorized API returns 401
Disabled button remains disabled
```

---

## 90. Negative Assertion Example

```ts
await expect(
  page.getByText(
    'Admin Panel'
  )
).not.toBeVisible();
```

Be careful: absence assertions can sometimes pass because the element has not loaded yet.

Use the correct assertion and context for the requirement.

---

## 91. Avoid False-Passing Negative Assertions

Suppose the page is still loading.

This:

```ts
await expect(
  locator
).not.toBeVisible();
```

could pass before the element has had a chance to appear if the test's setup is incorrect.

Ensure the application reaches the relevant state before asserting absence.

---

## 92. Assertion Ordering

Good:

```text
Navigate
 ↓
Wait for meaningful page state
 ↓
Perform action
 ↓
Assert outcome
```

Poor:

```text
Assert absence immediately
 ↓
Application still loading
```

Assertions should reflect meaningful application states.

---

## 93. Assertion Independence

Keep assertions focused.

Example:

```ts
await expect(
  heading
).toBeVisible();

await expect(
  status
).toHaveText(
  'Active'
);

await expect(
  button
).toBeEnabled();
```

Each checks a distinct requirement.

---

## 94. Avoid Over-Assertion

A test does not need to verify every DOM property.

Bad:

```text
Every class
Every CSS property
Every wrapper
Every internal attribute
Every DOM node
```

Focus on behavior and business requirements.

---

## 95. Contract vs Implementation Assertions

Prefer:

```text
User sees "Saved"
```

over:

```text
div has class "toast-success"
```

unless the CSS class itself is part of the contract.

---

## 96. Assertions in Page Objects

Page Objects can expose meaningful verification methods.

Example:

```ts
async expectDashboardVisible() {
  await expect(
    this.heading
  ).toBeVisible();
}
```

This keeps test code concise.

---

## 97. Assertions in Tests

Avoid putting every assertion inside Page Objects.

For example:

```ts
await dashboardPage
  .expectDashboardVisible();
```

is useful.

But business-specific validation may belong in the test:

```ts
await expect(
  dashboardPage.total
).toHaveText(
  '₹10,000'
);
```

Choose based on reuse and readability.

---

## 98. Assertion Layer Architecture

A mature framework can use:

```text
Tests
 ↓
Page Objects
 ↓
Assertion Helpers
 ↓
Playwright Expect
```

For API tests:

```text
Tests
 ↓
API Client
 ↓
Response Assertions
```

---

## 99. Enterprise Assertion Strategy

Use different assertion levels:

```text
UI State
API State
Business Rule
Database State
Visual State
Security State
Performance Threshold
```

Do not force every test to validate every layer.

---

## 100. Validation Matrix

| Validation | Example |
|---|---|
| UI | Button visible |
| Text | Success message |
| State | Button enabled |
| URL | Dashboard route |
| Attribute | Link href |
| Count | 10 rows |
| API | HTTP 201 |
| JSON | Response field |
| Business | Total = amount + tax |
| Security | Unauthorized blocked |
| Visual | Screenshot |
| Async | Job becomes completed |

---

## 101. Assertion Strategy for FinOps

For a financial application, useful assertions may include:

```text
Expense created
Expense amount correct
Category correct
Approval status correct
Total correct
Report total correct
Unauthorized user blocked
API status correct
Dashboard summary correct
```

Business assertions are especially important.

---

## 102. Example Financial Calculation Assertion

```ts
const expectedTotal =
  amount +
  tax -
  discount;

expect(
  actualTotal
).toBe(
  expectedTotal
);
```

Use numeric values rather than formatted UI strings when validating calculations.

---

## 103. Assertion Strategy for Login

```text
Input validation
 ↓
Login request
 ↓
Successful response
 ↓
Dashboard navigation
 ↓
Dashboard visible
```

Possible assertions:

```ts
await expect(
  response
).toBeOK();

await expect(
  page
).toHaveURL(
  /dashboard/
);

await expect(
  dashboardHeading
).toBeVisible();
```

---

## 104. Assertion Strategy for Admin

```text
Admin page
 ↓
User search
 ↓
Correct user row
 ↓
Status
 ↓
Action
```

Validate the specific business entity rather than only the page title.

---

## 105. Assertion Strategy for Reports

Reports often require:

```text
Filters
Date range
Row count
Totals
Grouping
Sorting
Export
```

Combine UI assertions with business-rule validation.

---

## 106. Assertion Strategy for API

For API tests:

```text
Status
Headers
Schema fields
Business values
Error contract
```

Do not only assert:

```ts
expect(
  response.ok()
).toBeTruthy();
```

Validate meaningful response content too.

---

## 107. API Error Contract

Example:

```ts
expect(
  response.status()
).toBe(400);

const body =
  await response.json();

expect(
  body
).toMatchObject({
  message:
    'Invalid request'
});
```

This validates both status and error behavior.

---

## 108. Async Workflow Validation

Example:

```text
Submit Report
 ↓
Job Created
 ↓
Poll Status
 ↓
Completed
 ↓
Download
 ↓
Validate File
```

Assertions should occur at each meaningful stage.

---

## 109. Avoid Arbitrary Retry Loops

Bad:

```ts
for (
  let i = 0;
  i < 10;
  i++
) {
  await page.waitForTimeout(
    1000
  );
}
```

Better:

```ts
await expect.poll(
  async () =>
    getJobStatus()
).toBe(
  'COMPLETED'
);
```

---

## 110. Assertion Failures

A good failure should tell you:

```text
Expected
Actual
Locator
Timeout
Call log
Test
```

This is one reason Playwright's built-in expectations are preferable to manually throwing generic errors.

---

## 111. Custom Error Messages

When adding custom error handling, preserve useful context.

Avoid:

```ts
throw new Error(
  'Failed'
);
```

Prefer:

```ts
throw new Error(
  `Expected job ${jobId} to complete`
);
```

Do not expose secrets in errors.

---

## 112. Assertions and Retries

A test retry does not replace good assertions.

Retry means:

```text
Run the test again after failure
```

Assertion retry means:

```text
Wait for expected state within the same test
```

These solve different problems.

---

## 113. Test Retry vs Assertion Retry

### Assertion Retry

Useful for:

```text
Async UI
Eventually consistent state
Network-driven rendering
```

### Test Retry

Useful for:

```text
Intermittent infrastructure
Unexpected transient failures
Flaky tests under investigation
```

Do not use test retries to hide deterministic defects.

---

## 114. Assertion Timeout vs Test Timeout

Test timeout controls the entire test.

Expectation timeout controls individual expectations.

Example:

```text
Test timeout
   ↓
Whole test

Expect timeout
   ↓
Individual assertion
```

Configure them intentionally.

---

## 115. Avoid Excessive Assertion Timeouts

Bad:

```ts
await expect(
  locator
).toBeVisible({
  timeout:
    120000
});
```

for every assertion.

This can make failures painfully slow.

Use realistic values.

---

## 116. Assertion Utility Example

```ts
export async function
expectVisible(
  locator: Locator
) {
  await expect(
    locator
  ).toBeVisible();
}
```

Keep wrappers only when they provide useful abstraction.

---

## 117. Domain Assertion Example

```ts
export async function
expectUserActive(
  row: Locator
) {
  await expect(
    row
  ).toContainText(
    'Active'
  );
}
```

This can make business-focused tests easier to read.

---

## 118. Assertion Naming

Good:

```text
expectDashboardVisible()
expectUserActive()
expectExpenseApproved()
expectReportTotal()
```

Poor:

```text
check1()
verify2()
assertThing()
```

---

## 119. Enterprise Assertion Review

Before approving an assertion:

```text
[ ] Does it verify a real requirement?
[ ] Does it auto-retry when appropriate?
[ ] Is timeout reasonable?
[ ] Is it stable?
[ ] Is it independent?
[ ] Does failure explain the problem?
[ ] Does it avoid implementation details?
[ ] Is it protected against false positives?
```

---

## 120. Common Assertion Mistakes

### Mistake 1

Using `waitForTimeout()` before every assertion.

### Mistake 2

Using `isVisible()` instead of auto-retrying expectations.

### Mistake 3

Using `toBeTruthy()` for everything.

### Mistake 4

Overusing soft assertions.

### Mistake 5

Over-asserting implementation details.

### Mistake 6

Using huge timeouts.

### Mistake 7

Ignoring business rules.

### Mistake 8

Only testing happy paths.

### Mistake 9

Using retries to hide defects.

### Mistake 10

Making negative assertions before the application reaches the correct state.

---

## 121. Recommended Assertion Pattern

Use:

```ts
await expect(
  locator
).toBeVisible();
```

instead of:

```ts
expect(
  await locator.isVisible()
).toBe(true);
```

The first approach provides Playwright's retry behavior.

---

## 122. Recommended Async Pattern

Use:

```ts
await expect.poll(
  async () =>
    getStatus()
).toBe(
  'COMPLETED'
);
```

instead of:

```ts
await page.waitForTimeout(
  10000
);

expect(
  await getStatus()
).toBe(
  'COMPLETED'
);
```

---

## 123. Recommended API Pattern

```ts
const response =
  await request.get(
    '/api/users'
  );

expect(
  response.status()
).toBe(200);

const body =
  await response.json();

expect(
  body.users
).toBeDefined();
```

Validate both protocol and meaningful data.

---

## 124. Recommended UI + Business Pattern

```ts
await expect(
  successMessage
).toBeVisible();

const actualTotal =
  await expensePage
    .getTotal();

expect(
  actualTotal
).toBe(
  expectedTotal
);
```

This combines UI validation with business logic.

---

## 125. Enterprise Assertion Architecture

```text
                         Test
                           │
             ┌─────────────┼─────────────┐
             │             │             │
            UI            API        Business
             │             │             │
          Locator       Response      Rules
          Expect         Expect        Expect
             │             │             │
             └─────────────┼─────────────┘
                           │
                    Assertion Helpers
                           │
                     Custom Matchers
                           │
                      Test Report
```

---

## 126. Hands-On Exercises

### Exercise 1 — Basic Assertions

Practice:

```text
toBeVisible
toHaveText
toHaveValue
toBeEnabled
toBeChecked
```

### Exercise 2 — URL Assertions

Validate navigation with:

```text
toHaveURL()
```

### Exercise 3 — Soft Assertions

Validate multiple dashboard widgets using:

```ts
expect.soft()
```

### Exercise 4 — Polling

Create an asynchronous job and poll until:

```text
COMPLETED
```

### Exercise 5 — API Assertions

Validate:

```text
Status
Headers
JSON
Business data
```

### Exercise 6 — Negative Assertions

Test:

```text
Unauthorized
Invalid login
Deleted record
Disabled action
```

### Exercise 7 — Business Assertions

Validate a financial calculation:

```text
Total =
Amount + Tax - Discount
```

### Exercise 8 — Visual Assertions

Create a stable visual baseline and validate a page.

### Exercise 9 — Custom Assertion Helper

Create:

```ts
expectUserActive()
```

### Exercise 10 — Enterprise Assertion Layer

Build reusable assertion helpers for:

```text
Users
Expenses
Reports
Authentication
API responses
```

---

## 127. Enterprise Assertion Challenge

Create an assertion framework supporting:

```text
UI Assertions
API Assertions
Business Assertions
Async Polling
Soft Assertions
Visual Assertions
Custom Domain Assertions
```

Requirements:

1. Use Playwright auto-retrying expectations.
2. Avoid unnecessary fixed waits.
3. Configure realistic assertion timeouts.
4. Use soft assertions for independent checks.
5. Use polling for eventually consistent state.
6. Validate API status and body.
7. Validate business calculations.
8. Validate negative scenarios.
9. Validate navigation.
10. Validate permissions.
11. Validate visual state where useful.
12. Build reusable domain helpers.
13. Produce clear failures.
14. Avoid excessive abstraction.
15. Protect sensitive diagnostic information.

---

## 128. Final Enterprise Validation Flow

```text
                     Test Action
                         │
                         ▼
                  Application State
                         │
          ┌──────────────┼──────────────┐
          │              │              │
         UI             API          Business
          │              │              │
       Locator        Response        Rules
       Expect          Expect         Expect
          │              │              │
          └──────────────┼──────────────┘
                         │
                    Poll if Needed
                         │
                    Soft if Needed
                         │
                  Failure Diagnostics
                         │
                     Test Report
```

---

## 129. Final Checklist

```text
[ ] expect() understood
[ ] Auto-retrying assertions understood
[ ] Assertion timeout understood
[ ] toBeVisible mastered
[ ] toBeHidden mastered
[ ] toBeEnabled mastered
[ ] toBeDisabled mastered
[ ] toBeEditable mastered
[ ] toBeChecked mastered
[ ] toHaveText mastered
[ ] toContainText mastered
[ ] toHaveValue mastered
[ ] toHaveAttribute mastered
[ ] toHaveCount mastered
[ ] toHaveURL mastered
[ ] toHaveTitle mastered
[ ] Screenshot assertions understood
[ ] Generic assertions understood
[ ] Soft assertions mastered
[ ] Polling understood
[ ] API assertions practiced
[ ] Business assertions practiced
[ ] Negative assertions practiced
[ ] Async workflow validation practiced
[ ] Custom assertion helpers understood
[ ] Test retry vs assertion retry understood
[ ] Assertion timeout vs test timeout understood
[ ] False-positive prevention understood
[ ] Enterprise assertion strategy documented
```

---

## 130. Key Takeaways

- Assertions are the verification layer of a reliable Playwright test.
- Prefer Playwright's auto-retrying expectations over manual state checks.
- `expect()` should validate meaningful application behavior.
- Soft assertions are useful for independent validations.
- Polling is valuable for eventually consistent backend and asynchronous workflows.
- `waitForTimeout()` should not be the default synchronization strategy.
- API tests should validate status, response data, and business meaning.
- UI tests should focus on user-visible behavior rather than implementation details.
- Business assertions catch defects that simple UI checks can miss.
- Negative scenarios are as important as happy paths.
- Test retries and assertion retries solve different problems.
- Assertion timeouts should be realistic.
- Custom matchers and helpers should be introduced only when they provide meaningful domain value.
- Enterprise automation should combine UI, API, business, asynchronous, and visual validation where appropriate.
- A good assertion makes failures easy to understand and helps identify the root cause quickly.

---

## 131. Next Step

**Step 100 — Mastering Playwright Test Data Management: Static Data, Dynamic Data, Fixtures, Factories, Faker, API Seeding, Database Setup, Cleanup, Isolation, Environment Data & Enterprise Test Data Architecture**

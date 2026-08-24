# Playwright with TypeScript Mastery Bootcamp

# Step 85 — Mastering Playwright Assertions, Auto-Waiting, Web-First Assertions, Soft Assertions, Custom Matchers & Advanced Validation

## 1. Introduction

Assertions are the foundation of reliable Playwright tests.

A test should not only perform actions such as:

```ts
await page.getByRole('button', {
  name: 'Login'
}).click();
```

It should also verify the expected result:

```ts
await expect(
  page.getByRole('heading', {
    name: 'Dashboard'
  })
).toBeVisible();
```

Playwright provides powerful assertion capabilities designed specifically for web applications.

Important concepts include:

```text
Assertions
Auto-Waiting
Web-First Assertions
Locator Assertions
Soft Assertions
Negative Assertions
URL Assertions
Text Assertions
Attribute Assertions
State Assertions
Custom Matchers
API Assertions
Advanced Validation
```

---

## 2. What Is an Assertion?

An assertion verifies that the actual result matches the expected result.

Example:

```ts
expect(user.name).toBe('Sandeep');
```

In UI automation:

```ts
await expect(
  page.getByText('Welcome')
).toBeVisible();
```

If the expected condition is not satisfied, the test fails.

---

## 3. Importing `expect`

Playwright tests normally use:

```ts
import {
  test,
  expect
} from '@playwright/test';
```

Example:

```ts
test('dashboard validation', async ({
  page
}) => {
  await page.goto('/dashboard');

  await expect(
    page.getByRole('heading', {
      name: 'Dashboard'
    })
  ).toBeVisible();
});
```

---

## 4. Why Playwright Assertions Are Special

Playwright assertions are designed to work with dynamic web pages.

For example:

```ts
await expect(
  page.getByText('Dashboard')
).toBeVisible();
```

Playwright can automatically wait for the expected condition instead of immediately checking once.

This is called a **web-first assertion**.

---

## 5. Auto-Waiting

Playwright automatically waits for many conditions before performing actions.

For example:

```ts
await page.getByRole('button', {
  name: 'Submit'
}).click();
```

Playwright waits for the element to become actionable.

This reduces the need for manual sleeps.

---

## 6. Why `waitForTimeout()` Is Usually a Bad Solution

Avoid:

```ts
await page.waitForTimeout(5000);
```

This creates an arbitrary delay.

Problems include:

- Slow tests
- Flaky tests
- Unnecessary waiting
- Timing differences between environments

Prefer:

```ts
await expect(
  page.getByText('Success')
).toBeVisible();
```

---

## 7. Web-First Assertions

Web-first assertions automatically wait for the expected web condition.

Example:

```ts
await expect(
  page.getByRole('button', {
    name: 'Save'
  })
).toBeEnabled();
```

The assertion waits until the condition is satisfied or the timeout is reached.

---

## 8. `toBeVisible()`

Use:

```ts
await expect(locator).toBeVisible();
```

Example:

```ts
await expect(
  page.getByText('Dashboard')
).toBeVisible();
```

This verifies that the element is visible to the user.

---

## 9. `toBeHidden()`

Use:

```ts
await expect(locator).toBeHidden();
```

Example:

```ts
await expect(
  page.getByText('Loading...')
).toBeHidden();
```

Useful for verifying that loading indicators disappear.

---

## 10. `toBeAttached()`

Use:

```ts
await expect(locator).toBeAttached();
```

This verifies that the element exists in the DOM.

An element can be attached without being visible.

---

## 11. `toBeDetached()`

Use:

```ts
await expect(locator).toBeDetached();
```

Useful when an element should be removed from the DOM.

---

## 12. `toBeEnabled()`

Example:

```ts
await expect(
  page.getByRole('button', {
    name: 'Submit'
  })
).toBeEnabled();
```

Useful for validating form state.

---

## 13. `toBeDisabled()`

Example:

```ts
await expect(
  page.getByRole('button', {
    name: 'Submit'
  })
).toBeDisabled();
```

Useful when a button should remain unavailable until required fields are completed.

---

## 14. `toBeEditable()`

Example:

```ts
await expect(
  page.getByLabel('Username')
).toBeEditable();
```

This verifies that the input can be edited.

---

## 15. `toBeChecked()`

For checkboxes or radio buttons:

```ts
await expect(
  page.getByRole('checkbox', {
    name: 'Accept Terms'
  })
).toBeChecked();
```

---

## 16. `toBeFocused()`

Example:

```ts
await expect(
  page.getByLabel('Username')
).toBeFocused();
```

Useful for keyboard and accessibility workflows.

---

## 17. `toBeEmpty()`

Example:

```ts
await expect(
  page.getByLabel('Search')
).toBeEmpty();
```

Useful for validating form reset behavior.

---

## 18. `toHaveText()`

Example:

```ts
await expect(
  page.getByRole('heading')
).toHaveText('Dashboard');
```

This validates the exact expected text content according to the matcher behavior.

---

## 19. `toContainText()`

Use when only part of the text matters:

```ts
await expect(
  page.getByRole('heading')
).toContainText('Dash');
```

This is useful when the element contains additional dynamic text.

---

## 20. `toHaveValue()`

For input fields:

```ts
await expect(
  page.getByLabel('Username')
).toHaveValue('sandeep');
```

Useful for verifying form values.

---

## 21. `toHaveAttribute()`

Example:

```ts
await expect(
  page.getByRole('button', {
    name: 'Submit'
  })
).toHaveAttribute(
  'type',
  'submit'
);
```

Useful for validating important DOM attributes.

---

## 22. `toHaveClass()`

Example:

```ts
await expect(
  page.locator('.status')
).toHaveClass(/success/);
```

Regular expressions can be useful when classes contain dynamic values.

---

## 23. `toHaveCSS()`

Example:

```ts
await expect(
  page.getByRole('button', {
    name: 'Submit'
  })
).toHaveCSS(
  'display',
  'block'
);
```

Use CSS assertions selectively. Prefer user-visible behavior over implementation details.

---

## 24. `toHaveId()`

Example:

```ts
await expect(
  page.locator('#username')
).toHaveId('username');
```

Use semantic locators where possible instead of relying heavily on IDs.

---

## 25. `toHaveCount()`

Example:

```ts
await expect(
  page.getByRole('listitem')
).toHaveCount(10);
```

Useful for validating collections.

---

## 26. `toHaveURL()`

Example:

```ts
await expect(page)
  .toHaveURL('/dashboard');
```

Or:

```ts
await expect(page)
  .toHaveURL(/dashboard/);
```

This is useful for validating navigation.

---

## 27. `toHaveTitle()`

Example:

```ts
await expect(page)
  .toHaveTitle('Dashboard');
```

Useful for validating page metadata.

---

## 28. `toHaveScreenshot()`

Playwright can perform visual comparisons:

```ts
await expect(page)
  .toHaveScreenshot();
```

This creates a screenshot comparison against a stored baseline.

Visual testing should be configured carefully because rendering differences can produce false positives.

---

## 29. Negative Assertions

Use `not` for negative expectations.

Example:

```ts
await expect(
  page.getByText('Error')
).not.toBeVisible();
```

Other examples:

```ts
await expect(locator)
  .not.toBeEnabled();

await expect(locator)
  .not.toHaveText('Invalid');
```

Negative assertions should represent meaningful application behavior.

---

## 30. Exact Text vs Partial Text

Exact:

```ts
await expect(locator)
  .toHaveText('Dashboard');
```

Partial:

```ts
await expect(locator)
  .toContainText('Dash');
```

Choose based on the requirement.

---

## 31. Regular Expressions

Assertions support regular expressions.

Example:

```ts
await expect(
  page.getByText(/welcome/i)
).toBeVisible();
```

Another example:

```ts
await expect(locator)
  .toHaveText(/Order #\d+/);
```

This is useful for dynamic content.

---

## 32. Array Text Assertions

For multiple elements:

```ts
await expect(
  page.getByRole('listitem')
).toHaveText([
  'Apple',
  'Banana',
  'Orange'
]);
```

This is useful when validating ordered lists.

---

## 33. `toHaveValues()`

For multiple selected options or relevant form values:

```ts
await expect(
  page.locator('select')
).toHaveValues([
  'admin',
  'manager'
]);
```

Use the matcher appropriate to the form control.

---

## 34. URL Assertions with Dynamic Parameters

Example:

```ts
await expect(page)
  .toHaveURL(/users\/\d+/);
```

This validates a URL containing a dynamic numeric ID.

---

## 35. Assertion Timeout

Configure globally:

```ts
expect: {
  timeout: 5_000
}
```

Or override a specific assertion:

```ts
await expect(locator)
  .toBeVisible({
    timeout: 10_000
  });
```

Use larger values only when the application's behavior genuinely requires them.

---

## 36. Test Timeout vs Assertion Timeout

### Test timeout

Controls the overall test duration:

```ts
timeout: 30_000
```

### Assertion timeout

Controls how long a web-first assertion waits:

```ts
expect: {
  timeout: 5_000
}
```

These solve different problems.

---

## 37. Soft Assertions

A normal assertion can stop the test after failure.

Example:

```ts
expect(actual).toBe(expected);
```

A soft assertion records the failure and allows the test to continue:

```ts
await expect.soft(
  page.getByText('Name')
).toBeVisible();
```

---

## 38. Why Use Soft Assertions?

Soft assertions are useful when you want to validate multiple independent UI elements in one test.

Example:

```ts
await expect.soft(
  page.getByText('Name')
).toBeVisible();

await expect.soft(
  page.getByText('Email')
).toBeVisible();

await expect.soft(
  page.getByText('Role')
).toBeVisible();
```

The test can collect multiple failures.

---

## 39. Hard vs Soft Assertions

### Hard assertion

```text
Assertion fails
     ↓
Test flow stops
```

### Soft assertion

```text
Assertion fails
     ↓
Failure recorded
     ↓
Test continues
```

Use hard assertions for critical prerequisites.

Use soft assertions for independent validations.

---

## 40. Checking Soft Assertion Results

Playwright can report soft assertion failures at the end of the test.

For advanced control, use the test information and Playwright's assertion mechanisms rather than manually maintaining many boolean flags.

---

## 41. Assertions on API Responses

Example:

```ts
const response =
  await request.get('/api/users');

expect(response.status())
  .toBe(200);
```

Then:

```ts
const data =
  await response.json();

expect(data.users.length)
  .toBeGreaterThan(0);
```

---

## 42. API Response Headers

Example:

```ts
expect(
  response.headers()['content-type']
).toContain('application/json');
```

This validates response metadata.

---

## 43. JSON Property Assertions

Example:

```ts
const data =
  await response.json();

expect(data).toHaveProperty('id');
expect(data).toHaveProperty('name');
```

For nested data:

```ts
expect(data.user)
  .toHaveProperty('id');
```

---

## 44. Numeric Assertions

Common Jest-style matchers available through Playwright's `expect` include:

```ts
expect(value).toBe(10);

expect(value).toEqual(10);

expect(value).toBeGreaterThan(5);

expect(value).toBeGreaterThanOrEqual(5);

expect(value).toBeLessThan(20);

expect(value).toBeLessThanOrEqual(20);
```

---

## 45. String Assertions

Examples:

```ts
expect(name).toBe('Sandeep');

expect(name).toContain('Sand');

expect(name).toMatch(/sand/i);
```

---

## 46. Object Assertions

Example:

```ts
expect(user).toEqual({
  id: 10,
  role: 'QA'
});
```

For partial matching:

```ts
expect(user).toMatchObject({
  role: 'QA'
});
```

---

## 47. Array Assertions

Example:

```ts
expect(users).toHaveLength(3);
```

Another:

```ts
expect(users)
  .toContainEqual({
    id: 10,
    name: 'Sandeep'
  });
```

---

## 48. `toBe` vs `toEqual`

### `toBe`

Primarily checks strict identity/equality behavior.

```ts
expect(status).toBe(200);
```

### `toEqual`

Performs deep equality comparison for objects and arrays.

```ts
expect(user).toEqual(expectedUser);
```

Use the matcher that matches the data type and requirement.

---

## 49. `toStrictEqual`

For stricter object comparison:

```ts
expect(actual)
  .toStrictEqual(expected);
```

This can be useful when object structure and undefined properties matter.

---

## 50. Custom Assertion Helpers

You can create reusable validation functions.

Example:

```ts
export async function
expectDashboard(page: Page) {

  await expect(
    page.getByRole('heading', {
      name: 'Dashboard'
    })
  ).toBeVisible();

  await expect(
    page.getByRole('navigation')
  ).toBeVisible();
}
```

Then:

```ts
await expectDashboard(page);
```

This is often simpler than creating custom matcher infrastructure.

---

## 51. Custom Matchers

For highly reusable domain-specific assertions, custom matchers can be created around Playwright/Jest-style expectations.

Conceptually:

```ts
expect.extend({
  async toBeValidUser(
    received
  ) {
    // validation
  }
});
```

Use custom matchers only when they provide clear value.

---

## 52. Domain-Specific Assertions

Example:

```text
expectUserToBeActive()
expectOrderToBeCompleted()
expectDashboardToBeLoaded()
expectExpenseToBeApproved()
```

These can make enterprise tests more expressive.

---

## 53. Page Object Assertions

A Page Object can contain reusable validation methods:

```ts
async verifyDashboard() {
  await expect(
    this.page.getByRole('heading', {
      name: 'Dashboard'
    })
  ).toBeVisible();
}
```

Test:

```ts
await dashboardPage.verifyDashboard();
```

Keep assertions related to the Page Object's UI responsibility when that improves reuse.

---

## 54. Business-Level Assertions

For complex workflows:

```text
UI Action
   ↓
Business Event
   ↓
API Validation
   ↓
Database Validation
```

Example:

```ts
await expensePage.submitExpense();

const response =
  await expensesApi.getExpense(id);

expect(response.status())
  .toBe(200);
```

This gives stronger end-to-end confidence.

---

## 55. Assertion Strategy

A good test should validate:

```text
Action
 ↓
Expected State
 ↓
Business Result
```

Avoid tests that contain actions but no meaningful assertions.

Bad:

```ts
await page.getByRole('button', {
  name: 'Submit'
}).click();
```

Better:

```ts
await page.getByRole('button', {
  name: 'Submit'
}).click();

await expect(
  page.getByText('Expense submitted')
).toBeVisible();
```

---

## 56. Avoiding Weak Assertions

Weak:

```ts
expect(true).toBeTruthy();
```

Weak:

```ts
expect(response).toBeDefined();
```

Better:

```ts
expect(response.status())
  .toBe(201);
```

Best:

```ts
expect(response.status())
  .toBe(201);

const data =
  await response.json();

expect(data.id).toBeDefined();
expect(data.status).toBe('CREATED');
```

Assertions should verify real requirements.

---

## 57. Assertions and Dynamic Applications

Modern applications often update asynchronously.

Prefer:

```ts
await expect(
  page.getByText('Saved')
).toBeVisible();
```

instead of:

```ts
await page.waitForTimeout(2000);

expect(
  await page.getByText('Saved').isVisible()
).toBe(true);
```

The first approach is more reliable because the assertion waits for the condition.

---

## 58. Locator Assertions vs Manual Checks

Prefer:

```ts
await expect(locator)
  .toBeVisible();
```

over:

```ts
expect(await locator.isVisible())
  .toBe(true);
```

The web-first assertion automatically waits.

---

## 59. Manual URL Check vs Web-First URL Assertion

Prefer:

```ts
await expect(page)
  .toHaveURL('/dashboard');
```

instead of immediately checking:

```ts
expect(page.url())
  .toContain('/dashboard');
```

The web-first version waits for navigation-related changes.

---

## 60. Assertion Best Practices

1. Assert observable behavior.
2. Prefer web-first assertions.
3. Avoid arbitrary sleeps.
4. Use specific assertions.
5. Use soft assertions only where appropriate.
6. Validate API response data, not just status.
7. Keep assertions close to the behavior they validate.
8. Avoid implementation-detail assertions when user behavior is available.
9. Use domain-specific assertion helpers for repeated business validations.
10. Keep timeout values realistic.

---

## 61. Production-Quality Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('user registration', async ({
  page
}) => {
  await page.goto('/register');

  await page.getByLabel('Name')
    .fill('Sandeep');

  await page.getByLabel('Email')
    .fill('sandeep@example.com');

  await page.getByLabel('Password')
    .fill('TestPassword');

  await page.getByRole('button', {
    name: 'Register'
  }).click();

  await expect(
    page.getByRole('heading', {
      name: 'Welcome'
    })
  ).toBeVisible();

  await expect(
    page.getByText('sandeep@example.com')
  ).toBeVisible();

  await expect(page)
    .toHaveURL(/dashboard/);
});
```

---

## 62. Multiple Soft Assertions Example

```ts
test('profile validation', async ({
  page
}) => {
  await page.goto('/profile');

  await expect.soft(
    page.getByLabel('Name')
  ).toHaveValue('Sandeep');

  await expect.soft(
    page.getByLabel('Email')
  ).toHaveValue(
    'sandeep@example.com'
  );

  await expect.soft(
    page.getByRole('button', {
      name: 'Save'
    })
  ).toBeEnabled();
});
```

This can reveal multiple independent UI problems in one execution.

---

## 63. API + UI Assertion Example

```ts
test('user creation', async ({
  page,
  request
}) => {
  const response =
    await request.post('/api/users', {
      data: {
        name: 'Sandeep'
      }
    });

  expect(response.status())
    .toBe(201);

  const user =
    await response.json();

  await page.goto('/users');

  await expect(
    page.getByText(user.name)
  ).toBeVisible();
});
```

This combines API validation with UI verification.

---

## 64. Common Mistakes

### Mistake 1 — Using `waitForTimeout()`

Use web-first assertions instead.

### Mistake 2 — Checking `isVisible()` immediately

Use:

```ts
await expect(locator)
  .toBeVisible();
```

### Mistake 3 — Asserting only status codes

Validate important response content too.

### Mistake 4 — Too many soft assertions

Use soft assertions for independent validations, not for every assertion.

### Mistake 5 — Weak assertions

Avoid:

```ts
expect(true).toBeTruthy();
```

### Mistake 6 — Overusing CSS assertions

Prefer user-visible behavior when possible.

### Mistake 7 — Excessive timeouts

Large timeouts can hide real performance or synchronization problems.

---

## 65. Interview Questions

### Q1. What is a web-first assertion?

An assertion designed for web applications that automatically waits for the expected condition.

### Q2. Why is `toBeVisible()` better than `isVisible()` followed by `expect()`?

`toBeVisible()` automatically waits for the condition.

### Q3. What is the difference between hard and soft assertions?

Hard assertions fail the test immediately when the assertion fails. Soft assertions record the failure and allow execution to continue.

### Q4. Why should `waitForTimeout()` generally be avoided?

It uses an arbitrary delay and can create slow and flaky tests.

### Q5. How do you assert a URL?

```ts
await expect(page)
  .toHaveURL('/dashboard');
```

### Q6. How do you assert input value?

```ts
await expect(locator)
  .toHaveValue('Sandeep');
```

### Q7. How do you assert multiple elements?

Use locator collections with matchers such as:

```ts
toHaveCount()
toHaveText()
```

### Q8. What is the difference between `toBe` and `toEqual`?

`toBe` is primarily for strict equality/identity behavior, while `toEqual` supports deep equality.

### Q9. When should custom matchers be used?

When repeated domain-specific validations provide enough value to justify the additional abstraction.

### Q10. What is the most important assertion principle?

Validate meaningful application behavior rather than merely proving that an action executed.

---

## 66. Hands-On Exercises

### Exercise 1 — Visibility

Verify that the dashboard heading is visible.

### Exercise 2 — State

Verify that:

```text
Save button
```

is disabled before required fields are completed.

### Exercise 3 — Form Value

Fill an input and assert its value.

### Exercise 4 — URL

Perform navigation and verify the destination URL.

### Exercise 5 — Collection

Verify that a table contains exactly 10 rows.

### Exercise 6 — Soft Assertions

Validate multiple independent profile fields using soft assertions.

### Exercise 7 — API Assertions

Validate:

```text
status
headers
JSON properties
```

### Exercise 8 — Dynamic Text

Use a regular expression to validate a dynamically generated order number.

### Exercise 9 — Custom Helper

Create:

```text
expectDashboard()
```

for repeated dashboard validation.

### Exercise 10 — Remove Sleeps

Find tests using:

```ts
waitForTimeout()
```

and replace unnecessary sleeps with web-first assertions.

---

## 67. Key Takeaways

- Assertions verify expected behavior.
- Playwright provides powerful web-first assertions.
- Auto-waiting reduces manual synchronization.
- `toBeVisible()` is preferable to immediate `isVisible()` checks.
- Avoid arbitrary `waitForTimeout()` calls.
- Use specific assertions such as `toHaveText`, `toHaveValue`, `toHaveCount`, and `toHaveURL`.
- Negative assertions use `.not`.
- Soft assertions allow multiple independent validations.
- API responses should be validated beyond status codes.
- Object and array assertions support deep validation.
- Custom assertion helpers can improve readability.
- Custom matchers should be introduced only when they provide clear domain value.
- Assertions should focus on observable behavior.
- Keep timeout values realistic.
- Strong assertions make automation more trustworthy and easier to debug.

---

## 68. Enterprise Assertion Challenge

Build a validation framework that supports:

```text
UI Assertions
      ↓
Page Object Assertions
      ↓
API Assertions
      ↓
Business Assertions
```

Requirements:

1. Use web-first assertions.
2. Remove unnecessary fixed waits.
3. Add meaningful negative assertions.
4. Use soft assertions for independent fields.
5. Validate API response bodies.
6. Create reusable dashboard assertions.
7. Validate dynamic URLs.
8. Validate collections.
9. Add at least one domain-specific assertion helper.
10. Ensure failures clearly identify the violated business requirement.

---

## 69. Next Step

**Step 86 — Mastering Playwright Locators, Locator Strategy, Strict Mode, Chaining, Filtering, Dynamic Elements & Enterprise Locator Design**

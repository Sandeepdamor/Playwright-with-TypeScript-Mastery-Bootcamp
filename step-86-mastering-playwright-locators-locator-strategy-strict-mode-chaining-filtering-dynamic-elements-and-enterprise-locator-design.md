# Playwright with TypeScript Mastery Bootcamp

# Step 86 — Mastering Playwright Locators, Locator Strategy, Strict Mode, Chaining, Filtering, Dynamic Elements & Enterprise Locator Design

## 1. Introduction

Locators are one of the most important concepts in Playwright.

A locator identifies an element on a web page and allows Playwright to interact with it or assert its state.

Examples:

```ts
page.getByRole('button', { name: 'Login' });

page.getByLabel('Username');

page.getByPlaceholder('Enter username');

page.getByText('Dashboard');

page.locator('.login-button');
```

A strong locator strategy makes tests:

- Stable
- Readable
- Maintainable
- Less flaky
- Easier to debug

---

## 2. What Is a Locator?

A locator is a Playwright object that identifies one or more elements.

Example:

```ts
const loginButton = page.getByRole(
  'button',
  { name: 'Login' }
);

await loginButton.click();
```

Locators are resolved when actions or assertions are performed, which helps Playwright work with dynamic pages.

---

## 3. Recommended Locator Priority

A practical locator strategy is:

```text
1. getByRole()
2. getByLabel()
3. getByPlaceholder()
4. getByText()
5. getByAltText()
6. getByTitle()
7. getByTestId()
8. locator()
```

The exact choice depends on the application's markup and accessibility.

---

## 4. `getByRole()`

Example:

```ts
await page.getByRole(
  'button',
  { name: 'Login' }
).click();
```

Common roles include:

```text
button
link
textbox
checkbox
radio
heading
combobox
listbox
option
dialog
tab
row
cell
```

Role-based locators are generally a strong default for user-facing elements.

---

## 5. `getByLabel()`

For form controls:

```ts
await page.getByLabel(
  'Username'
).fill('sandeep');
```

Example HTML:

```html
<label for="username">Username</label>
<input id="username">
```

This is a strong strategy for forms.

---

## 6. Other Built-In Locators

### Placeholder

```ts
page.getByPlaceholder('Enter username');
```

### Text

```ts
page.getByText('Dashboard');
```

### Alt Text

```ts
page.getByAltText('Company Logo');
```

### Title

```ts
page.getByTitle('Refresh');
```

### Test ID

```ts
page.getByTestId('login-button');
```

Use the most stable and meaningful locator available.

---

## 7. Custom Test ID

Configure:

```ts
use: {
  testIdAttribute: 'data-qa'
}
```

HTML:

```html
<button data-qa="login">
  Login
</button>
```

Then:

```ts
page.getByTestId('login');
```

Test IDs are useful when semantic locators are insufficient.

---

## 8. CSS and XPath

CSS:

```ts
page.locator('.login-button');
```

XPath:

```ts
page.locator(
  '//button[@type="submit"]'
);
```

They are supported, but should usually not be the first choice.

Avoid brittle selectors such as:

```text
/html/body/div[2]/div[3]/button[1]
```

or generated classes such as:

```text
.css-1a2b3c
```

---

## 9. Strict Mode

Playwright actions are strict when they require a single target.

If this matches multiple buttons:

```ts
await page.getByRole(
  'button',
  { name: 'Delete' }
).click();
```

Playwright can report a strict mode violation.

This is useful because it prevents accidentally interacting with the wrong element.

---

## 10. Fixing Strict Mode Violations

Do not immediately use:

```ts
.first()
```

Instead, make the locator more specific.

Example:

```ts
const row = page.getByRole(
  'row',
  { name: 'Sandeep' }
);

await row.getByRole(
  'button',
  { name: 'Delete' }
).click();
```

The locator is now scoped to the intended row.

---

## 11. `.first()`, `.last()`, and `.nth()`

Use these only when position has actual meaning.

```ts
locator.first();
locator.last();
locator.nth(2);
```

Example:

```ts
await page
  .getByRole('listitem')
  .first()
  .click();
```

Avoid using positional selectors merely to hide ambiguous locators.

---

## 12. Locator Chaining

Example:

```ts
const row = page.getByRole(
  'row',
  { name: 'Sandeep' }
);

await row.getByRole(
  'button',
  { name: 'Edit' }
).click();
```

Chaining creates a clear hierarchy:

```text
Page
 ↓
Row
 ↓
Button
```

---

## 13. Filtering

Use `.filter()` to narrow matching elements.

```ts
const row = page
  .getByRole('row')
  .filter({
    hasText: 'Sandeep'
  });
```

Then:

```ts
await row.getByRole(
  'button',
  { name: 'Edit' }
).click();
```

---

## 14. `hasText`

Example:

```ts
const card = page
  .getByRole('article')
  .filter({
    hasText: 'Laptop'
  });
```

This is useful for repeated cards and lists.

---

## 15. `has`

Example:

```ts
const card = page
  .getByRole('article')
  .filter({
    has: page.getByRole(
      'button',
      { name: 'Buy' }
    )
  });
```

This finds an article containing the specified button.

---

## 16. Combining Filters

```ts
const card = page
  .getByRole('article')
  .filter({
    hasText: 'Laptop',
    has: page.getByRole(
      'button',
      { name: 'Buy' }
    )
  });
```

This provides precise component-level selection.

---

## 17. `and()` and `or()`

Playwright supports combining locators.

Example:

```ts
const button = page
  .getByRole('button')
  .and(page.getByTitle('Save'));
```

Alternative matching:

```ts
const target = page
  .getByRole('button', {
    name: 'Accept'
  })
  .or(
    page.getByRole('button', {
      name: 'Continue'
    })
  );
```

Use alternatives carefully when both elements could exist.

---

## 18. Table Locators

For enterprise tables:

```text
Table
 ↓
Row
 ↓
Cell
 ↓
Action
```

Example:

```ts
const row = page
  .getByRole('row')
  .filter({
    hasText: 'Sandeep'
  });

await row.getByRole(
  'button',
  { name: 'Edit' }
).click();
```

---

## 19. Dialog Locators

Example:

```ts
const dialog =
  page.getByRole('dialog');

await dialog.getByLabel(
  'Reason'
).fill('Approved');

await dialog.getByRole(
  'button',
  { name: 'Confirm' }
).click();
```

Scoping actions to the dialog prevents duplicate-element problems.

---

## 20. Dynamic Elements

Avoid generated IDs:

```text
#user-839291
```

Avoid generated classes:

```text
.css-1x8abc
```

Prefer:

```ts
page.getByRole(
  'row',
  { name: /Sandeep/ }
);
```

or a stable test ID.

---

## 21. Dynamic Locator Factories

A Page Object can expose reusable dynamic locators:

```ts
userRow(name: string) {
  return this.page
    .getByRole('row')
    .filter({
      hasText: name
    });
}
```

Usage:

```ts
await adminPage
  .userRow('Sandeep')
  .getByRole(
    'button',
    { name: 'Edit' }
  )
  .click();
```

---

## 22. Component Objects

Complex pages can be divided into components:

```text
AdminPage
 ├── UserTable
 ├── UserDialog
 └── SearchPanel
```

Example:

```ts
class UserTable {
  constructor(
    private readonly page: Page
  ) {}

  row(name: string) {
    return this.page
      .getByRole('row')
      .filter({
        hasText: name
      });
  }

  editButton(name: string) {
    return this.row(name)
      .getByRole(
        'button',
        { name: 'Edit' }
      );
  }
}
```

This makes repeated UI structures reusable.

---

## 23. Page Object Locator Strategy

Example:

```ts
export class LoginPage {
  constructor(
    private readonly page: Page
  ) {}

  readonly username =
    this.page.getByLabel('Username');

  readonly password =
    this.page.getByLabel('Password');

  readonly loginButton =
    this.page.getByRole(
      'button',
      { name: 'Login' }
    );
}
```

Centralizing locators makes UI changes easier to maintain.

---

## 24. Locator Assertions

Use web-first assertions:

```ts
await expect(locator)
  .toBeVisible();

await expect(locator)
  .toHaveText('Dashboard');

await expect(locator)
  .toHaveCount(10);
```

Avoid immediate checks such as:

```ts
expect(await locator.isVisible())
  .toBe(true);
```

when a web-first assertion can be used.

---

## 25. Locator Count

You can inspect count:

```ts
const count =
  await page.getByRole(
    'row'
  ).count();
```

For validation, prefer:

```ts
await expect(
  page.getByRole('row')
).toHaveCount(10);
```

This can wait for the expected state.

---

## 26. Locator Evaluation

DOM evaluation is available when necessary:

```ts
const value =
  await locator.evaluate(
    element => element.textContent
  );
```

Use this sparingly.

For normal interactions, prefer Playwright APIs.

Avoid:

```ts
await locator.evaluate(
  element => {
    (element as HTMLElement).click();
  }
);
```

Prefer:

```ts
await locator.click();
```

---

## 27. Locator Strategy for Modern Frameworks

For React, Angular, Vue, and similar applications, avoid selectors based on:

```text
Generated classes
nth-child
Temporary IDs
Framework internals
DOM position
```

Prefer:

```text
Roles
Labels
Stable text
Test IDs
Stable business attributes
```

---

## 28. Locator Strategy for Angular and Material UI

Angular and component libraries may generate complex DOM structures and classes.

Prefer:

```ts
getByRole()
getByLabel()
getByText()
getByTestId()
```

instead of generated CSS classes.

---

## 29. Search Results

Example:

```ts
const result =
  page.getByRole('link')
    .filter({
      hasText: 'Playwright'
    });

await expect(result)
  .toBeVisible();
```

Scope the search to the results container when possible.

---

## 30. Cards

Example:

```ts
const card =
  page.getByRole('article')
    .filter({
      hasText: 'MacBook'
    });

await card.getByRole(
  'button',
  { name: 'Add to cart' }
).click();
```

This is more stable than selecting the second or third card.

---

## 31. Menus

Example:

```ts
await page.getByRole(
  'button',
  { name: 'Settings' }
).click();

await page.getByRole(
  'menuitem',
  { name: 'Profile' }
).click();
```

Role-based menu locators are clear and maintainable.

---

## 32. Toast Messages

Example:

```ts
await expect(
  page.getByRole('status')
).toContainText(
  'Saved successfully'
);
```

Use accessible semantics when available.

---

## 33. Loading States

Instead of:

```ts
await page.waitForTimeout(3000);
```

prefer:

```ts
await expect(
  page.getByText('Loading...')
).toBeHidden();

await expect(
  page.getByRole('heading', {
    name: 'Dashboard'
  })
).toBeVisible();
```

Wait for meaningful application state.

---

## 34. Date Picker Locators

Scope the calendar:

```ts
const calendar =
  page.getByRole('dialog');

await calendar.getByRole(
  'button',
  { name: '15' }
).click();
```

If multiple dates match, identify the correct month or calendar first.

---

## 35. Pagination Locators

Prefer:

```ts
await page.getByRole(
  'button',
  { name: 'Next page' }
).click();
```

Avoid:

```ts
page.locator('button').nth(8);
```

unless the position is genuinely part of the requirement.

---

## 36. File Upload

Use the file input:

```ts
await page
  .getByLabel('Upload document')
  .setInputFiles(
    'test-data/document.pdf'
  );
```

Avoid manipulating the DOM directly to trigger upload behavior.

---

## 37. Locator Debugging

Useful techniques:

```bash
npx playwright test --debug
```

Also validate uniqueness:

```ts
await expect(locator)
  .toHaveCount(1);
```

When strict mode fails, inspect why multiple elements match before changing the selector.

---

## 38. Locator Debugging Checklist

When a locator fails:

```text
1. Does the element exist?
2. Is the role correct?
3. Is the accessible name correct?
4. Are multiple elements matching?
5. Is it inside a dialog or component?
6. Is the text dynamic?
7. Is a generated class being used?
8. Can the locator be scoped?
9. Is a stable test ID available?
10. Is the application still loading?
```

---

## 39. Enterprise Locator Contract

A mature application/testing team can agree on:

```text
data-testid
data-qa
aria-label
Accessible roles
Stable business attributes
```

Example:

```html
<button
  data-testid="expense-submit">
  Submit
</button>
```

Then:

```ts
page.getByTestId(
  'expense-submit'
);
```

---

## 40. Test ID Naming Convention

Good:

```text
login-submit
expense-submit
user-search
user-edit
user-delete
report-export
```

Avoid:

```text
button1
btn2
element3
test123
```

The identifier should describe purpose.

---

## 41. Locator Decision Tree

Use:

```text
Can the element be identified by role?
        ↓
      Yes → getByRole()

No
 ↓
Can it be identified by label?
 ↓
Yes → getByLabel()

No
 ↓
Can stable text identify it?
 ↓
Yes → getByText()

No
 ↓
Is there a stable test ID?
 ↓
Yes → getByTestId()

No
 ↓
Can the component be scoped?
 ↓
Yes → filter()/chaining

No
 ↓
Use a stable locator()
```

---

## 42. Locator Anti-Patterns

Avoid:

```ts
page.locator(
  'div:nth-child(3) button'
);
```

Avoid:

```ts
page.locator(
  '.css-1x8abc'
);
```

Avoid:

```ts
page.locator(
  '//*[@id="app"]/div[2]/div[1]'
);
```

Avoid blindly using:

```ts
page.getByText('Delete').first();
```

when multiple matches indicate a locator design problem.

---

## 43. Locator Refactoring

Bad:

```ts
await page.locator(
  'div:nth-child(3) button'
).click();
```

Better:

```ts
const userRow =
  page.getByRole(
    'row',
    { name: 'Sandeep' }
  );

await userRow.getByRole(
  'button',
  { name: 'Delete' }
).click();
```

This expresses the business target instead of the DOM position.

---

## 44. Stable Locator Checklist

Prefer:

```text
Accessible role
Accessible name
Label
Stable test ID
Stable business attribute
Meaningful text
```

Avoid:

```text
Generated CSS classes
Long XPath
DOM position
Temporary IDs
Unstable text
```

---

## 45. Production-Quality Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('edit employee', async ({
  page
}) => {
  await page.goto('/admin/users');

  const employeeRow =
    page.getByRole('row')
      .filter({
        hasText: 'Sandeep'
      });

  await expect(
    employeeRow
  ).toBeVisible();

  await employeeRow
    .getByRole(
      'button',
      { name: 'Edit' }
    )
    .click();

  const dialog =
    page.getByRole('dialog');

  await expect(dialog)
    .toBeVisible();

  await dialog
    .getByLabel('Email')
    .fill(
      'sandeep@example.com'
    );

  await dialog
    .getByRole(
      'button',
      { name: 'Save' }
    )
    .click();

  await expect(
    page.getByRole('status')
  ).toContainText(
    'updated successfully'
  );
});
```

---

## 46. Interview Questions

### Q1. What is a locator in Playwright?

A locator identifies page elements and provides Playwright's mechanism for interacting with and asserting against them.

### Q2. What is the recommended locator strategy?

Prefer user-facing and accessible locators such as `getByRole()` and `getByLabel()`, followed by stable test IDs when necessary.

### Q3. What is strict mode?

Strictness prevents single-element actions from silently operating on multiple matching elements.

### Q4. Why is strict mode useful?

It catches ambiguous locators and prevents accidental interaction with the wrong element.

### Q5. Why should `.first()` not be used blindly?

It can hide the fact that the locator is ambiguous.

### Q6. How do you locate a button inside a specific table row?

```ts
const row = page
  .getByRole('row')
  .filter({
    hasText: 'Sandeep'
  });

await row.getByRole(
  'button',
  { name: 'Edit' }
).click();
```

### Q7. What is locator chaining?

It narrows a locator by searching within an already identified element or component.

### Q8. What does `.filter({ hasText })` do?

It narrows a locator to elements containing specified text.

### Q9. When should test IDs be used?

When semantic locators are insufficient or a stable automation contract is needed.

### Q10. What is a major locator anti-pattern?

Using brittle selectors based on DOM position, generated classes, or long XPath instead of stable semantic identifiers.

---

## 47. Hands-On Exercises

### Exercise 1 — Role Locators

Replace CSS selectors with:

```text
getByRole()
```

where possible.

### Exercise 2 — Form Locators

Replace input selectors with:

```text
getByLabel()
```

and verify the fields.

### Exercise 3 — Strict Mode

Create duplicate buttons and fix the strict mode violation without blindly using `.first()`.

### Exercise 4 — Table Filtering

Locate a specific user row and click its Edit button.

### Exercise 5 — Dialog Scoping

Locate a modal and interact only with controls inside it.

### Exercise 6 — Dynamic Cards

Locate a product card by product name and click its action button.

### Exercise 7 — Test IDs

Add stable test IDs to a sample form and automate it.

### Exercise 8 — Locator Factory

Create:

```ts
userRow(name: string)
```

for dynamic user tables.

### Exercise 9 — Component Object

Create a reusable table component containing:

```text
row()
editButton()
deleteButton()
search()
```

### Exercise 10 — Refactor Brittle Locators

Take selectors based on:

```text
nth-child
generated classes
long XPath
```

and replace them with stable locators.

---

## 48. Key Takeaways

- Locators are fundamental to Playwright automation.
- Prefer user-facing and accessible locators.
- `getByRole()` is an excellent default for many UI elements.
- `getByLabel()` is ideal for form controls.
- `getByTestId()` provides a stable automation contract when needed.
- Strict mode helps detect ambiguous selectors.
- Avoid blindly using `.first()` or `.nth()`.
- Use filtering and chaining for dynamic components.
- Scope locators to tables, dialogs, cards, and other components.
- Avoid generated CSS classes and long XPath.
- Locator factories are useful for dynamic business data.
- Page Objects and Component Objects can centralize locator logic.
- Stable locator design improves test reliability and maintainability.
- Locator strategy should be consistent across an enterprise test framework.

---

## 49. Enterprise Locator Challenge

Build a reusable locator architecture for an enterprise application containing:

```text
Login
Dashboard
Admin
User Management
Tables
Dialogs
Reports
Search
Pagination
Notifications
```

Requirements:

1. Use semantic locators wherever possible.
2. Add stable test IDs where semantic locators are insufficient.
3. Avoid long XPath.
4. Avoid generated CSS classes.
5. Handle duplicate elements with scoped locators.
6. Create dynamic locator factories.
7. Build reusable component objects.
8. Verify strict-mode safety.
9. Use web-first assertions.
10. Document the team's locator conventions.

---

## 50. Next Step

**Step 87 — Mastering Playwright Frames & iFrames, Frame Locators, Nested Frames, Cross-Origin Frames & Enterprise Frame Automation**

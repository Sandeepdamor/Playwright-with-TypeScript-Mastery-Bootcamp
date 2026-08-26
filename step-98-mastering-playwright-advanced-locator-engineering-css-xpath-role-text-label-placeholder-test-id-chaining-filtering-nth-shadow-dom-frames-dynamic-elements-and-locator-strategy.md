# Playwright with TypeScript Mastery Bootcamp

# Step 98 — Mastering Playwright Advanced Locator Engineering: CSS, XPath, Role, Text, Label, Placeholder, Test ID, Chaining, Filtering, nth(), Shadow DOM, Frames, Dynamic Elements & Locator Strategy

## 1. Introduction

Locators are one of the foundations of reliable Playwright automation.

A locator tells Playwright:

```text
Which element should I interact with?
```

For example:

```ts
page.getByRole(
  'button',
  {
    name: 'Login'
  }
);
```

A strong locator should be:

```text
Readable
Stable
Unique
User-facing
Maintainable
Resistant to UI changes
```

A weak locator can make an otherwise good test suite flaky.

Example of a fragile locator:

```ts
page.locator(
  'div:nth-child(4) > div:nth-child(2) > button'
);
```

A stronger locator might be:

```ts
page.getByRole(
  'button',
  {
    name: 'Save'
  }
);
```

This step covers locator design from basic selectors through enterprise locator strategy.

---

# 2. Why Locator Engineering Matters

Suppose a test uses:

```ts
page.locator(
  '#submit-btn'
);
```

If the developer changes the ID:

```text
submit-btn
```

to:

```text
save-button
```

the test fails.

But if the user-facing button is still:

```text
Save
```

then:

```ts
page.getByRole(
  'button',
  {
    name: 'Save'
  }
);
```

may continue to work.

The goal is to locate elements based on stable application behavior rather than implementation details whenever possible.

---

# 3. Locator Priority Strategy

A useful locator preference order is:

```text
1. getByRole()
2. getByLabel()
3. getByPlaceholder()
4. getByText()
5. getByTestId()
6. CSS
7. XPath
```

This is not an absolute rule.

The best locator is the one that is:

```text
Stable
Specific
Readable
Semantically meaningful
```

---

# 4. `getByRole()`

`getByRole()` locates elements through accessible roles.

Example:

```ts
page.getByRole(
  'button',
  {
    name: 'Login'
  }
);
```

Common roles include:

```text
button
link
heading
textbox
checkbox
radio
list
listitem
table
row
cell
dialog
navigation
```

---

# 5. Why `getByRole()` Is Powerful

It represents how users and accessibility tools understand the interface.

Example HTML:

```html
<button>Login</button>
```

Locator:

```ts
page.getByRole(
  'button',
  {
    name: 'Login'
  }
);
```

This is usually better than:

```ts
page.locator(
  'button.login-btn'
);
```

---

# 6. Accessible Name

The `name` in:

```ts
getByRole(
  'button',
  {
    name: 'Save'
  }
)
```

refers to the accessible name.

It may come from:

```text
Visible text
aria-label
aria-labelledby
Associated labels
Other accessibility semantics
```

Understanding accessible names is important for locator debugging.

---

# 7. Role + Exact Name

Example:

```ts
page.getByRole(
  'button',
  {
    name: 'Save',
    exact: true
  }
);
```

This helps when the page contains:

```text
Save
Save Changes
Save Draft
```

---

# 8. Role with Regular Expression

Example:

```ts
page.getByRole(
  'button',
  {
    name:
      /save/i
  }
);
```

This can match case-insensitively.

Use regex when text variation is expected.

Avoid overly broad patterns.

---

# 9. Heading Locator

Example:

```ts
page.getByRole(
  'heading',
  {
    name: 'Dashboard'
  }
);
```

This is stronger than:

```ts
page.locator('h1');
```

when the page contains multiple headings.

---

# 10. Link Locator

Example:

```ts
page.getByRole(
  'link',
  {
    name: 'Admin'
  }
);
```

This communicates the purpose of the element clearly.

---

# 11. Textbox Locator

Example:

```ts
page.getByRole(
  'textbox',
  {
    name: 'Username'
  }
);
```

If the form has proper accessibility labels, this is a strong choice.

---

# 12. `getByLabel()`

Use:

```ts
page.getByLabel(
  'Username'
);
```

Example HTML:

```html
<label for="username">
  Username
</label>

<input
  id="username"
  type="text"
/>
```

The label connects the visible field name with the input.

---

# 13. Label-Based Form Testing

Example:

```ts
await page
  .getByLabel(
    'Username'
  )
  .fill(
    'Admin'
  );

await page
  .getByLabel(
    'Password'
  )
  .fill(
    'password'
  );
```

This is readable and close to the user's experience.

---

# 14. `getByPlaceholder()`

Example:

```ts
page.getByPlaceholder(
  'Username'
);
```

Useful when the input has:

```html
<input
  placeholder="Username"
/>
```

This is common in applications where explicit labels are missing.

---

# 15. Label vs Placeholder

Prefer:

```text
Label
```

when a real accessible label exists.

Use:

```text
Placeholder
```

when placeholder text is the most stable meaningful identifier available.

Remember that placeholders can change for UX reasons.

---

# 16. `getByText()`

Example:

```ts
page.getByText(
  'Welcome'
);
```

Useful for:

```text
Messages
Labels
Cards
Status text
Notifications
Static content
```

---

# 17. Exact Text

Example:

```ts
page.getByText(
  'Admin',
  {
    exact: true
  }
);
```

This helps avoid matching:

```text
Admin
Administrator
Admin Panel
```

---

# 18. Text with Regex

Example:

```ts
page.getByText(
  /welcome/i
);
```

Useful when capitalization or a small amount of dynamic text varies.

---

# 19. Avoid Overly Generic Text

Weak:

```ts
page.getByText(
  'User'
);
```

This might match:

```text
User
User Management
Add User
User Role
```

Use additional context or filtering.

---

# 20. `getByTestId()`

Example:

```ts
page.getByTestId(
  'submit-button'
);
```

HTML:

```html
<button
  data-testid="submit-button"
>
  Save
</button>
```

Test IDs are useful when user-facing semantics are not enough.

---

# 21. When to Use Test IDs

Use test IDs when:

```text
Element has no stable accessible name
Text is highly dynamic
Multiple visually identical controls exist
UI implementation makes semantic targeting difficult
Team has an agreed testing contract
```

Test IDs should be intentionally maintained.

---

# 22. Test ID Naming

Good:

```text
login-submit
user-search
expense-save
profile-menu
```

Weak:

```text
btn1
test123
element-x
abc
```

Use meaningful names.

---

# 23. Custom Test ID Attribute

Playwright can be configured to use another attribute.

Example concept:

```ts
use: {
  testIdAttribute:
    'data-pw'
}
```

HTML:

```html
<button
  data-pw="save-user"
>
  Save
</button>
```

Then:

```ts
page.getByTestId(
  'save-user'
);
```

---

# 24. CSS Locators

Example:

```ts
page.locator(
  '.login-button'
);
```

CSS selectors are useful when semantic locators are not practical.

Examples:

```text
#id
.class
tag
attribute
descendant
child
```

---

# 25. CSS ID Selector

Example:

```ts
page.locator(
  '#username'
);
```

Good when the ID is stable and intentionally maintained.

Avoid generated IDs such as:

```text
input-847392
```

---

# 26. CSS Class Selector

Example:

```ts
page.locator(
  '.user-card'
);
```

Be careful with classes used only for styling.

A redesign may change them without changing application behavior.

---

# 27. CSS Attribute Selector

Example:

```ts
page.locator(
  '[name="username"]'
);
```

This can be stable when the attribute is part of the form contract.

---

# 28. CSS Combination

Example:

```ts
page.locator(
  'input[name="username"]'
);
```

This is more specific than:

```ts
page.locator(
  'input'
);
```

---

# 29. CSS Descendant Selector

Example:

```ts
page.locator(
  '.login-form button'
);
```

Use carefully because DOM restructuring can break descendant selectors.

---

# 30. Avoid Deep CSS Chains

Fragile:

```ts
page.locator(
  '#app > div > div:nth-child(2) > form > div:nth-child(3) > button'
);
```

This depends heavily on DOM structure.

Prefer semantic locators or shorter stable selectors.

---

# 31. XPath

Example:

```ts
page.locator(
  '//button[text()="Login"]'
);
```

XPath can locate elements through complex DOM relationships.

However, it is often less readable than Playwright's user-facing locators.

---

# 32. XPath Example

```ts
page.locator(
  '//input[@name="username"]'
);
```

Equivalent CSS might be:

```ts
page.locator(
  'input[name="username"]'
);
```

Prefer the simpler form when both are equally reliable.

---

# 33. When XPath Can Be Useful

XPath may help when:

```text
Complex ancestor relationships are required
Legacy application markup is difficult
No stable semantic locator exists
```

Still avoid extremely long XPath expressions.

---

# 34. Absolute XPath

Avoid:

```text
/html/body/div[2]/div[1]/form/div[3]/button
```

This is very fragile.

Small DOM changes can break it.

---

# 35. Relative XPath

Better than absolute XPath:

```text
//form[@id="login-form"]//button[@type="submit"]
```

But if possible, a semantic locator may still be clearer.

---

# 36. Locator Chaining

Locators can be scoped.

Example:

```ts
const card =
  page.getByRole(
    'article'
  );

const button =
  card.getByRole(
    'button',
    {
      name: 'Edit'
    }
  );
```

This means:

```text
Find article
 ↓
Find Edit button inside it
```

---

# 37. Chaining Example

Suppose there are several user cards.

```ts
const userCard =
  page
    .getByTestId(
      'user-card'
    )
    .filter({
      hasText:
        'Automation User'
    });

await userCard
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .click();
```

This is much safer than choosing an arbitrary button.

---

# 38. `filter()`

Example:

```ts
page
  .getByRole(
    'row'
  )
  .filter({
    hasText:
      'Sandeep'
  });
```

This finds the row containing the required text.

---

# 39. `filter({ hasText })`

Example:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        'Admin'
    });
```

Then:

```ts
await row
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .click();
```

---

# 40. `filter({ has })`

You can filter using another locator.

Example:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      has:
        page.getByText(
          'Automation User'
        )
    });
```

This is useful for structured components.

---

# 41. `filter({ hasNotText })`

Example concept:

```ts
page
  .getByRole(
    'row'
  )
  .filter({
    hasNotText:
      'Disabled'
  });
```

Useful when excluding certain states.

---

# 42. Locator Lists

A locator may match multiple elements.

Example:

```ts
const rows =
  page.getByRole(
    'row'
  );
```

You can inspect:

```ts
const count =
  await rows.count();
```

---

# 43. `count()`

Example:

```ts
const count =
  await page
    .getByRole(
      'row'
    )
    .count();

console.log(
  count
);
```

For assertions, prefer:

```ts
await expect(
  page.getByRole(
    'row'
  )
).toHaveCount(5);
```

---

# 44. `first()`

Example:

```ts
page
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .first();
```

Use only when selecting the first element is actually part of the intended behavior.

---

# 45. `last()`

Example:

```ts
page
  .getByRole(
    'row'
  )
  .last();
```

Again, use only when position is meaningful.

---

# 46. `nth()`

Example:

```ts
page
  .getByRole(
    'row'
  )
  .nth(2);
```

Indexes are zero-based.

Conceptually:

```text
nth(0) → first
nth(1) → second
nth(2) → third
```

---

# 47. Risks of `nth()`

Suppose:

```text
User A
User B
User C
```

and the test uses:

```ts
rows.nth(1);
```

If sorting changes:

```text
User C
User A
User B
```

the test now targets a different user.

Prefer content-based filtering when possible.

---

# 48. Good Use of `nth()`

`nth()` can be appropriate when position itself is the requirement.

Example:

```text
Verify the third carousel item
```

Then:

```ts
items.nth(2);
```

is meaningful.

---

# 49. Strictness

Playwright locators are strict for operations that expect one element.

Example:

```ts
await page
  .getByRole(
    'button'
  )
  .click();
```

If 10 buttons match, Playwright may report a strict-mode violation.

This protects the test from ambiguous behavior.

---

# 50. Strict Mode Violation

Example error:

```text
strict mode violation:
locator resolved to 2 elements
```

Do not immediately fix this with:

```ts
.first()
```

Instead ask:

```text
Which element should the test actually target?
```

Then improve the locator.

---

# 51. Locator Auto-Waiting

Playwright locators work with auto-waiting.

Before actions, Playwright checks conditions such as:

```text
Element attached
Visible
Stable
Receives events
Enabled
```

depending on the action.

This reduces the need for manual waits.

---

# 52. Locator Re-Evaluation

A locator represents a way to find an element.

Example:

```ts
const button =
  page.getByRole(
    'button',
    {
      name: 'Save'
    }
  );
```

When an action is performed, Playwright resolves the locator against the current DOM.

This helps with dynamic applications.

---

# 53. Locator vs Element Handle

Prefer locators for most test automation.

Locators provide:

```text
Auto-waiting
Retryability
Re-evaluation
Strictness
Better assertions
```

Element handles are lower-level and are less commonly needed for normal tests.

---

# 54. Dynamic Elements

Modern applications may generate:

```text
Dynamic IDs
Dynamic text
Dynamic rows
Dynamic components
Lazy-loaded content
```

Avoid depending on values that change every run.

---

# 55. Dynamic ID Example

Fragile:

```ts
page.locator(
  '#user-739284'
);
```

Better:

```ts
page
  .getByRole(
    'row'
  )
  .filter({
    hasText:
      'Automation User'
  });
```

---

# 56. Dynamic Text

Suppose:

```text
Welcome, Sandeep
Welcome, Rahul
Welcome, Priya
```

Use a regex if the username varies:

```ts
page.getByText(
  /^Welcome,/
);
```

Or use known test data:

```ts
page.getByText(
  `Welcome, ${user.name}`
);
```

---

# 57. Dynamic Tables

Example table:

```text
Name             Role       Status
Automation User  Admin      Active
Test Employee    Employee   Active
```

Locate the row:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        'Automation User'
    });
```

Then target cells/actions inside the row.

---

# 58. Table Cell Locator

Example:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        'Automation User'
    });

await expect(
  row.getByRole(
    'cell',
    {
      name: 'Admin'
    }
  )
).toBeVisible();
```

This is readable and scoped.

---

# 59. Dynamic List Items

Example:

```ts
const item =
  page
    .getByRole(
      'listitem'
    )
    .filter({
      hasText:
        'Playwright'
    });
```

Then:

```ts
await item
  .getByRole(
    'button',
    {
      name: 'Remove'
    }
  )
  .click();
```

---

# 60. Locating Parent Components

Instead of locating a button globally:

```ts
page.getByRole(
  'button',
  {
    name: 'Delete'
  }
);
```

first locate the relevant component:

```ts
const card =
  page
    .getByTestId(
      'project-card'
    )
    .filter({
      hasText:
        'Automation Project'
    });
```

Then:

```ts
card.getByRole(
  'button',
  {
    name: 'Delete'
  }
);
```

---

# 61. Shadow DOM

Web components may use Shadow DOM.

Conceptually:

```text
Page
 ↓
Custom Element
 ↓
Shadow Root
 ↓
Button
```

Playwright locators can work with open Shadow DOM in many common locator scenarios.

---

# 62. Shadow DOM Example

Suppose:

```html
<user-card>
  #shadow-root
    <button>Delete</button>
</user-card>
```

A Playwright locator may locate the button through the component using supported locator behavior.

For example:

```ts
page
  .locator(
    'user-card'
  )
  .getByRole(
    'button',
    {
      name: 'Delete'
    }
  );
```

---

# 63. Shadow DOM Limitation

Closed Shadow DOM cannot generally be inspected in the same way as open Shadow DOM.

If the application uses closed roots, coordinate with developers on testability.

---

# 64. Frames

An iframe has its own document.

Example:

```html
<iframe
  id="payment-frame"
></iframe>
```

Normal page locators do not directly target content inside a different frame document.

Use:

```ts
page.frameLocator(
  '#payment-frame'
);
```

---

# 65. `frameLocator()`

Example:

```ts
const paymentFrame =
  page.frameLocator(
    '#payment-frame'
  );

await paymentFrame
  .getByLabel(
    'Card Number'
  )
  .fill(
    '4111111111111111'
  );
```

Use synthetic test payment data only.

---

# 66. Frame Button

Example:

```ts
await page
  .frameLocator(
    '#payment-frame'
  )
  .getByRole(
    'button',
    {
      name: 'Pay'
    }
  )
  .click();
```

---

# 67. Nested Frames

Some applications contain:

```text
Page
 ↓
Frame A
 ↓
Frame B
```

Use frame locators carefully to scope into the correct frame hierarchy.

Keep frame interactions encapsulated in Page Objects when possible.

---

# 68. Frame Locator Strategy

For a stable frame:

```ts
readonly paymentFrame:
  FrameLocator;
```

can be stored in a Page Object.

Example:

```ts
this.paymentFrame =
  page.frameLocator(
    '#payment-frame'
  );
```

---

# 69. Locator Assertions

Playwright assertions work directly with locators.

Examples:

```ts
await expect(
  locator
).toBeVisible();

await expect(
  locator
).toHaveText(
  'Dashboard'
);

await expect(
  locator
).toBeEnabled();

await expect(
  locator
).toHaveCount(5);
```

These assertions automatically retry until timeout.

---

# 70. `toHaveText()`

Example:

```ts
await expect(
  page.getByRole(
    'heading'
  )
).toHaveText(
  'Dashboard'
);
```

Use exact expectations when the full text is known.

---

# 71. `toContainText()`

Example:

```ts
await expect(
  page.getByRole(
    'alert'
  )
).toContainText(
  'Invalid credentials'
);
```

Useful when the element contains additional text.

---

# 72. `toHaveValue()`

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

Useful for form validation.

---

# 73. `toBeChecked()`

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

# 74. `toBeDisabled()`

Example:

```ts
await expect(
  page.getByRole(
    'button',
    {
      name: 'Submit'
    }
  )
).toBeDisabled();
```

This validates UI state.

---

# 75. `toHaveAttribute()`

Example:

```ts
await expect(
  page.getByRole(
    'link',
    {
      name: 'Profile'
    }
  )
).toHaveAttribute(
  'href',
  '/profile'
);
```

Use attribute assertions when the attribute is part of the behavior being validated.

---

# 76. Locator Variables

Good:

```ts
const saveButton =
  page.getByRole(
    'button',
    {
      name: 'Save'
    }
  );
```

Then:

```ts
await saveButton.click();
```

Meaningful names improve readability.

---

# 77. Page Object Locators

Example:

```ts
export class LoginPage {

  readonly usernameInput:
    Locator;

  readonly passwordInput:
    Locator;

  readonly loginButton:
    Locator;

  constructor(
    private readonly page:
      Page
  ) {

    this.usernameInput =
      page.getByPlaceholder(
        'Username'
      );

    this.passwordInput =
      page.getByPlaceholder(
        'Password'
      );

    this.loginButton =
      page.getByRole(
        'button',
        {
          name: 'Login'
        }
      );
  }
}
```

---

# 78. Locator Encapsulation

Tests should not repeatedly define:

```ts
page.locator(
  '...'
);
```

for the same application component.

Keep stable reusable locators in Page Objects or component objects.

---

# 79. Component Objects

If a UI component appears across pages:

```text
Header
Sidebar
Navigation
Modal
Data Table
Toast
```

create a reusable component object.

Example:

```ts
export class Sidebar {

  constructor(
    private readonly page:
      Page
  ) {}

  adminLink() {
    return this.page
      .getByRole(
        'link',
        {
          name: 'Admin'
        }
      );
  }
}
```

---

# 80. Locator Method vs Property

Static locator:

```ts
readonly saveButton:
  Locator;
```

Dynamic locator:

```ts
userRow(
  username: string
) {
  return this.page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        username
    });
}
```

Use methods for locators that depend on runtime data.

---

# 81. Dynamic Locator Method

Example:

```ts
userRow(
  username: string
): Locator {

  return this.page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        username
    });
}
```

Then:

```ts
await expect(
  adminPage.userRow(
    'Automation User'
  )
).toBeVisible();
```

---

# 82. Locator Duplication

Avoid repeating:

```ts
page.getByRole(
  'button',
  {
    name: 'Add'
  }
);
```

across many tests if it represents the same application action.

Centralize it where appropriate.

---

# 83. Over-Abstraction

Do not create a locator helper for every selector.

Bad:

```ts
getButtonByText(
  text
)
```

when Playwright already provides:

```ts
page.getByRole(
  'button',
  {
    name: text
  }
);
```

Abstraction should add domain meaning.

---

# 84. Domain-Oriented Locator

Better:

```ts
adminPage.addUserButton
```

than:

```ts
locatorHelper
  .getButtonByText(
    'Add'
  );
```

The first expresses business intent.

---

# 85. Locator Resilience

A locator is resilient when it survives:

```text
CSS redesign
DOM wrapper changes
Class renaming
Layout changes
Minor visual changes
```

Semantic locators often provide better resilience.

---

# 86. Accessibility and Testing

Using:

```text
getByRole()
getByLabel()
```

encourages accessible application markup.

If a button cannot be located by a meaningful role/name, that may indicate an accessibility problem.

Testing and accessibility can support each other.

---

# 87. Locator Performance

Avoid unnecessarily broad selectors such as:

```ts
page.locator('*');
```

or repeated searches across huge DOM trees.

Scope locators to relevant containers.

Example:

```ts
const table =
  page.getByRole(
    'table'
  );

const row =
  table
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        'Automation User'
    });
```

---

# 88. Locator Scope

Instead of:

```ts
page.getByText(
  'Active'
);
```

use:

```ts
userRow.getByText(
  'Active',
  {
    exact: true
  }
);
```

This reduces ambiguity.

---

# 89. Locator Debugging

When a locator fails, check:

```text
1. Is the page correct?
2. Is the element present?
3. Is it visible?
4. Does the role match?
5. Is the accessible name correct?
6. Does it match multiple elements?
7. Is it inside an iframe?
8. Is it inside Shadow DOM?
9. Is the content dynamic?
10. Did navigation/authentication fail?
```

---

# 90. Inspect Locator Count

Example:

```ts
const locator =
  page.getByRole(
    'button',
    {
      name: 'Save'
    }
  );

console.log(
  await locator.count()
);
```

This helps identify:

```text
0 matches
1 match
Multiple matches
```

---

# 91. Playwright Inspector

Playwright Inspector helps with:

```text
Locator exploration
Step debugging
Page inspection
Action debugging
```

Run:

```bash
npx playwright test \
  --debug
```

for interactive debugging.

---

# 92. Locator Picker

Playwright tooling can suggest locators for elements.

Treat generated locators as a starting point.

Review them for:

```text
Readability
Stability
Semantics
Uniqueness
```

Do not blindly accept generated selectors.

---

# 93. Codegen

Playwright code generation can help discover locators.

Example:

```bash
npx playwright codegen \
  https://example.test
```

Generated code should be reviewed and refactored before becoming production test code.

---

# 94. Generated Locator Review

Suppose codegen creates:

```ts
page.getByRole(
  'button',
  {
    name: 'Save'
  }
);
```

Good.

If it generates a fragile implementation-based selector, consider replacing it with a stronger locator.

---

# 95. Locator Strategy for Forms

Recommended:

```text
Input
 ↓
getByLabel()

Button
 ↓
getByRole()

Checkbox
 ↓
getByRole('checkbox')

Radio
 ↓
getByRole('radio')
```

Fallback:

```text
getByPlaceholder()
getByTestId()
```

---

# 96. Locator Strategy for Tables

Recommended:

```text
Table
 ↓
Row
 ↓
Filter by unique data
 ↓
Cell / Action
```

Example:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        user.email
    });

await row
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .click();
```

---

# 97. Locator Strategy for Cards

Recommended:

```text
Card collection
 ↓
Filter by card title
 ↓
Find action inside card
```

Example:

```ts
const card =
  page
    .getByTestId(
      'project-card'
    )
    .filter({
      hasText:
        'Playwright Project'
    });

await card
  .getByRole(
    'button',
    {
      name: 'Open'
    }
  )
  .click();
```

---

# 98. Locator Strategy for Menus

Example:

```ts
await page
  .getByRole(
    'button',
    {
      name:
        'Profile'
    }
  )
  .click();

await page
  .getByRole(
    'menuitem',
    {
      name:
        'Logout'
    }
  )
  .click();
```

Use the actual roles exposed by the application.

---

# 99. Locator Strategy for Modals

Example:

```ts
const dialog =
  page.getByRole(
    'dialog'
  );

await expect(
  dialog
).toBeVisible();

await dialog
  .getByRole(
    'button',
    {
      name:
        'Confirm'
    }
  )
  .click();
```

Scope actions to the dialog.

---

# 100. Locator Strategy for Toasts

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

If the application does not expose an appropriate role, use a stable test ID or scoped text locator.

---

# 101. Locator Strategy for Dropdowns

Native select:

```ts
await page
  .getByLabel(
    'Role'
  )
  .selectOption(
    'admin'
  );
```

Custom dropdowns may require:

```text
Open combobox
 ↓
Locate option
 ↓
Select option
```

Example:

```ts
await page
  .getByRole(
    'combobox',
    {
      name: 'Role'
    }
  )
  .click();

await page
  .getByRole(
    'option',
    {
      name: 'Admin'
    }
  )
  .click();
```

---

# 102. Locator Strategy for Autocomplete

Typical flow:

```text
Textbox
 ↓
Type search
 ↓
Wait for suggestions
 ↓
Choose option
```

Example:

```ts
const employeeInput =
  page.getByRole(
    'textbox',
    {
      name:
        'Employee Name'
    }
  );

await employeeInput.fill(
  'John'
);

await page
  .getByRole(
    'option',
    {
      name:
        /John/
    }
  )
  .click();
```

---

# 103. Locator Strategy for Repeated Buttons

Suppose every row has:

```text
Edit
Delete
```

Do not use:

```ts
page
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .nth(3);
```

Prefer:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        'Automation User'
    });

await row
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .click();
```

---

# 104. Locator Strategy for Dynamic IDs

If HTML is:

```html
<input
  id="input-173892"
/>
```

avoid the generated ID.

Look for:

```text
Label
Name
Placeholder
Role
Test ID
Stable parent
```

---

# 105. Locator Strategy for Icons

An icon-only button should ideally expose an accessible name.

Example:

```html
<button
  aria-label="Delete user"
>
  ...
</button>
```

Then:

```ts
page.getByRole(
  'button',
  {
    name:
      'Delete user'
  }
);
```

This is better than locating the SVG path.

---

# 106. Avoid SVG Path Selectors

Fragile:

```ts
page.locator(
  'svg path[d="M10..."]'
);
```

The icon implementation can change easily.

Target the interactive control containing the icon.

---

# 107. Avoid Styling Classes

Fragile:

```ts
page.locator(
  '.btn-primary'
);
```

If the design changes from primary to secondary, the test fails even if the behavior is unchanged.

Prefer:

```ts
page.getByRole(
  'button',
  {
    name: 'Submit'
  }
);
```

---

# 108. Avoid Text When Text Is Unstable

If the product supports many languages, a text locator may vary:

```text
Save
Guardar
Enregistrer
Speichern
```

In such cases, stable test IDs may be more appropriate for tests that are not specifically validating localization.

---

# 109. Localization Testing

When testing localization itself, user-facing text is the behavior.

Example:

```ts
await expect(
  page.getByRole(
    'button',
    {
      name:
        'Guardar'
    }
  )
).toBeVisible();
```

Choose locator strategy based on test purpose.

---

# 110. Locator Contracts with Developers

Teams can agree on:

```text
Accessible labels
Stable roles
data-testid
Component conventions
Unique names
```

This reduces automation maintenance.

Testing should be considered during UI development.

---

# 111. Enterprise Locator Standards

A team may define:

```text
Priority 1 → Role
Priority 2 → Label
Priority 3 → Test ID
Priority 4 → Stable CSS
Priority 5 → XPath
```

The exact standard can vary.

Document it and apply it consistently.

---

# 112. Locator Review Checklist

Before accepting a locator:

```text
[ ] Is it unique?
[ ] Is it readable?
[ ] Is it stable?
[ ] Does it express user intent?
[ ] Is it scoped correctly?
[ ] Does it avoid dynamic IDs?
[ ] Does it avoid styling classes?
[ ] Does it avoid unnecessary nth()?
[ ] Does it avoid deep DOM chains?
[ ] Does it work with auto-waiting?
```

---

# 113. Locator Anti-Patterns

Avoid:

```text
Absolute XPath
Deep CSS chains
Generated IDs
Styling classes
Unnecessary nth()
Global text matches
Force clicks used to hide locator issues
Manual sleeps for locator timing
Duplicated selectors across tests
Huge generic locator helpers
```

---

# 114. Production Page Object Example

```ts
import {
  expect,
  Locator,
  Page
} from '@playwright/test';

export class AdminPage {

  readonly page:
    Page;

  readonly heading:
    Locator;

  readonly addButton:
    Locator;

  readonly usernameInput:
    Locator;

  constructor(
    page: Page
  ) {

    this.page =
      page;

    this.heading =
      page.getByRole(
        'heading',
        {
          name: 'Admin',
          exact: true
        }
      );

    this.addButton =
      page.getByRole(
        'button',
        {
          name: 'Add'
        }
      );

    this.usernameInput =
      page.getByLabel(
        'Username'
      );
  }

  userRow(
    username: string
  ): Locator {

    return this.page
      .getByRole(
        'row'
      )
      .filter({
        hasText:
          username
      });
  }

  async verifyLoaded() {

    await expect(
      this.heading
    ).toBeVisible();
  }

  async editUser(
    username: string
  ) {

    const row =
      this.userRow(
        username
      );

    await row
      .getByRole(
        'button',
        {
          name: 'Edit'
        }
      )
      .click();
  }
}
```

---

# 115. Production Test Example

```ts
test(
  'admin can edit user',
  async ({
    adminPage
  }) => {

    await adminPage
      .verifyLoaded();

    await adminPage
      .editUser(
        'Automation User'
      );
  }
);
```

The test expresses business intent rather than selector details.

---

# 116. Locator Refactoring Example

Before:

```ts
await page
  .locator(
    'div:nth-child(2) > table > tbody > tr:nth-child(3) > td:nth-child(6) button'
  )
  .click();
```

After:

```ts
const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        'Automation User'
    });

await row
  .getByRole(
    'button',
    {
      name: 'Edit'
    }
  )
  .click();
```

The second version is easier to understand and maintain.

---

# 117. Locator Failure Analysis

When a locator fails, classify it:

```text
0 matches
 ↓
Wrong selector / wrong state / wrong frame

Multiple matches
 ↓
Insufficient specificity

Element exists but action fails
 ↓
Visibility / overlay / disabled / animation

Works locally only
 ↓
Timing / data / environment / browser difference
```

This makes debugging systematic.

---

# 118. Locator + Network Debugging

Sometimes the locator is not the real problem.

Example:

```text
GET /api/users
 ↓
500
 ↓
Table never renders
 ↓
Row locator times out
```

The locator error is only the final symptom.

Use trace and network diagnostics.

---

# 119. Locator + Authentication Debugging

Example:

```text
Storage state expired
 ↓
Redirect to login
 ↓
Dashboard locator missing
```

Before rewriting the dashboard locator, check the current URL and authentication state.

---

# 120. Locator + Test Data Debugging

Example:

```text
Expected user not created
 ↓
Search returns nothing
 ↓
Row locator fails
```

Verify setup data before changing the locator.

---

# 121. Locator Performance in Large Tables

For a large table, scope first:

```ts
const table =
  page.getByRole(
    'table'
  );

const row =
  table
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        email
    });
```

This communicates the intended search area.

---

# 122. Locator Design for Parallel Tests

Do not depend on shared names such as:

```text
Test User
```

if many parallel tests create the same record.

Use unique test data:

```ts
const email =
  `user-${Date.now()}@example.test`;
```

Then locate using that unique value.

For stronger parallel uniqueness, include worker/test identifiers or a UUID-based data factory.

---

# 123. Locator and Unique Test Data

A powerful strategy is:

```text
Create unique data
 ↓
Use unique value in locator
 ↓
No ambiguity
```

Example:

```ts
const user =
  await usersApi
    .createUser();

const row =
  page
    .getByRole(
      'row'
    )
    .filter({
      hasText:
        user.email
    });
```

---

# 124. Enterprise Locator Architecture

```text
                         Test
                           │
                           ▼
                       Page Object
                           │
               ┌───────────┼───────────┐
               │           │           │
              Role       Label       Test ID
               │           │           │
               └───────────┼───────────┘
                           │
                       Filtering
                           │
                       Component
                           │
                       UI Element
                           │
                    Playwright Action
                           │
                       Assertion
```

---

# 125. Locator Strategy Decision Tree

```text
Can the element be identified by role + accessible name?
        │
       Yes
        ↓
   getByRole()
        │
       No
        ↓
Does it have a meaningful label?
        │
       Yes
        ↓
   getByLabel()
        │
       No
        ↓
Is placeholder meaningful and stable?
        │
       Yes
        ↓
getByPlaceholder()
        │
       No
        ↓
Is visible text stable?
        │
       Yes
        ↓
   getByText()
        │
       No
        ↓
Is there a stable test ID?
        │
       Yes
        ↓
  getByTestId()
        │
       No
        ↓
Stable CSS / XPath as last-resort options
```

---

# 126. Interview Questions

## Q1. What is a Playwright locator?

A locator represents a strategy for finding one or more elements and provides Playwright's actionability, retry, and assertion behavior.

## Q2. Why prefer `getByRole()`?

It targets accessible semantics and usually reflects how a user understands the interface.

## Q3. What is a strict-mode violation?

It occurs when an operation expecting one element resolves to multiple matching elements.

## Q4. How do you locate a row containing a specific user?

Example:

```ts
page
  .getByRole(
    'row'
  )
  .filter({
    hasText:
      username
  });
```

## Q5. Why avoid `nth()`?

Because index-based selectors can target the wrong element when ordering changes.

## Q6. When is `nth()` acceptable?

When position itself is part of the requirement.

## Q7. When should you use `getByTestId()`?

When semantic/user-facing attributes are insufficient or unstable and the team maintains a stable test identifier.

## Q8. CSS vs XPath?

Both can locate DOM elements, but Playwright's semantic locators are usually more readable and maintainable. CSS is often simpler than XPath for implementation-based selection.

## Q9. How do you interact with an iframe?

Use a frame locator, such as:

```ts
page.frameLocator(
  '#frame'
);
```

## Q10. How should dynamic table rows be located?

Use stable unique row content and scope actions inside the matching row.

---

# 127. Hands-On Exercises

## Exercise 1 — Role Locator

Replace CSS button selectors with:

```text
getByRole()
```

## Exercise 2 — Label Locator

Locate:

```text
Username
Password
Email
```

using labels.

## Exercise 3 — Placeholder

Use:

```text
getByPlaceholder()
```

for a form without proper labels.

## Exercise 4 — Test ID

Add and use:

```text
data-testid
```

for a difficult component.

## Exercise 5 — Filter

Locate a user row using:

```text
filter({ hasText })
```

## Exercise 6 — Table Actions

Find a row by email and click its Edit button.

## Exercise 7 — `nth()`

Find a case where position is genuinely part of the requirement.

## Exercise 8 — Frame

Automate a form inside an iframe.

## Exercise 9 — Shadow DOM

Locate an interactive element inside an open Shadow DOM component.

## Exercise 10 — Refactoring

Take five fragile CSS/XPath selectors and replace them with stronger locator strategies.

---

# 128. Enterprise Locator Challenge

Create a locator architecture for:

```text
Login
Dashboard
Admin
Users
Reports
Profile
```

Requirements:

1. Prefer role-based locators.
2. Use labels for form fields.
3. Use test IDs only when appropriate.
4. Avoid generated IDs.
5. Avoid absolute XPath.
6. Avoid deep CSS chains.
7. Avoid unnecessary `nth()`.
8. Use filters for dynamic rows.
9. Scope actions to components.
10. Support tables.
11. Support modals.
12. Support dropdowns.
13. Support autocomplete.
14. Support iframes.
15. Support open Shadow DOM where needed.
16. Use dynamic locator methods.
17. Centralize reusable locators in Page Objects/components.
18. Use unique test data.
19. Make selectors parallel-safe.
20. Document the locator standard.

---

# 129. Final Checklist

```text
[ ] getByRole() mastered
[ ] Accessible names understood
[ ] getByLabel() mastered
[ ] getByPlaceholder() mastered
[ ] getByText() mastered
[ ] getByTestId() mastered
[ ] CSS selectors understood
[ ] XPath understood
[ ] Absolute XPath avoided
[ ] Deep CSS chains avoided
[ ] Locator chaining understood
[ ] filter() understood
[ ] hasText understood
[ ] has locator filtering understood
[ ] first() understood
[ ] last() understood
[ ] nth() understood
[ ] nth() risks understood
[ ] Strictness understood
[ ] Auto-waiting understood
[ ] Dynamic elements handled
[ ] Dynamic tables handled
[ ] Frames handled
[ ] Shadow DOM understood
[ ] Page Object locators implemented
[ ] Dynamic locator methods implemented
[ ] Locator debugging practiced
[ ] Locator picker/codegen understood
[ ] Unique test data used
[ ] Enterprise locator standard documented
```

---

# 130. Key Takeaways

- Locator quality directly affects Playwright test reliability.
- Prefer semantic, user-facing locators when possible.
- `getByRole()` is often the strongest first choice.
- `getByLabel()` is excellent for accessible forms.
- `getByPlaceholder()` is useful when stable labels are unavailable.
- `getByTestId()` provides a deliberate automation contract for difficult elements.
- CSS and XPath remain useful fallback tools but should not be the default for every element.
- Avoid deep DOM chains and generated IDs.
- Use locator chaining and filtering to target repeated components.
- Use `nth()` only when position is genuinely meaningful.
- Strictness protects tests from ambiguous actions.
- Locators work with Playwright auto-waiting and re-evaluate against the current DOM.
- Use frame locators for iframe content.
- Open Shadow DOM can be handled through Playwright's locator capabilities in supported scenarios.
- Page Objects should encapsulate reusable application locators.
- Dynamic locator methods are useful for tables, cards, users, and other runtime data.
- Unique test data makes dynamic locators more reliable under parallel execution.
- A locator failure may be caused by authentication, network, test data, or page-state problems rather than the selector itself.

---

# 131. Next Step

**Step 99 — Mastering Playwright Visual Testing & Screenshot Comparison: Baseline Screenshots, toHaveScreenshot(), Snapshot Management, Thresholds, Masks, Animations, Cross-Browser Rendering, CI Baselines & Enterprise Visual Regression Testing**

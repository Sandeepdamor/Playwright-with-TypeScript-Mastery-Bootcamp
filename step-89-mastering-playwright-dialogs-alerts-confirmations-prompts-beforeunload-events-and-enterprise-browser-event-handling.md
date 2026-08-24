# Playwright with TypeScript Mastery Bootcamp

# Step 89 — Mastering Playwright Dialogs, Alerts, Confirmations, Prompts, Beforeunload Events & Enterprise Browser Event Handling

## 1. Introduction

Web applications can display browser-level dialogs that are different from normal HTML elements.

Common browser dialogs include:

```text
Alert
Confirm
Prompt
Beforeunload
```

Examples:

```text
Are you sure you want to delete this record?
Enter your name:
Leave this page?
```

Playwright provides the `dialog` event for handling these browser dialogs.

Understanding dialog behavior is important because an unhandled dialog can block the page and cause a test to hang or fail.

---

## 2. What Is a Browser Dialog?

A browser dialog is a native browser UI element created by JavaScript.

Examples:

```ts
alert('Hello');

confirm('Delete this user?');

prompt('Enter your name');
```

These are different from HTML modals.

---

## 3. Browser Dialog vs HTML Modal

### Browser Dialog

Created using:

```ts
alert();
confirm();
prompt();
```

Handled with:

```ts
page.on('dialog', ...)
```

### HTML Modal

Created using HTML/CSS/JavaScript.

Handled using normal locators:

```ts
page.getByRole('dialog');
```

This distinction is extremely important.

---

## 4. Alert Dialog

JavaScript:

```ts
alert('Operation completed');
```

The browser displays:

```text
Operation completed
[ OK ]
```

Playwright can handle it using the `dialog` event.

---

## 5. Basic Alert Handling

Example:

```ts
page.on('dialog', async dialog => {
  console.log(dialog.message());

  await dialog.accept();
});
```

Then trigger the action:

```ts
await page.getByRole(
  'button',
  { name: 'Show Alert' }
).click();
```

---

## 6. Why Dialog Handling Must Be Registered First

Bad:

```ts
await page.getByRole(
  'button',
  { name: 'Show Alert' }
).click();

page.on('dialog', async dialog => {
  await dialog.accept();
});
```

The dialog may appear before the listener is registered.

Better:

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);

await page.getByRole(
  'button',
  { name: 'Show Alert' }
).click();
```

---

## 7. `page.on('dialog')`

Use:

```ts
page.on(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);
```

This listener can handle dialogs repeatedly.

Use it when multiple dialogs are expected.

---

## 8. `page.once('dialog')`

For a single expected dialog:

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);
```

This is often safer because the handler is removed after the first matching event.

---

## 9. Accepting a Dialog

Use:

```ts
await dialog.accept();
```

For:

```text
Alert
Confirm
Prompt
```

accepting means:

```text
Alert → OK
Confirm → OK
Prompt → Submit/OK
```

---

## 10. Dismissing a Dialog

Use:

```ts
await dialog.dismiss();
```

For a confirmation:

```text
Confirm
 ↓
Cancel
```

This is useful for testing negative workflows.

---

## 11. Reading Dialog Message

Example:

```ts
page.once(
  'dialog',
  async dialog => {
    console.log(
      dialog.message()
    );

    await dialog.accept();
  }
);
```

You can validate the message before accepting.

---

## 12. Dialog Type

You can inspect:

```ts
dialog.type()
```

Possible values include:

```text
alert
confirm
prompt
beforeunload
```

Example:

```ts
page.once(
  'dialog',
  async dialog => {
    console.log(
      dialog.type()
    );

    await dialog.accept();
  }
);
```

---

## 13. Validating Alert Message

Example:

```ts
page.once(
  'dialog',
  async dialog => {
    expect(
      dialog.message()
    ).toBe(
      'Operation completed'
    );

    await dialog.accept();
  }
);
```

This ensures the test validates the dialog rather than merely dismissing it.

---

## 14. Confirm Dialog

JavaScript:

```ts
const result =
  confirm(
    'Are you sure you want to delete?'
  );
```

The browser shows:

```text
Are you sure you want to delete?

[ OK ] [ Cancel ]
```

---

## 15. Accepting Confirm

```ts
page.once(
  'dialog',
  async dialog => {
    expect(
      dialog.type()
    ).toBe('confirm');

    await dialog.accept();
  }
);
```

This simulates clicking OK.

---

## 16. Dismissing Confirm

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.dismiss();
  }
);
```

This simulates clicking Cancel.

---

## 17. Testing Delete Confirmation

Example:

```ts
test('delete user', async ({
  page
}) => {

  page.once(
    'dialog',
    async dialog => {
      expect(
        dialog.message()
      ).toContain(
        'delete this user'
      );

      await dialog.accept();
    }
  );

  await page.getByRole(
    'button',
    { name: 'Delete' }
  ).click();

  await expect(
    page.getByText(
      'User deleted'
    )
  ).toBeVisible();
});
```

---

## 18. Testing Delete Cancellation

```ts
test('cancel delete', async ({
  page
}) => {

  page.once(
    'dialog',
    async dialog => {
      await dialog.dismiss();
    }
  );

  await page.getByRole(
    'button',
    { name: 'Delete' }
  ).click();

  await expect(
    page.getByText(
      'User deleted'
    )
  ).not.toBeVisible();
});
```

The exact final assertion should match the application's behavior.

---

## 19. Prompt Dialog

JavaScript:

```ts
const name =
  prompt(
    'Enter your name'
  );
```

The user enters a value.

Playwright can supply the value through:

```ts
dialog.accept('Sandeep');
```

---

## 20. Accepting Prompt with Input

Example:

```ts
page.once(
  'dialog',
  async dialog => {
    expect(
      dialog.type()
    ).toBe('prompt');

    await dialog.accept(
      'Sandeep'
    );
  }
);
```

---

## 21. Dismissing Prompt

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.dismiss();
  }
);
```

This simulates Cancel.

---

## 22. Reading Prompt Default Value

Dialogs can expose a default prompt value.

Example:

```ts
page.once(
  'dialog',
  async dialog => {
    console.log(
      dialog.defaultValue()
    );

    await dialog.accept(
      'Sandeep'
    );
  }
);
```

This can be useful for validating prompt initialization.

---

## 23. Prompt Validation

Example:

```ts
page.once(
  'dialog',
  async dialog => {

    expect(
      dialog.type()
    ).toBe('prompt');

    expect(
      dialog.message()
    ).toBe(
      'Enter employee name'
    );

    expect(
      dialog.defaultValue()
    ).toBe('');

    await dialog.accept(
      'Sandeep'
    );
  }
);
```

---

## 24. Beforeunload Dialog

A `beforeunload` event can appear when a page tries to navigate away while there are unsaved changes.

Example concept:

```text
Unsaved changes
      ↓
Leave page?
      ↓
Stay / Leave
```

Playwright exposes this through the dialog event.

---

## 25. Handling Beforeunload

Example:

```ts
page.on(
  'dialog',
  async dialog => {
    if (
      dialog.type() ===
      'beforeunload'
    ) {
      await dialog.accept();
    }
  }
);
```

The exact browser behavior can vary depending on the application and browser.

---

## 26. Beforeunload Test Scenario

Typical workflow:

```text
Open Form
   ↓
Modify Data
   ↓
Navigate Away
   ↓
Beforeunload
   ↓
Accept
   ↓
New Page
```

This is useful for testing unsaved-change protection.

---

## 27. Dismissing Beforeunload

Example:

```ts
page.on(
  'dialog',
  async dialog => {
    if (
      dialog.type() ===
      'beforeunload'
    ) {
      await dialog.dismiss();
    }
  }
);
```

This can simulate staying on the page.

---

## 28. Handling Different Dialog Types

A single handler can branch:

```ts
page.on(
  'dialog',
  async dialog => {

    switch (
      dialog.type()
    ) {

      case 'alert':
        await dialog.accept();
        break;

      case 'confirm':
        await dialog.dismiss();
        break;

      case 'prompt':
        await dialog.accept(
          'Sandeep'
        );
        break;

      case 'beforeunload':
        await dialog.dismiss();
        break;
    }
  }
);
```

Use this only when the behavior is intentionally global.

---

## 29. Avoid Global Dialog Handlers When Possible

A global handler like:

```ts
page.on(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);
```

can hide unexpected dialogs.

A test may accidentally pass because an unexpected confirmation was automatically accepted.

Prefer targeted handlers when the expected dialog is part of the test.

---

## 30. Unexpected Dialogs

An unexpected dialog can block the page.

For example:

```text
Test
 ↓
Click button
 ↓
Unexpected confirm
 ↓
Page blocked
 ↓
Next action hangs
```

This is why dialog handling must be deliberate.

---

## 31. Dialog Event and Assertions

Example:

```ts
page.once(
  'dialog',
  async dialog => {
    expect(
      dialog.message()
    ).toContain(
      'Are you sure'
    );

    await dialog.accept();
  }
);
```

The dialog itself can be part of the test's validation.

---

## 32. Dialog Event Ordering

The correct sequence is:

```text
Register Dialog Handler
          ↓
Perform Action
          ↓
Dialog Appears
          ↓
Validate Dialog
          ↓
Accept/Dismiss
          ↓
Continue Test
```

This prevents event race conditions.

---

## 33. Dialog Handling with `Promise`

You can coordinate dialog behavior with a promise.

Example:

```ts
const dialogPromise =
  new Promise<void>(
    resolve => {

      page.once(
        'dialog',
        async dialog => {
          expect(
            dialog.message()
          ).toContain(
            'Delete'
          );

          await dialog.accept();

          resolve();
        }
      );

    }
  );

await page.getByRole(
  'button',
  { name: 'Delete' }
).click();

await dialogPromise;
```

For most tests, `page.once()` is simpler.

---

## 34. Dialog Handling with Test Info

For advanced diagnostics:

```ts
page.once(
  'dialog',
  async dialog => {

    console.log({
      type: dialog.type(),
      message: dialog.message()
    });

    await dialog.accept();
  }
);
```

Never log sensitive values from production-like environments.

---

## 35. Dialogs and Page Objects

A Page Object can encapsulate a dialog-triggering action.

Example:

```ts
async deleteUser() {
  await this.page.getByRole(
    'button',
    { name: 'Delete' }
  ).click();
}
```

The test can register the expected dialog behavior:

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);

await adminPage.deleteUser();
```

This keeps the business action inside the Page Object while keeping test-specific dialog decisions in the test.

---

## 36. Dialog Helpers

A reusable helper can be created:

```ts
export async function
acceptDialog(
  page: Page
) {
  page.once(
    'dialog',
    async dialog => {
      await dialog.accept();
    }
  );
}
```

Usage:

```ts
await acceptDialog(page);

await page.getByRole(
  'button',
  { name: 'Delete' }
).click();
```

Keep helpers small and predictable.

---

## 37. Dialog Message Helper

Example:

```ts
export function
expectDialogMessage(
  page: Page,
  expected: string
) {
  page.once(
    'dialog',
    async dialog => {
      expect(
        dialog.message()
      ).toBe(expected);

      await dialog.accept();
    }
  );
}
```

This can reduce repeated dialog setup.

---

## 38. Dialog Fixture

In larger frameworks, dialog handling can be provided through a fixture.

Conceptually:

```text
Test
 ↓
Dialog Fixture
 ↓
Register Handler
 ↓
Trigger Action
 ↓
Validate Dialog
```

Use fixtures when dialog behavior is reused across many test suites.

---

## 39. Dialogs and Parallel Tests

Each Playwright Page has its own event handlers.

This supports isolation between parallel tests.

Avoid storing mutable global dialog state such as:

```ts
let currentDialog;
```

because parallel execution can create race conditions.

---

## 40. Dialogs and Multiple Pages

If multiple pages exist:

```text
Page A → Main App
Page B → Popup
```

each page can have its own dialog handler:

```ts
pageA.once(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);

pageB.once(
  'dialog',
  async dialog => {
    await dialog.dismiss();
  }
);
```

Keep dialog ownership clear.

---

## 41. Dialogs and Browser Contexts

Dialog events belong to pages.

Separate contexts therefore naturally isolate page dialog handling.

Example:

```text
Admin Context
 └── Admin Page
      └── Dialog Handler

Employee Context
 └── Employee Page
      └── Dialog Handler
```

---

## 42. HTML Modal vs Browser Dialog

HTML modal:

```ts
await page.getByRole(
  'dialog'
).getByRole(
  'button',
  { name: 'Confirm' }
).click();
```

Browser dialog:

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);
```

Do not confuse these mechanisms.

---

## 43. Browser Dialog Detection

A useful debugging handler:

```ts
page.on(
  'dialog',
  async dialog => {
    console.log(
      'Dialog:',
      dialog.type(),
      dialog.message()
    );

    await dialog.dismiss();
  }
);
```

Use only during debugging or controlled test environments.

---

## 44. Testing Negative Dialog Scenarios

For confirmation dialogs, test both:

```text
Accept
Dismiss
```

Example:

```text
Delete User
   ├── Confirm → User deleted
   └── Cancel  → User remains
```

This gives better functional coverage.

---

## 45. Dialog Testing Matrix

| Dialog | Accept | Dismiss | Input |
|---|---|---|---|
| Alert | Yes | No | No |
| Confirm | Yes | Yes | No |
| Prompt | Yes | Yes | Yes |
| Beforeunload | Browser-dependent behavior | Browser-dependent behavior | No |

This is a useful test-planning reference.

---

## 46. Common Dialog Mistakes

### Mistake 1 — Registering the handler after the action

Bad:

```ts
await button.click();

page.once(
  'dialog',
  handler
);
```

### Mistake 2 — Accepting every dialog globally

This can hide unexpected application behavior.

### Mistake 3 — Treating HTML modals as browser dialogs

Use locators for HTML modals.

### Mistake 4 — Not validating the message

The test may miss an incorrect confirmation message.

### Mistake 5 — Forgetting prompt input

Use:

```ts
dialog.accept('value');
```

when a prompt expects input.

### Mistake 6 — Ignoring unexpected dialogs

An unexpected dialog can block the page.

---

## 47. Production-Quality Alert Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('verify success alert', async ({
  page
}) => {
  await page.goto('/alerts');

  page.once(
    'dialog',
    async dialog => {

      expect(
        dialog.type()
      ).toBe('alert');

      expect(
        dialog.message()
      ).toBe(
        'Operation completed'
      );

      await dialog.accept();
    }
  );

  await page.getByRole(
    'button',
    { name: 'Show Alert' }
  ).click();
});
```

---

## 48. Production-Quality Confirm Example

```ts
test('confirm deletion', async ({
  page
}) => {
  await page.goto('/users');

  page.once(
    'dialog',
    async dialog => {

      expect(
        dialog.type()
      ).toBe('confirm');

      expect(
        dialog.message()
      ).toContain(
        'Delete'
      );

      await dialog.accept();
    }
  );

  await page.getByRole(
    'button',
    { name: 'Delete' }
  ).click();

  await expect(
    page.getByRole('status')
  ).toContainText(
    'deleted'
  );
});
```

---

## 49. Production-Quality Prompt Example

```ts
test('enter employee name', async ({
  page
}) => {
  await page.goto('/employees');

  page.once(
    'dialog',
    async dialog => {

      expect(
        dialog.type()
      ).toBe('prompt');

      expect(
        dialog.message()
      ).toBe(
        'Enter employee name'
      );

      await dialog.accept(
        'Sandeep'
      );
    }
  );

  await page.getByRole(
    'button',
    { name: 'Add Employee' }
  ).click();

  await expect(
    page.getByText('Sandeep')
  ).toBeVisible();
});
```

---

## 50. Production-Quality Cancel Example

```ts
test('cancel deletion', async ({
  page
}) => {
  await page.goto('/users');

  page.once(
    'dialog',
    async dialog => {
      await dialog.dismiss();
    }
  );

  await page.getByRole(
    'button',
    { name: 'Delete' }
  ).click();

  await expect(
    page.getByText(
      'User still exists'
    )
  ).toBeVisible();
});
```

---

## 51. Enterprise Dialog Architecture

A mature framework can organize:

```text
utils/
└── dialog-utils.ts

fixtures/
└── dialog.fixture.ts

pages/
├── UserPage.ts
├── EmployeePage.ts
└── SettingsPage.ts

tests/
├── user-delete.spec.ts
└── employee-create.spec.ts
```

The goal is to keep dialog mechanics reusable without hiding test intent.

---

## 52. Recommended Dialog Strategy

### For one expected dialog

Use:

```ts
page.once(
  'dialog',
  handler
);
```

### For repeated dialogs

Use:

```ts
page.on(
  'dialog',
  handler
);
```

### For complex reusable dialog behavior

Use:

```text
Helper
or
Fixture
```

### For HTML modals

Use:

```text
Locator
```

---

## 53. Dialog Security Considerations

Dialog messages may contain:

- User information
- Transaction details
- Account information
- Internal system information

Avoid logging sensitive messages in CI.

If diagnostics are needed, sanitize the message:

```ts
console.log(
  'Dialog type:',
  dialog.type()
);
```

rather than printing sensitive content.

---

## 54. Dialog Best Practices

1. Register handlers before triggering actions.
2. Use `page.once()` for one expected dialog.
3. Validate dialog type.
4. Validate dialog message when important.
5. Use `accept()` for positive paths.
6. Use `dismiss()` for negative paths.
7. Provide prompt input when required.
8. Do not confuse browser dialogs with HTML modals.
9. Avoid global handlers that silently accept every dialog.
10. Keep dialog logic isolated and reusable.
11. Do not log sensitive dialog content.
12. Test both confirmation and cancellation paths.

---

## 55. Interview Questions

### Q1. How do you handle an alert in Playwright?

```ts
page.once(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);
```

### Q2. How do you dismiss a confirmation?

```ts
await dialog.dismiss();
```

### Q3. How do you enter text into a prompt?

```ts
await dialog.accept('Sandeep');
```

### Q4. How do you read the dialog message?

```ts
dialog.message();
```

### Q5. How do you identify the dialog type?

```ts
dialog.type();
```

### Q6. What is the difference between `page.on()` and `page.once()`?

`on()` registers a persistent event listener, while `once()` handles the next matching event and then removes the listener.

### Q7. Why register the dialog listener before clicking?

To prevent a race condition where the dialog appears before the handler is registered.

### Q8. How are HTML modals handled?

Using normal Playwright locators, such as:

```ts
page.getByRole('dialog');
```

### Q9. Why should a global `dialog.accept()` handler be avoided?

It can silently accept unexpected dialogs and hide application defects.

### Q10. What should a robust dialog test validate?

At minimum, validate the expected dialog behavior and, where relevant, its type and message before accepting or dismissing it.

---

## 56. Hands-On Exercises

### Exercise 1 — Alert

Handle an alert and validate its message.

### Exercise 2 — Confirm Accept

Accept a delete confirmation and verify deletion.

### Exercise 3 — Confirm Cancel

Dismiss a delete confirmation and verify the record remains.

### Exercise 4 — Prompt

Enter a name into a prompt and verify the resulting UI.

### Exercise 5 — Prompt Cancel

Dismiss a prompt and verify that no record is created.

### Exercise 6 — Beforeunload

Create an unsaved form and test the navigation protection behavior.

### Exercise 7 — Dialog Utility

Create:

```ts
acceptDialog()
dismissDialog()
expectDialog()
```

helpers.

### Exercise 8 — Multiple Dialogs

Create a workflow that produces multiple dialogs and handle them deterministically.

### Exercise 9 — Multi-Page Dialogs

Open a popup and handle a dialog belonging specifically to the popup page.

### Exercise 10 — Enterprise Dialog Fixture

Build a reusable fixture that supports:

```text
expected type
expected message
accept
dismiss
prompt input
```

without hiding the test's business intent.

---

## 57. Key Takeaways

- Browser dialogs are different from HTML modals.
- Playwright uses the `dialog` event for browser dialogs.
- Use `accept()` to accept a dialog.
- Use `dismiss()` to cancel it.
- Use `accept('value')` for prompts.
- `dialog.message()` returns the dialog message.
- `dialog.type()` identifies the dialog type.
- `dialog.defaultValue()` provides a prompt's default value.
- Register dialog handlers before triggering the action.
- `page.once()` is ideal for one expected dialog.
- `page.on()` is useful for repeated dialog events.
- Avoid global handlers that automatically accept everything.
- Validate dialog behavior instead of merely dismissing it.
- Use locators for HTML modals.
- Protect sensitive dialog information in logs.
- Test both positive and negative confirmation paths.

---

## 58. Enterprise Dialog Challenge

Build a reusable browser-event handling framework supporting:

```text
Alert
Confirm
Prompt
Beforeunload
```

Requirements:

1. Create reusable dialog helpers.
2. Validate dialog type.
3. Validate dialog message.
4. Support accept.
5. Support dismiss.
6. Support prompt input.
7. Handle dialogs in popup pages.
8. Avoid global hidden behavior.
9. Protect sensitive information.
10. Integrate the solution with Page Objects and fixtures.

---

## 59. Final Architecture

```text
Test
 │
 ├── Page Object
 │      ↓
 │   Business Action
 │      ↓
 │   Browser Dialog
 │      ↓
 │   Dialog Utility / Fixture
 │      ↓
 │   Validate
 │      ↓
 │   Accept / Dismiss
 │
 └── Business Assertion
```

This architecture keeps browser-event handling predictable, reusable, and easy to debug.

---

## 60. Next Step

**Step 90 — Mastering Playwright Downloads, File Uploads, File Choosers, Download Validation, Temporary Files & Enterprise File Automation**

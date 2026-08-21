# Playwright with TypeScript Mastery Bootcamp

# Step 77 — Mastering Playwright Browser Dialogs, Alerts, Confirms & Prompts

## 1. Introduction

Modern web applications sometimes display browser-native dialogs such as:

- JavaScript alerts
- Confirmation dialogs
- Prompt dialogs
- Before-unload dialogs

Playwright provides built-in support for handling these dialogs through the `Dialog` API and the `dialog` event.

Browser-native dialogs are not normal DOM elements, so they cannot be handled with regular locators.

---

## 2. Types of Browser Dialogs

### Alert

```ts
alert('Operation completed');
```

An alert normally contains a message and an OK button.

### Confirm

```ts
confirm('Are you sure?');
```

A confirm dialog provides OK and Cancel choices.

### Prompt

```ts
prompt('Enter your name');
```

A prompt allows the user to enter text.

### Before-Unload

A before-unload dialog can appear when a page attempts to navigate away while unsaved changes exist.

---

## 3. Playwright Dialog API

Important APIs include:

```ts
dialog.type()
dialog.message()
dialog.defaultValue()
dialog.accept()
dialog.dismiss()
```

These APIs allow tests to inspect and control browser dialogs.

---

## 4. Reading Dialog Type

Use:

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
page.on('dialog', async dialog => {
  console.log(dialog.type());

  await dialog.accept();
});
```

---

## 5. Reading the Dialog Message

Use:

```ts
dialog.message()
```

Example:

```ts
page.on('dialog', async dialog => {
  console.log('Dialog message:', dialog.message());

  await dialog.accept();
});
```

This is useful for validating the message displayed to the user.

---

## 6. Handling an Alert

```ts
import { test, expect } from '@playwright/test';

test('handle alert dialog', async ({ page }) => {
  page.once('dialog', async dialog => {
    expect(dialog.type()).toBe('alert');
    expect(dialog.message()).toBe('Hello Playwright');

    await dialog.accept();
  });

  await page.goto('/dialogs');

  await page.getByRole('button', { name: 'Show Alert' }).click();
});
```

The important point is that the dialog handler is registered before the action that opens the dialog.

---

## 7. Accepting a Confirm Dialog

```ts
page.once('dialog', async dialog => {
  expect(dialog.type()).toBe('confirm');

  await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

`accept()` simulates clicking OK.

---

## 8. Dismissing a Confirm Dialog

```ts
page.once('dialog', async dialog => {
  await dialog.dismiss();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

`dismiss()` simulates clicking Cancel.

---

## 9. Handling a Prompt Dialog

A prompt can receive a value through `accept()`:

```ts
page.once('dialog', async dialog => {
  expect(dialog.type()).toBe('prompt');

  await dialog.accept('Sandeep');
});

await page.getByRole('button', { name: 'Enter Name' }).click();
```

---

## 10. Reading a Prompt Default Value

Use:

```ts
dialog.defaultValue()
```

Example:

```ts
page.once('dialog', async dialog => {
  console.log('Default value:', dialog.defaultValue());

  await dialog.accept('Playwright');
});
```

---

## 11. Why the Dialog Handler Must Be Registered First

Correct:

```ts
page.once('dialog', async dialog => {
  await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

Incorrect:

```ts
await page.getByRole('button', { name: 'Delete' }).click();

page.once('dialog', async dialog => {
  await dialog.accept();
});
```

The dialog can block the page before the second handler is registered.

---

## 12. `page.once()` vs `page.on()`

### `page.once()`

Use when one dialog is expected:

```ts
page.once('dialog', async dialog => {
  await dialog.accept();
});
```

The listener executes once.

### `page.on()`

Use when multiple dialogs may occur:

```ts
page.on('dialog', async dialog => {
  await dialog.accept();
});
```

The listener remains registered.

---

## 13. Using `waitForEvent('dialog')`

For explicit dialog inspection:

```ts
const dialogPromise = page.waitForEvent('dialog');

await page.getByRole('button', { name: 'Delete' }).click();

const dialog = await dialogPromise;

expect(dialog.type()).toBe('confirm');
expect(dialog.message()).toContain('delete');

await dialog.accept();
```

This pattern is especially useful when the test needs to inspect the dialog before deciding how to handle it.

---

## 14. Using `Promise.all()`

A compact pattern is:

```ts
const [dialog] = await Promise.all([
  page.waitForEvent('dialog'),
  page.getByRole('button', { name: 'Delete' }).click()
]);

expect(dialog.message()).toContain('delete');

await dialog.accept();
```

The event wait is established before the action executes.

---

## 15. Conditional Dialog Handling

Different dialog types can require different behavior:

```ts
page.on('dialog', async dialog => {
  if (dialog.type() === 'confirm') {
    await dialog.accept();
  } else if (dialog.type() === 'prompt') {
    await dialog.accept('Playwright');
  } else {
    await dialog.dismiss();
  }
});
```

This pattern can be useful in reusable framework utilities.

---

## 16. Validating Dialog Content

You can validate:

```ts
dialog.type()
dialog.message()
dialog.defaultValue()
```

Example:

```ts
page.once('dialog', async dialog => {
  expect(dialog.type()).toBe('confirm');
  expect(dialog.message()).toContain('delete');

  await dialog.accept();
});
```

---

## 17. Dialogs Are Not DOM Elements

Incorrect:

```ts
await page.getByText('Are you sure?').click();
```

Native browser dialogs are outside the normal DOM interaction model.

Correct:

```ts
page.once('dialog', async dialog => {
  await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

---

## 18. Handling Multiple Dialogs

```ts
page.on('dialog', async dialog => {
  console.log('Type:', dialog.type());
  console.log('Message:', dialog.message());

  await dialog.accept();
});
```

The handler can process each dialog as it appears.

Use broad listeners carefully because they may hide unexpected application behavior.

---

## 19. Removing a Dialog Listener

A persistent listener can be removed:

```ts
const handler = async (dialog: any) => {
  await dialog.accept();
};

page.on('dialog', handler);

page.off('dialog', handler);
```

For one expected dialog, `page.once()` is usually simpler.

---

## 20. Dialog Handling in Page Objects

Example:

```ts
import { Page } from '@playwright/test';

export class DeletePage {
  constructor(private readonly page: Page) {}

  async deleteWithConfirmation() {
    this.page.once('dialog', async dialog => {
      await dialog.accept();
    });

    await this.page
      .getByRole('button', { name: 'Delete' })
      .click();
  }

  async cancelDelete() {
    this.page.once('dialog', async dialog => {
      await dialog.dismiss();
    });

    await this.page
      .getByRole('button', { name: 'Delete' })
      .click();
  }
}
```

This keeps dialog implementation details inside the Page Object.

---

## 21. Enterprise Dialog Strategy

In a larger framework, repeated dialog behavior can be centralized in:

- Page Objects
- Fixtures
- Utility classes
- Test hooks

However, avoid automatically accepting every dialog unless that is intentionally required.

Unexpected dialogs can indicate:

- Application defects
- Validation problems
- Authentication issues
- Navigation problems
- Data problems

---

## 22. Unexpected Dialog Detection

A strict handler can fail the test:

```ts
page.on('dialog', async dialog => {
  throw new Error(
    `Unexpected dialog: ${dialog.message()}`
  );
});
```

This can be useful when the test explicitly expects that no browser dialogs should appear.

---

## 23. Production-Quality Example

```ts
import { test, expect } from '@playwright/test';

test('should delete record after confirmation', async ({ page }) => {
  await page.goto('/users');

  const dialogPromise = page.waitForEvent('dialog');

  await page.getByRole('button', { name: 'Delete' }).click();

  const dialog = await dialogPromise;

  expect(dialog.type()).toBe('confirm');
  expect(dialog.message()).toContain('Are you sure');

  await dialog.accept();

  await expect(
    page.getByText('User deleted successfully')
  ).toBeVisible();
});
```

This follows a strong automation pattern:

1. Navigate.
2. Prepare for the dialog.
3. Trigger the business action.
4. Inspect the dialog.
5. Accept or dismiss it.
6. Verify the business result.

---

## 24. Best Practices

### Register the handler before the action

```ts
page.once('dialog', handler);

await page.getByRole('button').click();
```

### Prefer `once()` for one expected dialog

```ts
page.once('dialog', async dialog => {
  await dialog.accept();
});
```

### Validate important dialog messages

```ts
expect(dialog.message()).toContain('delete');
```

### Keep dialog handling close to the triggering action

This improves test readability.

### Avoid unnecessary hard waits

Do not use:

```ts
await page.waitForTimeout(5000);
```

Dialog handling should use Playwright events rather than arbitrary delays.

### Centralize repeated behavior

Use Page Objects or fixtures when the same dialog behavior occurs repeatedly.

---

## 25. Real-World Enterprise Scenarios

Browser dialogs may appear in:

- Delete confirmations
- Logout confirmations
- Unsaved-change warnings
- Legacy JavaScript applications
- Administrative workflows
- Data export operations
- Security workflows
- Legacy enterprise applications

A robust automation framework should distinguish expected dialogs from unexpected dialogs.

---

## 26. Debugging Checklist

When a dialog test fails, check:

1. Is the dialog actually being triggered?
2. Was the handler registered before the action?
3. Is the dialog type correct?
4. Is the expected message correct?
5. Should the dialog be accepted or dismissed?
6. Does a prompt require input?
7. Is another unexpected dialog appearing?
8. Is a persistent listener affecting another test?
9. Is the dialog generated by the application or navigation?
10. Can the issue be reproduced in an isolated test?

---

## 27. Interview Questions

### Q1. How does Playwright handle JavaScript alerts?

Using the `dialog` event:

```ts
page.once('dialog', async dialog => {
  await dialog.accept();
});
```

### Q2. How do you dismiss a confirm dialog?

```ts
await dialog.dismiss();
```

### Q3. How do you enter text into a prompt?

```ts
await dialog.accept('Playwright');
```

### Q4. How do you get the dialog message?

```ts
dialog.message()
```

### Q5. How do you identify the dialog type?

```ts
dialog.type()
```

### Q6. Why should the dialog listener be registered before clicking the button?

Because the browser dialog can block the page. The listener must be ready when the dialog appears.

### Q7. What is the difference between `page.on()` and `page.once()`?

`page.on()` remains registered for future events, while `page.once()` handles only the next matching event.

### Q8. Can browser dialogs be located with Playwright locators?

No. Native browser dialogs are handled through Playwright's `Dialog` API rather than normal DOM locators.

---

## 28. Hands-On Practice

### Exercise 1 — Alert

Create a test that:

- Opens an alert.
- Verifies the type.
- Verifies the message.
- Accepts the alert.

### Exercise 2 — Confirm

Create tests for:

- Accepting a confirmation.
- Dismissing a confirmation.

### Exercise 3 — Prompt

Create a test that:

- Opens a prompt.
- Reads the default value.
- Enters a custom value.
- Accepts the prompt.

### Exercise 4 — Reusable Dialog Utility

Create a reusable utility supporting:

```ts
accept()
dismiss()
acceptWithText()
verifyMessage()
verifyType()
```

### Exercise 5 — Page Object

Move repeated dialog handling into a Page Object while keeping the test focused on business behavior.

---

## 29. Key Takeaways

- Browser dialogs are not DOM elements.
- Playwright handles them through the `Dialog` API.
- `dialog.accept()` accepts a dialog.
- `dialog.dismiss()` dismisses a dialog.
- `dialog.message()` returns the dialog message.
- `dialog.type()` returns the dialog type.
- `dialog.defaultValue()` returns a prompt's default value.
- `dialog.accept('value')` provides prompt input.
- Register dialog handling before triggering the dialog.
- `page.once('dialog')` is useful for one expected dialog.
- `page.on('dialog')` is useful for repeated dialog handling.
- `page.waitForEvent('dialog')` is useful for explicit dialog inspection.
- Page Objects and fixtures can centralize repeated dialog behavior.
- Avoid blanket automatic acceptance because it can hide application defects.

---

## 30. Next Step

**Step 78 — Mastering Playwright Downloads, Uploads, File Choosers & Enterprise File Handling**

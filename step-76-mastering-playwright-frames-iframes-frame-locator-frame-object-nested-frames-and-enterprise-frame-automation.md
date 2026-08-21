# Step 76 — Mastering Playwright Frames & Iframes

## 1. Introduction

Modern web applications often contain content inside `<iframe>` elements. Examples include:

- Payment gateways
- Embedded videos
- Rich text editors
- Maps
- Third-party widgets
- Authentication components
- Embedded reports

Playwright provides the `Frame` API and `frameLocator()` to work with elements inside iframes.

---

## 2. What Is an iframe?

An iframe (inline frame) allows one HTML page to display another HTML document inside it.

Example:

```html
<iframe src="https://example.com/payment"></iframe>
```

The content inside the iframe belongs to a different document context.

A normal locator such as:

```ts
page.getByRole('button', { name: 'Pay' })
```

may not find a button if the button exists inside an iframe.

---

## 3. Understanding Playwright Frames

A Playwright `Page` can contain one or more frames.

Important APIs include:

```ts
page.frames()
page.mainFrame()
page.frame()
page.frameLocator()
```

### Main frame

The main page document is called the main frame.

```ts
const mainFrame = page.mainFrame();
```

### Get all frames

```ts
const frames = page.frames();

for (const frame of frames) {
  console.log(frame.url());
}
```

---

## 4. Using `frameLocator()`

The recommended approach for most UI automation is `frameLocator()`.

Example:

```ts
const frame = page.frameLocator('#payment-frame');

await frame.getByLabel('Card number').fill('4111111111111111');
```

You can chain normal Playwright locators inside the frame:

```ts
await page
  .frameLocator('#payment-frame')
  .getByRole('button', { name: 'Pay Now' })
  .click();
```

---

## 5. Locating an iframe

### By CSS selector

```ts
const paymentFrame = page.frameLocator('#payment-frame');
```

### By iframe title

```ts
const editorFrame = page.frameLocator('iframe[title="Editor"]');
```

### By iframe name

```ts
const frame = page.frameLocator('iframe[name="payment"]');
```

Prefer stable attributes such as:

- `id`
- `name`
- `title`
- `data-testid`

Avoid fragile selectors when possible.

---

## 6. Interacting with Elements Inside an iframe

Suppose the iframe contains:

```html
<input aria-label="Username">
<input aria-label="Password">
<button>Login</button>
```

Playwright test:

```ts
const loginFrame = page.frameLocator('#login-frame');

await loginFrame.getByLabel('Username').fill('admin');
await loginFrame.getByLabel('Password').fill('secret');
await loginFrame.getByRole('button', { name: 'Login' }).click();
```

This keeps the test readable and follows Playwright's locator model.

---

## 7. Using the `Frame` Object

Sometimes you need direct access to a frame.

```ts
const frame = page.frame({ name: 'payment' });

if (!frame) {
  throw new Error('Payment frame was not found');
}

await frame.getByLabel('Card Number').fill('4111111111111111');
```

You can also locate a frame by URL:

```ts
const frame = page.frame({
  url: /payment/
});
```

---

## 8. `frameLocator()` vs `Frame`

### `frameLocator()`

Best for normal UI interaction:

```ts
await page
  .frameLocator('#my-frame')
  .getByRole('button', { name: 'Submit' })
  .click();
```

### `Frame`

Useful when you need direct frame-level operations:

```ts
const frame = page.frame({ name: 'my-frame' });

if (frame) {
  console.log(await frame.title());
}
```

### Practical rule

Use `frameLocator()` by default for UI automation.

Use the `Frame` object when you specifically need frame-level APIs or programmatic access.

---

## 9. Nested Iframes

An iframe can contain another iframe.

Example:

```ts
const outerFrame = page.frameLocator('#outer-frame');

const innerFrame = outerFrame.frameLocator('#inner-frame');

await innerFrame.getByRole('button', { name: 'Submit' }).click();
```

This allows Playwright to traverse multiple frame levels.

---

## 10. Frame Assertions

Assertions can also be performed inside a frame.

```ts
import { expect } from '@playwright/test';

const frame = page.frameLocator('#payment-frame');

await expect(
  frame.getByRole('heading', { name: 'Payment Details' })
).toBeVisible();
```

This is useful for validating that the embedded component loaded successfully.

---

## 11. Handling Dynamic Iframes

Sometimes iframe attributes change dynamically.

Avoid relying on unstable IDs.

Instead, use stable attributes:

```ts
const frame = page.frameLocator(
  'iframe[title="Secure Payment"]'
);
```

Or use a test-specific attribute when available:

```ts
const frame = page.frameLocator(
  'iframe[data-testid="payment-frame"]'
);
```

---

## 12. Common Mistake

### Incorrect

```ts
await page.getByRole('button', { name: 'Pay Now' }).click();
```

If the button is inside an iframe, this locator searches the main document.

### Correct

```ts
await page
  .frameLocator('#payment-frame')
  .getByRole('button', { name: 'Pay Now' })
  .click();
```

---

## 13. Reusable Page Object Example

### PaymentPage.ts

```ts
import { Page } from '@playwright/test';

export class PaymentPage {
  constructor(private readonly page: Page) {}

  private paymentFrame = this.page.frameLocator('#payment-frame');

  async enterCardNumber(cardNumber: string) {
    await this.paymentFrame
      .getByLabel('Card number')
      .fill(cardNumber);
  }

  async enterExpiryDate(expiry: string) {
    await this.paymentFrame
      .getByLabel('Expiry date')
      .fill(expiry);
  }

  async enterCvv(cvv: string) {
    await this.paymentFrame
      .getByLabel('CVV')
      .fill(cvv);
  }

  async clickPay() {
    await this.paymentFrame
      .getByRole('button', { name: 'Pay Now' })
      .click();
  }
}
```

This keeps iframe implementation details inside the Page Object.

---

## 14. Complete Test Example

```ts
import { test, expect } from '@playwright/test';

test('should interact with payment iframe', async ({ page }) => {
  await page.goto('/checkout');

  const paymentFrame = page.frameLocator('#payment-frame');

  await paymentFrame
    .getByLabel('Card number')
    .fill('4111111111111111');

  await paymentFrame
    .getByLabel('Expiry date')
    .fill('12/30');

  await paymentFrame
    .getByLabel('CVV')
    .fill('123');

  await paymentFrame
    .getByRole('button', { name: 'Pay Now' })
    .click();

  await expect(
    page.getByText('Payment successful')
  ).toBeVisible();
});
```

---

## 15. Debugging iframe Tests

When an iframe test fails, inspect the frames:

```ts
console.log('Frame count:', page.frames().length);

for (const frame of page.frames()) {
  console.log('Frame URL:', frame.url());
}
```

You can also inspect the page using:

```bash
npx playwright test --debug
```

Or run headed:

```bash
npx playwright test --headed
```

---

## 16. Best Practices

### Use `frameLocator()` for UI interactions

```ts
page.frameLocator('#my-frame')
```

### Prefer stable selectors

```ts
iframe[data-testid="payment-frame"]
```

### Keep iframe logic inside Page Objects

This prevents test files from becoming tightly coupled to iframe implementation details.

### Avoid unnecessary waits

Do not use:

```ts
await page.waitForTimeout(5000);
```

Playwright automatically waits for locator actions and assertions to become actionable.

### Use meaningful assertions

```ts
await expect(
  paymentFrame.getByText('Payment successful')
).toBeVisible();
```

---

## 17. Interview Questions

### Q1. What is an iframe?

An iframe is an HTML element that embeds another HTML document inside the current page.

### Q2. How do you interact with elements inside an iframe in Playwright?

Use `frameLocator()`:

```ts
await page
  .frameLocator('#my-frame')
  .getByRole('button', { name: 'Submit' })
  .click();
```

### Q3. What is the difference between `frameLocator()` and `Frame`?

`frameLocator()` is primarily designed for locator-based UI interactions, while the `Frame` object provides direct access to the frame and its APIs.

### Q4. How do you get all frames?

```ts
const frames = page.frames();
```

### Q5. How do you get the main frame?

```ts
const mainFrame = page.mainFrame();
```

### Q6. Can Playwright handle nested iframes?

Yes.

```ts
const outer = page.frameLocator('#outer');
const inner = outer.frameLocator('#inner');

await inner.getByRole('button').click();
```

---

## 18. Key Takeaways

- An iframe represents a separate document context.
- Normal page locators may not directly access iframe content.
- `frameLocator()` is the preferred approach for UI automation.
- `page.frames()` returns all frames.
- `page.mainFrame()` returns the main document frame.
- The `Frame` object provides direct frame-level access.
- Nested iframes can be handled by chaining `frameLocator()`.
- Stable iframe selectors make tests more reliable.
- Keep iframe interactions inside Page Objects.
- Avoid unnecessary hard waits.

---

## 19. Practice Exercise

Create a Playwright test that:

1. Opens a page containing an iframe.
2. Locates the iframe using a stable selector.
3. Enters text into an input inside the iframe.
4. Clicks a button inside the iframe.
5. Verifies the expected result.
6. Converts the iframe interaction into a reusable Page Object method.
7. Adds an assertion for the iframe content.

---

## 20. Next Step

**Step 77 — Mastering Playwright Browser Dialogs, Alerts, Confirms & Prompts**

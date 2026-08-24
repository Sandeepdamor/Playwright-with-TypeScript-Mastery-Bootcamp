# Playwright with TypeScript Mastery Bootcamp

# Step 87 — Mastering Playwright Frames & iFrames, Frame Locators, Nested Frames, Cross-Origin Frames & Enterprise Frame Automation

## 1. Introduction

Modern web applications sometimes embed content inside HTML frames or iframes.

Examples include:

- Payment forms
- Embedded reports
- Rich text editors
- Authentication widgets
- Maps
- External applications
- Third-party integrations
- Legacy enterprise applications

Playwright provides dedicated APIs for working with frames.

The most important APIs are:

```text
page.frames()
page.frame()
page.frameLocator()
frameLocator.locator()
frameLocator.getByRole()
frameLocator.getByLabel()
```

Understanding frame boundaries is essential because elements inside an iframe belong to a different document context.

---

## 2. What Is an iframe?

An iframe is an HTML element that embeds another document inside the current page.

Example:

```html
<iframe
  src="/payment.html"
  title="Payment">
</iframe>
```

Conceptually:

```text
Main Page
   |
   └── iframe
        |
        └── Embedded Document
```

The embedded document has its own DOM.

---

## 3. Why Frames Need Special Handling

Suppose the main page contains:

```html
<button>Checkout</button>

<iframe src="/payment.html">
</iframe>
```

Inside the iframe:

```html
<input name="cardNumber">
<button>Pay</button>
```

A normal page locator such as:

```ts
page.getByRole(
  'button',
  { name: 'Pay' }
)
```

may not find the button because it belongs to the iframe document.

You need to work within the frame.

---

## 4. Detecting Frames

Playwright provides:

```ts
page.frames()
```

Example:

```ts
const frames = page.frames();

console.log(frames.length);
```

This returns the frames currently attached to the page.

---

## 5. Main Frame

The main page itself is represented by the main frame.

You can access it with:

```ts
page.mainFrame()
```

Example:

```ts
const mainFrame =
  page.mainFrame();

console.log(mainFrame.url());
```

---

## 6. Child Frames

An iframe appears as a child frame.

You can inspect:

```ts
for (const frame of page.frames()) {
  console.log(frame.url());
}
```

This can help identify which frame contains the required content.

---

## 7. `page.frame()`

You can locate a frame by name or URL-related criteria depending on the API usage.

Example:

```ts
const frame =
  page.frame({
    name: 'payment'
  });
```

Another approach is to inspect available frames and select the required one.

For many UI tests, `frameLocator()` is simpler.

---

## 8. `frameLocator()`

The recommended API for many iframe interactions is:

```ts
const paymentFrame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );
```

Then:

```ts
await paymentFrame
  .getByLabel('Card number')
  .fill('4111111111111111');
```

This keeps the frame boundary explicit.

---

## 9. Frame Locator with Role

Example:

```ts
const paymentFrame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await paymentFrame.getByRole(
  'button',
  { name: 'Pay' }
).click();
```

This combines:

```text
Frame Locator
+
Role Locator
```

---

## 10. Frame Locator with Text

Example:

```ts
const frame =
  page.frameLocator(
    'iframe[title="Help"]'
  );

await expect(
  frame.getByText('Support')
).toBeVisible();
```

---

## 11. Frame Locator with Label

Example:

```ts
const frame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await frame
  .getByLabel('Card number')
  .fill('4111111111111111');
```

This is often cleaner than manually retrieving a `Frame` object.

---

## 12. Frame Locator with Test ID

Example:

```ts
const frame =
  page.frameLocator(
    '[data-testid="payment-frame"]'
  );

await frame
  .getByTestId('card-number')
  .fill('4111111111111111');
```

---

## 13. Frame Locator Chaining

Frame locators can be combined with normal locator operations.

Example:

```ts
const frame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

const cardNumber =
  frame.getByLabel('Card number');

await cardNumber.fill(
  '4111111111111111'
);
```

---

## 14. Frame vs FrameLocator

There are two important concepts.

### Frame

```ts
const frame =
  page.frames()[1];
```

A `Frame` represents a browsing context.

### FrameLocator

```ts
const frame =
  page.frameLocator('iframe');
```

A `FrameLocator` is designed for locating and interacting with elements inside the frame.

For normal UI automation, `FrameLocator` is often the simpler approach.

---

## 15. When to Use `Frame`

Use a `Frame` when you need frame-level operations such as:

```text
URL inspection
Frame navigation
Frame-specific evaluation
Frame lifecycle information
```

Example:

```ts
const frame =
  page.frames()
    .find(frame =>
      frame.url().includes('payment')
    );

if (!frame) {
  throw new Error(
    'Payment frame not found'
  );
}
```

---

## 16. When to Use `FrameLocator`

Use `FrameLocator` when your main goal is UI interaction.

Example:

```ts
const frame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await frame
  .getByLabel('Card number')
  .fill('4111111111111111');
```

This is concise and readable.

---

## 17. Frame Locator and Auto-Waiting

Frame locators work with Playwright's normal waiting behavior.

Example:

```ts
await expect(
  page.frameLocator(
    'iframe[title="Payment"]'
  ).getByRole(
    'button',
    { name: 'Pay' }
  )
).toBeVisible();
```

Playwright waits for the target element to satisfy the assertion.

---

## 18. iframe Selector Stability

Avoid:

```ts
page.frameLocator(
  'iframe:nth-child(4)'
);
```

Prefer:

```ts
page.frameLocator(
  'iframe[title="Payment"]'
);
```

or:

```ts
page.frameLocator(
  '[data-testid="payment-frame"]'
);
```

Use a stable iframe identifier.

---

## 19. Multiple iframes

A page may contain:

```text
Main Page
 ├── Header iframe
 ├── Payment iframe
 ├── Chat iframe
 └── Analytics iframe
```

Do not assume:

```ts
page.frames()[1]
```

is always the payment frame.

Frame ordering can change.

Use a meaningful identifier.

---

## 20. Frame by URL

You can inspect frame URLs:

```ts
for (const frame of page.frames()) {
  console.log(frame.url());
}
```

Then identify the expected frame.

Example:

```ts
const paymentFrame =
  page.frames().find(frame =>
    frame.url().includes(
      '/payment'
    )
  );
```

---

## 21. Frame by Name

If the iframe has:

```html
<iframe
  name="payment-frame">
</iframe>
```

You can identify the frame by name when using the `Frame` API.

For UI interaction, a stable iframe selector with `frameLocator()` is often easier.

---

## 22. Nested Frames

An iframe can contain another iframe.

Conceptually:

```text
Main Page
   ↓
Outer iframe
   ↓
Inner iframe
   ↓
Button
```

Playwright can work with nested frame structures.

---

## 23. Nested Frame Example

HTML:

```html
<iframe id="outer">
  <iframe id="inner">
  </iframe>
</iframe>
```

Conceptually:

```ts
const outer =
  page.frameLocator('#outer');

const inner =
  outer.frameLocator('#inner');

await inner.getByRole(
  'button',
  { name: 'Submit' }
).click();
```

Keep the hierarchy explicit.

---

## 24. Nested Frame Scoping

Nested frame automation should follow:

```text
Page
 ↓
Outer Frame
 ↓
Inner Frame
 ↓
Component
 ↓
Element
```

This avoids ambiguous selectors.

---

## 25. Cross-Origin Frames

An iframe can load content from another origin.

Example:

```text
Main Application
https://app.example.com

iframe
https://payments.example.com
```

The browser enforces origin boundaries.

Playwright can interact with frames when the embedded content is available to the browser automation context.

However, application behavior and third-party restrictions can affect what can be automated.

---

## 26. Third-Party Payment Frames

Payment providers frequently use iframes.

Conceptually:

```text
Checkout Page
    ↓
Payment Provider iframe
    ↓
Card Number
Expiry
CVV
```

Example:

```ts
const paymentFrame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await paymentFrame
  .getByLabel('Card number')
  .fill('...');
```

In real systems, follow the provider's testing documentation and use test credentials/data.

---

## 27. Same-Origin vs Cross-Origin

Same-origin iframe:

```text
https://app.example.com
        ↓
https://app.example.com/payment
```

Cross-origin iframe:

```text
https://app.example.com
        ↓
https://payments.example.com
```

Cross-origin behavior is controlled by browser security policies.

Do not assume that JavaScript DOM access rules are identical to Playwright's automation APIs.

---

## 28. Frame Navigation

A frame can navigate independently.

Using a `Frame`:

```ts
await frame.goto(
  'https://example.com'
);
```

Use this only when direct frame navigation is actually required.

Most end-to-end workflows should navigate through the application's UI.

---

## 29. Frame URL Validation

Example:

```ts
const frame =
  page.frames().find(frame =>
    frame.url().includes(
      '/payment'
    )
  );

expect(frame).toBeDefined();
```

For a web-first UI test, you can also assert visible frame content instead of relying only on URL details.

---

## 30. Frame Content Validation

Example:

```ts
const paymentFrame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await expect(
  paymentFrame.getByText(
    'Secure Payment'
  )
).toBeVisible();
```

This validates user-visible behavior.

---

## 31. Frame Forms

Example:

```ts
const frame =
  page.frameLocator(
    'iframe[title="Profile"]'
  );

await frame
  .getByLabel('First name')
  .fill('Sandeep');

await frame
  .getByLabel('Last name')
  .fill('Damor');

await frame
  .getByRole(
    'button',
    { name: 'Save' }
  )
  .click();
```

---

## 32. Frame Assertions

Example:

```ts
await expect(
  frame.getByRole(
    'heading',
    { name: 'Payment' }
  )
).toBeVisible();
```

Frame content can use normal Playwright assertions.

---

## 33. Frame Screenshots

You can capture the entire page:

```ts
await page.screenshot({
  path: 'page.png'
});
```

For component-level diagnostics, consider screenshots of the relevant locator where supported.

---

## 34. Frame Debugging

When frame interaction fails:

```text
1. Verify iframe exists.
2. Verify iframe selector.
3. Inspect frame URL.
4. Check whether content has loaded.
5. Check whether the target is nested.
6. Check for duplicate frames.
7. Check cross-origin behavior.
8. Use Playwright Inspector.
```

Run:

```bash
npx playwright test --debug
```

---

## 35. Inspecting All Frames

Useful debugging code:

```ts
for (const frame of page.frames()) {
  console.log({
    url: frame.url(),
    name: frame.name()
  });
}
```

This can quickly reveal unexpected frame structures.

Do not leave verbose debugging logs in production CI unless needed.

---

## 36. Waiting for a Frame

Instead of using:

```ts
await page.waitForTimeout(5000);
```

prefer an observable condition:

```ts
const frame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await expect(
  frame.getByText(
    'Payment'
  )
).toBeVisible();
```

This waits for meaningful application state.

---

## 37. Frame Locator and Dynamic iframe Loading

Some applications create iframes only after a user action.

Example:

```ts
await page.getByRole(
  'button',
  { name: 'Checkout' }
).click();

const paymentFrame =
  page.frameLocator(
    'iframe[title="Payment"]'
  );

await expect(
  paymentFrame.getByLabel(
    'Card number'
  )
).toBeVisible();
```

The assertion provides synchronization.

---

## 38. Frames and Page Objects

A Page Object can expose a frame:

```ts
export class PaymentPage {
  constructor(
    private readonly page: Page
  ) {}

  get paymentFrame() {
    return this.page.frameLocator(
      'iframe[title="Payment"]'
    );
  }

  get cardNumber() {
    return this.paymentFrame
      .getByLabel('Card number');
  }

  get payButton() {
    return this.paymentFrame
      .getByRole(
        'button',
        { name: 'Pay' }
      );
  }
}
```

Then:

```ts
await paymentPage.cardNumber
  .fill('...');

await paymentPage.payButton
  .click();
```

---

## 39. Frame Component Object

For reusable embedded widgets:

```ts
class PaymentWidget {
  constructor(
    private readonly frame: FrameLocator
  ) {}

  cardNumber() {
    return this.frame
      .getByLabel('Card number');
  }

  expiry() {
    return this.frame
      .getByLabel('Expiry');
  }

  pay() {
    return this.frame.getByRole(
      'button',
      { name: 'Pay' }
    );
  }
}
```

This is useful when multiple pages contain the same embedded widget.

---

## 40. Frame Fixtures

A custom fixture can provide a reusable frame component.

Conceptually:

```text
Test
 ↓
Payment Fixture
 ↓
Payment Frame
 ↓
Payment Actions
```

This can reduce repeated frame initialization across tests.

---

## 41. Frame and Authentication

If an authentication widget is embedded:

```text
Main Application
      ↓
Login iframe
      ↓
Username
Password
```

Create a dedicated component object or fixture rather than duplicating frame selectors in every test.

---

## 42. Frame and API Validation

A strong test can combine:

```text
Frame UI
   ↓
Submit
   ↓
API Validation
```

Example:

```ts
await paymentFrame
  .getByRole(
    'button',
    { name: 'Pay' }
  )
  .click();

const response =
  await request.get(
    `/api/orders/${orderId}`
  );

expect(response.status())
  .toBe(200);
```

This validates both UI behavior and backend state.

---

## 43. Frame and Network Events

For troubleshooting embedded integrations:

```ts
page.on(
  'request',
  request => {
    console.log(
      request.url()
    );
  }
);
```

Use network logging carefully because requests can contain sensitive data.

Never log credentials, tokens, or payment information.

---

## 44. Frame Failure Diagnostics

When a frame test fails, capture:

```text
Screenshot
Trace
Console messages
Frame URLs
Relevant request failures
```

Playwright configuration can automatically retain screenshots, videos, and traces on failures.

---

## 45. Common Frame Mistakes

### Mistake 1 — Using page locators inside an iframe

Incorrect:

```ts
await page.getByLabel(
  'Card number'
).fill('...');
```

Correct:

```ts
await page.frameLocator(
  'iframe[title="Payment"]'
).getByLabel(
  'Card number'
).fill('...');
```

### Mistake 2 — Assuming frame index

Avoid:

```ts
page.frames()[2]
```

### Mistake 3 — Using arbitrary waits

Avoid:

```ts
await page.waitForTimeout(5000);
```

### Mistake 4 — Ignoring nested frames

Always inspect the frame hierarchy.

### Mistake 5 — Hard-coding third-party implementation details

Prefer stable iframe identifiers and provider-supported test interfaces.

---

## 46. Frame vs Locator Decision

Use:

```text
Need UI interaction?
        ↓
   FrameLocator

Need frame metadata/navigation?
        ↓
      Frame

Need embedded component abstraction?
        ↓
   Component Object

Need reusable lifecycle?
        ↓
      Fixture
```

---

## 47. Enterprise Frame Architecture

A scalable structure can be:

```text
pages/
├── CheckoutPage.ts
└── PaymentWidget.ts

fixtures/
└── payment.fixture.ts

tests/
└── checkout.spec.ts
```

Architecture:

```text
Test
 ↓
CheckoutPage
 ↓
PaymentWidget
 ↓
FrameLocator
 ↓
Embedded UI
```

---

## 48. Frame Locator Best Practices

1. Prefer `frameLocator()` for UI interactions.
2. Use stable iframe selectors.
3. Scope locators inside the frame.
4. Avoid frame indexes.
5. Avoid arbitrary waits.
6. Handle nested frames explicitly.
7. Use Page Objects or Component Objects for reusable widgets.
8. Validate meaningful UI state.
9. Use fixtures for complex reusable frame dependencies.
10. Protect sensitive information when debugging third-party frames.

---

## 49. Production-Quality Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('complete payment', async ({
  page
}) => {
  await page.goto('/checkout');

  await page.getByRole(
    'button',
    { name: 'Checkout' }
  ).click();

  const paymentFrame =
    page.frameLocator(
      'iframe[title="Payment"]'
    );

  await expect(
    paymentFrame.getByText(
      'Secure Payment'
    )
  ).toBeVisible();

  await paymentFrame
    .getByLabel('Card number')
    .fill('4111111111111111');

  await paymentFrame
    .getByLabel('Expiry')
    .fill('12/30');

  await paymentFrame
    .getByLabel('CVV')
    .fill('123');

  await paymentFrame
    .getByRole(
      'button',
      { name: 'Pay' }
    )
    .click();

  await expect(
    page.getByRole('status')
  ).toContainText(
    'Payment successful'
  );
});
```

Use only provider-approved test data in real payment integrations.

---

## 50. Interview Questions

### Q1. What is an iframe?

An iframe embeds another HTML document inside the current page.

### Q2. Why can't normal page locators always find iframe elements?

Because iframe content belongs to a separate document and browsing context.

### Q3. What is `frameLocator()`?

It provides a convenient way to locate and interact with elements inside an iframe.

### Q4. What is the difference between `Frame` and `FrameLocator`?

`Frame` represents a browsing context and provides frame-level APIs. `FrameLocator` is designed for locating elements inside the frame.

### Q5. How do you interact with an input inside an iframe?

```ts
await page
  .frameLocator('iframe')
  .getByLabel('Username')
  .fill('sandeep');
```

### Q6. How do you handle nested frames?

Scope the outer frame first and then locate the inner frame within it.

### Q7. Why should frame indexes be avoided?

Frame ordering can change and does not communicate which business component is being targeted.

### Q8. How should third-party payment frames be tested?

Use the provider's supported test environment, test credentials/data, and stable frame selectors.

### Q9. Can Playwright interact with cross-origin frames?

Playwright can automate frame content available through its browser automation context, while browser security and third-party implementation restrictions still apply.

### Q10. What is the recommended architecture for reusable embedded widgets?

Use a Component Object, optionally exposed through a fixture, with the frame locator encapsulated inside it.

---

## 51. Hands-On Exercises

### Exercise 1 — Basic iframe

Create a test that locates an iframe and verifies text inside it.

### Exercise 2 — Form

Fill username and password fields inside an iframe.

### Exercise 3 — Button

Click a button inside an iframe and verify the result.

### Exercise 4 — Multiple Frames

Create a page with multiple iframes and identify the correct one using a stable selector.

### Exercise 5 — Nested Frames

Automate an element inside a nested iframe structure.

### Exercise 6 — Dynamic iframe

Click a button that creates an iframe and interact with its content.

### Exercise 7 — Frame Page Object

Create:

```text
PaymentPage
```

with a reusable payment frame.

### Exercise 8 — Component Object

Create:

```text
PaymentWidget
```

containing:

```text
cardNumber()
expiry()
cvv()
pay()
```

### Exercise 9 — Frame Fixture

Create a fixture that provides the PaymentWidget.

### Exercise 10 — Debugging

Create a failing frame test and diagnose it using:

```bash
npx playwright test --debug
```

and frame URL logging.

---

## 52. Key Takeaways

- Iframes contain separate document contexts.
- Normal page locators should not be used blindly for iframe content.
- `frameLocator()` is the preferred approach for many UI interactions.
- `Frame` is useful for frame-level operations.
- Use stable iframe selectors.
- Avoid frame indexes.
- Nested frames require explicit scoping.
- Cross-origin frames may have additional browser and provider restrictions.
- Page Objects and Component Objects are excellent for reusable frame widgets.
- Fixtures can provide reusable frame components.
- Web-first assertions provide reliable synchronization.
- Avoid arbitrary `waitForTimeout()` calls.
- Protect sensitive information when debugging embedded third-party content.
- Enterprise frame automation should isolate frame-specific logic from business tests.

---

## 53. Enterprise Frame Automation Challenge

Build a reusable automation framework for:

```text
Checkout Page
     ↓
Payment iframe
     ↓
Card Form
     ↓
Confirmation
```

Requirements:

1. Locate the payment iframe using a stable selector.
2. Use `frameLocator()`.
3. Create a PaymentWidget component.
4. Implement card, expiry, and CVV fields.
5. Implement Pay action.
6. Add web-first assertions.
7. Support a dynamically loaded iframe.
8. Validate the final UI result.
9. Add API validation for the created order.
10. Make the component reusable across multiple checkout tests.

---

## 54. Final Best-Practice Architecture

```text
Test
 │
 ├── Page Object
 │      │
 │      └── Component Object
 │              │
 │              └── FrameLocator
 │                      │
 │                      └── Embedded UI
 │
 └── API Validation
```

This architecture keeps iframe implementation details out of the test cases and makes embedded integrations easier to maintain.

---

## 55. Next Step

**Step 88 — Mastering Playwright Browser Contexts, Pages, Multi-Tab Workflows, Popups, Multiple Windows & Enterprise Session Management**

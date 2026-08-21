# Step 74 --- Mastering Playwright Frames & iFrames: FrameLocator, Nested Frames, Dynamic Frames & Cross-Frame Automation

## 1. Learning Objectives

By the end of this step, you will be able to:

-   Understand what an iframe is and why applications use frames.
-   Understand the difference between a page and a frame.
-   Identify whether an element belongs to the main page or an iframe.
-   Use `frameLocator()` effectively.
-   Access frames using `page.frames()`.
-   Work with frame names and URLs.
-   Handle nested iframes.
-   Handle dynamically loaded iframes.
-   Interact with forms, buttons, dropdowns, checkboxes, and tables
    inside frames.
-   Perform assertions inside frames.
-   Handle multiple frames on the same page.
-   Debug frame-related failures.
-   Build reusable frame-handling methods using Page Object Model.
-   Apply enterprise-level iframe automation practices.

------------------------------------------------------------------------

## 2. What Is an iframe?

An **iframe**, or inline frame, allows one HTML document to be embedded
inside another HTML document.

Conceptually:

``` text
Main Web Page
│
├── Header
├── Navigation
├── Main Content
│
└── iframe
     │
     ├── Login Form
     ├── Input
     ├── Button
     └── Other Content
```

For example:

``` html
<iframe src="https://example.com/payment"></iframe>
```

The browser loads the iframe as a separate document.

Therefore, an element inside the iframe belongs to the iframe's document
context.

------------------------------------------------------------------------

## 3. Why Are iFrames Important in Test Automation?

Modern enterprise applications frequently use iframes for:

-   Payment gateways
-   Embedded reports
-   Rich text editors
-   Video players
-   Third-party widgets
-   Authentication components
-   Document viewers
-   Chat widgets
-   Maps
-   Analytics dashboards
-   Legacy applications
-   Embedded applications

A test may look simple:

``` text
Open application
       ↓
Click Payment
       ↓
Payment iframe appears
       ↓
Enter payment information
       ↓
Submit payment
```

But the payment fields may actually exist inside an iframe.

------------------------------------------------------------------------

## 4. Main Page vs Frame

Consider:

``` html
<html>
    <body>

        <h1>Checkout</h1>

        <iframe src="payment.html"></iframe>

    </body>
</html>
```

The following belongs to the main page:

``` html
<h1>Checkout</h1>
```

The following belongs to the iframe document:

``` html
<input id="cardNumber">
```

The iframe content has its own document context.

------------------------------------------------------------------------

## 5. Understanding the DOM Boundary

Think of the application as:

``` text
Browser Page
│
├── Main Document
│    ├── Header
│    ├── Checkout
│    └── iframe element
│
└── iframe Document
     ├── Card Number
     ├── Expiry
     ├── CVV
     └── Pay
```

This is the most important concept to understand before writing iframe
automation.

------------------------------------------------------------------------

## 6. Creating a Simple iframe Example

Consider:

``` html
<iframe id="paymentFrame">
    <input id="cardNumber">
    <button id="pay">Pay</button>
</iframe>
```

The test needs to identify the frame before interacting with elements
inside it.

------------------------------------------------------------------------

## 7. Playwright's Recommended Approach --- `frameLocator()`

Playwright provides:

``` typescript
page.frameLocator()
```

Example:

``` typescript
const paymentFrame =
    page.frameLocator('#paymentFrame');
```

Then:

``` typescript
await paymentFrame
    .locator('#cardNumber')
    .fill('4111111111111111');
```

And:

``` typescript
await paymentFrame
    .getByRole('button', {
        name: 'Pay'
    })
    .click();
```

This is one of the cleanest ways to automate iframe content.

------------------------------------------------------------------------

## 8. Basic iframe Test

``` typescript
import { test } from '@playwright/test';

test('handle iframe', async ({ page }) => {

    await page.goto('/payment');

    const frame =
        page.frameLocator('#paymentFrame');

    await frame
        .locator('#cardNumber')
        .fill('4111111111111111');

    await frame
        .locator('#expiry')
        .fill('12/30');

    await frame
        .locator('#cvv')
        .fill('123');

    await frame
        .getByRole('button', {
            name: 'Pay'
        })
        .click();
});
```

Notice that we do not need a Selenium-style manual frame switch.

------------------------------------------------------------------------

## 9. Using `getByRole()` Inside an iframe

You can use Playwright's recommended locators inside frames.

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .getByRole('button', {
        name: 'Submit'
    })
    .click();
```

You can also use:

``` typescript
await frame
    .getByRole('textbox', {
        name: 'Card Number'
    })
    .fill('4111111111111111');
```

------------------------------------------------------------------------

## 10. Using `getByLabel()` Inside an iframe

Example:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .getByLabel('Card Number')
    .fill('4111111111111111');

await frame
    .getByLabel('Expiry Date')
    .fill('12/30');

await frame
    .getByLabel('CVV')
    .fill('123');
```

Prefer accessible labels over brittle selectors when available.

------------------------------------------------------------------------

## 11. Using `getByPlaceholder()` Inside an iframe

Example:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .getByPlaceholder('Card number')
    .fill('4111111111111111');
```

Another example:

``` typescript
await frame
    .getByPlaceholder('MM/YY')
    .fill('12/30');
```

------------------------------------------------------------------------

## 12. Using CSS Selectors Inside Frames

CSS is supported:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .locator('#cardNumber')
    .fill('4111111111111111');
```

Or:

``` typescript
await frame
    .locator('input[name="cardNumber"]')
    .fill('4111111111111111');
```

Use semantic locators when practical.

------------------------------------------------------------------------

## 13. Using XPath Inside Frames

XPath is also supported:

``` typescript
await frame
    .locator('//input[@id="cardNumber"]')
    .fill('4111111111111111');
```

Avoid XPath when a stable role, label, text, or test ID is available.

------------------------------------------------------------------------

## 14. Assertions Inside an iframe

Assertions work normally.

``` typescript
import { expect } from '@playwright/test';

const frame =
    page.frameLocator('#paymentFrame');

await expect(
    frame.getByRole('heading', {
        name: 'Payment'
    })
).toBeVisible();
```

Another example:

``` typescript
await expect(
    frame.locator('#successMessage')
).toHaveText(
    'Payment successful'
);
```

------------------------------------------------------------------------

## 15. Verify an Input Value Inside an iframe

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

const cardNumber =
    frame.locator('#cardNumber');

await cardNumber.fill(
    '4111111111111111'
);

await expect(cardNumber)
    .toHaveValue(
        '4111111111111111'
    );
```

------------------------------------------------------------------------

## 16. Handling Multiple iframes

A page may contain multiple frames:

``` text
Main Page
│
├── Header
├── iframe → Advertisement
├── Content
├── iframe → Payment
└── iframe → Chat
```

Identify the correct iframe before interacting with it.

Example:

``` typescript
const paymentFrame =
    page.frameLocator('#paymentFrame');

await paymentFrame
    .getByLabel('Card Number')
    .fill('4111111111111111');
```

Do not blindly select:

``` typescript
page.frameLocator('iframe')
```

when multiple frames exist.

------------------------------------------------------------------------

## 17. Multiple iframe Locators

You can maintain separate frame references:

``` typescript
const paymentFrame =
    page.frameLocator('#paymentFrame');

const chatFrame =
    page.frameLocator('#chatFrame');

const reportFrame =
    page.frameLocator('#reportFrame');
```

Then:

``` typescript
await paymentFrame
    .getByRole('button', {
        name: 'Pay'
    })
    .click();
```

And:

``` typescript
await chatFrame
    .getByPlaceholder(
        'Type a message'
    )
    .fill('Hello');
```

------------------------------------------------------------------------

## 18. Inspecting All Frames Using `page.frames()`

Playwright provides:

``` typescript
page.frames()
```

Example:

``` typescript
const frames = page.frames();

console.log(
    `Total frames: ${frames.length}`
);
```

This includes the main frame and child frames.

------------------------------------------------------------------------

## 19. Understanding `Frame`

Playwright provides a `Frame` object representing a page frame.

Example:

``` typescript
const frames = page.frames();

for (const frame of frames) {
    console.log(frame.url());
}
```

This is extremely useful for debugging.

------------------------------------------------------------------------

## 20. Finding a Frame by URL

Suppose an iframe loads:

``` text
https://example.com/payment
```

You can inspect frames:

``` typescript
const frame =
    page.frames().find(frame =>
        frame.url().includes('/payment')
    );
```

Then:

``` typescript
if (!frame) {
    throw new Error(
        'Payment frame not found'
    );
}
```

------------------------------------------------------------------------

## 21. Finding a Frame by Name

If the iframe has:

``` html
<iframe name="paymentFrame">
```

you can find it:

``` typescript
const frame = page.frame({
    name: 'paymentFrame'
});
```

Then:

``` typescript
if (!frame) {
    throw new Error(
        'Payment frame not found'
    );
}
```

------------------------------------------------------------------------

## 22. Frame Name vs Frame Locator

### `frameLocator()`

Best for direct interaction:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .locator('#cardNumber')
    .fill('1234');
```

### `page.frame()`

Useful when you need the actual `Frame` object:

``` typescript
const frame = page.frame({
    name: 'paymentFrame'
});
```

For most UI interactions, `frameLocator()` is simpler.

------------------------------------------------------------------------

## 23. Nested iFrames

Some applications contain nested frames:

``` text
Main Page
│
└── iframe A
     │
     └── iframe B
          │
          └── Button
```

Example:

``` typescript
const outerFrame =
    page.frameLocator('#outerFrame');

const innerFrame =
    outerFrame.frameLocator(
        '#innerFrame'
    );

await innerFrame
    .getByRole('button', {
        name: 'Submit'
    })
    .click();
```

------------------------------------------------------------------------

## 24. Nested iframe Architecture

A complex enterprise application may look like:

``` text
Browser
│
├── Main Application
│
└── iframe
     │
     ├── Header
     │
     └── iframe
          │
          ├── Form
          └── iframe
               │
               └── Submit Button
```

The automation follows the same hierarchy.

------------------------------------------------------------------------

## 25. Dynamic iFrames

Sometimes the iframe does not exist immediately.

For example:

``` text
Page loads
   ↓
JavaScript executes
   ↓
Payment widget initializes
   ↓
iframe created
   ↓
iframe content loads
```

Avoid:

``` typescript
await page.waitForTimeout(5000);
```

Instead, use locator-based synchronization:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .getByLabel('Card Number')
    .waitFor();
```

------------------------------------------------------------------------

## 26. Why `waitForTimeout()` Is Usually a Bad Solution

Avoid:

``` typescript
await page.waitForTimeout(5000);
```

because:

-   It slows tests.
-   It creates unnecessary delays.
-   Five seconds may be too short.
-   Five seconds may be unnecessarily long.
-   It does not guarantee application readiness.

Prefer condition-based waiting.

------------------------------------------------------------------------

## 27. Dynamic iframe Example

``` typescript
const paymentFrame =
    page.frameLocator('#paymentFrame');

await paymentFrame
    .getByLabel('Card Number')
    .fill('4111111111111111');
```

Playwright waits for the target locator to become actionable.

------------------------------------------------------------------------

## 28. Detecting if an iframe Exists

You can check the iframe element:

``` typescript
const iframe =
    page.locator('#paymentFrame');

await expect(iframe)
    .toBeVisible();
```

Then interact with it:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .getByLabel('Card Number')
    .fill('4111');
```

------------------------------------------------------------------------

## 29. iframe URL Validation

When debugging:

``` typescript
for (const frame of page.frames()) {
    console.log(
        'Frame URL:',
        frame.url()
    );
}
```

Output might look like:

``` text
Frame URL: https://example.com/checkout
Frame URL: https://payment.example.com/widget
Frame URL: https://chat.example.com/widget
```

------------------------------------------------------------------------

## 30. Debugging Frame Problems

When an iframe test fails:

### Step 1 --- Check whether iframe exists

``` typescript
console.log(
    await page.locator('iframe').count()
);
```

### Step 2 --- Inspect frame URLs

``` typescript
for (const frame of page.frames()) {
    console.log(frame.url());
}
```

### Step 3 --- Verify iframe selector.

### Step 4 --- Verify element selector inside iframe.

### Step 5 --- Check whether the iframe is dynamically loaded.

------------------------------------------------------------------------

## 31. Counting iFrames

``` typescript
const count =
    await page.locator('iframe').count();

console.log(
    `Iframe count: ${count}`
);
```

This is useful during test development and debugging.

------------------------------------------------------------------------

## 32. Handling an iframe by `src`

Example:

``` html
<iframe
    src="https://payment.example.com/widget">
</iframe>
```

You can use:

``` typescript
const frame =
    page.frameLocator(
        'iframe[src*="payment"]'
    );
```

Then:

``` typescript
await frame
    .getByLabel('Card Number')
    .fill('4111111111111111');
```

------------------------------------------------------------------------

## 33. Handling iframe by Title

If the iframe has:

``` html
<iframe title="Payment Form">
```

Use:

``` typescript
const frame =
    page.frameLocator(
        'iframe[title="Payment Form"]'
    );
```

This can be more readable than positional selectors.

------------------------------------------------------------------------

## 34. Handling iframe by `data-testid`

If your application provides:

``` html
<iframe data-testid="payment-frame">
```

Use:

``` typescript
const frame =
    page.frameLocator(
        '[data-testid="payment-frame"]'
    );
```

This is often a strong enterprise locator strategy.

------------------------------------------------------------------------

## 35. Avoid `.nth()` When Possible

Avoid:

``` typescript
page.frameLocator('iframe').nth(2)
```

because iframe ordering can change.

Prefer:

``` typescript
page.frameLocator(
    '[data-testid="payment-frame"]'
);
```

or:

``` typescript
page.frameLocator(
    'iframe[title="Payment Form"]'
);
```

------------------------------------------------------------------------

## 36. iframe Inside a Page Object Model

A Page Object can encapsulate frame handling.

``` typescript
import { Page } from '@playwright/test';

export class PaymentPage {

    constructor(
        private readonly page: Page
    ) {}

    private paymentFrame() {
        return this.page.frameLocator(
            '#paymentFrame'
        );
    }

    async enterCardNumber(
        cardNumber: string
    ) {
        await this.paymentFrame()
            .getByLabel('Card Number')
            .fill(cardNumber);
    }

    async enterExpiry(
        expiry: string
    ) {
        await this.paymentFrame()
            .getByLabel('Expiry Date')
            .fill(expiry);
    }

    async enterCvv(
        cvv: string
    ) {
        await this.paymentFrame()
            .getByLabel('CVV')
            .fill(cvv);
    }

    async submitPayment() {
        await this.paymentFrame()
            .getByRole('button', {
                name: 'Pay'
            })
            .click();
    }
}
```

------------------------------------------------------------------------

## 37. Using the Payment Page Object

``` typescript
import { test } from '@playwright/test';

test('payment using iframe', async ({ page }) => {

    const paymentPage =
        new PaymentPage(page);

    await page.goto('/checkout');

    await paymentPage
        .enterCardNumber(
            '4111111111111111'
        );

    await paymentPage
        .enterExpiry('12/30');

    await paymentPage
        .enterCvv('123');

    await paymentPage
        .submitPayment();
});
```

The test remains clean and business-focused.

------------------------------------------------------------------------

## 38. Frame Component Object

For reusable iframe components, create a dedicated component.

``` typescript
import {
    FrameLocator,
    Page
} from '@playwright/test';

export class PaymentFrame {

    private readonly frame: FrameLocator;

    constructor(page: Page) {
        this.frame =
            page.frameLocator(
                '#paymentFrame'
            );
    }

    async fillCardNumber(
        value: string
    ) {
        await this.frame
            .getByLabel('Card Number')
            .fill(value);
    }

    async fillExpiry(
        value: string
    ) {
        await this.frame
            .getByLabel('Expiry Date')
            .fill(value);
    }

    async fillCvv(
        value: string
    ) {
        await this.frame
            .getByLabel('CVV')
            .fill(value);
    }
}
```

This is a good enterprise architecture pattern.

------------------------------------------------------------------------

## 39. FrameLocator Type

You can import:

``` typescript
import {
    FrameLocator
} from '@playwright/test';
```

Then:

``` typescript
private readonly frame: FrameLocator;
```

This provides strong TypeScript typing.

------------------------------------------------------------------------

## 40. Frame vs FrameLocator --- Important Difference

### Frame

Represents the actual browser frame.

Useful for:

-   URL inspection
-   Frame-level operations
-   Evaluating JavaScript
-   Accessing frame metadata

### FrameLocator

Represents a locator for elements inside a frame.

Useful for:

-   Locating elements
-   Clicking
-   Filling
-   Assertions
-   User interactions

------------------------------------------------------------------------

## 41. Practical Comparison

  Requirement                   Recommended API
  ----------------------------- ---------------------------------
  Click element inside iframe   `frameLocator()`
  Fill field inside iframe      `frameLocator()`
  Assert iframe element         `frameLocator()`
  Find frame by name            `page.frame()`
  Find frame by URL             `page.frames()`
  Inspect all frames            `page.frames()`
  Work with nested frames       `frameLocator().frameLocator()`

------------------------------------------------------------------------

## 42. Common Mistake --- Treating iframe Content as Main Page Content

Incorrect:

``` typescript
await page
    .getByLabel('Card Number')
    .fill('4111');
```

Correct:

``` typescript
const frame =
    page.frameLocator('#paymentFrame');

await frame
    .getByLabel('Card Number')
    .fill('4111');
```

------------------------------------------------------------------------

## 43. Common Mistake --- Using `page.locator()` for Inner Elements

Incorrect:

``` typescript
await page
    .locator('#cardNumber')
    .fill('4111');
```

Correct:

``` typescript
await page
    .frameLocator('#paymentFrame')
    .locator('#cardNumber')
    .fill('4111');
```

------------------------------------------------------------------------

## 44. Common Mistake --- Using Arbitrary Sleeps

Avoid:

``` typescript
await page.waitForTimeout(3000);
```

Use condition-based locator operations instead.

------------------------------------------------------------------------

## 45. Common Mistake --- Using `nth()` Everywhere

Avoid:

``` typescript
page.frameLocator('iframe').nth(1)
```

Prefer:

``` typescript
page.frameLocator(
    'iframe[title="Payment Form"]'
);
```

------------------------------------------------------------------------

## 46. Common Mistake --- Assuming Every iframe Is Automatable

Some third-party widgets may have unusual loading behavior.

Before building automation, verify:

-   iframe exists
-   iframe is accessible
-   required elements are exposed
-   application security restrictions
-   authentication behavior
-   dynamic loading behavior

------------------------------------------------------------------------

## 47. Cross-Origin iframe Concept

An iframe can load content from another domain.

Example:

``` text
Application:
https://shop.example.com

iframe:
https://payment.example.com
```

This is a cross-origin iframe.

Playwright provides frame APIs for interacting with frame content,
subject to the application's architecture and browser security
boundaries.

------------------------------------------------------------------------

## 48. Third-Party Payment iframe Example

Typical architecture:

``` text
Shopping Application
│
├── Product Page
├── Cart
├── Checkout
│
└── Payment Provider iframe
      │
      ├── Card Number
      ├── Expiry
      ├── CVV
      └── Submit
```

Automation:

``` typescript
const paymentFrame =
    page.frameLocator(
        'iframe[title="Payment Form"]'
    );

await paymentFrame
    .getByLabel('Card Number')
    .fill('4111111111111111');
```

------------------------------------------------------------------------

## 49. Enterprise Best Practices

Follow these principles:

### 1. Prefer `frameLocator()`

Use it for normal iframe interactions.

### 2. Use stable iframe selectors

Prefer:

-   `data-testid`
-   title
-   stable id
-   stable attributes

### 3. Avoid positional iframe selection

Avoid unnecessary:

``` typescript
.nth()
```

### 4. Encapsulate frames

Keep frame details inside Page Objects or Components.

### 5. Use semantic locators inside frames

Prefer:

``` typescript
getByRole()
getByLabel()
getByPlaceholder()
getByText()
```

### 6. Avoid fixed sleeps

Use condition-based synchronization.

------------------------------------------------------------------------

## 50. Enterprise Frame Architecture

A scalable framework might look like:

``` text
pages/
│
├── checkout/
│    ├── CheckoutPage.ts
│    └── PaymentFrame.ts
│
├── admin/
│    └── AdminDashboardPage.ts
│
└── components/
     ├── Header.ts
     └── ChatFrame.ts
```

This keeps frame implementation details isolated.

------------------------------------------------------------------------

## 51. Real-World Test Flow

``` text
Login
  ↓
Product Selection
  ↓
Add to Cart
  ↓
Checkout
  ↓
Payment iframe
  ↓
Enter Card
  ↓
Submit Payment
  ↓
Payment Confirmation
  ↓
Order Confirmation
```

This is a typical end-to-end automation scenario.

------------------------------------------------------------------------

## 52. Complete Example

``` typescript
import {
    test,
    expect
} from '@playwright/test';

test(
    'complete payment through iframe',
    async ({ page }) => {

        await page.goto('/checkout');

        const paymentFrame =
            page.frameLocator(
                '#paymentFrame'
            );

        await paymentFrame
            .getByLabel('Card Number')
            .fill('4111111111111111');

        await paymentFrame
            .getByLabel('Expiry Date')
            .fill('12/30');

        await paymentFrame
            .getByLabel('CVV')
            .fill('123');

        await paymentFrame
            .getByRole('button', {
                name: 'Pay'
            })
            .click();

        await expect(
            page.getByText(
                'Payment successful'
            )
        ).toBeVisible();
    }
);
```

------------------------------------------------------------------------

## 53. Debugging Checklist

When iframe automation fails, check:

``` text
☐ Is the iframe present?
☐ Is the iframe selector correct?
☐ Is the iframe dynamically created?
☐ Is the target element inside the iframe?
☐ Is there more than one iframe?
☐ Is the iframe nested?
☐ Is the element selector correct?
☐ Is the iframe loaded?
☐ Is the iframe URL what you expected?
☐ Is the third-party component available in the test environment?
```

------------------------------------------------------------------------

## 54. Interview Questions

### Beginner

1.  What is an iframe?
2.  Why do we need special handling for iframe elements?
3.  What is `frameLocator()`?
4.  How do you interact with an element inside an iframe?
5.  How do you count iframes on a page?

### Intermediate

6.  What is the difference between `Frame` and `FrameLocator`?
7.  How do you find a frame by name?
8.  How do you find a frame by URL?
9.  How do you handle multiple iframes?
10. How do you handle nested iframes?

### Advanced

11. How would you design reusable iframe handling in Page Object Model?
12. How do you debug dynamically loaded iframes?
13. How would you automate a third-party payment iframe?
14. How do you avoid brittle iframe selectors?
15. What challenges can cross-origin iframe applications introduce?

### Expert

16. How would you architect iframe handling across a large enterprise
    Playwright framework?
17. How would you design a reusable `FrameComponent` abstraction?
18. How would you troubleshoot intermittent iframe failures in CI?
19. How would you handle nested dynamic frames?
20. What logging and diagnostics would you add for iframe failures?

------------------------------------------------------------------------

## 55. Knowledge Check

### Question 1

What is the preferred Playwright API for interacting with elements
inside an iframe?

### Question 2

How would you fill a textbox inside:

``` html
<iframe id="loginFrame">
```

where the textbox has:

``` html
<input id="username">
```

### Question 3

How can you inspect all frames?

### Question 4

How do you handle a nested iframe?

### Question 5

Why should you avoid:

``` typescript
page.waitForTimeout(5000);
```

for iframe synchronization?

------------------------------------------------------------------------

## 56. Mini Challenge

Create a Page Object called:

``` text
PaymentFrame.ts
```

It should contain:

``` typescript
fillCardNumber()
fillExpiry()
fillCvv()
clickPay()
verifyPaymentSuccess()
```

Then create:

``` text
payment.spec.ts
```

and automate:

``` text
Open Checkout
       ↓
Access Payment iframe
       ↓
Enter card details
       ↓
Submit payment
       ↓
Verify payment success
```

Use:

-   TypeScript
-   Playwright Test
-   Page Object Model
-   `FrameLocator`
-   Semantic locators
-   Assertions
-   No arbitrary `waitForTimeout()`

------------------------------------------------------------------------

## 57. Key Takeaways

The core concept is:

> **An iframe is a separate document context embedded inside a web
> page.**

For normal iframe interaction, Playwright's:

``` typescript
page.frameLocator()
```

is usually the cleanest approach.

The fundamental pattern is:

``` typescript
const frame =
    page.frameLocator(
        'iframe-selector'
    );

await frame
    .locator('element-selector')
    .click();
```

For nested frames:

``` typescript
const outer =
    page.frameLocator('#outerFrame');

const inner =
    outer.frameLocator('#innerFrame');

await inner
    .getByRole('button')
    .click();
```

For frame inspection:

``` typescript
page.frames()
```

is extremely useful.

------------------------------------------------------------------------

## 58. Milestone

### 🎯 Milestone: Advanced Browser Interaction

At this point, you should be comfortable automating:

-   Forms
-   Date pickers
-   Dialogs
-   Alerts
-   Popups
-   Browser events
-   iFrames
-   Nested frames
-   Dynamic frame content

The next lessons will progressively move toward more complex browser
workflows such as multiple tabs/windows, authentication/session
handling, file operations, and advanced interaction patterns.

------------------------------------------------------------------------

# Step 75 Preview

**Step 75 --- Mastering Multiple Windows, Tabs, Popups & Browser Pages
in Playwright**

You will learn how Playwright handles multiple `Page` objects, new tabs,
popup windows, `context.waitForEvent('page')`,
`page.waitForEvent('popup')`, multi-tab workflows, authentication
windows, OAuth flows, and enterprise-level multi-page automation.

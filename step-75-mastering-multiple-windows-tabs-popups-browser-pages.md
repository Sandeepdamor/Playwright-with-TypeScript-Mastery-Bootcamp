# Step 75 --- Mastering Multiple Windows, Tabs, Popups & Browser Pages in Playwright

## 1. Learning Objectives

By the end of this step, you will be able to:

-   Understand the difference between a browser, context, page, tab, and
    popup.
-   Understand how Playwright represents multiple browser pages.
-   Handle links that open new tabs.
-   Handle links that open new browser windows.
-   Handle JavaScript popup windows.
-   Use `browserContext.waitForEvent('page')`.
-   Use `page.waitForEvent('popup')`.
-   Capture newly opened pages reliably.
-   Interact with elements on newly opened pages.
-   Handle multiple tabs simultaneously.
-   Handle authentication and OAuth-style popup flows.
-   Close child pages safely.
-   Return to the original page.
-   Build reusable multi-page handling methods.
-   Apply Page Object Model to multi-tab workflows.
-   Debug flaky multi-page tests.
-   Design enterprise-grade multi-window automation.

------------------------------------------------------------------------

## 2. Why Multiple Pages Matter in Playwright

Modern web applications frequently open additional pages.

Examples include:

-   Help documentation
-   Terms and conditions
-   Payment providers
-   Authentication providers
-   Reports
-   External applications
-   Customer portals
-   Admin portals
-   OAuth authentication
-   Social login
-   PDF/report viewers
-   External dashboards

A typical workflow may look like:

``` text
Main Application
       ↓
Click "Open Report"
       ↓
New Tab Opens
       ↓
Report Page
       ↓
Validate Report
       ↓
Close Report
       ↓
Return to Main Application
```

Playwright provides powerful APIs for handling these scenarios.

------------------------------------------------------------------------

## 3. Understanding Browser Hierarchy

Before learning multiple tabs, understand Playwright's browser
hierarchy.

``` text
Browser
│
├── BrowserContext
│    │
│    ├── Page 1
│    ├── Page 2
│    └── Page 3
│
└── BrowserContext
     │
     ├── Page 1
     └── Page 2
```

The important relationship is:

``` text
Browser
   ↓
BrowserContext
   ↓
Page
```

A browser context can contain multiple pages.

------------------------------------------------------------------------

## 4. What Is a Page?

In Playwright:

``` typescript
Page
```

represents a browser tab or browser page.

For example:

``` typescript
const newPage = await context.newPage();
```

creates a new page.

A user may visually think of it as a new tab, but Playwright represents
it as a `Page`.

------------------------------------------------------------------------

## 5. Multiple Tabs Are Multiple `Page` Objects

Suppose the application has:

``` text
Tab 1 → Application
Tab 2 → Documentation
Tab 3 → Report
```

Playwright can represent them as:

``` text
Page 1
Page 2
Page 3
```

All may belong to the same browser context.

------------------------------------------------------------------------

## 6. Getting All Pages

You can retrieve all pages from a browser context:

``` typescript
const pages = context.pages();
```

Example:

``` typescript
console.log(`Total pages: ${pages.length}`);
```

This is useful when debugging multi-page scenarios.

------------------------------------------------------------------------

## 7. Creating a New Page Manually

You can create a new page:

``` typescript
const newPage = await context.newPage();
```

Then navigate:

``` typescript
await newPage.goto('https://example.com');
```

This is useful when the test itself needs multiple independent pages.

------------------------------------------------------------------------

## 8. Example --- Two Independent Pages

``` typescript
import { test } from '@playwright/test';

test('multiple pages', async ({ context }) => {

    const page1 = await context.newPage();
    const page2 = await context.newPage();

    await page1.goto('https://example.com');
    await page2.goto('https://playwright.dev');

    console.log(await page1.title());
    console.log(await page2.title());
});
```

Here:

``` text
Page 1 → example.com
Page 2 → playwright.dev
```

------------------------------------------------------------------------

## 9. New Tab vs New Window

From Playwright's perspective, both are generally represented as:

``` typescript
Page
```

The browser may visually open:

``` text
New Tab
```

or:

``` text
New Window
```

but Playwright handles the new browser document as a new `Page`.

Therefore, the automation approach is usually the same.

------------------------------------------------------------------------

## 10. The Most Important Rule

When an action opens a new page, do not blindly click first and then try
to discover the new page afterward.

Instead, wait for the expected event while performing the action.

Use:

``` typescript
context.waitForEvent('page')
```

or:

``` typescript
page.waitForEvent('popup')
```

This prevents race conditions.

------------------------------------------------------------------------

## 11. Handling a New Tab with `context.waitForEvent('page')`

Suppose:

``` text
Click "Open Report"
        ↓
New Tab
```

Use:

``` typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByRole('link', {
    name: 'Open Report'
}).click();

const newPage =
    await newPagePromise;
```

Now:

``` typescript
await newPage.waitForLoadState();
```

------------------------------------------------------------------------

## 12. Complete New Tab Example

``` typescript
import { test, expect } from '@playwright/test';

test('handle new tab', async ({ page, context }) => {

    await page.goto('/dashboard');

    const newPagePromise =
        context.waitForEvent('page');

    await page.getByRole('link', {
        name: 'Open Report'
    }).click();

    const newPage =
        await newPagePromise;

    await newPage.waitForLoadState();

    await expect(newPage).toHaveTitle(
        /Report/
    );

});
```

------------------------------------------------------------------------

## 13. Why Promise Setup Happens Before the Click

This is extremely important.

Correct:

``` typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByRole('link', {
    name: 'Open Report'
}).click();

const newPage =
    await newPagePromise;
```

Incorrect:

``` typescript
await page.getByRole('link', {
    name: 'Open Report'
}).click();

const newPagePromise =
    context.waitForEvent('page');
```

The new page may open immediately after the click.

By the time you start listening for the event, the event may already
have occurred.

------------------------------------------------------------------------

## 14. Understanding the Event Flow

The correct sequence is:

``` text
Register listener
       ↓
Perform action
       ↓
Browser opens page
       ↓
Playwright captures event
       ↓
Await new Page
       ↓
Interact with new Page
```

This pattern is extremely important in Playwright.

------------------------------------------------------------------------

## 15. Using `page.waitForEvent('popup')`

A popup is usually created from the current page through JavaScript.

For example:

``` javascript
window.open(...)
```

Playwright provides:

``` typescript
page.waitForEvent('popup')
```

Example:

``` typescript
const popupPromise =
    page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Open Popup'
}).click();

const popup =
    await popupPromise;
```

------------------------------------------------------------------------

## 16. Popup Example

``` typescript
import { test, expect } from '@playwright/test';

test('handle popup', async ({ page }) => {

    await page.goto('/dashboard');

    const popupPromise =
        page.waitForEvent('popup');

    await page.getByRole('button', {
        name: 'Open Details'
    }).click();

    const popup =
        await popupPromise;

    await popup.waitForLoadState();

    await expect(
        popup.getByRole('heading', {
            name: 'Details'
        })
    ).toBeVisible();
});
```

------------------------------------------------------------------------

## 17. Understanding `popup`

A popup is typically a page opened by the current page.

For example:

``` javascript
window.open('/details');
```

The browser creates another page.

Playwright can capture it using:

``` typescript
page.waitForEvent('popup')
```

------------------------------------------------------------------------

## 18. New Page vs Popup

Use:

### `context.waitForEvent('page')`

When you want to detect a newly created page in the browser context.

``` typescript
context.waitForEvent('page')
```

Use:

### `page.waitForEvent('popup')`

When the new page is specifically opened by the current page.

``` typescript
page.waitForEvent('popup')
```

------------------------------------------------------------------------

## 19. Opening a Link in a New Tab

Suppose HTML contains:

``` html
<a
    href="/terms"
    target="_blank">
    Terms and Conditions
</a>
```

Automation:

``` typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByRole('link', {
    name: 'Terms and Conditions'
}).click();

const termsPage =
    await newPagePromise;

await termsPage.waitForLoadState();
```

------------------------------------------------------------------------

## 20. Validating the New Tab URL

``` typescript
await expect(termsPage)
    .toHaveURL(/terms/);
```

Or:

``` typescript
console.log(
    await termsPage.url()
);
```

------------------------------------------------------------------------

## 21. Validating New Tab Title

``` typescript
await expect(termsPage)
    .toHaveTitle(/Terms/);
```

This verifies that the correct page was opened.

------------------------------------------------------------------------

## 22. Interacting With the New Page

Once you have the new `Page` object, interact with it normally.

``` typescript
await expect(
    termsPage.getByRole('heading', {
        name: 'Terms and Conditions'
    })
).toBeVisible();
```

You can use:

-   `locator()`
-   `getByRole()`
-   `getByText()`
-   `getByLabel()`
-   `getByPlaceholder()`
-   Assertions
-   Keyboard actions
-   Mouse actions
-   Navigation

------------------------------------------------------------------------

## 23. Returning to the Original Page

You do not need a special "switch tab" command.

Simply retain the original page reference.

``` typescript
const mainPage = page;

const newPagePromise =
    context.waitForEvent('page');

await mainPage
    .getByRole('link', {
        name: 'Report'
    })
    .click();

const reportPage =
    await newPagePromise;

await reportPage.waitForLoadState();

await reportPage.close();

await expect(
    mainPage.getByRole('heading', {
        name: 'Dashboard'
    })
).toBeVisible();
```

The original `page` object remains available.

------------------------------------------------------------------------

## 24. There Is No Selenium-Style Window Handle Requirement

In Selenium, engineers often work with:

``` text
window handles
```

and manually switch:

``` text
driver.switchTo().window(...)
```

Playwright's model is different.

Instead of manually switching window handles:

``` text
Page 1
Page 2
Page 3
```

you hold references to the specific `Page` objects.

This is one of Playwright's major advantages.

------------------------------------------------------------------------

## 25. Comparing Selenium and Playwright

  Selenium                   Playwright
  -------------------------- -----------------------------
  Window handles             Page objects
  `switchTo().window()`      Use the target `Page`
  Manual handle management   Event-based page management
  Explicit switching         Direct page reference
  More state management      Simpler multi-page model

------------------------------------------------------------------------

## 26. Getting Pages From Context

You can inspect:

``` typescript
const pages =
    context.pages();
```

Example:

``` typescript
for (const currentPage of pages) {
    console.log(
        'URL:',
        currentPage.url()
    );
}
```

This is very useful when debugging unexpected tabs.

------------------------------------------------------------------------

## 27. Finding a Specific Page

Suppose multiple pages are open.

You can find a page by URL:

``` typescript
const reportPage =
    context.pages().find(page =>
        page.url().includes('/report')
    );
```

Then:

``` typescript
if (!reportPage) {
    throw new Error(
        'Report page not found'
    );
}
```

------------------------------------------------------------------------

## 28. Waiting for a Specific New Page

Sometimes multiple pages can be opened.

Use a predicate:

``` typescript
const reportPage =
    await context.waitForEvent(
        'page',
        page =>
            page.url().includes('/report')
    );
```

This helps identify the correct page.

------------------------------------------------------------------------

## 29. Handling Multiple New Pages

Imagine:

``` text
Click
 ↓
Page A
Page B
Page C
```

You can inspect:

``` typescript
const pages =
    context.pages();

for (const currentPage of pages) {
    console.log(
        await currentPage.title()
    );
}
```

Or use event predicates to capture the expected page.

------------------------------------------------------------------------

## 30. Waiting for Page Load

After capturing a new page:

``` typescript
const newPage =
    await newPagePromise;
```

you can wait for a desired state:

``` typescript
await newPage.waitForLoadState();
```

Possible states include:

``` typescript
'domcontentloaded'
```

``` typescript
'load'
```

and:

``` typescript
'networkidle'
```

Use the least strict state that provides reliable synchronization for
your application.

------------------------------------------------------------------------

## 31. `domcontentloaded`

Example:

``` typescript
await newPage.waitForLoadState(
    'domcontentloaded'
);
```

This waits until the initial HTML document has been loaded and parsed.

------------------------------------------------------------------------

## 32. `load`

Example:

``` typescript
await newPage.waitForLoadState(
    'load'
);
```

This waits for the page load event.

------------------------------------------------------------------------

## 33. `networkidle`

Example:

``` typescript
await newPage.waitForLoadState(
    'networkidle'
);
```

This waits for network activity to become idle.

Do not use `networkidle` automatically for every test.

Modern applications may continuously make network requests.

------------------------------------------------------------------------

## 34. Popup That Does Not Immediately Load

Some popups may initially have:

``` text
about:blank
```

and then navigate.

Use:

``` typescript
const popupPromise =
    page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Open Report'
}).click();

const popup =
    await popupPromise;

await popup.waitForLoadState();

console.log(
    await popup.url()
);
```

------------------------------------------------------------------------

## 35. Authentication Popup Example

A common enterprise flow:

``` text
Application
     ↓
Login with SSO
     ↓
Authentication Popup
     ↓
Enter Credentials
     ↓
Authenticate
     ↓
Popup Closes
     ↓
Return to Application
```

Example:

``` typescript
const authPopupPromise =
    page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Login with SSO'
}).click();

const authPopup =
    await authPopupPromise;

await authPopup.waitForLoadState();

await authPopup
    .getByLabel('Username')
    .fill('testuser');
```

------------------------------------------------------------------------

## 36. OAuth-Style Popup Workflow

Typical flow:

``` text
Main Application
       ↓
Click Google/Microsoft/SSO
       ↓
Authentication Popup
       ↓
Identity Provider
       ↓
Authentication
       ↓
Popup closes
       ↓
Main Application
```

Playwright can manage the popup as a separate `Page`.

------------------------------------------------------------------------

## 37. Handling a Popup That Closes Automatically

Example:

``` typescript
const popupPromise =
    page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Authenticate'
}).click();

const popup =
    await popupPromise;

await popup.waitForLoadState();

await popup
    .getByLabel('Username')
    .fill('testuser');

await popup
    .getByRole('button', {
        name: 'Continue'
    })
    .click();
```

The popup may then close automatically.

The original page remains available.

------------------------------------------------------------------------

## 38. Detecting Popup Closure

You can listen for:

``` typescript
popup.on('close', () => {
    console.log('Popup closed');
});
```

Example:

``` typescript
const popupPromise =
    page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Authenticate'
}).click();

const popup =
    await popupPromise;

popup.on('close', () => {
    console.log('Authentication popup closed');
});
```

------------------------------------------------------------------------

## 39. Closing a Page Manually

You can close a child page:

``` typescript
await newPage.close();
```

Then verify:

``` typescript
console.log(
    newPage.isClosed()
);
```

------------------------------------------------------------------------

## 40. Checking Whether a Page Is Closed

``` typescript
if (newPage.isClosed()) {
    console.log('Page is closed');
}
```

This can be useful in complex multi-page workflows.

------------------------------------------------------------------------

## 41. Multi-Tab Test Example

``` typescript
test('multiple tabs', async ({
    page,
    context
}) => {

    await page.goto('/dashboard');

    const reportPromise =
        context.waitForEvent('page');

    await page.getByRole('link', {
        name: 'Report'
    }).click();

    const reportPage =
        await reportPromise;

    await reportPage.waitForLoadState();

    await expect(reportPage)
        .toHaveTitle(/Report/);

    await expect(
        reportPage.getByRole('heading', {
            name: 'Sales Report'
        })
    ).toBeVisible();

    await reportPage.close();

    await expect(page)
        .toHaveURL(/dashboard/);
});
```

------------------------------------------------------------------------

## 42. Handling Multiple Pages Created Before the Test

Sometimes your application may already have multiple pages.

You can inspect:

``` typescript
const pages =
    context.pages();

console.log(
    `Pages: ${pages.length}`
);
```

Then:

``` typescript
for (const currentPage of pages) {
    console.log(
        await currentPage.title()
    );
}
```

------------------------------------------------------------------------

## 43. Page Identification Strategy

In enterprise frameworks, identify pages using:

1.  URL
2.  Title
3.  Unique page element
4.  Application-specific identifier

Example:

``` typescript
const reportPage =
    context.pages().find(page =>
        page.url().includes('/reports')
    );
```

Then validate:

``` typescript
if (!reportPage) {
    throw new Error(
        'Expected report page was not found'
    );
}
```

------------------------------------------------------------------------

## 44. Avoid Blind Page Selection

Avoid:

``` typescript
const pages = context.pages();

const secondPage = pages[1];
```

This is fragile.

Page ordering can change.

Prefer:

``` typescript
const reportPage =
    context.pages().find(page =>
        page.url().includes('/reports')
    );
```

------------------------------------------------------------------------

## 45. Using a Predicate With `waitForEvent`

A powerful pattern is:

``` typescript
const reportPage =
    await context.waitForEvent(
        'page',
        page => page.url().includes(
            '/reports'
        )
    );
```

This allows your test to capture a specific page.

------------------------------------------------------------------------

## 46. Popup Predicate

Similarly:

``` typescript
const popup =
    await page.waitForEvent(
        'popup',
        popup =>
            popup.url().includes('/login')
    );
```

This is useful when several popup-related operations exist.

------------------------------------------------------------------------

## 47. Handling New Page and Assertion Together

Example:

``` typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByText('Open Report').click();

const reportPage =
    await newPagePromise;

await expect(reportPage)
    .toHaveURL(/report/);
```

This makes the test concise.

------------------------------------------------------------------------

## 48. Multi-Page Page Object Model

You can create a page object for the report.

``` typescript
import {
    Page
} from '@playwright/test';

export class ReportPage {

    constructor(
        private readonly page: Page
    ) {}

    async verifyLoaded() {
        await this.page
            .getByRole('heading', {
                name: 'Sales Report'
            })
            .isVisible();
    }

    async getUrl() {
        return this.page.url();
    }

    async close() {
        await this.page.close();
    }
}
```

------------------------------------------------------------------------

## 49. Creating a Multi-Page Helper

A reusable helper can encapsulate new-page handling.

``` typescript
import {
    BrowserContext,
    Page
} from '@playwright/test';

export async function
waitForNewPage(
    context: BrowserContext,
    action: () => Promise<void>
): Promise<Page> {

    const pagePromise =
        context.waitForEvent('page');

    await action();

    return await pagePromise;
}
```

------------------------------------------------------------------------

## 50. Using the Multi-Page Helper

``` typescript
const reportPage =
    await waitForNewPage(
        context,
        async () => {
            await page.getByText(
                'Open Report'
            ).click();
        }
    );

await reportPage.waitForLoadState();
```

This can reduce repeated multi-page code across a framework.

------------------------------------------------------------------------

## 51. Generic Popup Helper

You can also create:

``` typescript
import { Page } from '@playwright/test';

export async function
waitForPopup(
    page: Page,
    action: () => Promise<void>
): Promise<Page> {

    const popupPromise =
        page.waitForEvent('popup');

    await action();

    return await popupPromise;
}
```

Then:

``` typescript
const popup =
    await waitForPopup(
        page,
        async () => {
            await page.getByText(
                'Open Popup'
            ).click();
        }
    );
```

------------------------------------------------------------------------

## 52. Enterprise Framework Structure

A scalable Playwright framework might contain:

``` text
pages/
│
├── DashboardPage.ts
├── ReportPage.ts
├── LoginPage.ts
│
components/
│
├── HeaderComponent.ts
├── PaymentFrame.ts
│
utils/
│
├── page.utils.ts
├── popup.utils.ts
└── frame.utils.ts
│
tests/
│
├── dashboard/
├── reports/
└── authentication/
```

Multi-page logic should be reusable rather than duplicated in every
test.

------------------------------------------------------------------------

## 53. Real-World Example --- Report Workflow

Imagine an application:

``` text
Admin Dashboard
       ↓
Reports
       ↓
Click "Generate Report"
       ↓
New Tab
       ↓
Report Viewer
       ↓
Validate Report
       ↓
Download
       ↓
Close Tab
```

Automation:

``` typescript
const reportPagePromise =
    context.waitForEvent('page');

await page.getByRole('button', {
    name: 'Generate Report'
}).click();

const reportPage =
    await reportPagePromise;

await reportPage.waitForLoadState();

await expect(reportPage)
    .toHaveURL(/report/);

await expect(
    reportPage.getByRole('heading', {
        name: 'Report'
    })
).toBeVisible();

await reportPage.close();
```

------------------------------------------------------------------------

## 54. Real-World Example --- External Portal

``` text
Main Application
       ↓
Click "Customer Portal"
       ↓
New Tab
       ↓
Customer Portal
       ↓
Validate Customer
       ↓
Return to Main Application
```

The important concept is:

``` typescript
const portalPage =
    await context.waitForEvent('page');
```

The new page is handled directly.

------------------------------------------------------------------------

## 55. Real-World Example --- Help Center

``` typescript
const helpPagePromise =
    context.waitForEvent('page');

await page.getByRole('link', {
    name: 'Help Center'
}).click();

const helpPage =
    await helpPagePromise;

await helpPage.waitForLoadState();

await expect(helpPage)
    .toHaveURL(/help/);

await expect(
    helpPage.getByRole('heading', {
        name: 'Help Center'
    })
).toBeVisible();

await helpPage.close();
```

------------------------------------------------------------------------

## 56. Common Mistake --- Waiting After Clicking

Incorrect:

``` typescript
await page.getByText(
    'Open Report'
).click();

const pagePromise =
    context.waitForEvent('page');
```

Correct:

``` typescript
const pagePromise =
    context.waitForEvent('page');

await page.getByText(
    'Open Report'
).click();

const reportPage =
    await pagePromise;
```

Always register the listener before triggering the event.

------------------------------------------------------------------------

## 57. Common Mistake --- Using `context.pages()[1]`

Avoid:

``` typescript
const newPage =
    context.pages()[1];
```

Why?

Because:

-   Multiple pages may already exist.
-   Page ordering is not a reliable identifier.
-   Another popup may open first.
-   Parallel behavior can make tests flaky.

Use event-based capture instead.

------------------------------------------------------------------------

## 58. Common Mistake --- Using Fixed Delays

Avoid:

``` typescript
await page.waitForTimeout(5000);
```

Then:

``` typescript
const pages = context.pages();
```

This is unreliable.

Use:

``` typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByText(
    'Open Report'
).click();

const newPage =
    await newPagePromise;
```

------------------------------------------------------------------------

## 59. Common Mistake --- Forgetting to Wait for the New Page

Capturing the page is not always enough.

Instead of immediately interacting:

``` typescript
const reportPage =
    await newPagePromise;
```

consider:

``` typescript
await reportPage.waitForLoadState();

await reportPage
    .getByRole('heading', {
        name: 'Report'
    })
    .isVisible();
```

The actual synchronization condition should match what your application
needs.

------------------------------------------------------------------------

## 60. Common Mistake --- Confusing `Page` and `BrowserContext`

A `BrowserContext` can contain multiple pages:

``` text
Context
│
├── Page 1
├── Page 2
└── Page 3
```

Therefore:

``` typescript
context.pages()
```

returns pages.

While:

``` typescript
page.context()
```

returns the context associated with that page.

------------------------------------------------------------------------

## 61. Browser Context and Multi-Page Isolation

Contexts provide isolation.

Example:

``` typescript
const context1 =
    await browser.newContext();

const context2 =
    await browser.newContext();
```

Then:

``` text
Context 1
├── Page A
└── Page B

Context 2
├── Page C
└── Page D
```

Pages in different contexts have separate browser state.

------------------------------------------------------------------------

## 62. Multi-Page Authentication Scenario

A complex application may use:

``` text
Main Application
       ↓
Authentication Page
       ↓
Second Factor Page
       ↓
Main Application
```

Each may be represented as a separate `Page`.

The test should retain explicit references:

``` typescript
const mainPage = page;
const authPage = await authPagePromise;
```

This avoids confusing page state.

------------------------------------------------------------------------

## 63. Multi-Tab Debug Logging

When debugging, log:

``` typescript
for (const currentPage of context.pages()) {
    console.log({
        url: currentPage.url(),
        closed: currentPage.isClosed()
    });
}
```

You can also log titles:

``` typescript
for (const currentPage of context.pages()) {
    console.log(
        await currentPage.title()
    );
}
```

------------------------------------------------------------------------

## 64. Multi-Page Event Logging

You can listen for new pages:

``` typescript
context.on('page', page => {
    console.log(
        'New page:',
        page.url()
    );
});
```

This can be useful during framework debugging.

Do not overuse verbose event logging in production CI because it can
create excessive logs.

------------------------------------------------------------------------

## 65. Complete Enterprise Example

``` typescript
import {
    test,
    expect
} from '@playwright/test';

test(
    'verify report opens in a new page',
    async ({ page, context }) => {

        await page.goto('/dashboard');

        const reportPagePromise =
            context.waitForEvent('page');

        await page.getByRole('button', {
            name: 'Open Report'
        }).click();

        const reportPage =
            await reportPagePromise;

        await reportPage.waitForLoadState(
            'domcontentloaded'
        );

        await expect(reportPage)
            .toHaveURL(/report/);

        await expect(
            reportPage.getByRole(
                'heading',
                {
                    name: 'Sales Report'
                }
            )
        ).toBeVisible();

        console.log(
            'Report URL:',
            reportPage.url()
        );

        await reportPage.close();

        await expect(page)
            .toHaveURL(/dashboard/);
    }
);
```

------------------------------------------------------------------------

## 66. Multi-Page Test Design Principles

A professional Playwright framework should:

### 1. Capture pages using events

Use:

``` typescript
waitForEvent()
```

### 2. Store page references

Example:

``` typescript
const reportPage = ...
```

### 3. Avoid positional page selection

Avoid:

``` typescript
context.pages()[1]
```

### 4. Validate the correct page

Check:

-   URL
-   Title
-   Unique element

### 5. Close unnecessary pages

``` typescript
await reportPage.close();
```

### 6. Encapsulate page-specific logic

Use Page Objects.

------------------------------------------------------------------------

## 67. Interview Questions

### Beginner

1.  What is a Page in Playwright?
2.  How does Playwright represent a browser tab?
3.  How do you create a new page?
4.  How do you get all pages in a browser context?
5.  What is a popup?

### Intermediate

6.  How do you handle a link that opens a new tab?
7.  What is `context.waitForEvent('page')`?
8.  What is `page.waitForEvent('popup')`?
9.  How do you validate a newly opened page?
10. How do you close a child page?

### Advanced

11. What is the difference between `page.waitForEvent('popup')` and
    `context.waitForEvent('page')`?
12. How would you handle multiple new pages?
13. How do you avoid race conditions when handling new tabs?
14. How would you design a reusable popup utility?
15. How would you handle an OAuth popup?

### Expert

16. How would you architect multi-page workflows in an enterprise
    Playwright framework?
17. How would you debug intermittent popup failures in CI?
18. How would you identify the correct page when multiple pages open
    simultaneously?
19. How would you design a reusable `PageManager`?
20. How would you manage lifecycle and cleanup for multiple pages?

------------------------------------------------------------------------

## 68. Knowledge Check

### Question 1

What Playwright object represents a browser tab?

### Question 2

What is the difference between:

``` typescript
context.waitForEvent('page')
```

and:

``` typescript
page.waitForEvent('popup')
```

### Question 3

Why should the event listener be created before clicking the link?

### Question 4

How can you get all open pages?

### Question 5

Why is this approach fragile?

``` typescript
const page = context.pages()[1];
```

### Question 6

How would you close a newly opened report page?

### Question 7

How would you verify that a newly opened page contains the expected URL?

------------------------------------------------------------------------

## 69. Mini Challenge

Create a test:

``` text
multi-tab.spec.ts
```

Automate the following:

``` text
Open Dashboard
       ↓
Click "Open Report"
       ↓
Capture New Tab
       ↓
Wait for Report Page
       ↓
Verify URL
       ↓
Verify Report Heading
       ↓
Verify Report Data
       ↓
Close Report Tab
       ↓
Verify Dashboard
```

Requirements:

-   Playwright
-   TypeScript
-   `BrowserContext`
-   `Page`
-   `context.waitForEvent('page')`
-   Page Object Model
-   Assertions
-   No arbitrary waits
-   Stable locators
-   Proper cleanup

------------------------------------------------------------------------

## 70. Key Takeaways

The most important concept in multi-page Playwright automation is:

> **Do not manually switch browser windows. Capture and work directly
> with the appropriate `Page` object.**

The fundamental new-tab pattern is:

``` typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByText(
    'Open Report'
).click();

const newPage =
    await newPagePromise;
```

For a popup:

``` typescript
const popupPromise =
    page.waitForEvent('popup');

await page.getByText(
    'Open Popup'
).click();

const popup =
    await popupPromise;
```

For multiple pages:

``` typescript
const pages =
    context.pages();
```

For closing a page:

``` typescript
await newPage.close();
```

For checking page state:

``` typescript
newPage.isClosed();
```

------------------------------------------------------------------------

## 71. Step 71 → Step 75 Progression

``` text
Step 71
Mastering Playwright Input Fields & Forms
        ↓
Step 72
Mastering Playwright Date Pickers & Calendar Components
        ↓
Step 73
Mastering Playwright Alerts, Dialogs, Popups & Browser-Level Events
        ↓
Step 74
Mastering Playwright Frames & iFrames
        ↓
Step 75
Mastering Multiple Windows, Tabs, Popups & Browser Pages
```

You have now progressed from individual UI interactions into
**multi-context browser automation**.

------------------------------------------------------------------------

## 72. Milestone

### 🎯 Milestone: Multi-Page Browser Automation

You should now be comfortable with:

-   Multiple browser pages
-   Multiple tabs
-   New windows
-   Popup pages
-   `BrowserContext`
-   `Page`
-   `context.waitForEvent('page')`
-   `page.waitForEvent('popup')`
-   Page lifecycle
-   Page identification
-   Multi-page assertions
-   Multi-page Page Objects
-   Popup debugging
-   OAuth-style popup workflows

These capabilities are essential for real-world enterprise applications.

------------------------------------------------------------------------

# Step 76 Preview

**Step 76 --- Mastering Playwright Browser Contexts, Sessions, Cookies,
Local Storage & Session Management**

In the next step, you will learn how Playwright manages isolated browser
contexts, authentication sessions, cookies, local storage, session
storage, storage state, reusable authentication, role-based sessions,
session cleanup, and enterprise authentication architecture.

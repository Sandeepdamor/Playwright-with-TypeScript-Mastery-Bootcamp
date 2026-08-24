# Playwright with TypeScript Mastery Bootcamp

# Step 88 — Mastering Playwright Browser Contexts, Pages, Multi-Tab Workflows, Popups, Multiple Windows & Enterprise Session Management

## 1. Introduction

Real-world web applications frequently open:

- New tabs
- New windows
- Popup pages
- Authentication windows
- Reports in separate tabs
- Download or preview pages
- External applications

Playwright provides APIs for handling these scenarios through:

```text
Browser
BrowserContext
Page
Popup
Multiple Pages
Multiple Contexts
Page Events
Context Events
Session Isolation
```

Understanding these concepts is essential for building reliable enterprise test automation.

---

## 2. Browser Architecture

A simplified Playwright architecture is:

```text
Browser
   ↓
Browser Context
   ↓
Pages
   ↓
Frames
   ↓
Web Elements
```

Example:

```text
Chromium
   │
   ├── Context 1
   │     ├── Page 1
   │     └── Page 2
   │
   └── Context 2
         └── Page 1
```

---

## 3. Browser

The `browser` represents the browser engine.

Example:

```ts
test('browser example', async ({
  browser
}) => {
  const context =
    await browser.newContext();

  const page =
    await context.newPage();

  await page.goto('/');

  await context.close();
});
```

In most Playwright tests, the built-in fixtures handle browser lifecycle automatically.

---

## 4. Browser Context

A BrowserContext is an isolated browser session.

It contains its own:

- Cookies
- Local storage
- Session storage
- Permissions
- Pages
- Authentication state

Example:

```ts
const context =
  await browser.newContext();
```

---

## 5. Why Browser Contexts Matter

Contexts provide isolation.

Conceptually:

```text
Context A
  User A
  Cookies A
  Storage A

Context B
  User B
  Cookies B
  Storage B
```

This is useful for testing:

- Multiple users
- Different roles
- Independent sessions
- Permission differences
- Multi-user workflows

---

## 6. Page

A `Page` represents a browser tab or window.

Example:

```ts
const page =
  await context.newPage();

await page.goto('/dashboard');
```

A context can contain multiple pages.

```text
Context
 ├── Page 1
 ├── Page 2
 └── Page 3
```

---

## 7. Creating Multiple Pages

Example:

```ts
const page1 =
  await context.newPage();

const page2 =
  await context.newPage();

await page1.goto('/home');
await page2.goto('/reports');
```

Both pages belong to the same browser context.

---

## 8. Pages in the Current Context

You can inspect pages:

```ts
const pages =
  context.pages();

console.log(pages.length);
```

This is useful for debugging multi-tab workflows.

---

## 9. Opening a New Tab

Suppose a link opens a new tab.

A common pattern is:

```ts
const newPagePromise =
  context.waitForEvent('page');

await page.getByRole(
  'link',
  { name: 'Report' }
).click();

const newPage =
  await newPagePromise;

await newPage.waitForLoadState();
```

The important principle is to start waiting for the new page before triggering the action.

---

## 10. Why Start Waiting Before Clicking?

Bad approach:

```ts
await page.getByText(
  'Open Report'
).click();

const newPage =
  await context.waitForEvent('page');
```

The event may occur before the listener is registered.

Better:

```ts
const newPagePromise =
  context.waitForEvent('page');

await page.getByText(
  'Open Report'
).click();

const newPage =
  await newPagePromise;
```

---

## 11. Popup Windows

A popup is a new page opened by a page action.

Example:

```ts
const popupPromise =
  page.waitForEvent('popup');

await page.getByRole(
  'button',
  { name: 'Open Popup' }
).click();

const popup =
  await popupPromise;
```

The popup is a `Page`.

---

## 12. Popup vs New Page

A popup is associated with a specific opener page.

Use:

```ts
page.waitForEvent('popup')
```

when the new page is directly opened by that page.

Use:

```ts
context.waitForEvent('page')
```

when you want to listen for a new page at the browser-context level.

---

## 13. Popup Example

```ts
test('handle popup', async ({
  page
}) => {
  await page.goto('/');

  const popupPromise =
    page.waitForEvent('popup');

  await page.getByRole(
    'button',
    { name: 'Open Help' }
  ).click();

  const popup =
    await popupPromise;

  await popup.waitForLoadState();

  await expect(
    popup.getByRole(
      'heading',
      { name: 'Help' }
    )
  ).toBeVisible();
});
```

---

## 14. `Promise.all()` Pattern

Another common pattern is:

```ts
const [popup] =
  await Promise.all([
    page.waitForEvent('popup'),
    page.getByRole(
      'button',
      { name: 'Open Help' }
    ).click()
  ]);
```

This ensures the listener and triggering action are started together.

---

## 15. Waiting for Page Load

After obtaining a new page:

```ts
await popup.waitForLoadState();
```

Possible states include:

```text
domcontentloaded
load
networkidle
```

Use the least restrictive state that matches the application's needs.

---

## 16. Avoid Unnecessary `networkidle`

Do not automatically use:

```ts
await page.waitForLoadState(
  'networkidle'
);
```

Modern applications may continuously make network requests.

Prefer a meaningful UI assertion:

```ts
await expect(
  page.getByRole(
    'heading',
    { name: 'Report' }
  )
).toBeVisible();
```

---

## 17. Switching Between Pages

Suppose:

```text
Page 1 → Main Application
Page 2 → Report
```

You can directly interact with the page object:

```ts
await page2.getByRole(
  'button',
  { name: 'Export' }
).click();
```

No browser-level tab switching command is required.

The `Page` object represents the target tab.

---

## 18. Returning to the Original Page

Keep a reference:

```ts
const mainPage = page;
```

Then:

```ts
await reportPage.close();

await mainPage.bringToFront();
```

Usually, simply using `mainPage` again is sufficient.

---

## 19. `bringToFront()`

Example:

```ts
await reportPage.bringToFront();
```

This can be useful when a workflow requires a particular tab to become active.

However, Playwright can interact with page objects without manually switching focus in many situations.

---

## 20. Closing a Page

Example:

```ts
await popup.close();
```

After closing:

```ts
console.log(popup.isClosed());
```

Page cleanup is especially important in long-running test suites.

---

## 21. Page Lifecycle

A page can move through states such as:

```text
Created
 ↓
Navigating
 ↓
Loaded
 ↓
Interactive
 ↓
Closed
```

Use Playwright events and assertions rather than arbitrary delays.

---

## 22. Page Events

You can listen for events:

```ts
page.on(
  'console',
  message => {
    console.log(
      message.text()
    );
  }
);
```

Other useful events include:

```text
request
response
requestfailed
dialog
popup
download
filechooser
console
pageerror
```

Use events for diagnostics and special workflows.

---

## 23. Context Page Event

You can listen for new pages at the context level:

```ts
context.on(
  'page',
  page => {
    console.log(
      page.url()
    );
  }
);
```

This can be useful when multiple pages can be created dynamically.

For deterministic tests, explicit promises are often easier to reason about.

---

## 24. Multi-Tab Workflow

Example business workflow:

```text
Main Application
      ↓
Open Invoice
      ↓
New Tab
      ↓
Verify Invoice
      ↓
Close Tab
      ↓
Return to Main
```

Implementation:

```ts
const invoicePagePromise =
  context.waitForEvent('page');

await page.getByText(
  'Invoice'
).click();

const invoicePage =
  await invoicePagePromise;

await expect(
  invoicePage.getByRole(
    'heading',
    { name: 'Invoice' }
  )
).toBeVisible();
```

---

## 25. Multiple Tabs with Different URLs

Example:

```ts
const pages =
  context.pages();

for (const currentPage of pages) {
  console.log(
    currentPage.url()
  );
}
```

This can help identify the correct page during debugging.

---

## 26. Do Not Identify Tabs by Array Index

Avoid:

```ts
const reportPage =
  context.pages()[1];
```

Page order may change.

Prefer the page returned by:

```ts
waitForEvent('page')
```

or identify it using stable application behavior.

---

## 27. Page Selection by URL

For debugging or dynamic workflows:

```ts
const reportPage =
  context.pages().find(page =>
    page.url().includes('/reports')
  );
```

If you use this approach, validate that the page exists before interacting with it.

---

## 28. Multi-User Sessions

Use separate browser contexts for separate users.

Example:

```ts
const adminContext =
  await browser.newContext();

const employeeContext =
  await browser.newContext();
```

Then:

```ts
const adminPage =
  await adminContext.newPage();

const employeePage =
  await employeeContext.newPage();
```

Now:

```text
Admin Session
      ↕
Employee Session
```

can be tested independently.

---

## 29. Multi-User Collaboration

Example:

```text
Admin
 ↓
Creates Expense

Employee
 ↓
Views Expense

Manager
 ↓
Approves Expense
```

Each user can have an isolated context.

This is useful for enterprise workflow testing.

---

## 30. Context Authentication

You can create contexts using storage state:

```ts
const adminContext =
  await browser.newContext({
    storageState:
      '.auth/admin.json'
  });
```

Another:

```ts
const employeeContext =
  await browser.newContext({
    storageState:
      '.auth/employee.json'
  });
```

This avoids repeating UI login for every test.

---

## 31. Session Isolation

Separate contexts provide:

```text
Cookies
Storage
Permissions
Authentication
Pages
```

isolated from each other.

This is one of Playwright's most important architectural features.

---

## 32. Context-Specific Permissions

Example:

```ts
const context =
  await browser.newContext({
    permissions: [
      'geolocation'
    ]
  });
```

Permissions should be configured per context when needed.

---

## 33. Geolocation

Example:

```ts
const context =
  await browser.newContext({
    geolocation: {
      latitude: 22.7196,
      longitude: 75.8577
    },
    permissions: [
      'geolocation'
    ]
  });
```

Use test locations appropriate to the application's requirements.

---

## 34. Locale Per Context

Example:

```ts
const context =
  await browser.newContext({
    locale: 'en-US'
  });
```

Another context could use:

```ts
locale: 'en-IN'
```

This is useful for localization testing.

---

## 35. Timezone Per Context

Example:

```ts
const context =
  await browser.newContext({
    timezoneId:
      'Asia/Kolkata'
  });
```

This is useful for testing date/time-sensitive applications.

---

## 36. User Agent Per Context

Example:

```ts
const context =
  await browser.newContext({
    userAgent:
      'Custom Test Agent'
  });
```

Use custom user agents only when the test requirement justifies it.

---

## 37. Viewport Per Context

Example:

```ts
const context =
  await browser.newContext({
    viewport: {
      width: 1440,
      height: 900
    }
  });
```

Playwright projects are often a better place for standard device configurations.

---

## 38. Context Storage State

You can save authentication state:

```ts
await context.storageState({
  path: '.auth/user.json'
});
```

Then reuse:

```ts
const context =
  await browser.newContext({
    storageState:
      '.auth/user.json'
  });
```

This is useful for authentication reuse.

---

## 39. Context Cleanup

Always close manually created contexts:

```ts
await adminContext.close();
await employeeContext.close();
```

The built-in Playwright test fixtures normally manage fixture-created browser resources automatically.

---

## 40. Multiple Contexts in One Test

Example:

```ts
test('multi-user workflow', async ({
  browser
}) => {
  const adminContext =
    await browser.newContext();

  const employeeContext =
    await browser.newContext();

  const adminPage =
    await adminContext.newPage();

  const employeePage =
    await employeeContext.newPage();

  // workflow

  await adminContext.close();
  await employeeContext.close();
});
```

This is powerful for collaboration scenarios.

---

## 41. Multi-User Workflow Design

Example:

```text
Admin Context
     ↓
Create User

Employee Context
     ↓
Login/Refresh
     ↓
Verify User

Admin Context
     ↓
Deactivate User

Employee Context
     ↓
Verify Access Removed
```

This tests real cross-user behavior.

---

## 42. Context Isolation Validation

You can verify that sessions are independent.

Example:

```ts
await adminPage.goto('/admin');

await employeePage.goto('/dashboard');
```

Actions in one context should not unexpectedly authenticate the other.

---

## 43. Cookies and Context Isolation

Cookies belong to the browser context.

Therefore:

```text
Context A
Cookies A

Context B
Cookies B
```

This makes contexts useful for authentication and role testing.

---

## 44. Local Storage Isolation

Local storage is also isolated by context.

This is important for applications that store:

- Tokens
- Preferences
- Feature flags
- Session information

in browser storage.

---

## 45. Session Management Architecture

A scalable enterprise framework can use:

```text
Authentication Setup
        ↓
Storage State
        ↓
Browser Context
        ↓
Page
        ↓
Page Objects
```

For multi-user tests:

```text
Admin Storage State
        ↓
Admin Context

Employee Storage State
        ↓
Employee Context
```

---

## 46. Popup Authentication

Some legacy applications open a login popup.

Example:

```ts
const popupPromise =
  page.waitForEvent('popup');

await page.getByRole(
  'button',
  { name: 'Sign in' }
).click();

const popup =
  await popupPromise;

await popup.getByLabel(
  'Username'
).fill('testuser');
```

After authentication:

```ts
await popup.close();
```

The main page may then receive the authentication result.

---

## 47. OAuth and External Authentication

Modern applications may redirect to an external authentication provider.

A robust strategy often uses:

```text
Dedicated authentication setup
        ↓
Storage State
        ↓
Authenticated tests
```

rather than automating the complete external login flow in every test.

Follow the application's supported test authentication strategy.

---

## 48. Popup Event Ordering

Always register the popup listener before triggering the action:

```ts
const popupPromise =
  page.waitForEvent('popup');

await page.getByRole(
  'link',
  { name: 'Open Report' }
).click();

const popup =
  await popupPromise;
```

This prevents race conditions.

---

## 49. Multiple Popup Handling

If an action can create multiple pages, listen at the context level:

```ts
const pagePromise =
  context.waitForEvent('page');
```

Then identify the correct page based on stable properties.

Avoid relying on page indexes.

---

## 50. Download Pages and New Tabs

Some applications open generated documents in new tabs.

Example:

```ts
const reportPromise =
  context.waitForEvent('page');

await page.getByText(
  'Preview Report'
).click();

const report =
  await reportPromise;

await report.waitForLoadState();

await expect(
  report
).toHaveURL(/report/);
```

---

## 51. Print/Preview Windows

Applications may open:

```text
Main Page
     ↓
Print Preview
     ↓
New Page
```

Handle it like another page:

```ts
const previewPromise =
  page.waitForEvent('popup');

await page.getByText(
  'Print Preview'
).click();

const preview =
  await previewPromise;
```

---

## 52. Page Errors

Capture page errors when diagnosing unexpected behavior:

```ts
page.on(
  'pageerror',
  error => {
    console.error(
      error.message
    );
  }
);
```

Use this for debugging rather than turning every console message into a test failure.

---

## 53. Console Messages

Example:

```ts
page.on(
  'console',
  message => {
    console.log(
      `[${message.type()}]`,
      message.text()
    );
  }
);
```

Avoid logging sensitive information.

---

## 54. Network Events Across Pages

You can monitor requests:

```ts
page.on(
  'request',
  request => {
    console.log(
      request.method(),
      request.url()
    );
  }
);
```

For multiple pages, attach listeners to the relevant page or context.

---

## 55. Context-Level Request Monitoring

Example:

```ts
context.on(
  'request',
  request => {
    console.log(
      request.url()
    );
  }
);
```

Use context-level monitoring carefully because it may capture traffic from all pages.

---

## 56. Page Object Design for Multi-Tab Workflows

Avoid putting page switching logic into every test.

Instead:

```ts
async openReport(): Promise<Page> {
  const pagePromise =
    this.page
      .context()
      .waitForEvent('page');

  await this.page.getByRole(
    'link',
    { name: 'Report' }
  ).click();

  const reportPage =
    await pagePromise;

  await reportPage.waitForLoadState();

  return reportPage;
}
```

This keeps multi-page mechanics inside the Page Object.

---

## 57. Popup Component Object

Example:

```ts
class ReportPage {
  constructor(
    private readonly page: Page
  ) {}

  async verifyReport() {
    await expect(
      this.page.getByRole(
        'heading',
        { name: 'Report' }
      )
    ).toBeVisible();
  }
}
```

The test can then work with the report as a normal object.

---

## 58. Multi-Page Fixture Architecture

A fixture can provide:

```text
mainPage
reportPage
adminPage
employeePage
```

But avoid creating unnecessary pages for every test.

Create additional contexts/pages only when the workflow requires them.

---

## 59. Resource Cleanup

For manually created resources:

```ts
try {
  // test workflow
} finally {
  await adminContext.close();
  await employeeContext.close();
}
```

This ensures cleanup even when the test fails.

Fixtures can automate this lifecycle when the resource is reusable.

---

## 60. Common Multi-Page Mistakes

### Mistake 1 — Waiting after the click

Bad:

```ts
await page.click('a');
await context.waitForEvent('page');
```

Better:

```ts
const promise =
  context.waitForEvent('page');

await page.click('a');

const newPage =
  await promise;
```

### Mistake 2 — Using page indexes

Avoid:

```ts
context.pages()[1]
```

### Mistake 3 — Sharing contexts unnecessarily

Separate users should normally have separate contexts.

### Mistake 4 — Forgetting cleanup

Close manually created contexts.

### Mistake 5 — Using arbitrary waits

Wait for meaningful page or UI conditions.

---

## 61. Production-Quality Multi-Tab Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('verify report in new tab', async ({
  page,
  context
}) => {
  await page.goto('/dashboard');

  const reportPromise =
    context.waitForEvent('page');

  await page.getByRole(
    'link',
    { name: 'Open Report' }
  ).click();

  const reportPage =
    await reportPromise;

  await expect(
    reportPage.getByRole(
      'heading',
      { name: 'Report' }
    )
  ).toBeVisible();

  await expect(
    reportPage
  ).toHaveURL(/report/);

  await reportPage.close();

  await expect(
    page.getByRole(
      'heading',
      { name: 'Dashboard' }
    )
  ).toBeVisible();
});
```

---

## 62. Production-Quality Multi-User Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('admin and employee workflow', async ({
  browser
}) => {
  const adminContext =
    await browser.newContext({
      storageState:
        '.auth/admin.json'
    });

  const employeeContext =
    await browser.newContext({
      storageState:
        '.auth/employee.json'
    });

  try {
    const adminPage =
      await adminContext.newPage();

    const employeePage =
      await employeeContext.newPage();

    await adminPage.goto('/admin');

    await employeePage.goto(
      '/dashboard'
    );

    await expect(
      adminPage.getByRole(
        'heading',
        { name: 'Admin' }
      )
    ).toBeVisible();

    await expect(
      employeePage.getByRole(
        'heading',
        { name: 'Dashboard' }
      )
    ).toBeVisible();

  } finally {
    await adminContext.close();
    await employeeContext.close();
  }
});
```

---

## 63. Enterprise Browser Session Architecture

A scalable design can look like:

```text
Browser
 │
 ├── Admin Context
 │     ├── Admin Page
 │     └── Report Page
 │
 ├── Employee Context
 │     └── Employee Page
 │
 └── Manager Context
       └── Approval Page
```

Each context owns its session state.

---

## 64. When to Use Multiple Contexts

Use multiple contexts when testing:

- Role-based access
- Multi-user collaboration
- Chat systems
- Approval workflows
- Admin/employee workflows
- Concurrent sessions
- Permission changes
- Session isolation

Do not create multiple contexts simply because the application has multiple pages.

Multiple pages can usually live inside one context.

---

## 65. When to Use Multiple Pages

Use multiple pages when:

- A link opens a new tab
- A popup is opened
- A report opens separately
- A workflow requires two browser tabs
- A document preview opens in another page

---

## 66. When to Use Multiple Browsers

Multiple browser instances are less common.

Usually:

```text
One Browser
  ↓
Multiple Contexts
  ↓
Multiple Pages
```

is sufficient.

Use separate browser instances only when the test requirement genuinely requires separate browser processes or engines.

---

## 67. Browser Context Best Practices

1. Use contexts for session isolation.
2. Use pages for tabs/windows.
3. Use separate contexts for separate users.
4. Reuse storage state for authentication.
5. Avoid unnecessary context creation.
6. Close manually created contexts.
7. Avoid page indexes.
8. Capture page promises before triggering new-page actions.
9. Prefer explicit page references.
10. Keep multi-page logic inside reusable Page Objects or fixtures.

---

## 68. Interview Questions

### Q1. What is a BrowserContext?

An isolated browser session containing its own cookies, storage, permissions, and pages.

### Q2. What is a Page?

A browser tab or window represented by a Playwright `Page` object.

### Q3. How do you handle a new tab?

```ts
const pagePromise =
  context.waitForEvent('page');

await link.click();

const newPage =
  await pagePromise;
```

### Q4. How do you handle a popup?

```ts
const popupPromise =
  page.waitForEvent('popup');

await button.click();

const popup =
  await popupPromise;
```

### Q5. Why should the event listener be registered before the action?

To avoid a race condition where the new page or popup opens before Playwright starts waiting for it.

### Q6. How do you test two users simultaneously?

Create separate browser contexts:

```ts
const adminContext =
  await browser.newContext();

const employeeContext =
  await browser.newContext();
```

### Q7. Why use separate contexts for users?

They isolate authentication, cookies, storage, and permissions.

### Q8. Can one context contain multiple pages?

Yes.

### Q9. Should tabs be identified using `context.pages()[1]`?

Generally no. Use the page returned by the relevant event or identify it using stable properties.

### Q10. What is the recommended enterprise architecture?

Use:

```text
Browser
 ↓
Context
 ↓
Page
 ↓
Page/Component Object
 ↓
Test
```

with separate contexts for independent user sessions.

---

## 69. Hands-On Exercises

### Exercise 1 — New Tab

Click a link that opens a new tab and verify the new page.

### Exercise 2 — Popup

Handle a popup and validate its content.

### Exercise 3 — Multiple Pages

Create two pages in the same context and navigate them independently.

### Exercise 4 — Page Cleanup

Close a generated report tab after validation.

### Exercise 5 — Multi-User

Create:

```text
Admin Context
Employee Context
```

and verify their different dashboards.

### Exercise 6 — Authentication State

Load different storage states into different contexts.

### Exercise 7 — Multi-User Workflow

Implement:

```text
Admin creates record
 ↓
Employee views record
 ↓
Manager approves record
```

using separate contexts.

### Exercise 8 — Popup Authentication

Automate a test login popup and return to the main page.

### Exercise 9 — Dynamic Pages

Handle a page that opens after an asynchronous UI action.

### Exercise 10 — Enterprise Session Manager

Create a reusable utility or fixture that manages:

```text
adminContext
employeeContext
managerContext
```

and cleans them up automatically.

---

## 70. Key Takeaways

- A BrowserContext represents an isolated session.
- A Page represents a browser tab or window.
- One context can contain multiple pages.
- Multiple contexts are ideal for multi-user workflows.
- `context.waitForEvent('page')` handles new pages.
- `page.waitForEvent('popup')` handles popups.
- Always start waiting before triggering the action.
- Avoid identifying tabs by array index.
- Use storage state for reusable authentication.
- Manually created contexts should be closed.
- Multi-page workflows are easier to maintain when encapsulated in Page Objects or fixtures.
- Context isolation is essential for reliable multi-user testing.
- Use meaningful assertions instead of arbitrary waits.
- Enterprise session management should clearly separate users, contexts, and pages.

---

## 71. Enterprise Multi-Session Challenge

Build a complete workflow:

```text
Browser
 │
 ├── Admin Context
 │      ↓
 │   Create Expense
 │
 ├── Employee Context
 │      ↓
 │   View Expense
 │
 └── Manager Context
        ↓
     Approve Expense
```

Requirements:

1. Use separate contexts for each user.
2. Use storage state authentication.
3. Use Page Objects for each role.
4. Handle a report opened in a new tab.
5. Validate the new tab.
6. Return to the original page.
7. Keep sessions isolated.
8. Clean up manually created contexts.
9. Avoid page indexes.
10. Make the workflow reusable through fixtures or helper classes.

---

## 72. Final Architecture

```text
Browser
 │
 ├── Admin Context
 │     ├── Admin Page
 │     └── Report Page
 │
 ├── Employee Context
 │     └── Employee Page
 │
 └── Manager Context
       └── Manager Page

        ↓

Page Objects
        ↓
Component Objects
        ↓
Assertions
        ↓
Business Validation
```

This architecture provides a strong foundation for complex enterprise applications involving multiple users, sessions, tabs, and windows.

---

## 73. Next Step

**Step 89 — Mastering Playwright Dialogs, Alerts, Confirmations, Prompts, Beforeunload Events & Enterprise Browser Event Handling**

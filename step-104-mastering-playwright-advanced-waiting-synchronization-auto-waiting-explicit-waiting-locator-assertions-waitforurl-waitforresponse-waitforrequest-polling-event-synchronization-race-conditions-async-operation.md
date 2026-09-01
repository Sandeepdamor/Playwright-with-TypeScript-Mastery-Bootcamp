# Playwright with TypeScript Mastery Bootcamp

# Step 104 — Mastering Playwright Advanced Waiting & Synchronization: Auto-Waiting, Explicit Waiting, Locator Assertions, waitForURL, waitForResponse, waitForRequest, Polling, Event Synchronization, Race Conditions, Async Operations & Enterprise Flakiness Prevention

## 1. Introduction

Waiting and synchronization are among the most important skills in Playwright automation.

Modern web applications are asynchronous:

```text
Click
 ↓
API Request
 ↓
Backend Processing
 ↓
DOM Update
 ↓
Animation
 ↓
UI State Change
```

A test that assumes everything happens immediately can become flaky.

The goal is not:

```text
Wait longer
```

The goal is:

```text
Wait for the correct condition
```

A strong synchronization strategy makes tests:

```text
Reliable
Fast
Readable
Deterministic
Parallel-safe
```

---

## 2. Why Waiting Is Required

Consider:

```ts
await page
  .getByRole('button', {
    name: 'Save'
  })
  .click();

await page
  .getByText('Saved')
  .click();
```

The second action may occur before the application finishes updating.

Instead, Playwright should synchronize with the application's observable state.

---

## 3. Playwright Auto-Waiting

Playwright automatically waits for many actionability conditions before performing actions.

For example:

```ts
await page
  .getByRole(
    'button',
    {
      name: 'Submit'
    }
  )
  .click();
```

Playwright can wait for the element to become actionable.

---

## 4. Actionability

Before an action, Playwright may check conditions such as:

```text
Visible
Stable
Receives Events
Enabled
Editable
```

The exact checks depend on the action.

This is why explicit sleeps are usually unnecessary.

---

## 5. Auto-Waiting Example

Instead of:

```ts
await page.waitForTimeout(
  3000
);

await page
  .getByRole(
    'button',
    {
      name: 'Submit'
    }
  )
  .click();
```

prefer:

```ts
await page
  .getByRole(
    'button',
    {
      name: 'Submit'
    }
  )
  .click();
```

Let Playwright wait for actionability.

---

## 6. Why `waitForTimeout()` Is Usually a Bad Strategy

Example:

```ts
await page.waitForTimeout(
  5000
);
```

Problems:

```text
Too slow when application is ready early
Still unreliable when application needs longer
Hides the real synchronization condition
Creates unnecessary test duration
```

Use condition-based waiting instead.

---

## 7. Explicit Waiting

Explicit waiting means waiting for a specific condition.

Examples:

```text
Element visible
URL changed
Response received
Request sent
Text changed
State changed
Element hidden
```

Playwright provides APIs for these conditions.

---

## 8. Locator Assertions as Synchronization

One of the best synchronization patterns is:

```ts
await expect(
  page.getByText(
    'Dashboard'
  )
).toBeVisible();
```

Assertions automatically retry until the expected condition is satisfied or the timeout is reached.

---

## 9. Why Assertions Are Powerful

This:

```ts
await expect(
  page.getByText('Success')
).toBeVisible();
```

does both:

```text
Wait
+
Validate
```

This is generally better than:

```ts
await page.waitForTimeout(
  2000
);

expect(
  await page.getByText(
    'Success'
  ).isVisible()
).toBe(true);
```

---

## 10. Auto-Retrying Assertions

Playwright assertions can retry.

Examples:

```ts
toBeVisible()
toBeHidden()
toHaveText()
toContainText()
toHaveValue()
toBeEnabled()
toBeDisabled()
toHaveURL()
toHaveTitle()
```

This is essential for asynchronous UIs.

---

## 11. `expect` Timeout

Assertions have configurable timeouts.

Example:

```ts
await expect(
  page.getByText(
    'Success'
  )
).toBeVisible({
  timeout:
    10_000
});
```

Use increased timeouts only when the application genuinely requires them.

---

## 12. Test Timeout vs Assertion Timeout

These are different.

### Test Timeout

Maximum time allowed for the entire test.

### Assertion Timeout

Maximum time allowed for an individual assertion to succeed.

Keep these concepts separate.

---

## 13. `waitForURL()`

Use:

```ts
await page.waitForURL(
  /dashboard/
);
```

when navigation is asynchronous and the URL change itself is the condition.

Example:

```ts
await page
  .getByRole(
    'button',
    {
      name: 'Login'
    }
  )
  .click();

await page.waitForURL(
  /dashboard/
);
```

---

## 14. Prefer URL Assertions When Appropriate

You can also use:

```ts
await expect(
  page
).toHaveURL(
  /dashboard/
);
```

This both waits and asserts.

When the test's purpose is to verify the final URL, this is often the clearest option.

---

## 15. `waitForResponse()`

Use:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url()
        .includes(
          '/api/users'
        )
  );
```

Then perform the action:

```ts
await page
  .getByRole(
    'button',
    {
      name: 'Search'
    }
  )
  .click();

const response =
  await responsePromise;
```

---

## 16. Why Start `waitForResponse()` First

Correct:

```ts
const responsePromise =
  page.waitForResponse(
    '/api/users'
  );

await page
  .getByRole(
    'button',
    {
      name: 'Search'
    }
  )
  .click();

await responsePromise;
```

The listener is registered before the request occurs.

If you wait after the action, the request might already have completed.

---

## 17. Response Status Validation

Example:

```ts
const response =
  await responsePromise;

expect(
  response.status()
).toBe(200);
```

This verifies actual network behavior.

---

## 18. Response Body Validation

Example:

```ts
const body =
  await response.json();

expect(
  body.users
).toBeDefined();
```

Use this when backend response data is part of the test objective.

---

## 19. `waitForRequest()`

Use:

```ts
const requestPromise =
  page.waitForRequest(
    request =>
      request.url()
        .includes(
          '/api/users'
        )
  );
```

Then trigger the action.

---

## 20. Request Validation

Example:

```ts
const request =
  await requestPromise;

expect(
  request.method()
).toBe(
  'POST'
);
```

You can also inspect:

```text
URL
Headers
Post Data
```

Avoid logging sensitive information.

---

## 21. Request and Response Together

For complete synchronization:

```text
User Action
 ↓
Request
 ↓
Response
 ↓
UI Update
```

You can validate all three stages.

---

## 22. Event Synchronization

Some browser actions trigger events:

```text
Download
Popup
Dialog
Page
Request
Response
WebSocket
```

Register the event promise before the action.

Example:

```ts
const popupPromise =
  page.waitForEvent(
    'popup'
  );

await page
  .getByRole(
    'link',
    {
      name: 'Open'
    }
  )
  .click();

const popup =
  await popupPromise;
```

---

## 23. Download Synchronization

Example:

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await page
  .getByRole(
    'button',
    {
      name: 'Download'
    }
  )
  .click();

const download =
  await downloadPromise;
```

Then validate the downloaded file.

---

## 24. Dialog Synchronization

Example:

```ts
page.on(
  'dialog',
  async dialog => {
    await dialog.accept();
  }
);
```

Register the handler before triggering the dialog.

---

## 25. New Page Synchronization

If an action opens a new page:

```ts
const pagePromise =
  context.waitForEvent(
    'page'
  );

await page
  .getByRole(
    'link',
    {
      name: 'Reports'
    }
  )
  .click();

const newPage =
  await pagePromise;
```

Then wait for its required state.

---

## 26. `Promise.all()`

A common synchronization pattern is:

```ts
const [response] =
  await Promise.all([
    page.waitForResponse(
      '/api/users'
    ),
    page
      .getByRole(
        'button',
        {
          name: 'Search'
        }
      )
      .click()
  ]);
```

This ensures the listener is established before the triggering action.

---

## 27. Why `Promise.all()` Helps

Without proper synchronization:

```text
Action
 ↓
Request happens
 ↓
Listener registered
 ↓
Request missed
```

With:

```text
Listener
+
Action
```

the test observes the expected event.

---

## 28. Promise Ordering

Important:

```ts
await Promise.all([
  page.waitForResponse(...),
  page.click(...)
]);
```

not:

```ts
await page.click(...);

await page.waitForResponse(...);
```

when the response can occur immediately.

---

## 29. Waiting for DOM State

Prefer locator assertions:

```ts
await expect(
  page.getByRole(
    'button',
    {
      name: 'Submit'
    }
  )
).toBeEnabled();
```

rather than manually checking with polling loops.

---

## 30. Waiting for Text

Example:

```ts
await expect(
  page.getByTestId(
    'status'
  )
).toHaveText(
  'Approved'
);
```

This automatically waits for the expected text.

---

## 31. Waiting for Value

Example:

```ts
await expect(
  page.getByLabel(
    'Username'
  )
).toHaveValue(
  'sandeep'
);
```

Useful for asynchronous form updates.

---

## 32. Waiting for Hidden State

Example:

```ts
await expect(
  page.getByText(
    'Loading...'
  )
).toBeHidden();
```

This is useful for:

```text
Spinner
Overlay
Skeleton
Loading message
```

---

## 33. Waiting for Enabled State

Example:

```ts
await expect(
  page.getByRole(
    'button',
    {
      name: 'Save'
    }
  )
).toBeEnabled();
```

Useful when buttons become enabled after validation or API processing.

---

## 34. Waiting for Disabled State

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

Useful for validating loading or validation states.

---

## 35. Polling

Some conditions are not represented by a simple locator.

For example:

```text
Background job
 ↓
Processing
 ↓
Completed
```

You may need polling.

Playwright provides:

```ts
expect.poll()
```

for polling a value until it matches the expectation.

---

## 36. `expect.poll()`

Example:

```ts
await expect.poll(
  async () => {

    const response =
      await request.get(
        '/api/job/123'
      );

    const body =
      await response.json();

    return body.status;
  }
).toBe(
  'COMPLETED'
);
```

This is much better than repeatedly sleeping for a fixed duration.

---

## 37. Polling with Custom Timeout

Example:

```ts
await expect.poll(
  async () => {
    // fetch status
  },
  {
    timeout:
      30_000,
    intervals: [
      1000,
      2000,
      5000
    ]
  }
).toBe(
  'COMPLETED'
);
```

Use intervals appropriate for the application.

---

## 38. Polling Use Cases

Useful for:

```text
Background jobs
Report generation
Async processing
Email delivery status
Eventual consistency
Distributed systems
```

---

## 39. Avoid Fixed Polling Loops

Avoid:

```ts
for (
  let i = 0;
  i < 10;
  i++
) {

  await page.waitForTimeout(
    2000
  );

  // check
}
```

Prefer:

```ts
await expect.poll(
  async () => {
    // check condition
  }
).toBe(
  expected
);
```

---

## 40. Eventual Consistency

Distributed systems may update asynchronously:

```text
Service A
 ↓
Message Queue
 ↓
Service B
 ↓
Database
 ↓
UI
```

The result may not be immediately visible.

Condition-based polling is appropriate.

---

## 41. Polling API Example

```ts
await expect.poll(
  async () => {

    const response =
      await request.get(
        `/api/expenses/${id}`
      );

    const expense =
      await response.json();

    return expense.status;
  }
).toBe(
  'APPROVED'
);
```

This is useful for asynchronous workflows.

---

## 42. Polling UI State

If the UI exposes the state:

```ts
await expect.poll(
  async () =>
    page.getByTestId(
      'job-status'
    ).textContent()
).toBe(
  'Completed'
);
```

However, a locator assertion such as `toHaveText()` is usually simpler when it can express the condition directly.

---

## 43. Waiting for Network Idle

Playwright supports network-related waiting patterns, but `networkidle` should not be used as a universal synchronization strategy.

Modern applications may keep network connections open indefinitely.

Prefer waiting for the specific application condition.

---

## 44. Why `networkidle` Can Be Misleading

Applications may continuously perform:

```text
Analytics
Polling
WebSocket
Background refresh
Telemetry
```

Therefore:

```text
No network activity
```

may never occur.

Use it only when it represents a meaningful condition.

---

## 45. Waiting for Page Load States

Playwright supports page lifecycle concepts such as:

```text
domcontentloaded
load
networkidle
```

Understand the difference before choosing one.

---

## 46. `domcontentloaded`

Indicates that the initial HTML has been parsed.

It does not guarantee:

```text
All images loaded
All API requests completed
Application initialized
```

---

## 47. `load`

Indicates that the page load event has fired.

It still may not mean that an SPA has finished fetching application data.

---

## 48. SPA Synchronization

For SPAs, prefer application-level conditions:

```text
Dashboard heading visible
API response completed
Table populated
Loading spinner hidden
```

instead of generic page lifecycle assumptions.

---

## 49. Synchronization with Loading UI

A robust pattern:

```text
Click
 ↓
Loading Visible
 ↓
API Response
 ↓
Loading Hidden
 ↓
Content Visible
```

Test the states that matter to the user.

---

## 50. Race Conditions

A race condition occurs when the result depends on timing.

Example:

```text
Test clicks Save
       │
       ├── Request starts
       │
       ├── UI changes
       │
       └── Test checks too early
```

The test may pass sometimes and fail sometimes.

---

## 51. Common Race Condition Sources

```text
API requests
Animations
Navigation
Popups
Downloads
WebSockets
Timers
Background jobs
React/Angular state updates
```

---

## 52. Fixing Race Conditions

Do not:

```text
Add random sleep
```

Instead:

```text
Identify event
 ↓
Register listener
 ↓
Trigger action
 ↓
Await event
 ↓
Assert final state
```

---

## 53. Race Condition Example

Bad:

```ts
await page.click(
  '#save'
);

await page.waitForTimeout(
  1000
);

expect(...);
```

Better:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url()
        .includes(
          '/api/save'
        )
  );

await page.click(
  '#save'
);

const response =
  await responsePromise;

expect(
  response.status()
).toBe(200);
```

Then verify the UI state.

---

## 54. Multiple Concurrent Requests

One action may trigger:

```text
POST /api/save
GET /api/profile
GET /api/notifications
```

Do not blindly wait for all network activity.

Identify the request that represents the business condition under test.

---

## 55. Synchronizing with Business Events

Prefer:

```text
Expense status = APPROVED
```

over:

```text
Network became idle
```

The business condition is more meaningful and stable.

---

## 56. Async Operations

TypeScript/JavaScript automation is asynchronous.

Always understand:

```ts
await
Promise
async
```

Example:

```ts
const response =
  await request.get(
    '/api/users'
  );
```

Without `await`, the test may work with a Promise instead of its result.

---

## 57. Missing `await`

Bad:

```ts
expect(
  page
    .getByText(
      'Dashboard'
    )
    .isVisible()
).toBe(true);
```

Correct:

```ts
expect(
  await page
    .getByText(
      'Dashboard'
    )
    .isVisible()
).toBe(true);
```

Even better for synchronization:

```ts
await expect(
  page.getByText(
    'Dashboard'
  )
).toBeVisible();
```

---

## 58. Promise Rejection

Unhandled asynchronous failures can make debugging difficult.

Use:

```ts
await
```

for Playwright operations unless you intentionally need to manage the Promise yourself.

---

## 59. Parallel Async Operations

Use `Promise.all()` when operations are independent.

Example:

```ts
const [
  users,
  departments
] = await Promise.all([
  userApi.getUsers(),
  departmentApi.getDepartments()
]);
```

This can reduce unnecessary execution time.

Do not parallelize operations that have dependencies.

---

## 60. Sequential Async Operations

If operation B depends on operation A:

```text
Create Company
 ↓
Create Department
```

use sequential execution:

```ts
const company =
  await createCompany();

const department =
  await createDepartment(
    company.id
  );
```

---

## 61. Synchronization and Fixtures

Fixtures can encapsulate synchronization.

Example:

```text
Fixture
 ↓
Create Data
 ↓
Wait Until Ready
 ↓
Provide Data
```

This keeps tests focused on business behavior.

---

## 62. API Fixture Polling

Example:

```ts
async function
waitForJob(
  request:
    APIRequestContext,
  id: string
) {

  await expect.poll(
    async () => {

      const response =
        await request.get(
          `/api/jobs/${id}`
        );

      const body =
        await response.json();

      return body.status;
    }
  ).toBe(
    'COMPLETED'
  );
}
```

---

## 63. Custom Wait Helper

Create helpers only for real recurring conditions.

Example:

```ts
async function
waitForExpenseStatus(
  request:
    APIRequestContext,
  id: number,
  status: string
) {

  await expect.poll(
    async () => {

      const response =
        await request.get(
          `/api/expenses/${id}`
        );

      const body =
        await response.json();

      return body.status;
    }
  ).toBe(status);
}
```

---

## 64. Avoid Generic `waitForSomething()`

Avoid helpers like:

```ts
waitForSomething()
```

because their behavior is unclear.

Prefer:

```ts
waitForExpenseApproval()
waitForReportCompletion()
waitForUserStatus()
```

Make the business condition explicit.

---

## 65. Waiting for Animations

Usually, interact with stable locators and let Playwright handle actionability.

If an animation itself is the requirement, verify its observable final state rather than sleeping for an assumed animation duration.

---

## 66. Waiting for Angular Applications

Angular applications may have:

```text
Change detection
HTTP requests
Observables
Route transitions
Lazy-loaded components
```

Do not create arbitrary Angular-specific sleeps.

Wait for observable application state.

---

## 67. Waiting for React Applications

React applications may have:

```text
State updates
Effects
Suspense
API requests
Client-side routing
```

Again, prefer user-visible or network/business conditions.

---

## 68. Waiting for Tables

A good strategy:

```ts
await expect(
  page.getByRole(
    'row'
  )
).toHaveCount(
  expectedCount
);
```

Or:

```ts
await expect(
  page.getByText(
    username
  )
).toBeVisible();
```

Choose the assertion that represents the requirement.

---

## 69. Waiting for Search Results

Example:

```text
Enter Search
 ↓
Click Search
 ↓
API Request
 ↓
Results Render
```

Synchronize with:

```ts
await expect(
  page.getByText(
    'Sandeep'
  )
).toBeVisible();
```

If necessary, additionally validate the relevant response.

---

## 70. Waiting for Form Submission

Use:

```text
Submit
 ↓
Response
 ↓
Success Message
```

Example:

```ts
await expect(
  page.getByRole(
    'alert'
  )
).toHaveText(
  'Saved successfully'
);
```

---

## 71. Waiting for Navigation

Prefer:

```ts
await expect(page)
  .toHaveURL(
    /dashboard/
  );
```

instead of:

```ts
await page.waitForTimeout(
  2000
);
```

---

## 72. Waiting for Downloads

Use event synchronization:

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await page
  .getByRole(
    'button',
    {
      name:
        'Export'
    }
  )
  .click();

const download =
  await downloadPromise;
```

Then verify file metadata or content.

---

## 73. Waiting for Popups

Use:

```ts
const popupPromise =
  page.waitForEvent(
    'popup'
  );

await page
  .getByRole(
    'link',
    {
      name:
        'Open Report'
    }
  )
  .click();

const popup =
  await popupPromise;
```

Then synchronize with the popup's URL or content.

---

## 74. Waiting for WebSockets

For real-time applications, synchronize with the observable business result:

```text
Socket message
 ↓
Application state update
 ↓
Notification visible
```

Do not rely only on arbitrary delays.

---

## 75. Waiting for Background Jobs

Example:

```text
Start Report
 ↓
Job ID
 ↓
Poll Job Status
 ↓
COMPLETED
 ↓
Open Report
```

This is an excellent use case for `expect.poll()`.

---

## 76. Synchronization Timeout Strategy

Timeouts should reflect realistic system behavior.

Avoid:

```text
Everything → 120 seconds
```

because failures become slow.

Instead:

```text
Normal UI assertion → short
Expected API latency → moderate
Background job → longer
```

Tune based on real application behavior.

---

## 77. Global Timeout vs Local Timeout

Global timeouts provide a safety boundary.

Local timeouts should be used for exceptional operations.

Example:

```ts
await expect(
  page.getByText(
    'Report Ready'
  )
).toBeVisible({
  timeout:
    30_000
});
```

Do not make every assertion use a huge timeout.

---

## 78. Flakiness

A flaky test:

```text
Passes sometimes
Fails sometimes
```

Often caused by:

```text
Timing
Shared state
Network instability
Poor locators
Test order
Async race conditions
Environment instability
```

Synchronization addresses only the timing portion.

---

## 79. Flakiness Prevention

Use:

```text
Auto-waiting
Condition-based assertions
Explicit event synchronization
Stable locators
Isolated test data
Controlled network dependencies
Proper fixture lifecycle
```

---

## 80. Flakiness Investigation

When a test is flaky, ask:

```text
What event is the test actually waiting for?
Did the request happen before the listener?
Is the assertion checking the correct condition?
Is data shared?
Is there eventual consistency?
Is a third-party service involved?
```

---

## 81. Retry vs Synchronization

Retries are not a replacement for synchronization.

Bad approach:

```text
Test fails
 ↓
Retry
 ↓
Maybe passes
```

Better:

```text
Identify race
 ↓
Synchronize correctly
 ↓
Test becomes deterministic
```

Retries can still be useful for genuine transient infrastructure failures, depending on CI strategy.

---

## 82. Detecting Bad Waits

Warning signs:

```text
Many waitForTimeout calls
Very high global timeouts
Repeated retry loops
Networkidle everywhere
Sleep after every click
```

These usually indicate missing synchronization conditions.

---

## 83. Good Synchronization Pattern

```text
Identify Trigger
      ↓
Identify Observable Condition
      ↓
Register Event / Assertion
      ↓
Perform Action
      ↓
Await Condition
      ↓
Validate Business Result
```

---

## 84. Bad Synchronization Pattern

```text
Perform Action
 ↓
Sleep 5 seconds
 ↓
Hope application is ready
 ↓
Check something
```

Avoid this pattern.

---

## 85. Enterprise Synchronization Architecture

A scalable framework can organize:

```text
utils/
├── waits/
│   ├── apiWaits.ts
│   ├── jobWaits.ts
│   ├── navigationWaits.ts
│   └── businessWaits.ts
```

Only create reusable helpers for recurring business conditions.

---

## 86. Business Wait Helpers

Examples:

```text
waitForExpenseApproval()
waitForReportGeneration()
waitForUserActivation()
waitForEmailDelivery()
waitForJobCompletion()
```

These are more useful than generic timeout wrappers.

---

## 87. Synchronization in CI

CI environments may be slower than local machines.

Do not solve this by adding sleeps everywhere.

Instead:

```text
Use condition-based waits
+
Tune appropriate timeouts
+
Collect diagnostics
```

---

## 88. Diagnostics for Timeout Failures

When a timeout occurs, collect:

```text
Screenshot
Trace
URL
Relevant request/response
Console error
Application state
```

This makes synchronization failures easier to diagnose.

---

## 89. Synchronization with Test Data

Data creation may be asynchronous:

```text
Create Expense
 ↓
Backend Processing
 ↓
Status Pending
 ↓
Approval Processing
 ↓
Status Approved
```

Poll the actual state instead of waiting a fixed duration.

---

## 90. Synchronization with Eventual Consistency

Example:

```ts
await expect.poll(
  async () => {
    const response =
      await request.get(
        `/api/orders/${id}`
      );

    const body =
      await response.json();

    return body.status;
  }
).toBe(
  'READY'
);
```

This is appropriate when the system is eventually consistent.

---

## 91. Synchronization and Parallelism

Parallel tests can increase timing pressure.

Make sure:

```text
Test A
 ↓
Own data

Test B
 ↓
Own data
```

and avoid shared mutable resources.

---

## 92. Synchronization and Network Mocking

From Step 103:

```text
Mock API
 ↓
Known Response
 ↓
Predictable UI State
```

Network mocking can reduce timing uncertainty for UI edge-case tests.

But real API tests should still exist.

---

## 93. Synchronization and API Testing

From Step 102:

```text
API
 ↓
Poll
 ↓
Business State
```

APIRequestContext plus `expect.poll()` is powerful for asynchronous backend workflows.

---

## 94. Synchronization and Authentication

Authentication can involve:

```text
Redirect
 ↓
Identity Provider
 ↓
Callback
 ↓
Session
```

Synchronize with the actual final authenticated state rather than sleeping after login.

---

## 95. Enterprise Best Practices

1. Prefer Playwright auto-waiting.
2. Prefer locator assertions for UI state.
3. Use `waitForURL()` or `toHaveURL()` for navigation conditions.
4. Register `waitForResponse()` before triggering the request.
5. Register `waitForRequest()` before triggering the request.
6. Use event promises for popups and downloads.
7. Use `Promise.all()` for trigger + event synchronization.
8. Use `expect.poll()` for eventual consistency.
9. Avoid arbitrary sleeps.
10. Avoid universal `networkidle` waits.
11. Synchronize with business conditions.
12. Keep timeouts realistic.
13. Use separate timeouts for exceptional slow operations.
14. Do not use retries to hide race conditions.
15. Keep tests isolated.
16. Create business-specific wait helpers when truly reusable.
17. Collect diagnostics on timeout failures.
18. Use API setup and verification strategically.
19. Use network mocking for deterministic edge cases.
20. Document synchronization strategy.

---

## 96. Hands-On Exercise 1 — Auto-Waiting

Remove unnecessary:

```ts
page.waitForTimeout()
```

calls from an existing Playwright test.

Replace them with locator actions and assertions.

---

## 97. Hands-On Exercise 2 — URL Synchronization

Implement:

```text
Login
 ↓
Dashboard
```

using:

```ts
toHaveURL()
```

---

## 98. Hands-On Exercise 3 — Response Synchronization

Click Search and wait for:

```text
GET /api/users
```

Then verify status `200`.

---

## 99. Hands-On Exercise 4 — Request Synchronization

Submit a form and validate:

```text
POST method
Request URL
Request body
```

---

## 100. Hands-On Exercise 5 — Download Synchronization

Trigger a report download and capture it using:

```ts
page.waitForEvent(
  'download'
);
```

---

## 101. Hands-On Exercise 6 — Popup Synchronization

Click a link that opens a new page and capture the popup.

---

## 102. Hands-On Exercise 7 — Polling

Create a background job and poll:

```text
PENDING
 ↓
PROCESSING
 ↓
COMPLETED
```

using `expect.poll()`.

---

## 103. Hands-On Exercise 8 — Retry Simulation

Using the network mocking techniques from Step 103:

```text
First API call → 500
Second API call → 200
```

Verify the application's retry behavior.

---

## 104. Hands-On Exercise 9 — Race Condition

Take a flaky test that uses:

```ts
waitForTimeout()
```

and replace it with event-based synchronization.

---

## 105. Hands-On Exercise 10 — Business Wait Helper

Create:

```ts
waitForExpenseApproval()
```

that polls the API until:

```text
status = APPROVED
```

---

## 106. Enterprise Synchronization Challenge

Build a synchronization layer supporting:

```text
Auto-Waiting
Locator Assertions
Navigation
Requests
Responses
Downloads
Popups
Dialogs
Polling
Background Jobs
Eventual Consistency
Network Failures
Business Conditions
```

Requirements:

1. Remove arbitrary sleeps.
2. Use locator assertions.
3. Use URL assertions.
4. Synchronize requests.
5. Synchronize responses.
6. Synchronize downloads.
7. Synchronize popups.
8. Implement API polling.
9. Implement business wait helpers.
10. Handle eventual consistency.
11. Handle retry workflows.
12. Detect race conditions.
13. Configure realistic timeouts.
14. Avoid unnecessary `networkidle`.
15. Keep synchronization helpers reusable.
16. Support parallel execution.
17. Add timeout diagnostics.
18. Integrate with API fixtures.
19. Integrate with network mocks.
20. Document the synchronization architecture.

---

## 107. Final Checklist

```text
[ ] Auto-waiting understood
[ ] Actionability understood
[ ] waitForTimeout limitations understood
[ ] Locator assertions understood
[ ] Auto-retrying assertions understood
[ ] Assertion timeout understood
[ ] Test timeout understood
[ ] waitForURL understood
[ ] toHaveURL understood
[ ] waitForResponse understood
[ ] waitForRequest understood
[ ] Request validation understood
[ ] Response validation understood
[ ] Promise.all synchronization understood
[ ] Popup synchronization understood
[ ] Download synchronization understood
[ ] Dialog synchronization understood
[ ] New page synchronization understood
[ ] expect.poll understood
[ ] Polling strategy understood
[ ] Eventual consistency understood
[ ] Race conditions understood
[ ] Async operations understood
[ ] Promise handling understood
[ ] Parallel async operations understood
[ ] Networkidle limitations understood
[ ] Page lifecycle states understood
[ ] SPA synchronization understood
[ ] Background job synchronization understood
[ ] Business wait helpers understood
[ ] Timeout strategy understood
[ ] Flakiness prevention understood
[ ] Retry vs synchronization understood
[ ] CI synchronization understood
[ ] Synchronization diagnostics understood
[ ] Enterprise synchronization architecture understood
```

---

## 108. Key Takeaways

- Playwright's auto-waiting should be your first synchronization mechanism.
- Locator assertions combine waiting and validation.
- Avoid arbitrary `waitForTimeout()` calls.
- Use `toHaveURL()` or `waitForURL()` for navigation conditions.
- Register request/response listeners before triggering actions.
- Use `Promise.all()` when synchronizing an event with its trigger.
- Use event-based synchronization for downloads and popups.
- Use `expect.poll()` for background jobs and eventual consistency.
- Synchronize with meaningful business conditions rather than generic network activity.
- `networkidle` is not a universal solution for modern applications.
- Test timeout and assertion timeout solve different problems.
- Retries should not be used to hide race conditions.
- Unique test data and network isolation also reduce timing-related flakiness.
- API setup and polling are powerful for asynchronous workflows.
- Business-specific wait helpers are better than generic sleep wrappers.
- Good synchronization produces tests that are fast, deterministic, and maintainable.

---

## 109. Next Step

**Step 105 — Mastering Playwright Page Object Model & Enterprise Framework Architecture: Page Objects, Component Objects, Base Pages, Locators, Actions, Assertions, Composition, Fixtures, Dependency Injection, SOLID Principles, Reusability & Scalable Automation Design**

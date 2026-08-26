# Playwright with TypeScript Mastery Bootcamp

# Step 96 — Mastering Playwright Debugging & Diagnostics: Trace Viewer, Screenshots, Video, Console Logs, Network Logs, Test Attachments, Error Analysis, VS Code Debugging & Enterprise Failure Investigation

## 1. Introduction

Debugging is one of the most important skills in Playwright automation.

A test failure should answer:

```text
What failed?
Why did it fail?
Where did it fail?
What was the browser doing?
What was the API doing?
What was the application state?
```

Playwright provides powerful diagnostics:

```text
Trace Viewer
Screenshots
Video
Console Logs
Network Logs
Test Attachments
Error Messages
Call Logs
Browser Logs
VS Code Debugging
HTML Reports
```

A mature automation framework does not simply report:

```text
Test Failed
```

It should provide enough evidence to investigate the failure quickly.

---

## 2. Why Debugging Matters

Consider:

```text
Test failed: timeout
```

That message alone may not tell you whether:

```text
Element was missing
Page was slow
API failed
Locator was wrong
Authentication expired
Network request failed
Application crashed
Test data was missing
```

Diagnostics provide the missing evidence.

---

## 3. Debugging Workflow

A practical workflow is:

```text
Test Failure
    ↓
Read Error
    ↓
Inspect Locator / Assertion
    ↓
Open HTML Report
    ↓
Inspect Trace
    ↓
Check Screenshot
    ↓
Check Video
    ↓
Check Console
    ↓
Check Network
    ↓
Reproduce Locally
    ↓
Fix Root Cause
```

Always investigate the root cause instead of simply increasing timeouts.

---

## 4. Playwright Error Messages

A Playwright error often contains:

```text
Test name
File
Line number
Expected result
Actual result
Locator
Timeout
Call log
```

Example:

```text
expect(locator).toBeVisible()

Timeout 5000ms exceeded
```

Start debugging from the first meaningful failure.

---

## 5. Locator Timeout

Example:

```ts
await expect(
  page.getByRole(
    'button',
    { name: 'Submit' }
  )
).toBeVisible();
```

If it times out, investigate:

```text
Is the page correct?
Is the locator correct?
Is the element rendered?
Is authentication valid?
Is the API response successful?
```

Do not immediately change:

```ts
timeout: 60000
```

---

## 6. Strict Mode Violation

A common error:

```text
strict mode violation
```

This means a locator matched multiple elements when one was expected.

Example:

```ts
page.locator('h6')
```

may match:

```text
Admin
User Management
```

Fix the locator using better semantics:

```ts
page.getByRole(
  'heading',
  { name: 'Admin' }
);
```

---

## 7. Element Not Found

Possible causes:

```text
Wrong selector
Wrong page
Wrong state
Slow application
Conditional rendering
Authentication failure
```

Debug by inspecting the page and trace before changing the locator.

---

## 8. Navigation Timeout

Example:

```text
page.goto() timeout
```

Investigate:

```text
URL
DNS
Network
Server
Redirect
Authentication
Browser connectivity
```

A navigation timeout may indicate an application or environment problem rather than a Playwright problem.

---

## 9. Trace Viewer

Trace Viewer is one of Playwright's most powerful debugging tools.

A trace can contain:

```text
Actions
Screenshots
DOM snapshots
Network
Console
Source
Timing
Metadata
```

It allows you to replay the test execution timeline.

---

## 10. Enabling Trace

Example:

```ts
use: {
  trace:
    'on-first-retry'
}
```

This is a good CI strategy because traces are collected when a test needs another attempt.

---

## 11. Trace Modes

Common strategies include:

```text
off
on
on-first-retry
retain-on-failure
```

Choose based on storage and debugging requirements.

---

## 12. Trace on Every Test

Example:

```ts
use: {
  trace: 'on'
}
```

Useful for:

```text
Local debugging
Difficult investigations
Temporary diagnostics
```

But it can generate large amounts of data.

---

## 13. Trace on First Retry

Example:

```ts
use: {
  trace:
    'on-first-retry'
}
```

Flow:

```text
Attempt 1
 ↓
Failure
 ↓
Retry
 ↓
Trace Captured
```

This is often a strong CI default.

---

## 14. Opening a Trace

Playwright provides a trace viewer command.

Example:

```bash
npx playwright show-trace trace.zip
```

The exact trace path depends on the generated test result.

---

## 15. What to Inspect in Trace Viewer

Look at:

```text
Timeline
Actions
DOM
Network
Console
Source
Before/After snapshots
```

Start around the action where the failure occurred.

---

## 16. Trace Timeline

The timeline helps answer:

```text
What happened immediately before failure?
```

Example:

```text
Click Login
 ↓
POST /login
 ↓
302 Redirect
 ↓
GET /dashboard
 ↓
Dashboard failed to load
 ↓
Assertion timeout
```

This can reveal the actual root cause.

---

## 17. DOM Snapshot

Trace snapshots help inspect the page state at a particular point in time.

Useful when:

```text
Locator matched nothing
Text changed
Element disappeared
Page navigated unexpectedly
```

---

## 18. Network in Trace

Network information can reveal:

```text
401
403
404
500
503
Slow requests
Failed requests
Unexpected redirects
```

This is extremely useful for API-backed applications.

---

## 19. Console in Trace

Browser console errors can reveal:

```text
JavaScript exceptions
Application errors
Warnings
Failed scripts
```

Example:

```text
TypeError:
Cannot read properties of undefined
```

This may explain a UI failure.

---

## 20. Screenshot on Failure

Configure:

```ts
use: {
  screenshot:
    'only-on-failure'
}
```

A screenshot gives a visual snapshot of the failure.

---

## 21. Screenshot Strategy

Common options:

```text
off
on
only-on-failure
```

Recommended for CI:

```text
only-on-failure
```

This keeps artifacts manageable.

---

## 22. Manual Screenshot

You can capture a screenshot manually:

```ts
await page.screenshot({
  path:
    'screenshots/debug.png',
  fullPage:
    true
});
```

Use manual screenshots for specific diagnostic checkpoints.

---

## 23. Screenshot Naming

Use meaningful paths when manually capturing artifacts:

```text
screenshots/
├── login-page.png
├── dashboard.png
└── failure-state.png
```

Avoid random names such as:

```text
image1.png
image2.png
```

---

## 24. Full-Page Screenshot

Example:

```ts
await page.screenshot({
  path:
    'screenshots/page.png',
  fullPage:
    true
});
```

Useful when the relevant element may be below the visible viewport.

---

## 25. Element Screenshot

You can capture a specific element:

```ts
await page
  .getByRole(
    'table'
  )
  .screenshot({
    path:
      'screenshots/users-table.png'
  });
```

Useful for focused visual diagnostics.

---

## 26. Video Recording

Example:

```ts
use: {
  video:
    'retain-on-failure'
}
```

Video can show:

```text
Mouse actions
Navigation
Page transitions
Visual state
Unexpected UI changes
```

---

## 27. Video Strategy

Common options include:

```text
off
on
retain-on-failure
on-first-retry
```

For CI, failure-focused video is usually more practical than recording every passing test.

---

## 28. Video Storage Cost

Videos can consume significant storage.

For large suites:

```text
1000 tests
 ×
video files
 =
large artifact volume
```

Use retention policies and failure-focused recording.

---

## 29. Browser Console Logs

You can listen for console messages:

```ts
page.on(
  'console',
  message => {
    console.log(
      `[browser] ${message.type()}:`,
      message.text()
    );
  }
);
```

This can be added to a debugging fixture.

---

## 30. Console Error Listener

Example:

```ts
page.on(
  'console',
  message => {

    if (
      message.type() ===
      'error'
    ) {
      console.error(
        message.text()
      );
    }
  }
);
```

This helps identify application-side errors.

---

## 31. Page Error Listener

Listen for uncaught page errors:

```ts
page.on(
  'pageerror',
  error => {
    console.error(
      'Page error:',
      error.message
    );
  }
);
```

This can expose JavaScript failures that may not be visible in the UI.

---

## 32. Request Listener

Example:

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

Useful when debugging unexpected API calls.

Avoid logging sensitive headers or request bodies.

---

## 33. Response Listener

Example:

```ts
page.on(
  'response',
  response => {
    console.log(
      response.status(),
      response.url()
    );
  }
);
```

This can help identify failed backend requests.

---

## 34. Failed Request Listener

Playwright can expose failed network requests.

Example:

```ts
page.on(
  'requestfailed',
  request => {

    console.error(
      'Request failed:',
      request.url(),
      request.failure()
    );

  }
);
```

Useful for:

```text
Connection errors
DNS problems
Blocked resources
Network failures
```

---

## 35. Network Debugging Strategy

When a UI action fails:

```text
UI Action
 ↓
Expected API Request
 ↓
Request Sent?
 ↓
Response Received?
 ↓
Status Code?
 ↓
Response Data?
 ↓
UI Updated?
```

This is often faster than debugging the locator alone.

---

## 36. HTTP Status Investigation

Look for:

```text
2xx → Success
3xx → Redirect
4xx → Client/auth/request problem
5xx → Server problem
```

Do not automatically classify every 4xx or 5xx as a test failure; determine whether that response was expected for the scenario.

---

## 37. Authentication Failure Diagnostics

If a test suddenly redirects to login:

```text
Test starts
 ↓
Storage State
 ↓
Request
 ↓
401
 ↓
Redirect
 ↓
Element missing
```

The final locator error may be misleading.

Inspect network and authentication state first.

---

## 38. API Response Body Debugging

For controlled debugging:

```ts
const response =
  await request.get(
    '/api/users'
  );

if (!response.ok()) {

  console.error(
    await response.text()
  );

}
```

Do not print secrets or sensitive user information.

---

## 39. Console Logging Strategy

Avoid:

```ts
console.log(
  everything
);
```

This creates noisy CI logs.

Prefer:

```text
Important state
Failure diagnostics
Identifiers
Statuses
Timing
```

---

## 40. Structured Logging

Instead of:

```ts
console.log(
  'Something failed'
);
```

use:

```ts
console.log(
  JSON.stringify({
    event:
      'api_failure',
    status:
      response.status(),
    url:
      response.url()
  })
);
```

Do not include credentials, tokens, or sensitive payloads.

---

## 41. Test Attachments

Playwright tests can attach diagnostic files or text.

Conceptually:

```ts
await testInfo
  .attach(
    'debug-info',
    {
      body:
        'Diagnostic information',
      contentType:
        'text/plain'
    }
  );
```

This allows diagnostics to appear in reports.

---

## 42. Attach JSON

Example:

```ts
await testInfo.attach(
  'response',
  {
    body:
      JSON.stringify(
        responseBody,
        null,
        2
      ),
    contentType:
      'application/json'
  }
);
```

Avoid attaching sensitive response data.

---

## 43. Attach Screenshot

Example:

```ts
const buffer =
  await page.screenshot();

await testInfo.attach(
  'failure-screenshot',
  {
    body: buffer,
    contentType:
      'image/png'
  }
);
```

This is useful when building custom diagnostic fixtures.

---

## 44. `testInfo`

Playwright exposes test execution metadata through `testInfo`.

Useful information includes:

```text
Test title
Project
Retry
Status
Output directory
Attachments
Duration
```

This can be used by advanced fixtures and reporters.

---

## 45. Output Directory

Use:

```ts
testInfo.outputDir
```

for test-specific artifacts.

This prevents multiple parallel tests from overwriting the same diagnostic file.

---

## 46. Test Title in Diagnostics

Example:

```ts
console.log(
  `Running: ${
    testInfo.title
  }`
);
```

Useful for debugging large parallel suites.

---

## 47. Worker Information

Diagnostics can include worker information where available.

Useful format:

```text
Test
Worker
Project
Retry
Environment
```

This helps identify parallel-execution issues.

---

## 48. Retry Diagnostics

When a test is retried, capture:

```text
Attempt number
Failure reason
Trace
Screenshot
Video
Network evidence
```

Compare:

```text
First attempt
vs
Retry
```

If only the first attempt fails, investigate possible flakiness.

---

## 49. Debugging Flaky Tests

A flaky test may:

```text
Pass
Fail
Pass
Fail
```

Possible causes:

```text
Race condition
Unstable environment
Shared data
Network timing
Async rendering
External service
Incorrect waits
```

Use trace and artifacts to compare successful and failed executions.

---

## 50. Do Not Fix Flakiness with Sleeps

Avoid:

```ts
await page.waitForTimeout(
  5000
);
```

as a default solution.

Prefer:

```ts
await expect(
  locator
).toBeVisible();
```

or wait for the actual application condition.

---

## 51. Debugging with `waitForTimeout`

Temporary use may help understand behavior:

```ts
await page.waitForTimeout(
  1000
);
```

But remove unnecessary fixed waits from production tests.

---

## 52. VS Code Debugging

Playwright works well with VS Code debugging.

Useful capabilities:

```text
Breakpoints
Step Over
Step Into
Variable inspection
Call stack
Debug console
```

---

## 53. Debugging a Test in VS Code

Typical workflow:

```text
Open test
 ↓
Set breakpoint
 ↓
Start Debug
 ↓
Browser pauses
 ↓
Inspect variables
 ↓
Step through actions
```

This is useful when understanding code-level behavior.

---

## 54. Playwright VS Code Extension

The Playwright extension can help with:

```text
Run tests
Debug tests
Pick locators
View test structure
Inspect failures
```

It can significantly improve local development productivity.

---

## 55. Debugging with `--debug`

Command:

```bash
npx playwright test   --debug
```

This provides an interactive debugging workflow.

---

## 56. Debugging with `PWDEBUG`

Playwright supports a debugging environment variable.

Example:

```bash
$env:PWDEBUG="1"
npx playwright test
```

On other shells, syntax differs.

Use the mechanism appropriate for your operating system.

---

## 57. Inspector

The Playwright Inspector can help inspect:

```text
Locators
Actions
Page state
Step execution
```

It is especially useful when learning locator behavior.

---

## 58. Debugging Locator Problems

Recommended process:

```text
1. Inspect DOM
2. Check role/name
3. Check text
4. Check accessible name
5. Verify visibility
6. Verify page state
7. Check strictness
8. Use trace
```

Prefer user-facing locators over fragile CSS/XPath.

---

## 59. Locator Debugging Example

Instead of:

```ts
page.locator(
  'div:nth-child(3) button'
)
```

prefer:

```ts
page.getByRole(
  'button',
  {
    name:
      'Save'
  }
);
```

This is usually more resilient.

---

## 60. `locator.count()`

For debugging:

```ts
const count =
  await page
    .getByRole(
      'button',
      {
        name:
          'Save'
      }
    )
    .count();

console.log(
  'Count:',
  count
);
```

This can help diagnose strict-mode failures.

---

## 61. Inspecting Text

Example:

```ts
console.log(
  await page
    .locator(
      'body'
    )
    .innerText()
);
```

Use carefully on large pages.

For debugging, prefer targeted locators.

---

## 62. Inspecting Element State

You can check:

```ts
await locator.isVisible();
await locator.isEnabled();
await locator.isChecked();
```

These are useful diagnostics, but assertions are generally preferred for test validation.

---

## 63. Debugging Hidden Elements

If an element exists but is not visible, inspect:

```text
CSS
Display
Visibility
Overlay
Animation
Viewport
Parent state
```

The trace DOM snapshot can help identify the actual state.

---

## 64. Debugging Click Failures

A click may fail because:

```text
Element hidden
Element covered
Element disabled
Wrong locator
Page changing
Animation
Wrong frame
```

Inspect the action log and trace before adding force clicks.

---

## 65. Avoid Unnecessary `force`

Avoid:

```ts
await locator.click({
  force: true
});
```

unless the test intentionally needs to bypass normal actionability checks.

Force clicking can hide real UI problems.

---

## 66. Debugging Frames

If an element is inside an iframe:

```ts
const frame =
  page.frameLocator(
    '#payment-frame'
  );

await frame
  .getByRole(
    'button',
    {
      name:
        'Pay'
    }
  )
  .click();
```

If the locator fails, verify:

```text
Frame exists
Frame loaded
Correct selector
Correct frame hierarchy
```

---

## 67. Debugging Popups

For a new page:

```ts
const popupPromise =
  page.waitForEvent(
    'popup'
  );

await page.getByRole(
  'link'
).click();

const popup =
  await popupPromise;
```

If debugging:

```text
Did popup open?
Which URL?
Which page?
```

---

## 68. Debugging Downloads

For download failures:

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await page.getByText(
  'Download'
).click();

const download =
  await downloadPromise;

console.log(
  download.suggestedFilename()
);
```

Verify the download event and file path.

---

## 69. Debugging Dialogs

A JavaScript dialog can block execution.

Example:

```ts
page.on(
  'dialog',
  async dialog => {

    console.log(
      dialog.message()
    );

    await dialog.accept();
  }
);
```

Unexpected dialogs can cause timeouts.

---

## 70. Debugging Authentication

Check:

```text
Storage state
Cookies
Redirects
API status
Login response
User role
```

A test that appears to have a locator issue may actually have an authentication problem.

---

## 71. Debugging Test Data

If a test fails because a record is missing:

```text
Was data created?
Was creation successful?
Was correct ID used?
Was data deleted early?
Did another test modify it?
```

Use API logs or database diagnostics where permitted.

---

## 72. Debugging Parallel Failures

If tests fail only in parallel:

```text
Run with workers=1
 ↓
Pass?
 ↓
Likely shared-state/race issue
```

Then investigate:

```text
Shared user
Shared file
Shared database record
Global variable
Server-side state
```

Do not permanently disable parallelism as the first solution.

---

## 73. Debugging CI-Only Failures

If:

```text
Local → Pass
CI → Fail
```

compare:

```text
Browser
OS
Node version
Environment
CPU
Memory
Network
Base URL
Credentials
Workers
Timing
```

CI-specific artifacts are critical here.

---

## 74. CI Environment Diagnostics

Capture safe metadata:

```text
Node version
Playwright version
Browser version
OS
Project
Environment
Worker
Retry
```

Never expose credentials.

---

## 75. Browser Version Diagnostics

Browser differences can cause failures.

Record:

```text
Chromium version
Firefox version
WebKit version
```

This helps determine compatibility problems.

---

## 76. API Diagnostics

For API-related failures record:

```text
HTTP method
URL path
Status
Duration
Correlation ID
```

Avoid:

```text
Authorization header
Password
Token
Sensitive payload
```

---

## 77. Correlation IDs

If the application supports request correlation IDs:

```text
Test
 ↓
Request ID
 ↓
Backend Logs
```

This can dramatically improve enterprise debugging.

Example:

```ts
const correlationId =
  `test-${Date.now()}`;
```

Send it only if the test environment supports such headers.

---

## 78. Timing Diagnostics

Measure important operations:

```text
Login
API call
Navigation
Page rendering
Report generation
```

Unexpectedly high durations can reveal performance problems.

---

## 79. Test Duration

Use test metadata to understand:

```text
Slow tests
Fast tests
Retries
Fixture overhead
```

A test that normally takes 2 seconds but suddenly takes 30 seconds may indicate an environment issue even if it passes.

---

## 80. Slow Test Investigation

When a test is slow:

```text
Trace
 ↓
Timeline
 ↓
Find long operation
 ↓
Network
 ↓
Application
 ↓
Fixture
```

Do not simply increase the timeout.

---

## 81. Enterprise Diagnostic Fixture

A diagnostic fixture can register:

```text
Console errors
Page errors
Failed requests
Screenshots
Metadata
```

Example concept:

```ts
diagnostics:
  async (
    { page },
    use
  ) => {

    const errors: string[] = [];

    page.on(
      'pageerror',
      error => {
        errors.push(
          error.message
        );
      }
    );

    await use(
      errors
    );
  }
```

For a full framework, attach diagnostics through `testInfo` and clean up listeners appropriately.

---

## 82. Automatic Failure Diagnostics

A mature fixture can:

```text
Test Failure
 ↓
Capture screenshot
 ↓
Capture console errors
 ↓
Capture failed requests
 ↓
Attach diagnostic JSON
```

This reduces manual debugging effort.

---

## 83. Diagnostic Data Structure

Example:

```ts
{
  test: 'login test',
  project: 'chromium',
  retry: 0,
  consoleErrors: [],
  pageErrors: [],
  failedRequests: []
}
```

This can be attached as JSON.

---

## 84. Failure Investigation Checklist

When a test fails:

```text
[ ] Read first error
[ ] Check line number
[ ] Check locator
[ ] Check page URL
[ ] Open trace
[ ] Inspect screenshot
[ ] Inspect video
[ ] Check console errors
[ ] Check page errors
[ ] Check failed requests
[ ] Check authentication
[ ] Check test data
[ ] Check parallelism
[ ] Reproduce locally
```

---

## 85. Root Cause Categories

Most failures fall into categories:

```text
Test Code
Locator
Application
API
Environment
Authentication
Data
Infrastructure
Concurrency
Browser Compatibility
```

Classifying the failure helps choose the correct fix.

---

## 86. Test Code Failure

Examples:

```text
Wrong assertion
Wrong locator
Incorrect test flow
Incorrect fixture
Incorrect expected data
```

Fix the automation code.

---

## 87. Application Failure

Examples:

```text
JavaScript exception
Wrong UI
Missing feature
Incorrect response handling
```

Report the application defect with diagnostic evidence.

---

## 88. Environment Failure

Examples:

```text
Server unavailable
Database unavailable
DNS issue
Certificate issue
Deployment incomplete
```

Do not rewrite test logic to hide infrastructure problems.

---

## 89. Data Failure

Examples:

```text
Test account disabled
Expected record missing
Duplicate test data
Stale environment state
```

Fix test-data management or environment state.

---

## 90. Authentication Failure

Examples:

```text
Expired token
Invalid cookie
Login API unavailable
Role mismatch
SSO issue
```

Inspect authentication artifacts before changing UI locators.

---

## 91. Browser Compatibility Failure

If:

```text
Chromium → Pass
Firefox → Fail
```

investigate:

```text
CSS behavior
Browser API
Timing
Rendering
Application compatibility
```

Do not automatically skip Firefox.

---

## 92. Debugging Strategy for Flaky Tests

Use repeated execution:

```bash
npx playwright test   tests/example.spec.ts   --repeat-each=10
```

This can help reproduce intermittent failures.

Use it as a diagnostic tool, not as proof that the test is stable.

---

## 93. Reproduce a Failure

Start with:

```bash
npx playwright test   tests/example.spec.ts   --project=chromium   --workers=1
```

Then:

```text
Add trace
Run headed
Use debugger
Inspect network
Compare data
```

---

## 94. Debugging with a Single Worker

Command:

```bash
npx playwright test   --workers=1
```

If the problem disappears, investigate shared state and race conditions.

---

## 95. Debugging a Specific Test

Use:

```bash
npx playwright test   tests/login.spec.ts   -g "login successfully"
```

Then run:

```bash
npx playwright test   tests/login.spec.ts   -g "login successfully"   --headed   --workers=1
```

---

## 96. Enterprise Failure Investigation Workflow

```text
CI Failure
    ↓
Open Report
    ↓
Identify Test
    ↓
Check Retry
    ↓
Open Trace
    ↓
Check Screenshot
    ↓
Check Video
    ↓
Check Console
    ↓
Check Network
    ↓
Classify Root Cause
    ↓
Reproduce
    ↓
Fix
    ↓
Run Targeted Test
    ↓
Run Related Suite
    ↓
Run Full Regression
```

---

## 97. Debugging Reports

An HTML report should help identify:

```text
Passed
Failed
Skipped
Flaky
Duration
Project
Retries
Artifacts
```

Use reports as the entry point for CI failure investigation.

---

## 98. Artifact Retention

Keep enough history to investigate recurring problems.

For example:

```text
PR artifacts → short retention
Nightly artifacts → longer retention
Release artifacts → longest retention
```

Follow organization security and storage policies.

---

## 99. Sensitive Diagnostic Data

Diagnostics can accidentally expose:

```text
Passwords
Tokens
Cookies
Personal information
API payloads
Internal URLs
```

Before storing or sharing artifacts, review what is captured.

---

## 100. Safe Logging

Good:

```text
POST /api/users → 201
```

Risky:

```text
Authorization: Bearer eyJ...
```

Never expose secrets in reports or CI logs.

---

## 101. Debugging Best Practices

1. Start with the actual error.
2. Use trace viewer for difficult failures.
3. Capture screenshots on failure.
4. Capture video selectively.
5. Monitor console errors.
6. Monitor page errors.
7. Monitor failed network requests.
8. Use test attachments for structured diagnostics.
9. Reproduce failures with one worker.
10. Avoid unnecessary fixed waits.
11. Avoid unnecessary force actions.
12. Protect sensitive diagnostic data.
13. Compare first attempts and retries.
14. Classify failures by root cause.
15. Fix the root cause instead of masking symptoms.

---

## 102. Interview Questions

### Q1. What is Trace Viewer?

A Playwright diagnostic tool that provides a detailed timeline of test actions, snapshots, network activity, console information, and other execution data.

### Q2. When should you capture traces?

A common CI strategy is `on-first-retry`, while `on` can be useful for local debugging.

### Q3. Why capture screenshots?

They provide visual evidence of the browser state at failure.

### Q4. Why capture video?

It provides a visual recording of the test execution and can help understand transitions and unexpected UI behavior.

### Q5. How do you detect browser JavaScript errors?

Use:

```ts
page.on(
  'pageerror',
  handler
);
```

and optionally monitor console messages.

### Q6. How do you detect failed network requests?

Use:

```ts
page.on(
  'requestfailed',
  handler
);
```

### Q7. Why should you avoid `waitForTimeout()`?

It creates fixed delays instead of waiting for the actual application condition and can make tests slow and flaky.

### Q8. How do you debug CI-only failures?

Use CI artifacts, compare environment differences, run with one worker locally, and inspect trace/network/console information.

### Q9. How do you debug parallel-only failures?

Run with one worker. If the issue disappears, investigate shared state, data collisions, global variables, or race conditions.

### Q10. What is the goal of diagnostics?

To provide enough evidence to identify the root cause quickly without reproducing the failure repeatedly.

---

## 103. Hands-On Exercises

### Exercise 1 — Trace

Configure:

```text
trace: on-first-retry
```

and inspect a failed test.

### Exercise 2 — Screenshot

Configure screenshots on failure.

### Exercise 3 — Video

Configure videos retained on failure.

### Exercise 4 — Console Errors

Capture browser console errors.

### Exercise 5 — Page Errors

Capture uncaught page errors.

### Exercise 6 — Network Failures

Capture failed requests.

### Exercise 7 — Attachments

Attach a diagnostic JSON file to the test report.

### Exercise 8 — VS Code

Debug a test using a breakpoint.

### Exercise 9 — Flaky Test

Use:

```bash
--repeat-each=10
```

to investigate intermittent behavior.

### Exercise 10 — Enterprise Diagnostics

Create a fixture that automatically captures:

```text
Screenshot
Console errors
Page errors
Failed requests
Test metadata
```

when a test fails.

---

## 104. Enterprise Debugging Challenge

Build a diagnostic framework that provides:

```text
Trace
Screenshot
Video
Console Logs
Page Errors
Network Failures
Test Metadata
Attachments
```

Requirements:

1. Capture trace on first retry.
2. Capture screenshot on failure.
3. Retain video on failure.
4. Capture browser console errors.
5. Capture page errors.
6. Capture failed network requests.
7. Attach diagnostic JSON.
8. Include project and retry information.
9. Support parallel workers.
10. Use test-specific output directories.
11. Protect secrets.
12. Support CI.
13. Detect authentication failures.
14. Detect data failures.
15. Make root-cause investigation easy.

---

## 105. Final Enterprise Architecture

```text
                         Test Execution
                               │
                               ▼
                         Test Failure
                               │
              ┌────────────────┼────────────────┐
              │                │                │
           Trace           Screenshot         Video
              │                │                │
              └────────────────┼────────────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              Browser Diagnostics    Network Diagnostics
                    │                     │
             ┌──────┴──────┐       ┌──────┴─────────┐
             │             │       │                │
          Console       PageError  Requests      Responses
             │             │       │                │
             └─────────────┼───────┴────────────────┘
                           │
                     Test Attachments
                           │
                     Failure Report
                           │
                     Root Cause Analysis
                           │
                       Correct Fix
```

---

## 106. Final Checklist

```text
[ ] Playwright errors understood
[ ] Locator timeout debugging understood
[ ] Strict mode debugging understood
[ ] Navigation timeout debugging understood
[ ] Trace Viewer understood
[ ] Trace configured
[ ] Screenshot strategy configured
[ ] Video strategy configured
[ ] Console logging implemented
[ ] Page error logging implemented
[ ] Network request logging understood
[ ] Failed request logging implemented
[ ] Test attachments understood
[ ] testInfo understood
[ ] Output directories understood
[ ] VS Code debugging practiced
[ ] Inspector practiced
[ ] Flaky test debugging practiced
[ ] Parallel failure debugging practiced
[ ] CI-only failure debugging practiced
[ ] Authentication failures investigated
[ ] Test-data failures investigated
[ ] Browser compatibility failures investigated
[ ] Sensitive diagnostics protected
[ ] Enterprise failure workflow documented
```

---

## 107. Key Takeaways

- Debugging is a core Playwright engineering skill.
- Trace Viewer provides the most complete view of a difficult failure.
- Screenshots show visual state.
- Videos show execution flow.
- Console and page errors reveal application-side problems.
- Network logs reveal backend, authentication, and connectivity problems.
- `testInfo` enables advanced diagnostic attachments and metadata.
- CI failures should preserve enough artifacts for investigation.
- Fixed waits are not a reliable solution to flaky tests.
- Running with one worker is useful for diagnosing concurrency problems.
- Authentication failures can appear as locator failures, so inspect network and session state.
- Test-data collisions are common causes of parallel failures.
- Diagnostic artifacts must be handled securely.
- Enterprise automation should make failures easy to reproduce and classify.
- The goal of diagnostics is root-cause identification, not simply collecting files.

---

## 108. Next Step

**Step 97 — Mastering Playwright Network & API Mocking: Route Interception, Request Modification, Response Mocking, HAR, WebSockets, Service Virtualization, Fault Injection & Enterprise Network Testing**

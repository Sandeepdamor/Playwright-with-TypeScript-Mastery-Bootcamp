# Playwright with TypeScript Mastery Bootcamp

# Step 103 — Mastering Playwright Network Interception & Mocking: page.route(), route.fulfill(), route.continue(), route.abort(), Request/Response Inspection, API Mocking, Dependency Isolation, HAR, WebSockets, Network Conditions & Enterprise Network Testing

## 1. Introduction

Modern web applications depend on many network services:

```text
Browser
 ↓
Frontend
 ↓
API Gateway
 ↓
Backend Services
 ↓
Database
```

They may also communicate with:

```text
Authentication Services
Payment Services
Email Services
File Storage
Third-Party APIs
Analytics
Feature Flag Services
WebSockets
CDNs
```

Testing all external dependencies directly can make tests:

```text
Slow
Flaky
Expensive
Environment-dependent
Difficult to reproduce
```

Playwright provides network capabilities that allow you to:

```text
Inspect Requests
Inspect Responses
Intercept Requests
Modify Requests
Mock Responses
Abort Requests
Continue Requests
Simulate Failures
Replay HAR
Control Dependencies
Test Network Behavior
```

---

## 2. Why Network Testing Matters

Consider a dashboard:

```text
Dashboard
 ├── User API
 ├── Notification API
 ├── Report API
 ├── Analytics API
 └── Feature Flag API
```

If the Notification API is unavailable, the dashboard test may fail even though the feature under test works correctly.

Network interception can isolate that dependency.

---

## 3. Real Network vs Mocked Network

### Real Network

```text
Browser
 ↓
Real API
 ↓
Real Backend
```

Advantages:

```text
Realistic
Integration coverage
Production-like behavior
```

Disadvantages:

```text
Slower
External dependency failures
Test data complexity
Less deterministic
```

### Mocked Network

```text
Browser
 ↓
Mock
 ↓
Controlled Response
```

Advantages:

```text
Fast
Deterministic
Easy failure simulation
Dependency isolation
```

Disadvantages:

```text
Can hide integration problems
Mock maintenance required
Not a replacement for real API tests
```

Use both strategically.

---

## 4. `page.route()`

Playwright provides:

```ts
page.route()
```

to intercept network requests.

Basic structure:

```ts
await page.route(
  '**/api/users',
  async route => {
    // handle request
  }
);
```

The route pattern determines which requests are intercepted.

---

## 5. Route Handler

A route handler receives a `Route` object.

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    console.log(
      route.request().url()
    );

    await route.continue();
  }
);
```

This allows the request to continue to the real server.

---

## 6. `route.continue()`

Use:

```ts
await route.continue();
```

when you want to intercept a request but still send it to the real server.

Useful for:

```text
Inspecting requests
Adding headers
Modifying request data
Debugging
Conditional routing
```

---

## 7. Modifying a Request

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    await route.continue({
      headers: {
        ...route.request().headers(),
        'X-Test-Mode':
          'true'
      }
    });
  }
);
```

This can be useful for test-specific headers.

---

## 8. Request Inspection

The request object can provide information such as:

```text
URL
Method
Headers
Post Data
Resource Type
```

Example:

```ts
const request =
  route.request();

console.log(
  request.method()
);

console.log(
  request.url()
);
```

Avoid logging secrets.

---

## 9. Request Method

Example:

```ts
const method =
  route.request().method();

expect(
  method
).toBe('POST');
```

This is useful when verifying frontend behavior.

---

## 10. Request URL

Example:

```ts
const url =
  route.request().url();

expect(
  url
).toContain(
  '/api/users'
);
```

---

## 11. Request Headers

Example:

```ts
const headers =
  route.request().headers();

expect(
  headers['accept']
).toBeDefined();
```

Do not expose:

```text
Authorization
Cookies
Tokens
API Keys
```

in logs or test reports.

---

## 12. Request Post Data

For requests with bodies:

```ts
const body =
  route.request()
    .postDataJSON();
```

Then:

```ts
expect(
  body.username
).toBe(
  'automation-user'
);
```

Use this to verify frontend request payloads.

---

## 13. `route.fulfill()`

`route.fulfill()` allows you to provide a mocked response.

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        users: [
          {
            id: 1,
            username:
              'mock-user'
          }
        ]
      })
    });
  }
);
```

The browser receives the mocked response instead of the real server response.

---

## 14. Why `route.fulfill()` Is Useful

Use it for:

```text
Unavailable backend
Rare error states
Large datasets
Edge cases
Third-party dependencies
Slow services
Deterministic UI tests
```

---

## 15. Mocking a Success Response

Example:

```ts
await page.route(
  '**/api/dashboard',
  async route => {

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        total:
          100,
        approved:
          80,
        pending:
          20
      })
    });
  }
);
```

Now the UI can be tested against controlled data.

---

## 16. Mocking an Empty Response

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        users: []
      })
    });
  }
);
```

Useful for testing:

```text
Empty state
No results
First-time user experience
```

---

## 17. Mocking a 404

Example:

```ts
await page.route(
  '**/api/users/999',
  async route => {

    await route.fulfill({
      status: 404,
      contentType:
        'application/json',
      body: JSON.stringify({
        message:
          'User not found'
      })
    });
  }
);
```

---

## 18. Mocking a 500

Example:

```ts
await page.route(
  '**/api/reports',
  async route => {

    await route.fulfill({
      status: 500,
      contentType:
        'application/json',
      body: JSON.stringify({
        message:
          'Internal server error'
      })
    });
  }
);
```

Use this to verify error handling.

---

## 19. Mocking a 401

Example:

```ts
await page.route(
  '**/api/profile',
  async route => {

    await route.fulfill({
      status: 401,
      contentType:
        'application/json',
      body: JSON.stringify({
        message:
          'Unauthorized'
      })
    });
  }
);
```

This can test session-expired behavior.

---

## 20. Mocking a 403

Example:

```ts
await page.route(
  '**/api/admin/users',
  async route => {

    await route.fulfill({
      status: 403,
      contentType:
        'application/json',
      body: JSON.stringify({
        message:
          'Forbidden'
      })
    });
  }
);
```

Useful for authorization UI behavior.

---

## 21. `route.abort()`

Use:

```ts
await route.abort();
```

to simulate a failed network request.

Example:

```ts
await page.route(
  '**/api/notifications',
  async route => {

    await route.abort();
  }
);
```

This helps test network failure handling.

---

## 22. Abort Error Codes

You can specify an error type where supported.

Conceptually:

```ts
await route.abort(
  'failed'
);
```

Other scenarios may include:

```text
connectionfailed
timedout
aborted
```

Choose the error that matches the behavior being tested.

---

## 23. Simulating a Timeout

Example concept:

```ts
await page.route(
  '**/api/reports',
  async route => {

    await route.abort(
      'timedout'
    );
  }
);
```

Then verify the UI's timeout/error state.

---

## 24. `route.fallback()`

When multiple route handlers are registered, route fallback behavior can allow another matching handler to process the request.

This is useful in layered network interception.

Design route handlers carefully to avoid confusing precedence.

---

## 25. Route Registration Order

Multiple routes can overlap.

For example:

```text
**/api/**
```

and:

```text
**/api/users
```

can both match.

Use specific patterns for specific tests and understand the route handler order/precedence.

---

## 26. Remove Routes

Playwright allows route handlers to be removed.

Example:

```ts
await page.unroute(
  '**/api/users'
);
```

This is useful when a mock is only required for part of a test.

---

## 27. `page.unrouteAll()`

For cleanup:

```ts
await page.unrouteAll();
```

Use this carefully when multiple tests or fixtures register routes.

---

## 28. Network Mocking Lifecycle

A typical test:

```text
Register Route
 ↓
Navigate
 ↓
Application Sends Request
 ↓
Route Intercepts
 ↓
Mock / Modify / Continue
 ↓
UI Receives Response
 ↓
Verify UI
 ↓
Remove Route
```

Register the route before the request occurs.

---

## 29. Route Before Navigation

Good:

```ts
await page.route(
  '**/api/dashboard',
  handler
);

await page.goto(
  '/dashboard'
);
```

Potentially too late:

```ts
await page.goto(
  '/dashboard'
);

await page.route(
  '**/api/dashboard',
  handler
);
```

If the request already happened, the route will not affect that request.

---

## 30. Mocking with `page.goto()`

Example:

```ts
await page.route(
  '**/api/products',
  async route => {

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        products: []
      })
    });
  }
);

await page.goto(
  '/products'
);
```

This is a common UI mocking pattern.

---

## 31. API Mocking vs API Testing

These are different.

### API Testing

```text
Send request
 ↓
Real API
 ↓
Validate response
```

### API Mocking

```text
UI request
 ↓
Intercept
 ↓
Fake response
 ↓
Validate UI
```

Do not confuse mocked integration with real backend testing.

---

## 32. Dependency Isolation

Suppose a dashboard depends on:

```text
User API
Report API
Notification API
Analytics API
```

If the test focuses on report rendering, mock unrelated dependencies.

Example:

```text
Report API → Real
User API → Real
Notification API → Mock
Analytics API → Mock
```

This creates focused tests.

---

## 33. Third-Party API Mocking

Third-party services may be:

```text
Payment Gateway
Maps
Email
SMS
Analytics
Identity Provider
```

Mock them when the test does not need real integration.

Keep separate integration tests for critical third-party integrations.

---

## 34. Feature Flag Mocking

Example:

```ts
await page.route(
  '**/api/features',
  async route => {

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        newDashboard:
          true
      })
    });
  }
);
```

This allows deterministic testing of feature-flagged UI.

---

## 35. Large Dataset Mocking

Instead of creating thousands of records:

```text
Real database
 ↓
Thousands of rows
```

mock the API response:

```text
UI
 ↓
Mock API
 ↓
Large controlled response
```

Useful for testing:

```text
Pagination
Virtual scrolling
Performance-related rendering
Large result states
```

---

## 36. Dynamic Mock Responses

A route handler can inspect the incoming request and choose a response.

Example:

```ts
await page.route(
  '**/api/users/**',
  async route => {

    const url =
      route.request().url();

    if (
      url.includes('admin')
    ) {

      await route.fulfill({
        status: 200,
        contentType:
          'application/json',
        body:
          JSON.stringify({
            role:
              'Admin'
          })
      });

      return;
    }

    await route.continue();
  }
);
```

---

## 37. Conditional Mocking

You can condition on:

```text
HTTP Method
URL
Query Parameter
Request Body
Header
```

Example:

```ts
const request =
  route.request();

if (
  request.method() ===
  'POST'
) {
  // mock
} else {
  await route.continue();
}
```

---

## 38. Query-Based Mocking

Example:

```text
/api/users?status=ACTIVE
```

The route can inspect the URL and return different datasets.

Useful for testing:

```text
Search
Filters
Pagination
Sorting
```

---

## 39. Request Body-Based Mocking

Example:

```ts
const body =
  route.request()
    .postDataJSON();

if (
  body.role ===
  'Admin'
) {
  // return admin response
}
```

This helps simulate backend behavior based on submitted data.

---

## 40. Modifying Responses

Playwright can intercept a real response and modify it before returning it to the page.

A common pattern is:

```ts
await page.route(
  '**/api/users',
  async route => {

    const response =
      await route.fetch();

    const body =
      await response.json();

    body.users.push({
      id: 999,
      username:
        'injected-user'
    });

    await route.fulfill({
      response,
      body:
        JSON.stringify(body)
    });
  }
);
```

This is useful when you want most of the real response but need controlled changes.

---

## 41. `route.fetch()`

`route.fetch()` can retrieve the real network response.

Conceptually:

```text
Browser Request
 ↓
Route
 ↓
route.fetch()
 ↓
Real Server
 ↓
Real Response
 ↓
Modify
 ↓
route.fulfill()
 ↓
Browser
```

This is powerful for response transformation.

---

## 42. Response Inspection

After fetching a real response:

```ts
const response =
  await route.fetch();

expect(
  response.status()
).toBe(200);
```

Then inspect:

```ts
const body =
  await response.json();
```

---

## 43. Response Modification

Example:

```ts
const response =
  await route.fetch();

const body =
  await response.json();

body.status =
  'MOCKED';

await route.fulfill({
  response,
  body:
    JSON.stringify(body)
});
```

This provides hybrid real/mock testing.

---

## 44. Hybrid Network Testing

Three useful strategies are:

```text
Real
Mocked
Hybrid
```

### Real

Use the actual service.

### Mocked

Replace the service completely.

### Hybrid

Call the real service and modify selected responses.

Choose based on the test objective.

---

## 45. Mocking Error States

Many error states are difficult to reproduce naturally.

Examples:

```text
500
503
Slow response
Malformed data
Empty data
Unauthorized
Forbidden
Connection failure
```

Network mocking makes these deterministic.

---

## 46. Malformed Response Testing

Example:

```ts
await route.fulfill({
  status: 200,
  contentType:
    'application/json',
  body:
    '{ invalid json'
});
```

Use this to test frontend resilience.

Only include malformed responses when the application is expected to handle them safely.

---

## 47. Missing Fields

Example:

```ts
body: JSON.stringify({
  id: 1
})
```

instead of:

```text
id
name
status
department
```

This can expose frontend assumptions.

---

## 48. Null Values

Example:

```ts
body: JSON.stringify({
  name: null,
  status: 'ACTIVE'
})
```

Test how the UI handles nullable backend values.

---

## 49. Empty Arrays

Example:

```ts
body: JSON.stringify({
  users: []
})
```

Validate:

```text
No results message
Empty state
Correct layout
No JavaScript errors
```

---

## 50. Slow Network Simulation

A delayed mocked response can be used to test loading states.

Conceptually:

```text
Request
 ↓
Delay
 ↓
Response
```

Test:

```text
Spinner
Skeleton
Disabled button
Loading text
Timeout handling
```

Keep artificial delays short enough to avoid making the suite unnecessarily slow.

---

## 51. Loading State Testing

A useful test flow:

```text
Start Request
 ↓
Loading UI Visible
 ↓
Response
 ↓
Content Visible
```

This validates intermediate UI states.

---

## 52. Network Failure UI

Simulate:

```text
API Request
 ↓
Network Failure
```

Then verify:

```text
Error message
Retry button
Fallback UI
No broken layout
```

---

## 53. Retry UI

If the application supports retry:

```text
Request
 ↓
Failure
 ↓
Retry
 ↓
Success
```

A mock can fail once and succeed on the next request.

---

## 54. Stateful Mock

Example concept:

```ts
let attempts = 0;

await page.route(
  '**/api/report',
  async route => {

    attempts++;

    if (
      attempts === 1
    ) {
      await route.fulfill({
        status: 500
      });

      return;
    }

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body:
        JSON.stringify({
          status:
            'success'
        })
    });
  }
);
```

This tests retry logic deterministically.

---

## 55. HAR

HAR means:

```text
HTTP Archive
```

It can contain recorded HTTP request and response information.

HAR-based routing can replay previously recorded network traffic.

This is useful for:

```text
Deterministic environments
Offline-like testing
Complex API interactions
Repeatable fixtures
```

---

## 56. HAR Recording

A typical process:

```text
Run Application
 ↓
Capture Network
 ↓
Save HAR
 ↓
Commit/Store Controlled Fixture
 ↓
Replay During Test
```

Be careful with sensitive information.

---

## 57. HAR Security

HAR files may contain:

```text
Cookies
Authorization headers
Personal data
Request bodies
Tokens
```

Never blindly commit HAR files.

Review and sanitize them.

---

## 58. HAR Replay

Conceptually:

```ts
await page.routeFromHAR(
  'fixtures/app.har'
);
```

Then matching requests can be served from the recorded archive.

Configure the behavior according to the application's test needs.

---

## 59. HAR and Determinism

HAR replay can provide:

```text
Stable responses
No external dependency
Faster execution
Repeatable data
```

But it can become stale when the API contract changes.

Treat HAR files as versioned test fixtures.

---

## 60. HAR Maintenance

When API behavior changes:

```text
Application changes
 ↓
HAR becomes stale
 ↓
Tests fail
```

Regenerate and review the HAR when appropriate.

---

## 61. WebSockets

Some applications use WebSockets for:

```text
Chat
Notifications
Live dashboards
Online status
Real-time updates
```

Network testing for WebSockets requires a different strategy from standard REST interception.

---

## 62. WebSocket Inspection

Playwright can observe WebSocket activity through page events.

Conceptually:

```ts
page.on(
  'websocket',
  ws => {

    console.log(
      ws.url()
    );
  }
);
```

Avoid logging sensitive message content.

---

## 63. WebSocket Lifecycle

Typical lifecycle:

```text
Connect
 ↓
Handshake
 ↓
Messages
 ↓
Close
```

Tests can observe:

```text
Connection
Frames
Close events
```

where supported.

---

## 64. WebSocket Testing Use Cases

Examples:

```text
Chat message received
Notification arrives
Live status updates
Connection reconnects
Socket closes
```

---

## 65. WebSocket Mocking Strategy

WebSocket mocking is more specialized than REST mocking.

Possible strategies include:

```text
Controlled backend
Test WebSocket server
Proxy
Application-level abstraction
Network-level test tooling
```

Choose the strategy based on the architecture.

---

## 66. Network Conditions

Real applications may behave differently under:

```text
Slow network
Offline
Intermittent connectivity
High latency
Failed requests
```

Tests should validate important resilience behavior.

---

## 67. Offline Testing

A useful scenario:

```text
Application
 ↓
Offline
 ↓
User action
 ↓
Expected offline behavior
```

Possible behavior:

```text
Offline banner
Queued operation
Error message
Cached data
```

---

## 68. Network-Aware UI

Modern applications may display:

```text
Online
Offline
Reconnecting
Syncing
```

Test each important state.

---

## 69. API Mocking with Fixtures

Network mocking can be wrapped in custom fixtures.

Example concept:

```ts
type Fixtures = {
  mockUsers:
    void;
};

export const test =
  base.extend<Fixtures>({
    mockUsers:
      async ({
        page
      }, use) => {

        await page.route(
          '**/api/users',
          async route => {
            await route.fulfill({
              status: 200,
              contentType:
                'application/json',
              body:
                JSON.stringify({
                  users: []
                })
            });
          }
        );

        await use();
      }
  });
```

This makes repeated mocks reusable.

---

## 70. Mock Helper Pattern

Instead of repeating:

```ts
page.route(...)
```

create helpers:

```ts
async function
mockUsers(
  page: Page,
  users: User[]
) {
  await page.route(
    '**/api/users',
    async route => {

      await route.fulfill({
        status: 200,
        contentType:
          'application/json',
        body:
          JSON.stringify({
            users
          })
      });
    }
  );
}
```

---

## 71. Domain Mock Helpers

A large project may use:

```text
mocks/
├── userMocks.ts
├── expenseMocks.ts
├── reportMocks.ts
├── authMocks.ts
└── notificationMocks.ts
```

This keeps network mocks organized.

---

## 72. Mock Data and Factories

Combine Step 100:

```text
Factory
 ↓
Mock Data
 ↓
Network Route
 ↓
UI
```

Example:

```ts
const user =
  createUser();

await mockUsers(
  page,
  [user]
);
```

---

## 73. Mock Data Reuse

Use the same domain models for:

```text
API Tests
UI Tests
Mock Responses
Fixtures
Factories
```

This reduces inconsistent data shapes.

---

## 74. Mock Contract Drift

A major risk is:

```text
Real API changes
 ↓
Mock remains old
 ↓
UI test still passes
```

This creates false confidence.

Countermeasures:

```text
Contract tests
Periodic real API tests
Typed models
Schema validation
Mock review
```

---

## 75. Mocking Strategy

Do not mock everything.

Prefer:

```text
Critical backend integration → Real
Stable third-party dependency → Mock
Rare failure scenario → Mock
UI loading state → Mock
Backend contract → Real/API test
```

---

## 76. Network Mock Naming

Use descriptive names:

```text
mockSuccessfulUserList
mockEmptyUserList
mockUnauthorizedProfile
mockServerErrorReports
mockSlowDashboard
```

Avoid generic:

```text
mock1
mock2
```

---

## 77. Network Test Organization

Possible structure:

```text
utils/
├── network/
│   ├── mocks/
│   ├── handlers/
│   ├── fixtures/
│   └── helpers/
```

---

## 78. Route Pattern Design

Prefer precise patterns.

Instead of:

```text
**
```

use:

```text
**/api/users
```

or:

```text
**/api/users/**
```

depending on the intended scope.

Broad interception can accidentally affect unrelated requests.

---

## 79. Intercepting Static Assets

Avoid accidentally mocking:

```text
JS
CSS
Images
Fonts
```

unless the test specifically needs to simulate those failures.

Prefer API-specific route patterns.

---

## 80. GraphQL

GraphQL commonly uses a single endpoint:

```text
POST /graphql
```

Different operations may be distinguished by the request body.

A route handler can inspect the GraphQL operation and return different mocks.

---

## 81. GraphQL Operation Mocking

Conceptually:

```ts
const body =
  route.request()
    .postDataJSON();

if (
  body.operationName ===
  'GetUsers'
) {
  // fulfill
}
```

This is useful when many operations share one endpoint.

---

## 82. GraphQL Error Mocking

You can simulate:

```text
HTTP 200
+
GraphQL errors
```

because GraphQL error semantics can differ from REST.

Test according to the application's GraphQL contract.

---

## 83. Network Authentication Testing

Network interception can help test:

```text
Missing token
Expired token
Invalid token
Wrong header
Unauthorized response
```

However, use real authentication flows for authentication tests and mocking for controlled edge cases.

---

## 84. Request Header Injection

Example:

```ts
await page.route(
  '**/api/**',
  async route => {

    await route.continue({
      headers: {
        ...route.request().headers(),
        'X-Test-Run':
          'playwright'
      }
    });
  }
);
```

This can support correlation or test-mode headers.

---

## 85. Network Correlation

Enterprise systems may use:

```text
X-Request-ID
X-Correlation-ID
```

A test can generate a safe identifier and attach it to selected requests.

This helps correlate frontend behavior with backend logs.

---

## 86. Network Assertions

Instead of mocking every request, sometimes simply wait for a real request:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url()
        .includes(
          '/api/users'
        )
  );

await page
  .getByRole(
    'button',
    {
      name:
        'Search'
    }
  )
  .click();

const response =
  await responsePromise;

expect(
  response.status()
).toBe(200);
```

This validates actual network behavior.

---

## 87. `waitForRequest()`

Use when you need to validate an outgoing request.

Example:

```ts
const requestPromise =
  page.waitForRequest(
    request =>
      request.url()
        .includes(
          '/api/users'
        )
  );

await page
  .getByRole(
    'button',
    {
      name:
        'Search'
    }
  )
  .click();

const request =
  await requestPromise;

expect(
  request.method()
).toBe(
  'GET'
);
```

---

## 88. Request vs Response Validation

### Request Validation

Verify:

```text
URL
Method
Headers
Query
Body
```

### Response Validation

Verify:

```text
Status
Headers
Body
```

Both can be valuable.

---

## 89. Network Assertions vs UI Assertions

A strong test may validate:

```text
UI Action
 ↓
Request Sent
 ↓
Response Received
 ↓
UI Updated
```

But avoid asserting implementation details that are not part of the product contract.

---

## 90. Network Event Debugging

Useful events include:

```text
request
response
requestfailed
requestfinished
websocket
```

Use these for targeted debugging rather than permanently logging every network event.

---

## 91. Failed Request Handling

A failed request can be observed:

```ts
page.on(
  'requestfailed',
  request => {

    console.log(
      request.url(),
      request.failure()
    );
  }
);
```

Avoid logging sensitive URLs or query parameters.

---

## 92. Network Failure vs HTTP Error

These are different:

### Network Failure

```text
Request
 ↓
Connection fails
```

### HTTP Error

```text
Request
 ↓
Server responds
 ↓
500
```

Test both when application resilience requires it.

---

## 93. Service Unavailability

Simulate:

```text
503 Service Unavailable
```

and verify:

```text
Retry
Fallback
Error UI
```

---

## 94. Dependency Failure Matrix

Example:

| Dependency | Failure | Expected UI |
|---|---|---|
| Notifications | Timeout | Dashboard still usable |
| Reports | 500 | Error state |
| Profile | 401 | Re-login |
| Admin API | 403 | Access denied |
| Payment | 503 | Retry message |

This makes resilience requirements explicit.

---

## 95. Network Mocking Anti-Patterns

Avoid:

```text
Mocking every API
Overly broad routes
Stale mock data
Secrets in HAR
Ignoring real API integration
Mocks that don't match contracts
Unclear mock names
Permanent global routes
```

---

## 96. Enterprise Network Architecture

A mature framework may use:

```text
tests/
├── api/
├── ui/
└── network/
    ├── mocks/
    ├── fixtures/
    ├── scenarios/
    └── helpers/
```

This separates network behavior from business tests.

---

## 97. Network Scenario Library

Example:

```text
network/scenarios/
├── success.ts
├── empty.ts
├── unauthorized.ts
├── forbidden.ts
├── serverError.ts
├── timeout.ts
└── offline.ts
```

Reusable scenarios reduce duplication.

---

## 98. Enterprise Mock Layer

```text
                    Playwright Test
                           │
                     Network Layer
                           │
          ┌────────────────┼────────────────┐
          │                │                │
        Route            HAR            WebSocket
          │                │                │
       Mock/Modify      Replay          Observe
          │                │                │
          └────────────────┼────────────────┘
                           │
                      Application
```

---

## 99. Network Testing Strategy

Use:

```text
Real API Tests
      ↓
Backend Contract Confidence

Network Mocks
      ↓
UI Edge Cases

Hybrid Interception
      ↓
Controlled Integration

HAR
      ↓
Deterministic Replay

WebSocket Tests
      ↓
Real-Time Features
```

---

## 100. Enterprise Best Practices

1. Mock only what needs to be isolated.
2. Keep critical API integration tests real.
3. Register routes before requests occur.
4. Use precise URL patterns.
5. Avoid broad `**` interception.
6. Never log secrets.
7. Sanitize HAR files.
8. Use factories for mock data.
9. Keep mock data typed.
10. Validate real API contracts regularly.
11. Test HTTP errors and network failures separately.
12. Test loading and retry states.
13. Use API/UI integration strategically.
14. Keep route handlers small.
15. Remove temporary routes when appropriate.
16. Use correlation IDs for debugging.
17. Separate network helpers from page objects.
18. Avoid permanent global mocks without a clear reason.
19. Design mocks for parallel execution.
20. Document which dependencies are mocked and why.

---

## 101. Hands-On Exercise 1 — Basic Route

Intercept:

```text
GET /api/users
```

and continue the request.

Verify the request URL and method.

---

## 102. Hands-On Exercise 2 — Mock Response

Mock:

```text
GET /api/users
```

with three synthetic users.

Verify they appear in the UI.

---

## 103. Hands-On Exercise 3 — Empty State

Return:

```json
{
  "users": []
}
```

Verify the empty-state UI.

---

## 104. Hands-On Exercise 4 — Server Error

Return:

```text
500
```

and verify the error UI.

---

## 105. Hands-On Exercise 5 — Unauthorized

Return:

```text
401
```

and verify that the application handles the expired session correctly.

---

## 106. Hands-On Exercise 6 — Network Failure

Abort the report request and verify the retry/error behavior.

---

## 107. Hands-On Exercise 7 — Request Validation

Capture a POST request and verify:

```text
Method
URL
Headers
Request body
```

---

## 108. Hands-On Exercise 8 — Response Modification

Fetch the real API response, modify one field, and fulfill the modified response.

---

## 109. Hands-On Exercise 9 — Retry Simulation

Return:

```text
First request → 500
Second request → 200
```

and verify the UI retry mechanism.

---

## 110. Hands-On Exercise 10 — HAR

Record a safe non-sensitive HAR fixture and replay it in a deterministic test.

Review it before committing it to the repository.

---

## 111. Hands-On Exercise 11 — WebSocket

Observe a WebSocket connection and verify that the expected socket URL is opened.

---

## 112. Hands-On Exercise 12 — Network Scenario Fixture

Create reusable fixtures for:

```text
Success
Empty
Unauthorized
Forbidden
Server Error
Timeout
```

---

## 113. Enterprise Network Challenge

Build a reusable network automation layer supporting:

```text
Request Inspection
Response Inspection
route()
route.continue()
route.fulfill()
route.abort()
route.fetch()
unroute()
HAR Replay
WebSocket Observation
Failure Simulation
Loading Simulation
Retry Simulation
API/UI Integration
```

Requirements:

1. Create network helper utilities.
2. Create domain mock helpers.
3. Create typed mock data.
4. Mock success responses.
5. Mock empty responses.
6. Mock 401.
7. Mock 403.
8. Mock 404.
9. Mock 500.
10. Simulate network failures.
11. Simulate timeout behavior.
12. Validate outgoing requests.
13. Validate incoming responses.
14. Implement response modification.
15. Implement retry simulation.
16. Add HAR replay safely.
17. Observe WebSocket connections.
18. Support parallel execution.
19. Prevent secrets from appearing in artifacts.
20. Document the network testing architecture.

---

## 114. Final Checklist

```text
[ ] Network interception understood
[ ] page.route() understood
[ ] route.continue() understood
[ ] route.fulfill() understood
[ ] route.abort() understood
[ ] route.fetch() understood
[ ] route.fallback() understood
[ ] unroute() understood
[ ] Request inspection understood
[ ] Response inspection understood
[ ] Request headers understood
[ ] Request body inspection understood
[ ] Response mocking understood
[ ] HTTP error mocking understood
[ ] Network failure simulation understood
[ ] Loading-state simulation understood
[ ] Retry simulation understood
[ ] Hybrid interception understood
[ ] Dependency isolation understood
[ ] Third-party mocking understood
[ ] Feature flag mocking understood
[ ] HAR understood
[ ] HAR security understood
[ ] WebSocket observation understood
[ ] GraphQL interception understood
[ ] Network assertions understood
[ ] waitForRequest understood
[ ] waitForResponse understood
[ ] requestfailed understood
[ ] API/UI integration understood
[ ] Mock contract drift understood
[ ] Network fixture architecture understood
[ ] Enterprise network strategy understood
```

---

## 115. Key Takeaways

- Playwright can intercept and control browser network traffic.
- `page.route()` is the main entry point for request interception.
- `route.continue()` allows the real request to proceed.
- `route.fulfill()` provides a controlled response.
- `route.abort()` simulates network failures.
- `route.fetch()` enables hybrid real-response interception.
- Network mocks are excellent for deterministic UI edge-case testing.
- Mocking everything can create false confidence.
- Critical backend integrations should also be tested against real services.
- Precise route patterns prevent accidental interception.
- API response errors and network failures are different scenarios.
- HAR can provide deterministic network replay but must be sanitized and maintained.
- WebSocket applications require specialized real-time testing strategies.
- Request and response assertions can verify frontend/backend integration behavior.
- Network fixtures and domain mock helpers make large suites easier to maintain.
- Test data factories should be reused for mock data.
- Authentication tokens, cookies, and sensitive request data must never be exposed in logs or artifacts.
- A strong network-testing layer improves reliability without replacing real integration coverage.

---

## 116. Next Step

**Step 104 — Mastering Playwright Advanced Waiting & Synchronization: Auto-Waiting, Explicit Waiting, Locator Assertions, waitForURL, waitForResponse, waitForRequest, Polling, Event Synchronization, Race Conditions, Async Operations & Enterprise Flakiness Prevention**

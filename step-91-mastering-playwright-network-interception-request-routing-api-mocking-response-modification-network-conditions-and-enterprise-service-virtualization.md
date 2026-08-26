# Playwright with TypeScript Mastery Bootcamp

# Step 91 — Mastering Playwright Network Interception, Request Routing, API Mocking, Response Modification, Network Conditions & Enterprise Service Virtualization

## 1. Introduction

Modern applications depend heavily on APIs and network services.

A typical workflow is:

```text
Browser
   ↓
Frontend
   ↓
API Request
   ↓
Backend Service
   ↓
Database / External Service
   ↓
API Response
   ↓
Frontend
```

Playwright can observe, intercept, modify, block, and mock network traffic.

Important APIs include:

```text
page.route()
context.route()
page.unroute()
context.unroute()
route.continue()
route.abort()
route.fulfill()
route.fetch()
page.on('request')
page.on('response')
page.waitForRequest()
page.waitForResponse()
```

Network control is useful for:

- API mocking
- Testing error scenarios
- Simulating slow services
- Blocking third-party resources
- Modifying responses
- Testing unavailable dependencies
- Creating deterministic UI tests
- Service virtualization
- Debugging network behavior

---

## 2. Why Network Interception Matters

Suppose a dashboard depends on:

```text
GET /api/dashboard
```

If the backend is unstable, the UI test can become flaky.

Instead, the test can provide a controlled response:

```text
Test
 ↓
Mock API
 ↓
Deterministic JSON
 ↓
Dashboard UI
```

This allows the test to focus on frontend behavior.

---

## 3. Network Lifecycle

A simplified lifecycle is:

```text
Browser
   ↓
Request
   ↓
Route Handler
   ↓
Continue / Abort / Fulfill
   ↓
Server
   ↓
Response
   ↓
Browser
```

Playwright can intercept the request before it reaches the server.

---

## 4. Observing Requests

Use:

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

This is useful for debugging.

Avoid logging passwords, tokens, API keys, or sensitive request data.

---

## 5. Observing Responses

Use:

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

This helps identify:

```text
200
201
400
401
403
404
500
```

responses.

---

## 6. Request Details

A request provides information such as:

```ts
request.method();
request.url();
request.headers();
request.postData();
```

Example:

```ts
page.on(
  'request',
  request => {
    console.log({
      method:
        request.method(),
      url:
        request.url()
    });
  }
);
```

---

## 7. Response Details

A response provides:

```ts
response.status();
response.url();
response.headers();
```

Example:

```ts
page.on(
  'response',
  response => {
    if (
      response.status() >= 400
    ) {
      console.log(
        'Failed:',
        response.status(),
        response.url()
      );
    }
  }
);
```

---

## 8. Waiting for a Specific Response

Example:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url().includes(
        '/api/users'
      ) &&
      response.status() === 200
  );

await page.getByRole(
  'button',
  { name: 'Load Users' }
).click();

const response =
  await responsePromise;
```

This synchronizes the test with a meaningful network event.

---

## 9. Waiting for a Specific Request

Example:

```ts
const requestPromise =
  page.waitForRequest(
    request =>
      request.url().includes(
        '/api/users'
      ) &&
      request.method() === 'POST'
  );

await page.getByRole(
  'button',
  { name: 'Save' }
).click();

const request =
  await requestPromise;
```

This can validate that the expected API call occurred.

---

## 10. Event Ordering

Bad:

```ts
await page.getByRole(
  'button',
  { name: 'Save' }
).click();

await page.waitForResponse(
  response =>
    response.url().includes(
      '/api/users'
    )
);
```

Better:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url().includes(
        '/api/users'
      )
  );

await page.getByRole(
  'button',
  { name: 'Save' }
).click();

const response =
  await responsePromise;
```

Always establish the listener before triggering the event.

---

## 11. Basic Request Routing

Use:

```ts
await page.route(
  '**/api/users',
  async route => {
    await route.continue();
  }
);
```

This intercepts matching requests and allows them to continue normally.

---

## 12. Route Matching

Examples:

```text
**/api/users
**/api/users/**
**/*.png
**/graphql
**/api/**
```

Use the narrowest pattern that matches the intended request.

Avoid routing every request when only one endpoint matters.

---

## 13. Continue a Request

Example:

```ts
await page.route(
  '**/api/users',
  async route => {
    await route.continue();
  }
);
```

This allows the request to proceed to the server.

---

## 14. Modify Request Headers

Example:

```ts
await page.route(
  '**/api/**',
  async route => {

    const headers = {
      ...route.request().headers(),
      'x-test-mode': 'true'
    };

    await route.continue({
      headers
    });
  }
);
```

This is useful for test-specific headers.

Never place real secrets into test code.

---

## 15. Modify Request Body

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    const body = {
      name: 'Test User',
      role: 'employee'
    };

    await route.continue({
      postData:
        JSON.stringify(body)
    });
  }
);
```

Use this only when changing the outgoing request is part of the test scenario.

---

## 16. Aborting Requests

Use:

```ts
await page.route(
  '**/api/analytics/**',
  async route => {
    await route.abort();
  }
);
```

This simulates a blocked or unavailable resource.

---

## 17. Testing Failed Dependencies

Example:

```ts
await page.route(
  '**/api/recommendations',
  async route => {
    await route.abort(
      'failed'
    );
  }
);
```

Then verify that the UI handles the dependency failure.

---

## 18. Mocking with `route.fulfill()`

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
            name: 'Sandeep'
          }
        ]
      })
    });
  }
);
```

The actual server is not required for this request.

---

## 19. Mocking a 200 Response

```ts
await page.route(
  '**/api/profile',
  async route => {
    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        id: 10,
        name: 'Sandeep',
        role: 'QA Engineer'
      })
    });
  }
);
```

---

## 20. Mocking Error Responses

### 400

```ts
await route.fulfill({
  status: 400,
  contentType:
    'application/json',
  body: JSON.stringify({
    message:
      'Invalid profile data'
  })
});
```

### 401

```ts
await route.fulfill({
  status: 401,
  contentType:
    'application/json',
  body: JSON.stringify({
    message:
      'Unauthorized'
  })
});
```

### 403

```ts
await route.fulfill({
  status: 403,
  contentType:
    'application/json',
  body: JSON.stringify({
    message:
      'Access denied'
  })
});
```

### 404

```ts
await route.fulfill({
  status: 404,
  contentType:
    'application/json',
  body: JSON.stringify({
    message:
      'Resource not found'
  })
});
```

### 500

```ts
await route.fulfill({
  status: 500,
  contentType:
    'application/json',
  body: JSON.stringify({
    message:
      'Internal server error'
  })
});
```

---

## 21. Mocking Empty Data

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

Verify the application's empty state.

---

## 22. Mocking Large Data

```ts
const users = Array.from(
  { length: 100 },
  (_, index) => ({
    id: index + 1,
    name:
      `User ${index + 1}`
  })
);

await route.fulfill({
  status: 200,
  contentType:
    'application/json',
  body: JSON.stringify({
    users
  })
});
```

This is useful for pagination and large-list UI testing.

---

## 23. REST Endpoint Mocking

Typical endpoints:

```text
GET    /api/users
GET    /api/users/101
POST   /api/users
PUT    /api/users/101
DELETE /api/users/101
```

Use URL and HTTP method together when necessary.

---

## 24. Route Based on HTTP Method

```ts
await page.route(
  '**/api/users',
  async route => {

    if (
      route.request().method()
      === 'GET'
    ) {
      await route.fulfill({
        status: 200,
        contentType:
          'application/json',
        body: JSON.stringify({
          users: []
        })
      });

      return;
    }

    await route.continue();
  }
);
```

This prevents accidentally mocking POST requests.

---

## 25. Route Based on Query Parameters

Example:

```ts
await page.route(
  '**/api/users**',
  async route => {

    const url =
      new URL(
        route.request().url()
      );

    const pageNumber =
      url.searchParams.get(
        'page'
      );

    console.log(
      pageNumber
    );

    await route.continue();
  }
);
```

Useful for pagination and search testing.

---

## 26. Mocking Pagination

Example scenarios:

```text
page=1 → users 1-10
page=2 → users 11-20
page=3 → users 21-30
```

The route handler can inspect the query parameter and return the correct dataset.

---

## 27. Mocking Search

Example:

```ts
const url =
  new URL(
    route.request().url()
  );

const search =
  url.searchParams.get(
    'search'
  );
```

Return different deterministic datasets for:

```text
search=sandeep
search=admin
search=unknown
```

---

## 28. Mocking GraphQL

GraphQL often uses:

```text
POST /graphql
```

Multiple operations may share the same endpoint.

Inspect the request body:

```ts
const body =
  route.request()
    .postDataJSON();

console.log(
  body.operationName
);
```

Then mock only the required operation.

---

## 29. GraphQL Operation Mocking

```ts
await page.route(
  '**/graphql',
  async route => {

    const body =
      route.request()
        .postDataJSON();

    if (
      body.operationName ===
      'GetUsers'
    ) {
      await route.fulfill({
        status: 200,
        contentType:
          'application/json',
        body: JSON.stringify({
          data: {
            users: []
          }
        })
      });

      return;
    }

    await route.continue();
  }
);
```

---

## 30. Response Modification with `route.fetch()`

Sometimes you need the real backend response but want to change part of it.

```ts
await page.route(
  '**/api/profile',
  async route => {

    const response =
      await route.fetch();

    const body =
      await response.json();

    body.role =
      'admin';

    await route.fulfill({
      response,
      json: body
    });
  }
);
```

Flow:

```text
Request
 ↓
Real Backend
 ↓
Response
 ↓
Modify
 ↓
Browser
```

---

## 31. Why `route.fetch()` Is Useful

It is useful when:

- Most real data is acceptable
- One field needs modification
- A rare state is difficult to create
- A feature flag must be changed
- A backend response needs controlled manipulation

This is often less brittle than recreating an entire response.

---

## 32. Mocking Loading Delays

For loading-state tests:

```ts
await page.route(
  '**/api/report',
  async route => {

    await new Promise(
      resolve =>
        setTimeout(
          resolve,
          3000
        )
    );

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        report: []
      })
    });
  }
);
```

Use controlled delays only when testing loading behavior.

---

## 33. Testing Loading Indicators

Workflow:

```text
Click Load
   ↓
API request
   ↓
Loading indicator
   ↓
Response
   ↓
Data displayed
```

Example:

```ts
await expect(
  page.getByRole(
    'progressbar'
  )
).toBeVisible();
```

Then verify the data appears.

---

## 34. Blocking Resources

Example:

```ts
await page.route(
  '**/*.{png,jpg,jpeg}',
  async route => {
    await route.abort();
  }
);
```

This can reduce overhead when images are irrelevant.

Do not block resources required by the test.

---

## 35. Blocking Third-Party Analytics

Example:

```ts
await page.route(
  '**/analytics/**',
  async route => {
    await route.abort();
  }
);
```

Use only when analytics is not part of the scenario.

---

## 36. External Service Mocking

Example:

```text
Application
    ↓
Weather API
    ↓
External Provider
```

Mocking changes this to:

```text
Application
    ↓
Mock Weather API
```

This makes tests deterministic and avoids external dependency failures.

---

## 37. Service Virtualization

Service virtualization means simulating dependent services.

Example:

```text
Order Service
      ↓
Payment Service
      ↓
Shipping Service
```

The Payment Service can be simulated as:

```text
Success
Declined
Timeout
Server Error
```

without requiring the real service.

---

## 38. Testing Payment Failure

```ts
await page.route(
  '**/api/payment',
  async route => {
    await route.fulfill({
      status: 402,
      contentType:
        'application/json',
      body: JSON.stringify({
        message:
          'Payment declined'
      })
    });
  }
);
```

Then verify the application's payment failure behavior.

Use only synthetic payment data in tests.

---

## 39. Testing Authentication Failure

Mock:

```text
401 Unauthorized
```

Then verify:

```text
Session expired
 ↓
Login screen
```

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

---

## 40. Testing Authorization Failure

Mock:

```text
403 Forbidden
```

Then verify:

```text
Access denied
```

This is especially useful for role-based applications.

---

## 41. Testing Retry Logic

Scenario:

```text
Request 1 → 500
Request 2 → 500
Request 3 → 200
```

Example:

```ts
let attempts = 0;

await page.route(
  '**/api/data',
  async route => {

    attempts++;

    if (
      attempts < 3
    ) {
      await route.fulfill({
        status: 500,
        body: 'Server error'
      });

      return;
    }

    await route.fulfill({
      status: 200,
      contentType:
        'application/json',
      body: JSON.stringify({
        data: []
      })
    });
  }
);
```

Then verify the application's retry behavior.

---

## 42. Testing Rate Limits

Simulate:

```text
429 Too Many Requests
```

Example:

```ts
await page.route(
  '**/api/search',
  async route => {
    await route.fulfill({
      status: 429,
      contentType:
        'application/json',
      body: JSON.stringify({
        message:
          'Too many requests'
      })
    });
  }
);
```

Verify the retry or user-facing message.

---

## 43. Testing Maintenance Mode

Simulate:

```text
503 Service Unavailable
```

Then verify the maintenance or unavailable state.

---

## 44. Page Routing vs Context Routing

Page-level:

```ts
await page.route(
  '**/api/users',
  handler
);
```

Context-level:

```ts
await context.route(
  '**/api/users',
  handler
);
```

Use `page.route()` when only one page needs the rule.

Use `context.route()` when multiple pages in the same context should share it.

---

## 45. Removing Routes

Use:

```ts
await page.unroute(
  '**/api/users'
);
```

This is useful when a mock should apply only during part of a workflow.

---

## 46. Network Mock Organization

Recommended structure:

```text
playwright/
├── mocks/
│   ├── users.mock.ts
│   ├── dashboard.mock.ts
│   ├── payment.mock.ts
│   └── reports.mock.ts
│
├── test-data/
│   └── mocks/
│       ├── users.json
│       ├── empty.json
│       └── errors.json
│
├── fixtures/
│   └── network.fixture.ts
│
├── pages/
│   └── DashboardPage.ts
│
└── tests/
    └── dashboard.spec.ts
```

---

## 47. Mock Naming Convention

Use scenario-focused names:

```text
mockUsersSuccess()
mockUsersEmpty()
mockUsersUnauthorized()
mockUsersForbidden()
mockUsersServerError()
mockUsersTimeout()
```

The test name should clearly communicate which network scenario is being simulated.

---

## 48. Mock Factory

Example:

```ts
export async function
mockUsers(
  page: Page,
  users: unknown[]
) {
  await page.route(
    '**/api/users',
    async route => {
      await route.fulfill({
        status: 200,
        contentType:
          'application/json',
        body: JSON.stringify({
          users
        })
      });
    }
  );
}
```

Usage:

```ts
await mockUsers(
  page,
  [
    {
      id: 1,
      name: 'Sandeep'
    }
  ]
);
```

---

## 49. Fixture-Based Mocking

For repeated scenarios:

```text
Fixture
 ↓
Network Mock
 ↓
Page
 ↓
Test
```

Possible fixtures:

```text
usersMock
dashboardMock
paymentMock
```

This can reduce repetitive network setup.

---

## 50. Network Mocking and Page Objects

Page Objects should generally describe UI behavior:

```ts
await dashboardPage.open();
await dashboardPage.verifyUsers();
```

Network mocks should remain separate:

```text
DashboardPage
      +
DashboardApiMock
```

This keeps UI and network responsibilities separated.

---

## 51. Network Mocking vs API Testing

Do not use UI network mocking as a replacement for API tests.

Use:

```text
API Tests
 ↓
Backend Contract

UI Tests
 ↓
Frontend Behavior
```

Mocking is especially valuable for frontend edge cases.

---

## 52. Mock vs Real Backend

Use a real backend when:

```text
End-to-end integration is the goal
Backend behavior must be verified
Database state matters
API contracts must be exercised
```

Use mocks when:

```text
Testing frontend edge cases
Simulating rare failures
External service is unavailable
Data is difficult to create
Deterministic UI behavior is required
```

A mature strategy uses both.

---

## 53. Mocking Tradeoffs

### Advantages

- Fast
- Deterministic
- Isolated
- Easy to simulate rare failures
- Less dependency on external services

### Disadvantages

- Can hide backend defects
- Mocks can become stale
- Does not validate real integration
- Requires maintenance

---

## 54. Network Assertions

Request:

```ts
expect(
  request.method()
).toBe('POST');

expect(
  request.url()
).toContain(
  '/api/users'
);
```

Response:

```ts
expect(
  response.status()
).toBe(201);
```

Use these when network-level behavior is part of the test requirement.

---

## 55. Request Body Validation

Example:

```ts
const requestPromise =
  page.waitForRequest(
    request =>
      request.url().includes(
        '/api/users'
      ) &&
      request.method() ===
        'POST'
  );

await page.getByRole(
  'button',
  { name: 'Save' }
).click();

const request =
  await requestPromise;

const body =
  request.postDataJSON();

expect(body.name)
  .toBe('Sandeep');
```

This validates the outgoing API payload.

---

## 56. Response Body Validation

Example:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url().includes(
        '/api/users'
      )
  );

await page.getByRole(
  'button',
  { name: 'Load Users' }
).click();

const response =
  await responsePromise;

const body =
  await response.json();

expect(body.users)
  .toHaveLength(1);
```

---

## 57. Request Failure Monitoring

Use:

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

This is useful for diagnosing real network failures.

---

## 58. Network Security

Never put real:

```text
API keys
JWT tokens
Passwords
Payment credentials
Private customer data
```

inside mocks.

Use synthetic test data.

Redact sensitive headers in diagnostic logs.

---

## 59. Avoid Over-Mocking

Do not automatically mock every endpoint.

If every backend call is mocked, the test may stop validating meaningful integration.

Mock only the dependencies required for the scenario.

---

## 60. Contract Awareness

If mocks represent real API contracts:

```text
API Contract
 ↓
Mock Schema
 ↓
Frontend Test
```

keep them synchronized with backend changes.

A stale mock can make a green test misleading.

---

## 61. Network Testing Strategy

A mature automation strategy can contain:

```text
Unit Tests
   ↓
Component Tests
   ↓
Mocked UI Tests
   ↓
API Tests
   ↓
Integration Tests
   ↓
End-to-End Tests
```

Each layer has a different purpose.

---

## 62. When to Mock

Mock when you need to test:

```text
Rare failures
Unavailable services
Slow services
Edge-case payloads
Permission responses
Frontend-only behavior
External dependencies
```

---

## 63. When Not to Mock

Avoid mocking when the purpose is to verify:

```text
Real backend integration
Database interaction
API contracts
Authentication integration
End-to-end business workflows
Deployment configuration
```

Use appropriate real environments for those tests.

---

## 64. Enterprise Service Virtualization

A service virtualization layer can provide:

```text
Payment Service
 ├── Success
 ├── Declined
 ├── Timeout
 └── 500

Notification Service
 ├── Success
 └── Failure

Reporting Service
 ├── Data
 ├── Empty
 └── Error
```

This enables resilience testing without requiring every dependency to be available.

---

## 65. Network Mock Documentation

Every reusable mock should document:

```text
Endpoint
HTTP Method
Scenario
Status
Payload
Business Purpose
```

Example:

```text
Endpoint:
GET /api/users

Scenario:
Unauthorized

Status:
401

Purpose:
Validate session-expired behavior
```

---

## 66. Network Mock Versioning

When APIs evolve:

```text
API v1
 ↓
Mock v1

API v2
 ↓
Mock v2
```

Avoid silently changing shared mocks in a way that breaks unrelated tests.

---

## 67. Production-Quality Mock Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('display mocked users', async ({
  page
}) => {

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
              name: 'Sandeep',
              role: 'QA Engineer'
            },
            {
              id: 2,
              name: 'Test User',
              role: 'Developer'
            }
          ]
        })
      });
    }
  );

  await page.goto('/users');

  await expect(
    page.getByText('Sandeep')
  ).toBeVisible();

  await expect(
    page.getByText('Test User')
  ).toBeVisible();
});
```

---

## 68. Production-Quality Error Mock

```ts
test('display server error', async ({
  page
}) => {

  await page.route(
    '**/api/dashboard',
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

  await page.goto('/dashboard');

  await expect(
    page.getByRole('alert')
  ).toContainText(
    'Internal server error'
  );
});
```

---

## 69. Production-Quality Response Modification

```ts
test('enable feature through response override', async ({
  page
}) => {

  await page.route(
    '**/api/config',
    async route => {

      const response =
        await route.fetch();

      const body =
        await response.json();

      body.featureEnabled =
        true;

      await route.fulfill({
        response,
        json: body
      });
    }
  );

  await page.goto('/dashboard');

  await expect(
    page.getByText(
      'New Feature'
    )
  ).toBeVisible();
});
```

---

## 70. Interview Questions

### Q1. What is network interception?

It is the ability to observe or control network requests and responses during browser automation.

### Q2. What does `page.route()` do?

It intercepts matching requests and allows the test to continue, abort, or fulfill them.

### Q3. What does `route.fulfill()` do?

It provides a custom response to the browser without requiring the actual backend response.

### Q4. What does `route.continue()` do?

It allows the intercepted request to proceed.

### Q5. What does `route.abort()` do?

It stops the request and simulates a network failure.

### Q6. What is `route.fetch()` useful for?

It obtains the real response so the test can inspect or modify it before returning it to the browser.

### Q7. How do you wait for a specific API response?

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url().includes(
        '/api/users'
      )
  );

await button.click();

const response =
  await responsePromise;
```

### Q8. Why should route patterns be narrow?

Broad routes can unintentionally modify unrelated requests and hide application problems.

### Q9. Should all UI tests mock APIs?

No. A balanced strategy includes mocked UI tests, API tests, integration tests, and real end-to-end tests.

### Q10. What is service virtualization?

It is a strategy for simulating dependent services and multiple service scenarios without relying on the actual external service.

---

## 71. Hands-On Exercises

### Exercise 1 — Request Logging

Log all requests and identify the API used by a page.

### Exercise 2 — Response Logging

Log response status codes and identify failed API responses.

### Exercise 3 — Mock Success

Mock a users endpoint with deterministic data.

### Exercise 4 — Mock Empty State

Return an empty list and verify the empty-state UI.

### Exercise 5 — Mock 401

Return unauthorized and verify login/session behavior.

### Exercise 6 — Mock 403

Return forbidden and verify access-denied behavior.

### Exercise 7 — Mock 500

Return server error and verify retry/error UI.

### Exercise 8 — Response Modification

Fetch a real configuration response and modify a feature flag.

### Exercise 9 — Retry Simulation

Return two failures followed by success and verify retry behavior.

### Exercise 10 — Enterprise Mock Framework

Build reusable mocks:

```text
usersMock
dashboardMock
paymentMock
reportsMock
```

with scenarios:

```text
success
empty
unauthorized
forbidden
notFound
serverError
timeout
```

---

## 72. Key Takeaways

- Playwright can observe and intercept network traffic.
- `page.route()` intercepts matching requests.
- `context.route()` applies routing across pages in a context.
- `route.continue()` sends the request onward.
- `route.abort()` simulates request failure.
- `route.fulfill()` provides a controlled response.
- `route.fetch()` allows modification of real responses.
- `waitForRequest()` validates outgoing requests.
- `waitForResponse()` validates API responses.
- Register event listeners before triggering actions.
- Keep route patterns narrow.
- Mock only what the test needs.
- Use mocks for deterministic frontend edge cases.
- Use real services for integration and end-to-end validation.
- Keep mock data and utilities organized.
- Protect secrets and sensitive information.
- Support parallel execution through isolated test contexts.
- Service virtualization helps simulate complex external dependencies.
- A mature enterprise framework combines mocked tests with API, integration, and real E2E tests.

---

## 73. Enterprise Network Automation Challenge

Build a network virtualization framework for:

```text
Dashboard
   ↓
Users API
   ↓
Payment API
   ↓
Notification API
   ↓
Reporting API
```

Implement:

```text
Users
 ├── Success
 ├── Empty
 ├── 401
 ├── 403
 └── 500

Payment
 ├── Success
 ├── Declined
 ├── Timeout
 └── 500

Notifications
 ├── Success
 └── Failure

Reports
 ├── Data
 ├── Empty
 └── Server Error
```

Requirements:

1. Use reusable route handlers.
2. Keep mock data separate from test logic.
3. Match HTTP methods correctly.
4. Handle query parameters.
5. Support GraphQL operation matching where required.
6. Support response modification.
7. Simulate network failures.
8. Test retry behavior.
9. Avoid broad hidden mocks.
10. Document every mock scenario.
11. Keep secrets out of logs.
12. Make mocks safe for parallel tests.
13. Combine mocked UI tests with real API/integration tests.
14. Integrate the solution with Page Objects and fixtures.
15. Make the framework CI-friendly.

---

## 74. Final Enterprise Architecture

```text
                         Playwright Test
                               │
              ┌────────────────┴────────────────┐
              │                                 │
         Page Objects                      API / Network
              │                                 │
              │                         ┌───────┴────────┐
              │                         │                │
              │                    Route Mocks      Real APIs
              │                         │                │
              │                    Service           Integration
              │                  Virtualization        Tests
              │
              └─────────────── UI Assertions ───────────┘
```

This architecture provides controlled network behavior while preserving separate coverage for real backend integration.

---

## 75. Final Checklist

Before considering network automation production-ready:

```text
[ ] Requests can be observed
[ ] Responses can be observed
[ ] Important API calls can be awaited
[ ] Routes use narrow patterns
[ ] HTTP methods are considered
[ ] REST and GraphQL scenarios are supported
[ ] Success responses are mockable
[ ] Error responses are mockable
[ ] Slow/failing dependencies can be simulated
[ ] Real responses can be modified
[ ] Mock data is isolated
[ ] Sensitive data is protected
[ ] Parallel tests are isolated
[ ] Mock lifecycle is controlled
[ ] Real integration tests still exist
```

---

## 76. Next Step

**Step 92 — Mastering Playwright API Testing, APIRequestContext, REST APIs, Authentication, CRUD Operations, API + UI Integration & Enterprise API Automation**

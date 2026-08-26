# Playwright with TypeScript Mastery Bootcamp

# Step 97 — Mastering Playwright Network & API Mocking: Route Interception, Request Modification, Response Mocking, HAR, WebSockets, Service Virtualization, Fault Injection & Enterprise Network Testing

## 1. Introduction

Modern web applications depend heavily on APIs and external services.

A browser test may involve:

```text
Browser
   ↓
Frontend
   ↓
REST API
   ↓
Database
```

and also:

```text
Frontend
   ↓
Payment Service
   ↓
Notification Service
   ↓
Analytics
   ↓
Third-Party Provider
```

If every test depends on real external systems, automation can become:

- Slow
- Flaky
- Expensive
- Difficult to reproduce
- Difficult to test failure scenarios

Playwright provides powerful network capabilities for:

```text
Request Interception
Request Modification
Response Mocking
Route Fulfillment
Route Aborting
HAR Replay
API Testing
Network Assertions
WebSocket Observation
Fault Injection
Service Virtualization
```

The goal is not to mock everything.

The goal is to control dependencies where doing so makes tests more deterministic and valuable.

---

## 2. Why Network Mocking Matters

Suppose the application calls:

```text
GET /api/users
```

and the backend is unavailable.

Without mocking:

```text
UI Test
 ↓
API
 ↓
Server Failure
 ↓
Test Failure
```

With controlled mocking:

```text
UI Test
 ↓
Mock API
 ↓
Predictable Response
 ↓
UI Validation
```

This allows testing frontend behavior independently.

---

## 3. Real API vs Mocked API

### Real API

```text
Browser
 ↓
Real Backend
 ↓
Real Database
```

Advantages:

- High integration confidence
- Real application behavior
- Real contracts

Disadvantages:

- Slower
- More environment dependencies
- Harder failure simulation

### Mocked API

```text
Browser
 ↓
Mock
 ↓
Controlled Response
```

Advantages:

- Fast
- Deterministic
- Easy failure simulation

Disadvantages:

- Can hide integration problems
- Mock can become outdated

A mature framework uses both.

---

## 4. Network Testing Layers

A good enterprise suite may use:

```text
Unit Tests
   ↓
API Tests
   ↓
Mocked UI Tests
   ↓
Integration Tests
   ↓
End-to-End Tests
```

Each layer provides different confidence.

---

## 5. Playwright `route()`

Playwright can intercept network requests.

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

Now the browser receives the mocked response.

---

## 6. Route Matching

The route pattern can target:

```text
Exact URL
Wildcard
URL pattern
Regular expression
```

Example:

```ts
'**/api/users'
```

matches requests ending with:

```text
/api/users
```

Use precise patterns to avoid accidentally intercepting unrelated requests.

---

## 7. Route Before Navigation

Usually configure important routes before triggering the request.

Example:

```ts
await page.route(
  '**/api/users',
  async route => {
    await route.fulfill({
      status: 200,
      body:
        JSON.stringify({
          users: []
        })
    });
  }
);

await page.goto(
  '/users'
);
```

This ensures the request can be intercepted.

---

## 8. `route.fulfill()`

`fulfill()` provides a custom response.

Example:

```ts
await route.fulfill({
  status: 200,
  contentType:
    'application/json',
  body: JSON.stringify({
    id: 1,
    name: 'Sandeep'
  })
});
```

Useful for:

```text
Success responses
Empty responses
Error responses
Feature states
Edge cases
```

---

## 9. Mocking JSON Responses

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
        products: [
          {
            id: 1,
            name:
              'Test Product',
            price: 100
          }
        ]
      })
    });

  }
);
```

The UI can now be tested against deterministic data.

---

## 10. Mocking Empty Data

Example:

```ts
await route.fulfill({
  status: 200,
  contentType:
    'application/json',
  body: JSON.stringify({
    products: []
  })
});
```

This can test:

```text
No products found
Empty state
No search results
```

---

## 11. Mocking Server Errors

Example:

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

Test:

```text
API Failure
 ↓
Frontend Error Handling
 ↓
Error Message
 ↓
Retry Option
```

---

## 12. Mocking Unauthorized Responses

Example:

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

Useful for testing:

```text
Session Expired
Login Redirect
Authentication Error
```

---

## 13. Mocking Forbidden Responses

Example:

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

Useful for authorization testing.

---

## 14. Mocking Not Found

Example:

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

Test:

```text
Missing Resource
 ↓
UI Handling
```

---

## 15. Mocking Rate Limiting

Example:

```ts
await route.fulfill({
  status: 429,
  headers: {
    'Retry-After':
      '30'
  },
  contentType:
    'application/json',
  body: JSON.stringify({
    message:
      'Too many requests'
  })
});
```

This allows controlled testing of rate-limit behavior.

---

## 16. Mocking Slow Responses

You can delay a mocked response.

Example:

```ts
await new Promise(
  resolve =>
    setTimeout(
      resolve,
      2000
    )
);

await route.fulfill({
  status: 200,
  body:
    JSON.stringify({
      data: []
    })
});
```

Use this for controlled latency testing.

Avoid arbitrary delays in normal tests.

---

## 17. `route.abort()`

A request can be intentionally aborted.

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    await route.abort(
      'failed'
    );

  }
);
```

Useful for testing:

```text
Network Failure
Offline Behavior
Retry Logic
Fallback UI
```

---

## 18. Abort Reasons

Depending on the Playwright API, abort reasons can simulate conditions such as:

```text
failed
aborted
timedout
connectionreset
```

Use only the behavior required by the test.

---

## 19. Request Modification

Instead of replacing the response, you can modify the request.

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    const request =
      route.request();

    await route.continue({
      headers: {
        ...request.headers(),
        'x-test-mode':
          'true'
      }
    });

  }
);
```

This is useful for controlled test headers.

---

## 20. Modifying Query Parameters

Conceptually:

```ts
const url =
  new URL(
    route.request().url()
  );

url.searchParams.set(
  'page',
  '2'
);

await route.continue({
  url:
    url.toString()
});
```

Useful for testing request variations.

---

## 21. Modifying HTTP Method

Request modification can be used carefully to test controlled scenarios.

Example concept:

```ts
await route.continue({
  method:
    'POST'
});
```

Do not change methods unless the test intentionally validates method handling.

---

## 22. Modifying POST Data

For controlled test cases, request payloads can be modified.

Conceptually:

```ts
await route.continue({
  postData:
    JSON.stringify({
      name:
        'Updated User'
    })
});
```

Ensure the resulting request matches the server contract.

---

## 23. Request Headers

Headers can be inspected:

```ts
const headers =
  route.request()
    .headers();
```

Useful for verifying:

```text
Authorization
Content-Type
Correlation ID
Feature flags
Custom headers
```

Never print secret header values.

---

## 24. Request Method Assertion

You can inspect:

```ts
route.request()
  .method();
```

Example:

```ts
expect(
  route.request()
    .method()
).toBe('POST');
```

This can validate frontend API behavior.

---

## 25. Request URL Assertion

Example:

```ts
expect(
  route.request().url()
).toContain(
  '/api/users'
);
```

This can verify that the UI calls the expected endpoint.

---

## 26. Request Post Data

Example:

```ts
const data =
  route.request()
    .postDataJSON();

expect(
  data.role
).toBe('employee');
```

Use carefully if payloads contain personal or sensitive data.

---

## 27. `route.continue()`

`continue()` allows the request to proceed, optionally with modifications.

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    await route.continue();
  }
);
```

This is useful for observing or modifying real network traffic.

---

## 28. `route.fetch()`

`route.fetch()` can obtain the real response first.

Conceptually:

```ts
const response =
  await route.fetch();
```

Then you can modify the response before fulfilling it.

This is powerful for partial mocking.

---

## 29. Partial Response Modification

Example:

```ts
await page.route(
  '**/api/users',
  async route => {

    const response =
      await route.fetch();

    const json =
      await response.json();

    json.users.push({
      id: 999,
      name:
        'Mock User'
    });

    await route.fulfill({
      response,
      body:
        JSON.stringify(json)
    });

  }
);
```

This preserves most real backend behavior while injecting controlled data.

---

## 30. Why Partial Mocking Is Useful

Suppose the backend returns:

```text
100 real users
```

but the test needs:

```text
100 real users
+
1 special user
```

Partial mocking avoids duplicating the complete API response.

---

## 31. Response Header Modification

A response can be modified before fulfillment.

Conceptually:

```ts
await route.fulfill({
  response,
  headers: {
    ...response.headers(),
    'x-test-response':
      'mocked'
  }
});
```

Useful for controlled test scenarios.

---

## 32. Response Status Modification

Example concept:

```ts
await route.fulfill({
  response,
  status: 503
});
```

This can simulate backend outages while preserving other response information.

---

## 33. Response Body Modification

Example:

```ts
const response =
  await route.fetch();

const body =
  await response.json();

body.featureEnabled =
  true;

await route.fulfill({
  response,
  body:
    JSON.stringify(body)
});
```

Useful for feature-flag scenarios.

---

## 34. `page.route()` vs `browserContext.route()`

### Page Route

```ts
page.route()
```

Applies to that page.

### Context Route

```ts
context.route()
```

Can apply to pages within the browser context.

Context-level routing is useful when several pages share the same network behavior.

---

## 35. Route Scope

Choose routing scope carefully.

```text
Test
 ↓
Page Route

Context
 ↓
Multiple Pages

Fixture
 ↓
Reusable Network Behavior
```

Avoid global routes that unexpectedly affect unrelated tests.

---

## 36. Removing Routes

Playwright supports removing route handlers.

Conceptually:

```ts
await page.unroute(
  '**/api/users'
);
```

This is useful when a test needs to switch between:

```text
Mocked
Real
Mocked
```

network behavior.

---

## 37. Route Handler Ordering

When multiple routes match, handler configuration matters.

Avoid overlapping patterns such as:

```text
**/api/**
**/api/users
```

unless the behavior is intentional and well understood.

Keep route patterns specific.

---

## 38. Network Assertions

You can wait for requests:

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url()
        .includes(
          '/api/users'
        ) &&
      response.status() ===
        200
  );

await page.getByRole(
  'button',
  {
    name:
      'Load Users'
  }
).click();

const response =
  await responsePromise;
```

This validates the actual network interaction.

---

## 39. `waitForRequest()`

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

await page.getByRole(
  'button',
  {
    name:
      'Load Users'
  }
).click();

const request =
  await requestPromise;
```

Useful for verifying frontend requests.

---

## 40. `waitForResponse()` vs `waitForRequest()`

### `waitForRequest()`

Use when you care about:

```text
URL
Method
Headers
Request body
```

### `waitForResponse()`

Use when you care about:

```text
Status
Response headers
Response body
```

---

## 41. Avoid Waiting After the Action

Risky:

```ts
await page.click(
  'button'
);

await page.waitForResponse(
  '**/api/users'
);
```

The response may already have occurred.

Better:

```ts
const responsePromise =
  page.waitForResponse(
    '**/api/users'
  );

await page.click(
  'button'
);

await responsePromise;
```

Set up the listener before the action that triggers the request.

---

## 42. API Request Assertions

Example:

```ts
const response =
  await request.get(
    '/api/users'
  );

expect(
  response.status()
).toBe(200);
```

This belongs to API testing rather than UI network interception.

---

## 43. UI + API Validation

A strong test can combine:

```text
UI Action
 ↓
API Request
 ↓
Backend State
 ↓
UI Verification
```

Example:

```text
Create User in UI
 ↓
POST /users
 ↓
GET /users/123
 ↓
Verify User
```

Use this selectively so tests remain focused.

---

## 44. HAR Files

HAR stands for:

```text
HTTP Archive
```

It can record network traffic.

Conceptually:

```text
Real Application
 ↓
Network Traffic
 ↓
HAR File
 ↓
Replay
 ↓
Deterministic Test
```

---

## 45. HAR Replay

HAR can be used to replay previously captured network responses.

This is useful when:

```text
External service is unstable
Network data is expensive
Offline testing is needed
A deterministic response set is required
```

---

## 46. HAR Advantages

Advantages:

```text
Deterministic
Fast
Reusable
Good for external APIs
Useful for debugging
```

---

## 47. HAR Limitations

Limitations:

```text
Can become stale
May contain sensitive data
May not represent current API behavior
Can hide integration issues
Requires maintenance
```

Never commit HAR files containing secrets or sensitive production traffic.

---

## 48. HAR Security

HAR files may contain:

```text
Cookies
Authorization headers
Personal data
Request bodies
Internal URLs
```

Treat them as sensitive artifacts unless sanitized.

---

## 49. Service Virtualization

Service virtualization replaces unavailable external dependencies.

Example:

```text
Application
 ↓
Payment Service
```

During test:

```text
Application
 ↓
Mock Payment Service
```

This allows testing:

```text
Payment Success
Payment Failure
Timeout
Declined Card
Service Unavailable
```

without calling a real payment provider.

---

## 50. External Dependency Mocking

Common candidates:

```text
Payment
SMS
Email
Maps
Analytics
Shipping
Identity Provider
Third-Party APIs
```

Mock only when the dependency is not the subject of the test.

---

## 51. What Should Not Be Mocked?

Avoid mocking:

```text
Core functionality being validated
Critical integration under test
Internal behavior that must be verified end-to-end
```

For example, if the goal is to validate:

```text
Frontend ↔ Backend Integration
```

mocking the backend removes the integration being tested.

---

## 52. Mocking Strategy

Ask:

```text
What is this test validating?
```

If testing UI state:

```text
Mock API
```

If testing API integration:

```text
Use real API
```

If testing external provider failure:

```text
Mock external provider
```

---

## 53. Fault Injection

Fault injection deliberately introduces failures.

Examples:

```text
500
503
Timeout
Network failure
Slow response
Malformed JSON
Empty response
Unauthorized
Forbidden
Rate limit
```

This tests resilience.

---

## 54. Testing Retry Logic

Simulate:

```text
First request → 500
Second request → 200
```

Conceptually:

```ts
let attempts = 0;

await page.route(
  '**/api/data',
  async route => {

    attempts++;

    if (attempts === 1) {

      await route.fulfill({
        status: 500,
        body:
          JSON.stringify({
            message:
              'Temporary failure'
          })
      });

      return;
    }

    await route.fulfill({
      status: 200,
      body:
        JSON.stringify({
          data: []
        })
    });
  }
);
```

This validates retry behavior.

---

## 55. Testing Timeout Handling

A controlled delayed response can simulate:

```text
Slow API
 ↓
Frontend timeout
 ↓
Error UI
```

Keep timeout tests deterministic.

---

## 56. Testing Malformed Responses

Example:

```ts
await route.fulfill({
  status: 200,
  contentType:
    'application/json',
  body:
    '{invalid-json'
});
```

This can test frontend parsing error handling.

Use only when the application is expected to handle malformed responses.

---

## 57. Testing Empty Response Bodies

Example:

```ts
await route.fulfill({
  status: 204
});
```

Useful for APIs that intentionally return no content.

---

## 58. Testing Redirects

Network mocking can simulate:

```text
302
301
307
308
```

Test how the application handles redirects where relevant.

---

## 59. Testing CORS Failures

CORS behavior depends on browser and server configuration.

For realistic CORS testing, prefer a controlled server configuration.

Do not assume that simply changing a response header always reproduces the complete browser security behavior.

---

## 60. Testing Offline Behavior

A route can be aborted to simulate network failure.

Example:

```ts
await page.route(
  '**/api/**',
  async route => {
    await route.abort(
      'failed'
    );
  }
);
```

Then verify:

```text
Offline Message
Retry Button
Cached Data
Fallback UI
```

---

## 61. Testing Specific API Failure

Do not abort every request unless required.

Prefer:

```ts
await page.route(
  '**/api/orders',
  async route => {
    await route.abort(
      'failed'
    );
  }
);
```

This keeps the rest of the application functional.

---

## 62. Mocking GraphQL

GraphQL often uses:

```text
POST /graphql
```

The operation is usually identified in the request body.

A route can inspect the payload:

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
        body:
          JSON.stringify({
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

## 63. Mocking REST vs GraphQL

### REST

Usually route by:

```text
HTTP method
URL
Query
```

### GraphQL

Usually route by:

```text
Endpoint
Operation name
Variables
Query
```

GraphQL mocking should be specific enough to avoid intercepting unrelated operations.

---

## 64. WebSockets

WebSockets provide persistent connections:

```text
Client
 ↕
WebSocket
 ↕
Server
```

Common use cases:

```text
Chat
Notifications
Live dashboards
Collaboration
Real-time status
```

Playwright supports observing and controlling WebSocket traffic through its browser networking capabilities.

---

## 65. WebSocket Testing

Typical validation:

```text
Open Page
 ↓
WebSocket Connection
 ↓
Receive Event
 ↓
UI Updates
```

Example scenarios:

```text
Connected
Disconnected
Reconnected
Message Received
Invalid Message
Server Error
```

---

## 66. WebSocket Observation

A page can listen for WebSocket events.

Conceptually:

```ts
page.on(
  'websocket',
  ws => {

    console.log(
      'WebSocket:',
      ws.url()
    );

    ws.on(
      'framereceived',
      data => {
        console.log(
          'Received:',
          data
        );
      }
    );

  }
);
```

Avoid logging sensitive real-time payloads.

---

## 67. WebSocket Frame Debugging

Useful information:

```text
Connection URL
Sent Frames
Received Frames
Connection Errors
Close Events
```

This can help debug real-time applications.

---

## 68. WebSocket Failure Scenarios

Test:

```text
Initial connection fails
Connection closes
Server sends invalid data
Network disconnects
Reconnect succeeds
Reconnect fails
```

Use controlled test infrastructure where possible.

---

## 69. Service Virtualization Architecture

```text
                  Test
                   │
                   ▼
               Application
                   │
          ┌────────┴────────┐
          │                 │
       Internal API     External API
          │                 │
       Real Service    Virtual Service
                            │
                     Controlled Response
```

This isolates unstable external dependencies.

---

## 70. Network Mocking with Fixtures

Network behavior can be placed in fixtures.

Example:

```ts
type Fixtures = {
  mockUsersApi:
    void;
};

export const test =
  base.extend<Fixtures>({
    mockUsersApi:
      async (
        { page },
        use
      ) => {

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

This creates reusable mock behavior.

---

## 71. Scenario-Based Mock Fixtures

Instead of hard-coding one response, define scenarios:

```text
success
empty
unauthorized
forbidden
serverError
timeout
```

Example concept:

```ts
type ApiScenario =
  | 'success'
  | 'empty'
  | 'serverError';
```

The fixture can select the scenario required by the test.

---

## 72. Mock Data Factories

Keep response creation separate:

```ts
function buildUsersResponse() {
  return {
    users: [
      {
        id: 1,
        name:
          'Automation User'
      }
    ]
  };
}
```

Then:

```ts
await route.fulfill({
  status: 200,
  contentType:
    'application/json',
  body:
    JSON.stringify(
      buildUsersResponse()
    )
});
```

---

## 73. Mock Data Should Be Realistic

Bad mock:

```json
{
  "name": "x"
}
```

Better:

```json
{
  "id": 101,
  "name": "Automation User",
  "email": "automation@example.test",
  "role": "employee"
}
```

Use realistic but synthetic data.

---

## 74. Mock Data Maintenance

Mock responses should evolve with API contracts.

If the backend changes:

```text
API Contract
 ↓
Mock Data
 ↓
Tests
```

all three may need updates.

Treat mocks as code.

---

## 75. Contract Drift

A dangerous situation:

```text
Real API
 ↓
Changed response

Mock
 ↓
Old response

Tests
 ↓
Still pass
```

This creates false confidence.

Use real API tests and contract validation to catch drift.

---

## 76. Mock vs Contract Testing

Mocking validates:

```text
Frontend behavior
```

Contract testing validates:

```text
Consumer
 ↕
Provider
```

Use both when the architecture requires strong API compatibility guarantees.

---

## 77. Network Mocking and CI

Mocks can improve CI stability:

```text
External Service
 ↓
Mock
 ↓
Deterministic CI
```

But critical integration tests should still exercise real services in controlled environments.

---

## 78. Network Mocking and Parallelism

Each test should ideally own its routing state.

Avoid:

```text
Global mutable route configuration
```

that can affect parallel tests.

Prefer:

```text
Test-scoped route
Fixture-scoped route
Context-scoped route
```

---

## 79. Route Cleanup

Routes should not unexpectedly leak between tests.

Use test/page/context lifecycle appropriately.

If manually registering and removing routes:

```ts
await page.unroute(
  '**/api/users'
);
```

or let the page/context lifecycle clean up the routing.

---

## 80. Network Debugging Checklist

When an API-backed test fails:

```text
[ ] Is request sent?
[ ] Is URL correct?
[ ] Is method correct?
[ ] Are headers correct?
[ ] Is authentication valid?
[ ] Is request body correct?
[ ] What is status code?
[ ] Is response body correct?
[ ] Is frontend handling response?
[ ] Is route accidentally intercepting it?
```

---

## 81. Common Mocking Mistakes

### Mistake 1

Mocking the system under test.

### Mistake 2

Using overly broad route patterns.

### Mistake 3

Creating unrealistic mock data.

### Mistake 4

Never running real integration tests.

### Mistake 5

Committing sensitive HAR files.

### Mistake 6

Leaving routes active unexpectedly.

### Mistake 7

Mocking authentication without testing real authentication.

### Mistake 8

Using mocks that no longer match the API contract.

### Mistake 9

Testing only happy paths.

### Mistake 10

Using network mocking to hide real defects.

---

## 82. Best Practices

1. Mock only dependencies that need control.
2. Keep route patterns specific.
3. Register routes before triggering requests.
4. Use `fulfill()` for deterministic responses.
5. Use `continue()` for real traffic with controlled modification.
6. Use `fetch()` for partial response modification.
7. Use `abort()` for network-failure scenarios.
8. Validate requests with `waitForRequest()`.
9. Validate responses with `waitForResponse()`.
10. Keep mock data realistic.
11. Keep mocks synchronized with API contracts.
12. Protect HAR files.
13. Test real integrations separately.
14. Use fixtures for reusable mock scenarios.
15. Avoid global mutable network state.
16. Test failure and recovery paths.
17. Use synthetic data.
18. Monitor WebSocket behavior for real-time applications.

---

## 83. Enterprise Network Test Architecture

```text
                         Playwright
                             │
                  ┌──────────┴──────────┐
                  │                     │
                UI Test              API Test
                  │                     │
             Network Layer         API Client
                  │                     │
       ┌──────────┼──────────┐          │
       │          │          │          │
     Real       Mock       Fault      Real API
     API        API       Injection      │
       │          │          │            │
       └──────────┴──────────┘            │
                  │                       │
             Application                  │
                  │                       │
             Verification                │
                  │                       │
               Reports
```

---

## 84. Enterprise Scenario Matrix

| Scenario | Network Strategy |
|---|---|
| UI happy path | Real API or controlled environment |
| Empty state | Mock |
| 401 | Mock |
| 403 | Mock |
| 404 | Mock |
| 500 | Mock |
| Timeout | Fault injection |
| External provider unavailable | Service virtualization |
| API integration | Real API |
| Contract validation | Real provider/contract test |
| WebSocket reconnect | Controlled real/test server |

---

## 85. Interview Questions

### Q1. What is network interception?

Intercepting browser network requests so the test can inspect, modify, continue, fulfill, or abort them.

### Q2. What is `route.fulfill()`?

It provides a controlled response to an intercepted request.

### Q3. What is `route.continue()`?

It allows the request to continue, optionally with modifications.

### Q4. What is `route.fetch()`?

It obtains the actual response so it can be inspected or modified before being returned.

### Q5. What is `route.abort()`?

It intentionally terminates a request to simulate network failure.

### Q6. What is HAR?

HTTP Archive, a captured representation of HTTP traffic that can be used for analysis and controlled replay.

### Q7. Why should you not mock every API?

Because excessive mocking removes integration confidence and can allow contract drift.

### Q8. How do you test API failures in UI automation?

Intercept the API and return controlled 4xx/5xx responses or simulate network failures.

### Q9. How do you test retry behavior?

Return a failure on the first request and a success on a later attempt, then verify the application's retry behavior.

### Q10. What is service virtualization?

Replacing an unavailable or expensive external dependency with a controlled simulated service.

---

## 86. Hands-On Exercises

### Exercise 1 — Mock Users

Mock:

```text
GET /api/users
```

with a successful response.

### Exercise 2 — Empty State

Return:

```json
{
  "users": []
}
```

and verify the empty-state UI.

### Exercise 3 — Server Error

Return:

```text
500
```

and verify the error UI.

### Exercise 4 — Unauthorized

Return:

```text
401
```

and verify login/session behavior.

### Exercise 5 — Request Validation

Verify:

```text
HTTP method
URL
Headers
Request body
```

### Exercise 6 — Response Modification

Use:

```text
route.fetch()
```

to modify a real response.

### Exercise 7 — Network Failure

Use:

```text
route.abort()
```

and verify offline behavior.

### Exercise 8 — Retry Logic

Return:

```text
500
then
200
```

and verify retry behavior.

### Exercise 9 — HAR

Capture a controlled API interaction and experiment with replay.

### Exercise 10 — WebSocket

Observe WebSocket messages in a real-time application.

---

## 87. Enterprise Network Challenge

Build a network-testing layer supporting:

```text
Real API
Mock API
Request Modification
Response Modification
Fault Injection
HAR
WebSocket Diagnostics
Service Virtualization
```

Required scenarios:

```text
Success
Empty
401
403
404
429
500
503
Timeout
Network Failure
Malformed Response
Retry
Recovery
```

Requirements:

1. Use route interception.
2. Keep route patterns specific.
3. Build reusable mock fixtures.
4. Create realistic mock data.
5. Validate request methods.
6. Validate request payloads.
7. Validate response status.
8. Validate response bodies.
9. Test network failure.
10. Test retry behavior.
11. Test recovery.
12. Protect sensitive HAR files.
13. Test real API integrations separately.
14. Support parallel execution.
15. Document every mock scenario.

---

## 88. Final Enterprise Architecture

```text
                         Network Test Layer
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
       Real API              Mock API             Fault Injection
          │                     │                     │
          │              ┌──────┼──────┐              │
          │              │      │      │              │
          │            200    4xx    5xx          Timeout
          │                                           │
          └─────────────────────┬─────────────────────┘
                                │
                         Application Under Test
                                │
                   ┌────────────┼────────────┐
                   │            │            │
                  UI           API        WebSocket
                   │            │            │
                   └────────────┼────────────┘
                                │
                           Verification
                                │
                            Diagnostics
                                │
                              Report
```

---

## 89. Final Checklist

```text
[ ] Network interception understood
[ ] page.route() understood
[ ] context.route() understood
[ ] route.fulfill() understood
[ ] route.continue() understood
[ ] route.fetch() understood
[ ] route.abort() understood
[ ] Request modification understood
[ ] Response modification understood
[ ] waitForRequest() understood
[ ] waitForResponse() understood
[ ] REST mocking practiced
[ ] GraphQL mocking understood
[ ] Error responses mocked
[ ] Empty responses mocked
[ ] Network failures tested
[ ] Retry behavior tested
[ ] Fault injection understood
[ ] HAR understood
[ ] HAR security understood
[ ] Service virtualization understood
[ ] WebSocket diagnostics understood
[ ] Mock fixtures created
[ ] Mock data factories created
[ ] Contract drift understood
[ ] Real integration tests preserved
[ ] Parallel-safe network behavior implemented
```

---

## 90. Key Takeaways

- Network control is essential for reliable modern Playwright automation.
- `page.route()` and `context.route()` can intercept browser traffic.
- `route.fulfill()` creates deterministic responses.
- `route.continue()` allows real traffic to proceed.
- `route.fetch()` enables partial response modification.
- `route.abort()` can simulate network failures.
- `waitForRequest()` and `waitForResponse()` validate real network interactions.
- HAR replay can make external dependencies deterministic but requires careful maintenance and security.
- Fault injection allows testing resilience scenarios that are difficult to reproduce naturally.
- Service virtualization is useful for unstable or expensive external services.
- WebSocket diagnostics are important for real-time applications.
- Mocking should complement, not replace, real integration testing.
- Unrealistic or outdated mocks can create false confidence.
- Enterprise network testing should cover success, failure, recovery, timeout, authorization, and rate-limit scenarios.
- Network behavior should remain isolated and safe under parallel execution.

---

## 91. Next Step

**Step 98 — Mastering Playwright Advanced Locator Engineering: CSS, XPath, Role, Text, Label, Placeholder, Test ID, Chaining, Filtering, nth(), Shadow DOM, Frames, Dynamic Elements & Locator Strategy**

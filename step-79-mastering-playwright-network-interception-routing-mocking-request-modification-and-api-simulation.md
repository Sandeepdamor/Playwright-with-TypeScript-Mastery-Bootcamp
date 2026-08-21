# Playwright with TypeScript Mastery Bootcamp

# Step 79 — Mastering Playwright Network Interception, Routing, Mocking, Request Modification & API Simulation

## 1. Introduction

Modern web applications depend heavily on network communication.

A browser may send requests for:

- REST APIs
- GraphQL APIs
- JSON data
- Images
- JavaScript files
- CSS files
- Authentication services
- Third-party services
- Analytics
- Feature flags

Playwright provides powerful network APIs that allow automation engineers to observe, modify, block, mock, and simulate network traffic.

Important APIs include:

```ts
page.on('request')
page.on('response')
page.on('requestfailed')
page.route()
page.unroute()
route.continue()
route.abort()
route.fulfill()
route.fallback()
```

Network control is one of the most important capabilities for building reliable enterprise automation frameworks.

---

## 2. Understanding Browser Network Traffic

A typical application flow looks like:

```text
User Action
    ↓
Browser
    ↓
HTTP Request
    ↓
Backend API
    ↓
HTTP Response
    ↓
Browser
    ↓
UI Update
```

Playwright can observe and control several points in this flow.

For example:

```text
Browser
   │
   ├── Request ──────→ API
   │
   │←──── Response ───
   │
   └── UI
```

---

## 3. Listening to Requests

Playwright provides the `request` event.

```ts
page.on('request', request => {
  console.log(
    request.method(),
    request.url()
  );
});
```

This allows you to inspect outgoing network requests.

---

## 4. Listening to Responses

Use:

```ts
page.on('response', response => {
  console.log(
    response.status(),
    response.url()
  );
});
```

Example:

```ts
page.on('response', response => {
  if (response.url().includes('/api/users')) {
    console.log('Users API status:', response.status());
  }
});
```

---

## 5. Listening to Failed Requests

Use:

```ts
page.on('requestfailed', request => {
  console.log(
    'Failed request:',
    request.url(),
    request.failure()?.errorText
  );
});
```

This is useful for diagnosing:

- Network failures
- DNS issues
- Connection problems
- Aborted requests
- Application dependency failures

---

## 6. Inspecting Request Details

A `Request` object provides useful information.

```ts
page.on('request', request => {
  console.log('URL:', request.url());
  console.log('Method:', request.method());
  console.log('Headers:', request.headers());
  console.log('Post data:', request.postData());
});
```

---

## 7. Inspecting Response Details

A `Response` object provides:

```ts
page.on('response', response => {
  console.log('URL:', response.url());
  console.log('Status:', response.status());
  console.log('Status Text:', response.statusText());
  console.log('Headers:', response.headers());
});
```

---

## 8. Waiting for a Specific Request

Use:

```ts
const requestPromise = page.waitForRequest(
  '**/api/users'
);

await page.getByRole('button', {
  name: 'Load Users'
}).click();

const request = await requestPromise;

console.log(request.url());
```

This is useful when a UI action triggers a known API request.

---

## 9. Waiting for a Specific Response

Use:

```ts
const responsePromise = page.waitForResponse(
  '**/api/users'
);

await page.getByRole('button', {
  name: 'Load Users'
}).click();

const response = await responsePromise;

expect(response.status()).toBe(200);
```

---

## 10. Waiting for a Response with a Predicate

For more precise matching:

```ts
const responsePromise = page.waitForResponse(
  response =>
    response.url().includes('/api/users') &&
    response.request().method() === 'GET' &&
    response.status() === 200
);

await page.getByRole('button', {
  name: 'Load Users'
}).click();

const response = await responsePromise;
```

This is more reliable than matching only a URL.

---

## 11. Using `Promise.all()` with Network Events

A common pattern is:

```ts
const [response] = await Promise.all([
  page.waitForResponse('**/api/users'),
  page.getByRole('button', {
    name: 'Load Users'
  }).click()
]);

expect(response.status()).toBe(200);
```

The response listener is registered before the triggering action.

---

## 12. Introduction to Routing

Playwright can intercept network requests using:

```ts
page.route()
```

Example:

```ts
await page.route('**/api/users', async route => {
  await route.continue();
});
```

This allows the request to continue normally while giving the test an opportunity to inspect or modify it.

---

## 13. Blocking Requests

You can block unwanted resources.

```ts
await page.route('**/*.{png,jpg,jpeg}', async route => {
  await route.abort();
});
```

This can be useful for performance-focused tests.

---

## 14. Blocking Third-Party Requests

Example:

```ts
await page.route('**/analytics/**', async route => {
  await route.abort();
});
```

This can prevent third-party analytics from affecting a test.

Be careful not to block dependencies required by the application itself.

---

## 15. Aborting Specific Requests

```ts
await page.route('**/api/ads', async route => {
  await route.abort();
});
```

Possible abort behavior can be useful when testing application handling of failed dependencies.

---

## 16. Request Mocking with `route.fulfill()`

The most powerful routing feature is response mocking.

Example:

```ts
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      users: [
        {
          id: 1,
          name: 'Sandeep'
        }
      ]
    })
  });
});
```

The real backend is not called.

The browser receives the mocked response.

---

## 17. Why API Mocking Is Useful

Mocking allows tests to simulate:

- Successful responses
- Empty responses
- Server errors
- Unauthorized responses
- Slow APIs
- Invalid data
- Feature flags
- Rare edge cases

This makes UI tests faster and more deterministic.

---

## 18. Mocking a 200 Response

```ts
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      users: [
        {
          id: 1,
          name: 'Alice'
        },
        {
          id: 2,
          name: 'Bob'
        }
      ]
    })
  });
});
```

---

## 19. Mocking an Empty Response

```ts
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      users: []
    })
  });
});
```

Then verify the UI:

```ts
await expect(
  page.getByText('No users found')
).toBeVisible();
```

---

## 20. Mocking a 401 Unauthorized Response

```ts
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 401,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'Unauthorized'
    })
  });
});
```

This is useful for testing authentication behavior.

---

## 21. Mocking a 403 Forbidden Response

```ts
await page.route('**/api/admin/**', async route => {
  await route.fulfill({
    status: 403,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'Access denied'
    })
  });
});
```

Then validate the application's authorization handling.

---

## 22. Mocking a 500 Server Error

```ts
await page.route('**/api/reports', async route => {
  await route.fulfill({
    status: 500,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'Internal server error'
    })
  });
});
```

This allows reliable testing of error states without making the real backend fail.

---

## 23. Modifying an Existing Response

Sometimes you want the real backend response but need to modify it.

Use:

```ts
await page.route('**/api/users', async route => {
  const response = await route.fetch();

  const json = await response.json();

  json.users.push({
    id: 999,
    name: 'Mock User'
  });

  await route.fulfill({
    response,
    json
  });
});
```

This approach is useful when most backend behavior should remain real while one piece of data needs modification.

---

## 24. Understanding `route.fetch()`

`route.fetch()` performs the actual network request.

Example:

```ts
await page.route('**/api/users', async route => {
  const response = await route.fetch();

  console.log(response.status());

  await route.fulfill({
    response
  });
});
```

This creates a powerful hybrid model:

```text
Browser
   ↓
Playwright Route
   ↓
Real API
   ↓
Modify Response
   ↓
Browser
```

---

## 25. Modifying Request Headers

You can modify outgoing headers.

```ts
await page.route('**/api/**', async route => {
  const headers = {
    ...route.request().headers(),
    'x-test-mode': 'true'
  };

  await route.continue({
    headers
  });
});
```

This is useful for:

- Feature flags
- Test environments
- Correlation IDs
- Tenant headers
- Debug headers

---

## 26. Modifying Request Method

You can modify request behavior using `route.continue()`.

Example:

```ts
await page.route('**/api/test', async route => {
  await route.continue({
    method: 'POST'
  });
});
```

Use request modification carefully because changing HTTP semantics can produce unrealistic test behavior.

---

## 27. Modifying POST Data

Example:

```ts
await page.route('**/api/users', async route => {
  await route.continue({
    postData: JSON.stringify({
      name: 'Modified User',
      role: 'QA'
    })
  });
});
```

This can be useful when testing how the application behaves with modified request payloads.

---

## 28. Adding Authentication Headers

Example:

```ts
await page.route('**/api/**', async route => {
  await route.continue({
    headers: {
      ...route.request().headers(),
      Authorization: 'Bearer test-token'
    }
  });
});
```

In real enterprise frameworks, authentication should generally be handled through proper test authentication or storage state rather than hard-coded tokens.

---

## 29. Conditional Routing

You can inspect the request before deciding what to do.

```ts
await page.route('**/api/**', async route => {
  const request = route.request();

  if (request.method() === 'GET') {
    await route.continue();
  } else {
    await route.abort();
  }
});
```

---

## 30. Routing Based on Query Parameters

```ts
await page.route('**/api/users**', async route => {
  const url = new URL(route.request().url());

  if (url.searchParams.get('role') === 'admin') {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        users: [
          { id: 1, name: 'Admin User' }
        ]
      })
    });
  } else {
    await route.continue();
  }
});
```

---

## 31. Routing Based on HTTP Method

```ts
await page.route('**/api/users', async route => {
  const method = route.request().method();

  if (method === 'GET') {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        users: []
      })
    });
  } else {
    await route.continue();
  }
});
```

---

## 32. Using `route.fallback()`

When multiple route handlers are registered, `route.fallback()` allows processing to continue to another matching route handler.

Conceptually:

```text
Route Handler A
      ↓
route.fallback()
      ↓
Route Handler B
      ↓
Network
```

This is useful in advanced layered frameworks.

---

## 33. Unrouting

Remove a route using:

```ts
await page.unroute('**/api/users');
```

This is useful when a mock should only apply during a specific portion of a test.

---

## 34. Route Registration Scope

Routes can be registered at different levels.

### Page level

```ts
await page.route('**/api/users', handler);
```

### Context level

```ts
await context.route('**/api/users', handler);
```

Context-level routing can affect multiple pages within that browser context.

Choose the narrowest appropriate scope.

---

## 35. Route Ordering and Specificity

When multiple routes match the same request, route ordering matters.

Avoid creating many overlapping wildcard routes without a clear strategy.

Prefer specific patterns:

```ts
'**/api/users'
```

over overly broad patterns such as:

```ts
'**/*'
```

Broad routes can unintentionally intercept application dependencies.

---

## 36. Mocking Third-Party APIs

Suppose an application calls:

```text
https://api.example.com/weather
```

You can mock it:

```ts
await page.route(
  '**/weather',
  async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        temperature: 25,
        condition: 'Sunny'
      })
    });
  }
);
```

This prevents tests from depending on an external service.

---

## 37. Testing API Failure Behavior

Mock the dependency failure:

```ts
await page.route('**/api/payment', async route => {
  await route.abort('failed');
});
```

Then verify:

```ts
await expect(
  page.getByText('Payment service unavailable')
).toBeVisible();
```

This is valuable for resilience testing.

---

## 38. Simulating Slow Responses

A route handler can delay before fulfilling:

```ts
await page.route('**/api/users', async route => {
  await new Promise(resolve =>
    setTimeout(resolve, 3000)
  );

  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      users: []
    })
  });
});
```

This allows testing:

- Loading indicators
- Skeleton screens
- Timeouts
- Retry behavior
- User feedback

Avoid excessive artificial delays in the normal test suite.

---

## 39. Testing Retry Logic

A powerful scenario is:

```text
Request 1 → 500
Request 2 → 500
Request 3 → 200
```

Example:

```ts
let attempts = 0;

await page.route('**/api/report', async route => {
  attempts++;

  if (attempts < 3) {
    await route.fulfill({
      status: 500,
      contentType: 'application/json',
      body: JSON.stringify({
        message: 'Temporary failure'
      })
    });
  } else {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        report: 'Ready'
      })
    });
  }
});
```

This is excellent for testing retry mechanisms.

---

## 40. Mocking Feature Flags

Example:

```ts
await page.route('**/api/feature-flags', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      newDashboard: true,
      advancedReports: false
    })
  });
});
```

This lets tests validate different feature configurations.

---

## 41. Mocking Paginated APIs

```ts
await page.route('**/api/users?page=1', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      page: 1,
      totalPages: 2,
      users: [
        { id: 1, name: 'Alice' }
      ]
    })
  });
});
```

You can create different mocks for different pages.

---

## 42. Mocking GraphQL Requests

GraphQL commonly uses POST requests to one endpoint.

Example:

```ts
await page.route('**/graphql', async route => {
  const body = route.request().postDataJSON();

  if (body.operationName === 'GetUsers') {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        data: {
          users: [
            { id: 1, name: 'Sandeep' }
          ]
        }
      })
    });
  } else {
    await route.continue();
  }
});
```

---

## 43. Network Mocking vs API Testing

These are different concepts.

### Network mocking

Used to control what the browser receives.

```text
UI Test
 ↓
Mocked Network
 ↓
UI Assertion
```

### API testing

Calls the API directly.

```text
API Test
 ↓
Backend API
 ↓
API Assertion
```

A mature framework often uses both.

---

## 44. Network Mocking vs Stubbing vs Interception

### Mocking

Provide a controlled fake response.

```ts
route.fulfill(...)
```

### Stubbing

Replace a dependency with predictable behavior.

### Interception

Observe or modify network traffic.

```ts
page.route(...)
```

These concepts overlap but describe different testing intentions.

---

## 45. Page Object Network Strategy

Network configuration should generally not be scattered throughout every Page Object.

Instead, consider:

```text
tests/
fixtures/
network/
mocks/
pages/
utils/
```

For example:

```text
network/
├── user-mocks.ts
├── report-mocks.ts
├── payment-mocks.ts
└── feature-flag-mocks.ts
```

---

## 46. Reusable Mock Function

```ts
import { Page } from '@playwright/test';

export async function mockUsers(
  page: Page,
  users: Array<{ id: number; name: string }>
) {
  await page.route('**/api/users', async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({ users })
    });
  });
}
```

Usage:

```ts
await mockUsers(page, [
  { id: 1, name: 'Sandeep' },
  { id: 2, name: 'Alice' }
]);
```

---

## 47. Fixture-Based Network Mocking

For large frameworks, network mocks can be integrated into fixtures.

Conceptually:

```text
Test
 ↓
Custom Fixture
 ↓
Network Mock
 ↓
Page
```

This provides centralized environment setup while keeping tests clean.

---

## 48. Network Logging Utility

A framework can capture important requests:

```ts
page.on('request', request => {
  if (request.url().includes('/api/')) {
    console.log(
      `[REQUEST] ${request.method()} ${request.url()}`
    );
  }
});

page.on('response', response => {
  if (response.url().includes('/api/')) {
    console.log(
      `[RESPONSE] ${response.status()} ${response.url()}`
    );
  }
});
```

Use logging selectively because excessive network logging can make reports difficult to read.

---

## 49. Capturing API Response JSON

```ts
const response = await page.waitForResponse(
  '**/api/users'
);

const data = await response.json();

expect(data.users.length).toBeGreaterThan(0);
```

This is useful when the UI assertion alone does not provide enough diagnostic information.

---

## 50. Network Assertions

Useful assertions include:

```ts
expect(response.status()).toBe(200);
expect(response.ok()).toBeTruthy();
expect(response.url()).toContain('/api/users');
```

You can also inspect:

```ts
response.headers()
response.request()
```

---

## 51. Authentication and Network Interception

Network interception can be useful for verifying that authentication information is sent.

Example:

```ts
const requestPromise = page.waitForRequest(
  '**/api/profile'
);

await page.goto('/profile');

const request = await requestPromise;

console.log(request.headers());
```

Do not print sensitive credentials or access tokens into test reports.

---

## 52. Security Considerations

Network logs can accidentally expose:

- Access tokens
- Cookies
- Authorization headers
- Personal information
- API keys
- Session identifiers

Never log secrets unnecessarily.

Use sanitized logging:

```ts
console.log({
  method: request.method(),
  url: request.url()
});
```

rather than dumping all headers.

---

## 53. Network Interception and CI/CD

Network mocks are especially valuable in CI because external dependencies may be:

- Unavailable
- Slow
- Rate-limited
- Environment-specific
- Unstable

Mocking controlled dependencies can make UI tests deterministic.

However, do not mock everything.

A healthy strategy combines:

```text
Real backend tests
+
Controlled network mocks
+
API tests
+
End-to-end tests
```

---

## 54. When Not to Mock

Avoid mocking a dependency when the test's purpose is specifically to validate:

- Real backend integration
- Authentication integration
- Contract compatibility
- Database integration
- End-to-end business flow

Mock only what is appropriate for the test's objective.

---

## 55. Common Mistakes

### Mistake 1 — Registering the route after the request

Incorrect:

```ts
await page.goto('/dashboard');

await page.route('**/api/dashboard', handler);
```

The request may already have occurred.

Correct:

```ts
await page.route('**/api/dashboard', handler);

await page.goto('/dashboard');
```

### Mistake 2 — Overly broad routing

Avoid:

```ts
await page.route('**/*', handler);
```

unless you have a very specific reason.

### Mistake 3 — Mocking everything

Over-mocking can produce tests that pass even when the real system is broken.

### Mistake 4 — Ignoring request method

A URL may support both GET and POST.

Match the intended method.

### Mistake 5 — Leaking secrets in logs

Never print authorization tokens or sensitive headers.

---

## 56. Production-Quality Example

```ts
import { test, expect } from '@playwright/test';

test('should display users from mocked API', async ({ page }) => {
  await page.route('**/api/users', async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        users: [
          {
            id: 1,
            name: 'Sandeep'
          },
          {
            id: 2,
            name: 'Alice'
          }
        ]
      })
    });
  });

  await page.goto('/users');

  await expect(
    page.getByText('Sandeep')
  ).toBeVisible();

  await expect(
    page.getByText('Alice')
  ).toBeVisible();
});
```

This test is fast and deterministic because the UI does not depend on a live users API.

---

## 57. Enterprise Network Testing Strategy

A mature Playwright framework can organize network testing as:

```text
Network Layer
│
├── Request Observers
│
├── Response Observers
│
├── Route Interceptors
│
├── API Mocks
│
├── Error Mocks
│
├── Feature Flag Mocks
│
├── Authentication Helpers
│
└── Network Assertions
```

This creates a maintainable architecture for large test suites.

---

## 58. Recommended Mock Categories

Create reusable mocks for common scenarios:

```text
mocks/
├── users/
│   ├── success.ts
│   ├── empty.ts
│   ├── unauthorized.ts
│   └── server-error.ts
├── reports/
│   ├── success.ts
│   ├── empty.ts
│   └── failure.ts
└── payments/
    ├── success.ts
    ├── declined.ts
    └── timeout.ts
```

This makes test intent immediately visible.

---

## 59. Interview Questions

### Q1. How do you intercept a network request in Playwright?

Use:

```ts
await page.route('**/api/users', async route => {
  await route.continue();
});
```

### Q2. How do you mock an API response?

Use:

```ts
await route.fulfill({
  status: 200,
  contentType: 'application/json',
  body: JSON.stringify(data)
});
```

### Q3. How do you block a request?

Use:

```ts
await route.abort();
```

### Q4. How do you modify a request?

Use:

```ts
await route.continue({
  headers: modifiedHeaders
});
```

### Q5. How do you modify a real API response?

Use:

```ts
const response = await route.fetch();

const data = await response.json();

await route.fulfill({
  response,
  json: data
});
```

### Q6. How do you wait for an API response?

```ts
const response = await page.waitForResponse(
  '**/api/users'
);
```

### Q7. What is the difference between `route.continue()` and `route.fulfill()`?

`continue()` sends the request onward, while `fulfill()` provides a response directly to the browser.

### Q8. What is `route.abort()` used for?

It intentionally stops a request, allowing tests to simulate network failures or block unwanted resources.

### Q9. Why should routes be registered before navigation?

The application may make API requests immediately during page load.

### Q10. Should every API be mocked?

No. Mock only dependencies that are appropriate for the test objective.

---

## 60. Hands-On Exercises

### Exercise 1 — Observe Requests

Log all API requests made by the application.

### Exercise 2 — Observe Responses

Log API response status codes.

### Exercise 3 — Mock Users

Return a static users response using `route.fulfill()`.

### Exercise 4 — Mock Empty State

Return an empty array and verify the UI's empty state.

### Exercise 5 — Mock Unauthorized

Return HTTP 401 and verify the application's authentication behavior.

### Exercise 6 — Mock Server Error

Return HTTP 500 and verify the error UI.

### Exercise 7 — Modify a Request

Add a custom test header to API requests.

### Exercise 8 — Modify a Response

Call the real API, modify the JSON, and return the modified response.

### Exercise 9 — Simulate Retry

Return 500 twice and 200 on the third request.

### Exercise 10 — Feature Flag

Mock a feature-flag API and verify both enabled and disabled states.

---

## 61. Key Takeaways

- `page.on('request')` observes outgoing requests.
- `page.on('response')` observes responses.
- `page.on('requestfailed')` observes failed requests.
- `waitForRequest()` waits for a specific request.
- `waitForResponse()` waits for a specific response.
- `page.route()` intercepts network traffic.
- `route.continue()` allows a request to proceed.
- `route.fulfill()` provides a mocked response.
- `route.abort()` blocks or fails a request.
- `route.fetch()` sends the real request so its response can be modified.
- `route.fallback()` supports layered route handling.
- Network mocks improve determinism and allow rare scenarios to be tested.
- Avoid over-mocking real integrations.
- Keep network mocks reusable and organized.
- Never expose secrets through network logging.
- Register routes before the request they are intended to intercept.

---

## 62. Next Step

**Step 80 — Mastering Playwright API Testing, APIRequestContext, REST Operations, Authentication & UI/API Hybrid Testing**

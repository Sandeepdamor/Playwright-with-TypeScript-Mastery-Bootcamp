# Playwright with TypeScript Mastery Bootcamp

# Step 92 — Mastering Playwright API Testing, APIRequestContext, REST APIs, Authentication, CRUD Operations, API + UI Integration & Enterprise API Automation

## 1. Introduction

Modern Playwright automation should not depend only on browser-based UI testing.

Many applications expose REST APIs that can be tested directly.

Playwright provides the `APIRequestContext` API for sending HTTP requests without opening a browser page.

Typical API automation flow:

```text
Test
 ↓
APIRequestContext
 ↓
REST API
 ↓
Response
 ↓
Assertions
```

API testing can be combined with UI testing:

```text
API
 ↓
Create Test Data
 ↓
UI
 ↓
Verify Data
 ↓
API
 ↓
Cleanup
```

This approach is powerful for enterprise test automation because API operations are generally faster and more deterministic than performing every setup and cleanup operation through the UI.

---

## 2. What Is `APIRequestContext`?

`APIRequestContext` is Playwright's API client for sending HTTP requests.

It supports common HTTP methods such as:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
```

Example:

```ts
const response =
  await request.get(
    '/api/users'
  );
```

---

## 3. Importing the API Request Fixture

Playwright provides a built-in `request` fixture.

Example:

```ts
import {
  test,
  expect
} from '@playwright/test';

test(
  'get users',
  async ({ request }) => {

    const response =
      await request.get(
        '/api/users'
      );

    expect(
      response.ok()
    ).toBe(true);
  }
);
```

---

## 4. API Testing vs UI Testing

### UI Test

```text
Open Browser
 ↓
Login
 ↓
Navigate
 ↓
Click
 ↓
Fill Form
 ↓
Submit
```

### API Test

```text
Send POST
 ↓
Receive Response
 ↓
Validate JSON
```

API tests are often faster because they skip browser rendering and UI interaction.

---

## 5. Why API Testing Is Important

API testing helps verify:

- Status codes
- Response bodies
- Headers
- Authentication
- Authorization
- Request payloads
- CRUD operations
- Business rules
- Error handling
- API contracts

---

## 6. HTTP Methods

Common methods:

| Method | Purpose |
|---|---|
| GET | Retrieve data |
| POST | Create data |
| PUT | Replace/update data |
| PATCH | Partially update data |
| DELETE | Delete data |
| HEAD | Retrieve headers |

Understanding HTTP methods is essential for API automation.

---

## 7. Basic GET Request

```ts
const response =
  await request.get(
    '/api/users'
  );

expect(
  response.status()
).toBe(200);
```

---

## 8. Checking Response Status

Use:

```ts
response.status();
```

Example:

```ts
expect(
  response.status()
).toBe(200);
```

For creation:

```ts
expect(
  response.status()
).toBe(201);
```

---

## 9. `response.ok()`

Example:

```ts
expect(
  response.ok()
).toBe(true);
```

This provides a convenient success check.

For detailed API validation, also assert the expected status code.

---

## 10. Reading Response Text

```ts
const body =
  await response.text();

console.log(body);
```

Use this when the response is plain text or when debugging unexpected responses.

---

## 11. Reading JSON Response

```ts
const body =
  await response.json();

console.log(body);
```

Example:

```ts
expect(
  body.users
).toBeDefined();
```

---

## 12. Response Headers

Use:

```ts
const headers =
  response.headers();

console.log(headers);
```

Specific header:

```ts
const contentType =
  response.headers()[
    'content-type'
  ];
```

---

## 13. Response URL

Use:

```ts
console.log(
  response.url()
);
```

This is useful when debugging redirects or complex API workflows.

---

## 14. GET with Query Parameters

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        page: 1,
        limit: 10,
        search: 'sandeep'
      }
    }
  );
```

This is cleaner than manually constructing the query string.

---

## 15. Validating Query Behavior

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        page: 1,
        limit: 10
      }
    }
  );

expect(
  response.status()
).toBe(200);

const body =
  await response.json();

expect(
  body.users
).toBeDefined();
```

---

## 16. POST Request

Example:

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {
        name: 'Sandeep',
        role: 'QA Engineer'
      }
    }
  );

expect(
  response.status()
).toBe(201);
```

---

## 17. JSON Request Body

When using:

```ts
data: {
  name: 'Sandeep'
}
```

Playwright can serialize the object for the request.

This is preferable to manually creating JSON strings for normal JSON APIs.

---

## 18. POST Response Validation

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {
        name: 'Sandeep',
        role: 'QA Engineer'
      }
    }
  );

expect(
  response.status()
).toBe(201);

const body =
  await response.json();

expect(
  body.name
).toBe('Sandeep');
```

---

## 19. PUT Request

Example:

```ts
const response =
  await request.put(
    '/api/users/101',
    {
      data: {
        name: 'Updated User',
        role: 'Manager'
      }
    }
  );

expect(
  response.status()
).toBe(200);
```

Use PUT when the API defines the operation as a full replacement/update.

---

## 20. PATCH Request

Example:

```ts
const response =
  await request.patch(
    '/api/users/101',
    {
      data: {
        role: 'Admin'
      }
    }
  );

expect(
  response.status()
).toBe(200);
```

PATCH is commonly used for partial updates.

---

## 21. DELETE Request

Example:

```ts
const response =
  await request.delete(
    '/api/users/101'
  );

expect(
  response.status()
).toBe(204);
```

The exact success status depends on the API contract.

---

## 22. CRUD Workflow

A complete CRUD test can look like:

```text
POST
 ↓
Create User
 ↓
GET
 ↓
Verify User
 ↓
PUT/PATCH
 ↓
Update User
 ↓
GET
 ↓
Verify Update
 ↓
DELETE
 ↓
Verify Removal
```

---

## 23. Create Test Data Through API

Instead of:

```text
Open UI
 ↓
Login
 ↓
Open Admin
 ↓
Click Add
 ↓
Fill Form
 ↓
Save
```

Use:

```text
POST /api/users
```

This can make UI tests much faster.

---

## 24. API Authentication

Authenticated APIs commonly use:

```text
Bearer Token
Cookie
Basic Authentication
API Key
OAuth2
JWT
```

Playwright can send appropriate headers or authentication state.

---

## 25. Bearer Token

Example:

```ts
const response =
  await request.get(
    '/api/profile',
    {
      headers: {
        Authorization:
          `Bearer ${token}`
      }
    }
  );
```

Never hard-code real production tokens.

---

## 26. API Key

Example:

```ts
const response =
  await request.get(
    '/api/data',
    {
      headers: {
        'x-api-key':
          process.env.API_KEY!
      }
    }
  );
```

Store secrets in environment variables or your CI secret manager.

---

## 27. Basic Authentication

Playwright can create an API request context with authentication options when the API requires them.

Conceptually:

```ts
const api =
  await request.newContext({
    httpCredentials: {
      username:
        process.env.API_USERNAME!,
      password:
        process.env.API_PASSWORD!
    }
  });
```

Close custom contexts after use.

---

## 28. API Request Context Options

A custom request context can configure:

```text
baseURL
extraHTTPHeaders
httpCredentials
storageState
ignoreHTTPSErrors
timeout
```

Example:

```ts
const api =
  await request.newContext({
    baseURL:
      'https://example.test',
    extraHTTPHeaders: {
      Accept:
        'application/json'
    }
  });
```

---

## 29. Base URL

If `baseURL` is configured:

```ts
const response =
  await request.get(
    '/api/users'
  );
```

The relative URL is resolved against the base URL.

This keeps tests clean and environment-independent.

---

## 30. Environment-Based API URL

Example:

```ts
const baseURL =
  process.env.API_BASE_URL;
```

Then:

```ts
const api =
  await request.newContext({
    baseURL
  });
```

This supports:

```text
DEV
QA
STAGING
```

without changing test code.

---

## 31. Custom API Request Context

Example:

```ts
const api =
  await request.newContext({
    baseURL:
      process.env.API_BASE_URL,
    extraHTTPHeaders: {
      Accept:
        'application/json'
    }
  });

const response =
  await api.get(
    '/api/users'
  );

await api.dispose();
```

Use `dispose()` when you create the context yourself.

---

## 32. API Request Fixture vs Custom Context

Built-in fixture:

```ts
async ({ request }) => {
  await request.get(
    '/api/users'
  );
}
```

Custom context:

```ts
const api =
  await request.newContext();
```

Use the fixture for standard scenarios.

Use a custom context when you need different configuration or independent API state.

---

## 33. Authentication with Login API

A common workflow:

```text
POST /auth/login
      ↓
Access Token
      ↓
Authenticated API Requests
```

Example:

```ts
const login =
  await request.post(
    '/auth/login',
    {
      data: {
        username:
          process.env.TEST_USERNAME,
        password:
          process.env.TEST_PASSWORD
      }
    }
  );

expect(
  login.status()
).toBe(200);

const auth =
  await login.json();

const token =
  auth.accessToken;
```

Never print the token in test logs.

---

## 34. Reusing Authentication

After obtaining a token:

```ts
const api =
  await request.newContext({
    baseURL:
      process.env.API_BASE_URL,
    extraHTTPHeaders: {
      Authorization:
        `Bearer ${token}`
    }
  });
```

Then:

```ts
await api.get(
  '/api/profile'
);
```

---

## 35. API Authentication Fixture

A reusable fixture can expose:

```text
authenticatedRequest
```

Then tests can simply use:

```ts
test(
  'get profile',
  async ({
    authenticatedRequest
  }) => {

    const response =
      await authenticatedRequest
        .get('/api/profile');

    expect(
      response.status()
    ).toBe(200);
  }
);
```

This is useful for enterprise test suites.

---

## 36. JWT Authentication

For JWT-based APIs:

```text
Login
 ↓
JWT Access Token
 ↓
Authorization Header
 ↓
Protected API
```

Header:

```ts
Authorization:
  `Bearer ${token}`
```

Keep JWT creation and storage inside a reusable authentication utility or fixture.

---

## 37. Refresh Token Workflow

If the application uses:

```text
Access Token
Refresh Token
```

the test can verify:

```text
Access Token Expired
 ↓
Refresh Endpoint
 ↓
New Access Token
 ↓
Original Request
```

This is an important authentication integration scenario.

---

## 38. API Negative Testing

Do not test only successful APIs.

Cover:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Validation Error
429 Rate Limit
500 Server Error
503 Service Unavailable
```

Use only the statuses defined by the actual API contract.

---

## 39. Validation Error

Example:

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {
        name: ''
      }
    }
  );

expect(
  response.status()
).toBe(400);

const body =
  await response.json();

expect(
  body.message
).toBeDefined();
```

---

## 40. Unauthorized API Test

```ts
const response =
  await request.get(
    '/api/profile'
  );

expect(
  response.status()
).toBe(401);
```

This verifies the API protects authenticated resources.

---

## 41. Forbidden API Test

```ts
const response =
  await request.get(
    '/api/admin'
  );

expect(
  response.status()
).toBe(403);
```

Use an authenticated user without the required permission.

---

## 42. Not Found Test

```ts
const response =
  await request.get(
    '/api/users/999999'
  );

expect(
  response.status()
).toBe(404);
```

Use an ID that is guaranteed to be absent in the test environment.

---

## 43. Conflict Test

Example:

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {
        username:
          'existing-user'
      }
    }
  );

expect(
  response.status()
).toBe(409);
```

This is useful for duplicate-resource scenarios.

---

## 44. API Schema Validation

A strong API test should validate more than status codes.

Validate:

```text
Status
Headers
Body
Required fields
Field types
Business values
```

Example:

```ts
const body =
  await response.json();

expect(body)
  .toMatchObject({
    id: expect.any(Number),
    name: expect.any(String)
  });
```

---

## 45. Response Structure Validation

Example:

```ts
expect(
  body.users
).toEqual(
  expect.arrayContaining([
    expect.objectContaining({
      id:
        expect.any(Number)
    })
  ])
);
```

This checks the response shape.

---

## 46. API Contract Testing

Contract validation ensures that the response matches an agreed structure.

Example:

```text
API Contract
 ↓
Expected Schema
 ↓
Actual Response
 ↓
Pass / Fail
```

For complex APIs, a JSON Schema validation library can be integrated into the project.

---

## 47. Headers Validation

Example:

```ts
expect(
  response.headers()[
    'content-type'
  ]
).toContain(
  'application/json'
);
```

Validate important security or caching headers when they are part of the API requirement.

---

## 48. API Request Headers

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      headers: {
        Accept:
          'application/json',
        'x-test-id':
          'automation'
      }
    }
  );
```

Keep headers relevant to the scenario.

---

## 49. Request Timeout

For APIs that may take longer:

```ts
const response =
  await request.get(
    '/api/report',
    {
      timeout: 60_000
    }
  );
```

Do not increase timeouts blindly.

Investigate slow API behavior first.

---

## 50. API Error Diagnostics

When an API test fails, capture:

```text
Method
URL
Status
Response body
Relevant headers
Request payload
Test data
```

Never include secrets in diagnostics.

---

## 51. API Helper Classes

A reusable API client can encapsulate endpoints.

Example:

```ts
export class UsersApi {
  constructor(
    private readonly request: APIRequestContext
  ) {}

  async getUsers() {
    return this.request.get(
      '/api/users'
    );
  }

  async getUser(id: number) {
    return this.request.get(
      `/api/users/${id}`
    );
  }
}
```

This reduces duplicated endpoint paths.

---

## 52. API Client for CRUD

Example:

```ts
export class UsersApi {

  constructor(
    private readonly request:
      APIRequestContext
  ) {}

  async createUser(
    data: object
  ) {
    return this.request.post(
      '/api/users',
      { data }
    );
  }

  async updateUser(
    id: number,
    data: object
  ) {
    return this.request.put(
      `/api/users/${id}`,
      { data }
    );
  }

  async deleteUser(
    id: number
  ) {
    return this.request.delete(
      `/api/users/${id}`
    );
  }
}
```

---

## 53. API Client vs Page Object

Page Object:

```text
UI interactions
```

API Client:

```text
HTTP interactions
```

Recommended:

```text
pages/
  UsersPage.ts

api/
  UsersApi.ts
```

Keep responsibilities separate.

---

## 54. API + UI Integration

One of the most useful patterns is:

```text
API Setup
 ↓
UI Verification
 ↓
API Cleanup
```

Example:

```text
POST user
 ↓
Open UI
 ↓
Search user
 ↓
Verify user
 ↓
DELETE user
```

This makes setup and cleanup fast.

---

## 55. API Creates UI Test Data

Example:

```ts
const createResponse =
  await request.post(
    '/api/users',
    {
      data: {
        name: 'Automation User'
      }
    }
  );

expect(
  createResponse.status()
).toBe(201);

const user =
  await createResponse.json();

const userId =
  user.id;
```

Then use the UI:

```ts
await page.goto(
  `/users/${userId}`
);
```

---

## 56. UI Creates Data, API Verifies It

Another useful pattern:

```text
UI
 ↓
Create Record
 ↓
API GET
 ↓
Validate Backend Data
```

This verifies that the UI correctly sent data to the backend.

---

## 57. API Cleanup

After a UI test:

```ts
await request.delete(
  `/api/users/${userId}`
);
```

This avoids slow UI cleanup.

Always clean up created resources where appropriate.

---

## 58. Cleanup with `try/finally`

Example:

```ts
let userId: number;

try {

  const response =
    await request.post(
      '/api/users',
      {
        data: {
          name:
            'Automation User'
        }
      }
    );

  const body =
    await response.json();

  userId = body.id;

  // UI test
  await page.goto(
    `/users/${userId}`
  );

} finally {

  if (userId) {
    await request.delete(
      `/api/users/${userId}`
    );
  }
}
```

This improves test isolation.

---

## 59. API Test Data Factories

Create reusable factories:

```ts
export function
buildUser(
  overrides = {}
) {
  return {
    name:
      'Automation User',
    role:
      'QA',
    ...overrides
  };
}
```

Usage:

```ts
const user =
  buildUser({
    role: 'Admin'
  });
```

This makes test data easier to maintain.

---

## 60. Unique Test Data

Parallel tests should avoid duplicate identifiers.

Example:

```ts
const username =
  `user-${Date.now()}`;
```

For high parallelism, use a stronger unique identifier strategy.

Do not use predictable IDs when uniqueness is required by the application.

---

## 61. API Fixture Architecture

Example:

```text
fixtures/
└── api.fixture.ts

api/
├── auth.api.ts
├── users.api.ts
├── reports.api.ts
└── payments.api.ts

test-data/
└── factories/
    └── user.factory.ts
```

This provides clean separation.

---

## 62. Authentication Fixture Architecture

Conceptually:

```text
Test
 ↓
Authenticated API Fixture
 ↓
API Client
 ↓
Protected Endpoint
```

The fixture can handle:

```text
Login
Token
Headers
API Context
Cleanup
```

---

## 63. API + Browser Authentication

For applications using shared authentication mechanisms, API setup can sometimes be combined with browser storage state.

Typical flow:

```text
API Login
 ↓
Create Authentication State
 ↓
Browser Context
 ↓
UI Test
```

This can reduce repeated UI login steps.

Use the application's actual authentication architecture rather than assuming that API tokens and browser cookies are interchangeable.

---

## 64. APIRequestContext and Browser Context

They are separate concepts:

```text
BrowserContext
 └── Page

APIRequestContext
 └── HTTP Requests
```

A test can use both independently.

---

## 65. Parallel API Tests

API tests should be isolated.

Avoid shared mutable data such as:

```text
same user
same invoice
same order
same database record
```

Create test-specific resources whenever possible.

---

## 66. API Idempotency

Understand whether an API operation is safe to repeat.

Examples:

```text
GET → generally safe
PUT → often designed to be idempotent
DELETE → depends on API behavior
POST → generally creates a new resource
```

Do not blindly retry non-idempotent operations.

---

## 67. API Retry Testing

When testing retry behavior:

```text
Request
 ↓
Failure
 ↓
Retry
 ↓
Success
```

Verify both:

```text
Number of attempts
Final result
```

Be careful with real systems where repeated POST operations could create duplicate resources.

---

## 68. API Pagination

Test:

```text
First page
Middle page
Last page
Empty page
Invalid page
Large page size
```

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        page: 2,
        limit: 20
      }
    }
  );
```

Validate both data and pagination metadata.

---

## 69. API Filtering

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        role: 'QA',
        active: true
      }
    }
  );
```

Verify only matching records are returned.

---

## 70. API Sorting

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        sort: 'name',
        direction: 'asc'
      }
    }
  );
```

Validate ordering.

---

## 71. API Search

Test:

```text
Exact match
Partial match
Case behavior
No result
Special characters
Large search term
```

Always follow the actual API contract.

---

## 72. API File Upload

API testing can also cover multipart uploads.

Depending on the endpoint, use the request options supported by Playwright and the API contract.

Conceptually:

```text
POST /api/documents
Content-Type: multipart/form-data
```

Validate:

```text
Status
Document ID
Filename
Upload result
```

---

## 73. API Download

API responses may contain files.

Validate:

```text
Status
Content-Type
Content-Length
Binary content
```

For file workflows, combine API validation with the file utilities learned in Step 90.

---

## 74. API Authentication Negative Matrix

Test:

| Scenario | Expected |
|---|---:|
| No token | 401 |
| Invalid token | 401 |
| Expired token | 401 |
| Valid token | 200 |
| Valid token, wrong role | 403 |
| Valid token, correct role | 200 |

Use statuses defined by the application's authentication contract.

---

## 75. API CRUD Test Matrix

| Operation | Positive | Negative |
|---|---|---|
| Create | Valid data | Validation / conflict |
| Read | Existing ID | Missing ID |
| Update | Valid update | Invalid data |
| Delete | Existing record | Missing/unauthorized |

---

## 76. API + UI End-to-End Workflow

Example:

```text
API Login
   ↓
API Create User
   ↓
Browser Open
   ↓
UI Search User
   ↓
UI Verify User
   ↓
UI Update User
   ↓
API GET
   ↓
Verify Backend State
   ↓
API Delete User
```

This is a powerful enterprise pattern.

---

## 77. API Test Reporting

Include useful information:

```text
Endpoint
Method
Expected status
Actual status
Scenario
Correlation/Test ID
```

Avoid reporting secrets.

---

## 78. API Tests and CI

API tests are well suited to CI because they:

- Run quickly
- Require no visual browser interaction
- Are easy to parallelize
- Can validate backend health
- Can create test data for UI suites

Still ensure the environment is stable and isolated.

---

## 79. Environment Management

Typical environments:

```text
LOCAL
DEV
QA
STAGING
```

Use environment variables:

```text
API_BASE_URL
TEST_USERNAME
TEST_PASSWORD
API_KEY
```

Never commit secrets to Git.

---

## 80. API Test Project Structure

Recommended:

```text
playwright/
├── api/
│   ├── auth.api.ts
│   ├── users.api.ts
│   └── reports.api.ts
│
├── fixtures/
│   ├── base.fixture.ts
│   └── api.fixture.ts
│
├── pages/
│   └── UsersPage.ts
│
├── test-data/
│   └── factories/
│       └── user.factory.ts
│
├── tests/
│   ├── api/
│   │   ├── users.api.spec.ts
│   │   └── auth.api.spec.ts
│   │
│   └── integration/
│       └── api-ui.spec.ts
│
└── utils/
    └── api-utils.ts
```

---

## 81. Production-Quality GET Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test(
  'get users',
  async ({ request }) => {

    const response =
      await request.get(
        '/api/users'
      );

    expect(
      response.status()
    ).toBe(200);

    const body =
      await response.json();

    expect(
      body.users
    ).toBeDefined();

    expect(
      Array.isArray(
        body.users
      )
    ).toBe(true);
  }
);
```

---

## 82. Production-Quality POST Example

```ts
test(
  'create user',
  async ({ request }) => {

    const response =
      await request.post(
        '/api/users',
        {
          data: {
            name:
              'Automation User',
            role:
              'QA'
          }
        }
      );

    expect(
      response.status()
    ).toBe(201);

    const body =
      await response.json();

    expect(
      body.id
    ).toBeDefined();

    expect(
      body.name
    ).toBe(
      'Automation User'
    );
  }
);
```

---

## 83. Production-Quality CRUD Example

```ts
test(
  'user CRUD lifecycle',
  async ({ request }) => {

    const create =
      await request.post(
        '/api/users',
        {
          data: {
            name:
              'Automation User',
            role:
              'QA'
          }
        }
      );

    expect(
      create.status()
    ).toBe(201);

    const created =
      await create.json();

    const id =
      created.id;

    try {

      const get =
        await request.get(
          `/api/users/${id}`
        );

      expect(
        get.status()
      ).toBe(200);

      const update =
        await request.patch(
          `/api/users/${id}`,
          {
            data: {
              role:
                'Admin'
            }
          }
        );

      expect(
        update.status()
      ).toBe(200);

      const updated =
        await update.json();

      expect(
        updated.role
      ).toBe('Admin');

    } finally {

      const remove =
        await request.delete(
          `/api/users/${id}`
        );

      expect([
        200,
        204
      ]).toContain(
        remove.status()
      );
    }
  }
);
```

---

## 84. Production-Quality API + UI Example

```ts
test(
  'create user through API and verify UI',
  async ({
    request,
    page
  }) => {

    const create =
      await request.post(
        '/api/users',
        {
          data: {
            name:
              'Automation User',
            role:
              'QA'
          }
        }
      );

    expect(
      create.status()
    ).toBe(201);

    const user =
      await create.json();

    try {

      await page.goto(
        `/users/${user.id}`
      );

      await expect(
        page.getByText(
          'Automation User'
        )
      ).toBeVisible();

    } finally {

      await request.delete(
        `/api/users/${user.id}`
      );
    }
  }
);
```

---

## 85. Common API Automation Mistakes

### Mistake 1

Checking only status code.

### Mistake 2

Ignoring response body.

### Mistake 3

Hard-coding authentication tokens.

### Mistake 4

Sharing test data between parallel tests.

### Mistake 5

Not cleaning up created records.

### Mistake 6

Using UI for all setup operations.

### Mistake 7

Treating mocked APIs as full integration testing.

### Mistake 8

Retrying non-idempotent operations blindly.

### Mistake 9

Logging sensitive request or response data.

### Mistake 10

Hard-coding environment URLs.

---

## 86. Best Practices

1. Use `APIRequestContext` for direct HTTP testing.
2. Validate status codes and response bodies.
3. Validate important headers.
4. Keep authentication reusable.
5. Store secrets in environment variables or CI secret stores.
6. Create test-specific data.
7. Clean up test data.
8. Use API setup for expensive UI preconditions.
9. Keep API clients separate from Page Objects.
10. Combine API and UI testing where useful.
11. Cover negative scenarios.
12. Avoid unnecessary shared state.
13. Keep API contracts and test data current.
14. Make API tests CI-friendly.
15. Do not replace all end-to-end tests with API tests.

---

## 87. Interview Questions

### Q1. What is `APIRequestContext`?

It is Playwright's API client used to send HTTP requests directly without requiring a browser page.

### Q2. How do you send a GET request?

```ts
await request.get(
  '/api/users'
);
```

### Q3. How do you send JSON data?

```ts
await request.post(
  '/api/users',
  {
    data: {
      name: 'Sandeep'
    }
  }
);
```

### Q4. How do you validate the response status?

```ts
expect(
  response.status()
).toBe(200);
```

### Q5. How do you read JSON?

```ts
const body =
  await response.json();
```

### Q6. How do you authenticate an API request?

Use the authentication mechanism required by the API, such as:

```ts
headers: {
  Authorization:
    `Bearer ${token}`
}
```

### Q7. What is the benefit of API + UI testing?

API calls can quickly create or clean up test data while the UI verifies the user-facing behavior.

### Q8. Should API and UI tests be separated?

Usually yes. Keep API clients and Page Objects separate, then combine them in integration scenarios when needed.

### Q9. Why is cleanup important?

Created test records can affect later tests and cause failures, especially in parallel execution.

### Q10. Why should API tests not replace E2E tests?

API tests validate backend behavior, while E2E tests validate the complete user workflow across UI, backend, and integrated services.

---

## 88. Hands-On Exercises

### Exercise 1 — GET

Retrieve users and validate the response.

### Exercise 2 — POST

Create a new user and validate the returned ID.

### Exercise 3 — PUT

Update a user completely.

### Exercise 4 — PATCH

Update only one field.

### Exercise 5 — DELETE

Delete a test user and validate the result.

### Exercise 6 — Authentication

Implement login and use the returned token.

### Exercise 7 — Negative Authentication

Test missing, invalid, and expired authentication.

### Exercise 8 — API Data Factory

Create reusable test-data builders.

### Exercise 9 — API + UI

Create a user through API and verify it through the UI.

### Exercise 10 — Enterprise API Client

Build reusable clients:

```text
AuthApi
UsersApi
ReportsApi
PaymentsApi
```

and integrate them with fixtures.

---

## 89. Enterprise API Automation Challenge

Build a complete API automation framework for:

```text
Authentication
      ↓
Users
      ↓
Roles
      ↓
Reports
      ↓
Expenses
```

Implement:

```text
Authentication
 ├── Login
 ├── Refresh
 ├── Invalid credentials
 └── Expired token

Users
 ├── Create
 ├── Read
 ├── Update
 ├── Delete
 ├── Search
 └── Pagination

Roles
 ├── Access
 ├── Forbidden
 └── Permission validation

Reports
 ├── Generate
 ├── Download
 ├── Empty
 └── Error

Expenses
 ├── Create
 ├── Approve
 ├── Reject
 └── Delete
```

Requirements:

1. Use `APIRequestContext`.
2. Build reusable API client classes.
3. Create authentication fixtures.
4. Use environment-based configuration.
5. Validate status, headers, and body.
6. Validate important response schemas.
7. Cover positive and negative scenarios.
8. Create isolated test data.
9. Clean up generated data.
10. Support parallel execution.
11. Integrate API setup with UI tests.
12. Keep secrets out of source control.
13. Generate useful test reports.
14. Make the framework CI-friendly.
15. Keep API clients separate from Page Objects.

---

## 90. Final Architecture

```text
                         Playwright Test Suite
                                  │
                ┌─────────────────┴─────────────────┐
                │                                   │
           API Automation                      UI Automation
                │                                   │
        ┌───────┴────────┐                    Page Objects
        │                │                         │
    API Clients       Fixtures                     │
        │                │                         │
   REST Endpoints   Authentication                 │
        │                │                         │
        └────────────┬───┴─────────────────────────┘
                     │
              API + UI Integration
                     │
              Test Data Management
                     │
                  Cleanup
```

This architecture provides fast API coverage while allowing UI tests to reuse API-based setup and cleanup.

---

## 91. Final Checklist

Before considering API automation production-ready:

```text
[ ] GET requests covered
[ ] POST requests covered
[ ] PUT/PATCH covered
[ ] DELETE covered
[ ] Status codes validated
[ ] Response bodies validated
[ ] Important headers validated
[ ] Authentication implemented
[ ] Authorization scenarios covered
[ ] Negative cases covered
[ ] Test data isolated
[ ] Cleanup implemented
[ ] API clients reusable
[ ] Fixtures organized
[ ] Secrets protected
[ ] Environment configuration externalized
[ ] Parallel execution supported
[ ] API + UI integration covered
[ ] CI execution supported
[ ] Reporting is useful
```

---

## 92. Next Step

**Step 93 — Mastering Playwright Authentication & Session Management: Storage State, Cookies, Local Storage, Session Storage, JWT, OAuth2, Multi-Role Login & Enterprise Auth Fixtures**

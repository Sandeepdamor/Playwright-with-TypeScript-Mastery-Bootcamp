# Playwright with TypeScript Mastery Bootcamp

# Step 102 — Mastering Playwright API Testing & Network Automation: APIRequestContext, REST Methods, Authentication, Headers, Query Parameters, Request Bodies, Response Validation, API Fixtures, CRUD Workflows, API/UI Integration & Enterprise API Test Architecture

## 1. Introduction

Modern web applications depend heavily on APIs.

A typical application works like:

```text
Browser
   ↓
Frontend
   ↓
REST / GraphQL / Backend APIs
   ↓
Business Logic
   ↓
Database
```

Playwright can test the UI and also interact directly with APIs.

API testing helps you validate:

```text
HTTP Methods
Authentication
Authorization
Headers
Query Parameters
Path Parameters
Request Bodies
Status Codes
Response Bodies
CRUD Operations
Error Handling
API Contracts
Data Setup
Data Cleanup
UI + API Integration
```

The goal is to build tests that are:

```text
Fast
Reliable
Focused
Maintainable
Parallel-safe
```

---

## 2. Why API Testing Matters

UI tests are valuable, but UI-only testing can be:

```text
Slow
Expensive
Difficult to debug
Dependent on frontend implementation
```

API tests can validate backend behavior directly.

Example:

```text
POST /api/users
      ↓
201 Created
      ↓
GET /api/users/{id}
      ↓
User Exists
```

---

## 3. API Testing vs UI Testing

### UI Test

```text
Open Browser
 ↓
Navigate
 ↓
Fill Form
 ↓
Click Button
 ↓
Wait
 ↓
Verify UI
```

### API Test

```text
Send Request
 ↓
Receive Response
 ↓
Validate Status
 ↓
Validate Body
```

API tests are usually faster for backend behavior.

---

## 4. Playwright APIRequestContext

Playwright provides:

```ts
APIRequestContext
```

for making HTTP requests.

You can obtain it from the built-in `request` fixture:

```ts
import {
  test,
  expect
} from '@playwright/test';

test(
  'get users',
  async ({
    request
  }) => {

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

## 5. APIRequestContext Advantages

It supports:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

and can handle:

```text
Headers
Authentication
Query parameters
Request bodies
Cookies
Responses
```

---

## 6. GET Request

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

---

## 7. POST Request

Example:

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {
        username:
          'automation-user',
        role:
          'Employee'
      }
    }
  );

expect(
  response.status()
).toBe(201);
```

---

## 8. PUT Request

PUT generally represents replacement/update behavior.

Example:

```ts
const response =
  await request.put(
    `/api/users/${userId}`,
    {
      data: {
        username:
          'updated-user',
        role:
          'Manager'
      }
    }
  );

expect(
  response.status()
).toBe(200);
```

The exact API semantics depend on the backend contract.

---

## 9. PATCH Request

PATCH generally represents a partial update.

Example:

```ts
const response =
  await request.patch(
    `/api/users/${userId}`,
    {
      data: {
        role:
          'Manager'
      }
    }
  );

expect(
  response.status()
).toBe(200);
```

---

## 10. DELETE Request

Example:

```ts
const response =
  await request.delete(
    `/api/users/${userId}`
  );

expect(
  response.status()
).toBe(204);
```

The API may return another documented success status.

---

## 11. Response Status

Validate the expected HTTP status.

Common statuses:

```text
200 OK
201 Created
202 Accepted
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content
429 Too Many Requests
500 Internal Server Error
```

Always use the application's actual API contract.

---

## 12. `response.ok()`

Example:

```ts
expect(
  response.ok()
).toBe(true);
```

This is useful for broad success validation.

For important scenarios, prefer asserting the exact expected status too.

---

## 13. Response Body

Read JSON:

```ts
const body =
  await response.json();

expect(
  body.username
).toBe(
  'automation-user'
);
```

---

## 14. Response Text

For non-JSON responses:

```ts
const text =
  await response.text();

expect(
  text
).toContain(
  'success'
);
```

Use the representation expected by the API contract.

---

## 15. Response Headers

Example:

```ts
const headers =
  response.headers();

expect(
  headers[
    'content-type'
  ]
).toContain(
  'application/json'
);
```

Validate headers that are important to the API contract.

---

## 16. Query Parameters

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        page:
          1,
        size:
          10
      }
    }
  );
```

This is preferable to manually concatenating query strings.

---

## 17. Multiple Query Parameters

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        page:
          1,
        size:
          20,
        status:
          'ACTIVE',
        role:
          'EMPLOYEE'
      }
    }
  );
```

---

## 18. Query Parameter Validation

Test:

```text
Pagination
Filtering
Sorting
Searching
Date ranges
Status
Role
```

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        status:
          'ACTIVE'
      }
    }
  );

expect(
  response.status()
).toBe(200);
```

Then validate returned data.

---

## 19. Path Parameters

Example:

```ts
const response =
  await request.get(
    `/api/users/${userId}`
  );
```

Use path parameters for resource identification.

---

## 20. Request Body

JSON request:

```ts
await request.post(
  '/api/users',
  {
    data: {
      username:
        'test-user',
      email:
        'test@example.test'
    }
  }
);
```

Playwright serializes the object appropriately for a JSON request.

---

## 21. Request Headers

Example:

```ts
await request.get(
  '/api/users',
  {
    headers: {
      'Accept':
        'application/json'
    }
  }
);
```

Headers can be configured per request or through a reusable request context.

---

## 22. Authorization Header

Token-based APIs may use:

```ts
headers: {
  Authorization:
    `Bearer ${token}`
}
```

Never log the token.

Use secure environment or fixture-based authentication.

---

## 23. API Authentication

Common approaches:

```text
Bearer Token
Cookie Session
Basic Authentication
OAuth
OIDC
API Key
```

The correct method depends on the API.

---

## 24. APIRequestContext with Base URL

Playwright configuration can define:

```ts
use: {
  baseURL:
    'https://example.test'
}
```

Then:

```ts
await request.get(
  '/api/users'
);
```

can use the configured base URL.

---

## 25. API Context Options

A reusable request context can define:

```text
Base URL
Extra Headers
Storage State
Ignore HTTPS Errors
Proxy
```

Keep configuration centralized where practical.

---

## 26. API Fixtures

The built-in:

```ts
request
```

fixture is useful for simple API tests.

For enterprise frameworks, create reusable API fixtures or clients.

---

## 27. API Client Pattern

Example:

```ts
class UserApi {

  constructor(
    private readonly request:
      APIRequestContext
  ) {}

  async getUser(
    id: number
  ) {
    return this.request.get(
      `/api/users/${id}`
    );
  }

  async createUser(
    data: UserData
  ) {
    return this.request.post(
      '/api/users',
      {
        data
      }
    );
  }
}
```

This hides endpoint details from tests.

---

## 28. Why Use API Clients?

Without an API client:

```text
Tests
 ↓
Raw URLs
 ↓
Headers
 ↓
Request Details
```

With a client:

```text
Tests
 ↓
UserApi
 ↓
HTTP API
```

This improves readability and maintainability.

---

## 29. API Client Responsibilities

An API client should generally handle:

```text
Endpoint construction
Request method
Request body
Headers
Authentication configuration
Response return
```

Business assertions can remain in tests or assertion helpers.

---

## 30. Avoid Over-Abstraction

Do not create a giant API class containing every endpoint.

Prefer domain-focused clients:

```text
UserApi
ExpenseApi
ReportApi
AuthApi
CompanyApi
```

---

## 31. CRUD

CRUD means:

```text
Create
Read
Update
Delete
```

A complete API workflow:

```text
POST
 ↓
GET
 ↓
PUT/PATCH
 ↓
DELETE
```

---

## 32. Create User

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: user
    }
  );

expect(
  response.status()
).toBe(201);

const created =
  await response.json();

expect(
  created.id
).toBeDefined();
```

---

## 33. Read User

```ts
const response =
  await request.get(
    `/api/users/${created.id}`
  );

expect(
  response.status()
).toBe(200);
```

---

## 34. Update User

```ts
const response =
  await request.patch(
    `/api/users/${created.id}`,
    {
      data: {
        role:
          'Manager'
      }
    }
  );

expect(
  response.status()
).toBe(200);
```

---

## 35. Delete User

```ts
const response =
  await request.delete(
    `/api/users/${created.id}`
  );

expect(
  response.status()
).toBe(204);
```

---

## 36. Verify Deletion

```ts
const response =
  await request.get(
    `/api/users/${created.id}`
  );

expect(
  response.status()
).toBe(404);
```

The expected behavior depends on the application's deletion model.

---

## 37. Soft Delete

Some enterprise applications use soft delete.

Example:

```text
Record
 ↓
isDeleted = true
```

The API may still return the record depending on the endpoint.

Test the documented behavior.

---

## 38. API Test Data

API tests should use controlled test data.

Combine with factories:

```ts
const user =
  createUser();

const response =
  await userApi.createUser(
    user
  );
```

This connects Step 100's data-management strategy with API testing.

---

## 39. API Seeding

API requests are excellent for preparing UI tests.

Example:

```text
API
 ↓
Create Expense
 ↓
Open Browser
 ↓
Navigate to Expense
 ↓
Test UI
```

This is usually much faster than creating the expense through many UI actions.

---

## 40. API + UI Integration

A powerful pattern:

```text
API Setup
   ↓
UI Action
   ↓
API Verification
```

Example:

```text
Create User through API
 ↓
Search User in UI
 ↓
Edit User in UI
 ↓
Verify update through API
```

This validates the complete integration.

---

## 41. UI + API Test Example

```ts
const user =
  createUser();

await userApi.createUser(
  user
);

await page.goto(
  '/users'
);

await expect(
  page.getByText(
    user.username
  )
).toBeVisible();
```

The API performs setup while the UI remains the system under test.

---

## 42. API Verification After UI Action

Example:

```text
UI
 ↓
Click Approve
 ↓
API
 ↓
Verify status = APPROVED
```

This can provide stronger evidence of backend state.

---

## 43. API Authentication State

When API and browser authentication share the same mechanism, authentication state can sometimes be reused.

However, do not assume that browser cookies and API authentication are interchangeable.

Validate the application's actual authentication architecture.

---

## 44. Request Context with Storage State

Conceptually:

```ts
const context =
  await request.newContext({
    storageState:
      '.auth/user.json'
  });
```

This can be useful when the application's authentication uses compatible cookie-based state.

---

## 45. Separate API Authentication

For token-based APIs:

```ts
const context =
  await request.newContext({
    extraHTTPHeaders: {
      Authorization:
        `Bearer ${token}`
    }
  });
```

Keep token retrieval and secret handling secure.

---

## 46. API Fixtures

A custom fixture can provide a domain API client:

```ts
type Fixtures = {
  userApi:
    UserApi;
};

export const test =
  base.extend<Fixtures>({
    userApi:
      async ({
        request
      }, use) => {

        await use(
          new UserApi(
            request
          )
        );
      }
  });
```

Tests become cleaner:

```ts
const response =
  await userApi.getUser(
    userId
  );
```

---

## 47. API Response Assertions

Validate:

```text
Status
Headers
Body
Business fields
Error contract
```

Example:

```ts
expect(
  response.status()
).toBe(200);

const body =
  await response.json();

expect(
  body.status
).toBe(
  'ACTIVE'
);
```

---

## 48. Flexible Response Validation

For responses containing dynamic fields:

```ts
expect(
  body
).toMatchObject({
  status:
    'success',
  data: {
    id:
      expect.any(Number)
  }
});
```

Avoid exact equality when dynamic fields are expected.

---

## 49. API Error Testing

Negative API tests are essential.

Examples:

```text
Missing required field
Invalid ID
Unauthorized request
Forbidden request
Duplicate record
Invalid state transition
```

---

## 50. 400 Bad Request

Example:

```ts
const response =
  await request.post(
    '/api/users',
    {
      data: {}
    }
  );

expect(
  response.status()
).toBe(400);
```

Then validate the error contract.

---

## 51. 401 Unauthorized

Typical scenario:

```text
No credentials
 ↓
Protected API
 ↓
401
```

Example:

```ts
const response =
  await request.get(
    '/api/profile'
  );

expect(
  response.status()
).toBe(401);
```

---

## 52. 403 Forbidden

Typical scenario:

```text
Authenticated Employee
 ↓
Admin API
 ↓
403
```

This tests authorization rather than authentication.

---

## 53. 404 Not Found

Example:

```ts
const response =
  await request.get(
    '/api/users/999999999'
  );

expect(
  response.status()
).toBe(404);
```

Use an ID that is guaranteed not to exist in the test environment.

---

## 54. 409 Conflict

Common cases:

```text
Duplicate email
Duplicate username
Concurrent update
Invalid state transition
```

Example:

```ts
expect(
  response.status()
).toBe(409);
```

---

## 55. Error Body Validation

Example:

```ts
const body =
  await response.json();

expect(
  body
).toMatchObject({
  message:
    expect.any(String)
});
```

Validate the fields required by the API contract.

---

## 56. API Contract Testing

Contract testing verifies that API behavior matches expectations.

Validate:

```text
Status
Required fields
Data types
Field names
Error structure
```

For complex APIs, dedicated schema validation tools may complement Playwright.

---

## 57. JSON Schema-Like Validation

A response may need:

```text
id → number
name → string
active → boolean
roles → array
```

You can combine Playwright with a schema-validation library where appropriate.

---

## 58. API Pagination

Test:

```text
page
size
total
items
```

Example:

```ts
const body =
  await response.json();

expect(
  body.page
).toBe(1);

expect(
  body.items
).toBeDefined();
```

---

## 59. Pagination Boundary Testing

Test:

```text
First page
Middle page
Last page
Empty page
Page beyond range
```

Validate the documented behavior.

---

## 60. API Filtering

Example:

```ts
const response =
  await request.get(
    '/api/users',
    {
      params: {
        status:
          'ACTIVE'
      }
    }
  );

const body =
  await response.json();

for (
  const user
  of body.users
) {
  expect(
    user.status
  ).toBe(
    'ACTIVE'
  );
}
```

---

## 61. API Sorting

Validate:

```text
Ascending
Descending
Multiple fields
Default sorting
```

Do not assume ordering unless it is part of the API contract.

---

## 62. API Search

Test:

```text
Exact search
Partial search
Case behavior
No results
Special characters
```

---

## 63. API Date Filtering

Example:

```ts
await request.get(
  '/api/expenses',
  {
    params: {
      from:
        '2026-01-01',
      to:
        '2026-01-31'
    }
  }
);
```

Control timezone and date representation carefully.

---

## 64. API File Upload

For multipart uploads, use Playwright's supported request options for form data/files.

Conceptually:

```text
POST
 ↓
multipart/form-data
 ↓
File
```

Validate:

```text
Status
File metadata
Processing result
```

---

## 65. API File Download

Validate:

```text
Status
Content-Type
Content-Disposition
File content
```

For generated reports, combine API and file validation.

---

## 66. API Request Timeout

Network requests can fail because of:

```text
Server latency
Network issues
Environment overload
Dependency outage
```

Use realistic request timeouts.

Do not use extremely large timeouts to hide unstable APIs.

---

## 67. API Retries

A retry strategy should be intentional.

Distinguish:

```text
Transient infrastructure failure
```

from:

```text
Deterministic application failure
```

Do not automatically retry every failed request.

---

## 68. Idempotency

Some HTTP operations can be repeated safely; others may create duplicate resources.

For example:

```text
GET → generally safe to repeat
DELETE → semantics depend on API
POST → may create duplicates
```

Understand the API's idempotency contract before adding retries.

---

## 69. Idempotency Keys

Payment or order APIs may support:

```text
Idempotency-Key
```

Example:

```ts
headers: {
  'Idempotency-Key':
    uniqueRequestId
}
```

Test this behavior when required.

---

## 70. API Concurrency

Some tests need:

```text
Request A
Request B
```

at nearly the same time.

Use controlled parallel requests to test:

```text
Race conditions
Duplicate creation
Optimistic locking
Conflict responses
```

Only introduce concurrency when it represents a real requirement.

---

## 71. API Dependency Chains

Example:

```text
Create Company
 ↓
Create Department
 ↓
Create User
 ↓
Create Expense
 ↓
Approve Expense
```

Store returned IDs rather than hard-coding them.

---

## 72. API Data Relationships

Example:

```ts
const company =
  await companyApi.create(
    companyData
  );

const user =
  await userApi.create({
    ...userData,
    companyId:
      company.id
  });

const expense =
  await expenseApi.create({
    ...expenseData,
    userId:
      user.id
  });
```

This creates realistic business relationships.

---

## 73. API Cleanup

After API-created test data:

```text
Test
 ↓
Cleanup
```

Use:

```text
DELETE endpoint
Database cleanup
Fixture teardown
```

depending on the architecture.

---

## 74. Cleanup Order

For dependent data:

```text
Expense
 ↓
User
 ↓
Department
 ↓
Company
```

Delete children before parents when foreign-key constraints require it.

---

## 75. API Test Isolation

Each test should ideally own:

```text
Created records
Authentication state
Temporary resources
```

Avoid shared mutable API data.

---

## 76. Parallel API Tests

Parallel API tests should use unique data.

Example:

```text
Worker 0
 → user-a

Worker 1
 → user-b
```

Factories and UUIDs are useful here.

---

## 77. API Test Logging

Log useful information:

```text
HTTP method
Endpoint
Status
Test identifier
Resource ID
```

Avoid logging:

```text
Authorization headers
Passwords
Tokens
Sensitive request bodies
Personal data
```

---

## 78. API Tracing and Diagnostics

When debugging API failures, capture:

```text
Method
URL
Status
Response headers
Safe response body
Correlation ID
```

Do not expose secrets.

---

## 79. Correlation IDs

Enterprise systems may use:

```text
X-Correlation-ID
X-Request-ID
```

These can help trace a failed test through backend logs.

Example:

```ts
headers: {
  'X-Test-ID':
    testId
}
```

Use a safe synthetic identifier.

---

## 80. API/UI Correlation

A useful debugging flow:

```text
Playwright Test
 ↓
API Request
 ↓
Correlation ID
 ↓
Backend Logs
 ↓
Failure Diagnosis
```

This is especially useful in distributed systems.

---

## 81. API Testing in Microservices

For microservice applications:

```text
Gateway
 ↓
Service A
 ↓
Service B
 ↓
Database
```

API tests can validate:

```text
Gateway routing
Authentication
Service response
Error propagation
Contract behavior
```

---

## 82. API Gateway Testing

Validate:

```text
Correct route
Authentication
Authorization
Status propagation
Headers
Error mapping
```

Example:

```text
GET /api/users
 ↓
Gateway
 ↓
User Service
```

---

## 83. Service-to-Service Behavior

Do not turn every Playwright test into a full microservice integration test.

Use Playwright primarily for externally observable API behavior.

Use lower-level tests for internal service contracts where appropriate.

---

## 84. API Test Layers

A mature architecture can use:

```text
API Unit Tests
 ↓
Service Integration Tests
 ↓
API Contract Tests
 ↓
Playwright API Tests
 ↓
UI + API End-to-End Tests
```

Each layer has a different purpose.

---

## 85. API Test Scope

Playwright API tests are ideal for:

```text
Critical REST workflows
Authentication
Authorization
Business endpoints
Integration behavior
UI setup
UI verification
```

---

## 86. Avoid Testing Everything Through Playwright

If 1,000 backend rules can be tested at a lower level, do not recreate all of them as slow end-to-end tests.

Use the right testing layer.

---

## 87. API Client Folder Structure

A scalable project can use:

```text
utils/
├── api/
│   ├── authApi.ts
│   ├── userApi.ts
│   ├── expenseApi.ts
│   ├── reportApi.ts
│   └── companyApi.ts
```

---

## 88. API Models

Use TypeScript types:

```ts
export interface UserData {
  username:
    string;

  email:
    string;

  role:
    string;
}
```

Response models can be defined separately.

---

## 89. Request vs Response Models

Keep these distinct when their shapes differ:

```text
CreateUserRequest
UserResponse
UpdateUserRequest
```

Example:

```ts
interface CreateUserRequest {
  username:
    string;
  email:
    string;
}

interface UserResponse {
  id:
    number;
  username:
    string;
  email:
    string;
}
```

---

## 90. API Service Example

```ts
class UserApi {

  constructor(
    private readonly request:
      APIRequestContext
  ) {}

  async create(
    data:
      CreateUserRequest
  ) {
    return this.request.post(
      '/api/users',
      {
        data
      }
    );
  }

  async get(
    id: number
  ) {
    return this.request.get(
      `/api/users/${id}`
    );
  }

  async update(
    id: number,
    data:
      UpdateUserRequest
  ) {
    return this.request.patch(
      `/api/users/${id}`,
      {
        data
      }
    );
  }

  async delete(
    id: number
  ) {
    return this.request.delete(
      `/api/users/${id}`
    );
  }
}
```

---

## 91. API Assertions Layer

For reusable business validation:

```ts
async function
expectSuccessfulUserCreation(
  response:
    APIResponse
) {

  expect(
    response.status()
  ).toBe(201);

  const body =
    await response.json();

  expect(
    body.id
  ).toBeDefined();
}
```

Keep helpers focused.

---

## 92. API Test Example

```ts
test(
  'create user through API',
  async ({
    request
  }) => {

    const user =
      createUser();

    const response =
      await request.post(
        '/api/users',
        {
          data: user
        }
      );

    expect(
      response.status()
    ).toBe(201);

    const body =
      await response.json();

    expect(
      body.username
    ).toBe(
      user.username
    );
  }
);
```

---

## 93. API + UI End-to-End Example

```ts
test(
  'user created through API appears in UI',
  async ({
    request,
    page
  }) => {

    const user =
      createUser();

    const response =
      await request.post(
        '/api/users',
        {
          data: user
        }
      );

    expect(
      response.status()
    ).toBe(201);

    await page.goto(
      '/users'
    );

    await expect(
      page.getByText(
        user.username
      )
    ).toBeVisible();
  }
);
```

---

## 94. UI Action + API Verification

```text
Open Expense
 ↓
Click Approve
 ↓
GET /api/expenses/{id}
 ↓
Verify APPROVED
```

This is useful when the UI does not expose enough information to confidently verify backend state.

---

## 95. API Contract and Business Assertion

Do both:

```text
HTTP Contract
+
Business Contract
```

Example:

```ts
expect(
  response.status()
).toBe(200);

expect(
  body.status
).toBe(
  'APPROVED'
);
```

---

## 96. API Security Testing

Important API security scenarios:

```text
No authentication
Invalid authentication
Expired authentication
Insufficient permission
Cross-user access
Cross-tenant access
Invalid object ID
```

Use controlled synthetic environments.

---

## 97. API Authorization Matrix

Example:

| Endpoint | Admin | Manager | Employee |
|---|---:|---:|---:|
| GET /users | ✓ | Limited | ✗ |
| POST /users | ✓ | ✗ | ✗ |
| DELETE /users | ✓ | ✗ | ✗ |
| POST /expenses | ✓ | ✓ | ✓ |
| POST /expenses/{id}/approve | ✓ | ✓ | ✗ |

This matrix should match the actual security policy.

---

## 98. API/UI Authentication Architecture

A strong framework can look like:

```text
                 Playwright
                     │
           ┌─────────┴─────────┐
           │                   │
          UI                  API
           │                   │
      Browser Context    APIRequestContext
           │                   │
           └─────────┬─────────┘
                     │
              Authentication
                     │
               Test Data Layer
                     │
                 Backend
```

---

## 99. Enterprise API Architecture

```text
                         Test
                          │
              ┌───────────┴───────────┐
              │                       │
             UI                      API
              │                       │
       Page Objects              API Clients
              │                       │
              └───────────┬───────────┘
                          │
                    Test Data Layer
                          │
              ┌───────────┼───────────┐
              │           │           │
           Factory      Seeder      Cleanup
              │           │           │
              └───────────┼───────────┘
                          │
                     Application
                          │
                     REST APIs
                          │
                       Database
```

---

## 100. API Testing Strategy

Use:

```text
API Tests
 ↓
Validate backend contracts

UI Tests
 ↓
Validate user journeys

UI + API Tests
 ↓
Validate integration

API Setup
 ↓
Create fast preconditions

API Verification
 ↓
Confirm backend state
```

This gives good coverage without making every test slow.

---

## 101. Common API Testing Mistakes

### Mistake 1

Testing only status code.

### Mistake 2

Ignoring response body.

### Mistake 3

Hard-coding IDs.

### Mistake 4

Sharing mutable records.

### Mistake 5

Logging authentication tokens.

### Mistake 6

Creating all setup data through UI.

### Mistake 7

Ignoring authorization.

### Mistake 8

Not cleaning created records.

### Mistake 9

Retrying every failure.

### Mistake 10

Using Playwright for every backend rule.

---

## 102. Enterprise Best Practices

1. Use `APIRequestContext` for API workflows.
2. Validate exact status codes where meaningful.
3. Validate response bodies.
4. Validate important headers.
5. Use query parameters rather than manual URL construction.
6. Use typed request and response models.
7. Create domain-specific API clients.
8. Use factories for test data.
9. Use APIs for fast test setup.
10. Keep UI setup focused on UI behavior.
11. Test positive and negative scenarios.
12. Test authentication and authorization.
13. Design for parallel execution.
14. Use unique test data.
15. Clean up API-created data.
16. Protect credentials and tokens.
17. Use correlation IDs for diagnostics.
18. Keep Playwright API tests focused on externally observable behavior.
19. Avoid excessive abstraction.
20. Document API test architecture.

---

## 103. Hands-On Exercise 1 — GET

Create a test that:

```text
GET /api/users
 ↓
Verify 200
 ↓
Verify response body
```

---

## 104. Hands-On Exercise 2 — POST

Create a synthetic user:

```text
POST /api/users
 ↓
Verify 201
 ↓
Capture ID
```

---

## 105. Hands-On Exercise 3 — PUT/PATCH

Update the user:

```text
PATCH /api/users/{id}
 ↓
Verify 200
 ↓
Verify updated field
```

---

## 106. Hands-On Exercise 4 — DELETE

Delete the user:

```text
DELETE /api/users/{id}
 ↓
Verify success
 ↓
GET user
 ↓
Verify expected deleted state
```

---

## 107. Hands-On Exercise 5 — API Client

Create:

```text
UserApi
ExpenseApi
CompanyApi
```

with reusable methods.

---

## 108. Hands-On Exercise 6 — API Fixture

Create a fixture:

```text
userApi
```

and inject it into tests.

---

## 109. Hands-On Exercise 7 — API Seeding

Create an expense through API, then open the UI and verify it appears.

---

## 110. Hands-On Exercise 8 — API Verification

Approve an expense through UI, then verify:

```text
GET /api/expenses/{id}
```

returns:

```text
APPROVED
```

---

## 111. Hands-On Exercise 9 — Authorization

Test:

```text
Admin → Allowed
Manager → Allowed / Denied
Employee → Denied
```

for a sensitive endpoint.

---

## 112. Hands-On Exercise 10 — Error Contract

Create tests for:

```text
400
401
403
404
409
```

and validate the documented error body.

---

## 113. Enterprise API Challenge

Build an API automation layer supporting:

```text
Authentication
GET
POST
PUT
PATCH
DELETE
Headers
Query Parameters
Path Parameters
Request Bodies
Response Validation
API Clients
API Fixtures
CRUD
API Seeding
API Cleanup
Authorization
UI + API Integration
```

Requirements:

1. Create typed request models.
2. Create typed response models.
3. Create API clients.
4. Implement CRUD methods.
5. Implement authentication.
6. Implement role-specific API access.
7. Create API fixtures.
8. Create synthetic test data.
9. Seed UI preconditions through API.
10. Verify UI actions through API.
11. Validate error contracts.
12. Validate authorization.
13. Support parallel execution.
14. Implement cleanup.
15. Add safe diagnostics.
16. Protect secrets.
17. Add correlation IDs where useful.
18. Keep API tests independent.
19. Avoid unnecessary fixed waits.
20. Document the API architecture.

---

## 114. Final Checklist

```text
[ ] APIRequestContext understood
[ ] GET understood
[ ] POST understood
[ ] PUT understood
[ ] PATCH understood
[ ] DELETE understood
[ ] Status codes understood
[ ] Response body understood
[ ] Response headers understood
[ ] Query parameters understood
[ ] Path parameters understood
[ ] Request body understood
[ ] Authentication understood
[ ] Authorization understood
[ ] API fixtures understood
[ ] API client pattern understood
[ ] CRUD workflow understood
[ ] API seeding understood
[ ] API cleanup understood
[ ] API data relationships understood
[ ] Pagination understood
[ ] Filtering understood
[ ] Sorting understood
[ ] Search understood
[ ] Date filtering understood
[ ] Error testing understood
[ ] API contract testing understood
[ ] Negative testing understood
[ ] API security testing understood
[ ] UI + API integration understood
[ ] API verification understood
[ ] Parallel API testing understood
[ ] Correlation IDs understood
[ ] Microservice API testing understood
[ ] API architecture understood
```

---

## 115. Key Takeaways

- Playwright can test both browser behavior and APIs.
- `APIRequestContext` provides a powerful way to perform HTTP requests.
- API tests are usually faster and more focused than equivalent UI setup.
- Validate status codes, response bodies, headers, and business behavior.
- Use API clients to hide repetitive endpoint details.
- Use TypeScript models for safer request and response handling.
- API seeding is excellent for creating UI preconditions.
- API verification can confirm backend state after UI actions.
- Authentication and authorization must both be tested.
- Use separate contexts or credentials when different roles are required.
- Generate unique test data for parallel execution.
- Clean up records created by API tests.
- Never log passwords, tokens, or sensitive authentication data.
- Use correlation IDs for enterprise diagnostics when supported.
- Do not use Playwright API tests for every backend rule; choose the appropriate testing layer.
- A strong API architecture makes the overall Playwright framework faster, more reliable, and easier to maintain.

---

## 116. Next Step

**Step 103 — Mastering Playwright Network Interception & Mocking: page.route(), route.fulfill(), route.continue(), route.abort(), Request/Response Inspection, API Mocking, Dependency Isolation, HAR, WebSockets, Network Conditions & Enterprise Network Testing**

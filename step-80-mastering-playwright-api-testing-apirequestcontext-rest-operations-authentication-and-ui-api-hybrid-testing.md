# Playwright with TypeScript Mastery Bootcamp

# Step 80 — Mastering Playwright API Testing, APIRequestContext, REST Operations, Authentication & UI/API Hybrid Testing

## 1. Introduction

Playwright is not limited to browser UI automation.

It also provides powerful API testing capabilities through:

```ts
request
APIRequestContext
```

API testing allows automation engineers to validate backend services directly without opening a browser.

A mature enterprise automation framework can combine:

```text
UI Testing
+
API Testing
+
Network Interception
+
Database Validation
+
End-to-End Testing
```

This creates faster, more reliable, and more comprehensive test coverage.

---

## 2. What Is API Testing?

API testing validates communication between software components.

A typical REST API flow is:

```text
Client
   ↓
HTTP Request
   ↓
REST API
   ↓
Business Logic
   ↓
Database
   ↓
HTTP Response
   ↓
Client
```

API tests validate:

- HTTP methods
- Status codes
- Headers
- Request bodies
- Response bodies
- Authentication
- Authorization
- Error handling
- Business rules

---

## 3. Why Use Playwright for API Testing?

Playwright API testing provides several advantages:

- Same TypeScript ecosystem as UI tests
- Easy UI + API integration
- Shared configuration
- Reusable authentication
- Fast execution
- Easy test-data setup
- Easy cleanup
- Strong assertion support

Instead of opening a browser for every setup operation, an API can create test data directly.

---

## 4. APIRequestContext

The central API testing object is:

```ts
APIRequestContext
```

It allows tests to send HTTP requests.

Typical operations include:

```ts
GET
POST
PUT
PATCH
DELETE
HEAD
```

---

## 5. Creating an API Request Context

You can use the built-in `request` fixture:

```ts
import { test } from '@playwright/test';

test('GET users API', async ({ request }) => {
  const response = await request.get('/api/users');

  console.log(response.status());
});
```

This is one of the simplest ways to perform API testing.

---

## 6. GET Request

Example:

```ts
test('get users', async ({ request }) => {
  const response = await request.get('/api/users');

  expect(response.ok()).toBeTruthy();
  expect(response.status()).toBe(200);
});
```

---

## 7. Reading Response JSON

Use:

```ts
const response = await request.get('/api/users');

const data = await response.json();

console.log(data);
```

Example assertion:

```ts
expect(data.users.length).toBeGreaterThan(0);
```

---

## 8. Reading Response Text

Use:

```ts
const response = await request.get('/api/message');

const body = await response.text();

console.log(body);
```

This is useful when the API returns plain text instead of JSON.

---

## 9. Inspecting Response Headers

```ts
const response = await request.get('/api/users');

const headers = response.headers();

console.log(headers);
```

Specific header:

```ts
const contentType =
  response.headers()['content-type'];

expect(contentType).toContain(
  'application/json'
);
```

---

## 10. POST Request

A POST request can include JSON data:

```ts
const response = await request.post('/api/users', {
  data: {
    name: 'Sandeep',
    role: 'QA Engineer'
  }
});
```

Then:

```ts
expect(response.status()).toBe(201);
```

---

## 11. POST Request with Assertions

```ts
test('create user', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: {
      name: 'Sandeep',
      role: 'QA Engineer'
    }
  });

  expect(response.status()).toBe(201);

  const data = await response.json();

  expect(data.name).toBe('Sandeep');
});
```

---

## 12. PUT Request

PUT generally replaces or updates a resource.

```ts
const response = await request.put('/api/users/10', {
  data: {
    name: 'Updated User',
    role: 'Senior QA'
  }
});

expect(response.ok()).toBeTruthy();
```

---

## 13. PATCH Request

PATCH is commonly used for partial updates.

```ts
const response = await request.patch('/api/users/10', {
  data: {
    role: 'Senior QA'
  }
});

expect(response.status()).toBe(200);
```

---

## 14. DELETE Request

```ts
const response = await request.delete('/api/users/10');

expect(response.status()).toBe(204);
```

The expected status depends on the API contract.

---

## 15. HEAD Request

A HEAD request retrieves response headers without the normal response body.

```ts
const response = await request.head('/api/users');

expect(response.ok()).toBeTruthy();
```

---

## 16. Query Parameters

You can send query parameters using `params`:

```ts
const response = await request.get('/api/users', {
  params: {
    page: 1,
    size: 10,
    role: 'admin'
  }
});
```

This results conceptually in:

```text
/api/users?page=1&size=10&role=admin
```

---

## 17. Request Headers

Example:

```ts
const response = await request.get('/api/users', {
  headers: {
    'Accept': 'application/json'
  }
});
```

Headers can be used for:

- Content negotiation
- Correlation IDs
- Tenant identification
- Feature flags
- Authentication

---

## 18. Authorization Header

Example:

```ts
const response = await request.get('/api/profile', {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

Do not hard-code real credentials in source code.

Use environment variables or secure authentication fixtures.

---

## 19. Content Type

For JSON requests:

```ts
const response = await request.post('/api/users', {
  headers: {
    'Content-Type': 'application/json'
  },
  data: {
    name: 'Sandeep'
  }
});
```

Playwright can handle JSON request data conveniently through the `data` option.

---

## 20. Form Data

Some APIs accept form data.

Example:

```ts
const response = await request.post('/api/login', {
  form: {
    username: 'admin',
    password: 'secret'
  }
});
```

The exact format depends on the backend contract.

---

## 21. URL-Encoded Form Data

```ts
const response = await request.post('/api/login', {
  form: {
    username: 'admin',
    password: 'secret'
  }
});
```

This is useful for APIs expecting:

```text
application/x-www-form-urlencoded
```

---

## 22. Multipart File Upload

Playwright can also send multipart requests.

Example:

```ts
const response = await request.post('/api/upload', {
  multipart: {
    file: {
      name: 'sample.txt',
      mimeType: 'text/plain',
      buffer: Buffer.from('Hello Playwright')
    }
  }
});
```

This is useful for API-level upload testing.

---

## 23. Status Code Assertions

Always validate the expected HTTP status.

```ts
expect(response.status()).toBe(200);
```

Useful status codes include:

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
422 Unprocessable Entity
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

The correct expectation should come from the API contract.

---

## 24. Using `response.ok()`

```ts
expect(response.ok()).toBeTruthy();
```

`ok()` indicates whether the HTTP response was successful according to Playwright's response handling.

For precise contract testing, prefer checking the exact expected status.

---

## 25. Validating Response JSON

Suppose the response is:

```json
{
  "id": 10,
  "name": "Sandeep",
  "role": "QA Engineer"
}
```

Test:

```ts
const data = await response.json();

expect(data.id).toBe(10);
expect(data.name).toBe('Sandeep');
expect(data.role).toBe('QA Engineer');
```

---

## 26. Validating Nested JSON

Example:

```json
{
  "user": {
    "id": 10,
    "profile": {
      "name": "Sandeep"
    }
  }
}
```

Test:

```ts
expect(data.user.id).toBe(10);
expect(data.user.profile.name).toBe('Sandeep');
```

---

## 27. Schema-Like Validation

For large APIs, simple assertions may not be enough.

You can validate important fields:

```ts
expect(data).toHaveProperty('id');
expect(data).toHaveProperty('name');
expect(data).toHaveProperty('role');
```

For strict contract validation, consider integrating a JSON Schema or runtime validation library into the framework.

---

## 28. API Authentication

Common authentication mechanisms include:

- Bearer tokens
- Basic authentication
- API keys
- OAuth 2.0
- JWT
- Session cookies

Example:

```ts
const response = await request.get('/api/profile', {
  headers: {
    Authorization: `Bearer ${accessToken}`
  }
});
```

---

## 29. Basic Authentication

Example:

```ts
const response = await request.get('/api/profile', {
  headers: {
    Authorization:
      'Basic ' +
      Buffer
        .from('username:password')
        .toString('base64')
  }
});
```

Never hard-code production credentials.

---

## 30. API Key Authentication

Example:

```ts
const response = await request.get('/api/weather', {
  headers: {
    'x-api-key': process.env.API_KEY!
  }
});
```

Environment variables should be managed securely in CI/CD.

---

## 31. Authentication Through `playwright.config.ts`

API configuration can be centralized.

Example:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: 'https://api.example.com'
  }
});
```

Tests can then use:

```ts
await request.get('/users');
```

instead of repeatedly writing the complete URL.

---

## 32. Creating a Dedicated API Request Context

For advanced scenarios:

```ts
import {
  request,
  APIRequestContext
} from '@playwright/test';

let api: APIRequestContext;

api = await request.newContext({
  baseURL: 'https://api.example.com'
});
```

Then:

```ts
const response = await api.get('/users');
```

Finally:

```ts
await api.dispose();
```

---

## 33. Why Use `request.newContext()`?

A dedicated API context is useful when you need:

- Different base URLs
- Different authentication
- Custom headers
- Separate sessions
- Multiple backend environments
- Independent API clients

---

## 34. API Client Class

Enterprise frameworks should encapsulate API operations.

Example:

```ts
import {
  APIRequestContext,
  APIResponse
} from '@playwright/test';

export class UsersApi {
  constructor(
    private readonly request: APIRequestContext
  ) {}

  async getUsers(): Promise<APIResponse> {
    return this.request.get('/api/users');
  }

  async createUser(data: object): Promise<APIResponse> {
    return this.request.post('/api/users', {
      data
    });
  }

  async deleteUser(id: number): Promise<APIResponse> {
    return this.request.delete(`/api/users/${id}`);
  }
}
```

---

## 35. Using an API Client

```ts
const usersApi = new UsersApi(request);

const response = await usersApi.getUsers();

expect(response.status()).toBe(200);
```

This keeps test cases focused on business behavior.

---

## 36. API Test Data Creation

API calls are excellent for test setup.

Instead of:

```text
Open UI
↓
Create user
↓
Fill 10 fields
↓
Save
```

You can:

```text
POST /users
↓
Create user
↓
Open UI
↓
Verify user
```

This makes tests faster.

---

## 37. UI + API Hybrid Testing

A powerful enterprise pattern is:

```text
API Setup
   ↓
UI Validation
   ↓
API Verification
```

Example:

```ts
const response = await request.post('/api/users', {
  data: {
    name: 'Sandeep'
  }
});

const user = await response.json();

await page.goto('/users');

await expect(
  page.getByText(user.name)
).toBeVisible();
```

---

## 38. API Cleanup After UI Test

You can also clean up through the API:

```ts
const response = await request.delete(
  `/api/users/${userId}`
);

expect(response.ok()).toBeTruthy();
```

This reduces dependence on UI cleanup steps.

---

## 39. Full Hybrid Example

```ts
test('create user through API and verify in UI', async ({
  page,
  request
}) => {
  const createResponse = await request.post('/api/users', {
    data: {
      name: 'Sandeep',
      role: 'QA Engineer'
    }
  });

  expect(createResponse.status()).toBe(201);

  const user = await createResponse.json();

  await page.goto('/users');

  await expect(
    page.getByText(user.name)
  ).toBeVisible();

  const deleteResponse = await request.delete(
    `/api/users/${user.id}`
  );

  expect(deleteResponse.ok()).toBeTruthy();
});
```

This is a strong enterprise automation pattern.

---

## 40. API Authentication Setup

A common pattern is:

```text
Login API
   ↓
Access Token
   ↓
API Context
   ↓
Authenticated API Calls
```

Example:

```ts
const loginResponse = await request.post('/api/login', {
  data: {
    username: process.env.API_USERNAME,
    password: process.env.API_PASSWORD
  }
});

const loginData = await loginResponse.json();

const token = loginData.accessToken;
```

Then:

```ts
const response = await request.get('/api/profile', {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

---

## 41. Reusable Authenticated API Context

```ts
const api = await request.newContext({
  baseURL: 'https://api.example.com',
  extraHTTPHeaders: {
    Authorization: `Bearer ${token}`
  }
});
```

Then every request from that context automatically includes the header.

---

## 42. API Fixtures

Custom fixtures can provide an API client to every test.

Conceptually:

```text
Test
 ↓
API Fixture
 ↓
Authenticated API Client
 ↓
REST API
```

Example:

```ts
type Fixtures = {
  usersApi: UsersApi;
};
```

Then tests can simply use:

```ts
test('user API test', async ({ usersApi }) => {
  const response = await usersApi.getUsers();

  expect(response.status()).toBe(200);
});
```

---

## 43. Environment-Based API Configuration

Use environment variables:

```ts
const baseURL =
  process.env.API_BASE_URL ??
  'http://localhost:8080';
```

Then:

```ts
const api = await request.newContext({
  baseURL
});
```

This supports:

```text
Local
QA
Staging
Production-like
```

environments without changing test code.

---

## 44. Negative API Testing

Professional API suites should test invalid scenarios.

Examples:

- Missing required fields
- Invalid ID
- Invalid token
- Expired token
- Insufficient permissions
- Duplicate record
- Invalid data type
- Invalid JSON
- Unsupported HTTP method

Example:

```ts
const response = await request.post('/api/users', {
  data: {}
});

expect(response.status()).toBe(400);
```

---

## 45. Authorization Testing

Authentication and authorization are different.

### Authentication

"Who are you?"

### Authorization

"What are you allowed to do?"

Example:

```ts
const response = await request.delete(
  '/api/admin/users/10',
  {
    headers: {
      Authorization: `Bearer ${regularUserToken}`
    }
  }
);

expect(response.status()).toBe(403);
```

---

## 46. CRUD Test Flow

A complete CRUD API test can follow:

```text
CREATE
  ↓
READ
  ↓
UPDATE
  ↓
READ
  ↓
DELETE
  ↓
VERIFY DELETED
```

Example:

```ts
const create = await request.post('/api/users', {
  data: {
    name: 'Sandeep'
  }
});

const user = await create.json();

const get = await request.get(
  `/api/users/${user.id}`
);

expect(get.status()).toBe(200);

const update = await request.patch(
  `/api/users/${user.id}`,
  {
    data: {
      name: 'Updated Sandeep'
    }
  }
);

expect(update.ok()).toBeTruthy();

const remove = await request.delete(
  `/api/users/${user.id}`
);

expect(remove.ok()).toBeTruthy();
```

---

## 47. API Response Time

You can measure response duration:

```ts
const start = Date.now();

const response = await request.get('/api/users');

const duration = Date.now() - start;

console.log(`Response time: ${duration} ms`);
```

Use performance thresholds carefully because CI environments can vary.

---

## 48. API Retry Strategy

Do not blindly retry every API test.

Retries should reflect the application's real behavior.

For application-level retry testing, deliberately simulate failures and verify the intended retry mechanism.

For infrastructure-level retries, configure Playwright or CI appropriately rather than hiding real defects.

---

## 49. API Test Organization

A scalable project can use:

```text
playwright/
├── api/
│   ├── clients/
│   │   ├── users.api.ts
│   │   ├── auth.api.ts
│   │   └── reports.api.ts
│   ├── models/
│   └── schemas/
├── tests/
│   ├── api/
│   └── ui/
├── fixtures/
├── test-data/
├── utils/
└── playwright.config.ts
```

---

## 50. API Models

TypeScript interfaces can describe API responses:

```ts
export interface User {
  id: number;
  name: string;
  role: string;
}

export interface UserResponse {
  users: User[];
}
```

Then:

```ts
const data =
  await response.json() as UserResponse;
```

This improves maintainability.

---

## 51. Type-Safe API Client

```ts
async getUsers(): Promise<UserResponse> {
  const response =
    await this.request.get('/api/users');

  expect(response.ok()).toBeTruthy();

  return await response.json();
}
```

Now tests can work with typed data.

---

## 52. Separating Transport and Assertions

Avoid putting every assertion inside API clients.

Prefer:

```text
API Client
  ↓
Sends request
  ↓
Returns response/data

Test
  ↓
Validates business behavior
```

This improves reuse.

---

## 53. Logging API Requests Safely

Good:

```ts
console.log({
  method: 'GET',
  endpoint: '/api/users'
});
```

Avoid:

```ts
console.log({
  headers,
  token,
  cookies,
  password
});
```

Never expose secrets in CI logs.

---

## 54. API Testing in CI/CD

API tests are usually fast enough to run frequently.

A mature pipeline can use:

```text
Pull Request
   ↓
API Smoke Tests
   ↓
UI Smoke Tests
   ↓
Regression
   ↓
Deployment Validation
```

API tests can also be used as setup and cleanup operations for UI tests.

---

## 55. Common Mistakes

### Mistake 1 — Testing only status codes

This is insufficient:

```ts
expect(response.status()).toBe(200);
```

Also validate important response data.

### Mistake 2 — Hard-coding credentials

Avoid:

```ts
password: 'MyPassword123'
```

Use environment variables or secure fixtures.

### Mistake 3 — Mixing all API logic into tests

Use API client classes.

### Mistake 4 — Ignoring cleanup

Created records should be cleaned up when appropriate.

### Mistake 5 — Using UI for all setup

Use APIs where they make setup faster and more reliable.

---

## 56. Production-Quality API Example

```ts
import { test, expect } from '@playwright/test';

test('create and validate user through API', async ({
  request
}) => {
  const createResponse = await request.post('/api/users', {
    data: {
      name: 'Sandeep',
      role: 'QA Engineer'
    }
  });

  expect(createResponse.status()).toBe(201);

  const createdUser = await createResponse.json();

  expect(createdUser).toHaveProperty('id');
  expect(createdUser.name).toBe('Sandeep');

  const getResponse = await request.get(
    `/api/users/${createdUser.id}`
  );

  expect(getResponse.status()).toBe(200);

  const fetchedUser = await getResponse.json();

  expect(fetchedUser.id).toBe(createdUser.id);
  expect(fetchedUser.name).toBe('Sandeep');

  const deleteResponse = await request.delete(
    `/api/users/${createdUser.id}`
  );

  expect(deleteResponse.ok()).toBeTruthy();
});
```

---

## 57. Interview Questions

### Q1. Can Playwright perform API testing?

Yes. Playwright provides `APIRequestContext` for sending HTTP requests.

### Q2. How do you send a GET request?

```ts
const response = await request.get('/users');
```

### Q3. How do you send JSON in a POST request?

```ts
await request.post('/users', {
  data: {
    name: 'Sandeep'
  }
});
```

### Q4. How do you read a JSON response?

```ts
const data = await response.json();
```

### Q5. How do you create a custom API context?

```ts
const api = await request.newContext({
  baseURL: 'https://api.example.com'
});
```

### Q6. Why use API calls for test setup?

They are usually faster and less fragile than performing setup through the UI.

### Q7. Can Playwright combine API and UI testing?

Yes. API calls can create or delete test data while the browser validates the UI.

### Q8. How do you authenticate API requests?

Use headers, cookies, API keys, or another mechanism required by the application's authentication contract.

### Q9. What is `APIRequestContext`?

It is Playwright's context for making HTTP API requests independently of browser UI interactions.

### Q10. Should API clients contain all assertions?

Generally no. API clients should focus on transport and reusable operations, while tests should validate business behavior.

---

## 58. Hands-On Exercises

### Exercise 1 — GET

Create a test that retrieves users and validates the status and response structure.

### Exercise 2 — POST

Create a user and validate the returned ID.

### Exercise 3 — PUT

Update a user's complete record.

### Exercise 4 — PATCH

Update only one user field.

### Exercise 5 — DELETE

Delete a user and verify the resource is no longer available.

### Exercise 6 — Authentication

Login through an API and use the returned token for another request.

### Exercise 7 — Negative Testing

Test invalid data and verify HTTP 400.

### Exercise 8 — Authorization

Use a regular-user token against an admin endpoint and verify HTTP 403.

### Exercise 9 — Hybrid Test

Create data through the API and verify it through the UI.

### Exercise 10 — Cleanup

Use the API to clean up records created by a UI test.

---

## 59. Key Takeaways

- Playwright supports API testing through `APIRequestContext`.
- The built-in `request` fixture is convenient for API tests.
- Playwright supports GET, POST, PUT, PATCH, DELETE, and HEAD operations.
- Use `data` for JSON request bodies.
- Use `params` for query parameters.
- Use headers for authentication and metadata.
- `response.json()` reads JSON responses.
- `response.text()` reads text responses.
- `response.status()` validates HTTP status codes.
- Dedicated API clients improve framework maintainability.
- API calls are excellent for test-data setup and cleanup.
- UI and API testing can be combined in hybrid tests.
- Authentication and authorization should be tested separately.
- Avoid hard-coded secrets.
- Use TypeScript interfaces for API models.
- Keep transport logic separate from business assertions.
- API tests can significantly improve overall automation speed and reliability.

---

## 60. Next Step

**Step 81 — Mastering Playwright Authentication, Storage State, Login State Reuse, Session Management & Multi-Role Testing**

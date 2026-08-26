# Playwright with TypeScript Mastery Bootcamp

# Step 100 — Mastering Playwright Test Data Management: Static Data, Dynamic Data, Fixtures, Factories, Faker, API Seeding, Database Setup, Cleanup, Isolation, Environment Data & Enterprise Test Data Architecture

## 1. Introduction

Test data is one of the most important foundations of reliable automation.

A Playwright test may be technically correct but still become:

```text
Flaky
Hard to maintain
Slow
Environment-dependent
Difficult to parallelize
Difficult to debug
```

if test data is poorly designed.

A mature test-data strategy should support:

```text
Static Data
Dynamic Data
Data Factories
Fixtures
API Seeding
Database Seeding
Cleanup
Isolation
Unique Data
Environment Data
Synthetic Data
Data Relationships
Parallel Execution
Enterprise Data Lifecycle
```

The goal is:

```text
Test
 ↓
Predictable Data
 ↓
Controlled Environment
 ↓
Deterministic Result
```

---

## 2. Why Test Data Matters

Consider a user-management test:

```text
Login
 ↓
Create User
 ↓
Search User
 ↓
Edit User
 ↓
Delete User
```

The test needs:

```text
Username
Password
Employee ID
Email
Role
Status
Department
```

If these values are shared by many tests, parallel execution can create conflicts.

---

## 3. Types of Test Data

Common categories include:

```text
Static Data
Dynamic Data
Generated Data
Seed Data
Reference Data
Transactional Data
Environment Data
Sensitive Data
Synthetic Data
```

Each category should be managed differently.

---

## 4. Static Test Data

Static data does not change frequently.

Example:

```json
{
  "username": "Admin",
  "password": "admin123",
  "role": "Admin"
}
```

This can be useful for:

```text
Login
Reference values
Fixed configuration
Known scenarios
```

---

## 5. Static Data File

Example:

```text
test-data/
├── loginData.json
├── adminData.json
├── expenseData.json
└── reportData.json
```

Example:

```json
{
  "validUser": {
    "username": "Admin",
    "password": "admin123"
  }
}
```

Avoid placing real credentials in source control.

---

## 6. Dynamic Test Data

Dynamic data is generated during test execution.

Example:

```ts
const username =
  `user_${Date.now()}`;
```

This can reduce collisions.

However, timestamp-only data is not always ideal for reproducibility.

---

## 7. Unique Test Data

A robust unique identifier may combine:

```text
Prefix
Timestamp
Random component
Worker ID
Test ID
```

Example concept:

```ts
const username =
  `user_${Date.now()}_${Math.random()
    .toString(36)
    .slice(2, 8)}`;
```

For stronger framework-level uniqueness, use a dedicated data factory or UUID.

---

## 8. UUID-Based Data

Example:

```ts
import { randomUUID } from 'node:crypto';

const userId =
  randomUUID();
```

Then:

```ts
const email =
  `test-${userId}@example.test`;
```

UUIDs are useful for avoiding collisions.

---

## 9. Synthetic Data

Synthetic data is artificial data created specifically for testing.

Example:

```text
Name:
Automation User

Email:
automation@example.test

Phone:
Synthetic test number
```

Use synthetic data instead of real customer information whenever possible.

---

## 10. Why Synthetic Data Is Important

Synthetic data helps protect:

```text
Privacy
Security
Compliance
Customer information
Production data
```

It also makes tests more deterministic.

---

## 11. Faker

A data-generation library can create realistic synthetic values.

Example concept:

```ts
import { faker } from '@faker-js/faker';

const user = {
  firstName:
    faker.person.firstName(),
  lastName:
    faker.person.lastName(),
  email:
    faker.internet.email()
};
```

This is useful for large test suites.

---

## 12. Faker and Reproducibility

Random generation can make failures difficult to reproduce.

You can use a seed.

Conceptually:

```ts
faker.seed(12345);
```

Then generated data becomes deterministic for that seed.

Use seeds strategically.

---

## 13. Random vs Deterministic Data

### Random

```text
Different data each run
```

Advantages:

```text
Collision reduction
Broader data variation
```

Disadvantages:

```text
Harder reproduction
Harder debugging
```

### Deterministic

```text
Same data for same seed
```

Advantages:

```text
Easy reproduction
Predictable
```

Disadvantages:

```text
Potential collisions
Less variation
```

A mature framework uses both intentionally.

---

## 14. Data Factory Pattern

A factory creates reusable test objects.

Example:

```ts
type UserData = {
  username: string;
  email: string;
  role: string;
};

function createUser(
  overrides:
    Partial<UserData> = {}
): UserData {

  return {
    username:
      `user_${Date.now()}`,
    email:
      `user_${Date.now()}@example.test`,
    role:
      'Employee',
    ...overrides
  };
}
```

Usage:

```ts
const user =
  createUser({
    role:
      'Admin'
  });
```

---

## 15. Why Factories Matter

Without a factory:

```text
Test 1 → manually creates user
Test 2 → manually creates user
Test 3 → manually creates user
```

With a factory:

```text
                User Factory
                /    |    \
              Test  Test  Test
```

Centralized data generation is easier to maintain.

---

## 16. Factory Overrides

Factories should support customization.

Example:

```ts
const admin =
  createUser({
    role:
      'Admin'
  });

const inactiveUser =
  createUser({
    status:
      'Inactive'
  });
```

This keeps common defaults centralized.

---

## 17. Nested Factories

Complex applications may need:

```text
Company
 ├── Department
 │    └── User
 └── User
```

Create factories for each domain object.

Example:

```text
companyFactory
departmentFactory
userFactory
expenseFactory
reportFactory
```

---

## 18. Related Test Data

Suppose an expense requires:

```text
User
Company
Category
Department
Expense
```

The test data dependency is:

```text
Company
  ↓
Department
  ↓
User
  ↓
Expense Category
  ↓
Expense
```

Create related records in a controlled order.

---

## 19. Test Data Builder

A builder can create complex objects step by step.

Conceptually:

```ts
const user =
  new UserBuilder()
    .withRole(
      'Admin'
    )
    .withStatus(
      'Active'
    )
    .build();
```

Builders are useful when an object has many optional properties.

---

## 20. Factory vs Builder

### Factory

Best for:

```text
Quick object creation
Common defaults
Simple overrides
```

### Builder

Best for:

```text
Complex objects
Many optional properties
Readable scenario construction
```

---

## 21. Fixtures and Test Data

Playwright fixtures can provide prepared data.

Example:

```ts
type Fixtures = {
  userData:
    UserData;
};
```

Then:

```ts
export const test =
  base.extend<Fixtures>({
    userData:
      async ({}, use) => {

        const user =
          createUser();

        await use(user);
      }
  });
```

---

## 22. Data Fixture Lifecycle

A fixture may:

```text
Create data
 ↓
Provide data to test
 ↓
Test uses data
 ↓
Cleanup
```

This is powerful for reusable setup.

---

## 23. Fixture Scope

Fixtures can be scoped appropriately.

Think about:

```text
Test scope
Worker scope
```

Test-scoped data is isolated per test.

Worker-scoped data can be shared by tests within a worker when safe.

---

## 24. Test-Scoped Data

Example:

```text
Test A
 ↓
Create User A

Test B
 ↓
Create User B
```

This is safer for parallel execution.

---

## 25. Worker-Scoped Data

Example:

```text
Worker 1
 ↓
Shared test environment

Test A
Test B
Test C
```

Worker-scoped data can improve performance.

But shared mutable state can cause conflicts.

---

## 26. Choosing Test vs Worker Scope

Use test scope when:

```text
Data is mutable
Tests modify records
Isolation is important
```

Use worker scope when:

```text
Data is read-only
Setup is expensive
Sharing is safe
```

---

## 27. API Data Seeding

Creating test data through the UI can be slow.

Example:

```text
Open UI
 ↓
Fill form
 ↓
Submit
 ↓
Wait
 ↓
Verify
```

For setup, API seeding can be much faster:

```text
API POST
 ↓
Data Created
 ↓
Open UI
 ↓
Test
```

---

## 28. Playwright APIRequestContext

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
  response.ok()
).toBe(true);
```

This can prepare backend state efficiently.

---

## 29. UI vs API Data Creation

### UI

Use when testing:

```text
User creation workflow
Validation
Frontend behavior
UI-to-API integration
```

### API

Use when preparing:

```text
Preconditions
Large datasets
Complex relationships
Repeated setup
```

---

## 30. Database Seeding

For environments where database access is permitted:

```text
Test
 ↓
Seed Database
 ↓
Application
 ↓
UI
```

Database setup can be fast but introduces stronger infrastructure coupling.

---

## 31. Database Setup Considerations

Consider:

```text
Credentials
Environment
Transactions
Cleanup
Permissions
Parallel execution
Schema version
Test isolation
```

Never expose production credentials.

---

## 32. Database Test Data Strategy

Possible strategies:

```text
Seed scripts
Fixtures
Factories
Transactions
Temporary schemas
Dedicated databases
```

Choose according to environment architecture.

---

## 33. Transaction-Based Isolation

Conceptually:

```text
Begin Transaction
 ↓
Create Test Data
 ↓
Run Test
 ↓
Rollback
```

This can provide excellent cleanup when the application architecture supports it.

---

## 34. Cleanup

Test data should not accumulate indefinitely.

Example:

```text
Create User
 ↓
Run Test
 ↓
Delete User
```

Cleanup can happen through:

```text
API
Database
Fixture teardown
Application endpoint
```

---

## 35. Why Cleanup Matters

Without cleanup:

```text
Test 1 → User A
Test 2 → User B
Test 3 → User C
...
```

Eventually:

```text
Database
 ↓
Thousands of test records
```

This can slow tests and cause collisions.

---

## 36. Cleanup in Fixtures

Conceptual pattern:

```ts
userFixture:
  async ({ request }, use) => {

    const user =
      await createUser(
        request
      );

    await use(user);

    await deleteUser(
      request,
      user.id
    );
  }
```

This gives setup and teardown in one place.

---

## 37. Cleanup Should Be Resilient

If the test fails:

```text
Test Failure
 ↓
Cleanup Still Runs
```

Use fixture teardown or appropriate test lifecycle mechanisms.

---

## 38. Cleanup Failures

If cleanup itself fails:

```text
Original Failure
+
Cleanup Failure
```

Debugging can become difficult.

Log enough context to identify cleanup problems without exposing secrets.

---

## 39. Cleanup by Test Prefix

An alternative strategy is to create data with a unique test prefix:

```text
pw-test-123-user
```

Then cleanup can target:

```text
pw-test-123-*
```

This can be useful for recovery cleanup jobs.

---

## 40. Data Ownership

Each test should know:

```text
What data it created
Who owns it
How to clean it
```

Example:

```ts
const createdUser =
  await createUser();

testInfo.attach(
  'created-user-id',
  {
    body:
      createdUser.id
  }
);
```

Do not expose sensitive information in artifacts.

---

## 41. Test Isolation

A test should ideally not depend on:

```text
Previous test
Shared mutable record
Execution order
Previous test's cleanup
```

Good:

```text
Test A → Own Data
Test B → Own Data
Test C → Own Data
```

---

## 42. Parallel Execution

Suppose two workers create:

```text
username = test-user
```

They may collide.

Instead:

```text
worker-1-user
worker-2-user
```

or use unique IDs.

---

## 43. Worker-Based Data

Playwright provides worker information through test fixtures.

A unique data namespace can be built from:

```text
Worker index
Test title
UUID
```

Example concept:

```ts
const namespace =
  `worker-${testInfo.workerIndex}`;
```

Use the available test information appropriate to the fixture design.

---

## 44. Parallel-Safe Email Data

Instead of:

```text
test@example.test
```

use:

```text
test-<unique-id>@example.test
```

This prevents duplicate account creation.

---

## 45. Data Collision

Common collision sources:

```text
Username
Email
Employee ID
Order Number
Invoice Number
Reference Number
```

Every uniquely constrained field should be considered.

---

## 46. Environment-Specific Data

Different environments may have:

```text
Different URLs
Different users
Different roles
Different feature flags
Different database state
```

Do not hard-code environment-specific values into every test.

---

## 47. Environment Variables

Example:

```env
BASE_URL=https://test.example.com
TEST_USERNAME=automation
TEST_PASSWORD=secret
```

Load through configuration.

Never commit secrets.

---

## 48. Environment Configuration

Example:

```ts
const baseURL =
  process.env.BASE_URL;
```

Playwright configuration can use environment variables for environment selection.

---

## 49. Environment Data vs Test Data

### Environment Data

```text
Base URL
API URL
Credentials
Tenant ID
Feature flags
```

### Test Data

```text
User
Expense
Order
Product
Report
```

Keep these concepts separate.

---

## 50. Secrets

Never store:

```text
Passwords
API tokens
Private keys
Production credentials
```

inside:

```text
JSON test data
Git repository
Test reports
Screenshots
HAR files
```

Use secure secret management.

---

## 51. Sensitive Test Data

Even non-production environments may contain sensitive information.

Prefer:

```text
Synthetic users
Synthetic emails
Synthetic phone numbers
Synthetic financial records
```

---

## 52. Test Data Repository

A large project may use:

```text
test-data/
├── users/
├── expenses/
├── companies/
├── reports/
├── authentication/
└── factories/
```

Keep the structure aligned with business domains.

---

## 53. JSON vs TypeScript Data

### JSON

Good for:

```text
Simple static data
External datasets
Data-driven scenarios
```

### TypeScript

Good for:

```text
Factories
Dynamic values
Functions
Conditional data
Typed models
```

A mature project often uses both.

---

## 54. Typed Test Data

Example:

```ts
export interface UserData {
  username:
    string;

  email:
    string;

  role:
    string;

  status:
    string;
}
```

This catches data-shape mistakes at compile time.

---

## 55. Type-Safe Factories

Example:

```ts
function createUser(
  overrides:
    Partial<UserData> = {}
): UserData {

  return {
    username:
      'automation-user',
    email:
      'automation@example.test',
    role:
      'Employee',
    status:
      'Active',
    ...overrides
  };
}
```

---

## 56. Data-Driven Testing

Example dataset:

```ts
const loginCases = [
  {
    name:
      'valid login',
    username:
      'Admin',
    password:
      'password',
    expected:
      'success'
  },
  {
    name:
      'invalid password',
    username:
      'Admin',
    password:
      'wrong',
    expected:
      'error'
  }
];
```

Then iterate over scenarios.

---

## 57. Data-Driven Test Example

```ts
for (
  const testCase
  of loginCases
) {

  test(
    testCase.name,
    async ({ page }) => {

      // execute scenario
    }
  );
}
```

Keep datasets readable and avoid generating huge numbers of nearly identical tests.

---

## 58. Scenario Data

A scenario can describe:

```text
Given
When
Then
```

Example:

```ts
{
  role:
    'Manager',
  expenseStatus:
    'Pending',
  expectedAction:
    'Approve'
}
```

This makes business scenarios explicit.

---

## 59. Test Data Dependencies

Some data requires previous data.

Example:

```text
Company
 ↓
Department
 ↓
Employee
 ↓
Expense
 ↓
Approval
```

Use factories or setup helpers that return IDs and relationships.

---

## 60. ID Propagation

Example:

```ts
const company =
  await createCompany();

const employee =
  await createEmployee({
    companyId:
      company.id
  });

const expense =
  await createExpense({
    employeeId:
      employee.id
  });
```

This is clearer than hard-coded IDs.

---

## 61. Data Setup Helper

Example:

```ts
async function
createExpenseScenario(
  request: APIRequestContext
) {

  const company =
    await createCompany(
      request
    );

  const user =
    await createUser(
      request,
      company.id
    );

  const expense =
    await createExpense(
      request,
      user.id
    );

  return {
    company,
    user,
    expense
  };
}
```

This creates a reusable business scenario.

---

## 62. Test Data Fixtures

A fixture can expose:

```ts
type Fixtures = {
  expenseScenario:
    ExpenseScenario;
};
```

Then tests can focus on behavior:

```ts
test(
  'approve expense',
  async ({
    expenseScenario,
    page
  }) => {

    // use prepared scenario
  }
);
```

---

## 63. Data Factories + API

A powerful architecture is:

```text
Factory
 ↓
Data Object
 ↓
API Seeder
 ↓
Application State
 ↓
Playwright Test
```

Example:

```ts
const user =
  createUser();

await userApi.create(
  user
);
```

---

## 64. Data Factories + Database

Another option:

```text
Factory
 ↓
Data Object
 ↓
Repository
 ↓
Database
```

This can be fast but should be used only when direct database setup is appropriate.

---

## 65. Avoid UI Setup When Possible

If a test is validating:

```text
Expense approval
```

do not necessarily create the expense through 15 UI steps.

Instead:

```text
API/DB setup
 ↓
Open approval UI
 ↓
Approve expense
 ↓
Validate result
```

This keeps the test focused.

---

## 66. Test Data vs Test Steps

Separate:

```text
Data Creation
```

from:

```text
Behavior Under Test
```

Good:

```text
Setup expense
 ↓
Test approval
```

Poor:

```text
Create company
 ↓
Create department
 ↓
Create user
 ↓
Create category
 ↓
Create expense
 ↓
Finally test approval
```

all through the UI in every test.

---

## 67. Data Cleanup Strategies

Common strategies:

```text
Delete created records
Rollback transaction
Reset database
Truncate test tables
Use isolated schema
Use disposable environment
```

Choose the least destructive strategy that provides isolation.

---

## 68. Reset Database

A test environment may periodically reset:

```text
Database
 ↓
Known baseline
 ↓
Test suite
```

Useful for integration environments.

Do not use destructive reset operations against production.

---

## 69. Seed Baseline Data

Example:

```text
Baseline
├── Admin
├── Manager
├── Employee
├── Categories
└── Departments
```

Then each test adds its own temporary records.

---

## 70. Baseline vs Scenario Data

### Baseline

Stable shared reference data:

```text
Roles
Permissions
Categories
Static configuration
```

### Scenario

Created specifically for one test:

```text
User
Expense
Order
Report
```

Keep them separate.

---

## 71. Reference Data

Reference data may include:

```text
Countries
Currencies
Roles
Departments
Expense Categories
Status values
```

These can often be seeded once if they are immutable for the test environment.

---

## 72. Mutable Reference Data

If reference data can change:

```text
Test A changes category
Test B expects original category
```

Shared state becomes dangerous.

Use isolation or restore the original state.

---

## 73. Data Versioning

When application schemas change:

```text
Old Data
 ↓
Migration
 ↓
New Schema
```

Test data should evolve with the application.

Version complex seed datasets when necessary.

---

## 74. Data Contract

Define required fields clearly.

Example:

```ts
interface ExpenseData {
  amount:
    number;
  categoryId:
    number;
  employeeId:
    number;
  status:
    'PENDING'
    | 'APPROVED'
    | 'REJECTED';
}
```

This prevents invalid test objects.

---

## 75. Data Validation Before Test

Factories can validate data before returning it.

Example:

```ts
const user =
  createUser();

if (
  !user.email
) {
  throw new Error(
    'User email is required'
  );
}
```

Typed factories reduce many of these issues at compile time.

---

## 76. Unique Namespaces

For parallel-safe tests:

```text
pw-<worker>-<uuid>
```

Example:

```ts
const prefix =
  `pw-${testInfo.workerIndex}-${randomUUID()}`;
```

Use it for:

```text
Username
Email
Company name
Order reference
```

---

## 77. Data Namespaces

Example:

```text
pw-worker-0-user-123
pw-worker-1-user-456
```

A namespace can help with:

```text
Debugging
Cleanup
Collision prevention
Ownership
```

---

## 78. Cleanup by Namespace

If a test fails and leaves data behind:

```text
pw-worker-0-*
```

can be identified and cleaned by a recovery process.

This is especially useful in shared test environments.

---

## 79. Test Data Logging

Log:

```text
Created entity type
Synthetic identifier
Test scenario
Cleanup result
```

Avoid logging:

```text
Passwords
Tokens
Sensitive personal data
```

---

## 80. Test Artifacts

Test data may accidentally appear in:

```text
Screenshots
Videos
Traces
Reports
Console logs
HAR files
```

Use synthetic data and mask sensitive values.

---

## 81. Data Masking

If sensitive data is unavoidable:

```text
123456789
```

should not appear as-is in logs.

Use masking:

```text
******789
```

Prefer synthetic data instead.

---

## 82. Test Data in CI

CI should be able to create its own test data.

Avoid depending on:

```text
Developer laptop database
Manual records
Previously executed tests
Shared user state
```

---

## 83. CI Data Architecture

```text
CI Job
 ↓
Environment
 ↓
Seed Baseline
 ↓
Create Test Data
 ↓
Run Tests
 ↓
Cleanup
 ↓
Publish Report
```

This creates a repeatable pipeline.

---

## 84. Test Data in Containers

When tests run in containers:

```text
Playwright Container
 ↓
Application Container
 ↓
Database Container
```

Data can be initialized through:

```text
Migration
Seed script
API
Fixture
```

This improves environment reproducibility.

---

## 85. Test Data and Docker

A common setup:

```text
docker-compose
 ├── app
 ├── postgres
 └── test-runner
```

The test runner can execute seed operations against the isolated database.

---

## 86. Data Isolation by Environment

Ideal:

```text
Developer
 → Local DB

CI
 → CI DB

QA
 → QA DB
```

Avoid cross-environment data dependencies.

---

## 87. Environment Matrix

Example:

| Environment | Data Strategy |
|---|---|
| Local | Seed/Fake |
| CI | Isolated Seed |
| QA | Controlled Shared |
| Staging | Synthetic |
| Production | Never test destructive data workflows |

---

## 88. Production Safety

Never let automated cleanup accidentally target production.

Use safeguards:

```text
Environment checks
Explicit allowlists
Dedicated credentials
Non-production-only cleanup
```

Example:

```ts
if (
  process.env.NODE_ENV ===
  'production'
) {
  throw new Error(
    'Destructive test-data cleanup is disabled in production'
  );
}
```

---

## 89. Data Ownership Model

Every test-created entity should have:

```text
Owner
Namespace
Creation method
Cleanup method
```

Example:

```text
Test: expense-approval
Owner: worker-2
Namespace: pw-2-abc123
Cleanup: DELETE /api/expenses/{id}
```

---

## 90. Data Lifecycle

A complete lifecycle:

```text
Generate
   ↓
Validate
   ↓
Seed
   ↓
Use
   ↓
Verify
   ↓
Cleanup
   ↓
Audit
```

---

## 91. Data Lifecycle in Fixtures

```text
Fixture Setup
      ↓
Create Data
      ↓
use(data)
      ↓
Test Executes
      ↓
Fixture Teardown
      ↓
Cleanup
```

This is one of the most useful patterns for Playwright frameworks.

---

## 92. Data Management Layer

A mature framework may contain:

```text
test-data/
├── factories/
├── builders/
├── fixtures/
├── seeders/
├── api/
├── repositories/
├── schemas/
└── cleanup/
```

This separates data concerns from test behavior.

---

## 93. Repository Pattern

A repository can encapsulate data operations.

Example:

```ts
class UserRepository {

  constructor(
    private readonly request:
      APIRequestContext
  ) {}

  async create(
    user: UserData
  ) {
    return this.request.post(
      '/api/users',
      {
        data: user
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

Tests do not need to know endpoint details.

---

## 94. Data Service Layer

Instead of:

```text
Test
 ↓
Raw API calls
```

use:

```text
Test
 ↓
Data Service
 ↓
API/DB
```

This improves maintainability.

---

## 95. Example Data Service

```ts
class TestDataService {

  async createUser(
    data: UserData
  ) {
    // create through API
  }

  async createExpense(
    data: ExpenseData
  ) {
    // create through API
  }

  async cleanupUser(
    id: number
  ) {
    // delete
  }
}
```

---

## 96. Data Factories Should Not Perform Side Effects

Prefer:

```ts
createUser()
```

to return data.

Then:

```ts
userService.create(
  user
);
```

performs the side effect.

This separation improves testing.

---

## 97. Factory vs Seeder

### Factory

Creates an object:

```ts
const user =
  createUser();
```

### Seeder

Persists the object:

```ts
await seedUser(
  user
);
```

Keeping these separate improves flexibility.

---

## 98. Data Builder Example

```ts
const expense =
  new ExpenseBuilder()
    .withAmount(
      1000
    )
    .withStatus(
      'PENDING'
    )
    .withCategory(
      category.id
    )
    .build();
```

Then:

```ts
await expenseService
  .create(expense);
```

---

## 99. Data Strategy for Playwright Bootcamp

For this project, a practical structure is:

```text
playwright/
├── test-data/
│   ├── static/
│   ├── factories/
│   ├── builders/
│   ├── schemas/
│   └── scenarios/
├── fixtures/
├── utils/
│   ├── data/
│   ├── api/
│   └── cleanup/
└── tests/
```

---

## 100. Example Bootcamp Data Structure

```text
test-data/
├── loginData.json
├── adminData.json
├── users/
│   ├── userFactory.ts
│   └── userData.ts
├── expenses/
│   ├── expenseFactory.ts
│   └── expenseData.ts
├── reports/
│   └── reportScenarios.ts
└── scenarios/
    ├── approvalScenario.ts
    └── rejectionScenario.ts
```

---

## 101. Complete User Factory

```ts
import {
  randomUUID
} from 'node:crypto';

export type UserData = {
  username: string;
  email: string;
  role: string;
  status: string;
};

export function createUser(
  overrides:
    Partial<UserData> = {}
): UserData {

  const id =
    randomUUID();

  return {
    username:
      `pw-user-${id}`,
    email:
      `pw-${id}@example.test`,
    role:
      'Employee',
    status:
      'Active',
    ...overrides
  };
}
```

---

## 102. API Seeder Example

```ts
async function seedUser(
  request:
    APIRequestContext,
  user: UserData
) {

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

  return response.json();
}
```

---

## 103. Fixture Combining Factory and Seeder

```ts
type Fixtures = {
  user:
    UserData;
};

export const test =
  base.extend<Fixtures>({
    user:
      async (
        { request },
        use
      ) => {

        const user =
          createUser();

        await seedUser(
          request,
          user
        );

        await use(user);

        // cleanup
      }
  });
```

---

## 104. Data-Driven Test Example

```ts
const roles = [
  'Admin',
  'Manager',
  'Employee'
];

for (
  const role
  of roles
) {

  test(
    `user with role ${role}`,
    async ({
      page
    }) => {

      // scenario
    }
  );
}
```

Keep generated test titles clear.

---

## 105. Avoid Huge Data-Driven Suites

If you have:

```text
100 roles
100 browsers
10 environments
20 scenarios
```

the test count can explode.

Use a representative matrix rather than testing every combination through UI.

---

## 106. Pairwise Data Strategy

For complex combinations:

```text
Role
Browser
Status
Feature Flag
Payment Type
```

consider pairwise or risk-based selection instead of exhaustive combinations.

This reduces execution time.

---

## 107. Boundary Test Data

Important values include:

```text
0
1
Maximum
Minimum
Empty
Null
Very long string
Special characters
Unicode
Negative values
```

Factories should support boundary overrides.

---

## 108. Example Boundary Factory

```ts
const user =
  createUser({
    username:
      'a'.repeat(255)
  });
```

Then validate the application's expected behavior.

---

## 109. Invalid Data Factories

It can be useful to create intentionally invalid data.

Example:

```ts
function createInvalidUser() {
  return {
    username: '',
    email:
      'not-an-email',
    role:
      'UNKNOWN'
  };
}
```

This supports negative testing.

---

## 110. Data Classification

A mature framework can classify data:

```text
SAFE
SYNTHETIC
SENSITIVE
SECRET
ENVIRONMENT
REFERENCE
TRANSACTIONAL
```

This helps determine storage and logging rules.

---

## 111. Data Storage Rules

Example:

```text
Static safe data
 → Git

Synthetic generated data
 → Runtime

Secrets
 → Secret manager / environment

Large datasets
 → Controlled fixture files

Sensitive production-like data
 → Avoid
```

---

## 112. Data Expiration

Temporary data should have a lifecycle.

For example:

```text
Created today
 ↓
Used by test
 ↓
Deleted after test
```

Recovery jobs can remove abandoned test namespaces periodically.

---

## 113. Orphan Data

Orphan data occurs when:

```text
Test crashes
 ↓
Cleanup does not complete
 ↓
Record remains
```

Prevent it through:

```text
Fixture teardown
Unique namespace
Periodic cleanup
TTL where supported
```

---

## 114. Cleanup Verification

Cleanup itself can be verified.

Example:

```ts
await deleteUser(
  user.id
);

const response =
  await getUser(
    user.id
  );

expect(
  response.status()
).toBe(404);
```

Do this when cleanup correctness matters.

---

## 115. Test Data Observability

Useful information:

```text
Test ID
Worker
Created entity IDs
Scenario
Environment
Cleanup status
```

This makes failed test investigations easier.

---

## 116. Data Debugging

When a test fails, ask:

```text
Was data created?
Was it created correctly?
Was the correct environment used?
Did another test modify it?
Was cleanup already executed?
Was the ID unique?
```

---

## 117. Data and Flakiness

Common data-related flakiness:

```text
Shared account
Duplicate username
Stale record
Missing prerequisite
Cleanup race
Parallel collision
Environment mismatch
```

Good test-data architecture eliminates many of these issues.

---

## 118. Test Data Anti-Patterns

Avoid:

```text
Shared mutable users
Hard-coded IDs
Production data
Manual setup dependencies
Test-order dependencies
Global cleanup
Random data without reproducibility
Data created through UI for every setup
Secrets in JSON
Unscoped database resets
```

---

## 119. Enterprise Best Practices

1. Prefer synthetic data.
2. Generate unique transactional data.
3. Keep static reference data separate.
4. Use factories for common objects.
5. Use builders for complex objects.
6. Use API setup for fast preconditions.
7. Use UI setup only when UI behavior is under test.
8. Keep cleanup close to ownership.
9. Design for parallel execution.
10. Separate environment configuration from test data.
11. Protect secrets.
12. Avoid production data.
13. Use typed models.
14. Support deterministic seeds when needed.
15. Build reusable data services.
16. Track test-created entities.
17. Recover orphaned data.
18. Validate important business data.
19. Keep datasets maintainable.
20. Document data lifecycle.

---

## 120. Enterprise Test Data Architecture

```text
                         Test Suite
                             │
                             ▼
                      Test Data Layer
                             │
       ┌─────────────────────┼─────────────────────┐
       │                     │                     │
    Static Data          Factories             Scenarios
       │                     │                     │
       └─────────────────────┼─────────────────────┘
                             │
                       Data Services
                             │
              ┌──────────────┼──────────────┐
              │                             │
          API Seeder                    DB Seeder
              │                             │
              └──────────────┬──────────────┘
                             │
                     Application State
                             │
                          Test
                             │
                         Cleanup
                             │
                      Data Lifecycle
```

---

## 121. Test Data Flow

```text
Scenario Definition
        ↓
Factory / Builder
        ↓
Validate Data
        ↓
Seed Through API/DB
        ↓
Run Playwright Test
        ↓
Verify Business Result
        ↓
Cleanup
        ↓
Recovery Cleanup if Needed
```

---

## 122. Interview Questions

### Q1. Why is test data management important?

Because reliable automation requires predictable, isolated, reproducible data.

### Q2. What is a test data factory?

A reusable function or object that creates test data with sensible defaults and optional overrides.

### Q3. Why use API seeding?

It is generally faster and more focused than creating setup data through the UI.

### Q4. What is test data isolation?

Ensuring one test does not unexpectedly affect another test's data.

### Q5. How do you make data safe for parallel execution?

Use unique identifiers, namespaces, isolated resources, and appropriate fixture scope.

### Q6. Factory vs builder?

Factories are convenient for common object creation; builders are useful for complex configurable objects.

### Q7. How should secrets be handled?

Use environment/secret-management mechanisms and never commit credentials to source control.

### Q8. How do you clean test data?

Use fixture teardown, API cleanup, database cleanup, transactions, or isolated environments.

### Q9. Why avoid production data?

It creates privacy, security, compliance, and test-determinism risks.

### Q10. How do you handle asynchronous test data?

Create the prerequisite, then poll the system state using a condition-based strategy rather than fixed sleeps.

---

## 123. Hands-On Exercises

### Exercise 1 — Static Data

Create:

```text
loginData.json
adminData.json
```

and use them in tests.

### Exercise 2 — User Factory

Create:

```ts
createUser()
```

with overrides.

### Exercise 3 — Faker

Generate synthetic:

```text
Name
Email
Address
Company
```

### Exercise 4 — Deterministic Faker

Use a fixed seed and verify reproducibility.

### Exercise 5 — API Seeder

Create a user through an API before a UI test.

### Exercise 6 — Fixture Data

Create a fixture that:

```text
Creates user
 ↓
Provides user
 ↓
Cleans user
```

### Exercise 7 — Parallel Data

Run tests in parallel and prove that unique users do not collide.

### Exercise 8 — Cleanup

Implement API-based cleanup after a test.

### Exercise 9 — Complex Scenario

Create:

```text
Company
 ↓
Department
 ↓
User
 ↓
Expense
```

through a reusable scenario builder.

### Exercise 10 — Boundary Data

Test:

```text
Empty
Minimum
Maximum
Long string
Special characters
```

---

## 124. Enterprise Test Data Challenge

Build a complete test-data framework supporting:

```text
Static Data
Dynamic Data
Factories
Builders
Faker
API Seeding
Database Seeding
Fixtures
Cleanup
Isolation
Parallel Execution
Environment Data
Synthetic Data
Boundary Data
Scenario Data
```

Requirements:

1. Create typed data models.
2. Create user factories.
3. Create expense factories.
4. Create scenario builders.
5. Generate unique data.
6. Support deterministic generation.
7. Seed data through API.
8. Support optional database seeding.
9. Implement fixture-based setup.
10. Implement fixture-based cleanup.
11. Support parallel workers.
12. Prevent collisions.
13. Protect secrets.
14. Prevent production cleanup.
15. Track created entity IDs.
16. Handle orphan data.
17. Document the data lifecycle.
18. Keep test setup independent from UI behavior.

---

## 125. Final Checklist

```text
[ ] Static test data understood
[ ] Dynamic test data understood
[ ] Synthetic data understood
[ ] Unique data generation understood
[ ] UUID generation understood
[ ] Faker understood
[ ] Faker seeding understood
[ ] Data factories understood
[ ] Factory overrides understood
[ ] Builders understood
[ ] Fixtures + data understood
[ ] Test scope understood
[ ] Worker scope understood
[ ] API seeding understood
[ ] Database seeding understood
[ ] Cleanup understood
[ ] Test isolation understood
[ ] Parallel data isolation understood
[ ] Environment data understood
[ ] Secrets management understood
[ ] Reference data understood
[ ] Transactional data understood
[ ] Data relationships understood
[ ] Data namespaces understood
[ ] Orphan data understood
[ ] Recovery cleanup understood
[ ] Boundary data understood
[ ] Invalid data understood
[ ] Data-driven testing understood
[ ] Data service layer understood
[ ] Repository pattern understood
[ ] Enterprise data architecture understood
```

---

## 126. Key Takeaways

- Test data is a core part of test automation architecture.
- Static data is useful for stable reference scenarios.
- Dynamic and unique data reduce collisions.
- Synthetic data is safer than real customer data.
- Faker can generate realistic data, but deterministic seeds can improve reproducibility.
- Factories centralize common data creation.
- Builders help create complex scenarios.
- API seeding is usually faster than UI-based setup.
- Database seeding can be powerful but increases infrastructure coupling.
- Fixtures are ideal for reusable setup and cleanup.
- Test-scoped data is generally safer for mutable records.
- Worker-scoped data should be used only when sharing is safe.
- Parallel tests require unique namespaces or isolated resources.
- Cleanup should happen even when tests fail.
- Environment configuration and test data should remain separate.
- Secrets must never be committed to test data files.
- Business scenarios should be represented through reusable data builders or services.
- Test data should have a complete lifecycle: generate, validate, seed, use, verify, and clean up.
- A strong data strategy reduces flakiness and makes Playwright suites faster, safer, and easier to maintain.

---

## 127. Next Step

**Step 101 — Mastering Playwright Authentication & Authorization at Enterprise Scale: Login Strategies, Storage State, Sessions, Cookies, Tokens, Multi-Role Testing, RBAC, Permissions, Session Expiry & Security Test Architecture**

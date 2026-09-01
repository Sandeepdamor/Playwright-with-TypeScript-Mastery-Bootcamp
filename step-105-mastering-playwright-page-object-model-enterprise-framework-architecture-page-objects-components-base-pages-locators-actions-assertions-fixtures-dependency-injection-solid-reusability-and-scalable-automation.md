# Playwright with TypeScript Mastery Bootcamp

# Step 105 — Mastering Playwright Page Object Model & Enterprise Framework Architecture

## 1. Introduction

The Page Object Model (POM) is an important design pattern for scalable Playwright automation.

A mature framework separates:

```text
Test Intent
     ↓
Page / Component Objects
     ↓
Locators + Actions
     ↓
Playwright
     ↓
Application
```

The goal is to make automation:

```text
Readable
Reusable
Maintainable
Testable
Scalable
Parallel-safe
Easy to debug
```

---

## 2. What Is Page Object Model?

Page Object Model represents application pages or meaningful UI areas as reusable objects.

Examples:

```text
LoginPage
DashboardPage
AdminPage
UserPage
ExpensePage
ReportPage
```

A test can then express business intent:

```ts
await loginPage.login(
  username,
  password
);

await dashboardPage
  .expectLoaded();
```

instead of repeating selectors and implementation details.

---

## 3. Why POM Is Useful

Without POM:

```text
Test 1 → selectors + actions
Test 2 → same selectors + actions
Test 3 → same selectors + actions
```

With POM:

```text
Tests
  ↓
Page Object
  ↓
Reusable implementation
```

When the UI changes, the implementation can often be updated in one place.

---

## 4. POM Does Not Mean One Class Per URL

A page may contain reusable components:

```text
DashboardPage
 ├── Header
 ├── Sidebar
 ├── NotificationPanel
 └── DataTable
```

Objects should represent meaningful behavior and UI boundaries.

---

## 5. Basic Page Object

```ts
import {
  Page,
  Locator
} from '@playwright/test';

export class LoginPage {

  readonly username: Locator;
  readonly password: Locator;
  readonly loginButton: Locator;

  constructor(
    private readonly page: Page
  ) {
    this.username =
      page.getByLabel('Username');

    this.password =
      page.getByLabel('Password');

    this.loginButton =
      page.getByRole(
        'button',
        {
          name: 'Login'
        }
      );
  }
}
```

---

## 6. Page Object Actions

```ts
async login(
  username: string,
  password: string
): Promise<void> {

  await this.username
    .fill(username);

  await this.password
    .fill(password);

  await this.loginButton
    .click();
}
```

The test becomes:

```ts
await loginPage.login(
  username,
  password
);
```

---

## 7. Locators Inside Page Objects

Keep locators close to the behavior that uses them.

Example:

```ts
readonly saveButton =
  this.page.getByRole(
    'button',
    {
      name: 'Save'
    }
  );
```

Avoid scattering selectors throughout tests.

---

## 8. Prefer Semantic Locators

Prefer:

```ts
getByRole()
getByLabel()
getByText()
getByPlaceholder()
```

when appropriate.

Example:

```ts
page.getByRole(
  'button',
  {
    name: 'Save'
  }
);
```

---

## 9. Test IDs

When semantic locators are not sufficient:

```ts
page.getByTestId(
  'expense-status'
);
```

Use stable test IDs rather than styling selectors.

---

## 10. Avoid Fragile Selectors

Avoid selectors based heavily on:

```text
Generated CSS classes
DOM position
nth-child
Deep XPath
Dynamic IDs
Styling implementation
```

Prefer stable application contracts.

---

## 11. Page Object Responsibilities

A page object should generally contain:

```text
Locators
Navigation
User Actions
Page-specific behavior
Page-level state checks
```

Do not turn it into a container for unrelated business logic.

---

## 12. What Should Stay in Tests?

Tests should normally contain:

```text
Scenario
Business intent
Test data selection
High-level assertions
Workflow orchestration
```

Example:

```ts
test(
  'manager approves expense',
  async ({
    expensePage
  }) => {

    await expensePage
      .openExpense(
        expenseId
      );

    await expensePage
      .approve();

    await expect(
      expensePage.status
    ).toHaveText(
      'Approved'
    );
  }
);
```

---

## 13. Actions vs Assertions

A reusable action:

```ts
async approve() {
  await this.approveButton
    .click();
}
```

The business assertion can remain visible:

```ts
await expect(
  expensePage.status
).toHaveText(
  'Approved'
);
```

This keeps the scenario readable.

---

## 14. Page-Level Assertions

Page objects can expose stable page-level checks:

```ts
async expectLoaded() {
  await expect(
    this.heading
  ).toBeVisible();
}
```

Use this when it improves test readability.

---

## 15. Page Object Granularity

Too little abstraction:

```text
Everything inside tests
```

Too much abstraction:

```text
Every click has a class
Every locator has a helper
Every text has a wrapper
```

Create abstractions around meaningful behavior.

---

## 16. Component Objects

Reusable UI components can have their own classes.

Examples:

```text
Header
Sidebar
Modal
Dropdown
DataTable
DatePicker
Pagination
Toast
```

Example:

```ts
export class Header {

  readonly profileMenu: Locator;

  constructor(
    private readonly page: Page
  ) {
    this.profileMenu =
      page.getByRole(
        'button',
        {
          name: 'Profile'
        }
      );
  }

  async logout() {
    // logout behavior
  }
}
```

---

## 17. Composition

A page can compose multiple components:

```ts
export class DashboardPage {

  readonly header: Header;
  readonly sidebar: Sidebar;

  constructor(
    private readonly page: Page
  ) {
    this.header =
      new Header(page);

    this.sidebar =
      new Sidebar(page);
  }
}
```

---

## 18. Why Composition Is Powerful

Instead of:

```text
DashboardPage
 └── 500 locators
```

use:

```text
DashboardPage
 ├── Header
 ├── Sidebar
 ├── NotificationPanel
 └── DataTable
```

Each object has a focused responsibility.

---

## 19. Reusable DataTable Component

```ts
export class DataTable {

  constructor(
    private readonly page: Page
  ) {}

  row(text: string) {
    return this.page
      .getByRole('row')
      .filter({
        hasText: text
      });
  }

  async expectRow(
    text: string
  ) {
    await expect(
      this.row(text)
    ).toBeVisible();
  }
}
```

---

## 20. Generic vs Domain-Specific Components

Generic:

```text
DataTable
Modal
Pagination
Toast
```

Domain-specific:

```text
ExpenseApprovalPanel
UserPermissionTable
ReportFilterPanel
```

Both approaches are useful.

Do not force every component into a generic abstraction.

---

## 21. Base Page

A small base page can contain genuinely shared page behavior:

```ts
export abstract class BasePage {

  constructor(
    protected readonly page: Page
  ) {}

  async goto(
    path: string
  ) {
    await this.page.goto(path);
  }
}
```

---

## 22. Base Page Benefits

A base page can contain:

```text
Navigation
Shared URL handling
Common page utilities
Small shared behavior
```

Keep it small.

---

## 23. Base Page Anti-Pattern

Avoid a huge class containing:

```text
Login
Users
Reports
Expenses
Payments
Admin
100+ methods
```

This creates tight coupling.

---

## 24. Inheritance vs Composition

Inheritance:

```text
DashboardPage
      ↓
   BasePage
```

Composition:

```text
DashboardPage
 ├── Header
 ├── Sidebar
 └── DataTable
```

Use inheritance for genuinely shared page behavior.

Prefer composition for reusable UI components.

---

## 25. Page Object Navigation

```ts
async goto() {
  await this.page.goto(
    '/dashboard'
  );
}
```

Test:

```ts
await dashboardPage.goto();
```

---

## 26. Predictable Navigation

A normal page method should not unexpectedly navigate to unrelated pages.

Keep navigation behavior explicit.

---

## 27. Method Naming

Prefer:

```text
login()
searchUser()
createExpense()
approveExpense()
openReport()
selectDepartment()
```

Avoid:

```text
click1()
doAction()
perform()
handle()
```

Names should describe intent.

---

## 28. Page Object Parameters

Use meaningful parameters:

```ts
async searchUser(
  username: string
) {
  await this.searchInput
    .fill(username);

  await this.searchButton
    .click();
}
```

Avoid excessive optional parameters.

---

## 29. Page Object Return Values

Return values when the test genuinely needs them.

Example:

```ts
async getStatus() {
  return this.status
    .textContent();
}
```

For simple UI validation, locator assertions are usually clearer.

---

## 30. Page Objects and Test Data

Avoid hard-coded datasets inside page objects.

Bad:

```ts
async createUser() {
  await this.name.fill('John');
  await this.email.fill(
    'john@example.com'
  );
}
```

Better:

```ts
async createUser(
  user: UserData
) {
  await this.name.fill(
    user.name
  );

  await this.email.fill(
    user.email
  );
}
```

---

## 31. Page Objects and Fixtures

Fixtures can create page objects automatically:

```ts
type Fixtures = {
  loginPage: LoginPage;
  dashboardPage:
    DashboardPage;
};
```

Tests then receive them through dependency injection.

---

## 32. Custom Fixture Example

```ts
export const test =
  base.extend<Fixtures>({
    loginPage:
      async ({
        page
      }, use) => {

        await use(
          new LoginPage(page)
        );
      },

    dashboardPage:
      async ({
        page
      }, use) => {

        await use(
          new DashboardPage(page)
        );
      }
  });
```

---

## 33. Fixture Dependency Injection

A common dependency graph:

```text
Browser
 ↓
Page
 ↓
Page Object
 ↓
Test
```

Fixtures manage creation and cleanup.

---

## 34. Page Objects with API Clients

Keep UI and backend responsibilities separate.

Prefer:

```text
pages/
  ExpensePage

api/
  ExpenseApi
```

rather than putting all API methods into `ExpensePage`.

---

## 35. UI/API Separation

A test can orchestrate:

```text
API setup
 ↓
UI workflow
 ↓
API verification
```

This combines the strengths of Steps 102 and 105.

---

## 36. Page Objects and SOLID

SOLID principles can improve maintainability:

```text
S → Single Responsibility
O → Open/Closed
L → Liskov Substitution
I → Interface Segregation
D → Dependency Inversion
```

Use them pragmatically.

---

## 37. Single Responsibility Principle

Good:

```text
LoginPage
```

handles login-page behavior.

Bad:

```text
LoginPage
 ├── Login
 ├── User Creation
 ├── Reports
 └── Database Cleanup
```

---

## 38. Open/Closed Principle

Create stable abstractions that can be extended when there is a genuine need.

Do not repeatedly modify a giant shared class for unrelated features.

---

## 39. Liskov Substitution Principle

A child class should remain compatible with the expectations established by its base class.

Avoid page subclasses that violate base-page behavior.

---

## 40. Interface Segregation

Prefer focused contracts.

Avoid:

```ts
interface Everything {
  login(): Promise<void>;
  createUser(): Promise<void>;
  exportReport(): Promise<void>;
  approveExpense(): Promise<void>;
}
```

Use smaller, meaningful interfaces when interfaces are actually needed.

---

## 41. Dependency Inversion

High-level tests should not need to know every low-level implementation detail.

Example:

```text
Test
 ↓
UserApi
 ↓
APIRequestContext
```

instead of embedding raw HTTP implementation throughout tests.

---

## 42. Page Object Dependency Injection

Inject dependencies:

```ts
constructor(
  private readonly page: Page
) {}
```

Do not create a browser or page inside the page object.

---

## 43. Why Not Create Browser Inside Page Objects?

Bad:

```ts
class LoginPage {
  constructor() {
    chromium.launch();
  }
}
```

Problems:

```text
Lifecycle conflicts
Resource leaks
Parallel issues
Harder testing
```

Playwright fixtures should own browser lifecycle.

---

## 44. Page Object Lifecycle

Typical lifecycle:

```text
Fixture
 ↓
Create Page Object
 ↓
Use in Test
 ↓
Fixture Teardown
```

This keeps resources controlled.

---

## 45. Page Object Folder Structure

A scalable structure:

```text
pages/
├── authentication/
│   └── LoginPage.ts
├── dashboard/
│   └── DashboardPage.ts
├── users/
│   └── UserPage.ts
├── expenses/
│   └── ExpensePage.ts
└── reports/
    └── ReportPage.ts

components/
├── Header.ts
├── Sidebar.ts
├── DataTable.ts
├── Modal.ts
└── Toast.ts
```

---

## 46. Domain-Based Organization

Large projects can also organize by domain:

```text
pages/
├── authentication/
├── users/
├── expenses/
├── reports/
├── companies/
└── shared/
```

Choose one consistent structure.

---

## 47. Page Object Naming Convention

Use:

```text
LoginPage.ts
DashboardPage.ts
AdminPage.ts
ExpensePage.ts
UserPage.ts
```

Components:

```text
Header.ts
Sidebar.ts
DataTable.ts
Modal.ts
```

---

## 48. Test Naming Convention

Prefer behavior-focused names:

```text
user-can-create-expense.spec.ts
manager-can-approve-expense.spec.ts
employee-cannot-access-admin.spec.ts
```

Avoid implementation-focused names such as:

```text
click-save-button.spec.ts
```

---

## 49. Encapsulation

Locators can be private:

```ts
private readonly
saveButton =
  this.page.getByRole(
    'button',
    {
      name: 'Save'
    }
  );
```

Expose behavior:

```ts
async save() {
  await this.saveButton
    .click();
}
```

---

## 50. Public vs Private Locators

Public:

```ts
readonly status: Locator;
```

Useful when tests need direct assertions.

Private:

```ts
private readonly
saveButton: Locator;
```

Useful for implementation details.

Use judgment.

---

## 51. Dynamic Locators

Example:

```ts
row(
  username: string
) {
  return this.page
    .getByRole('row')
    .filter({
      hasText:
        username
    });
}
```

Dynamic locator methods are useful for data-driven tests.

---

## 52. Scoped Component Objects

A component can receive a root locator:

```ts
export class Modal {

  constructor(
    private readonly root:
      Locator
  ) {}

  async confirm() {
    await this.root
      .getByRole(
        'button',
        {
          name:
            'Confirm'
        }
      )
      .click();
  }
}
```

This prevents selecting elements outside the component.

---

## 53. Component Usage

```ts
const modal =
  new Modal(
    page.getByRole(
      'dialog'
    )
  );

await modal.confirm();
```

This is a strong composition pattern.

---

## 54. Reusable Modal Component

A modal may expose:

```text
open
close
confirm
cancel
expectVisible
expectHidden
```

Keep the component focused on modal behavior.

---

## 55. Reusable Toast Component

A toast component can provide:

```text
Success
Error
Warning
Info
```

Example:

```ts
async expectSuccess(
  message: string
) {
  await expect(
    this.root
  ).toContainText(
    message
  );
}
```

---

## 56. Page Object and Business Logic

Keep multi-page business orchestration outside individual page objects.

Example:

```text
EmployeePage
ManagerPage
ExpenseApi
```

A workflow can coordinate them.

---

## 57. Workflow Objects

For repeated complex business processes:

```text
ExpenseApprovalWorkflow
```

can coordinate:

```text
Employee
 ↓
Submit
 ↓
Manager
 ↓
Approve
 ↓
Verify
```

Create workflow objects only when repeated complexity justifies them.

---

## 58. Page Objects vs Workflow Objects

### Page Object

Represents:

```text
UI surface
```

### Workflow Object

Represents:

```text
Business process
```

Example:

```text
ExpensePage
```

vs:

```text
ExpenseApprovalWorkflow
```

---

## 59. Service Objects

Backend capabilities belong in API/service classes:

```text
UserApi
ExpenseApi
ReportApi
CompanyApi
```

Architecture:

```text
Test
 ├── Page Object
 ├── Workflow
 └── API Client
```

---

## 60. Avoid God Objects

Avoid:

```text
ApplicationManager
```

with hundreds of unrelated methods.

Prefer focused domain objects.

---

## 61. Framework Dependency Graph

A healthy architecture:

```text
Tests
  ↓
Fixtures
  ↓
Page Objects / API Clients
  ↓
Playwright
  ↓
Application
```

Keep dependencies flowing in one direction.

---

## 62. Avoid Circular Dependencies

Bad:

```text
LoginPage
 ↓
DashboardPage
 ↓
LoginPage
```

Use fixtures or workflows to coordinate objects.

---

## 63. Page Object Factory

A simple factory can create related objects:

```ts
function createPages(
  page: Page
) {
  return {
    login:
      new LoginPage(page),

    dashboard:
      new DashboardPage(page),

    admin:
      new AdminPage(page)
  };
}
```

For Playwright lifecycle management, fixtures are generally more appropriate.

---

## 64. Page Object Configuration

Do not hard-code environment URLs inside page objects.

Use:

```text
playwright.config.ts
Environment variables
Configuration objects
```

for environment-specific settings.

---

## 65. Base URL

Prefer:

```ts
await page.goto(
  '/dashboard'
);
```

with:

```ts
use: {
  baseURL:
    process.env.BASE_URL
}
```

rather than hard-coded environment URLs.

---

## 66. Authentication Separation

Do not make every page object perform login.

Prefer:

```text
Authentication Setup
 ↓
Storage State
 ↓
Authenticated Project
 ↓
Page Objects
```

This keeps business pages independent of authentication mechanics.

---

## 67. Test Isolation

Each test should receive its own:

```text
Page
Context
Page Objects
```

as appropriate.

Avoid mutable global page objects.

---

## 68. Parallel Execution

Avoid:

```text
Global page
Global context
Shared mutable object state
```

Use test-scoped fixtures and isolated data.

---

## 69. Retry Compatibility

Page objects should work with fresh test contexts when a test retries.

Do not retain stale page references globally.

---

## 70. Page Object Logging

Avoid excessive logging inside page objects.

Use framework-level diagnostics where possible.

Never log:

```text
Passwords
Tokens
Cookies
Secrets
Sensitive personal data
```

---

## 71. Page Object Waits

Do not hide arbitrary sleeps inside page objects.

Bad:

```ts
await this.page
  .waitForTimeout(
    3000
  );
```

Better:

```ts
await expect(
  this.status
).toHaveText(
  'Approved'
);
```

or synchronize with a meaningful event.

---

## 72. Network Synchronization

A page action can coordinate with a known request when necessary:

```ts
async save() {

  const responsePromise =
    this.page.waitForResponse(
      response =>
        response.url()
          .includes(
            '/api/expenses'
          )
    );

  await this.saveButton
    .click();

  return responsePromise;
}
```

Do this only when the network event is meaningful to the action.

---

## 73. Type Safety

Use TypeScript models:

```ts
interface ExpenseData {
  amount: number;
  category: string;
  description: string;
}
```

Then:

```ts
async createExpense(
  data: ExpenseData
) {
  // ...
}
```

---

## 74. Request and UI Models

Keep request/response/test models separate when their structures differ:

```text
CreateUserRequest
UserResponse
UpdateUserRequest
UserTestData
```

This prevents accidental coupling.

---

## 75. Composition Over Deep Inheritance

Prefer:

```text
Page
 ├── Header
 ├── Sidebar
 └── Table
```

over:

```text
BasePage
 ↓
AuthenticatedPage
 ↓
AdminPage
 ↓
SpecialAdminPage
 ↓
RegionalAdminPage
```

Deep inheritance usually increases maintenance cost.

---

## 76. Framework Layering

A mature framework may contain:

```text
tests/
fixtures/
pages/
components/
api/
workflows/
utils/
test-data/
config/
```

Each layer should have a clear responsibility.

---

## 77. Example Enterprise Structure

```text
playwright/
├── tests/
│   ├── authentication/
│   ├── users/
│   ├── expenses/
│   └── reports/
│
├── pages/
│   ├── authentication/
│   ├── users/
│   ├── expenses/
│   └── reports/
│
├── components/
│   ├── Header.ts
│   ├── Sidebar.ts
│   ├── DataTable.ts
│   ├── Modal.ts
│   └── Toast.ts
│
├── api/
│   ├── AuthApi.ts
│   ├── UserApi.ts
│   ├── ExpenseApi.ts
│   └── ReportApi.ts
│
├── workflows/
│   ├── LoginWorkflow.ts
│   └── ExpenseApprovalWorkflow.ts
│
├── fixtures/
│   └── baseTest.ts
│
├── test-data/
│   ├── factories/
│   └── static/
│
└── utils/
```

---

## 78. Test Layer

Tests should answer:

```text
What is being verified?
```

Example:

```ts
test(
  'manager can approve expense',
  async ({
    expensePage
  }) => {

    await expensePage
      .openExpense(
        expenseId
      );

    await expensePage
      .approve();

    await expect(
      expensePage.status
    ).toHaveText(
      'Approved'
    );
  }
);
```

---

## 79. Page Layer

Page objects answer:

```text
How does the UI perform the action?
```

Example:

```ts
async approve() {
  await this.approveButton
    .click();
}
```

---

## 80. Component Layer

Components answer:

```text
How does this reusable UI component work?
```

Examples:

```text
Header
Modal
DataTable
Pagination
Toast
```

---

## 81. API Layer

API clients answer:

```text
How does the backend capability work?
```

Example:

```ts
async getExpense(
  id: number
) {
  return this.request.get(
    `/api/expenses/${id}`
  );
}
```

---

## 82. Workflow Layer

Workflow objects answer:

```text
How does a business process span multiple objects?
```

Example:

```text
Create data
 ↓
Open page
 ↓
Perform action
 ↓
Verify API state
```

---

## 83. Fixture Layer

Fixtures answer:

```text
How are dependencies created and managed?
```

Examples:

```text
Page Objects
API Clients
Authenticated Contexts
Test Data
Network Mocks
```

---

## 84. Dependency Injection Architecture

```text
Playwright Test
      │
      ├── Page
      │    ↓
      │  Page Object
      │
      ├── Request
      │    ↓
      │  API Client
      │
      └── Fixtures
           ↓
      Shared Dependencies
```

---

## 85. Page Object Lifecycle with Fixtures

```text
Test Starts
    ↓
Fixture Resolves
    ↓
Page Created
    ↓
Page Object Created
    ↓
Test Executes
    ↓
Fixture Teardown
    ↓
Context Closed
```

Understanding lifecycle prevents resource leaks.

---

## 86. Avoid Shared Page Objects

Do not create:

```ts
export const
dashboardPage =
  new DashboardPage(
    globalPage
  );
```

Global instances can cause:

```text
State leakage
Parallel conflicts
Stale references
Unpredictable tests
```

---

## 87. Test-Scoped Page Objects

Prefer:

```ts
test(
  'dashboard',
  async ({
    dashboardPage
  }) => {
    // test
  }
);
```

The fixture controls lifecycle.

---

## 88. Framework Governance

Enterprise teams should define standards for:

```text
Naming
Locator strategy
Page object structure
Fixture usage
API clients
Test data
Assertions
Logging
Timeouts
Retries
Code review
```

---

## 89. POM Code Review Questions

Ask:

```text
Is the locator stable?
Is the method name meaningful?
Does the class have one responsibility?
Is business logic in the correct layer?
Is test data externalized?
Is authentication separated?
Are waits condition-based?
Is the object test-isolated?
```

---

## 90. Common POM Anti-Patterns

```text
One giant BasePage
One class for entire application
Hard-coded data
Global page objects
Deep inheritance
Arbitrary waits
Business logic everywhere
API calls inside every page object
Fragile selectors
Browser creation inside page objects
```

---

## 91. When POM Is Not Necessary

For a tiny one-off test:

```ts
test(
  'title',
  async ({
    page
  }) => {

    await page.goto('/');

    await expect(
      page
    ).toHaveTitle(
      /Home/
    );
  }
);
```

Do not create multiple abstraction layers for a trivial test.

---

## 92. Enterprise POM Decision Rule

Create an abstraction when it provides meaningful:

```text
Reuse
Readability
Encapsulation
Maintainability
Test isolation
Domain clarity
```

Do not abstract code only because it can be abstracted.

---

## 93. Refactoring Strategy

When converting existing tests:

### Step 1

Identify repeated locators.

### Step 2

Identify repeated actions.

### Step 3

Create page objects.

### Step 4

Extract reusable components.

### Step 5

Move setup into fixtures.

### Step 6

Move backend operations into API clients.

### Step 7

Create workflow objects only when repeated business workflows justify them.

---

## 94. Before POM

```ts
await page
  .getByLabel(
    'Username'
  )
  .fill(username);

await page
  .getByLabel(
    'Password'
  )
  .fill(password);

await page
  .getByRole(
    'button',
    {
      name: 'Login'
    }
  )
  .click();
```

---

## 95. After POM

```ts
await loginPage.login(
  username,
  password
);
```

The test now focuses on intent.

---

## 96. Component Extraction

Before:

```ts
await page
  .getByRole(
    'dialog'
  )
  .getByRole(
    'button',
    {
      name:
        'Confirm'
    }
  )
  .click();
```

After:

```ts
await confirmModal
  .confirm();
```

The implementation is centralized.

---

## 97. Hands-On Exercise 1 — LoginPage

Create:

```text
pages/authentication/LoginPage.ts
```

with:

```text
username
password
login button
error message
login()
expectLoginError()
```

---

## 98. Hands-On Exercise 2 — DashboardPage

Create:

```text
DashboardPage.ts
```

with:

```text
heading
sidebar
profile
expectLoaded()
```

---

## 99. Hands-On Exercise 3 — Header Component

Create:

```text
components/Header.ts
```

with:

```text
profile menu
logout
notifications
```

---

## 100. Hands-On Exercise 4 — DataTable

Create:

```text
DataTable
```

supporting:

```text
row()
expectRow()
expectEmpty()
```

---

## 101. Hands-On Exercise 5 — Modal

Create:

```text
Modal
```

supporting:

```text
confirm()
cancel()
close()
expectVisible()
```

---

## 102. Hands-On Exercise 6 — Fixtures

Create fixtures for:

```text
loginPage
dashboardPage
adminPage
```

and inject them into tests.

---

## 103. Hands-On Exercise 7 — API Client

Create:

```text
UserApi
ExpenseApi
```

and keep HTTP implementation outside page objects.

---

## 104. Hands-On Exercise 8 — Workflow

Create:

```text
ExpenseApprovalWorkflow
```

that coordinates:

```text
Employee
 ↓
Submit
 ↓
Manager
 ↓
Approve
 ↓
Verify
```

---

## 105. Hands-On Exercise 9 — Refactoring

Take an existing test containing:

```text
Repeated selectors
waitForTimeout()
Hard-coded data
Direct API calls
```

Refactor it into:

```text
Page Object
Fixture
API Client
Factory
Condition-based synchronization
```

---

## 106. Enterprise Challenge

Build an enterprise Playwright architecture containing:

```text
Tests
Fixtures
Page Objects
Component Objects
API Clients
Workflow Objects
Test Data Factories
Utilities
Configuration
```

Requirements:

1. Create a clean directory structure.
2. Implement LoginPage.
3. Implement DashboardPage.
4. Implement at least one domain page.
5. Implement Header component.
6. Implement DataTable component.
7. Implement Modal component.
8. Create custom fixtures.
9. Inject page objects through fixtures.
10. Create API clients.
11. Keep API and UI responsibilities separate.
12. Create typed test data.
13. Use stable locators.
14. Avoid global page objects.
15. Avoid arbitrary waits.
16. Avoid deep inheritance.
17. Apply SOLID principles where useful.
18. Support parallel execution.
19. Keep objects test-scoped.
20. Document framework conventions.

---

## 107. Enterprise Architecture

```text
                         Playwright
                             │
                           Tests
                             │
                    ┌────────┴────────┐
                    │                 │
                 Fixtures         Workflows
                    │                 │
          ┌─────────┼─────────┐       │
          │         │         │       │
        Pages   Components    APIs     │
          │         │         │       │
          └─────────┼─────────┘       │
                    │                 │
                Utilities       Test Data
                    │                 │
                    └────────┬────────┘
                             │
                       Application
```

---

## 108. Final Checklist

```text
[ ] Page Object Model understood
[ ] Page object responsibilities understood
[ ] Locator encapsulation understood
[ ] Semantic locators understood
[ ] Test IDs understood
[ ] Fragile locator problems understood
[ ] Page actions understood
[ ] Page assertions understood
[ ] Component objects understood
[ ] Component composition understood
[ ] Base page understood
[ ] Inheritance vs composition understood
[ ] Dynamic locators understood
[ ] Scoped component locators understood
[ ] DataTable component understood
[ ] Modal component understood
[ ] Header component understood
[ ] Page object fixtures understood
[ ] Dependency injection understood
[ ] Authenticated page objects understood
[ ] API/page separation understood
[ ] Workflow objects understood
[ ] Service/API objects understood
[ ] SOLID principles understood
[ ] Single Responsibility understood
[ ] Open/Closed understood
[ ] Liskov Substitution understood
[ ] Interface Segregation understood
[ ] Dependency Inversion understood
[ ] Test-scoped objects understood
[ ] Parallel execution considerations understood
[ ] Page object lifecycle understood
[ ] POM anti-patterns understood
[ ] Enterprise architecture understood
[ ] Framework governance understood
```

---

## 109. Key Takeaways

- Page Object Model separates test intent from UI implementation.
- Page objects should represent meaningful UI behavior.
- Components are useful for reusable UI areas.
- Composition is often better than deep inheritance.
- Keep page objects focused and small.
- Prefer stable semantic locators.
- Use test IDs when appropriate.
- Keep business scenarios visible in tests.
- Keep reusable UI mechanics inside page/component objects.
- Keep API behavior in API clients.
- Use fixtures for dependency injection and lifecycle management.
- Never create browser instances inside page objects.
- Avoid global page-object instances.
- Avoid arbitrary waits inside page objects.
- Use typed data models and factories.
- Workflow objects can coordinate complex business processes when justified.
- SOLID principles help prevent framework complexity.
- Do not over-engineer tiny tests.
- Enterprise architecture should improve maintainability rather than add unnecessary abstraction.
- A well-designed POM framework makes a large Playwright suite easier to read, scale, debug, and maintain.

---

## 110. Next Step

**Step 106 — Mastering Playwright Test Fixtures at Enterprise Scale: Built-in Fixtures, Custom Fixtures, Fixture Composition, Worker vs Test Scope, Automatic Fixtures, Dependency Injection, Authentication Fixtures, API Fixtures, Data Fixtures, Teardown, Lifecycle & Scalable Fixture Architecture**

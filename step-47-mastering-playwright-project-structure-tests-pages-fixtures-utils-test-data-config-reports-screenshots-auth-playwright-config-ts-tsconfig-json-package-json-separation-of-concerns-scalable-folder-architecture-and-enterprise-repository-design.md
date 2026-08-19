# Playwright with TypeScript Mastery Bootcamp
## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 21 — Playwright Project Structure and Enterprise Repository Architecture

# Step 47 — Mastering Playwright Project Structure: `tests`, `pages`, `fixtures`, `utils`, `test-data`, `config`, `reports`, `screenshots`, `auth`, `playwright.config.ts`, `tsconfig.json`, `package.json`, Separation of Concerns, Scalable Folder Architecture, and Enterprise Repository Design

---

# Objective

In this lesson, you will learn:

- What project structure means
- Why project structure matters
- Why automation frameworks need separation of concerns
- The purpose of the `tests` directory
- The purpose of the `pages` directory
- The purpose of `fixtures`
- The purpose of `utils`
- The purpose of `test-data`
- The purpose of `config`
- The purpose of `auth`
- The purpose of `reports`
- The purpose of `screenshots`
- The purpose of `playwright.config.ts`
- The purpose of `tsconfig.json`
- The purpose of `package.json`
- Folder-based organization
- Domain-based organization
- Layered architecture
- Enterprise repository design
- Shared libraries
- Ownership boundaries
- Dependency direction
- Anti-patterns
- Framework scalability

By the end of this lesson, you will understand how to organize a Playwright repository so that it remains maintainable when it grows from:

```text
10 Tests
```

to:

```text
10,000+ Tests
```

---

# Before We Start

Imagine a large hospital.

A hospital does not put everything into one room.

Instead:

```text
Emergency

↓

Pharmacy

↓

Laboratory

↓

Radiology

↓

Reception

↓

Operation Theatre

↓

Medical Records
```

Each area has a specific responsibility.

Why?

Because organization makes the system:

- Easier to operate
- Easier to maintain
- Easier to scale
- Easier to understand

A Playwright framework needs the same discipline.

---

# The Problem

Imagine a beginner creates:

```text
tests.ts
```

and puts everything inside it:

```text
Locators

↓

Login Logic

↓

Test Data

↓

Assertions

↓

API Calls

↓

Database Logic

↓

Screenshots

↓

Utility Functions
```

It may work initially.

But after hundreds of tests:

```text
One Huge File

↓

Difficult Navigation

↓

Duplicate Code

↓

Hard Maintenance

↓

High Technical Debt
```

This is why architecture matters.

---

# What is Project Structure?

Project Structure is the organized arrangement of files and directories that defines where different responsibilities live within an automation framework.

Conceptually:

```text
Project

↓

Folders

↓

Responsibilities

↓

Dependencies

↓

Execution
```

A good structure makes the framework understandable before someone even reads the code.

---

# Why Project Structure Matters

Good structure provides:

- Maintainability
- Discoverability
- Reusability
- Separation of concerns
- Scalability
- Easier onboarding
- Better code reviews
- Easier debugging
- Reduced duplication

Project structure is therefore an architectural decision.

---

# The Core Principle

The most important rule is:

> **One responsibility should have one logical home.**

For example:

```text
Test Scenario

↓

tests/
```

UI interaction abstraction:

```text
Page Object

↓

pages/
```

Reusable setup:

```text
fixtures/
```

Reusable helper:

```text
utils/
```

Test data:

```text
test-data/
```

This prevents responsibility from spreading randomly throughout the repository.

---

# A Typical Playwright Framework

A scalable framework might conceptually look like:

```text
playwright-project/

├── tests/
├── pages/
├── components/
├── fixtures/
├── utils/
├── test-data/
├── config/
├── auth/
├── reports/
├── screenshots/
├── downloads/
├── api/
├── playwright.config.ts
├── tsconfig.json
├── package.json
└── lockfile
```

Not every project needs every directory.

The structure should evolve according to real requirements.

---

# The `tests` Directory

The `tests` directory contains test specifications.

Its responsibility is:

```text
What Should Be Tested?
```

Examples include:

```text
Login

↓

Checkout

↓

Order Creation

↓

User Management

↓

Reports
```

The test should describe business behavior rather than implementation details.

---

# What Belongs in `tests`?

Typically:

- Test scenarios
- Test descriptions
- Test steps
- Assertions
- Test-specific orchestration

A test should answer:

```text
What behavior are we validating?
```

---

# What Should NOT Belong in `tests`?

Avoid putting large amounts of:

- Locator definitions
- Generic helper logic
- Browser initialization
- Authentication infrastructure
- Reusable data generation
- Common API clients

These belong in appropriate abstraction layers.

---

# Test Responsibility

Think of a test as:

```text
Business Scenario

↓

Arrange

↓

Act

↓

Assert
```

It should not become a dumping ground for framework infrastructure.

---

# The `pages` Directory

The `pages` directory commonly contains Page Object classes.

Its responsibility is:

```text
How Do We Interact With This Page?
```

For example:

```text
LoginPage

↓

Login UI

```

```text
DashboardPage

↓

Dashboard UI
```

---

# Why Page Objects Exist

Suppose 100 tests use:

```text
Username Field

Password Field

Login Button
```

If every test defines those locators independently:

```text
100 Definitions
```

A Page Object centralizes the interaction model.

Conceptually:

```text
Login Page

↓

One Page Object

↓

100 Tests Reuse It
```

---

# What Belongs in a Page Object?

Typically:

- Locators
- Page-specific actions
- Page-specific state queries
- Page-level interaction behavior

The Page Object represents the UI interface.

---

# What Should Not Belong in a Page Object?

Avoid putting:

- Large business workflows
- Test assertions unrelated to page behavior
- Environment-specific configuration
- Test data generation
- CI logic
- Reporting logic

A Page Object should not become a second test framework.

---

# The `components` Directory

Modern applications often contain reusable UI components.

Examples:

```text
Header

↓

Sidebar

↓

Navigation Menu

↓

Date Picker

↓

Modal

↓

Data Table
```

These components may appear on many pages.

A Component Object Model can represent them.

---

# Page vs Component

A Page represents:

```text
Entire Page
```

A Component represents:

```text
Reusable Section
```

For example:

```text
DashboardPage

├── HeaderComponent

├── SidebarComponent

└── NotificationComponent
```

This improves reuse.

---

# The `fixtures` Directory

Fixtures provide reusable test setup and dependencies.

Examples:

```text
Authenticated Page

↓

API Client

↓

Database Connection

↓

Test Data

↓

Page Objects
```

Fixtures answer:

```text
What does the test need before it starts?
```

---

# Why Fixtures Matter

Without fixtures:

```text
Test 1

↓

Setup Login

↓

Create Browser

↓

Create Page

↓

Create Data
```

Test 2 repeats everything.

Fixtures centralize reusable lifecycle behavior.

---

# Fixture Responsibility

Fixtures should provide:

```text
Reusable Test Infrastructure
```

Examples:

- Authenticated contexts
- Page objects
- API clients
- Test data
- Service clients

---

# The `utils` Directory

Utilities contain reusable generic functionality.

Examples:

```text
Date Utility

↓

Random Data Utility

↓

File Utility

↓

String Utility

↓

Environment Utility
```

The key characteristic is:

```text
Generic Reusability
```

---

# Utility vs Page Object

A Page Object is:

```text
Application-Specific
```

A Utility is usually:

```text
Application-Independent
```

For example:

```text
LoginPage

↓

Specific To Application
```

while:

```text
DateFormatter

↓

Potentially Generic
```

---

# Utility Anti-Pattern

Do not create:

```text
utils.ts
```

and put everything inside it.

That becomes another dumping ground.

Prefer meaningful boundaries:

```text
utils/

├── date.util.ts

├── file.util.ts

├── string.util.ts

└── random.util.ts
```

---

# The `test-data` Directory

Test data represents the inputs required to execute tests.

Examples:

```text
Users

↓

Products

↓

Orders

↓

Customers

↓

Payment Data
```

Test data may be stored as:

- JSON
- CSV
- TypeScript objects
- Generated data
- Database fixtures
- API-created entities

---

# Test Data vs Test Logic

This separation is important.

Test logic says:

```text
Create User

↓

Verify User
```

Test data says:

```text
User Name

↓

Email

↓

Role
```

Separating them makes tests easier to maintain.

---

# Static vs Dynamic Test Data

### Static Data

Predefined:

```text
admin-user.json
```

### Dynamic Data

Generated during execution:

```text
Unique Customer

↓

Created At Runtime
```

Enterprise frameworks often combine both approaches.

---

# The `config` Directory

The `config` directory can contain configuration-related modules.

Examples:

```text
Environment Configuration

↓

Application Configuration

↓

Feature Configuration

↓

Framework Constants
```

The exact structure depends on project complexity.

---

# Configuration Responsibility

Configuration should answer:

```text
How Should The Framework Behave?
```

It should not contain:

```text
Business Test Logic
```

---

# The `auth` Directory

Authentication artifacts may be stored or generated here.

For example:

```text
auth/

├── admin.json

├── user.json

└── agent.json
```

These may represent Playwright Storage State files.

---

# Why Authentication State Needs Organization

Enterprise applications often have multiple personas:

```text
Admin

↓

User

↓

Agent

↓

Manager

↓

Auditor
```

Authentication artifacts should be managed systematically.

---

# Important Security Warning

Authentication state can contain sensitive session information.

Therefore:

```text
Auth State

↓

Treat As Sensitive
```

Do not blindly commit authentication state files to source control.

---

# The `reports` Directory

The `reports` directory may contain generated test reports.

Examples:

```text
HTML

↓

JSON

↓

JUnit

↓

Allure
```

Reports are outputs,

not source code.

---

# Generated Artifacts vs Source Code

A useful distinction is:

```text
Source

↓

Tests

↓

Framework

```

versus:

```text
Generated

↓

Reports

↓

Screenshots

↓

Videos

↓

Traces
```

Generated content should generally not clutter the source tree unnecessarily.

---

# The `screenshots` Directory

Screenshots are diagnostic artifacts.

They may be captured:

```text
On Failure

↓

For Visual Testing

↓

For Debugging
```

Large organizations often store them as CI artifacts rather than permanently committing them.

---

# The `downloads` Directory

File-download tests may generate:

```text
PDF

↓

CSV

↓

Excel

↓

Images
```

Temporary downloads should usually be isolated from source code.

---

# The `api` Directory

As the framework grows,

API testing may become a separate abstraction layer.

Conceptually:

```text
api/

├── clients/

├── models/

├── requests/

└── services/
```

This supports API testing and UI/API hybrid workflows.

---

# Why API Abstraction Matters

A UI test might need:

```text
Create Customer

↓

API

↓

Open UI

↓

Verify Customer
```

API setup can be significantly faster than performing every setup step through the UI.

---

# The `playwright.config.ts` File

This file controls framework-level execution.

It can define:

```text
Projects

↓

Browsers

↓

Timeouts

↓

Retries

↓

Workers

↓

Reporters

↓

Artifacts

↓

Base URL
```

It belongs at the project root because it controls the entire test framework.

---

# The `tsconfig.json` File

This file controls TypeScript behavior.

It defines concepts such as:

```text
Compiler Settings

↓

Module Rules

↓

Type Checking

↓

Source Inclusion
```

It belongs at the root because it generally governs the TypeScript project.

---

# The `package.json` File

`package.json` defines project-level package metadata and scripts.

Conceptually:

```text
Project

↓

Dependencies

↓

Scripts

↓

Package Metadata
```

It is foundational to the Node.js project.

---

# Root-Level Files

A clean repository may therefore have:

```text
playwright-project/

├── package.json
├── playwright.config.ts
├── tsconfig.json
└── lockfile
```

These files describe the framework itself.

---

# Separation of Concerns

One of the most important architectural principles is:

```text
Test Logic

≠

Page Logic

≠

Fixture Logic

≠

Configuration

≠

Test Data

≠

Utilities
```

Each layer has a different responsibility.

---

# Layered Architecture

A conceptual architecture can look like:

```text
Tests

↓

Fixtures

↓

Page Objects / Components

↓

Playwright

↓

Browser
```

Supporting layers provide:

```text
Configuration

↓

Test Data

↓

Utilities

↓

API Services
```

---

# Dependency Direction

A healthy framework generally follows:

```text
Tests

↓

Framework Abstractions

↓

Playwright APIs
```

rather than:

```text
Playwright Internals

↓

Randomly Calling Tests
```

Tests should depend on stable abstractions.

---

# Architecture Diagram

```text
                         Tests
                           │
                           ▼
                    Test Orchestration
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         Fixtures      Page Objects   API Clients
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                      Playwright
                           │
                           ▼
                    BrowserContext
                           │
                           ▼
                         Page
                           │
                           ▼
                       Browser
```

Supporting systems:

```text
Configuration → Test Runner
Test Data     → Tests / Fixtures
Utilities     → Multiple Layers
Reports       ← Test Runner
Artifacts     ← Test Runner
```

---

# Domain-Based Organization

For a large application,

organization by business domain can be more scalable.

For example:

```text
tests/

├── authentication/

├── customers/

├── orders/

├── payments/

└── reports/
```

This helps teams locate related scenarios quickly.

---

# Layer-Based Organization

Another strategy is:

```text
pages/

fixtures/

utils/

tests/
```

This emphasizes technical layers.

Both approaches can work.

---

# Hybrid Organization

Large enterprises often combine both.

For example:

```text
modules/

├── authentication/
│   ├── pages/
│   ├── components/
│   ├── fixtures/
│   └── tests/
│
├── payments/
│   ├── pages/
│   ├── components/
│   ├── fixtures/
│   └── tests/
│
└── orders/
    ├── pages/
    ├── components/
    ├── fixtures/
    └── tests/
```

This creates strong domain ownership.

---

# Centralized vs Domain-Oriented Architecture

### Centralized

```text
pages/
tests/
fixtures/
```

Advantages:

- Simple
- Easy for small projects
- Familiar

Disadvantages:

- Can become crowded
- Harder domain ownership
- Difficult at very large scale

---

### Domain-Oriented

```text
payments/

orders/

customers/
```

Advantages:

- Strong ownership
- Better scalability
- Easier domain navigation

Disadvantages:

- More architectural complexity
- Shared components require careful design

---

# Enterprise Recommendation

Start simple.

Then evolve.

Do not build a 500-folder architecture for 20 tests.

A practical progression is:

```text
Small Project

↓

Simple Layered Structure

↓

Growing Project

↓

Domain Organization

↓

Enterprise Platform

↓

Shared Libraries
```

Architecture should evolve with complexity.

---

# Framework Boundaries

A strong framework defines boundaries.

For example:

```text
Test

↓

May call Page Object
```

Page Object:

```text
May use Playwright
```

Fixture:

```text
May create Page Objects
```

Configuration:

```text
Should not execute business workflows
```

This prevents architectural confusion.

---

# Shared Libraries

At enterprise scale,

common capabilities may move into shared packages.

For example:

```text
Company Automation Library

↓

Authentication

↓

Logging

↓

Reporting

↓

Common Components

↓

Utilities
```

Multiple projects can consume the shared library.

---

# Enterprise Repository Model

A large organization may eventually have:

```text
Enterprise Automation Platform

├── shared-core/
├── shared-ui-components/
├── shared-api-client/
├── web-project-a/
├── web-project-b/
├── mobile-project/
└── reporting-platform/
```

This is significantly more scalable than copying utilities between projects.

---

# Code Ownership

Each directory or module should have clear ownership.

For example:

```text
Payments

↓

Payments QA Team
```

```text
Shared Framework

↓

Automation Platform Team
```

Ownership reduces ambiguity.

---

# Code Review Benefits

Good structure makes review easier.

A reviewer can ask:

```text
Is this test logic?

↓

Should it be in tests?
```

or:

```text
Is this reusable UI logic?

↓

Should it be in a Page Object?
```

Architecture provides review rules.

---

# Onboarding Benefits

A new engineer should be able to understand:

```text
Where Are Tests?

↓

Where Are Pages?

↓

Where Are Fixtures?

↓

Where Is Test Data?

↓

Where Is Configuration?
```

within minutes.

Good project structure reduces onboarding time.

---

# Common Beginner Mistakes

Avoid:

- One giant utility file
- One giant Page Object
- Putting all tests in one directory without domain organization
- Storing generated reports in source-control
- Committing screenshots and videos
- Mixing secrets with test data
- Putting environment logic inside tests
- Duplicating common framework code

---

# Anti-Pattern: God Page Object

A Page Object becomes problematic when it contains:

```text
Login

↓

Orders

↓

Payments

↓

Reports

↓

Users

↓

Notifications
```

One class controls half the application.

This is called a:

```text
God Object
```

Break responsibilities into meaningful pages and components.

---

# Anti-Pattern: God Fixture

Another common problem:

```text
Global Fixture

↓

Login

↓

Database

↓

API

↓

Files

↓

All Page Objects

↓

All Test Data
```

Every test receives everything.

This increases:

- Startup cost
- Coupling
- Debugging difficulty

Fixtures should remain focused.

---

# Anti-Pattern: Utility Dump

Avoid:

```text
utils.ts
```

containing:

```text
Login

↓

Database

↓

Date

↓

API

↓

Screenshots

↓

Random Data
```

Utilities should have cohesive responsibilities.

---

# Anti-Pattern: Environment Logic in Tests

Avoid:

```text
if QA

↓

do this

if UAT

↓

do that

if staging

↓

do something else
```

This creates environment-specific business logic.

Move environment behavior into configuration.

---

# Anti-Pattern: Copy-Paste Frameworks

Avoid:

```text
Project A

↓

Copy Framework

Project B

↓

Copy Framework

Project C

↓

Copy Framework
```

Eventually:

```text
Fix Applied To A

↓

B Misses Fix

↓

C Misses Fix
```

Shared libraries solve this problem.

---

# Workflow Diagram

```text
Business Requirement

↓

Test Scenario

↓

Test Layer

↓

Fixture / Page / Component / API

↓

Playwright

↓

Browser

↓

Result

↓

Report
```

Each layer has a clear responsibility.

---

# Enterprise Architecture Diagram

```text
                         Enterprise Automation Platform
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
        Shared Libraries        Application Projects      Reporting
              │                       │                       │
       ┌──────┼──────┐          ┌─────┼─────┐                │
       ▼      ▼      ▼          ▼     ▼     ▼                ▼
     Core    API     UI       App A  App B  App C        Analytics
      │       │      │
      └───────┼──────┘
              ▼
        Playwright Layer
              │
              ▼
          Test Runner
              │
              ▼
            Browser
```

This is the direction in which a mature automation ecosystem can evolve.

---

# Enterprise Perspective

Fortune 500-scale organizations rarely treat automation as:

```text
A Folder Full Of Tests
```

They treat it as:

```text
Software Platform
```

That platform requires:

- Architecture
- Ownership
- Versioning
- Security
- Governance
- CI/CD
- Documentation
- Dependency management
- Maintenance strategy

Project structure is the physical representation of that architecture.

---

# Best Practices

Professional recommendations:

1. Keep tests focused on behavior.
2. Keep Page Objects focused on UI interaction.
3. Keep Components focused on reusable UI regions.
4. Keep Fixtures focused on lifecycle and dependencies.
5. Keep Utilities generic.
6. Keep test data separate.
7. Keep configuration centralized.
8. Keep secrets outside source control.
9. Keep generated artifacts out of source code.
10. Organize large applications by business domain.
11. Establish ownership.
12. Avoid unnecessary abstraction.
13. Evolve architecture gradually.
14. Prefer reuse over copy-paste.
15. Keep dependency direction clear.

---

# Professional Tips

An experienced architect does not ask:

```text
"What folders should I create?"
```

The better question is:

```text
"What responsibilities exist?"
```

Then:

```text
Which responsibility belongs where?
```

Architecture should emerge from responsibility boundaries.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What should be stored in the `tests` directory?

**Answer:**

Test specifications and business scenarios should primarily live there. Tests should describe what behavior is being validated rather than contain large amounts of reusable framework infrastructure.

---

### Mid-Level Question

**Q:** What is the purpose of Page Objects?

**Answer:**

Page Objects encapsulate page-specific locators and interactions so that test cases remain readable and UI implementation changes can be handled in a centralized abstraction.

---

### Senior-Level Question

**Q:** What is the difference between a Page Object and a Fixture?

**Answer:**

A Page Object models application UI behavior, while a Fixture provides reusable test setup, dependencies, lifecycle management, or test resources.

---

### Lead-Level Question

**Q:** How would you structure a large Playwright repository?

**Answer:**

I would separate tests, page/component abstractions, fixtures, utilities, configuration, test data, API clients, and generated artifacts. As the application grows, I would introduce domain-oriented organization and shared libraries where appropriate.

---

### Architect-Level Question

**Q:** How would you prevent an enterprise Playwright framework from becoming tightly coupled?

**Answer:**

I would establish clear responsibility boundaries, dependency direction, reusable abstractions, domain ownership, shared libraries, configuration separation, and architectural review rules. I would also prevent Page Objects, fixtures, and utilities from becoming generic dumping grounds.

---

# Knowledge Check

Before moving to the next lesson, answer these questions:

1. What is project structure?
2. Why does project structure matter?
3. What belongs in `tests`?
4. What belongs in `pages`?
5. What is a Component Object?
6. What belongs in `fixtures`?
7. What belongs in `utils`?
8. What belongs in `test-data`?
9. What is the purpose of `config`?
10. Why should authentication state be handled carefully?
11. Why should generated reports generally be separated from source code?
12. What is separation of concerns?
13. What is layered architecture?
14. What is domain-based organization?
15. What is hybrid organization?
16. What is a God Page Object?
17. What is a God Fixture?
18. Why are shared libraries useful?
19. Why is dependency direction important?
20. How should a Playwright repository evolve as it grows?

---

# Step Summary

In this lesson, you learned how to design the physical architecture of a Playwright repository.

You learned the purpose of:

```text
tests/

pages/

components/

fixtures/

utils/

test-data/

config/

auth/

api/

reports/

screenshots/

downloads/

playwright.config.ts

tsconfig.json

package.json
```

You also learned:

- Separation of concerns
- Layered architecture
- Domain-based organization
- Hybrid architecture
- Dependency direction
- Shared libraries
- Ownership
- Framework boundaries
- Enterprise repository design
- Common architecture anti-patterns

The most important principle is:

> **A framework should be organized around responsibilities, not around random collections of files.**

A strong repository makes it obvious:

```text
Where Tests Live

↓

Where UI Behavior Lives

↓

Where Setup Lives

↓

Where Data Lives

↓

Where Configuration Lives

↓

Where Shared Infrastructure Lives
```

That clarity becomes increasingly valuable as the framework grows.

---

# Progress Milestone

✅ You have completed **Step 47** of approximately **230** planned learning steps.

**What you've mastered:**

- Playwright Repository Structure
- `tests`
- `pages`
- `components`
- `fixtures`
- `utils`
- `test-data`
- `config`
- `auth`
- `api`
- `reports`
- `screenshots`
- `downloads`
- `playwright.config.ts`
- `tsconfig.json`
- `package.json`
- Separation of Concerns
- Layered Architecture
- Domain Architecture
- Hybrid Architecture
- Shared Libraries
- Enterprise Repository Design
- Architecture Anti-Patterns

**Coming next — Step 48:**

**Mastering the Browser, BrowserContext, and Page Architecture — Browser Lifecycle, BrowserContext Isolation, Pages, Context Creation, Context Options, Resource Ownership, Lifecycle Management, Multi-Context Testing, Multi-Page Testing, and Enterprise Session Architecture.**

The next lesson begins one of the most important technical sections of the entire Bootcamp: understanding exactly how Playwright models browsers, sessions, contexts, and pages—and why this architecture is fundamental to parallel execution, authentication isolation, test reliability, and enterprise scalability.

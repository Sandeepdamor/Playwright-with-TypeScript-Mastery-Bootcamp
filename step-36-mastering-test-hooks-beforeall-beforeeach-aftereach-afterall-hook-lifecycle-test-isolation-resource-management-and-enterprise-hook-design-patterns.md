# Playwright with TypeScript Mastery Bootcamp
## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 14 — Playwright Test Runner Fundamentals

# Step 36 — Mastering Test Hooks: `beforeAll`, `beforeEach`, `afterEach`, `afterAll`, Hook Lifecycle, Test Isolation, Resource Management, and Enterprise Hook Design Patterns

---

# Objective

In this lesson, you will learn:

- What Test Hooks are
- Why Hooks exist
- Hook execution lifecycle
- `beforeAll`
- `beforeEach`
- `afterEach`
- `afterAll`
- Hook execution order
- Resource management
- Test isolation
- Enterprise hook design
- Best practices

By the end of this lesson, you will understand how Playwright Test Hooks organize test execution, eliminate duplicate code, and provide a structured lifecycle for enterprise automation frameworks.

---

# Before We Start

Imagine a hotel.

Every guest follows a similar process.

```
Guest Arrives

↓

Reception

↓

Room Assignment

↓

Stay

↓

Checkout

↓

Room Cleaning
```

Notice something.

Some activities happen:

- Once for every guest
- Once before every stay
- Once after every stay
- Once when the hotel closes

Automation follows a similar pattern.

---

# The Problem

Imagine writing ten login tests.

Every test begins with:

```
Open Browser

↓

Navigate Login

↓

Login

↓

Execute Test
```

And every test ends with:

```
Logout

↓

Close Resources
```

Without Hooks,

every test repeats the same setup and cleanup code.

This creates:

- Duplicate code
- Higher maintenance
- Increased risk of inconsistencies

Hooks solve this problem.

---

# What are Test Hooks?

Test Hooks are predefined lifecycle methods that execute automatically before or after tests.

They allow engineers to perform:

- Initialization
- Cleanup
- Environment preparation
- Resource management

without duplicating logic inside every test.

---

# Real-World Analogy

Imagine a classroom.

Before every lecture:

```
Teacher Enters

↓

Projector Starts

↓

Attendance

↓

Lecture Begins
```

After every lecture:

```
Questions

↓

Save Notes

↓

Turn Off Projector

↓

Class Ends
```

Students focus on learning.

Routine preparation happens automatically.

Hooks provide the same convenience for automation.

---

# Why Hooks Exist

Hooks separate:

```
Test Logic

From

Infrastructure Logic
```

Instead of mixing setup code with business validation,

Hooks organize responsibilities.

---

# High-Level Hook Lifecycle

A simplified execution flow looks like:

```
Framework Starts

↓

Global Setup

↓

beforeAll

↓

beforeEach

↓

Test

↓

afterEach

↓

Repeat

↓

afterAll

↓

Framework Ends
```

Hooks surround test execution.

---

# Four Primary Hooks

Playwright provides four major lifecycle hooks.

```
beforeAll

↓

Run Once

-------------------------

beforeEach

↓

Run Before Every Test

-------------------------

afterEach

↓

Run After Every Test

-------------------------

afterAll

↓

Run Once After All Tests
```

Each has a specific purpose.

---

# Understanding `beforeAll`

Conceptually,

`beforeAll` performs work:

```
One Time

↓

Before All Tests
```

Typical responsibilities include:

- Starting shared resources
- Creating reusable test data
- Initializing services
- Preparing the environment

It is not intended for per-test activities.

---

# Real-World Example

Imagine organizing a conference.

Before attendees arrive:

```
Open Building

↓

Arrange Chairs

↓

Prepare Audio System

↓

Open Registration Desk
```

These activities happen only once.

This is similar to `beforeAll`.

---

# Internal Lifecycle of `beforeAll`

```
Framework Starts

↓

Discover Tests

↓

Execute beforeAll

↓

Environment Ready

↓

Begin Test Execution
```

Once completed,

tests begin.

---

# Understanding `beforeEach`

`beforeEach` runs:

```
Before Every Individual Test
```

Regardless of how many tests exist,

this hook executes before each one.

---

# Why `beforeEach` Is Important

Imagine five login tests.

```
Test 1

↓

beforeEach

----------------------

Test 2

↓

beforeEach

----------------------

Test 3

↓

beforeEach
```

Every test begins from a known, clean state.

---

# Enterprise Uses of `beforeEach`

Common activities include:

- Opening application
- Logging in
- Resetting test data
- Navigating to a page
- Preparing browser state

Anything required for every test belongs here.

---

# Understanding `afterEach`

`afterEach` executes:

```
After Every Test
```

Its primary responsibility is cleanup.

Examples include:

- Logout
- Remove temporary data
- Capture diagnostics
- Reset application state

Cleanup ensures one test does not affect another.

---

# Why Cleanup Matters

Suppose Test 1 creates:

```
Customer A
```

If cleanup never occurs,

Test 2 unexpectedly finds:

```
Customer A Already Exists
```

Now Test 2 may fail for the wrong reason.

Proper cleanup prevents cascading failures.

---

# Understanding `afterAll`

`afterAll` runs:

```
Once

↓

After Every Test Finishes
```

Typical responsibilities:

- Close shared resources
- Delete shared test data
- Stop servers
- Generate summaries
- Release connections

It represents the final cleanup stage.

---

# Real-World Analogy

Returning to the hotel example:

```
All Guests Leave

↓

Close Reception

↓

Turn Off Lights

↓

Lock Building
```

These activities occur only once.

This is similar to `afterAll`.

---

# Complete Hook Execution Order

Consider three tests.

Execution becomes:

```
beforeAll

↓

beforeEach

↓

Test 1

↓

afterEach

↓

beforeEach

↓

Test 2

↓

afterEach

↓

beforeEach

↓

Test 3

↓

afterEach

↓

afterAll
```

Understanding this sequence is essential.

---

# Visual Timeline

```
Framework Start

│

├── beforeAll

│

├── beforeEach

│      │

│      └── Test 1

│

├── afterEach

│

├── beforeEach

│      │

│      └── Test 2

│

├── afterEach

│

└── afterAll
```

The lifecycle remains consistent.

---

# Test Isolation

One of Playwright's core principles is:

```
Every Test

↓

Independent
```

No test should rely on:

- Previous execution
- Shared UI state
- Execution order

Hooks help enforce this isolation.

---

# Why Test Isolation Matters

Imagine:

```
Test A

↓

Creates Employee

↓

Test B

↓

Edits Employee
```

If Test A fails,

Test B also fails.

Now one bug creates multiple failures.

Independent tests avoid this problem.

---

# Resource Management

Hooks also manage expensive resources.

Examples:

- Database connections
- API clients
- Authentication sessions
- Test environments
- Temporary files

Instead of repeatedly creating these resources,

Hooks manage their lifecycle efficiently.

---

# Internal Working

Internally,

Playwright organizes execution like this:

```
Initialize Runner

↓

Execute Hooks

↓

Execute Test

↓

Capture Result

↓

Execute Cleanup

↓

Continue
```

Hooks become part of the execution engine,

not ordinary test code.

---

# Hook Architecture

```
Test Runner

↓

Lifecycle Manager

↓

Hook Engine

↓

Test Execution

↓

Result Collection
```

The Test Runner coordinates everything.

---

# Workflow Diagram

```
Framework

↓

Hook

↓

Test

↓

Cleanup

↓

Next Test

↓

Final Cleanup
```

Hooks surround every important stage.

---

# Architecture

```
                Test Runner

                     │

                     ▼

              Lifecycle Manager

                     │

      ┌──────────────┼──────────────┐

      ▼              ▼              ▼

 beforeAll      beforeEach     afterEach

                     │

                     ▼

                  Test Body

                     │

                     ▼

                 afterAll

                     │

                     ▼

             Test Results
```

Notice that the Test Runner—not the test itself—controls hook execution.

---

# Enterprise Example

Imagine an online banking framework.

Workflow:

```
beforeAll

↓

Start Test Environment

↓

beforeEach

↓

Login

↓

Execute Transfer Test

↓

afterEach

↓

Logout

↓

afterAll

↓

Close Resources
```

Each hook performs a clearly defined responsibility.

---

# Enterprise Hook Strategy

Professional teams generally organize hooks like this:

```
beforeAll

↓

Environment Initialization

----------------------------

beforeEach

↓

Prepare Clean Test State

----------------------------

afterEach

↓

Diagnostics

↓

Cleanup

----------------------------

afterAll

↓

Release Resources
```

Responsibilities remain predictable.

---

# What Should NOT Go Into Hooks?

Avoid placing:

- Business assertions
- Complex test logic
- Scenario-specific validation
- Conditional business workflows

Hooks should prepare and clean,

not validate business functionality.

---

# Best Practices

Professional engineers:

- Keep hooks focused.
- Make hooks predictable.
- Avoid unnecessary complexity.
- Ensure cleanup always occurs.
- Use `beforeEach` to maintain test isolation.
- Use `beforeAll` only for shared initialization.

---

# Common Beginner Mistakes

Many beginners:

- Put all logic into `beforeAll`.
- Share test data between tests.
- Skip cleanup.
- Place assertions inside hooks.
- Depend on execution order.

Remember:

Hooks support tests.

They should never replace test logic.

---

# Professional Tips

Experienced automation engineers ask:

```
Does every test require this?
```

If:

```
Yes

↓

beforeEach
```

If:

```
Only Once

↓

beforeAll
```

If:

```
Cleanup After Every Test

↓

afterEach
```

Choosing the correct lifecycle stage keeps frameworks clean and maintainable.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What are Test Hooks?

**Answer:**

Test Hooks are lifecycle methods that automatically execute before or after tests, allowing setup and cleanup logic to be centralized instead of duplicated.

---

### Mid-Level Question

**Q:** What is the difference between `beforeAll` and `beforeEach`?

**Answer:**

`beforeAll` executes once before all tests, whereas `beforeEach` executes before every individual test, ensuring each test starts from a consistent state.

---

### Senior-Level Question

**Q:** Why is `afterEach` important?

**Answer:**

`afterEach` performs cleanup after every test, preventing shared state, reducing flaky tests, and maintaining test independence.

---

### Lead-Level Question

**Q:** Why should hooks avoid business assertions?

**Answer:**

Hooks should manage infrastructure and lifecycle concerns. Business assertions belong inside test cases so failures clearly identify business behavior rather than setup logic.

---

### Architect-Level Question

**Q:** How would you define hook usage standards for an enterprise framework?

**Answer:**

I would enforce minimal, predictable hooks focused on environment preparation and cleanup, maintain strict test isolation, avoid business logic in hooks, and standardize lifecycle responsibilities across all automation projects.

---

# Knowledge Check

Before moving to the next lesson, answer these questions:

1. What are Test Hooks?
2. Why do Hooks exist?
3. What is the purpose of `beforeAll`?
4. What is the purpose of `beforeEach`?
5. Why is `afterEach` important?
6. What does `afterAll` typically perform?
7. What is Test Isolation?
8. Why should hooks avoid business assertions?
9. How do hooks improve maintainability?
10. What is the execution order of Playwright hooks?

---

# Step Summary

In this lesson, you learned:

- The purpose of Test Hooks
- The responsibilities of `beforeAll`, `beforeEach`, `afterEach`, and `afterAll`
- Hook execution order
- Resource management
- Test isolation
- Enterprise hook design patterns
- Best practices for building maintainable automation frameworks

You now understand that Hooks are the lifecycle backbone of the Playwright Test Runner. They organize setup and cleanup responsibilities, eliminate duplication, and ensure every test begins and ends in a predictable, isolated environment.

---

# Progress Milestone

✅ You have completed **Step 36** of approximately **230** planned learning steps.

**What you've mastered:**

- Test Hooks
- `beforeAll`
- `beforeEach`
- `afterEach`
- `afterAll`
- Hook Lifecycle
- Test Isolation
- Resource Management
- Enterprise Hook Design
- Hook Execution Order

**Coming next (Step 37):**

**Mastering Playwright Fixtures — Built-in Fixtures, Custom Fixtures, Dependency Injection, Fixture Scope, Worker Fixtures, Test Fixtures, Fixture Lifecycle, and Enterprise Dependency Management.**

In the next lesson, we will explore one of Playwright's most powerful features—**Fixtures**—learning how enterprise frameworks use dependency injection, reusable resources, worker-scoped objects, and custom fixtures to build highly scalable, maintainable automation architectures.
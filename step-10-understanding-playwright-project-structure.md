# Playwright with TypeScript Mastery Bootcamp
## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 2 — Development Environment & Installation

# Step 10 — Deep Dive into the Playwright Project Structure: Understanding Every Folder and File

---

# Objective

In this lesson, you will learn:

- Why project structure is important
- Every folder generated inside a Playwright project
- Every important configuration file
- The purpose of each file
- How these files interact with one another
- Which files should be committed to Git
- Which files should never be committed
- How enterprise Playwright frameworks organize projects
- Best practices for project organization

By the end of this lesson, you will be able to open any Playwright project and immediately understand its structure.

---

# Before We Start

Imagine entering a modern hospital for the first time.

Inside, there are many departments:

- Reception
- Emergency
- ICU
- Pharmacy
- Radiology
- Operation Theatre
- Administration

Each department has a different responsibility.

Imagine if everything were mixed together.

Medicines inside ICU.

Reception inside Pharmacy.

Patient records inside Cafeteria.

The hospital would become impossible to manage.

Software projects are no different.

Every folder has a purpose.

Professional engineers organize projects so that anyone can quickly understand where everything belongs.

---

# The Problem

Many beginners create projects like this:

```
login.spec.ts

dashboard.spec.ts

api.ts

test.ts

new.ts

sample.ts

utils.ts

newfile.ts

abc.ts
```

Everything is stored together.

After six months:

- Nobody knows what each file does.
- Duplicate code appears.
- Debugging becomes difficult.
- New team members struggle to understand the project.

Enterprise automation frameworks avoid this by following a structured organization.

---

# What is a Project Structure?

A project structure is the logical organization of files and folders inside a software project.

Good project structure provides:

- Readability
- Maintainability
- Scalability
- Collaboration
- Reusability

Think of it as the blueprint of a building.

Without a blueprint,

the building becomes difficult to expand.

---

# Typical Playwright Project Structure

A newly created Playwright project may look similar to:

```
Playwright-Project/

│

├── node_modules/

├── tests/

├── playwright.config.ts

├── package.json

├── package-lock.json

├── tsconfig.json

├── playwright-report/

├── test-results/
```

This structure may vary slightly depending on the Playwright version and options selected during project creation.

As our framework grows,

many additional folders will be added.

---

# High-Level Architecture

```
Project Root

│

├── Configuration

├── Dependencies

├── Test Files

├── Reports

├── Results

├── TypeScript Configuration

└── Package Management
```

Notice something important.

Each folder has exactly one primary responsibility.

This is good software design.

---

# Understanding the Project Root

The Project Root is the top-level directory.

Everything inside the automation framework exists beneath it.

Example:

```
Playwright-Framework/

↓

Everything Lives Here
```

Enterprise teams generally keep one Git repository for one automation framework.

---

# Understanding node_modules

We introduced this earlier,

but now let's understand it in context.

Purpose:

Stores installed packages.

Example:

```
node_modules/

↓

Playwright

↓

TypeScript

↓

Other Dependencies
```

Think of it as your local software warehouse.

---

# Should node_modules Be Committed to Git?

No.

Never.

Why?

Because every developer can recreate it using:

```
package.json

+

package-lock.json
```

Committing node_modules causes:

- Huge repositories
- Slow cloning
- Merge conflicts
- Storage waste

Professional projects ignore this folder using:

```
.gitignore
```

We'll learn Git later.

---

# Understanding package.json

This is one of the most frequently opened files.

It defines:

- Project name
- Version
- Dependencies
- Scripts
- Project metadata

Think of it as:

The identity card of the framework.

Whenever a new dependency is installed,

this file usually changes.

---

# How package.json Fits Into the Architecture

```
Developer

↓

Install Package

↓

package.json Updated

↓

Project Knows Dependency Exists
```

Without this file,

other developers wouldn't know which packages are required.

---

# Understanding package-lock.json

This file records the exact dependency versions installed.

Purpose:

Consistency.

Imagine:

Developer A installs today.

Developer B installs three months later.

Without package-lock.json,

they might receive different dependency versions.

With package-lock.json,

everyone receives identical versions.

---

# Enterprise Importance

Enterprise CI/CD pipelines depend heavily upon:

```
package.json

+

package-lock.json
```

These files guarantee reproducible builds.

---

# Understanding playwright.config.ts

This is arguably the most important file in the framework.

Think of it as the central control room.

It controls:

- Browser selection
- Timeouts
- Retries
- Reporters
- Projects
- Screenshots
- Videos
- Traces
- Base URL

Almost every Playwright project customization begins here.

---

# Architecture of playwright.config.ts

```
playwright.config.ts

│

├── Browser Settings

├── Timeout Settings

├── Reporter Settings

├── Retry Settings

├── Project Configuration

├── Execution Configuration

└── Test Behavior
```

Every future lesson involving configuration will reference this file.

---

# Understanding tsconfig.json

Since this bootcamp uses TypeScript,

we need TypeScript configuration.

Purpose:

Tell the TypeScript compiler:

- How to compile
- Which files to include
- Which language features to support
- Module configuration
- Strictness rules

Think of tsconfig.json as:

The instruction manual for the TypeScript compiler.

---

# Internal Workflow

```
TypeScript Source

↓

Read tsconfig.json

↓

Compile

↓

JavaScript Output
```

Without tsconfig.json,

TypeScript wouldn't know how to compile your project correctly.

---

# Understanding tests/

This folder stores your automation tests.

Initially,

it may contain:

```
example.spec.ts
```

Later,

it may contain:

```
tests/

│

├── smoke/

├── regression/

├── api/

├── ui/

├── e2e/
```

Good organization makes large projects manageable.

---

# Why Separate Test Types?

Imagine placing:

- Smoke Tests
- Regression Tests
- API Tests
- UI Tests

inside one folder.

Soon,

finding tests becomes difficult.

Instead,

logical grouping improves maintainability.

---

# Understanding playwright-report/

After test execution,

Playwright generates an HTML report.

Purpose:

Provide a visual summary of execution.

Contents may include:

- Passed tests
- Failed tests
- Duration
- Screenshots
- Error messages

This folder is usually generated automatically.

---

# Workflow

```
Run Tests

↓

Collect Results

↓

Generate HTML Report

↓

Store Inside

playwright-report/
```

---

# Understanding test-results/

This folder stores execution artifacts.

Examples:

- Screenshots
- Videos
- Traces
- Temporary execution files

Unlike source code,

these files are generated during execution.

They usually should not be committed to version control.

---

# Relationship Between Reports and Results

Many beginners confuse these folders.

Remember:

```
test-results/

↓

Execution Artifacts

---------------------

playwright-report/

↓

Human-Friendly Report
```

One stores raw execution data.

The other presents readable output.

---

# File Interaction

Let's observe how everything works together.

```
Developer

↓

Writes Test

↓

tests/

↓

Reads Configuration

↓

playwright.config.ts

↓

Uses Dependencies

↓

node_modules

↓

Execution

↓

Results

↓

test-results/

↓

Generate Report

↓

playwright-report/
```

Every folder participates in the execution lifecycle.

---

# Enterprise Project Structure

A real enterprise framework grows beyond the default structure.

Example:

```
Playwright-Framework/

│

├── tests/

├── pages/

├── components/

├── fixtures/

├── utils/

├── helpers/

├── api/

├── test-data/

├── config/

├── reports/

├── screenshots/

├── videos/

├── logs/

├── playwright.config.ts

├── package.json

├── tsconfig.json
```

Don't worry.

We'll build every one of these folders throughout this bootcamp.

---

# Which Files Should Be Committed?

Generally:

✅ Commit:

- package.json
- package-lock.json
- playwright.config.ts
- tsconfig.json
- tests
- source code

Usually ignore:

- node_modules
- test-results
- playwright-report
- temporary files
- logs
- videos

This keeps repositories clean.

---

# Internal Working

When a test executes:

```
Read Configuration

↓

Locate Test Files

↓

Load Dependencies

↓

Launch Browser

↓

Execute Tests

↓

Capture Results

↓

Generate Report
```

Every important folder contributes to the process.

---

# Architecture

Complete project architecture:

```
                 Project Root

                      │

      ┌───────────────┼────────────────┐

      ▼               ▼                ▼

Configuration     Dependencies      Test Files

      │               │                │

      ▼               ▼                ▼

Execution       Browser Control    Assertions

                      │

                      ▼

              Execution Results

                      │

         ┌────────────┼────────────┐

         ▼                         ▼

  test-results             playwright-report
```

---

# Enterprise Perspective

Large organizations maintain strict folder conventions.

Why?

Because hundreds of engineers may contribute to the same framework.

Standardization improves:

- Onboarding
- Code reviews
- Reusability
- Debugging
- Collaboration

Many companies even enforce project structures through templates and automated quality checks.

---

# Best Practices

As your framework grows:

- Keep folders focused on one responsibility.
- Separate tests from reusable code.
- Store configuration centrally.
- Never commit generated artifacts unnecessarily.
- Follow naming conventions consistently.
- Organize tests logically rather than alphabetically.

---

# Common Beginner Mistakes

Many beginners:

- Put everything inside the tests folder.
- Delete configuration files without understanding them.
- Commit node_modules to Git.
- Mix utilities with test files.
- Store screenshots alongside source code permanently.

These habits reduce maintainability.

---

# Professional Tips

Experienced automation engineers can often understand a project within minutes simply by examining its structure.

A clean folder hierarchy communicates:

- Team maturity
- Framework quality
- Maintainability
- Engineering discipline

Project structure is often the first impression reviewers receive.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What is the purpose of `playwright.config.ts`?

**Answer:**

It is the central configuration file that controls browser settings, execution behavior, reporters, retries, timeouts, projects, and many other Playwright options.

---

### Mid-Level Question

**Q:** Why shouldn't `node_modules` be committed to Git?

**Answer:**

Because it contains downloaded dependencies that can be recreated from `package.json` and `package-lock.json`. Committing it unnecessarily increases repository size and creates maintenance problems.

---

### Senior-Level Question

**Q:** What is the difference between `playwright-report` and `test-results`?

**Answer:**

`test-results` contains execution artifacts such as screenshots, videos, and traces. `playwright-report` contains the generated HTML report used for reviewing execution results.

---

### Lead-Level Question

**Q:** Why is project organization important in enterprise automation?

**Answer:**

Well-organized projects improve maintainability, reduce onboarding time, encourage code reuse, simplify debugging, and enable multiple teams to collaborate effectively.

---

### Architect-Level Question

**Q:** How would you design a scalable Playwright project structure?

**Answer:**

I would separate concerns into dedicated folders such as pages, components, fixtures, utilities, APIs, test data, configuration, reports, and tests. Configuration would remain centralized, generated artifacts excluded from version control, and folder conventions standardized across teams.

---

# Knowledge Check

Before moving to the next lesson, answer these questions:

1. What is a project structure?
2. Why is good project organization important?
3. What is stored inside `node_modules`?
4. What is the purpose of `playwright.config.ts`?
5. Why do we need `tsconfig.json`?
6. What belongs inside the `tests` folder?
7. What is the difference between `playwright-report` and `test-results`?
8. Which files should generally be committed to Git?
9. Why should generated artifacts usually be ignored?
10. How do enterprise teams organize large Playwright frameworks?

---

# Step Summary

In this lesson, you learned:

- The purpose of every major Playwright project file and folder
- How project structure supports maintainability
- The role of configuration, dependencies, reports, and test organization
- Which files are source code versus generated artifacts
- How enterprise Playwright frameworks evolve beyond the default project layout

You can now confidently explore a Playwright repository and understand what each major component is responsible for.

---

# Progress Milestone

✅ You have completed **Step 10** of approximately **230** planned learning steps.

**What you've mastered:**

- Complete Playwright project structure
- Configuration files
- Dependency management files
- Test organization
- Report folders
- TypeScript configuration
- Enterprise project layout

**Coming next (Step 11):**

**Your First Playwright Test — Understanding `import`, `test()`, `expect()`, `Page`, Browser Lifecycle, and the Complete Execution Flow of a Playwright Test from Start to Finish.**
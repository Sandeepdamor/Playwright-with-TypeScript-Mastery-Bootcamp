# Playwright with TypeScript Mastery Bootcamp
## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 18 — Playwright Configuration Architecture

# Step 44 — Mastering Playwright Configuration: `playwright.config.ts`, Configuration Hierarchy, `use`, `baseURL`, Timeouts, Retries, Workers, Reporters, Projects, Web Server, Global Settings, Environment-Aware Configuration, and Enterprise Configuration Architecture

---

# Objective

In this lesson, you will learn:

- What Playwright configuration is
- Why `playwright.config.ts` exists
- Configuration hierarchy
- Global configuration
- Project configuration
- The `use` configuration object
- Browser configuration
- `baseURL`
- Timeouts
- Retries
- Workers
- Reporters
- Screenshots
- Videos
- Traces
- Projects
- Web Server configuration
- Output directories
- Environment-aware configuration
- Configuration inheritance
- CI-specific configuration
- Enterprise configuration architecture

By the end of this lesson, you will understand how `playwright.config.ts` acts as the central control plane of a Playwright automation framework.

---

# Before We Start

Imagine an airplane.

Before the aircraft takes off,

many systems must be configured:

```
Flight Route

↓

Fuel

↓

Passengers

↓

Navigation

↓

Safety Systems

↓

Weather Conditions
```

The pilot should not manually configure every system for every passenger.

The aircraft has a centralized configuration system.

Playwright has a similar concept.

Instead of configuring every test individually,

you define common behavior centrally.

That central configuration is commonly represented by:

```
playwright.config.ts
```

---

# The Problem

Imagine having 2,000 tests.

Suppose every test individually specifies:

```
Browser

↓

Timeout

↓

Base URL

↓

Screenshot Policy

↓

Trace Policy

↓

Retries
```

This creates massive duplication.

Now imagine changing:

```
Timeout = 30 seconds
```

to:

```
Timeout = 60 seconds
```

You would have to update hundreds or thousands of files.

That is not scalable.

---

# What is Playwright Configuration?

Playwright Configuration defines how the Test Runner should execute the automation suite.

Conceptually:

```
Configuration

↓

Execution Rules

↓

Browser Behavior

↓

Test Behavior

↓

Reporting

↓

Environment
```

It provides centralized control.

---

# What is `playwright.config.ts`?

`playwright.config.ts` is the primary configuration file used by Playwright Test.

It allows the framework to define:

- Test locations
- Browser behavior
- Base URL
- Timeouts
- Retries
- Workers
- Projects
- Reporters
- Artifacts
- Web servers
- Global execution policies

It becomes the central configuration layer of the framework.

---

# Real-World Analogy

Think about a company's employee handbook.

Instead of telling every employee individually:

```
Work Hours

↓

Leave Rules

↓

Security Rules

↓

Office Policies
```

the organization defines them centrally.

Employees follow those policies automatically.

Playwright configuration works similarly.

---

# Why Centralized Configuration Matters

Central configuration provides:

- Consistency
- Maintainability
- Reusability
- Governance
- Easier CI/CD integration
- Environment control

The larger the project,

the more valuable centralized configuration becomes.

---

# Configuration Hierarchy

Playwright configuration can be thought of in layers.

Conceptually:

```
Global Configuration

↓

Project Configuration

↓

Test-Level Configuration

↓

Individual Operation
```

Higher-level configuration provides defaults.

More specific configuration can override those defaults where appropriate.

---

# Configuration Flow

```
Global Defaults

↓

Project Settings

↓

Test Settings

↓

Operation Settings
```

This creates flexibility without forcing duplication.

---

# Global Configuration

Global configuration applies broadly across the test suite.

Examples include:

- Test directory
- Reporter
- Global timeout
- Worker settings
- Retry policy
- Output directory

These define the overall execution behavior.

---

# Project Configuration

Project configuration applies to a specific execution profile.

Example:

```
Chromium Project

↓

Chromium Settings
```

Another:

```
Firefox Project

↓

Firefox Settings
```

A third:

```
Mobile Project

↓

Mobile Settings
```

This is where Project architecture and Configuration architecture meet.

---

# The `use` Configuration

One of the most important concepts is:

```
use
```

The `use` section defines common browser and test interaction behavior.

It can control concepts such as:

- Browser behavior
- Base URL
- Authentication state
- Viewport
- Permissions
- Locale
- Timezone
- Geolocation
- Color scheme
- Screenshot behavior
- Video behavior
- Trace behavior

Think of `use` as:

```
Default Browser/Test Context Behavior
```

---

# Why `use` Is Important

Suppose every test needs:

```
Same Base URL

↓

Same Authentication State

↓

Same Viewport

↓

Same Locale
```

Instead of repeating these settings,

define them centrally.

Then every relevant test inherits the behavior.

---

# `baseURL`

`baseURL` defines the application's base address.

Conceptually:

```
Base URL

↓

Application Root

↓

Test Navigation
```

Instead of repeatedly thinking about the full address,

tests can operate relative to the configured application root.

---

# Why `baseURL` Matters

Enterprise applications frequently have environments such as:

```
Development

↓

QA

↓

UAT

↓

Staging

↓

Production
```

Each environment has a different address.

A centralized `baseURL` allows the same tests to run against different environments.

---

# Environment-Aware Configuration

A mature framework may conceptually use:

```
Environment

↓

Configuration Resolver

↓

Base URL

↓

Project

↓

Test Execution
```

For example:

```
QA

↓

QA Configuration

↓

QA Application
```

and:

```
UAT

↓

UAT Configuration

↓

UAT Application
```

The test logic remains unchanged.

---

# Why Test Code Should Not Contain Environment URLs

Bad architecture:

```
Test

↓

Hardcoded QA URL
```

Another test:

```
Test

↓

Hardcoded UAT URL
```

This creates environment coupling.

Better:

```
Test

↓

Configuration

↓

Environment URL
```

Now environment changes do not require test-code modifications.

---

# Configuration and Timeouts

Timeout policies can be centralized.

For example:

```
Action Timeout

Navigation Timeout

Expect Timeout

Test Timeout
```

Each controls a different execution boundary.

---

# Why Centralized Timeout Policies Matter

Without centralization:

```
Test 1 → 10 seconds

Test 2 → 30 seconds

Test 3 → 45 seconds

Test 4 → 90 seconds
```

This becomes difficult to reason about.

A framework should have intentional timeout standards.

---

# Configuration and Retries

Retries can also be controlled centrally.

Conceptually:

```
Local Development

↓

Few or No Retries

----------------

CI

↓

Controlled Retry Policy
```

This allows different execution strategies for different environments.

---

# Configuration and Workers

Worker count determines how much parallelism the Test Runner can use.

Conceptually:

```
Workers

↓

Parallel Capacity
```

But more Workers do not automatically mean faster execution.

---

# Why Worker Count Is a Configuration Decision

Increasing Workers increases:

- CPU usage
- Memory usage
- Browser count
- Network traffic

Therefore:

```
Worker Count

↓

Must Match Infrastructure Capacity
```

---

# Configuration and Reporters

The configuration file can define reporting behavior.

Examples include:

```
List

↓

HTML

↓

JSON

↓

JUnit

↓

Custom Reporter
```

Different organizations may require different outputs.

---

# Why Reporter Configuration Matters

Local development may prefer:

```
Readable Terminal Output
```

CI may require:

```
Machine-Readable Results

+

HTML Report
```

Enterprise pipelines often consume test results programmatically.

---

# Screenshot Configuration

Screenshots can be controlled centrally.

Common strategies include:

```
Always

↓

On Failure

↓

Never
```

The enterprise default is often failure-oriented evidence collection to control artifact volume.

---

# Video Configuration

Video recording can also be configured centrally.

Possible strategies include:

```
Always

↓

On Failure

↓

Retain On Retry
```

Video is useful but consumes storage.

Therefore,

it should be enabled intentionally.

---

# Trace Configuration

Tracing is another major configuration capability.

A framework may choose strategies such as:

```
Always

↓

On First Retry

↓

On Failure
```

Tracing provides extremely valuable diagnostics,

but trace artifacts can be large.

---

# Artifact Strategy

Enterprise frameworks must balance:

```
Diagnostic Value

vs

Storage Cost
```

For example:

```
Screenshot

↓

Small Artifact

----------------

Video

↓

Larger Artifact

----------------

Trace

↓

Rich Diagnostic Artifact
```

Artifact retention policies should therefore be deliberate.

---

# Configuration and Projects

Projects are defined through configuration.

Conceptually:

```
Configuration

↓

Projects

├── Chromium

├── Firefox

├── WebKit

├── Mobile

└── API
```

Each Project can inherit common settings and override project-specific behavior.

---

# Configuration Inheritance

Imagine:

```
Base Configuration

↓

Common Settings

↓

Project Override
```

Example concept:

```
Common:

Retries

Reporter

Base URL

Tracing

↓

Chromium:

Desktop

↓

Firefox:

Desktop

↓

Mobile:

Mobile Device
```

This prevents duplication.

---

# Web Server Configuration

Many applications are locally hosted during development.

For example:

```
Test Runner

↓

Start Application

↓

Wait For Application

↓

Execute Tests

↓

Stop Application
```

Playwright can integrate with application startup through Web Server configuration.

---

# Why Web Server Integration Matters

Without automation:

```
Developer

↓

Start Application Manually

↓

Wait

↓

Run Tests
```

With integrated startup:

```
Playwright

↓

Start Application

↓

Wait Until Ready

↓

Run Tests
```

This makes local and CI execution more reproducible.

---

# Enterprise Web Server Considerations

A mature framework should define:

- Startup command
- Readiness condition
- Port strategy
- Reuse behavior
- CI behavior

The goal is:

```
Known Application State

↓

Known Test State
```

---

# Configuration and Test Discovery

The configuration can define where tests are located.

Conceptually:

```
Project

↓

Test Directory

↓

Test Files

↓

Test Discovery
```

This keeps the repository structure predictable.

---

# Configuration and Output Directories

Automation generates artifacts such as:

- Reports
- Screenshots
- Videos
- Traces
- Test results

These should be organized consistently.

Conceptually:

```
Test Execution

↓

Artifacts

↓

Output Directory

↓

CI Storage
```

---

# Why Artifact Organization Matters

Imagine 20,000 tests generating artifacts.

Without structure:

```
Files Everywhere
```

With structured output:

```
reports/

screenshots/

videos/

traces/

results/
```

Debugging and CI integration become easier.

---

# Configuration and Authentication

Authentication can also be incorporated into Project configuration.

Conceptually:

```
Authentication State

↓

Project

↓

Browser Context

↓

Authenticated Test
```

This allows tests to start with controlled session state.

---

# Configuration and Browser Context

Remember:

```
Browser

↓

BrowserContext

↓

Page
```

Many context-level behaviors can be configured centrally.

Examples include:

- Locale
- Timezone
- Permissions
- Geolocation
- Color scheme
- Viewport
- Storage state

---

# Configuration and Emulation

Projects can configure device behavior.

Conceptually:

```
Project

↓

Device Profile

↓

Browser Context

↓

Page
```

This allows consistent mobile and desktop execution.

---

# Configuration and Permissions

Some applications require browser permissions.

Examples:

```
Geolocation

↓

Notifications

↓

Camera

↓

Microphone
```

Projects can define controlled permission behavior.

---

# Configuration and Locale

International applications may need:

```
English

Hindi

French

German

Japanese
```

Locale configuration allows tests to validate localized behavior.

---

# Configuration and Timezone

Time-sensitive applications may behave differently depending on timezone.

Examples:

```
India

↓

United States

↓

United Kingdom

↓

Japan
```

A controlled timezone configuration improves test reproducibility.

---

# Configuration Architecture

A strong enterprise framework separates:

```
Framework Configuration

↓

Environment Configuration

↓

Project Configuration

↓

Test Data

↓

Secrets
```

These should not be mixed together.

---

# Configuration vs Secrets

This distinction is extremely important.

Configuration can contain:

```
Browser

↓

Timeout

↓

Reporter

↓

Base URL
```

Secrets include:

```
Passwords

↓

API Keys

↓

Tokens

↓

Private Credentials
```

Secrets should not be committed to source control.

---

# Environment Variables

Environment variables provide a common mechanism for external configuration.

Conceptually:

```
CI Environment

↓

Environment Variables

↓

Configuration Resolver

↓

Playwright Configuration
```

This allows the same codebase to behave differently in different environments.

---

# Enterprise Configuration Flow

```
CI/CD

↓

Environment Variables

↓

Configuration Layer

↓

Playwright Projects

↓

Browser Context

↓

Tests
```

The test code remains environment-independent.

---

# Configuration Anti-Pattern

Avoid:

```
Test Code

↓

if QA

↓

if UAT

↓

if Production

↓

Different URL

↓

Different Credentials

↓

Different Logic
```

This creates environment-specific test code.

---

# Better Architecture

Prefer:

```
Environment

↓

Configuration

↓

Same Test Logic
```

This is simpler and more scalable.

---

# Configuration as a Control Plane

At enterprise scale,

`playwright.config.ts` becomes more than a settings file.

It becomes a control plane for:

```
Execution

↓

Browser

↓

Environment

↓

Parallelism

↓

Artifacts

↓

Reporting

↓

Authentication

↓

Projects
```

---

# Workflow Diagram

```
CI/CD Pipeline

↓

Environment Selection

↓

Configuration Resolution

↓

Project Selection

↓

Browser Initialization

↓

Test Execution

↓

Artifact Collection

↓

Reporting
```

Configuration connects all major framework layers.

---

# Architecture

```
                       CI/CD

                         │

                         ▼

               Environment Variables

                         │

                         ▼

              Configuration Resolver

                         │

                         ▼

               playwright.config.ts

                         │

        ┌────────────────┼────────────────┐

        ▼                ▼                ▼

   Global Settings    Projects        Web Server

        │                │                │

        ▼                ▼                ▼

    Test Runner     Browser/Device    Application

        │                │                │

        └────────────────┼────────────────┘

                         ▼

                      Tests

                         │

                         ▼

                    Artifacts

                         │

                         ▼

                     Reports
```

The configuration layer connects execution infrastructure with test behavior.

---

# Enterprise Configuration Strategy

A mature enterprise framework commonly separates:

### Base Configuration

Contains:

- Shared reporters
- Common timeouts
- Default artifact policies
- Test directories

### Environment Configuration

Contains:

- Base URL
- Environment-specific settings

### Project Configuration

Contains:

- Browser
- Device
- Permissions
- Locale

### CI Configuration

Contains:

- Workers
- Retries
- Artifact retention
- Reporting

This creates clear separation of concerns.

---

# Configuration Profiles

Organizations may define conceptual profiles such as:

```
Local Development

↓

Fast Feedback
```

```
Pull Request

↓

Smoke + Chromium
```

```
Nightly

↓

Full Regression + Cross Browser
```

```
Release

↓

Critical Business Validation
```

The underlying tests remain the same.

Only execution configuration changes.

---

# Enterprise Example

Imagine an online banking platform.

### Developer Environment

```
Local

↓

Chromium

↓

Minimal Retries

↓

Fast Execution
```

### Pull Request

```
QA

↓

Chromium

↓

Smoke

↓

Controlled Retries
```

### Nightly

```
QA

↓

Chromium + Firefox + WebKit

↓

Regression

↓

Full Diagnostics
```

### Release

```
UAT

↓

Critical Business Tests

↓

Required Browser Matrix

↓

Release Decision
```

This is configuration-driven automation.

---

# Best Practices

Professional engineers:

- Keep configuration centralized.
- Avoid hardcoded environment URLs in tests.
- Separate secrets from configuration.
- Use Projects for execution variants.
- Keep common settings shared.
- Use environment variables for environment-specific values.
- Tune Workers according to infrastructure.
- Keep artifact policies intentional.
- Document configuration decisions.
- Avoid turning the configuration file into business logic.

---

# Common Beginner Mistakes

Many beginners:

- Put credentials directly into configuration files.
- Hardcode environment URLs.
- Create separate test files for each browser.
- Duplicate configuration across Projects.
- Set extremely high timeouts globally.
- Maximize Workers without considering machine capacity.
- Record videos and traces for every test without considering storage.
- Put business logic inside configuration.

---

# Professional Tips

Experienced automation architects treat configuration as code.

They ask:

```
What behavior should be centralized?

↓

What behavior should be project-specific?

↓

What should come from the environment?

↓

What is a secret?

↓

What belongs in the test itself?
```

These boundaries determine whether a framework remains maintainable as it grows.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What is `playwright.config.ts`?

**Answer:**

It is the central configuration file used by Playwright Test to define test execution behavior, browsers, projects, timeouts, retries, reporters, environments, artifacts, and other framework settings.

---

### Mid-Level Question

**Q:** What is the purpose of the `use` section?

**Answer:**

The `use` configuration defines common browser and BrowserContext behavior such as base URL, storage state, viewport, permissions, locale, screenshots, videos, and traces.

---

### Senior-Level Question

**Q:** How would you support QA, UAT, and staging environments without duplicating tests?

**Answer:**

I would externalize environment-specific values such as base URLs into environment configuration and keep test logic environment-independent. Projects or configuration profiles can then select the appropriate environment at runtime.

---

### Lead-Level Question

**Q:** Why shouldn't secrets be stored directly in `playwright.config.ts`?

**Answer:**

Because configuration files are normally committed to source control. Credentials and tokens can be exposed to developers, repositories, logs, and CI artifacts. Secrets should be supplied securely through secret-management mechanisms or protected environment variables.

---

### Architect-Level Question

**Q:** How would you design enterprise Playwright configuration?

**Answer:**

I would create a layered configuration architecture containing shared defaults, environment-specific settings, Project-specific browser/device configurations, CI execution policies, and secure secret injection. I would keep business logic out of configuration and make all important execution decisions explicit, documented, and reproducible.

---

# Knowledge Check

Before moving to the next lesson, answer these questions:

1. What is Playwright configuration?
2. What is `playwright.config.ts`?
3. Why is centralized configuration important?
4. What is configuration hierarchy?
5. What is the purpose of `use`?
6. Why is `baseURL` important?
7. How should environments be separated from test logic?
8. Why should secrets be separated from configuration?
9. How do Projects interact with configuration?
10. Why is Worker count a configuration decision?
11. Why should screenshot, video, and trace policies be centralized?
12. What is the purpose of Web Server configuration?
13. How can configuration support CI/CD?
14. Why should configuration not contain business logic?

---

# Step Summary

In this lesson, you learned:

- The purpose of `playwright.config.ts`
- Configuration hierarchy
- Global and Project configuration
- The `use` configuration
- `baseURL`
- Timeouts
- Retries
- Workers
- Reporters
- Screenshots
- Videos
- Traces
- Web Server integration
- Authentication state
- Browser Context configuration
- Device and environment configuration
- Environment variables
- Secrets separation
- Enterprise configuration architecture

The most important principle is:

> **Configuration should control how tests execute, while tests should describe what the application must do.**

A strong enterprise framework centralizes execution behavior, keeps environments externalized, protects secrets, and uses Projects and configuration layers to avoid test-code duplication.

---

# Progress Milestone

✅ You have completed **Step 44** of approximately **230** planned learning steps.

**What you've mastered:**

- `playwright.config.ts`
- Configuration Hierarchy
- Global Configuration
- Project Configuration
- `use`
- `baseURL`
- Timeout Configuration
- Retry Configuration
- Worker Configuration
- Reporter Configuration
- Screenshot Configuration
- Video Configuration
- Trace Configuration
- Web Server Configuration
- Environment-Aware Configuration
- Configuration Inheritance
- Enterprise Configuration Architecture

**Coming next — Step 45:**

**Mastering Environment Management — Development, QA, UAT, Staging, Production, `.env` Files, Environment Variables, Configuration Resolution, Secret Injection, Environment Validation, Multi-Environment Framework Design, and Enterprise Deployment Strategies.**

In the next lesson, we will go deeper into environment management and learn how enterprise automation frameworks safely execute the same Playwright suite across multiple environments without hardcoding URLs, credentials, or environment-specific business logic.
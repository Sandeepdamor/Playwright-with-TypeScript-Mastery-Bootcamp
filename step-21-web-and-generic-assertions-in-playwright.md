# Playwright with TypeScript Mastery Bootcamp
## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 6 — Assertions and Verification

# Step 21 — Understanding Assertions in Playwright: `expect()`, Web Assertions, Generic Assertions, Auto-Retrying Assertions, Hard Assertions, and Soft Assertions

---

# Objective

In this lesson, you will learn:

- What an assertion is
- Why assertions are the heart of testing
- The purpose of `expect()`
- Generic Assertions
- Web Assertions
- Auto-Retrying Assertions
- Hard Assertions
- Soft Assertions
- Assertion lifecycle
- Internal working of Playwright assertions
- Enterprise assertion strategy
- Best practices

By the end of this lesson, you will understand **why automation without assertions is not actually testing**.

---

# Before We Start

Imagine you are a quality inspector in a car manufacturing company.

A new car comes off the production line.

Do you simply look at the car and say:

> "Looks good."

No.

You verify:

- Are the brakes working?
- Are the headlights functioning?
- Does the engine start?
- Are the airbags installed?
- Are the doors locking correctly?

Only after verification do you approve the car.

Software testing works exactly the same way.

Automation performs actions.

Assertions verify whether those actions produced the correct results.

---

# The Problem

Imagine an automation script that performs:

```
Open Browser

↓

Open Website

↓

Click Login

↓

Enter Username

↓

Enter Password

↓

Click Submit

↓

Close Browser
```

Question:

Did login succeed?

Nobody knows.

Why?

Because nothing was verified.

This is automation.

It is **not** testing.

---

# What is an Assertion?

An assertion is a verification.

It compares:

```
Expected Result

↓

Actual Result

↓

Comparison

↓

Pass or Fail
```

An assertion answers one simple question:

> "Did the application behave exactly as expected?"

If yes,

the test passes.

If no,

the test fails.

---

# Why Are Assertions So Important?

Imagine a calculator.

You enter:

```
2 + 2
```

Expected:

```
4
```

Application returns:

```
5
```

Without an assertion,

the mistake goes unnoticed.

Assertions are what detect defects.

---

# Testing vs Automation

Many beginners confuse these concepts.

Automation means:

```
Perform Actions
```

Testing means:

```
Perform Actions

↓

Verify Results
```

Without verification,

there is no testing.

---

# The Role of `expect()`

Playwright performs assertions through:

```
expect()
```

Conceptually,

`expect()` means:

> "I expect the application to behave like this."

Example expectations include:

- Button should be visible.
- URL should change.
- Page title should match.
- Checkbox should be checked.
- Text should appear.

---

# High-Level Assertion Workflow

```
Action

↓

Application Responds

↓

expect()

↓

Verification

↓

Pass or Fail
```

Every meaningful test follows this pattern.

---

# Types of Assertions in Playwright

Playwright provides two major categories.

```
Assertions

│

├── Generic Assertions

└── Web Assertions
```

Understanding the difference is extremely important.

---

# What are Generic Assertions?

Generic Assertions verify ordinary values.

Examples:

- Numbers
- Strings
- Arrays
- Objects
- Boolean values

These assertions are not related to browsers.

Think of them as:

```
General Programming Assertions
```

---

# Example Conceptually

Suppose we compare:

```
Expected Number

↓

Actual Number
```

Or:

```
Expected Text

↓

Actual Text
```

These are generic comparisons.

They do not require a webpage.

---

# What are Web Assertions?

Web Assertions verify browser-related conditions.

Examples include:

- Visibility
- Text
- URL
- Title
- Attribute
- Checkbox state
- Input value
- Enabled state
- Disabled state

These assertions understand webpages.

---

# Why Separate Generic and Web Assertions?

Imagine verifying:

```
5 + 5 = 10
```

No browser required.

Now imagine verifying:

```
Login Button

↓

Visible
```

Now the browser becomes essential.

Different problems require different assertion types.

---

# Auto-Retrying Assertions

This is one of Playwright's biggest advantages.

Imagine verifying:

```
Order Status

↓

Approved
```

Immediately after clicking:

```
Submit
```

The application updates after:

```
3 Seconds
```

Should the assertion fail immediately?

No.

Instead,

Playwright waits.

---

# Auto-Retry Workflow

```
Assertion Starts

↓

Condition True?

↓

No

↓

Wait Briefly

↓

Check Again

↓

Condition True?

↓

Yes

↓

Pass
```

Notice something.

The assertion keeps trying automatically.

---

# Why Auto-Retry Is Powerful

Traditional frameworks often behave like this:

```
Check Once

↓

Wrong

↓

Fail
```

Playwright behaves differently.

```
Check

↓

Not Ready

↓

Retry

↓

Retry

↓

Retry

↓

Correct

↓

Pass
```

This dramatically reduces flaky tests.

---

# Relationship with Expect Timeout

Auto-Retry does **not** continue forever.

Remember Step 20.

Assertions use:

```
Expect Timeout
```

Workflow:

```
Retry

↓

Retry

↓

Retry

↓

Success

or

↓

Timeout

↓

Failure
```

---

# Hard Assertions

A Hard Assertion immediately stops the current test if it fails.

Example:

```
Login Failed

↓

Continue Checkout?
```

No.

The test stops.

Why?

Because continuing no longer makes sense.

---

# Hard Assertion Workflow

```
Assertion

↓

Pass?

↓

Yes

↓

Continue

-------------------

No

↓

Stop Test

↓

Report Failure
```

This is the default behavior.

---

# Soft Assertions

Sometimes,

we want to continue even if one verification fails.

Imagine verifying a dashboard.

Checks include:

- Logo
- Welcome Message
- Profile Picture
- Notifications
- Menu

Suppose:

```
Logo Missing
```

Do we still want to check everything else?

Sometimes,

Yes.

Soft Assertions allow this.

---

# Soft Assertion Workflow

```
Check Logo

↓

Fail

↓

Record Failure

↓

Continue

↓

Check Menu

↓

Continue

↓

Check Notifications

↓

Continue

↓

Generate Report
```

Instead of stopping immediately,

all failures are collected.

---

# When Should Soft Assertions Be Used?

Soft Assertions are useful when:

- Validating dashboards.
- Checking multiple UI elements.
- Performing visual inspections.
- Auditing large pages.

They are generally **not** suitable for critical business flows.

---

# When Should Hard Assertions Be Used?

Use Hard Assertions when:

Failure makes the rest of the test meaningless.

Examples:

- Login
- Payment
- Authentication
- Navigation
- Checkout completion

If login fails,

there is little value in testing logout.

---

# Assertion Lifecycle

Let's examine the complete lifecycle.

```
Action Completed

↓

Assertion Starts

↓

Evaluate Condition

↓

True?

↓

Yes

↓

Pass

-------------------

No

↓

Retry (Web Assertions)

↓

Timeout?

↓

Fail
```

This lifecycle applies to most Playwright web assertions.

---

# Internal Working

Suppose we verify:

```
Button Visible
```

Internally,

Playwright performs:

```
Locate Element

↓

Evaluate Visibility

↓

Visible?

↓

No

↓

Retry

↓

Visible?

↓

Yes

↓

Pass
```

This intelligent retry mechanism is built into the framework.

---

# Workflow Diagram

```
Test

↓

Action

↓

expect()

↓

Evaluate Condition

↓

Auto Retry

↓

Success

↓

Continue

or

↓

Timeout

↓

Failure
```

Notice that assertions integrate closely with Auto Waiting.

---

# Architecture

```
               Test Script

                    │

                    ▼

                expect()

                    │

       ┌────────────┼────────────┐

       ▼                         ▼

 Generic Assertions      Web Assertions

       │                         │

       ▼                         ▼

 Immediate Check         Auto Retry Engine

                                 │

                                 ▼

                           Expect Timeout

                                 │

                                 ▼

                          Pass / Fail
```

Playwright provides specialized behavior for browser-related assertions.

---

# Enterprise Example

Imagine testing an e-commerce checkout.

Workflow:

```
Login

↓

Search Product

↓

Add to Cart

↓

Checkout

↓

Order Confirmation
```

Assertions verify every business milestone.

Example:

- Login successful.
- Correct product displayed.
- Cart updated.
- Payment completed.
- Confirmation shown.

Without assertions,

automation cannot prove business correctness.

---

# Enterprise Assertion Strategy

Large organizations often define assertion guidelines.

Typical strategy:

```
Critical Business Flow

↓

Hard Assertions

--------------------

Dashboard Validation

↓

Soft Assertions

--------------------

Dynamic UI

↓

Auto-Retrying Web Assertions
```

Different situations require different assertion strategies.

---

# Best Practices

Professional engineers:

- Assert meaningful business outcomes.
- Prefer Web Assertions for UI verification.
- Use Hard Assertions for critical workflows.
- Use Soft Assertions for independent validations.
- Avoid unnecessary assertions.
- Keep assertions readable.

---

# Common Beginner Mistakes

Many beginners:

- Write tests without assertions.
- Verify implementation details instead of business behavior.
- Use Hard Assertions everywhere.
- Use Soft Assertions for login or payment.
- Duplicate the same assertion repeatedly.
- Ignore assertion failures.

Remember:

An assertion should verify **business value**,

not simply technical implementation.

---

# Professional Tips

Experienced automation engineers ask:

- What business requirement am I verifying?
- Does this assertion protect against a real defect?
- Is this the right place for a Hard Assertion?
- Would a Soft Assertion improve reporting?
- Is the assertion understandable to another engineer?

Good assertions communicate intent.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What is an assertion?

**Answer:**

An assertion verifies that the application's actual behavior matches the expected behavior. If they match, the test passes; otherwise, it fails.

---

### Mid-Level Question

**Q:** What is the difference between Generic Assertions and Web Assertions?

**Answer:**

Generic Assertions verify ordinary programming values such as strings or numbers. Web Assertions verify browser-related conditions such as visibility, text, URLs, titles, and element states.

---

### Senior-Level Question

**Q:** What are Auto-Retrying Assertions?

**Answer:**

Auto-Retrying Assertions repeatedly evaluate browser conditions until they succeed or the Expect Timeout expires, making tests more reliable for dynamic applications.

---

### Lead-Level Question

**Q:** When should Soft Assertions be used?

**Answer:**

Soft Assertions are appropriate when multiple independent validations should continue even after one failure, such as verifying dashboards or UI audits.

---

### Architect-Level Question

**Q:** How would you define an enterprise assertion strategy?

**Answer:**

I would use Hard Assertions for critical business workflows, Soft Assertions for independent UI validations, Web Assertions for browser state verification, and ensure every assertion directly maps to a business requirement.

---

# Knowledge Check

Before moving to the next lesson, answer these questions:

1. What is an assertion?
2. Why are assertions considered the heart of testing?
3. What is the purpose of `expect()`?
4. What is the difference between Generic Assertions and Web Assertions?
5. What are Auto-Retrying Assertions?
6. What is a Hard Assertion?
7. What is a Soft Assertion?
8. When should Soft Assertions be preferred?
9. Why do Web Assertions improve test reliability?
10. How do enterprise teams typically organize their assertion strategy?

---

# Step Summary

In this lesson, you learned:

- The role of assertions in software testing
- Why `expect()` is central to Playwright
- The difference between Generic and Web Assertions
- How Auto-Retrying Assertions work
- The concepts of Hard and Soft Assertions
- The assertion lifecycle
- Enterprise assertion strategies and best practices

You now understand that automation becomes **testing** only when meaningful assertions verify business outcomes.

---

# Progress Milestone

✅ You have completed **Step 21** of approximately **230** planned learning steps.

**What you've mastered:**

- Assertion fundamentals
- `expect()`
- Generic Assertions
- Web Assertions
- Auto-Retrying Assertions
- Hard Assertions
- Soft Assertions
- Assertion lifecycle
- Enterprise assertion strategy

**Coming next (Step 22):**

**Mastering Playwright Web Assertions — `toBeVisible()`, `toBeHidden()`, `toHaveText()`, `toContainText()`, `toHaveValue()`, `toHaveAttribute()`, `toBeChecked()`, `toBeEnabled()`, `toBeDisabled()`, `toHaveURL()`, `toHaveTitle()`, and the Internal Working of Every Web Assertion.**

In the next lesson, we will begin a detailed exploration of every major **Playwright Web Assertion**, understanding not only how they work but also **why**, **when**, and **where** to use each one in enterprise automation.
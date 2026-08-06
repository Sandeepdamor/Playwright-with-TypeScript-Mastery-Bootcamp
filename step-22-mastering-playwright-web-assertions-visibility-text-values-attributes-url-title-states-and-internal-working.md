# Playwright with TypeScript Mastery Bootcamp
## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 6 — Assertions and Verification

# Step 22 — Mastering Playwright Web Assertions: Visibility, Text, Values, Attributes, URL, Title, States, and Internal Working

---

# Objective

In this lesson, you will learn:

- Why Web Assertions are different from Generic Assertions
- Every major Playwright Web Assertion
- When to use each assertion
- Internal working of Web Assertions
- Auto-Retry behavior
- Enterprise use cases
- Assertion selection strategy
- Best practices
- Common mistakes

By the end of this lesson, you will understand **which Web Assertion should be used for every UI verification scenario**.

---

# Before We Start

Imagine you are inspecting a newly built house.

You don't ask just one question.

Instead, you inspect multiple aspects.

Examples:

```
Is the door installed?

↓

Is the window open?

↓

Does the light work?

↓

Is the water running?

↓

Is the address correct?
```

Each inspection verifies a different property.

Similarly,

Playwright provides different assertions because webpages have many different properties.

---

# The Problem

Suppose a Login page appears.

How do you verify it?

Possible checks include:

- Login button is visible
- Username field is enabled
- Password field accepts input
- Page title is correct
- URL changed correctly
- Welcome message appears

One assertion cannot verify all these conditions.

Different situations require different assertions.

---

# Understanding Web Assertions

Web Assertions verify the state of a webpage.

Unlike Generic Assertions,

they understand browser behavior.

Examples:

```
Visible

Hidden

Enabled

Disabled

Checked

URL

Title

Text

Attribute

Input Value
```

These assertions automatically interact with the browser.

---

# Why Are Web Assertions Intelligent?

Remember Step 21.

Web Assertions include:

```
Auto Retry
```

Workflow:

```
Condition False

↓

Retry

↓

Retry

↓

Retry

↓

Success

↓

Pass
```

Unlike ordinary comparisons,

they understand that webpages change over time.

---

# Complete Web Assertion Family

The most commonly used assertions include:

```
Visibility

↓

toBeVisible()

toBeHidden()

-----------------------

State

↓

toBeEnabled()

toBeDisabled()

toBeEditable()

toBeChecked()

-----------------------

Content

↓

toHaveText()

toContainText()

-----------------------

Input

↓

toHaveValue()

-----------------------

Attributes

↓

toHaveAttribute()

-----------------------

Navigation

↓

toHaveURL()

toHaveTitle()
```

Each assertion answers a different business question.

---

# `toBeVisible()`

Question answered:

```
Can the user currently see this element?
```

Examples:

- Login Button
- Error Message
- Success Banner
- Navigation Menu
- Modal Dialog

Visibility is one of the most common UI validations.

---

# Internal Working of `toBeVisible()`

```
Locate Element

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

Notice that Playwright retries automatically.

---

# Enterprise Usage

Typical validations include:

- Login button visible
- Dashboard loaded
- Success notification displayed
- Loading spinner disappeared
- Confirmation dialog shown

---

# `toBeHidden()`

Sometimes,

we verify the opposite.

Question:

```
Has this element disappeared?
```

Examples:

- Loading Spinner
- Popup
- Progress Bar
- Modal Window
- Temporary Notification

---

# Internal Workflow

```
Locate Element

↓

Hidden?

↓

No

↓

Retry

↓

Hidden?

↓

Yes

↓

Pass
```

---

# `toHaveText()`

Question:

```
Does this element contain exactly the expected text?
```

This assertion performs strict comparison.

Example conceptually:

Expected:

```
Login Successful
```

Actual:

```
Login Successful
```

Pass.

---

# Why Exact Text Matters

Business applications often require precise wording.

Examples:

- Invoice Status
- Payment Status
- Account Status
- Approval Result

Exact text verification prevents subtle UI defects.

---

# `toContainText()` 

Sometimes,

exact text is unnecessary.

Example:

```
Welcome Rahul Sharma
```

Tomorrow:

```
Welcome Priya Patel
```

The username changes,

but the message still begins with:

```
Welcome
```

Partial matching becomes more appropriate.

---

# When Should Partial Text Be Used?

Suitable for:

- Dynamic greetings
- Notifications
- Messages containing timestamps
- User-specific information
- IDs generated at runtime

---

# `toHaveValue()`

Question:

```
What value currently exists inside this input?
```

Useful for verifying:

- Username field
- Search box
- Quantity input
- Date picker
- Form values

Notice:

This checks the input value,

not the visible label.

---

# Internal Workflow

```
Locate Input

↓

Read Current Value

↓

Compare

↓

Pass or Retry
```

---

# `toHaveAttribute()`

HTML elements contain attributes.

Examples include:

- href
- src
- alt
- title
- disabled
- placeholder

This assertion verifies those values.

---

# Enterprise Example

Imagine a download link.

Verification:

```
Correct File

↓

Correct Link

↓

Correct Attribute
```

Attribute verification protects against incorrect application behavior.

---

# `toBeChecked()`

Checkboxes and radio buttons have state.

Question:

```
Is the option selected?
```

Examples:

- Accept Terms
- Remember Me
- Newsletter
- Payment Option

This assertion verifies selection state.

---

# `toBeEnabled()`

Question:

```
Can the user currently interact with this control?
```

Common examples:

- Submit Button
- Save Button
- Search Button

Many applications disable buttons until required information is entered.

---

# `toBeDisabled()`

The opposite question.

```
Should this control currently prevent interaction?
```

Enterprise examples:

- Save button before mandatory fields
- Delete button without permission
- Approve button awaiting authorization

---

# `toBeEditable()`

Some fields exist,

but cannot currently accept input.

Question:

```
Can the user type here?
```

Useful when:

- Read-only mode
- Locked forms
- Permission-based editing

---

# `toHaveURL()`

Navigation verification.

Question:

```
Did the browser navigate to the expected page?
```

Examples:

```
Login

↓

Dashboard
```

or

```
Checkout

↓

Confirmation
```

Navigation is a critical business validation.

---

# `toHaveTitle()`

Question:

```
Does the browser title match expectations?
```

Although less common than URL verification,

title validation remains useful for:

- SEO
- Browser tabs
- Legacy applications
- Multi-page workflows

---

# Internal Working of Web Assertions

Regardless of assertion type,

Playwright follows a similar pattern.

```
Locate Element

↓

Evaluate Condition

↓

Satisfied?

↓

Yes

↓

Pass

---------------------

No

↓

Retry

↓

Retry

↓

Retry

↓

Timeout

↓

Fail
```

This retry mechanism makes Web Assertions extremely reliable.

---

# Assertion Selection Strategy

Professional engineers think in terms of business questions.

Instead of asking:

```
Which assertion should I use?
```

they ask:

```
What am I trying to verify?
```

Examples:

```
Visible?

↓

toBeVisible()
```

```
Exact Text?

↓

toHaveText()
```

```
Partial Text?

↓

toContainText()
```

```
Navigation?

↓

toHaveURL()
```

```
Input Value?

↓

toHaveValue()
```

Business intent determines the assertion.

---

# Workflow Diagram

```
Business Requirement

↓

Choose Assertion

↓

Auto Retry

↓

Expectation Met?

↓

Pass

or

↓

Timeout

↓

Fail
```

Assertions should reflect business requirements,

not implementation details.

---

# Architecture

```
                 Test Script

                      │

                      ▼

                  expect()

                      │

        ┌─────────────┼─────────────┐

        ▼             ▼             ▼

   Visibility     Content       Navigation

        │             │             │

        ▼             ▼             ▼

     State      Text / Value   URL / Title

                      │

                      ▼

              Auto Retry Engine

                      │

                      ▼

               Pass / Fail
```

Different assertions verify different browser properties,

but all use the same intelligent retry engine.

---

# Enterprise Example

Imagine testing an employee portal.

Business workflow:

```
Login

↓

Dashboard Opens

↓

Employee Name Appears

↓

Profile Button Enabled

↓

Welcome Message Displayed

↓

Correct URL Loaded
```

Different assertions verify each milestone.

No single assertion can replace them all.

---

# Enterprise Assertion Strategy

Large organizations often establish standards.

Example:

```
Navigation

↓

URL Assertions

-----------------------

Forms

↓

Value Assertions

-----------------------

Messages

↓

Text Assertions

-----------------------

Buttons

↓

State Assertions

-----------------------

Dialogs

↓

Visibility Assertions
```

Consistent assertion strategies improve framework readability.

---

# Best Practices

Professional engineers:

- Verify business outcomes.
- Choose assertions that match application behavior.
- Prefer exact text only when wording must not change.
- Use partial text for dynamic content.
- Verify navigation after important workflows.
- Keep assertions simple and meaningful.

---

# Common Beginner Mistakes

Many beginners:

- Use `toHaveText()` for dynamic messages.
- Verify implementation details instead of business behavior.
- Forget URL verification after navigation.
- Duplicate assertions unnecessarily.
- Ignore built-in auto retry.

Remember:

Choose assertions based on the **business requirement**,

not convenience.

---

# Professional Tips

Experienced automation engineers often write assertions that read almost like acceptance criteria.

Example mentally:

```
User logs in

↓

Dashboard becomes visible

↓

Welcome message appears

↓

Correct URL loads

↓

Profile menu becomes enabled
```

Well-designed assertions make tests understandable to both developers and business analysts.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What does `toBeVisible()` verify?

**Answer:**

It verifies that an element is visible to the user and automatically retries until the condition is satisfied or the Expect Timeout expires.

---

### Mid-Level Question

**Q:** What is the difference between `toHaveText()` and `toContainText()`?

**Answer:**

`toHaveText()` verifies the complete expected text, while `toContainText()` verifies only that the expected text appears somewhere within the actual content.

---

### Senior-Level Question

**Q:** Why do Playwright Web Assertions reduce flaky tests?

**Answer:**

Because they automatically retry browser-related conditions instead of performing a single immediate comparison. This accommodates asynchronous UI updates and dynamic rendering.

---

### Lead-Level Question

**Q:** How do you decide which assertion to use?

**Answer:**

I first identify the business requirement being validated—visibility, navigation, text, state, value, or attribute—and then select the assertion that most accurately expresses that requirement.

---

### Architect-Level Question

**Q:** Why should assertion standards be established across enterprise automation teams?

**Answer:**

Consistent assertion strategies improve readability, simplify maintenance, encourage reusable patterns, reduce ambiguity during code reviews, and ensure tests clearly reflect business requirements.

---

# Knowledge Check

Before moving to the next lesson, answer these questions:

1. What is the difference between Generic Assertions and Web Assertions?
2. What does `toBeVisible()` verify?
3. When should `toBeHidden()` be used?
4. What is the difference between `toHaveText()` and `toContainText()`?
5. What does `toHaveValue()` verify?
6. Why is `toHaveAttribute()` useful?
7. When should `toBeChecked()` be used?
8. Why is `toHaveURL()` important after navigation?
9. Why do Web Assertions automatically retry?
10. How should enterprise teams choose the correct assertion?

---

# Step Summary

In this lesson, you learned:

- The purpose of Playwright Web Assertions
- How visibility, state, content, navigation, and attribute assertions work
- The difference between exact and partial text assertions
- The internal lifecycle of Web Assertions
- Enterprise assertion selection strategies
- Best practices for writing meaningful and maintainable assertions

You now understand how to choose the **right assertion for the right business requirement**, making your tests both expressive and reliable.

---

# Progress Milestone

✅ You have completed **Step 22** of approximately **230** planned learning steps.

**What you've mastered:**

- `toBeVisible()`
- `toBeHidden()`
- `toHaveText()`
- `toContainText()`
- `toHaveValue()`
- `toHaveAttribute()`
- `toBeChecked()`
- `toBeEnabled()`
- `toBeDisabled()`
- `toBeEditable()`
- `toHaveURL()`
- `toHaveTitle()`
- Web Assertion lifecycle
- Enterprise assertion strategy

**Coming next (Step 23):**

**Understanding User Actions in Playwright — Click, Fill, Type, Press, Check, Uncheck, Select Option, Hover, Focus, Blur, Drag & Drop, and the Complete Internal Lifecycle of Every User Interaction.**

In the next lesson, we will begin a comprehensive exploration of **Playwright User Actions**, learning not just how to perform them but also what happens internally before, during, and after every interaction.
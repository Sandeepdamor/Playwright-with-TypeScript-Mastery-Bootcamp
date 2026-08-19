# Playwright with TypeScript Mastery Bootcamp

## From Absolute Beginner to Enterprise Automation Architect

# Stage 29 — Locator Engineering

# Step 61 — Mastering `getByText()`: Visible Text, Text Matching, Exact Matching, Whitespace, Nested Text, Dynamic Text, Repeated Text, Localization, Scope, Filtering, and Enterprise Text-Locator Strategy

---

# Objective

In the previous lesson, you learned:

```text
getByRole()
```

and understood how Playwright can identify elements through semantic meaning.

Now we study another extremely useful locator:

```text
getByText()
```

Text is one of the most obvious things a human sees on a webpage.

A user thinks:

```text
"Click Orders."
"Open Settings."
"Select Pending."
"Verify Payment Successful."
```

Therefore text can be a powerful automation identifier.

But text-based locating has many subtle problems.

Consider:

```text
Orders
```

It may appear in:

```text
Navigation
Page Heading
Breadcrumb
Button
Table
Footer
```

So simply saying:

```text
Find "Orders"
```

may not be enough.

In this lesson, you will deeply understand:

- What `getByText()` does
- Visible text
- Text nodes
- Text content
- Nested text
- Whitespace normalization
- Case sensitivity
- Exact matching
- Partial matching
- Substring matching
- Regular-expression concepts
- Dynamic text
- Repeated text
- Text inside components
- Text inside tables
- Text inside cards
- Text inside dialogs
- Text-based filtering
- Scoping text locators
- Localization problems
- Internationalization
- Why text can be unstable
- When `getByText()` is appropriate
- When `getByText()` should be avoided
- `getByText()` versus `getByRole()`
- Enterprise text-locator strategy

---

# Before We Start

Imagine entering a supermarket.

You ask an employee:

```text
"Where is milk?"
```

They understand what you mean.

But imagine the supermarket contains:

```text
Milk
Milk Powder
Milk Chocolate
Milkshake
Milk Cookies
```

Now the word:

```text
Milk
```

is not enough.

You may need:

```text
"Dairy Milk"
```

or:

```text
"Milk in the refrigerated section"
```

or:

```text
"Milk, 1 litre"
```

The same principle applies to web automation.

Text can identify something very well when it is:

```text
Unique
Stable
Meaningful
```

But text becomes dangerous when it is:

```text
Repeated
Dynamic
Localized
Ambiguous
```

---

# The Problem

Suppose an application contains:

```text
Dashboard

Orders

Products

Orders
```

A beginner may think:

```text
Find "Orders"
```

But which Orders?

```text
Navigation Orders
        OR
Page Heading Orders
```

Both may be valid.

The automation engineer must understand:

```text
Text
+
Context
+
Scope
```

---

# What is `getByText()`?

`getByText()` is a Playwright locator API used to locate elements based on text content.

Conceptually:

```text
Page
 ↓
Text
 ↓
Matching Element
```

The important question is:

> What text does Playwright consider relevant to the locator?

To answer that, we need to understand how text exists in HTML.

---

# What is Text in the DOM?

Consider a conceptual element:

```text
<button>Save Order</button>
```

The element contains text:

```text
Save Order
```

At a lower level, text is represented through text nodes within the DOM.

Conceptually:

```text
Button Element
     │
     └── Text Node
           │
           └── "Save Order"
```

---

# Visible Text vs DOM Text

These are related but not always identical.

A page can contain:

```text
Text in DOM
```

that is:

```text
Hidden
Invisible
Decorative
Generated
```

Therefore, when designing text-based locators, you must think about what the user actually perceives.

---

# Why Text Matching Is Useful

Text is often:

```text
Business Meaning
```

For example:

```text
Orders
Approve
Cancel
Pending
Completed
Payment Successful
```

These words communicate application behavior.

This can make tests highly readable.

---

# Readability Example

Compare:

```text
Locate element with class:
"menu-item-active-2f83"
```

with:

```text
Locate:
"Orders"
```

The second is immediately understandable.

---

# Text Matching Is Not Always Unique

Suppose the page contains:

```text
Orders
```

in five places.

Then:

```text
getByText("Orders")
```

may represent multiple matching elements.

This can result in:

```text
Ambiguity
```

or:

```text
Unexpected Interaction
```

depending on how the locator is used.

---

# Text Locator Mental Model

Use this mental model:

```text
Text Requirement
      ↓
Is text stable?
      ↓
Is text unique?
      ↓
Is text user-visible?
      ↓
Is text localized?
      ↓
Does context matter?
      ↓
Can role identify it better?
      ↓
Choose getByText()
```

---

# `getByText()` vs `getByRole()`

Suppose you have:

```text
Save
```

If it is a button, consider:

```text
Role = button
Name = Save
```

rather than using text alone.

Why?

Because:

```text
getByRole()
```

expresses:

```text
This is the Save button.
```

while:

```text
getByText()
```

expresses:

```text
Find something containing Save.
```

The first can carry more semantic information.

---

# When `getByText()` Is Excellent

Text locators are especially useful for:

```text
Static visible labels
Messages
Notifications
Headings
Status text
Empty-state messages
User-facing informational content
Business values
```

For example:

```text
Payment Successful
```

is often an excellent assertion target.

---

# When `getByRole()` May Be Better

For:

```text
Buttons
Links
Checkboxes
Radio buttons
Tabs
Dialogs
Navigation
```

role-based locating often expresses more precise intent.

For example:

```text
Button "Delete"
```

is more specific than:

```text
Text "Delete"
```

---

# Exact Matching

Text matching can involve the difference between:

```text
Exact
```

and:

```text
Partial
```

Consider:

```text
Save
Save Draft
Save Order
```

A broad text search for:

```text
Save
```

could match multiple elements.

Exact matching conceptually means:

```text
"Save"
```

rather than:

```text
Anything containing "Save"
```

---

# Why Exact Matching Matters

Imagine an application with:

```text
Delete
Delete User
Delete Account
Delete All
```

If your requirement is:

```text
Click Delete
```

you must ensure you are not accidentally selecting:

```text
Delete User
```

or:

```text
Delete All
```

---

# Partial Matching

Partial matching can be useful when text contains dynamic content.

For example:

```text
Order #1001 created successfully
```

The stable portion may be:

```text
created successfully
```

or:

```text
Order #1001
```

depending on the requirement.

---

# Dynamic Text

Modern applications frequently display dynamic text.

Examples:

```text
Welcome, Sandeep
Order #12345
Total: ₹5,420
Last updated: 2 minutes ago
5 notifications
```

The entire string may change between executions.

---

# Dynamic Text Problem

Suppose the application displays:

```text
Order #1001 created successfully
```

Tomorrow:

```text
Order #1002 created successfully
```

A locator depending on:

```text
Order #1001 created successfully
```

is not reusable.

The automation engineer must distinguish:

```text
Stable Text
```

from:

```text
Dynamic Data
```

---

# Dynamic Text Strategy

Break the requirement into:

```text
Stable Business Meaning
+
Dynamic Data
```

For example:

```text
Order #1001
```

contains:

```text
Order
+
1001
```

The framework should decide whether the test needs:

```text
"Order"
```

or:

```text
"1001"
```

or:

```text
"Order #1001"
```

---

# Dynamic Text and Business Requirements

Suppose the requirement says:

> Verify that the application displays the created order number.

Then the locator strategy should be driven by:

```text
Expected Order ID
```

not by a hard-coded previous order ID.

This is where:

```text
Test Data
+
Locator Strategy
```

must work together.

---

# Whitespace

Web text can contain whitespace that is not obvious visually.

For example:

```text
"Save     Order"
```

may visually appear as:

```text
Save Order
```

There can also be:

```text
Line Breaks
Tabs
Multiple Spaces
Indentation
```

---

# Why Whitespace Matters

HTML formatting can introduce:

```text
Newlines
Spaces
Indentation
```

without changing what the user sees.

Therefore, text matching should not be designed around accidental source-code formatting.

---

# Text Normalization

Playwright's text matching behavior accounts for whitespace in ways intended to make user-facing text matching practical.

The important engineering principle is:

> **Do not depend on meaningless formatting whitespace.**

Think in terms of:

```text
Human-Visible Meaning
```

rather than:

```text
HTML Source Formatting
```

---

# Case Sensitivity

Text matching can be affected by how the locator is configured.

For example:

```text
Orders
```

and:

```text
orders
```

are not necessarily equivalent in every matching context.

Do not casually assume that capitalization does not matter.

---

# Why Case Matters

Consider:

```text
Pending
pending
PENDING
```

These may represent:

```text
Same Business Concept
```

but text matching may still need deliberate handling.

---

# Regular Expressions

Text locators can also be designed around patterns.

For example, conceptually:

```text
Order #<dynamic number>
```

can be represented as:

```text
Stable Pattern
+
Variable Data
```

Regular expressions can be useful when:

```text
Dynamic Values
```

follow a predictable structure.

---

# Regular Expression Mental Model

Suppose text is:

```text
Order #12345
```

and later:

```text
Order #98431
```

The pattern is:

```text
Order #
+
digits
```

This is more reusable than hard-coding:

```text
Order #12345
```

---

# Warning About Regular Expressions

Regular expressions are powerful but should not become unnecessarily complicated.

Bad:

```text
Huge Pattern
+
Multiple Conditions
+
Implementation Details
```

Good:

```text
Simple Stable Business Pattern
```

---

# Nested Text

Text may be spread across nested elements.

For example conceptually:

```text
<button>
    <span>Save</span>
    <span>Order</span>
</button>
```

A user sees:

```text
Save Order
```

even though the words are represented through multiple child elements.

---

# Why Nested Text Matters

You should think:

```text
Rendered Meaning
```

rather than:

```text
One Literal Text Node
```

when evaluating a text locator.

---

# Icon + Text

A common enterprise UI pattern is:

```text
[icon] Orders
```

The visible label may contain:

```text
Icon
+
Orders
```

The automation engineer should identify the meaningful user-facing text:

```text
Orders
```

rather than depending on the icon implementation.

---

# Text Inside Buttons

Suppose:

```text
[✓] Approve
```

The button may contain:

```text
Icon
+
Text
```

The actual accessible name may be:

```text
Approve
```

In such cases:

```text
getByRole()
```

may be more semantically appropriate than `getByText()`.

---

# Text Inside Cards

Consider:

```text
Product Card

Laptop Pro
₹75,000
In Stock
Add to Cart
```

A page may contain dozens of cards.

A global text locator for:

```text
Add to Cart
```

is ambiguous.

Instead:

```text
Product Card
 ↓
Laptop Pro
 ↓
Add to Cart
```

provides component scope.

---

# Text Inside Tables

Consider:

```text
Order ID | Customer | Status
1001     | John     | Pending
1002     | Mary     | Completed
```

The text:

```text
Pending
```

may appear in many rows.

The requirement might be:

```text
Verify Order 1001 is Pending
```

Therefore:

```text
Order 1001 Row
 ↓
Pending
```

is better than:

```text
Page
 ↓
Pending
```

---

# Text Inside Dialogs

Suppose the main page contains:

```text
Delete
```

and a dialog contains:

```text
Delete Order?
Cancel
Delete
```

A global text locator for:

```text
Delete
```

may become ambiguous.

Instead:

```text
Dialog
 ↓
Delete
```

provides a clear scope.

---

# Scope

Scope means:

```text
Limit Search To A Specific Region
```

Conceptually:

```text
Page
 ↓
Orders Section
 ↓
Text "Pending"
```

instead of:

```text
Entire Page
 ↓
Text "Pending"
```

---

# Why Scope Is Important

Enterprise applications often contain repeated business terminology.

Examples:

```text
Orders
Edit
Delete
Save
Pending
Active
Completed
View
```

Repeated text is normal.

Scope makes the text meaningful.

---

# Text + Component Strategy

A powerful pattern is:

```text
Component
 ↓
Business Text
 ↓
Action
```

For example:

```text
Order Card
 ↓
Order #1001
 ↓
Approve
```

This expresses:

```text
Which component?
Which record?
Which action?
```

---

# Text Filtering

Text can also be used to filter collections.

Conceptually:

```text
All Order Cards
      ↓
Cards containing "Order #1001"
      ↓
Target Card
```

This is often better than searching globally.

---

# `hasText`

`hasText` is especially useful for component filtering.

Mental model:

```text
Find component
WHERE
component contains specific text
```

For example:

```text
Order row
WHERE
row contains "1001"
```

Then:

```text
Within row
Find Approve
```

---

# `getByText()` and `hasText` Are Related but Different

Think:

```text
getByText()
```

as:

```text
Find element based on text
```

while:

```text
hasText
```

is:

```text
Filter another locator based on contained text
```

This distinction becomes very important in complex component trees.

---

# Repeated Text

Suppose:

```text
Edit
Edit
Edit
Edit
Edit
```

appear in five rows.

The text:

```text
Edit
```

does not identify the business target.

You need:

```text
Specific Row
+
Edit
```

---

# Why Global Text Is Dangerous

Global text matching assumes:

```text
Text
=
Identity
```

But in enterprise applications:

```text
Text
+
Context
=
Identity
```

This is one of the most important lessons in locator engineering.

---

# Text and Localization

Localization can change text.

For example:

```text
English:
Orders
```

might become:

```text
Hindi:
ऑर्डर
```

or another localized representation.

If your test suite runs across languages, a hard-coded English text locator can fail.

---

# Internationalization

Internationalization, commonly called i18n, refers to designing an application so it can support multiple languages and regional settings.

Automation must account for:

```text
Language
Currency
Date Format
Number Format
Text Direction
```

---

# Localization

Localization, commonly called l10n, is the adaptation of the application for a particular language or locale.

For example:

```text
English
Hindi
French
German
Japanese
```

may produce different visible text.

---

# Localization and Text Locators

If you write:

```text
getByText("Orders")
```

you have implicitly created:

```text
English Dependency
```

That may be acceptable for:

```text
English-only Test Suite
```

but not necessarily for:

```text
Multi-Locale Test Suite
```

---

# Enterprise Localization Strategy

For multi-language applications, consider:

```text
Stable Test IDs
+
Semantic Roles
+
Accessible Labels
+
Locale-Aware Test Data
```

rather than relying exclusively on English text.

---

# Translation Changes

Even within one language, product teams may change wording:

```text
"Submit Order"
```

to:

```text
"Place Order"
```

This may be a legitimate product change.

Text locators will correctly reveal this change.

But if dozens of tests depend on the same text, maintenance cost can become high.

---

# Centralizing Text

Some frameworks maintain centralized constants or localization resources for known application labels.

Conceptually:

```text
Application Text
      ↓
Central Definition
      ↓
Tests
```

This can reduce duplication.

However, over-centralization can also hide the business meaning of a test.

---

# Test Readability vs Centralization

Bad abstraction:

```text
clickText(CONSTANT_17)
```

A reviewer has no idea what the test is doing.

Better conceptual readability:

```text
Click the "Place Order" action
```

The framework should preserve business meaning.

---

# Text Locators and Assertions

Text is especially powerful for assertions.

Examples:

```text
Payment Successful

Order Created

No Results Found

Invalid Password

Account Locked
```

These are user-facing outcomes.

---

# Why Text Assertions Are Valuable

A UI test should often verify what the user experiences.

For example:

```text
Submit Payment
      ↓
Application Processes
      ↓
"Payment Successful"
```

The message itself is part of the user experience.

---

# Text vs API Assertions

A mature test strategy may combine:

```text
API Assertion
+
UI Assertion
```

For example:

```text
API confirms order created
+
UI displays "Order Created"
```

This gives stronger coverage.

---

# Text Locator and Business Language

Good text locators often align with product language:

```text
Order Created
Payment Failed
Pending
Approved
Cancelled
```

This makes tests easier for product and QA teams to understand.

---

# Dynamic Notifications

Notifications may contain:

```text
Order 1001 created successfully
```

or:

```text
Successfully created
```

The test should determine whether it needs:

```text
Exact Message
```

or:

```text
Business Signal
```

before choosing the locator strategy.

---

# Toast Messages

Enterprise applications frequently use toast notifications.

Conceptually:

```text
Page
 ↓
Toast
 ↓
"Order created successfully"
```

A text locator may be useful.

But the toast may:

```text
Disappear
Move
Stack
Re-render
```

Therefore synchronization and timing must be considered.

---

# Transient Text

Transient text means content that appears temporarily.

Examples:

```text
Saving...
Processing...
Loading...
Success
```

These elements may appear for only a short time.

A text locator can identify them, but the test must understand:

```text
When should it appear?
How long should it remain?
What event causes it?
```

---

# Text and Synchronization

Do not solve transient text problems with arbitrary sleeps.

Prefer:

```text
Wait for expected state
```

rather than:

```text
Wait 3 seconds
```

Synchronization will be covered deeply in a later stage.

---

# Text and Hidden Elements

Applications may contain text that exists in the DOM but is not meaningfully visible to the user.

Therefore:

```text
Text Exists
```

does not automatically mean:

```text
User Sees Text
```

This distinction is important for reliable assertions.

---

# Text and Responsive UI

Mobile and desktop versions may use different labels.

For example:

```text
Desktop:
"Create New Order"
```

Mobile:

```text
"+"
```

or:

```text
"Create"
```

A text locator designed for one viewport may not work for another.

---

# Responsive Testing Strategy

Instead of forcing one text locator across all experiences, determine:

```text
Shared Business Meaning
```

and use:

```text
Role
+
Accessible Name
+
Responsive Scope
```

where appropriate.

---

# Text and Icons

Icons can create misleading visual interpretation.

For example:

```text
[trash icon]
```

may have no visible text.

A text locator cannot identify something that has no text.

This is another situation where:

```text
Role
+
Accessible Name
```

or:

```text
Test ID
```

may be better.

---

# Text Locator Decision Framework

Before using `getByText()`, ask:

```text
Is the text visible?
        ↓
Is it stable?
        ↓
Is it unique?
        ↓
Is it localized?
        ↓
Is it inside a repeated component?
        ↓
Does role provide stronger semantics?
        ↓
Does a stable test ID exist?
        ↓
Choose strategy
```

---

# Architecture

```text
                         Business Requirement
                                  │
                                  ▼
                              Text Signal
                                  │
                  ┌───────────────┼───────────────┐
                  ▼               ▼               ▼
               Stable          Dynamic         Repeated
                  │               │               │
                  ▼               ▼               ▼
             getByText       Pattern/Scope      Scope
                  │               │               │
                  └───────────────┼───────────────┘
                                  ▼
                              Locator
                                  │
                                  ▼
                          Action / Assertion
```

---

# Workflow

```text
Requirement
    ↓
Identify User-Facing Text
    ↓
Check Uniqueness
    ↓
Check Stability
    ↓
Check Localization
    ↓
Check Semantic Role
    ↓
Check Component Scope
    ↓
Build Text Locator
    ↓
Validate Against Real UI
    ↓
Use in Action / Assertion
```

---

# Enterprise Workflow

```text
Product Requirement
        ↓
UI Component
        ↓
User-Facing Text
        ↓
Accessibility Semantics
        ↓
Locator Strategy
        ↓
Scope / Filter
        ↓
Playwright
        ↓
Action / Assertion
        ↓
Test Result
```

---

# Enterprise Perspective

Large organizations rarely want automation that is tightly coupled to visual implementation.

A strong enterprise strategy treats text as:

```text
Business Contract
```

when the text is intentionally stable.

But organizations must also recognize that text is often:

```text
Localization-Sensitive
Product-Change-Sensitive
Content-Sensitive
```

Therefore:

```text
Text
```

should be one tool within the locator strategy, not the only tool.

---

# Enterprise Locator Matrix

| UI Situation | Preferred Strategy |
|---|---|
| Unique static message | Text |
| Unique heading | Role + name |
| Button | Role + name |
| Link | Role + name |
| Form field | Label |
| Repeated card text | Scoped text |
| Table record | Row/component + text |
| Dynamic message | Pattern or stable scope |
| Localized content | Role/Test ID where appropriate |
| Icon-only control | Role + accessible name/Test ID |
| Complex component boundary | Test ID |
| Ambiguous repeated text | Scope + filter |

---

# Best Practices

1. Use `getByText()` when text itself is a meaningful identifier.
2. Prefer role-based locating for semantic interactive controls when appropriate.
3. Check whether the text is unique.
4. Use exact matching when ambiguity requires it.
5. Be careful with partial matching.
6. Avoid hard-coding dynamic values unnecessarily.
7. Separate stable text from dynamic data.
8. Use component scope for repeated text.
9. Use filtering for repeated cards and rows.
10. Consider localization before building text-dependent suites.
11. Use stable test IDs where text is inherently dynamic or localized.
12. Use text assertions for meaningful user-facing messages.
13. Avoid arbitrary waits for transient text.
14. Consider responsive variations.
15. Do not assume visible text and accessible name are identical.
16. Avoid giant regular expressions.
17. Keep text-based locators readable.
18. Review text locators during code review.
19. Treat product wording changes as intentional locator changes when appropriate.
20. Prefer business meaning over DOM structure.

---

# Common Beginner Mistakes

## Mistake 1 — Using Text Everywhere

Not every element is best identified by text.

---

## Mistake 2 — Assuming Text Is Unique

Enterprise applications frequently repeat:

```text
Edit
Delete
Save
View
Orders
```

---

## Mistake 3 — Ignoring Scope

Global text matching can produce ambiguity.

---

## Mistake 4 — Hard-Coding Dynamic Text

For example:

```text
Order #12345
```

when the order number changes.

---

## Mistake 5 — Ignoring Localization

English text may not exist in every environment.

---

## Mistake 6 — Using Partial Matching Accidentally

Searching for:

```text
Save
```

may match:

```text
Save Draft
Save Order
Save Changes
```

---

## Mistake 7 — Confusing Text With Accessible Name

A button can have an accessible name derived from more than simple visible text.

---

## Mistake 8 — Using Text Instead of Role for Buttons

If the requirement is:

```text
Click the Delete button
```

a role-based locator may express intent better.

---

## Mistake 9 — Depending on Formatting Whitespace

HTML indentation should not become a test dependency.

---

## Mistake 10 — Using Complex Regex to Compensate for Bad Locators

If a simple scope or role would solve the problem, do not create an enormous regular expression.

---

# Professional Tips

Before writing a text locator, ask:

```text
What does the user see?

↓

Is this text intentionally stable?

↓

Could the product team change the wording?

↓

Could localization change it?

↓

Could the same text appear elsewhere?

↓

Does the element have a stronger semantic role?

↓

Can I scope it to the correct component?
```

A particularly powerful professional habit is:

> **Use text to identify business meaning, but use context to identify business identity.**

For example:

```text
"Approve"
```

may identify the action.

But:

```text
Order #1001
    ↓
Approve
```

identifies the business operation.

---

# Real Interview Discussion

## Junior-Level

### Q1. What is `getByText()`?

**Answer:**

`getByText()` is a Playwright locator API used to locate elements based on text content.

---

### Q2. When would you use `getByText()`?

**Answer:**

I would use it when the visible text itself is a meaningful and sufficiently stable identifier, especially for user-facing messages, labels, status information, or other content where text is the actual business signal.

---

### Q3. Is `getByText()` always better than CSS?

**Answer:**

No. Locator selection depends on stability and intent. `getByText()` is useful when text represents stable meaning, but a role locator or stable test ID may be better for certain elements.

---

# Mid-Level

### Q4. What happens if text appears multiple times?

**Answer:**

The locator may match multiple elements. For operations requiring one target, this can lead to a strictness violation. I would refine the locator using scope, role, filtering, or another stable identifier rather than blindly selecting a position.

---

### Q5. How do you handle dynamic text?

**Answer:**

I separate stable business text from dynamic data. Depending on the requirement, I may use a pattern, component scope, expected dynamic data, or a stable test contract.

---

### Q6. What is the difference between `getByText()` and `hasText`?

**Answer:**

`getByText()` identifies elements based on text. `hasText` is commonly used to filter an existing locator based on text contained within the matched component.

---

# Senior-Level

### Q7. Why can text locators become problematic in multilingual applications?

**Answer:**

Because visible text changes by locale. A locator hard-coded to English can fail in another language. For multi-locale suites, I would prefer stable semantic or test contracts where appropriate and keep localization-specific tests separate.

---

### Q8. How would you locate an action inside a specific table row?

**Answer:**

I would identify the row using stable business data, such as an order ID, then scope the action locator within that row. This prevents repeated action text from becoming ambiguous.

---

### Q9. How do you distinguish text from accessible name?

**Answer:**

Visible text is what is rendered to the user, while accessible name is the semantic name exposed through the accessibility model. They can be related but are not guaranteed to be identical.

---

# Lead-Level

### Q10. How would you create an enterprise standard for text locators?

**Answer:**

I would define that text should be used when it represents stable business meaning. Interactive controls should generally prefer semantic role and accessible name. Repeated text should be scoped to the relevant component. Dynamic and localized text should use appropriate stable contracts, and text locator quality should be part of code review standards.

---

### Q11. How would you prevent text locator duplication across hundreds of tests?

**Answer:**

I would avoid blindly centralizing every piece of text. I would establish reusable component abstractions for repeated UI structures and use application-level localization or stable test contracts where appropriate. The goal is to preserve readability while reducing maintenance duplication.

---

# Architect-Level

### Q12. What is the architectural risk of relying heavily on text locators?

**Strong Answer:**

Text is a business-facing contract, but it is also vulnerable to product wording changes, localization, content changes, and duplication. An enterprise framework should therefore use text strategically rather than universally. Semantic roles, accessible names, stable test IDs, component scope, and business identifiers should complement text-based locating.

---

# Knowledge Check

Answer these questions before proceeding:

1. What is `getByText()`?
2. What is text content?
3. What is a text node?
4. Why is visible text useful for automation?
5. Why is text not always unique?
6. What is exact matching?
7. What is partial matching?
8. Why can partial matching be dangerous?
9. How does whitespace affect text?
10. Why should formatting whitespace not be a test dependency?
11. What is dynamic text?
12. How should dynamic text be handled?
13. What are regular expressions useful for?
14. Why should regex remain simple?
15. What is nested text?
16. Why is nested text important?
17. What is component scope?
18. Why is scope important for repeated text?
19. What is the difference between `getByText()` and `hasText`?
20. Why is text useful inside assertions?
21. Why can localization break text locators?
22. What is internationalization?
23. What is localization?
24. Why might a role locator be better than a text locator for a button?
25. Why can repeated text cause strictness errors?
26. How would you locate "Approve" for Order #1001?
27. How would you handle a localized application?
28. How would you handle dynamic toast text?
29. When should a test ID be preferred?
30. What makes a text locator enterprise-grade?

---

# Step Summary

You have now learned that text is powerful because it represents:

```text
User Meaning
+
Business Language
```

But text is not automatically unique or stable.

The correct mental model is:

```text
Text
 ↓
Stable?
 ↓
Unique?
 ↓
Visible?
 ↓
Localized?
 ↓
Dynamic?
 ↓
Semantic Role Available?
 ↓
Component Scope Required?
 ↓
Choose Locator
```

The most important enterprise principle is:

> **Text can identify business meaning, but context often identifies business identity.**

For example:

```text
Approve
```

identifies an action.

But:

```text
Order #1001
   ↓
Approve
```

identifies the specific business operation.

You also learned that:

```text
getByText()
```

is particularly useful for:

```text
Messages
Statuses
Headings
Labels
Business Content
Notifications
```

while:

```text
getByRole()
```

is often stronger for:

```text
Buttons
Links
Checkboxes
Radio Buttons
Tabs
Dialogs
Navigation
```

And for repeated components:

```text
Component
 ↓
Text Filter
 ↓
Target
```

is generally more reliable than:

```text
Page
 ↓
Global Text
```

---

# Progress Milestone

✅ **Step 61 completed.**

You now understand:

- `getByText()`
- Text nodes
- Visible text
- Text content
- Exact matching
- Partial matching
- Whitespace
- Case considerations
- Dynamic text
- Nested text
- Repeated text
- Component scope
- `hasText`
- Text filtering
- Regular-expression concepts
- Text assertions
- Toast messages
- Transient text
- Localization
- Internationalization
- Responsive text
- Text versus accessible name
- Text versus role
- Enterprise text-locator strategy

Your locator decision process should now look like:

```text
Business Requirement
        ↓
What does the user recognize?
        ↓
Role?
        ↓
Accessible Name?
        ↓
Text?
        ↓
Stable Test ID?
        ↓
Business Attribute?
        ↓
Scope?
        ↓
Filter?
        ↓
Final Locator
```

---

# Next Step

# Step 62 — Mastering `getByLabel()`: Form Semantics, Label Association, Explicit Labels, Implicit Labels, Input Types, Checkboxes, Radio Buttons, Select Controls, Custom Forms, Accessible Names, Validation Messages, Dynamic Forms, and Enterprise Form Locator Strategy

The next lesson moves into one of the most important areas in business applications:

```text
Forms
```

You will learn how to reliably locate:

```text
Username
Password
Email
Phone
Address
Checkboxes
Radio Buttons
Dropdowns
Date Fields
Search Fields
Dynamic Form Controls
```

using the relationship between:

```text
Label
   ↓
Form Control
   ↓
Accessible Name
   ↓
Playwright Locator
```

The central idea will be:

```text
Do not locate the input merely because you can see an input.

Locate the input because you understand what business field it represents.
```
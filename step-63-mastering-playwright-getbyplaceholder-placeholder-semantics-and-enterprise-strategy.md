# Playwright with TypeScript Mastery Bootcamp

## From Absolute Beginner to Enterprise Automation Architect

# Stage 29 — Locator Engineering

# Step 63 — Mastering `getByPlaceholder()`: Placeholder Semantics, Placeholder vs Label, Dynamic Placeholders, Search Inputs, Localization, Accessibility Risks, Strictness, and Enterprise Placeholder Strategy

---

# Objective

In the previous lesson, you learned:

```text
getByLabel()
```

and understood why form controls should ideally be identified by their business meaning.

Now we study:

```text
getByPlaceholder()
```

A placeholder is one of the most visible hints inside a form field.

Examples:

```text
[ Enter your email ]

[ Search orders ]

[ Type customer name ]

[ DD/MM/YYYY ]

[ Enter phone number ]
```

Because these strings are visible, beginners often use them as the primary locator.

That can work.

But it is important to understand:

> A placeholder is not the same thing as a label.

This distinction becomes extremely important in enterprise automation.

You will learn:

- What a placeholder is
- What `getByPlaceholder()` does
- Placeholder versus label
- Placeholder versus accessible name
- Why placeholders exist
- When placeholders are useful
- When placeholders are unstable
- Exact placeholder matching
- Partial matching
- Dynamic placeholders
- Search fields
- Forms
- Optional fields
- Password fields
- Date fields
- Masked inputs
- Localization
- Responsive applications
- Accessibility concerns
- Duplicate placeholders
- Component scoping
- Placeholder changes
- Design-system components
- Enterprise locator strategy
- When to prefer `getByLabel()`
- When to prefer `getByPlaceholder()`
- When to prefer `getByRole()`
- When to prefer `getByTestId()`

---

# Before We Start

Imagine a hotel check-in form.

You see:

```text
Guest Name
[ Enter full name ]

Room Number
[ Enter room number ]

Special Requests
[ Type your request... ]
```

The text:

```text
Guest Name
```

tells you:

> What is this field?

The text:

```text
Enter full name
```

tells you:

> What should I type here?

These are different concepts.

Think of it this way:

```text
Label
   ↓
Identity

Placeholder
   ↓
Instruction
```

This distinction is the foundation of this lesson.

---

# The Problem

Consider:

```text
Email Address
[ Enter your email address ]
```

A beginner may say:

```text
The field contains "Enter your email address".

I will locate it using that text.
```

This can work.

But imagine the product team changes it to:

```text
[ example@company.com ]
```

The business field is still:

```text
Email Address
```

but the placeholder changed.

Your automation now fails even though the business functionality did not change.

---

# What is a Placeholder?

A placeholder is temporary instructional or example text displayed inside a form control before the user provides a value.

Conceptually:

```text
Input
 └── Placeholder
       ↓
    Instruction
```

Examples:

```text
Enter email
Search orders
Enter amount
Type message
```

---

# What is `getByPlaceholder()`?

`getByPlaceholder()` is a Playwright locator API used to locate form controls based on their placeholder attribute/value.

Conceptually:

```text
Placeholder
     ↓
Matching Form Control
     ↓
Locator
```

The locator is based on:

```text
placeholder
```

rather than:

```text
label
role
id
class
```

---

# Placeholder Mental Model

Think:

```text
Form Control
     │
     ├── Label
     │      ↓
     │   Identity
     │
     └── Placeholder
            ↓
         Hint
```

This distinction is extremely important.

---

# Label vs Placeholder

Consider:

```text
Email Address
[ Enter your email ]
```

### Label

```text
Email Address
```

means:

```text
This field represents the user's email address.
```

### Placeholder

```text
Enter your email
```

means:

```text
Here is a hint about what to enter.
```

Therefore:

```text
Label
=
Field Identity
```

while:

```text
Placeholder
=
Input Guidance
```

---

# Why This Difference Matters

A field may keep the same business identity while changing its instructional text.

For example:

```text
Label:
Email Address
```

could have:

```text
Placeholder:
Enter email

```

then later:

```text
Placeholder:
name@example.com
```

then:

```text
Placeholder:
Work email address
```

The field remains:

```text
Email Address
```

Therefore the label is generally the stronger business contract.

---

# Placeholder and Accessible Name

Another important distinction:

```text
Placeholder
≠
Accessible Name
```

An accessible name identifies the control semantically.

A placeholder is generally supplementary instructional text.

Do not assume:

```text
placeholder = field name
```

---

# Why Labels Are Usually Better

Consider:

```text
Email Address
[ example@company.com ]
```

The label:

```text
Email Address
```

remains meaningful even after the user types:

```text
sandeep@example.com
```

The placeholder may disappear.

Therefore:

```text
Label
```

provides persistent identity.

---

# Placeholder Disappears

This is a critical property.

Suppose:

```text
Email Address
[ Enter email ]
```

The user types:

```text
john@example.com
```

Now:

```text
Email Address
[ john@example.com ]
```

The placeholder is no longer visible.

The field's identity remains:

```text
Email Address
```

---

# Placeholder Is Not User Data

A placeholder is not the value entered into the field.

Conceptually:

```text
Placeholder
    ↓
Instruction

Value
    ↓
Actual User Data
```

For example:

```text
Placeholder:
Enter phone number

Value:
9876543210
```

These are completely different concepts.

---

# Placeholder and Form State

A form control can move through:

```text
Empty
   ↓
Placeholder visible
   ↓
User enters value
   ↓
Placeholder disappears
   ↓
Actual value visible
```

This means placeholder-based reasoning is closely connected to form state.

---

# When `getByPlaceholder()` Is Useful

It can be useful when:

```text
The application intentionally exposes a stable placeholder
+
The placeholder uniquely identifies the field
+
The placeholder is part of the UI contract
```

Examples may include:

```text
Search
[ Search orders ]

Filter
[ Filter products ]

Command input
[ Type a command ]

Quick search
[ Search by order ID ]
```

---

# Search Inputs

Search boxes are a common use case.

For example:

```text
[ Search orders ]
```

If there is no visible label and the placeholder is intentionally stable, `getByPlaceholder()` can be practical.

However, you should still consider whether the control has an accessible name.

---

# Search Example

Suppose the application has:

```text
[ Search orders ]
```

and this is the only search input on the page.

The placeholder:

```text
Search orders
```

may provide a clear locator.

But if the page later adds:

```text
[ Search customers ]
```

and:

```text
[ Search products ]
```

then each placeholder must be evaluated for uniqueness.

---

# Placeholder Uniqueness

A locator should ideally identify the intended control uniquely.

Suppose:

```text
[ Enter name ]
[ Enter name ]
[ Enter name ]
```

appears in three components.

The placeholder alone is insufficient.

You need:

```text
Component
   ↓
Placeholder
```

---

# Component Scope

Imagine:

```text
Billing Information
[ Enter name ]

Shipping Information
[ Enter name ]
```

The placeholder is identical.

The business context is different.

Therefore:

```text
Billing Component
     ↓
Enter name

Shipping Component
     ↓
Enter name
```

Scope provides uniqueness.

---

# Placeholder in Reusable Components

Enterprise applications often reuse design-system components.

For example:

```text
SearchInput
```

may appear in:

```text
Orders
Customers
Products
Reports
```

Each component may use:

```text
Search...
```

as its placeholder.

Global placeholder matching is then ambiguous.

A component-oriented framework should scope the locator.

---

# Dynamic Placeholders

Some applications dynamically change placeholders.

For example:

```text
Search by order ID
```

may become:

```text
Search by customer name
```

depending on the selected search mode.

This means the placeholder represents:

```text
Current UI State
```

rather than a permanent field identity.

---

# Dynamic Placeholder Strategy

If the placeholder changes based on state:

```text
Search Mode
   ↓
Placeholder
   ↓
Input
```

the test should first establish the desired mode.

Then locate the corresponding field.

The automation should not assume the placeholder is permanently fixed.

---

# Placeholder and Product Changes

A product team may change:

```text
Enter email
```

to:

```text
Enter your email address
```

No functionality changed.

But:

```text
getByPlaceholder("Enter email")
```

may fail.

This demonstrates an important principle:

> **Placeholder text can be a presentation-level contract rather than a business-level contract.**

---

# Product Copy Changes

Text used for:

```text
Marketing
UX Guidance
Examples
Instructions
```

often changes more frequently than:

```text
Business Field Identity
```

Therefore placeholder locators may have a higher maintenance cost.

---

# Placeholder and Localization

Imagine:

```text
English:
Search orders
```

German:

```text
Bestellungen suchen
```

Hindi:

```text
ऑर्डर खोजें
```

A placeholder-based locator using English text will not work across all locales.

---

# Multi-Locale Enterprise Automation

For applications supporting multiple languages, consider:

```text
Role
+
Accessible Name
+
Stable Test ID
+
Locale-Aware Data
```

rather than relying entirely on placeholder strings.

---

# Placeholder and Responsive Design

Desktop:

```text
[ Search orders by ID or customer ]
```

Mobile:

```text
[ Search... ]
```

The same business field now has different placeholder text.

A test suite that assumes identical placeholder text across viewports can become fragile.

---

# Placeholder and Accessibility

This is a particularly important topic.

A placeholder should not generally be treated as a replacement for a proper persistent label.

Why?

Because placeholder text can:

```text
Disappear
Have low visual contrast
Be difficult to interpret after input
Be translated
Change dynamically
```

A persistent label gives the user a stable description of the field.

---

# Accessibility Architecture

A good form conceptually provides:

```text
Persistent Label
       ↓
Accessible Name
       ↓
Form Control
       ↓
Optional Placeholder
       ↓
Instruction / Example
```

Not:

```text
Placeholder
       ↓
Everything
```

---

# Placeholder Contrast

Some designs use light placeholder text.

This can create readability problems.

Automation engineers should not try to solve accessibility problems by choosing a locator based on placeholder text.

Instead, recognize:

```text
Locator Strategy
```

and:

```text
Accessibility Quality
```

as related but separate concerns.

---

# Placeholder vs Label Decision

Use this question:

```text
Does this text identify the field?
```

If yes:

```text
Label
```

is likely the stronger concept.

Ask:

```text
Does this text tell me what or how to enter?
```

If yes:

```text
Placeholder
```

is likely what you are looking at.

---

# Placeholder and Password Fields

Consider:

```text
Password
[ Enter password ]
```

The placeholder can be useful.

But:

```text
Password
```

is the stronger field identity.

A robust enterprise strategy generally prefers the semantic identity of the control.

---

# Placeholder and Search

Search fields are one of the more reasonable places to use placeholders.

Example:

```text
[ Search orders ]
```

If there is no persistent label and the placeholder is intentionally part of the UI contract, `getByPlaceholder()` can be useful.

But accessibility should still be considered.

---

# Placeholder and Date Inputs

Consider:

```text
Date of Birth
[ DD/MM/YYYY ]
```

The placeholder:

```text
DD/MM/YYYY
```

communicates format.

It does not mean:

```text
This is the Date of Birth field.
```

The label provides that identity.

---

# Placeholder and Currency Inputs

Consider:

```text
Amount
[ 0.00 ]
```

The placeholder communicates:

```text
Expected format
```

while:

```text
Amount
```

communicates:

```text
Business meaning
```

---

# Placeholder and Masked Inputs

Phone numbers may display:

```text
[ +91 XXXXX XXXXX ]
```

or:

```text
[ (###) ###-#### ]
```

These placeholders communicate formatting.

They are often not ideal primary identities.

---

# Placeholder and Autocomplete

Autocomplete fields may display:

```text
[ Search customer ]
```

Then dynamically show:

```text
John Smith
John Samuel
Johnny Doe
```

The placeholder identifies the search purpose.

The result options require a different locator strategy.

---

# Placeholder and Empty State

Placeholder is generally meaningful when:

```text
Field is empty
```

Once the user enters a value:

```text
Placeholder disappears
```

Therefore tests should not confuse placeholder text with expected field value.

---

# Placeholder Assertions

You may sometimes need to verify:

```text
The search field has the expected placeholder.
```

This is different from:

```text
The field contains the expected value.
```

For example:

```text
Placeholder:
Search orders

Value:
ORD-1001
```

These represent two different assertions.

---

# Placeholder as Product Contract

There are situations where placeholder text is intentionally part of the product requirement.

For example:

```text
Requirement:
Search box must display "Search by Order ID".
```

In that situation, validating the placeholder is legitimate.

The key is:

```text
Intentional Contract
```

rather than:

```text
Convenient Selector
```

---

# Placeholder Locator vs Placeholder Assertion

This distinction is valuable.

### Locator

```text
Use placeholder to find the field.
```

### Assertion

```text
Verify the field's placeholder is correct.
```

The second is explicitly testing UI copy.

---

# Duplicate Placeholders

Suppose an application contains:

```text
Search orders
Search orders
```

The placeholder is not unique.

Do not immediately use:

```text
nth()
```

Instead ask:

```text
Which component?
Which business context?
Which page region?
```

---

# Scope Strategy

Conceptually:

```text
Orders Page
   ↓
Search Section
   ↓
"Search orders"
```

versus:

```text
Entire Page
   ↓
"Search orders"
```

The first is more intentional.

---

# Placeholder and Test IDs

Sometimes a design-system component has:

```text
Placeholder:
Search...
```

across dozens of screens.

A stable test ID may be a better component contract.

For example:

```text
SearchInput
   ↓
Stable Automation Contract
```

This avoids making every test depend on UI copy.

---

# Locator Decision Matrix

| Situation | Preferred Strategy |
|---|---|
| Field has a stable label | `getByLabel()` |
| Button with meaningful action | `getByRole()` |
| Unique static visible content | `getByText()` |
| Search field identified only by stable placeholder | `getByPlaceholder()` |
| Complex reusable component | Test ID / component contract |
| Localized placeholder | Locale-aware strategy |
| Dynamic placeholder | State-aware locator |
| Repeated placeholder | Component scope |
| Placeholder is only an example | Prefer label |
| Placeholder is product requirement | Placeholder may be appropriate |

---

# Placeholder Locator Architecture

```text
                       Form Control
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
           Label                       Placeholder
             │                             │
             ▼                             ▼
        Field Identity                 Input Hint
             │                             │
             └──────────────┬──────────────┘
                            ▼
                    Locator Decision
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       Label             Placeholder        Test ID
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                         Playwright
                            │
                            ▼
                      Action / Assertion
```

---

# Workflow

```text
Business Requirement
        ↓
Identify Form Field
        ↓
Check Persistent Label
        ↓
Check Accessible Name
        ↓
Check Placeholder
        ↓
Determine Stability
        ↓
Determine Uniqueness
        ↓
Check Localization
        ↓
Check Responsive Behavior
        ↓
Choose Locator
        ↓
Perform Action
        ↓
Verify Result
```

---

# Enterprise Workflow

```text
UX / Product Design
        ↓
Form Component
        ↓
Label + Accessibility
        ↓
Optional Placeholder
        ↓
Automation Contract
        ↓
Playwright Locator
        ↓
Business Action
        ↓
Validation
        ↓
Reporting
```

---

# Enterprise Perspective

At enterprise scale, the key question is not:

> "Can I use the placeholder?"

The question is:

> "Is the placeholder an appropriate long-term automation contract?"

Consider a large organization with:

```text
500 Screens
2,000 Forms
10,000 Inputs
Multiple Locales
Multiple Brands
Multiple Design Systems
```

If automation relies heavily on placeholders, a UX copy change can create widespread failures.

Therefore enterprise teams usually prefer a hierarchy based on semantic strength and stability.

---

# Enterprise Locator Hierarchy

A practical strategy may look like:

```text
1. Semantic Role + Accessible Name
2. Label
3. Stable Test ID
4. Stable User-Facing Text
5. Placeholder
6. CSS
7. XPath
8. Positional Selectors
```

This is not an absolute law.

The correct strategy depends on the application's architecture and the element's contract.

---

# Design System Perspective

Enterprise design systems often create reusable controls:

```text
TextInput
SearchInput
DateInput
CurrencyInput
Select
Autocomplete
```

Each component should ideally provide:

```text
Accessible Name
Stable Semantics
Predictable Behavior
Optional Test Contract
```

The automation framework should consume these contracts rather than reverse-engineering internal markup.

---

# Framework Governance

An enterprise automation architect can define rules such as:

```text
Rule 1:
Do not use placeholder when a stable label exists.

Rule 2:
Do not use placeholder as a substitute for accessibility.

Rule 3:
Scope repeated placeholders.

Rule 4:
Avoid placeholder locators for frequently changing copy.

Rule 5:
Use stable test contracts for complex shared components.

Rule 6:
Document intentional placeholder dependencies.
```

---

# Placeholder and Technical Debt

Suppose 300 tests use:

```text
"Search..."
```

Then UX changes it to:

```text
"Search here..."
```

Potential result:

```text
300 Broken Tests
```

This is automation technical debt.

A good architecture minimizes dependencies on low-stability UI details.

---

# How to Reduce Placeholder Technical Debt

Use:

```text
Stable Semantics
+
Component Scope
+
Test IDs Where Appropriate
+
Centralized Design Standards
+
Code Reviews
```

rather than blindly replacing every placeholder locator.

---

# Placeholder and Test Readability

Compare:

```text
getByPlaceholder("Enter customer email")
```

with:

```text
getByTestId("customer-email")
```

Which is better?

There is no universal answer.

If the placeholder itself clearly represents the business field and is stable, it may be readable.

If the placeholder is merely:

```text
Type here...
```

then it provides very little business meaning.

---

# Weak Placeholder

Consider:

```text
[ Type here... ]
```

This tells you almost nothing.

If five fields use:

```text
Type here...
```

the placeholder is useless as a locator.

---

# Strong Placeholder

Consider:

```text
[ Search orders by ID ]
```

This is much more descriptive.

But still ask:

```text
Is it stable?
Is it unique?
Is it localized?
```

---

# Placeholder Quality Spectrum

```text
Very Weak
"Type here..."

       ↓

Weak
"Enter value"

       ↓

Moderate
"Enter email"

       ↓

Strong
"Search orders by ID"

       ↓

Business Identity
"Order ID"
```

The last category is often better represented as a label.

---

# Professional Tips

When you encounter a placeholder-based locator, ask during code review:

```text
Why is placeholder being used?

Is there a label?

Is the placeholder stable?

Is it unique?

Could UX copy change?

Does it vary by locale?

Does it vary by viewport?

Is it dynamic?

Could a component-level test ID be more appropriate?
```

This turns locator selection into an engineering decision rather than personal preference.

---

# Common Beginner Mistakes

## Mistake 1 — Treating Placeholder as Label

```text
"Enter email"
```

is usually an instruction.

```text
"Email Address"
```

is field identity.

---

## Mistake 2 — Using Placeholder Everywhere

Placeholder should not become the default locator for every input.

---

## Mistake 3 — Ignoring Placeholder Disappearance

Once a value is entered, the placeholder may no longer be visible.

---

## Mistake 4 — Using Placeholder as the Expected Value

```text
Placeholder:
Enter email

Actual value:
john@example.com
```

They are not the same.

---

## Mistake 5 — Ignoring Localization

Placeholder strings may vary by language.

---

## Mistake 6 — Ignoring Responsive Changes

Mobile and desktop may use different placeholder copy.

---

## Mistake 7 — Using Generic Placeholders

```text
Type here...
```

is rarely a good unique locator.

---

## Mistake 8 — Ignoring Duplicate Placeholders

Repeated design-system components can use identical placeholders.

---

## Mistake 9 — Using Placeholder Instead of Proper Accessibility

A placeholder should not replace a persistent accessible label.

---

## Mistake 10 — Assuming Placeholder Stability

UX copy can change frequently.

---

# Best Practices

1. Prefer `getByLabel()` when a stable label exists.
2. Use `getByPlaceholder()` when the placeholder is intentionally stable and meaningful.
3. Do not treat placeholders as permanent field identity.
4. Do not use placeholders as a replacement for labels.
5. Check uniqueness.
6. Scope repeated placeholders.
7. Consider localization.
8. Consider responsive behavior.
9. Consider dynamic state.
10. Consider product-copy volatility.
11. Use placeholder assertions when placeholder text is itself a requirement.
12. Avoid generic placeholders.
13. Avoid depending on placeholders for complex shared components.
14. Use stable test IDs when appropriate.
15. Keep component boundaries clear.
16. Encourage accessible labels in application development.
17. Review placeholder locators during code review.
18. Separate field identity from input instructions.
19. Treat locator stability as an architectural concern.
20. Document intentional placeholder dependencies.

---

# Real Interview Discussion

## Junior-Level

### Q1. What is `getByPlaceholder()`?

**Answer:**

`getByPlaceholder()` is a Playwright locator API used to locate form controls based on their placeholder text.

---

### Q2. What is a placeholder?

**Answer:**

A placeholder is temporary instructional or example text displayed inside a form control, usually before the user enters a value.

---

### Q3. Is a placeholder the same as a label?

**Answer:**

No.

A label identifies the purpose of a field, while a placeholder generally provides an example or instruction for what to enter.

---

# Mid-Level

### Q4. When would you use `getByPlaceholder()`?

**Answer:**

I would use it when the placeholder is a stable, meaningful, unique part of the application's UI contract and there is no stronger semantic locator available or appropriate.

---

### Q5. Why is `getByLabel()` generally preferable when a label exists?

**Answer:**

Because the label represents the field's persistent semantic identity, while the placeholder is usually temporary instructional content.

---

### Q6. Why can placeholders cause flaky or maintenance-heavy tests?

**Answer:**

Because placeholders can change due to UX copy changes, localization, responsive design, or dynamic application state without changing the underlying business functionality.

---

# Senior-Level

### Q7. How would you handle a search field with only a placeholder?

**Answer:**

I would first verify that the placeholder is stable and unique and that the field has appropriate accessibility semantics. If the placeholder is the intended UI contract, I can use it. Otherwise, I would consider a label, role/name, or stable test contract.

---

### Q8. How would you handle duplicate placeholders?

**Answer:**

I would scope the locator to the relevant component or form section. I would avoid using positional selection unless position is genuinely part of the business requirement.

---

### Q9. How does localization affect placeholder locators?

**Answer:**

Localized applications can change placeholder text for each language, so a hard-coded English placeholder will not be portable across locales. I would use locale-aware data or a more stable semantic/test contract.

---

# Lead-Level

### Q10. Would you ban `getByPlaceholder()` in an enterprise framework?

**Answer:**

No. I would establish governance rather than a blanket ban. Placeholder locators are appropriate when the placeholder is intentionally stable, meaningful, unique, and represents the correct test contract. However, labels, roles, accessible names, and stable test IDs should generally be preferred when they provide stronger contracts.

---

### Q11. How would you reduce placeholder-related maintenance?

**Answer:**

I would encourage semantic labels, establish component contracts, use stable test IDs where justified, review locator choices during pull requests, and track locator failures caused by product-copy changes.

---

# Architect-Level

### Q12. What is the architectural difference between a label and a placeholder?

**Strong Answer:**

A label primarily establishes the semantic identity of a form control, while a placeholder provides temporary instructional or example content. From an automation architecture perspective, labels are generally a stronger business contract because they remain meaningful after the field receives a value and are less dependent on presentation copy. Placeholders can still be useful when they are intentionally designed as stable UI contracts, particularly for search and specialized inputs.

---

### Q13. How should an enterprise design system support automation?

**Strong Answer:**

Reusable form components should expose meaningful accessible names, proper label associations, predictable roles and states, and stable test contracts where semantic locators are insufficient. Automation should consume these public component contracts instead of depending on internal markup or volatile placeholder text.

---

# Knowledge Check

Answer these before proceeding:

1. What is a placeholder?
2. What does `getByPlaceholder()` do?
3. What is the difference between label and placeholder?
4. Why is a label generally more persistent?
5. Why does a placeholder disappear?
6. Why is placeholder not the same as field value?
7. When is `getByPlaceholder()` appropriate?
8. Why can placeholder text be unstable?
9. How can localization affect placeholders?
10. How can responsive design affect placeholders?
11. What is a dynamic placeholder?
12. How should dynamic placeholders be handled?
13. Why can duplicate placeholders cause ambiguity?
14. How does component scope solve duplicate placeholders?
15. Why should placeholder not replace a label?
16. What is the difference between placeholder and accessible name?
17. When might a search field be a good candidate for `getByPlaceholder()`?
18. Why is `"Type here..."` a weak placeholder locator?
19. Why can `"Search orders by ID"` be stronger?
20. When should a test ID be preferred?
21. How can product-copy changes affect placeholder locators?
22. Why is placeholder-based technical debt dangerous at enterprise scale?
23. How should design-system components support automation?
24. How would you handle a localized search field?
25. How would you handle two identical placeholders?
26. When would `getByLabel()` be better?
27. When would `getByRole()` be better?
28. When would `getByText()` be better?
29. When would `getByPlaceholder()` be better?
30. What makes a placeholder locator enterprise-grade?

---

# Step Summary

You have now learned that:

```text
Placeholder
≠
Label
```

The most important distinction is:

```text
Label
 ↓
"What is this field?"
```

while:

```text
Placeholder
 ↓
"What should I enter?"
```

A useful conceptual model is:

```text
Form Control
      │
      ├── Label
      │      ↓
      │   Persistent Identity
      │
      └── Placeholder
             ↓
          Temporary Hint
```

You also learned why placeholder-based locators can become fragile because placeholders may change due to:

```text
UX Copy Changes
Localization
Responsive Design
Dynamic State
Design-System Updates
Product Experiments
```

Therefore:

```text
getByPlaceholder()
```

is a useful tool, but it should not automatically become the primary strategy for every form field.

The enterprise decision should be:

```text
Stable Label?
   ↓
getByLabel()

Semantic Interactive Control?
   ↓
getByRole()

Stable User-Facing Content?
   ↓
getByText()

Meaningful Stable Placeholder?
   ↓
getByPlaceholder()

Stable Automation Contract?
   ↓
getByTestId()
```

The central enterprise principle is:

> **Use the strongest stable contract that represents the actual intent of the test.**

---

# Progress Milestone

✅ **Step 63 completed.**

You now understand:

- `getByPlaceholder()`
- Placeholder semantics
- Placeholder versus label
- Placeholder versus accessible name
- Temporary placeholder behavior
- Placeholder disappearance
- Search inputs
- Dynamic placeholders
- Duplicate placeholders
- Component scope
- Localization
- Responsive behavior
- Accessibility implications
- Product-copy changes
- Design-system components
- Placeholder assertions
- Placeholder technical debt
- Enterprise locator governance

Your form locator hierarchy is becoming:

```text
                     FORM CONTROL
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
      Label          Accessible Role     Placeholder
        │                 │                 │
        ▼                 ▼                 ▼
   Field Identity     Semantic Type     Input Hint
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                  Locator Strategy
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   getByLabel()     getByRole()      getByPlaceholder()
                          │
                          ▼
                    Action / Assert
```

You are now ready to move beyond form-specific locating and learn another fundamental locator strategy.

---

# Next Step

# Step 64 — Mastering `getByTestId()`: Test IDs, Automation Contracts, `data-testid`, Custom Test ID Attributes, Stability, Governance, Component Boundaries, Design Systems, Dynamic UIs, Micro Frontends, Enterprise Locator Strategy, and When Test IDs Are Better Than User-Facing Locators

The next lesson will answer one of the most debated questions in enterprise automation:

> **Should automation tests use `data-testid`, or should they always use user-facing locators?**

We will deeply examine:

```text
User-Facing Locator
        vs
Automation Contract
```

and learn when a test ID is:

```text
Excellent Architecture
```

versus:

```text
A Shortcut Hiding Poor Locator Design
```

We will also cover:

```text
data-testid
Custom Test ID Attributes
Test ID Governance
Component Contracts
Design Systems
Micro Frontends
Dynamic Data
Generated IDs
Locator Stability
Code Reviews
Enterprise Standards
```
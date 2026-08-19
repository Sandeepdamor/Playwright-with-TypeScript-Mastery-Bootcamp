# Playwright with TypeScript Mastery Bootcamp

## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 23 — Locator Architecture and Element Identification

# Step 50 — Mastering Advanced Locator Engineering: `getByRole`, `getByText`, `getByLabel`, `getByPlaceholder`, `getByAltText`, `getByTitle`, `getByTestId`, CSS vs XPath, Locator Composition, Filtering, `has`, `hasText`, `nth`, `first`, `last`, Collections, Strictness Errors, and Dynamic Element Strategies

---

# Objective

In this lesson, you will move from understanding locator philosophy to mastering the detailed Locator API.

You will learn:

- How Playwright's semantic locator methods work
- `getByRole`
- `getByText`
- `getByLabel`
- `getByPlaceholder`
- `getByAltText`
- `getByTitle`
- `getByTestId`
- CSS selectors
- XPath selectors
- Locator composition
- Chaining
- `filter`
- `has`
- `hasText`
- `hasNot`
- `hasNotText`
- `first`
- `last`
- `nth`
- Counting matches
- Strictness
- Collections
- Dynamic tables
- Dynamic cards
- Repeated components
- Nested elements
- Locator debugging
- Enterprise locator standards

The goal is not to memorize methods.

The goal is to understand:

```text
Which locator should I use?

↓

Why should I use it?

↓

How stable is it?

↓

How unique is it?

↓

Will it survive UI changes?
```

---

# Before We Start

Imagine a library containing 100,000 books.

You want:

```text
One specific book.
```

You could search by:

```text
Shelf number

↓

Position

↓

Row

↓

Third book
```

But if the books are rearranged, your strategy fails.

A better strategy is:

```text
Title

↓

Author

↓

ISBN
```

The best identification method depends on what makes the book uniquely identifiable.

UI elements work the same way.

---

# The Problem

Modern applications contain:

```text
Repeated Buttons

Repeated Links

Dynamic Tables

Cards

Menus

Forms

Dialogs

Virtualized Lists

Nested Components
```

A simple selector such as:

```text
button
```

may match:

```text
20 elements
```

You need to narrow the target intelligently.

---

# What is Locator Engineering?

Locator engineering is the disciplined design of element-identification strategies that are:

```text
Stable

↓

Unique

↓

Readable

↓

Semantic

↓

Maintainable
```

It is a software-engineering activity, not just a selector-writing activity.

---

# The Locator API Family

Playwright provides several semantic locator methods.

Conceptually:

```text
Page
 │
 ├── getByRole()
 ├── getByText()
 ├── getByLabel()
 ├── getByPlaceholder()
 ├── getByAltText()
 ├── getByTitle()
 ├── getByTestId()
 └── locator()
```

The first group emphasizes user-facing or explicit semantics.

`locator()` provides lower-level selector capabilities such as CSS and XPath.

---

# `getByRole`

`getByRole` identifies elements according to their accessible role.

Common roles include:

```text
button

link

textbox

checkbox

radio

heading

combobox

listbox

option

dialog

tab

tabpanel

alert

row

cell
```

The exact role depends on the element's semantics and accessibility tree.

---

# Why `getByRole` Is Usually Powerful

Consider:

```text
<button>Save</button>
```

Its important identity is:

```text
Role = button

Accessible Name = Save
```

That is much more meaningful than:

```text
div:nth-child(7)
```

The semantic locator reflects how a user understands the interface.

---

# Accessible Name

When using role-based locators, the accessible name is extremely important.

For example:

```text
button

↓

"Save Order"
```

The accessible name can come from mechanisms such as:

- Visible text
- Associated labels
- Accessible naming attributes
- Other accessibility semantics

Therefore, a role locator is not simply searching raw HTML text.

---

# Role + Name

The strongest common role pattern is:

```text
Role

+

Accessible Name
```

Conceptually:

```text
Button

↓

"Submit Order"
```

This often produces a highly readable locator.

---

# Why Role Alone Can Be Too Broad

Suppose the page contains:

```text
Save

Save

Save
```

A locator targeting:

```text
button
```

is ambiguous.

Adding the accessible name may still produce multiple matches.

In that case, you need additional context.

---

# Role + Context

Suppose there are multiple cards:

```text
Product A
    Buy

Product B
    Buy

Product C
    Buy
```

Instead of selecting:

```text
Buy
```

globally,

you can conceptually narrow:

```text
Product B Card

↓

Button "Buy"
```

This is locator composition.

---

# `getByText`

`getByText` identifies elements based on text content.

It is useful when the visible text itself is the meaningful identity.

For example:

```text
Order Confirmed
```

may be a meaningful target.

---

# When Text Is a Good Choice

Text is useful when:

```text
Text is stable

+

Text is meaningful

+

Target is unique
```

Examples:

```text
Order Confirmed

Payment Successful

Account Locked
```

These are business-facing messages.

---

# When Text Can Be Risky

Suppose a product team changes:

```text
Submit Order
```

to:

```text
Place Order
```

The application behavior is unchanged.

A text-based locator can now fail.

Therefore:

```text
Visible Text

↓

Useful

but

Potentially Changeable
```

---

# Exact vs Partial Text

Text matching can involve different matching behaviors.

Conceptually:

```text
Exact

↓

"Orders"
```

versus:

```text
Contains

↓

"Orders Management"
```

Be careful with partial matching.

A broad match may accidentally identify multiple elements.

---

# `getByLabel`

`getByLabel` is especially valuable for forms.

Imagine:

```text
Email Address
[____________]
```

The field has a semantic relationship with:

```text
Email Address
```

A label-based locator can use that relationship.

---

# Why Label-Based Locators Are Strong

Forms often have:

```text
Label

↓

Control
```

The label represents business meaning.

Therefore:

```text
Email Address

↓

Email Input
```

is generally more expressive than:

```text
input:nth-child(4)
```

---

# `getByPlaceholder`

Placeholder-based identification uses the placeholder shown inside an input.

For example:

```text
Search orders...
```

can identify a search field.

---

# When Placeholder Is Useful

Use it when:

```text
No stable label exists

+

Placeholder is meaningful

+

Placeholder is stable
```

---

# Placeholder Limitation

A placeholder is often UX copy.

It can change during:

```text
Localization

UX redesign

Copy updates
```

Therefore:

```text
Label

↓

Usually Preferable

↓

Placeholder
```

when both provide equivalent stable identification.

---

# `getByAltText`

`getByAltText` can identify elements such as images through their alternative text.

Example concept:

```text
Image

↓

Alt Text = Company Logo
```

This is useful when the alternative text is meaningful and stable.

---

# Accessibility Importance

Alt text is not only an automation technique.

It also supports:

```text
Screen Readers

↓

Accessibility

↓

User Understanding
```

Therefore, meaningful alt text provides value beyond testing.

---

# `getByTitle`

`getByTitle` identifies elements through their title attribute.

Example:

```text
title="Settings"
```

This can be useful for tooltips or controls where title is intentionally part of the UI.

---

# Caution With Title

Do not assume every title attribute is a permanent automation contract.

If title values are generated or frequently changed:

```text
Title Locator

↓

Potentially Fragile
```

Use it when the title is stable and meaningful.

---

# `getByTestId`

Test IDs provide explicit automation identifiers.

Conceptually:

```text
data-testid="checkout-submit"
```

The automation contract becomes:

```text
Checkout Submit

↓

checkout-submit
```

---

# Why Test IDs Are Powerful

Test IDs can survive changes to:

```text
DOM Structure

CSS Classes

Visual Layout

Component Wrappers
```

as long as the test ID remains stable.

---

# Test ID Governance

An enterprise organization should define conventions.

For example:

```text
domain-component-action
```

Conceptually:

```text
checkout-submit
orders-create
customer-search
```

The exact naming convention should be standardized by the team.

---

# Test ID Anti-Pattern

Avoid meaningless IDs:

```text
test1

test2

abc123

element7
```

These do not communicate intent.

Prefer:

```text
order-create-button
```

or another meaningful organization-approved identifier.

---

# `locator()`

The generic Locator API allows more direct selector strategies.

Conceptually:

```text
page.locator(...)
```

can work with selectors such as:

```text
CSS

XPath
```

It provides flexibility when semantic locators are insufficient.

---

# CSS Selectors

CSS selectors can identify:

```text
Tag

Class

ID

Attribute

Relationship
```

Conceptually:

```text
button.primary
```

or:

```text
input[name="email"]
```

The exact selector should depend on the application's stable attributes.

---

# Good CSS

A stable CSS selector might rely on:

```text
Explicit Stable Attribute
```

For example:

```text
data-state="active"
```

if that attribute is intentionally stable and appropriate for automation.

---

# Bad CSS

Avoid selectors based on generated styling classes such as:

```text
css-8fj32k

sc-aBc123

Button_root__x7a8
```

These often represent implementation details rather than stable identity.

---

# XPath

XPath can express complex relationships.

Conceptually:

```text
Find Row

↓

Find Cell

↓

Find Button
```

XPath can be useful when no suitable semantic or stable CSS strategy exists.

---

# Absolute XPath

Avoid patterns conceptually like:

```text
/html/body/div[2]/div[1]/main/div[3]/table/tbody/tr[4]/td[6]/button
```

This is heavily dependent on DOM structure.

A small layout change can break it.

---

# Relative XPath

A meaningful relative relationship may be more maintainable.

For example:

```text
Find element associated with specific text

↓

Find related action
```

Even then,

prefer semantic Playwright locators when they provide a clearer solution.

---

# Locator Composition

Locator composition means building a specific target from smaller locator concepts.

For example:

```text
Page

↓

Order Card

↓

Customer Name

↓

Action Button
```

This is often much stronger than a single huge selector.

---

# Why Composition Matters

Suppose a page has:

```text
100 Edit buttons
```

You need:

```text
Edit button for Customer A
```

Composition lets you express:

```text
Customer A Row

↓

Edit
```

The locator communicates the business relationship.

---

# `filter`

Filtering narrows a collection of matching elements.

Conceptually:

```text
All Orders

↓

Filter

↓

Orders belonging to Customer A
```

This is extremely useful for repeated UI structures.

---

# `hasText`

`hasText` can narrow a locator based on text contained within an element.

Imagine:

```text
Order Card
    Order #1001
    Customer: Sandeep
    Status: Paid
    Edit
```

You can conceptually locate:

```text
Card

↓

hasText("Sandeep")
```

Then:

```text
Card

↓

Edit Button
```

This is far more stable than:

```text
Third Card
```

---

# `has`

`has` allows one locator to be narrowed based on the presence of another locator.

Conceptually:

```text
Cards

↓

Cards containing a specific child element

↓

Target Card
```

This becomes powerful when text alone is not sufficient.

---

# `hasNot`

`hasNot` helps exclude elements containing a particular matching locator.

Conceptually:

```text
All Cards

↓

Exclude Archived Cards

↓

Active Cards
```

This is useful when the business requirement is defined by absence.

---

# `hasNotText`

`hasNotText` can exclude elements based on text.

Conceptually:

```text
All Orders

↓

Exclude "Cancelled"

↓

Remaining Orders
```

This can simplify dynamic collection handling.

---

# Chaining

Chaining means narrowing from a broader context to a more specific target.

Conceptually:

```text
Page

↓

Table

↓

Row

↓

Button
```

Each stage adds context.

---

# Why Chaining Is Better Than Giant Selectors

Compare:

```text
One Huge Selector
```

with:

```text
Table

↓

Matching Row

↓

Edit Button
```

The second communicates intent.

It is easier to:

- Read
- Debug
- Maintain
- Review

---

# `first`

`first` selects the first matching element.

Conceptually:

```text
Matching Elements

↓

[1] [2] [3] [4]

↓

first

↓

[1]
```

---

# Is `first` Bad?

Not necessarily.

The problem is using it without understanding why the first element is correct.

Bad reasoning:

```text
There are multiple matches.

↓

Use first.
```

Better reasoning:

```text
Business requirement says:

↓

Use the first available item.
```

Then `first` may be legitimate.

---

# `last`

`last` selects the final matching element.

Again:

```text
Many Matches

↓

last

↓

Final Match
```

Use it when the business meaning actually requires the final element.

---

# `nth`

`nth` selects an element at a specific zero-based index.

Conceptually:

```text
Items

0 → A
1 → B
2 → C
3 → D
```

Therefore:

```text
nth(2)

↓

C
```

---

# Why `nth` Can Be Dangerous

Suppose:

```text
nth(2)

↓

Product C
```

Tomorrow the application adds:

```text
Product X
```

before Product C.

Now:

```text
nth(2)

↓

Product B
```

The test still runs.

But it interacts with the wrong element.

This is more dangerous than an obvious failure.

---

# The Principle of Safe Positional Selection

Use:

```text
first

last

nth
```

when position is part of the intended behavior.

Do not use them merely to silence strictness errors.

---

# Strictness Errors

Suppose an action targets:

```text
5 matching buttons
```

Playwright can report a strictness violation because the action does not identify one unique target.

This is a valuable failure.

It tells you:

```text
Your locator is ambiguous.
```

---

# Wrong Way to Fix Strictness

Do not immediately do:

```text
first()
```

just to make the test pass.

Instead ask:

```text
Why are there five matches?

↓

Which one is intended?

↓

What makes it unique?
```

Then design a better locator.

---

# Locator Cardinality

Every locator can be thought of as a collection.

Conceptually:

```text
Locator

↓

0 elements

1 element

Many elements
```

The framework can inspect how many elements match.

---

# Count

Counting is useful when the business requirement concerns a collection.

For example:

```text
Number of Search Results

↓

Expected = 10
```

This is different from:

```text
Click Search Result
```

where a specific target may be required.

---

# Collection vs Unique Element

A Locator can represent:

```text
Collection
```

while an action may require:

```text
Unique Element
```

Understanding this distinction prevents many locator errors.

---

# Dynamic Tables

Tables are one of the best places to apply advanced locator composition.

Imagine:

```text
Customer       Status       Action

Alice          Active       Edit
Bob            Pending      Edit
Charlie        Active       Edit
```

The requirement:

```text
Edit Bob
```

should not become:

```text
Second Row

↓

Third Button
```

Instead:

```text
Find Row Containing Bob

↓

Find Edit Within Row
```

---

# Why This Is Enterprise-Grade

The test describes:

```text
Edit Bob
```

rather than:

```text
Click button #7
```

The first expresses business intent.

The second expresses implementation position.

---

# Dynamic Cards

Suppose a dashboard displays:

```text
Order 1001

Order 1002

Order 1003
```

Each card contains:

```text
View

Edit

Cancel
```

To cancel Order 1002:

```text
Find Order 1002 Card

↓

Find Cancel

↓

Cancel
```

This remains stable if card ordering changes.

---

# Nested Locator Architecture

A powerful pattern is:

```text
Page

↓

Region

↓

Component

↓

Target
```

For example:

```text
Page

↓

Orders Table

↓

Order Row

↓

Customer Name

↓

Action Button
```

This is effectively hierarchical locator design.

---

# Dialog Example

Suppose the page contains:

```text
Delete buttons
```

and then opens:

```text
Delete Confirmation Dialog
```

The dialog may contain:

```text
Cancel

Confirm Delete
```

Instead of searching globally:

```text
Confirm Delete
```

you can conceptually scope the search to:

```text
Dialog

↓

Confirm Delete
```

This reduces ambiguity.

---

# Navigation Example

Suppose there are multiple:

```text
Orders
```

elements.

A useful strategy may be:

```text
Navigation Region

↓

Orders Link
```

rather than:

```text
Text "Orders"
```

across the entire page.

---

# Locator Scope

Scope answers:

```text
Where should Playwright search?
```

Examples:

```text
Entire Page
```

or:

```text
Specific Dialog
```

or:

```text
Specific Table Row
```

or:

```text
Specific Component
```

Good scoping reduces ambiguity.

---

# Locator Reusability

A Page Object can define meaningful locator concepts:

```text
ordersTable

customerSearch

createOrderButton
```

Then methods can operate on those targets.

This separates:

```text
What the page contains

↓

How tests use it
```

---

# Locator Abstraction

A good abstraction hides implementation details.

Test:

```text
Create Order
```

Page Object:

```text
Click Create Order
```

Locator:

```text
Stable UI Target
```

The test does not need to know whether the button uses:

```text
Role

↓

Test ID

↓

CSS
```

---

# Why This Is Important

If the application changes from:

```text
CSS class
```

to:

```text
Test ID
```

the Page Object can change.

The test scenario remains:

```text
Create Order
```

This reduces maintenance cost.

---

# Dynamic Element Strategy

For dynamic applications, think in terms of:

```text
Stable Parent

↓

Meaningful Child

↓

Action
```

rather than:

```text
Wait

↓

Position

↓

Click
```

---

# Locator + Synchronization

A strong locator should work with Playwright's automatic waiting.

Conceptually:

```text
Locate Target

↓

Target Appears

↓

Actionability

↓

Action
```

Avoid replacing locator design with arbitrary delays.

---

# Locator Debugging

When a locator fails, investigate:

```text
1. Does the element exist?

2. Does the locator match anything?

3. Does it match too many elements?

4. Is the accessible name correct?

5. Is the element inside an iframe?

6. Is it inside a shadow DOM?

7. Is it dynamically rendered?

8. Is the target hidden?

9. Is there another matching element?
```

This diagnostic sequence is much more effective than blindly changing selectors.

---

# Using the Playwright Inspector

The Inspector can help explore:

```text
DOM

↓

Locators

↓

Element Properties

↓

Actions
```

It can help engineers understand how Playwright sees the page.

But generated locators should be reviewed rather than blindly copied.

---

# Codegen and Locator Discovery

Playwright's code-generation tooling can suggest locator strategies.

This is useful for:

```text
Learning

↓

Exploration

↓

Initial Test Creation
```

However:

```text
Generated Locator

≠

Automatically Best Locator
```

Human review remains important.

---

# Locator Review Checklist

Before approving a locator, ask:

```text
Is it unique?

↓

Is it stable?

↓

Is it semantic?

↓

Does it represent business intent?

↓

Could localization break it?

↓

Could CSS changes break it?

↓

Could DOM restructuring break it?

↓

Is there a better locator?
```

---

# Enterprise Locator Standards

A mature organization can establish:

```text
Preferred

↓

Role / Label / Semantic

↓

Test ID

↓

Stable Attribute

↓

CSS

↓

XPath
```

And rules such as:

```text
No absolute XPath

No generated CSS classes

No unexplained nth()

No unexplained first()/last()

No arbitrary positional selectors
```

---

# Locator Naming Standards

Use names that describe the business target.

Good:

```text
createOrderButton

customerSearchInput

pendingOrderRow
```

Poor:

```text
btn

input1

row2
```

Good names improve code review and debugging.

---

# Locator Stability Score

A useful mental model is:

```text
Semantic Meaning
       +
Uniqueness
       +
Stability
       +
Readability
       =
Strong Locator
```

If one factor is weak,

the locator deserves review.

---

# Enterprise Locator Governance Workflow

```text
Developer Builds UI

↓

Semantic HTML / Accessible Name

↓

Stable Automation Attribute When Needed

↓

Automation Engineer Creates Locator

↓

Code Review

↓

Locator Stability Review

↓

Approved
```

This makes locator quality a team responsibility.

---

# Architecture

```text
                         Page
                          │
                          ▼
                    Locator Strategy
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
      Semantic        Explicit ID       Structural
      Locators         Locators          Locators
          │               │                │
          │               │          ┌─────┴─────┐
          │               │          ▼           ▼
          │               │         CSS        XPath
          │               │
          └───────┬───────┘
                  ▼
            Locator Composition
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      filter     has      hasText
        │         │         │
        └─────────┼─────────┘
                  ▼
              Unique Target
                  │
                  ▼
              Action / Assertion
```

---

# Workflow

```text
Business Requirement

↓

Identify Business Target

↓

Determine Element Semantics

↓

Check Accessibility

↓

Choose Primary Locator

↓

Check Uniqueness

↓

Add Context If Needed

↓

Filter / Compose

↓

Validate Stability

↓

Use In Test / Page Object

↓

Review
```

---

# Enterprise Perspective

At enterprise scale, locator quality directly affects:

```text
Test Stability

↓

Maintenance Cost

↓

Pipeline Reliability

↓

Release Confidence
```

Suppose an organization has:

```text
20,000 Tests
```

and poor locator design causes:

```text
5% unnecessary failures
```

That creates:

```text
1,000 potentially avoidable failures
```

Every release.

Even small locator-quality improvements can therefore produce substantial organizational benefits.

---

# Best Practices

Use the following strategy:

```text
1. Prefer semantic locators.

2. Prefer accessible roles for interactive elements.

3. Use labels for form controls.

4. Use stable text when text is business-significant.

5. Use test IDs when an explicit automation contract is appropriate.

6. Use stable attributes when meaningful.

7. Use CSS when it provides a stable selector.

8. Use XPath only when it provides a clear advantage.

9. Scope locators to meaningful regions.

10. Use `filter`, `has`, and `hasText` for repeated structures.

11. Avoid unexplained positional selection.

12. Treat strictness failures as useful design feedback.

13. Keep locator names meaningful.

14. Review selectors during code review.

15. Collaborate with developers on locator stability.
```

---

# Common Beginner Mistakes

### Mistake 1 — XPath for Everything

```text
Every element

↓

XPath
```

This ignores Playwright's semantic locator capabilities.

---

### Mistake 2 — Using `first()` to Fix Strictness

```text
Multiple Matches

↓

first()

↓

Problem Hidden
```

The correct question is:

```text
Why are there multiple matches?
```

---

### Mistake 3 — Using `nth()` Everywhere

Position-based selection creates hidden dependencies on UI ordering.

---

### Mistake 4 — Generated CSS Classes

Generated classes are often implementation details.

---

### Mistake 5 — Ignoring Accessibility

If the application lacks meaningful semantics,

automation often becomes harder.

---

### Mistake 6 — Giant Selectors

Extremely long selectors are difficult to:

```text
Understand

↓

Debug

↓

Maintain
```

---

# Professional Tips

A senior SDET follows this rule:

> **Use the smallest amount of locator information necessary to uniquely identify the intended element.**

For example:

```text
Role

+

Name
```

may be enough.

If not:

```text
Region

↓

Role

+

Name
```

If still not:

```text
Stable Test ID
```

The goal is not the longest selector.

The goal is the most meaningful selector.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What is `getByRole`?

**Answer:**

`getByRole` identifies elements using their accessible role, optionally combined with an accessible name. It is useful because it targets semantic UI behavior rather than implementation-specific DOM details.

---

### Mid-Level Question

**Q:** When would you use `getByLabel`?

**Answer:**

I would use it primarily for form controls that have meaningful associated labels, such as username, email, password, or address fields.

---

### Senior-Level Question

**Q:** What is the purpose of `filter({ hasText })`?

**Answer:**

It narrows a collection of matching elements to those containing specified text. It is particularly useful for repeated components such as table rows, cards, or list items where the target must be identified by its content.

---

### Senior-Level Question

**Q:** Why shouldn't you use `nth()` just to resolve a strictness error?

**Answer:**

Because it hides locator ambiguity instead of solving it. The UI may change order later, causing the test to interact with the wrong element. The locator should instead be made unique using meaningful context.

---

### Lead-Level Question

**Q:** How would you locate an Edit button belonging to a specific customer in a dynamic table?

**Answer:**

I would first identify the table row using a stable customer-specific locator, such as the customer's name or identifier, then scope the locator to that row and locate the Edit action within it. This avoids dependence on row position.

---

### Architect-Level Question

**Q:** How would you establish locator governance across multiple automation teams?

**Answer:**

I would define a locator hierarchy favoring semantic roles and labels, establish conventions for test IDs, prohibit brittle selectors such as absolute XPath and generated classes where possible, require locator uniqueness and stability review, and include locator standards in framework documentation and code-review guidelines.

---

# Knowledge Check

Answer these before proceeding:

1. What is locator engineering?
2. What does `getByRole` identify?
3. What is an accessible name?
4. Why is role plus accessible name powerful?
5. When should `getByText` be used?
6. What is the advantage of `getByLabel`?
7. What is a placeholder?
8. Why can placeholders be less stable than labels?
9. What is `getByAltText` useful for?
10. What does `getByTitle` use?
11. What is a Test ID?
12. Why are Test IDs considered an explicit automation contract?
13. When should `locator()` be used?
14. What are CSS selectors?
15. What is XPath?
16. Why is absolute XPath fragile?
17. What is locator composition?
18. What does `filter` accomplish?
19. What are `has` and `hasText` useful for?
20. What is the purpose of `first()`?
21. What is the purpose of `last()`?
22. What is the purpose of `nth()`?
23. Why can positional locators be dangerous?
24. What is a strictness violation?
25. Why should strictness errors not simply be solved with `first()`?
26. How should dynamic table rows be located?
27. How should repeated cards be located?
28. Why is locator scope important?
29. How does accessibility influence locator strategy?
30. What makes an enterprise-grade locator?

---

# Step Summary

This lesson transformed locator knowledge from:

```text
"How do I find an element?"
```

into:

```text
"How do I reliably identify the correct element?"
```

You learned the major Playwright locator families:

```text
getByRole

getByText

getByLabel

getByPlaceholder

getByAltText

getByTitle

getByTestId

locator()
```

You also learned advanced composition concepts:

```text
filter

has

hasText

hasNot

hasNotText

first

last

nth

count
```

The central engineering model is:

```text
Business Intent

↓

Semantic Target

↓

Stable Locator

↓

Context / Filtering

↓

Unique Element

↓

Action
```

The most important lesson is:

> **Never use a positional or structural selector merely because it works today. First determine what makes the element logically unique.**

For enterprise automation:

```text
Stable Locator

↓

Stable Test

↓

Lower Maintenance

↓

Higher Pipeline Reliability

↓

Higher Release Confidence
```

---

# Progress Milestone

✅ You have completed **Step 50** of approximately **230** planned learning steps.

**What you've mastered:**

- `getByRole`
- Accessible Names
- `getByText`
- `getByLabel`
- `getByPlaceholder`
- `getByAltText`
- `getByTitle`
- `getByTestId`
- `locator()`
- CSS
- XPath
- Locator Composition
- Chaining
- `filter`
- `has`
- `hasText`
- `hasNot`
- `hasNotText`
- `first`
- `last`
- `nth`
- Locator Cardinality
- Collections
- Strictness
- Dynamic Tables
- Dynamic Cards
- Nested Components
- Locator Scope
- Locator Governance
- Enterprise Locator Standards

**Coming next — Step 51:**

**Mastering Playwright Actions: Click, Fill, Press, Check, Uncheck, Select, Hover, Focus, Drag and Drop, Keyboard Actions, Mouse Actions, Scroll, Double Click, Right Click, Force Actions, Actionability Checks, Navigation, and Reliable User Interaction Design.**

The next lesson will explain not just how Playwright performs actions, but why Playwright performs **actionability checks before interacting with elements**, and how that architecture eliminates many synchronization problems found in traditional UI automation.
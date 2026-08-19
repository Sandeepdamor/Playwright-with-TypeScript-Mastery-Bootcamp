# Playwright with TypeScript Mastery Bootcamp

## From Absolute Beginner to Enterprise Automation Architect

# Stage 29 — Locator Engineering

# Step 60 — Mastering `getByRole()`: Accessible Roles, Accessible Names, Implicit Roles, Explicit Roles, Role Discovery, Role Filtering, Role-Based Assertions, and Enterprise Locator Design

---

# Objective

In the previous step, you learned the philosophy behind locator engineering.

You learned that a locator should preferably represent:

```text
Business Meaning
      ↓
User Meaning
      ↓
Accessibility Meaning
      ↓
Stable Test Contract
```

Now we focus deeply on:

```text
getByRole()
```

`getByRole()` is one of the most important locator strategies in Playwright.

You will learn:

- What a role is
- What an accessible role means
- What an accessible name means
- Implicit roles
- Explicit roles
- Native HTML semantics
- ARIA roles
- Role discovery
- Role + name
- Role + exact matching
- Role filtering
- Role + state
- Buttons
- Links
- Headings
- Textboxes
- Checkboxes
- Radio buttons
- Comboboxes
- Listboxes
- Dialogs
- Navigation
- Menus
- Tables
- Rows
- Cells
- Accessibility tree concepts
- Why role locators are powerful
- When role locators should not be forced
- Debugging role locators
- Enterprise role-based locator standards

---

# Before We Start

Imagine walking into a hospital.

You see:

```text
Reception Desk
Emergency Department
Pharmacy
Laboratory
Radiology
```

You don't identify the emergency department by saying:

```text
"The fourth room after the blue wall."
```

You identify it by its meaning:

```text
"Emergency Department."
```

Now imagine asking a staff member:

```text
"Where is the pharmacy?"
```

You are identifying something by:

```text
Purpose
+
Meaning
```

Web accessibility works similarly.

Instead of thinking:

```text
div
button-class
nth-child(3)
```

we can think:

```text
Button
Link
Heading
Textbox
Checkbox
Dialog
```

The role describes **what the element is** from the user's perspective.

---

# The Problem

Suppose an application contains:

```text
Login

Username
Password

Remember Me

Login
```

A traditional selector approach might inspect:

```text
class
id
CSS structure
XPath
```

But the user doesn't care about those implementation details.

The user sees:

```text
Textbox
Textbox
Checkbox
Button
```

Therefore a better automation question is:

> What does this element represent to the user?

This is the foundation of role-based locating.

---

# What is a Role?

A role describes the semantic purpose of a UI element.

Examples include:

```text
button
link
heading
textbox
checkbox
radio
combobox
listbox
dialog
navigation
menu
table
row
cell
```

A role is not simply a CSS class.

It represents semantic meaning.

---

# What is `getByRole()`?

`getByRole()` is a Playwright locator API used to locate elements according to their accessible role and, optionally, other accessible properties such as their accessible name.

Conceptually:

```text
Page
 ↓
Accessible Semantics
 ↓
Role
 ↓
Optional Name / State
 ↓
Matching Element
```

---

# Basic Mental Model

When you see:

```text
Login
```

and determine that it is a button, your thinking should become:

```text
Role = button
Name = Login
```

The locator conceptually becomes:

```text
Button
+
Accessible Name "Login"
```

This is much more meaningful than:

```text
button:nth-child(4)
```

---

# Role and Accessible Name

A role alone may not uniquely identify an element.

Consider:

```text
Save
Cancel
Delete
```

All three may have:

```text
role = button
```

Therefore:

```text
button
```

is not enough.

You need:

```text
role + accessible name
```

Conceptually:

```text
button
+
"Delete"
```

---

# What is an Accessible Name?

The accessible name is the meaningful name exposed to accessibility technologies and used to identify an interactive or semantic element.

For example:

```text
<button>
    Save Order
</button>
```

may have:

```text
Role:
button

Accessible Name:
Save Order
```

---

# Why Accessible Name Matters

Consider:

```text
Button
Button
Button
```

Role alone is ambiguous.

But:

```text
Button "Save"
Button "Cancel"
Button "Delete"
```

is precise.

Therefore:

```text
Role
+
Accessible Name
```

is one of the strongest locator concepts.

---

# Role Locator Architecture

```text
                  Web Application
                         │
                         ▼
                        DOM
                         │
                         ▼
                Semantic Information
                         │
                         ▼
                Accessibility Model
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
            Role                Accessible Name
             │                       │
             └───────────┬───────────┘
                         ▼
                    getByRole()
                         │
                         ▼
                  Matching Element
                         │
                         ▼
                  Action / Assertion
```

---

# Native HTML and Implicit Roles

Many native HTML elements have semantic meaning automatically.

For example:

```text
button
```

naturally represents:

```text
button
```

A link element naturally represents:

```text
link
```

A heading element naturally represents:

```text
heading
```

This is called an implicit semantic role.

---

# Why Native HTML Matters

Good HTML gives automation a semantic foundation.

For example:

```text
Native Button
    ↓
Button Role
    ↓
Accessible Name
    ↓
Reliable Locator
```

This is one reason semantic HTML is valuable.

---

# Explicit ARIA Roles

Applications can also use ARIA attributes to communicate semantics.

Conceptually:

```text
role="button"
```

can tell accessibility technology that an element should be treated as a button.

However:

> Adding a role does not automatically make an element behave exactly like a native element.

---

# Native HTML vs ARIA

Prefer:

```text
Native Semantic HTML
```

when appropriate.

For example, a real button is generally preferable to making a generic container behave like a button through ARIA alone.

Why?

Because native controls come with expected browser behavior and accessibility semantics.

---

# Important Principle

```text
Native Semantics
      >
Artificial Semantics
```

when native semantics can correctly represent the UI.

---

# What is ARIA?

ARIA stands for:

```text
Accessible Rich Internet Applications
```

ARIA provides semantic information for accessibility.

It is particularly useful for complex widgets and application interfaces.

---

# ARIA and Playwright

Playwright role locators can use accessibility semantics exposed by the page.

Therefore:

```text
Accessible UI
      ↓
Meaningful Roles
      ↓
Better Role Locators
```

This creates a strong connection between:

```text
Accessibility
+
Automation
```

---

# Common Roles

You should become comfortable recognizing roles such as:

```text
button
link
heading
textbox
checkbox
radio
combobox
listbox
option
dialog
alert
alertdialog
navigation
main
banner
contentinfo
menu
menuitem
tab
tabpanel
table
row
cell
columnheader
rowheader
```

Not every HTML element maps to every role directly.

The important principle is understanding semantic meaning.

---

# Button Role

A button represents an action.

Examples:

```text
Login
Save
Submit
Delete
Approve
Cancel
```

The conceptual target is:

```text
Role = button
Name = action name
```

---

# Why Button Role Is Strong

Suppose a developer changes:

```text
class="primary-button"
```

to:

```text
class="success-action"
```

A semantic role-based locator can continue to work if the actual semantic button and accessible name remain unchanged.

This is resilience against irrelevant implementation changes.

---

# Button Example

Imagine:

```text
[ Save Order ]
```

The user's understanding is:

```text
Button
+
Save Order
```

not:

```text
div.container > span:nth-child(2)
```

---

# Link Role

A link represents navigation to another location or resource.

Examples:

```text
Home
Orders
Products
Reports
Logout
```

when they are implemented as links.

The semantic concept is:

```text
Role = link
Name = destination/action label
```

---

# Link vs Button

This distinction matters.

### Link

Usually means:

```text
Navigate Somewhere
```

### Button

Usually means:

```text
Perform an Action
```

For example:

```text
Orders
```

may be a link if it navigates to an orders page.

While:

```text
Delete Order
```

is generally an action and therefore a button.

---

# Why This Matters in Automation

Your locator should reflect the application's semantic contract.

Do not simply think:

```text
"I need something clickable."
```

Think:

```text
"Is this navigation or an action?"
```

---

# Heading Role

Headings communicate page structure.

Examples:

```text
Dashboard
Orders
Order Details
Customer Information
```

Headings can be organized into levels.

Conceptually:

```text
Heading Level 1
    ↓
Page Title

Heading Level 2
    ↓
Major Section

Heading Level 3
    ↓
Subsection
```

---

# Why Heading Locators Are Useful

They can help identify:

```text
Page Sections
Dialogs
Major Components
Expected Page Identity
```

For example:

```text
Expected heading:
Order Management
```

can be a strong validation of page context.

---

# Textbox Role

A textbox represents an input area intended for text entry.

Examples:

```text
Username
Email
Password
Search
Address
```

Depending on the HTML implementation, the accessibility role may vary for specialized controls.

---

# Password Inputs

A password field is still an input control, but its accessible semantics and implementation details should be considered when selecting a locator.

The most important principle is:

```text
Use the meaningful form label
```

rather than relying only on:

```text
CSS class
```

---

# Checkbox Role

A checkbox represents an independently selectable boolean option.

Example:

```text
☐ Remember Me
```

Conceptually:

```text
Role = checkbox
Name = Remember Me
```

---

# Checkbox States

A checkbox can have states such as:

```text
Checked
Unchecked
```

Some advanced widgets may also have:

```text
Indeterminate
```

State-aware role locating and assertions become especially useful here.

---

# Radio Role

Radio buttons represent mutually exclusive choices within a group.

Example:

```text
( ) Male
( ) Female
( ) Other
```

The role is:

```text
radio
```

with an accessible name representing the option.

---

# Radio Group Concept

The important difference is:

```text
Checkbox
    ↓
Multiple options may be selected
```

while:

```text
Radio Group
    ↓
Usually one option selected
```

This semantic difference matters when designing tests.

---

# Combobox Role

A combobox represents an input/control used to select or search among options.

Common UI examples include:

```text
Country
City
Status
Category
```

Modern applications may implement sophisticated custom combobox widgets.

---

# Why Custom Comboboxes Are Difficult

A custom dropdown may visually look like:

```text
Select Status
```

but internally consist of:

```text
div
button
input
listbox
option-like elements
```

The correct automation strategy should be based on the accessible behavior rather than blindly selecting DOM tags.

---

# Listbox Role

A listbox represents a collection of selectable options.

Conceptually:

```text
Listbox
   │
   ├── Option A
   ├── Option B
   └── Option C
```

This semantic model is useful when testing custom selection widgets.

---

# Option Role

An option represents a selectable item inside a selection widget such as a listbox or related control.

Conceptually:

```text
Listbox
 ↓
Option
 ↓
"Pending"
```

---

# Dialog Role

A dialog is a distinct interactive region that appears above or alongside the main page content.

Examples:

```text
Confirm Delete
Edit User
Create Order
Payment Details
```

Conceptually:

```text
Page
 │
 └── Dialog
      │
      ├── Heading
      ├── Form
      └── Buttons
```

---

# Why Dialog Scoping Is Powerful

Suppose the page contains:

```text
Delete
```

and the dialog also contains:

```text
Delete
```

A global button locator may become ambiguous.

Instead:

```text
Dialog
 ↓
Delete Button
```

provides a clear scope.

---

# Navigation Role

Navigation represents a section of the page containing navigation links.

A common enterprise application might have:

```text
Navigation
 ├── Dashboard
 ├── Orders
 ├── Products
 └── Reports
```

This is useful for component-level scoping.

---

# Main Role

The main region represents the primary content of the document/application.

Conceptually:

```text
Page
 ├── Banner
 ├── Navigation
 └── Main
      ├── Heading
      ├── Table
      └── Forms
```

---

# Banner Role

The banner region commonly represents site-level header content.

For example:

```text
Logo
User Menu
Notifications
```

---

# Contentinfo Role

This generally represents footer information.

Conceptually:

```text
Page
 └── Contentinfo
      ├── Copyright
      ├── Privacy
      └── Terms
```

---

# Table Role

Tables represent structured two-dimensional data.

For example:

```text
Order ID | Customer | Status
---------|----------|---------
1001     | John     | Pending
1002     | Mary     | Approved
```

The semantic model can include:

```text
table
 ↓
row
 ↓
cell
```

and headers such as:

```text
columnheader
```

---

# Row Role

A row represents one logical record in a table or grid.

For example:

```text
Order #1001
```

can identify:

```text
One Row
```

---

# Cell Role

A cell represents a data value within a row.

Conceptually:

```text
Row
 ├── Order ID Cell
 ├── Customer Cell
 ├── Status Cell
 └── Action Cell
```

---

# Enterprise Table Example

Suppose an order-management page contains:

```text
Order #1001 | John | Pending | Approve
Order #1002 | Mary | Completed | View
```

The business requirement is:

```text
Approve Order #1001
```

A semantic strategy is:

```text
Table
 ↓
Row containing "Order #1001"
 ↓
Approve Button
```

This is significantly stronger than:

```text
Second button on page
```

---

# Role Filtering

A role locator can be refined using additional information.

Conceptually:

```text
All Buttons

↓

Name = "Approve"

↓

Target Button
```

Or:

```text
All Rows

↓

Row contains "Order #1001"

↓

Target Row
```

---

# Role + Name

This is one of the most common patterns.

Conceptually:

```text
Role
+
Accessible Name
```

Examples:

```text
button + "Login"

link + "Orders"

heading + "Dashboard"

checkbox + "Remember Me"
```

---

# Exact Matching

Sometimes multiple names may partially match.

For example:

```text
Save
Save Draft
Save Order
```

A broader text match may identify multiple elements.

Exact matching can be useful when the requirement is:

```text
Exactly "Save"
```

---

# Why Exactness Matters

Suppose:

```text
Delete
Delete User
Delete All
```

A loose matching strategy can produce ambiguity.

You need to distinguish:

```text
Delete
```

from:

```text
Delete User
```

and:

```text
Delete All
```

---

# Role + State

Some UI controls have state.

For example:

```text
Checkbox
    ↓
Checked
```

or:

```text
Tab
    ↓
Selected
```

or:

```text
Button
    ↓
Disabled
```

State can be used to make locators and assertions more meaningful.

---

# State-Aware Thinking

Instead of:

```text
Find Tab
```

you may need:

```text
Find selected Orders tab
```

This represents:

```text
Role
+
Name
+
State
```

---

# Tabs

Tabs represent different views within a page.

Example:

```text
[Overview] [Orders] [Payments]
```

Conceptually:

```text
tab
+
accessible name
+
selected state
```

The associated content may be represented by:

```text
tabpanel
```

---

# Menus

Menus contain actions or navigation choices.

Conceptually:

```text
Menu
 ├── Profile
 ├── Settings
 └── Logout
```

Menu systems can be complex because they may include:

```text
menu
menuitem
submenu
menuitemcheckbox
menuitemradio
```

The important principle remains:

```text
Model the user's interaction semantics.
```

---

# Accessibility Tree

To understand role-based locators deeply, you need to understand the accessibility tree conceptually.

The browser maintains accessibility information derived from:

```text
DOM
+
Semantics
+
ARIA
+
Names
+
States
```

Conceptually:

```text
DOM
 ↓
Accessibility Semantics
 ↓
Accessibility Tree
 ↓
Assistive Technologies
```

---

# DOM vs Accessibility Tree

The DOM answers:

```text
"What elements exist structurally?"
```

The accessibility tree answers more like:

```text
"What meaningful controls and information are exposed to users of assistive technology?"
```

These models are related but not identical.

---

# Why This Matters to `getByRole`

Role-based locating is aligned with accessible semantics.

Therefore:

```text
DOM Structure
     ↓
Semantic Interpretation
     ↓
Accessible Role
     ↓
getByRole()
```

This is why role-based locators can remain stable even when internal DOM structure changes.

---

# Accessibility Tree Example

Conceptually, a DOM might contain:

```text
div
 ├── span
 │    └── "Save"
```

while the accessibility interpretation might be:

```text
Button
 └── Name: "Save"
```

The automation engineer cares about the semantic target:

```text
Button "Save"
```

---

# Role Does Not Mean HTML Tag

This is a critical concept.

A role is semantic.

An HTML tag is structural.

For example:

```text
HTML tag
    ↓
button
```

may naturally map to:

```text
Role
    ↓
button
```

But other implementations can expose semantics through ARIA.

Therefore:

```text
Tag
≠
Role
```

although native HTML often establishes a role automatically.

---

# Implicit Role

A native element can provide semantics automatically.

Conceptually:

```text
Native Button
 ↓
Implicit Button Role
```

---

# Explicit Role

An element may explicitly declare an ARIA role.

Conceptually:

```text
Generic Element
+
ARIA Role
 ↓
Accessible Semantic
```

But explicit ARIA must be used correctly.

---

# ARIA Misuse

Consider a generic element made to look like a button.

Visual appearance:

```text
Looks Like Button
```

But actual behavior may not provide:

```text
Keyboard Interaction
Focus Behavior
Native Semantics
```

Therefore:

```text
ARIA Role
```

does not magically fix poor implementation.

---

# Automation Lesson

If role-based automation fails unexpectedly, do not immediately assume:

```text
Playwright Problem
```

Investigate:

```text
DOM
+
Accessibility Semantics
+
Accessible Name
+
Actual Widget Behavior
```

The issue may be in the application's implementation.

---

# Role Discovery

When debugging an unfamiliar page, ask:

```text
What is this element?

↓

What role does it expose?

↓

What is its accessible name?

↓

Is the role unique?

↓

What state does it have?
```

This is much more powerful than immediately opening DevTools and copying a CSS selector.

---

# Inspector and Role Discovery

Playwright's tooling can help inspect elements and understand their locator representation.

You can use the Inspector to investigate:

```text
Element
Role
Locator Candidate
DOM Structure
```

The goal is not merely to copy generated selectors.

The goal is to understand the semantic contract.

---

# Codegen and Role Locators

Playwright's Codegen can suggest locators based on the page.

However:

> **Generated locators should be reviewed by a human.**

Codegen optimizes for producing a usable locator.

An enterprise engineer optimizes for:

```text
Stability
Readability
Architecture
```

---

# Codegen Mental Model

```text
Application
 ↓
Codegen
 ↓
Suggested Locator
 ↓
Human Review
 ↓
Enterprise Locator
```

Do not blindly accept every generated selector.

---

# Role Locator and Dynamic Text

Suppose a button says:

```text
Approve Order #1001
```

but the order number changes dynamically.

You must decide what the test requirement actually is.

Possible intent:

```text
Find Approve Action
```

or:

```text
Find Approve Action For Order #1001
```

The second requires component scoping.

---

# Component Scope

For complex applications:

```text
Page
 ↓
Order Row
 ↓
Button
```

is usually more stable than:

```text
Page
 ↓
Button
```

when many buttons exist.

---

# Role Locator Composition

The architecture becomes:

```text
Page
 ↓
getByRole("row")
 ↓
Filter by order identity
 ↓
getByRole("button")
 ↓
Filter by action
```

The exact implementation syntax will be covered in subsequent locator lessons.

For now, focus on the architecture.

---

# Why Role Locators Improve Readability

Compare:

```text
Find button with CSS class "btn-primary"
```

versus:

```text
Find the "Place Order" button
```

The second is understandable to:

```text
Developer
Tester
Product Owner
Reviewer
Architect
```

without requiring knowledge of CSS implementation.

---

# Role Locators and Code Reviews

During code review, this:

```text
Role + Meaning
```

is easier to evaluate than:

```text
Complex CSS Structure
```

A reviewer can immediately ask:

```text
Is this the correct business control?
```

---

# Role Locators and Refactoring

Suppose developers refactor:

```text
div
 └── span
      └── button
```

into:

```text
section
 └── button
```

A semantic role locator may continue to work.

A deep structural XPath may fail.

This is a major maintainability advantage.

---

# When `getByRole()` Should NOT Be Forced

Role-based locating is powerful, but it is not a religion.

Do not force `getByRole()` when:

```text
No Meaningful Role Exists

OR

The Application's Test Contract Is Better Represented By A Stable Test ID

OR

The Element Is Primarily Identified By Another Stable Contract
```

---

# Example: Complex Non-Interactive Container

Suppose a container has:

```text
data-testid="order-summary"
```

and its internal structure is intentionally an implementation detail.

A stable test ID may be more appropriate for the component boundary.

---

# Role Locator vs Test ID

Think:

```text
Role
 ↓
User / Accessibility Semantics
```

while:

```text
Test ID
 ↓
Automation Contract
```

Both can be valuable.

---

# Enterprise Locator Decision

```text
Is there a clear user-facing semantic role?
          │
          ├── YES
          │    ↓
          │  Prefer Role
          │
          └── NO
               ↓
       Is there a stable test contract?
               │
               ├── YES → Test ID
               │
               └── NO → Other stable strategy
```

---

# Role Locators and Accessibility Testing

Role-based automation can indirectly encourage better accessibility.

For example, if an important control cannot be meaningfully described as:

```text
Role
+
Name
```

the development team may need to investigate the component's semantics.

This does not replace formal accessibility testing.

But it is a valuable signal.

---

# Enterprise Role Standards

A large organization may define rules such as:

```text
Buttons
    ↓
Role + Accessible Name

Forms
    ↓
Label + Control

Navigation
    ↓
Navigation Region + Link

Dialogs
    ↓
Dialog + Name

Tables
    ↓
Table + Row + Business Identity
```

Such conventions make the automation suite consistent.

---

# Workflow

```text
Business Requirement
        ↓
Identify UI Component
        ↓
Determine Semantic Role
        ↓
Determine Accessible Name
        ↓
Check Uniqueness
        ↓
Add State / Scope If Needed
        ↓
Create Role-Based Locator
        ↓
Perform Action / Assertion
        ↓
Validate Result
```

---

# Enterprise Workflow

```text
Developer
    ↓
Semantic HTML / ARIA
    ↓
Browser
    ↓
Accessibility Semantics
    ↓
Playwright
    ↓
getByRole()
    ↓
Locator Resolution
    ↓
Actionability
    ↓
Action
    ↓
Assertion
```

---

# Architecture

```text
                       Web Application
                              │
                              ▼
                             DOM
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
        Native HTML          ARIA          Visible Text
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                    Accessibility Semantics
                              │
                              ▼
                    Accessibility Tree
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                   Role           Accessible Name
                    │                   │
                    └─────────┬─────────┘
                              ▼
                         getByRole()
                              │
                              ▼
                          Locator
                              │
                              ▼
                    Action / Assertion
```

---

# Enterprise Perspective

Enterprise automation teams increasingly want tests that describe application behavior rather than DOM implementation.

Compare:

```text
div:nth-child(3) button:nth-child(2)
```

with:

```text
Approve button
```

The second expresses business intent.

This produces several advantages:

```text
Readable Tests
       ↓
Better Reviews
       ↓
Lower Maintenance
       ↓
Better Accessibility Alignment
       ↓
More Resilient Automation
```

However, enterprise teams should not blindly mandate one locator type.

A mature policy is:

```text
Use Role When Semantic Meaning Is Strong

↓

Use Test ID When A Stable Automation Contract Is Appropriate

↓

Use Other Selectors When They Represent Stable Intent
```

---

# Best Practices

1. Prefer semantic role locators for meaningful interactive elements.
2. Combine role with accessible name when needed.
3. Prefer native semantic HTML.
4. Understand implicit roles.
5. Understand explicit ARIA roles.
6. Do not use ARIA as a substitute for proper native HTML when native HTML is appropriate.
7. Validate the accessible name.
8. Use exact matching when ambiguity requires it.
9. Scope role locators to components.
10. Use role and state when state is part of the requirement.
11. Treat strictness errors as locator-quality feedback.
12. Use role locators for buttons, links, headings, forms, dialogs, and other semantic controls.
13. Use stable test IDs for appropriate component contracts.
14. Do not force `getByRole()` for every element.
15. Review Codegen-generated locators.
16. Use Playwright Inspector to understand locator behavior.
17. Keep locator intent obvious in code reviews.
18. Encourage developers to build accessible, testable components.
19. Avoid DOM-position-based role alternatives.
20. Think about user behavior before selector syntax.

---

# Common Beginner Mistakes

## Mistake 1 — Using Only the Role

If the page contains:

```text
Save
Cancel
Delete
```

then:

```text
button
```

may be ambiguous.

Use the accessible name when appropriate.

---

## Mistake 2 — Confusing Text With Accessible Name

Visible text can contribute to accessible naming, but accessible-name computation can involve multiple sources.

Do not assume:

```text
Visible Text
=
Accessible Name
```

in every situation.

---

## Mistake 3 — Treating Every `div` as a Meaningful Role

A generic container is not automatically a button, link, dialog, or textbox.

---

## Mistake 4 — Adding ARIA Instead of Using Native HTML

If a native button is appropriate, prefer a native button rather than constructing a fake button from a generic element.

---

## Mistake 5 — Using Role Without Understanding the Widget

A custom dropdown may require understanding:

```text
combobox
listbox
option
```

rather than simply searching for text.

---

## Mistake 6 — Blindly Trusting Codegen

Generated locators are suggestions.

Review them.

---

## Mistake 7 — Ignoring Scope

If there are multiple dialogs or tables, a global role locator may be ambiguous.

---

## Mistake 8 — Assuming Role Means Visual Appearance

A component can look like a button without exposing correct button semantics.

---

## Mistake 9 — Using Role Locators Without Accessibility Awareness

Role-based automation is strongest when the application has meaningful semantics.

---

## Mistake 10 — Using `nth()` Immediately After Finding Multiple Roles

First ask:

```text
Why are there multiple matches?
```

Then determine whether the locator should be scoped or refined.

---

# Professional Tips

A senior automation engineer should be able to look at a UI and mentally translate it into:

```text
Role
+
Name
+
State
+
Scope
```

For example:

```text
Orders Page

↓

Navigation

↓

Orders Link
```

or:

```text
Order Dialog

↓

Dialog

↓

Heading: "Delete Order"

↓

Button: "Confirm"
```

or:

```text
Orders Table

↓

Row containing Order #1001

↓

Button: "Approve"
```

This mental translation is more important than memorizing locator syntax.

---

# Real Interview Discussion

## Junior-Level

### Q1. What is `getByRole()`?

**Answer:**

`getByRole()` is a Playwright locator API that identifies elements according to their accessible role and can optionally use properties such as accessible name and state to narrow the target.

---

### Q2. What is an accessible role?

**Answer:**

It describes the semantic type of an element as exposed through the accessibility model, such as button, link, heading, checkbox, textbox, dialog, or navigation.

---

### Q3. Why is role-based locating useful?

**Answer:**

It expresses user-facing meaning rather than relying heavily on DOM structure or CSS implementation details. This can improve readability, stability, and accessibility alignment.

---

# Mid-Level

### Q4. What is the difference between role and accessible name?

**Answer:**

The role describes what the element is, such as `button`. The accessible name identifies that particular element, such as `Save Order`.

Together:

```text
Role = button
Name = Save Order
```

can identify a specific control.

---

### Q5. What are implicit roles?

**Answer:**

Implicit roles are semantic roles provided by native HTML elements without requiring an explicit ARIA role declaration.

---

### Q6. What are explicit roles?

**Answer:**

Explicit roles are semantic roles declared through accessibility mechanisms such as ARIA. They should be used correctly and should not unnecessarily replace appropriate native HTML semantics.

---

# Senior-Level

### Q7. Why would you prefer a role locator over a CSS selector?

**Answer:**

When the element has strong semantic meaning, a role locator usually expresses user intent more clearly and can remain stable across changes to classes or DOM structure.

---

### Q8. What if `getByRole()` matches multiple elements?

**Answer:**

I would first determine why the locator is ambiguous. I would refine it using accessible name, state, component scope, or another stable contract. I would not immediately use `nth()` unless position is actually part of the requirement.

---

### Q9. How would you locate an Approve button for a specific order?

**Answer:**

I would first locate the row or component representing the specific order, then scope the search within that component for the Approve button. This avoids ambiguity when multiple Approve buttons exist on the page.

---

# Lead-Level

### Q10. How would you introduce role-based locator standards across an organization?

**Answer:**

I would define a locator hierarchy, establish semantic HTML and accessibility expectations with developers, encourage role-and-name locators for interactive controls, define when test IDs are appropriate, provide code-review guidelines, and include locator quality in framework standards.

---

### Q11. How does accessibility affect automation?

**Answer:**

Accessible roles, names, and states provide meaningful identifiers for UI elements. Better accessibility semantics often make automation more stable and understandable. However, role-based automation does not replace dedicated accessibility testing.

---

# Architect-Level

### Q12. Why should locator strategy be considered an application architecture concern?

**Strong Answer:**

Because locator stability depends on how the application exposes semantics and testability contracts. If the UI is built with meaningful semantic HTML, accessible names, stable identifiers, and predictable component boundaries, automation becomes cheaper to maintain. Therefore locator architecture should involve developers, QA, accessibility, and framework architects rather than being treated solely as a tester concern.

---

# Knowledge Check

Before continuing, answer these:

1. What is a role?
2. What is `getByRole()`?
3. What is an accessible name?
4. Why are role and name often used together?
5. What is an implicit role?
6. What is an explicit role?
7. What is ARIA?
8. Why is native semantic HTML preferred?
9. What is the difference between a button and a link?
10. What is a textbox?
11. What is a checkbox?
12. What is a radio button?
13. What is a combobox?
14. What is a listbox?
15. What is an option?
16. What is a dialog?
17. What is a navigation region?
18. What is a table role?
19. What is a row role?
20. What is a cell role?
21. What is an accessibility tree?
22. How is the accessibility tree related to the DOM?
23. Why does `getByRole()` align well with accessibility?
24. Why can role alone be ambiguous?
25. Why is accessible name important?
26. What is role + name?
27. What is role + state?
28. Why is scoping useful?
29. Why should `nth()` not be the first solution to ambiguity?
30. When might a test ID be preferable to a role locator?
31. Why should Codegen output be reviewed?
32. How can role-based locators survive DOM refactoring?
33. How would you locate an Order #1001 row?
34. How would you find Approve within that row?
35. How would you design enterprise role-locator standards?

---

# Step Summary

You have now mastered the conceptual foundation of:

```text
getByRole()
```

The core mental model is:

```text
Element
   ↓
Semantic Role
   ↓
Accessible Name
   ↓
State
   ↓
Scope
   ↓
Locator
```

The most important distinction is:

```text
Role
=
What is this?

Accessible Name
=
Which one is it?
```

For example:

```text
button
+
"Place Order"
```

is much more meaningful than:

```text
button:nth-child(4)
```

You also learned:

```text
DOM
 ↓
Semantic Interpretation
 ↓
Accessibility Tree
 ↓
Role + Name
 ↓
Playwright Locator
```

This is why role-based locators can provide strong resilience against irrelevant DOM and CSS changes.

But remember:

> **Do not use `getByRole()` simply because it is popular. Use it when the element's accessible semantics represent the most stable and meaningful test contract.**

A mature enterprise locator strategy uses:

```text
Role
+
Accessible Name
+
State
+
Component Scope
```

when appropriate.

---

# Progress Milestone

✅ **Step 60 completed.**

You have now mastered:

- `getByRole()`
- Accessible roles
- Accessible names
- Implicit roles
- Explicit roles
- Native HTML semantics
- ARIA concepts
- Buttons
- Links
- Headings
- Textboxes
- Checkboxes
- Radio buttons
- Comboboxes
- Listboxes
- Options
- Dialogs
- Navigation
- Main regions
- Banner regions
- Contentinfo
- Tables
- Rows
- Cells
- Tabs
- Menus
- Accessibility tree concepts
- Role filtering
- Role + name
- Role + state
- Role scoping
- Role-based debugging
- Codegen review
- Accessibility-driven automation
- Enterprise role locator standards

---

# Next Step

# Step 61 — Mastering `getByText()`: Visible Text, Text Matching, Exact Matching, Substrings, Nested Text, Whitespace, Case Sensitivity, Dynamic Text, Localization, Repeated Text, Scope, Filtering, and Enterprise Text-Locator Strategy

The next lesson will answer:

```text
When should I locate an element by its visible text?
```

You will learn why:

```text
"Orders"
```

can sometimes be an excellent locator,

but in other situations:

```text
"Orders"
```

can be dangerously ambiguous.

We will deeply examine:

```text
Visible Text
      ↓
Text Nodes
      ↓
Whitespace
      ↓
Exact vs Partial Matching
      ↓
Nested Elements
      ↓
Dynamic Text
      ↓
Repeated Text
      ↓
Localization
      ↓
Component Scope
      ↓
Filtering
      ↓
Enterprise Text Locator Design
```
```
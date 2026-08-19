# Playwright with TypeScript Mastery Bootcamp

## From Absolute Beginner to Enterprise Automation Architect

**Version:** 1.0

---

# Stage 24 — User Actions and Browser Interaction

# Step 51 — Mastering Playwright Actions: Click, Fill, Press, Check, Uncheck, Select, Hover, Focus, Drag and Drop, Keyboard Actions, Mouse Actions, Scroll, Double Click, Right Click, Force Actions, Actionability Checks, Navigation, and Reliable User Interaction Design

---

# Objective

In this lesson, you will learn how Playwright models real user interaction with web applications.

You will understand:

- What a Playwright action is
- Why actions are different from raw DOM manipulation
- `click`
- `dblclick`
- Right-click
- `fill`
- `press`
- `check`
- `uncheck`
- `selectOption`
- `hover`
- `focus`
- `blur`
- `dragTo`
- Mouse actions
- Keyboard actions
- Scrolling
- Navigation actions
- Actionability checks
- Visibility
- Stability
- Enabled state
- Receives-events checks
- Editable state
- Force actions
- Action timeouts
- Action failure diagnosis
- Reliable interaction patterns
- Enterprise interaction architecture

The most important goal is to understand:

```text
Locator

↓

Actionability

↓

Action

↓

Application State Change

↓

Assertion
```

rather than thinking:

```text
Find Element

↓

Immediately Click
```

---

# Before We Start

Imagine a human employee using a banking application.

Suppose the employee needs to click:

```text
Transfer Money
```

A human does not blindly click an arbitrary coordinate.

The human first:

```text
Looks for the button

↓

Confirms it is visible

↓

Confirms it is not covered

↓

Confirms it can be interacted with

↓

Clicks it
```

Playwright follows a similar philosophy.

Before performing many actions, Playwright checks whether the target is actually ready for interaction.

---

# The Problem

Traditional UI automation often suffers from:

```text
Find Element

↓

Click Immediately

↓

Element Not Ready

↓

Failure
```

Or:

```text
Find Element

↓

Element Covered by Overlay

↓

Click Fails
```

Or:

```text
Element Exists

↓

Animation Still Running

↓

Click Happens Too Early
```

These problems create flaky tests.

Playwright addresses many of them through:

```text
Actionability Checks
```

---

# What is an Action?

An action is an operation that interacts with a web application.

Examples:

```text
Click

↓

Fill

↓

Check

↓

Select

↓

Hover

↓

Press

↓

Drag
```

Actions simulate meaningful user interaction rather than simply manipulating the DOM.

---

# Why Actions Matter

A test should model:

```text
User Behavior
```

For example:

```text
User Opens Login Page

↓

Enters Username

↓

Enters Password

↓

Clicks Login

↓

Sees Dashboard
```

This is more valuable than directly modifying:

```text
input.value
```

because the purpose of end-to-end automation is to validate the real application interaction path.

---

# Action Lifecycle

A simplified Playwright action lifecycle is:

```text
Locator

↓

Resolve Target

↓

Actionability Checks

↓

Scroll Into View If Needed

↓

Perform Interaction

↓

Wait For Appropriate Completion

↓

Continue
```

This architecture is central to Playwright reliability.

---

# Actionability Checks

For many actions, Playwright verifies conditions such as:

```text
Visible

↓

Stable

↓

Receives Events

↓

Enabled
```

For editable operations, additional conditions such as:

```text
Editable
```

may be relevant.

The exact checks depend on the specific action.

---

# Visible

An element may exist in the DOM but not actually be visible to the user.

For example:

```text
Element Exists

↓

display: none
```

or:

```text
Element Exists

↓

Hidden Behind UI
```

A user cannot interact with it normally.

Therefore, Playwright considers visibility as part of actionability for appropriate actions.

---

# Stable

An element may be visible but still moving.

For example:

```text
Button

↓

Animation

↓

Position Changing
```

Clicking while the element is moving can be unreliable.

Playwright can wait for the element to become stable before interacting.

---

# Receives Events

An element may be visible but covered by another element.

For example:

```text
Submit Button

↓

Loading Overlay

↓

User Cannot Actually Click Button
```

The target may exist and be visible,

but it does not receive the mouse event.

Playwright can detect this type of condition.

---

# Enabled

A button may be visible but disabled.

For example:

```text
Submit

↓

Disabled
```

A human cannot normally use it.

Playwright checks the appropriate enabled state before actions such as clicking.

---

# Editable

For actions that modify input values,

the control generally needs to be editable.

For example:

```text
Username Input

↓

Editable
```

versus:

```text
Username Input

↓

Read Only
```

The action requirements depend on the operation being performed.

---

# Actionability Concept

The important mental model is:

```text
Element Exists

≠

Element Is Ready For Interaction
```

This distinction is fundamental.

---

# Click

`click` represents a mouse click interaction.

Conceptually:

```text
Locate Button

↓

Verify Actionability

↓

Click
```

A click may trigger:

```text
Navigation

↓

Modal

↓

API Request

↓

State Change

↓

Validation
```

---

# Why Click Is More Than a DOM Event

A real user clicks a visual target.

Playwright's click interaction accounts for browser interaction conditions.

This is different from simply executing arbitrary JavaScript against the DOM.

---

# Double Click

A double click represents two clicks performed as a double-click interaction.

Common use cases include:

```text
Open File

↓

Edit Table Row

↓

Expand Item
```

The application must actually support double-click behavior for this to make sense.

---

# Right Click

A right-click can be used to open contextual menus.

Conceptually:

```text
Target

↓

Right Click

↓

Context Menu
```

This is useful for applications that implement custom context menus.

---

# Fill

`fill` is designed for entering text into editable form controls.

Conceptually:

```text
Input

↓

Clear Existing Value

↓

Enter New Value
```

It is generally preferable to manually simulating every individual keystroke when the requirement is simply to populate a field.

---

# Why Fill Is Powerful

Suppose an input currently contains:

```text
Old Value
```

The requirement is:

```text
New Value
```

A filling operation conceptually performs:

```text
Old Value

↓

Replace

↓

New Value
```

This makes tests concise and deterministic.

---

# Fill vs Keyboard Typing

There is an important distinction.

### Fill

Models:

```text
Set the input's value
```

### Keyboard Typing

Models:

```text
Individual keyboard interaction
```

Use `fill` when you only care about entering a value.

Use keyboard interaction when the application behavior depends on actual key events.

---

# When Keyboard Interaction Matters

Examples:

```text
Press Enter to Search

↓

Press Escape to Close

↓

Keyboard Shortcut

↓

Arrow Navigation

↓

Tab Navigation
```

In such cases, keyboard actions are part of the business behavior.

---

# Press

`press` sends a keyboard key or key combination to the target.

Examples conceptually include:

```text
Enter

Escape

Tab

ArrowDown

Control+A
```

The key depends on the required workflow.

---

# Keyboard Architecture

Conceptually:

```text
Locator

↓

Keyboard Event

↓

Browser

↓

Application Event Handler

↓

UI State Change
```

This allows testing of keyboard-driven interactions.

---

# Check

`check` is designed for checkboxes and compatible controls.

Conceptually:

```text
Checkbox

↓

Unchecked

↓

Check

↓

Checked
```

Playwright can ensure the control reaches the desired checked state.

---

# Why Check Is Better Than Clicking a Checkbox Blindly

A normal click means:

```text
Toggle
```

Depending on current state:

```text
Unchecked → Checked
```

or:

```text
Checked → Unchecked
```

But `check` communicates the desired state:

```text
Ensure Checked
```

This makes test intent clearer.

---

# Uncheck

`uncheck` similarly communicates:

```text
Ensure Unchecked
```

rather than:

```text
Toggle
```

This is more deterministic.

---

# Radio Buttons

Radio buttons represent mutually exclusive choices.

Conceptually:

```text
Option A

Option B

Option C
```

Selecting:

```text
Option B
```

should result in:

```text
A = Not Selected

B = Selected

C = Not Selected
```

Playwright provides appropriate interaction methods for checkable controls.

---

# Select Option

`selectOption` is designed for native HTML `<select>` elements.

Conceptually:

```text
Select

↓

Option A

Option B

Option C

↓

Select Option B
```

This should not be confused with custom dropdown components.

---

# Native Select vs Custom Dropdown

This distinction is extremely important.

### Native Select

```text
<select>
```

Use:

```text
selectOption
```

### Custom Dropdown

May be:

```text
div

↓

button

↓

list

↓

option-like elements
```

This requires normal locator and interaction strategies.

---

# Why This Distinction Matters

A custom dropdown is not necessarily an HTML `<select>`.

Therefore:

```text
selectOption

↓

May Not Work
```

The test must interact with the actual component behavior.

---

# Hover

Hover moves the pointer over an element.

Common use cases:

```text
Tooltip

↓

Mega Menu

↓

Hover Actions

↓

Hidden Controls
```

Conceptually:

```text
Locate Element

↓

Move Pointer Over Element

↓

UI Responds
```

---

# Hover Example Concept

Imagine:

```text
Products
```

When hovered:

```text
Electronics
Clothing
Furniture
```

A hover action can trigger this menu.

---

# Focus

Focus places keyboard focus on an element.

This is useful for testing:

```text
Keyboard Navigation

↓

Focus Styling

↓

Accessibility

↓

Keyboard-Driven Behavior
```

---

# Why Focus Matters

Some applications react to focus events:

```text
Input Focused

↓

Validation Message

↓

Autocomplete Appears
```

Focus behavior can therefore be part of the application contract.

---

# Blur

Blur occurs when an element loses focus.

Applications may validate fields when focus leaves them.

For example:

```text
Email Input

↓

User Leaves Field

↓

Validation
```

Testing this behavior may require focus transitions.

---

# Drag and Drop

Drag-and-drop workflows can involve:

```text
Source

↓

Pointer Down

↓

Movement

↓

Target

↓

Pointer Up
```

Playwright provides higher-level drag-and-drop support.

---

# Drag and Drop Use Cases

Examples:

```text
Kanban Board

↓

Move Card

↓

Drop Into Column
```

or:

```text
Upload Area

↓

Drag File

↓

Drop
```

The exact behavior depends on the application's implementation.

---

# Why Drag and Drop Can Be Difficult

Drag-and-drop implementations vary.

Some use:

```text
HTML Drag and Drop API
```

Others use:

```text
Pointer Events

↓

Custom JavaScript
```

Therefore, a strategy that works for one application may not work for another.

---

# Mouse Actions

Playwright exposes mouse interaction through the Page's mouse interface.

Conceptually:

```text
Mouse

├── Move
├── Down
├── Up
└── Click
```

Mouse operations are lower-level than normal Locator actions.

---

# When to Use Mouse APIs

Use lower-level mouse operations when the business behavior genuinely requires pointer control.

Examples:

```text
Canvas

↓

Drawing

↓

Drag

↓

Resize

↓

Custom Gesture
```

For ordinary buttons,

prefer Locator-based actions.

---

# Keyboard Actions

Keyboard APIs can model:

```text
Key Press

↓

Key Combination

↓

Typing

↓

Navigation
```

Use them when keyboard behavior itself matters.

---

# Keyboard Accessibility Testing

Keyboard interactions are especially important for accessibility.

Examples:

```text
Tab

↓

Focus Button

↓

Enter

↓

Activate
```

Testing keyboard navigation can reveal usability and accessibility problems.

---

# Scrolling

Users may scroll pages to access content.

Playwright can interact with elements that may require scrolling into view.

For many locator actions,

Playwright handles scrolling as part of the interaction process.

---

# Why Manual Scrolling Is Often Unnecessary

A common beginner pattern is:

```text
Wait

↓

Scroll

↓

Wait

↓

Click
```

For many ordinary interactions,

this is unnecessary.

Playwright can scroll a target into view when required by the action.

---

# When Explicit Scrolling Is Useful

Explicit scrolling can be meaningful when the test specifically validates:

```text
Infinite Scroll

↓

Lazy Loading

↓

Scroll-Triggered Behavior

↓

Sticky Headers
```

In those cases,

scrolling itself is part of the business behavior.

---

# Navigation Actions

Navigation can occur through:

```text
Direct URL Navigation

↓

Link Click

↓

Form Submission

↓

Browser History

↓

Application Routing
```

A Playwright test may navigate directly when setup requires it.

---

# Direct Navigation vs User Navigation

### Direct Navigation

```text
Open Target URL
```

Useful for:

```text
Setup

↓

Deep Linking

↓

Specific Page Tests
```

### User Navigation

```text
Click Menu

↓

Click Link

↓

Reach Page
```

Useful when the navigation path itself is being tested.

---

# Navigation Is a Business Decision

Ask:

```text
Am I testing the destination page?

```

Then direct navigation may reduce unnecessary setup.

Ask:

```text
Am I testing the navigation flow?

```

Then interact with the UI.

---

# Action Timeout

Actions do not wait forever.

Playwright has action timeout concepts controlling how long an action can wait for required conditions.

If the target never becomes actionable:

```text
Timeout

↓

Action Failure
```

---

# Why Action Timeout Is Useful

Without timeouts:

```text
Test

↓

Wait Forever
```

This would cause pipelines to hang.

Timeouts provide bounded failure.

---

# Timeout Is Not a Synchronization Strategy

This is an important distinction.

Bad strategy:

```text
Wait 10 seconds

↓

Click
```

Better:

```text
Wait Until Target Is Actionable

↓

Click
```

The second approach is condition-based rather than time-based.

---

# Force Actions

Playwright provides a `force` option for certain actions.

Conceptually:

```text
Normal Action

↓

Actionability Checks

↓

Interaction
```

versus:

```text
Force Action

↓

Bypass Certain Actionability Checks

↓

Interaction
```

---

# Why Force Is Dangerous

Suppose a button is covered by:

```text
Loading Overlay
```

Normal Playwright behavior may wait or fail.

A forced click may interact with the DOM despite the visual condition.

This can produce a test that passes while a real user could not perform the action.

---

# When Force May Be Justified

Force may be appropriate when:

```text
The Application Intentionally Uses Atypical Interaction

+

The Test Engineer Understands The Behavior

+

Normal Actionability Does Not Represent The Correct Test Model
```

This should be an exception.

---

# Force Is Not a Fix for Bad Locators

Do not do:

```text
Locator Wrong

↓

Force Click
```

That hides the real problem.

---

# Action Errors

When an action fails,

classify the problem.

Possible causes:

```text
Locator Ambiguous

↓

Element Missing

↓

Element Hidden

↓

Element Disabled

↓

Element Moving

↓

Element Covered

↓

Wrong Frame

↓

Wrong Page

↓

Application Failure
```

This diagnostic mindset is essential.

---

# Actionability Failure

A failure may effectively communicate:

```text
The target exists,

but it is not ready for the requested interaction.
```

That is valuable information.

---

# Example Failure Reasoning

Suppose:

```text
Click Submit
```

fails.

Do not immediately increase timeout.

Ask:

```text
Is Submit visible?

↓

Is it enabled?

↓

Is it covered?

↓

Is it moving?

↓

Is the locator unique?

↓

Is it in the correct context?
```

Only after understanding the reason should configuration be changed.

---

# Action + Assertion

A robust test generally follows:

```text
Locate

↓

Interact

↓

Assert Result
```

For example:

```text
Fill Login Form

↓

Click Login

↓

Verify Dashboard
```

The action changes state.

The assertion verifies the expected state.

---

# Actions Are Not Assertions

This distinction matters.

```text
Click Login
```

does not prove:

```text
Login Successful
```

You need an assertion.

For example conceptually:

```text
Click Login

↓

Verify Dashboard
```

---

# Action Idempotency

Some actions are state-changing.

For example:

```text
Click Toggle
```

may alternate state.

Others are intended to establish a state:

```text
Check Checkbox
```

This distinction matters when designing reliable tests.

---

# Check vs Click as an Example

Suppose the checkbox is already checked.

```text
click()

↓

Could uncheck it
```

Whereas:

```text
check()

↓

Ensures checked
```

Therefore state-oriented actions are often more deterministic.

---

# Reliable Interaction Model

A good Playwright interaction looks like:

```text
Stable Locator

↓

Correct Context

↓

Actionability

↓

Action

↓

Expected State

↓

Assertion
```

This is the foundation of reliable UI automation.

---

# Enterprise Interaction Architecture

A mature framework might represent:

```text
Test

↓

Business Action

↓

Page Object

↓

Locator

↓

Playwright Action

↓

Browser
```

For example:

```text
Test

↓

"Create Order"

↓

OrderPage.createOrder()

↓

Fill Customer

↓

Select Product

↓

Click Submit

↓

Verify Confirmation
```

The test remains business-readable.

---

# Page Object Responsibility

Page Objects should encapsulate interaction details.

For example:

```text
OrderPage

↓

fillCustomer()

↓

selectProduct()

↓

submitOrder()
```

The test should not need to understand every low-level mouse operation.

---

# Component Interaction

Reusable components can encapsulate common interactions.

For example:

```text
DatePickerComponent

↓

Open Calendar

↓

Select Month

↓

Select Day
```

Then many pages can reuse the same interaction abstraction.

---

# Workflow

```text
Business Requirement

↓

Test Scenario

↓

Page / Component

↓

Stable Locator

↓

Actionability Checks

↓

User Action

↓

Application Processing

↓

Assertion
```

---

# Architecture

```text
                    Test Scenario
                          │
                          ▼
                   Page / Component
                          │
                          ▼
                       Locator
                          │
                          ▼
                 Actionability Layer
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Visible       Stable      Enabled
             │            │            │
             └────────────┼────────────┘
                          ▼
                       Action
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
           Browser      Events      Network
              │           │           │
              └───────────┼───────────┘
                          ▼
                    Application State
                          │
                          ▼
                      Assertion
```

---

# Enterprise Perspective

Large organizations care about actions because unreliable interactions create:

```text
Flaky Tests

↓

False Failures

↓

Engineer Investigation

↓

Pipeline Delays

↓

Release Risk
```

A good automation framework therefore standardizes:

```text
Locator Quality

↓

Action Strategy

↓

Synchronization

↓

Error Handling

↓

Assertion Strategy
```

---

# Best Practices

Follow these principles:

1. Prefer Locator-based actions.
2. Let Playwright perform actionability checks.
3. Use `fill` for straightforward input population.
4. Use keyboard actions when keyboard behavior matters.
5. Use `check` and `uncheck` for checkbox state.
6. Use `selectOption` for native `<select>` elements.
7. Treat custom dropdowns as normal UI components.
8. Use hover only when hover behavior is relevant.
9. Use mouse APIs for genuinely pointer-driven interactions.
10. Avoid arbitrary scrolling.
11. Avoid arbitrary delays.
12. Use direct navigation when navigation itself is not under test.
13. Use UI navigation when navigation behavior is under test.
14. Treat `force` as an exception.
15. Never use `force` to hide a bad locator.
16. Understand actionability failures before increasing timeouts.
17. Follow actions with meaningful assertions.
18. Prefer state-setting actions over ambiguous toggles where available.

---

# Common Beginner Mistakes

### Mistake 1 — Adding Sleeps Everywhere

```text
Click

↓

Wait 5 seconds

↓

Click
```

This creates slow and brittle tests.

---

### Mistake 2 — Force Clicking Everything

```text
Click fails

↓

force: true
```

This hides real application or locator problems.

---

### Mistake 3 — Clicking Checkboxes to Reach a State

```text
click()

↓

Maybe Checked
```

Use state-oriented interactions where appropriate.

---

### Mistake 4 — Using Keyboard Actions for Everything

Keyboard interaction should be used when keyboard behavior matters.

---

### Mistake 5 — Using Mouse APIs for Normal Buttons

Prefer:

```text
Locator

↓

Click
```

over low-level coordinates for ordinary UI interactions.

---

### Mistake 6 — Treating Custom Dropdowns as Native Selects

`selectOption` is for native select controls.

---

### Mistake 7 — Testing Without Assertions

```text
Click Submit

↓

Test Ends
```

This does not prove the business operation succeeded.

---

# Professional Tips

An experienced SDET asks:

```text
What is the user trying to accomplish?
```

Then chooses the appropriate action.

Not:

```text
Which Playwright command can I use?
```

For example:

```text
Requirement:
Ensure newsletter is enabled.

↓

Use state-oriented checkbox interaction.
```

Rather than:

```text
Click checkbox.
```

The first describes intent.

---

# Real Interview Discussion

### Junior-Level Question

**Q:** What is actionability in Playwright?

**Answer:**

Actionability refers to Playwright verifying that an element is in a suitable state for interaction before performing supported actions. Depending on the action, this can involve checks such as visibility, stability, event reception, enabled state, or editability.

---

### Mid-Level Question

**Q:** What is the difference between `fill()` and keyboard typing?

**Answer:**

`fill()` is intended to set the value of an editable field efficiently, while keyboard actions model actual key interactions. Keyboard interaction is preferable when the application behavior depends on specific keyboard events such as Enter, Escape, Tab, or shortcuts.

---

### Senior-Level Question

**Q:** Why should `force: true` not be used routinely?

**Answer:**

Force can bypass certain actionability checks. It may allow a test to interact with an element that a real user could not interact with, such as an element covered by an overlay. Therefore it should be used only when the atypical behavior is understood and intentional.

---

### Lead-Level Question

**Q:** How would you design reliable UI actions in a Page Object Model?

**Answer:**

I would expose business-level methods from Page Objects, use stable Locators internally, rely on Playwright's built-in actionability and synchronization, avoid arbitrary waits, and keep assertions primarily in the test or appropriate validation abstraction.

---

### Architect-Level Question

**Q:** How would you reduce action-related flakiness across a large Playwright framework?

**Answer:**

I would establish locator standards, rely on Playwright's actionability model, eliminate arbitrary sleeps, minimize forced actions, standardize page/component abstractions, classify interaction failures, monitor flaky-test patterns, and review synchronization problems at the framework and application levels rather than repeatedly increasing timeouts.

---

# Knowledge Check

Answer these questions before continuing:

1. What is a Playwright action?
2. Why are actions different from raw DOM manipulation?
3. What is actionability?
4. What does visibility mean in the context of interaction?
5. Why does element stability matter?
6. What does "receives events" mean?
7. Why does enabled state matter?
8. When is editability relevant?
9. What does `click()` do conceptually?
10. What is the purpose of `dblclick()`?
11. When would you use a right-click?
12. What is `fill()`?
13. How does `fill()` differ conceptually from keyboard typing?
14. What is `press()`?
15. Why are `check()` and `uncheck()` preferable to blind checkbox clicks in state-based scenarios?
16. When should `selectOption()` be used?
17. Why does it not normally work for custom dropdowns?
18. What is `hover()` useful for?
19. What are mouse APIs useful for?
20. When should keyboard APIs be used?
21. Why is arbitrary scrolling often unnecessary?
22. When is explicit scrolling meaningful?
23. When should direct navigation be used?
24. When should UI navigation be tested?
25. What is an action timeout?
26. Why is a timeout not a synchronization strategy?
27. What does `force` do conceptually?
28. Why can force actions hide real problems?
29. Why should every important state-changing action usually be followed by validation?
30. How would you design an enterprise-grade interaction strategy?

---

# Step Summary

You have now learned how Playwright performs browser interactions.

The key architecture is:

```text
Locator

↓

Resolve Target

↓

Actionability Checks

↓

Interaction

↓

Application State Change

↓

Assertion
```

You studied:

```text
click

dblclick

right-click

fill

press

check

uncheck

selectOption

hover

focus

blur

drag and drop

mouse actions

keyboard actions

scrolling

navigation

force actions
```

You also learned the most important principle of Playwright interaction:

> **The existence of an element does not mean that the element is ready for interaction.**

Playwright's actionability model bridges that gap.

Therefore:

```text
DOM Element Exists

≠

User Can Interact With It
```

A reliable automation engineer understands the difference.

The second major principle is:

> **Do not replace synchronization with arbitrary waiting.**

Prefer:

```text
Condition

↓

Auto-Waiting

↓

Action
```

over:

```text
Fixed Delay

↓

Action
```

And finally:

```text
Action

≠

Validation
```

A click only performs an interaction.

A strong test verifies the resulting business state.

---

# Progress Milestone

✅ You have completed **Step 51** of approximately **230** planned learning steps.

**What you've mastered:**

- Playwright Actions
- `click`
- `dblclick`
- Right Click
- `fill`
- `press`
- `check`
- `uncheck`
- `selectOption`
- Native vs Custom Dropdowns
- `hover`
- `focus`
- `blur`
- Drag and Drop
- Mouse Actions
- Keyboard Actions
- Scrolling
- Navigation
- Actionability Checks
- Visibility
- Stability
- Receives Events
- Enabled State
- Editability
- Action Timeouts
- Force Actions
- Action Failure Analysis
- Reliable Interaction Design
- Enterprise Action Architecture

**Coming next — Step 52:**

**Mastering Playwright Assertions: `expect`, Web Assertions, Page Assertions, Locator Assertions, API Assertions, Soft Assertions, Negated Assertions, Polling, Retry Behavior, Assertion Timeout, State Assertions, Text Assertions, Attribute Assertions, URL Assertions, Screenshot Assertions, Accessibility Assertions, and Enterprise Validation Strategy.**

The next lesson will establish the second half of reliable automation:

```text
Action

↓

Expected State

↓

Assertion

↓

Confidence
```

You will learn why a test that performs actions without strong assertions is not actually validating the application.
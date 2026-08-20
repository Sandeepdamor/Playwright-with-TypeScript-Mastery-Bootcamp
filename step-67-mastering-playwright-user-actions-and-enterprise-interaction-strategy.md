# Playwright with TypeScript Mastery Bootcamp

# Stage 30 — Actions, Actionability, and Synchronization

# Step 67 — Mastering Playwright User Actions: `click()`, `dblclick()`, `fill()`, `type()`, `press()`, `check()`, `uncheck()`, `selectOption()`, `hover()`, `focus()`, `blur()`, `dragTo()`, Mouse Actions, Keyboard Actions, Event Flow, Actionability, and Enterprise Interaction Strategy

---

# Objective

In the previous lesson, you learned:

```text
Locator
   ↓
Strictness
   ↓
Actionability
   ↓
Action
```

Now we focus on the actual actions.

Playwright provides high-level APIs that model user interactions:

```text
click()
dblclick()
fill()
press()
check()
uncheck()
selectOption()
hover()
focus()
blur()
dragTo()
```

It also provides lower-level browser interaction APIs through:

```text
mouse
keyboard
```

The important question is not:

> "Which method can I use?"

The better question is:

> "Which interaction most accurately represents the user's intended behavior?"

You will learn:

- `click()`
- `dblclick()`
- `fill()`
- `type()`
- `press()`
- `check()`
- `uncheck()`
- `selectOption()`
- `hover()`
- `focus()`
- `blur()`
- `dragTo()`
- Mouse actions
- Keyboard actions
- Modifier keys
- Event flow
- Actionability
- Input behavior
- Checkbox behavior
- Radio buttons
- Select elements
- Drag and drop
- Keyboard navigation
- Special keys
- When to use low-level APIs
- Why `evaluate()` should not replace user actions
- Enterprise interaction strategy

---

# Before We Start

Think about a real person using a website.

A person can:

```text
Click a button
Double-click an item
Type into a field
Press Enter
Check a checkbox
Select a dropdown option
Hover over a menu
Drag an item
Press Tab
Press Escape
```

Playwright provides APIs that model these interactions.

This gives us:

```text
Business Intent
      ↓
User Interaction
      ↓
Playwright Action
      ↓
Browser Event
      ↓
Application Behavior
```

---

# User Action Architecture

```text
                    Test Intent
                        │
                        ▼
                  Playwright Locator
                        │
                        ▼
                  User Action API
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
       Mouse         Keyboard         Form
         │              │              │
         ▼              ▼              ▼
      click()         press()        fill()
      hover()         type()         check()
      dragTo()        keyDown()      selectOption()
         │              │              │
         └──────────────┼──────────────┘
                        ▼
                   Browser Events
                        │
                        ▼
                 Application Logic
```

---

# 1. `click()`

The most common Playwright action is:

```text
click()
```

It represents a user clicking an element.

Example:

```ts
await page.getByRole('button', { name: 'Login' }).click();
```

Conceptually:

```text
Locate
  ↓
Check Actionability
  ↓
Perform Click
  ↓
Browser Event
  ↓
Application Handler
```

---

# Why `click()` Is More Than "Click"

A click is not merely:

```text
Mouse button down
+
Mouse button up
```

A browser interaction can trigger:

```text
pointer events
mouse events
focus
click event
application handlers
navigation
state updates
```

The application may respond with:

```text
API request
DOM update
navigation
modal
toast
validation
```

---

# Typical Click Workflow

```text
Locator
 ↓
Resolve
 ↓
Strictness
 ↓
Actionability
 ↓
Mouse Interaction
 ↓
Browser Event
 ↓
Application Handler
 ↓
UI State Change
```

---

# Example

```ts
await page.getByRole('button', { name: 'Submit Order' }).click();
```

This is generally preferable to:

```ts
await page.locator('button').click();
```

because the first expresses:

```text
Business Intent
```

more clearly.

---

# Click Options

Playwright supports options for certain interaction scenarios.

Examples include:

```ts
await locator.click({
  button: 'right'
});
```

or:

```ts
await locator.click({
  modifiers: ['Shift']
});
```

or:

```ts
await locator.click({
  position: { x: 20, y: 10 }
});
```

Use options only when they represent real interaction requirements.

---

# Left Click

The default click is normally:

```text
left mouse button
```

Example:

```ts
await button.click();
```

---

# Right Click

Context-menu interaction:

```ts
await button.click({ button: 'right' });
```

This can trigger:

```text
contextmenu
```

behavior.

---

# Middle Click

For specialized applications:

```ts
await locator.click({ button: 'middle' });
```

Use this only when middle-click behavior is genuinely part of the application.

---

# Modifier Clicks

Examples:

```text
Ctrl + Click
Shift + Click
Alt + Click
Meta + Click
```

Playwright supports modifier keys.

Example:

```ts
await locator.click({
  modifiers: ['Control']
});
```

On macOS, the appropriate modifier may be:

```text
Meta
```

depending on the desired behavior.

---

# 2. `dblclick()`

Double-clicking is represented by:

```ts
await locator.dblclick();
```

Typical use cases:

```text
Open file
Edit table cell
Rename item
Open desktop-style component
```

---

# Example

```ts
await page.getByText('Invoice.pdf').dblclick();
```

The application may respond to:

```text
dblclick
```

with:

```text
Open editor
```

---

# Don't Replace Double Click with Two Clicks

Avoid manually doing:

```ts
await locator.click();
await locator.click();
```

when the application explicitly requires:

```text
double-click
```

Use:

```ts
dblclick()
```

because the browser interaction semantics are different.

---

# 3. `fill()`

For text-entry controls:

```ts
fill()
```

is one of the most useful APIs.

Example:

```ts
await page.getByLabel('Email').fill('user@example.com');
```

---

# What Does `fill()` Represent?

Conceptually:

```text
Clear Existing Value
        ↓
Set Desired Value
        ↓
Dispatch Appropriate Input Behavior
```

It is designed for efficiently setting form values.

---

# Why `fill()` Is Usually Preferred

Suppose the field contains:

```text
old@example.com
```

You want:

```text
new@example.com
```

With:

```ts
fill('new@example.com')
```

you do not need:

```text
Ctrl+A
Backspace
Type
```

---

# `fill()` vs `type()`

This distinction is important.

```text
fill()
```

is generally best for:

```text
Set this field to this value.
```

while:

```text
type()
```

is useful when:

```text
Simulate character-by-character typing behavior.
```

---

# 4. `type()`

Playwright also provides:

```text
type()
```

Historically, this was commonly used for typing text character by character.

Example:

```ts
await page.getByLabel('Search').type('Playwright');
```

---

# `type()` Mental Model

Think:

```text
P
l
a
y
w
r
i
g
h
t
```

as a sequence of keyboard input.

This can matter when testing:

```text
Key-driven UI
Input masks
Keyboard-triggered behavior
Per-character application logic
```

---

# `fill()` vs `type()`

| Scenario | Preferred |
|---|---|
| Set a form value | `fill()` |
| Replace existing value | `fill()` |
| Fast form population | `fill()` |
| Simulate character-by-character behavior | `type()` |
| Test key/input events individually | `type()` |
| Test keyboard shortcuts | `press()` / keyboard APIs |

---

# Important Modern Practice

For ordinary form entry:

```text
Prefer fill()
```

Use lower-level typing behavior only when the test genuinely needs to verify typing semantics.

---

# 5. `press()`

`press()` is used to send a keyboard key or key combination to an element.

Example:

```ts
await page.getByLabel('Search').press('Enter');
```

---

# Common Keys

Examples:

```text
Enter
Tab
Escape
ArrowDown
ArrowUp
ArrowLeft
ArrowRight
Backspace
Delete
Home
End
PageUp
PageDown
```

---

# Keyboard Combinations

You can represent combinations such as:

```text
Control+A
Control+C
Control+V
Shift+Tab
Alt+ArrowDown
```

Example:

```ts
await input.press('Control+A');
```

---

# `press()` vs `fill()`

Use:

```text
fill()
```

when you want to set a value.

Use:

```text
press()
```

when you want to simulate a specific key action.

Example:

```ts
await search.fill('Playwright');
await search.press('Enter');
```

This represents:

```text
Enter search text
+
Submit via keyboard
```

---

# Keyboard-Driven UI

Consider:

```text
Search Box
```

The application may only submit when:

```text
Enter
```

is pressed.

Then:

```ts
await search.fill('ORD-1001');
await search.press('Enter');
```

is more realistic than trying to directly invoke a JavaScript handler.

---

# 6. `check()`

Checkboxes should generally be interacted with using:

```ts
check()
```

Example:

```ts
await page.getByRole('checkbox', { name: 'Remember me' }).check();
```

---

# Why Use `check()`?

A checkbox has a meaningful state:

```text
Checked
Unchecked
```

The action should respect that semantic state.

---

# `check()` Behavior

Conceptually:

```text
Checkbox
   ↓
Already checked?
   │
   ├── YES → Remain checked
   │
   └── NO → Check it
```

This makes the action state-aware.

---

# `uncheck()`

To ensure a checkbox is unchecked:

```ts
await checkbox.uncheck();
```

Conceptually:

```text
Checked
   ↓
Uncheck
   ↓
Unchecked
```

---

# `check()` vs `click()`

You could technically click a checkbox.

But:

```text
click()
```

means:

```text
Perform click
```

while:

```text
check()
```

means:

```text
Ensure checkbox is checked
```

This expresses stronger intent.

---

# Example

Weak:

```ts
await checkbox.click();
```

This can toggle:

```text
checked → unchecked
```

or:

```text
unchecked → checked
```

depending on current state.

Stronger:

```ts
await checkbox.check();
```

because the desired final state is explicit.

---

# 7. Radio Buttons

Radio buttons represent a mutually exclusive choice.

Example:

```text
○ Credit Card
○ UPI
○ Net Banking
```

A radio button can be selected using:

```ts
await page.getByRole('radio', { name: 'UPI' }).check();
```

---

# Radio Button Mental Model

```text
Payment Method
      │
 ┌────┼────┐
 ▼    ▼    ▼
Card  UPI  Net Banking
```

Selecting:

```text
UPI
```

should generally deselect the previous option in the same group.

---

# `check()` Is Also Useful for Radios

The semantic meaning remains:

```text
Ensure selected
```

rather than:

```text
Just click.
```

---

# 8. `selectOption()`

For native HTML `<select>` elements, Playwright provides:

```text
selectOption()
```

Example:

```ts
await page.getByLabel('Country').selectOption('IN');
```

---

# Native Select

HTML:

```html
<select>
  <option value="IN">India</option>
  <option value="US">United States</option>
  <option value="UK">United Kingdom</option>
</select>
```

Playwright:

```ts
await page.getByLabel('Country').selectOption('IN');
```

---

# Selection by Value

```ts
selectOption('IN')
```

selects:

```text
value="IN"
```

---

# Selection by Label

Depending on the API form, you can select based on option label.

Conceptually:

```text
India
```

rather than:

```text
IN
```

---

# Selection by Multiple Properties

For more complex cases, Playwright supports option descriptors.

Conceptually:

```ts
await select.selectOption({
  label: 'India'
});
```

or:

```ts
await select.selectOption({
  value: 'IN'
});
```

---

# Native Select vs Custom Dropdown

This distinction is critical.

A native:

```html
<select>
```

should generally use:

```text
selectOption()
```

A custom React/Angular/Vue dropdown may actually be:

```text
div
button
listbox
role=option
```

In that case:

```text
selectOption()
```

may not apply.

---

# Custom Dropdown Architecture

Example:

```text
<button>
    Select Country
</button>

<div role="listbox">
    <div role="option">India</div>
    <div role="option">USA</div>
</div>
```

The interaction may be:

```text
Click dropdown
   ↓
Find option
   ↓
Click option
```

rather than:

```text
selectOption()
```

---

# Native vs Custom Dropdown

| Dropdown | Strategy |
|---|---|
| Native `<select>` | `selectOption()` |
| Custom `listbox` | Role-based interaction |
| Custom menu | Click + option |
| Autocomplete | Fill + select suggestion |
| Searchable dropdown | Fill + option |

---

# 9. `hover()`

Hovering is represented by:

```ts
await locator.hover();
```

Example:

```ts
await page.getByText('Products').hover();
```

---

# Hover Use Cases

Hover may reveal:

```text
Dropdown
Tooltip
Action buttons
Submenu
Additional information
```

---

# Example

```text
Products
   ↓ hover
Electronics
Laptops
Phones
```

Automation:

```ts
await page.getByText('Products').hover();
await page.getByText('Laptops').click();
```

---

# Hover and CSS

Hover can trigger:

```css
:hover
```

and application event handlers.

Playwright's `hover()` models pointer movement toward the target.

---

# 10. `focus()`

Focus places keyboard focus on an element.

Example:

```ts
await page.getByLabel('Email').focus();
```

---

# Why Focus Matters

Focus affects:

```text
Keyboard input
Validation
Focus styles
Keyboard shortcuts
Accessibility
```

---

# Example

```text
Input
 ↓
Focus
 ↓
User types
```

You may need explicit focus when testing keyboard navigation or focus-specific behavior.

---

# 11. `blur()`

Blur removes focus from an element.

Example:

```ts
await page.getByLabel('Email').blur();
```

This can trigger:

```text
blur event
validation
formatting
state updates
```

---

# Form Validation Example

Some applications validate on:

```text
blur
```

Workflow:

```text
Fill Email
   ↓
Blur
   ↓
Validation
   ↓
Error Message
```

This is a valid use case for `blur()`.

---

# Focus vs Blur

```text
focus()
   ↓
Element receives focus

blur()
   ↓
Element loses focus
```

---

# 12. `dragTo()`

Playwright supports drag-and-drop between locators.

Conceptually:

```ts
await source.dragTo(target);
```

Example:

```ts
await page.getByTestId('task-1')
  .dragTo(page.getByTestId('done-column'));
```

---

# Drag-and-Drop Architecture

```text
Source
  │
  ▼
Mouse Down
  │
  ▼
Move
  │
  ▼
Target
  │
  ▼
Mouse Up
```

---

# Drag-and-Drop Use Cases

Examples:

```text
Kanban board
File manager
Sorting lists
Dashboard widgets
Upload areas
Workflow designers
```

---

# Drag-and-Drop Challenges

Custom drag-and-drop implementations can behave differently.

Some applications use:

```text
HTML5 drag/drop
Pointer events
Mouse events
Libraries
Custom gesture systems
```

Therefore a simple `dragTo()` may not always match the application's internal interaction model.

---

# 13. Mouse API

Playwright also exposes:

```text
page.mouse
```

for lower-level pointer interaction.

Examples conceptually include:

```text
move()
down()
up()
click()
dblclick()
wheel()
```

---

# High-Level vs Low-Level Interaction

### High-Level

```text
locator.click()
locator.fill()
locator.hover()
locator.dragTo()
```

### Low-Level

```text
page.mouse.move()
page.mouse.down()
page.mouse.up()
```

---

# Which Should You Prefer?

Generally:

```text
High-Level User Action
```

first.

Use:

```text
Low-Level Mouse API
```

when the application requires precise pointer behavior that high-level locators cannot express.

---

# Mouse Example

A canvas-based application may require:

```text
Move to coordinate
Mouse down
Move
Mouse up
```

Then:

```ts
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.move(300, 200);
await page.mouse.up();
```

This is a specialized interaction.

---

# 14. Keyboard API

Playwright also exposes:

```text
page.keyboard
```

for lower-level keyboard control.

Conceptually:

```text
keyboard.down()
keyboard.up()
keyboard.press()
insertText()
```

---

# Keyboard Use Cases

Examples:

```text
Keyboard shortcuts
Rich text editors
Canvas applications
Complex accessibility navigation
Special key sequences
```

---

# `locator.press()` vs `page.keyboard`

Prefer:

```ts
locator.press('Enter');
```

when the key should be sent to a specific focused element.

Use:

```text
page.keyboard
```

when you intentionally need page-level keyboard control.

---

# Keyboard Navigation

A realistic accessibility test may involve:

```text
Tab
Tab
Enter
Escape
ArrowDown
ArrowUp
```

Example:

```ts
await page.keyboard.press('Tab');
await page.keyboard.press('Enter');
```

However, prefer targeting the intended element directly when the test is not specifically testing keyboard navigation.

---

# Modifier Keys

Common modifiers:

```text
Alt
Control
Meta
Shift
```

Examples:

```text
Control+A
Control+C
Control+V
Shift+Tab
```

---

# Keyboard Shortcut Testing

Suppose:

```text
Ctrl+S
```

saves a document.

Then:

```ts
await page.keyboard.press('Control+S');
```

may be appropriate.

The test is specifically validating:

```text
Keyboard Shortcut
```

rather than simply:

```text
Save button
```

---

# Event Flow

User actions can trigger browser events.

For a click, conceptually:

```text
Pointer Interaction
      ↓
Mouse Events
      ↓
Focus Behavior
      ↓
Click Event
      ↓
Application Handler
      ↓
State Update
```

The exact event sequence can vary by element and browser behavior, but the important concept is:

```text
User action
```

can produce a chain of browser and application events.

---

# Why Event Flow Matters

Some applications listen to:

```text
input
change
keydown
keyup
focus
blur
click
pointerdown
pointerup
```

The test should interact at the correct abstraction level.

---

# Example: Search Input

```text
Fill
 ↓
Input event
 ↓
Search logic
```

Then:

```text
Press Enter
 ↓
Keyboard event
 ↓
Submit
```

This is different from directly modifying:

```text
DOM value
```

using JavaScript.

---

# Why `evaluate()` Is Not a Replacement

You might be tempted to do:

```ts
await page.locator('input').evaluate(
  (el) => (el as HTMLInputElement).value = 'test'
);
```

This changes the DOM property.

But it may not reproduce the complete user interaction and framework event flow required by the application.

Therefore:

```text
Use user-facing Playwright actions
```

whenever possible.

---

# Framework Event Handling

Modern frontend frameworks often rely on event systems.

For example:

```text
User Input
   ↓
Browser Event
   ↓
Framework Handler
   ↓
State Update
   ↓
Render
```

Direct DOM manipulation can bypass parts of that flow.

---

# User Action vs DOM Manipulation

| Strategy | Represents User? | Recommended? |
|---|---:|---|
| `click()` | Yes | Usually |
| `fill()` | Yes | Usually |
| `check()` | Yes | Usually |
| `selectOption()` | Yes | Usually |
| `hover()` | Yes | Usually |
| `page.mouse` | Low-level user interaction | Specialized |
| `page.keyboard` | Keyboard interaction | Specialized |
| `evaluate()` DOM mutation | Not necessarily | Avoid as default |

---

# Actionability Still Applies

Remember:

```text
click()
fill()
check()
hover()
```

are not isolated commands.

They interact with the actionability system.

Conceptually:

```text
Locator
 ↓
Strictness
 ↓
Actionability
 ↓
User Action
 ↓
Application
```

---

# Click Example with Full Mental Model

```ts
await page
  .getByRole('button', { name: 'Save' })
  .click();
```

Think:

```text
Find Save button
       ↓
Exactly one?
       ↓
Visible?
       ↓
Stable?
       ↓
Receives events?
       ↓
Enabled?
       ↓
Click
       ↓
Application reacts
```

---

# Fill Example with Full Mental Model

```ts
await page
  .getByLabel('Email')
  .fill('user@example.com');
```

Think:

```text
Find Email field
       ↓
Exactly one?
       ↓
Visible?
       ↓
Stable?
       ↓
Receives events?
       ↓
Enabled?
       ↓
Editable?
       ↓
Fill value
       ↓
Application reacts
```

---

# Check Example

```ts
await page
  .getByRole('checkbox', { name: 'Accept terms' })
  .check();
```

Think:

```text
Find checkbox
       ↓
Exactly one?
       ↓
Actionable?
       ↓
Enabled?
       ↓
Ensure checked
```

---

# Select Example

```ts
await page
  .getByLabel('Country')
  .selectOption('IN');
```

Think:

```text
Find select
       ↓
Verify target
       ↓
Select option
       ↓
Dispatch selection behavior
       ↓
Application reacts
```

---

# Hover Example

```ts
await page
  .getByRole('button', { name: 'Products' })
  .hover();
```

Think:

```text
Find Products
       ↓
Actionability
       ↓
Move pointer
       ↓
Hover state
       ↓
Menu appears
```

---

# Keyboard Example

```ts
await page
  .getByLabel('Search')
  .press('Enter');
```

Think:

```text
Find Search
       ↓
Ensure target can receive keyboard interaction
       ↓
Send Enter
       ↓
Application handles keyboard event
```

---

# Choosing the Right Action

Use this mental model:

```text
What is the user doing?
          │
 ┌────────┼────────┬─────────┐
 ▼        ▼        ▼         ▼
Click    Type    Select     Hover
 │        │        │         │
 ▼        ▼        ▼         ▼
click() fill() selectOption hover()
```

For keyboard behavior:

```text
press()
keyboard
```

For specialized pointer interaction:

```text
mouse
dragTo()
```

---

# Action Selection Matrix

| User Intent | Playwright API |
|---|---|
| Click button | `click()` |
| Double-click | `dblclick()` |
| Set text | `fill()` |
| Simulate typing | `type()` |
| Press key | `press()` |
| Check checkbox | `check()` |
| Uncheck checkbox | `uncheck()` |
| Select radio | `check()` |
| Native select | `selectOption()` |
| Hover | `hover()` |
| Focus | `focus()` |
| Blur | `blur()` |
| Drag component | `dragTo()` |
| Low-level mouse | `page.mouse` |
| Low-level keyboard | `page.keyboard` |

---

# Enterprise Interaction Hierarchy

A practical strategy:

```text
1. Semantic Locator + High-Level Action
2. Component Locator + High-Level Action
3. Specialized High-Level Action
4. Mouse / Keyboard API
5. DOM Evaluation
```

The lower you go, the stronger the justification should be.

---

# Why High-Level APIs Are Preferred

High-level actions:

```text
Readable
Maintainable
User-oriented
Actionability-aware
```

Example:

```ts
await submitButton.click();
```

is much easier to understand than:

```ts
await page.mouse.click(x, y);
```

---

# Coordinate-Based Interaction

Avoid:

```ts
await page.mouse.click(482, 311);
```

unless the application is genuinely coordinate-driven.

Why?

Because coordinates depend on:

```text
Viewport
Resolution
Responsive Layout
Zoom
Position
Scrolling
```

---

# Coordinate Fragility

Consider:

```text
Desktop:
Submit at (482, 311)

Mobile:
Submit at (100, 650)
```

A coordinate-based test breaks.

A semantic locator can survive the layout change.

---

# Canvas Applications

Coordinates may be appropriate for:

```text
Drawing Applications
Maps
Games
Charts
CAD Tools
Canvas Editors
```

These are specialized cases.

---

# Enterprise Rule

> **Use the highest-level Playwright action that accurately represents the intended user behavior.**

---

# Common Beginner Mistakes

## Mistake 1 — Using `click()` for Everything

Not every interaction is a click.

---

## Mistake 2 — Using `click()` Instead of `check()`

For stateful checkboxes:

```text
check()
```

better expresses:

```text
Ensure checked.
```

---

## Mistake 3 — Using `click()` for Native Selects

Use:

```text
selectOption()
```

for native `<select>` elements.

---

## Mistake 4 — Using `fill()` When Testing Character-by-Character Behavior

If the requirement is specifically about typing behavior, consider:

```text
type()
```

or keyboard APIs.

---

## Mistake 5 — Using `type()` for Every Form Field

For ordinary data entry:

```text
fill()
```

is generally simpler and more efficient.

---

## Mistake 6 — Using Coordinates Everywhere

This creates fragile tests.

---

## Mistake 7 — Using `evaluate()` Instead of User Actions

Direct DOM mutation can bypass important application event behavior.

---

## Mistake 8 — Using `force` for Normal Interaction

Force should not be the default.

---

## Mistake 9 — Using Low-Level Mouse APIs When a Locator Action Exists

Prefer:

```text
locator.click()
```

over:

```text
page.mouse.click()
```

when both correctly represent the requirement.

---

## Mistake 10 — Confusing Native and Custom Dropdowns

```text
selectOption()
```

is intended for native select controls.

---

# Professional Tips

Before choosing an action, ask:

```text
What would the user actually do?
```

Then map it:

```text
User clicks
     ↓
click()

User enters a value
     ↓
fill()

User types character-by-character
     ↓
type()

User presses Enter
     ↓
press()

User checks a checkbox
     ↓
check()

User selects native option
     ↓
selectOption()

User hovers
     ↓
hover()

User drags
     ↓
dragTo()
```

This creates highly readable tests.

---

# Real Interview Discussion

## Junior-Level

### Q1. What is the difference between `click()` and `dblclick()`?

**Answer:**

`click()` performs a single click, while `dblclick()` represents a double-click interaction.

---

### Q2. What is the difference between `fill()` and `type()`?

**Answer:**

`fill()` is generally used to set an input's value efficiently, while `type()` is useful when character-by-character keyboard input behavior matters.

---

### Q3. How do you select a checkbox?

**Answer:**

Use:

```ts
await checkbox.check();
```

and use:

```ts
await checkbox.uncheck();
```

when the desired state is unchecked.

---

# Mid-Level

### Q4. Why is `check()` preferable to `click()` for checkboxes?

**Answer:**

Because `check()` expresses the intended state rather than simply toggling the control. It makes the test more deterministic.

---

### Q5. When do you use `selectOption()`?

**Answer:**

For native HTML `<select>` elements.

---

### Q6. What is the difference between native and custom dropdowns?

**Answer:**

A native dropdown is represented by an HTML `<select>` element and can use `selectOption()`. A custom dropdown is usually built from other elements and should be interacted with according to its semantic roles and actual user behavior.

---

# Senior-Level

### Q7. When would you use `page.mouse` instead of `locator.click()`?

**Answer:**

Only when low-level pointer control is genuinely required, such as canvas-based applications, coordinate-driven interactions, or specialized drag/gesture behavior that cannot be accurately expressed through a locator action.

---

### Q8. Why should direct DOM manipulation through `evaluate()` not be the default?

**Answer:**

Because directly changing DOM properties may bypass the browser event sequence and frontend framework state management that real user interaction would trigger.

---

### Q9. How do actionability and user actions work together?

**Answer:**

The locator identifies the target, Playwright applies the relevant actionability checks, waits when necessary, and then performs the requested action once the target is ready.

---

# Lead-Level

### Q10. How would you establish an enterprise action strategy?

**Answer:**

I would prioritize semantic locators with high-level user actions, use component abstractions for repeated controls, reserve mouse and keyboard APIs for specialized interactions, and treat direct DOM evaluation and force actions as exceptional techniques requiring justification.

---

### Q11. How would you test a complex custom drag-and-drop component?

**Answer:**

I would first determine whether the component exposes a meaningful semantic or high-level drag contract. If `dragTo()` accurately reproduces the required behavior, I would use it. If the application relies on specialized pointer events, I would use the mouse API and encapsulate the interaction inside a reusable component abstraction.

---

# Architect-Level

### Q12. What is the ideal abstraction level for Playwright interactions?

**Strong Answer:**

The ideal level is the highest abstraction that accurately represents the user's intended behavior. Semantic locator actions such as `click()`, `fill()`, `check()`, and `selectOption()` should be preferred because they are readable, actionability-aware, and less coupled to implementation details. Lower-level mouse, keyboard, or DOM APIs should be introduced only when the application's interaction model requires them.

---

# Knowledge Check

1. What does `click()` do?
2. What is `dblclick()`?
3. Why is `fill()` generally preferred for ordinary form entry?
4. When is `type()` useful?
5. What does `press()` do?
6. How do you press Enter?
7. What does `check()` do?
8. What does `uncheck()` do?
9. Why is `check()` often better than clicking a checkbox?
10. How do you select a radio button?
11. What is `selectOption()`?
12. When should you not use `selectOption()`?
13. What is a custom dropdown?
14. What does `hover()` do?
15. What does `focus()` do?
16. What does `blur()` do?
17. Why can blur trigger validation?
18. What is `dragTo()`?
19. When would you use `page.mouse`?
20. When would you use `page.keyboard`?
21. What is the difference between `locator.press()` and `page.keyboard`?
22. Why are coordinate-based interactions fragile?
23. When can coordinate-based interaction be justified?
24. Why should `evaluate()` not replace normal user actions?
25. How do user actions interact with actionability?
26. What is the difference between high-level and low-level interaction?
27. What is the enterprise interaction hierarchy?
28. Why should force actions be exceptional?
29. How would you test keyboard shortcuts?
30. What is the highest-level interaction that accurately represents user intent?

---

# Step Summary

The key principle of this lesson is:

> **Choose the Playwright action that most accurately represents the user's actual interaction.**

Your mental model should now be:

```text
User Intent
     ↓
Semantic Locator
     ↓
Actionability
     ↓
Correct User Action
     ↓
Browser Event
     ↓
Application Behavior
     ↓
Business Assertion
```

Examples:

```text
Click
 ↓
click()

Set field value
 ↓
fill()

Simulate typing
 ↓
type()

Press key
 ↓
press()

Ensure checkbox selected
 ↓
check()

Ensure checkbox unselected
 ↓
uncheck()

Native dropdown
 ↓
selectOption()

Hover
 ↓
hover()

Focus
 ↓
focus()

Blur
 ↓
blur()

Drag
 ↓
dragTo()
```

For specialized interactions:

```text
page.mouse
page.keyboard
```

can provide lower-level control.

But the enterprise principle remains:

```text
High-Level
   ↓
First Choice

Low-Level
   ↓
Specialized Requirement

DOM Evaluation
   ↓
Exceptional
```

---

# Progress Milestone

✅ **Step 67 completed.**

You now understand:

- `click()`
- `dblclick()`
- `fill()`
- `type()`
- `press()`
- `check()`
- `uncheck()`
- Radio buttons
- `selectOption()`
- Native dropdowns
- Custom dropdowns
- `hover()`
- `focus()`
- `blur()`
- `dragTo()`
- `page.mouse`
- `page.keyboard`
- Modifier keys
- Keyboard shortcuts
- Event flow
- Actionability
- High-level actions
- Low-level actions
- Coordinate interactions
- `evaluate()` limitations
- Enterprise interaction strategy

Your complete interaction architecture is now:

```text
                    USER INTENT
                         │
                         ▼
                    LOCATOR
                         │
                         ▼
                   STRICTNESS
                         │
                         ▼
                  ACTIONABILITY
                         │
                         ▼
                  USER ACTION
                         │
        ┌────────────────┼─────────────────┐
        ▼                ▼                 ▼
      Mouse           Keyboard            Form
        │                │                 │
     click()          press()            fill()
     hover()          keyboard           check()
     dragTo()                             selectOption()
        │
        └────────────────┼─────────────────┘
                         ▼
                  BROWSER EVENTS
                         │
                         ▼
                  APPLICATION STATE
                         │
                         ▼
                  BUSINESS ASSERTION
```

You are now ready for the next major topic: **mouse and keyboard interaction in depth**, including coordinates, buttons, modifiers, scrolling, wheel events, keyboard combinations, key sequences, and realistic user workflows.

---

# Next Step

# Step 68 — Mastering Playwright Mouse & Keyboard Actions: `page.mouse`, `page.keyboard`, Coordinates, `move()`, `down()`, `up()`, `click()`, `dblclick()`, `wheel()`, Keyboard Modifiers, `press()`, `down()`, `up()`, `insertText()`, Shortcuts, Dragging, Scrolling, Canvas, Rich Text Editors, Accessibility Navigation, and Enterprise Low-Level Interaction Strategy

The next lesson will answer:

> **When should you move below locator-level actions and control the mouse or keyboard directly?**

We will cover:

```text
Mouse
 ↓
move()
down()
up()
click()
dblclick()
wheel()

Keyboard
 ↓
press()
down()
up()
insertText()

Modifiers
 ↓
Control
Shift
Alt
Meta

Specialized UI
 ↓
Canvas
Drag/Drop
Rich Text
Keyboard Navigation
Complex Editors
```

and, most importantly:

> **When low-level interaction is justified—and when it is a sign that the locator strategy needs improvement.**
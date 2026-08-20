# Playwright with TypeScript Mastery Bootcamp

# Stage 30 — Actions, Actionability, and Synchronization

# Step 68 — Mastering Playwright Mouse & Keyboard Actions: `page.mouse`, `page.keyboard`, Coordinates, `move()`, `down()`, `up()`, `click()`, `dblclick()`, `wheel()`, Keyboard Modifiers, `press()`, `down()`, `up()`, `insertText()`, Shortcuts, Dragging, Scrolling, Canvas, Rich Text Editors, Accessibility Navigation, and Enterprise Low-Level Interaction Strategy

---

# Objective

In the previous lesson, you learned high-level Playwright actions:

```text
click()
fill()
check()
selectOption()
hover()
press()
dragTo()
```

Now we go one level deeper.

Playwright also provides direct control over:

```text
Mouse
Keyboard
```

through:

```ts
page.mouse
page.keyboard
```

These APIs are powerful, but they should be used intentionally.

The goal is not:

> "Use low-level APIs everywhere."

The goal is:

> "Use low-level APIs when the application's interaction model genuinely requires low-level control."

You will learn:

- `page.mouse`
- `mouse.move()`
- `mouse.down()`
- `mouse.up()`
- `mouse.click()`
- `mouse.dblclick()`
- `mouse.wheel()`
- Mouse coordinates
- Mouse buttons
- Modifier keys
- `page.keyboard`
- `keyboard.press()`
- `keyboard.down()`
- `keyboard.up()`
- `keyboard.insertText()`
- Keyboard shortcuts
- Key combinations
- Dragging
- Scrolling
- Canvas applications
- Rich text editors
- Accessibility keyboard navigation
- Coordinate-based automation
- High-level vs low-level interaction
- Low-level actionability considerations
- Enterprise governance

---

# 1. High-Level vs Low-Level Interaction

Before learning the APIs, understand the difference.

## High-Level

```ts
await locator.click();
```

You are saying:

> Click this logical UI element.

## Low-Level

```ts
await page.mouse.click(500, 300);
```

You are saying:

> Perform a mouse click at this physical page coordinate.

These are fundamentally different abstractions.

---

# Interaction Abstraction Pyramid

```text
                 Business Action
                       │
                       ▼
              Semantic Locator
                       │
                       ▼
              High-Level Action
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       click()       fill()      check()
          │
          ▼
       Low-Level APIs
          │
     ┌────┴─────┐
     ▼          ▼
   Mouse      Keyboard
     │          │
     ▼          ▼
 Coordinates  Key Events
```

The higher the abstraction, the more maintainable the interaction usually is.

---

# 2. `page.mouse`

Playwright exposes:

```ts
page.mouse
```

for lower-level mouse interaction.

It provides operations such as:

```text
move()
down()
up()
click()
dblclick()
wheel()
```

---

# 3. `mouse.move()`

`mouse.move()` moves the mouse pointer to a coordinate.

Example:

```ts
await page.mouse.move(500, 300);
```

Conceptually:

```text
Current Position
       ↓
Mouse Movement
       ↓
(500, 300)
```

---

# Coordinates

Coordinates are generally expressed relative to the page viewport.

For example:

```text
(0, 0)
```

represents the top-left reference point of the page viewport.

Conceptually:

```text
(0,0)
 ┌──────────────────────────────┐
 │                              │
 │             (500,300)        │
 │                  ●           │
 │                              │
 └──────────────────────────────┘
```

---

# Coordinate-Based Automation

Example:

```ts
await page.mouse.click(500, 300);
```

This is very different from:

```ts
await page.getByRole('button', { name: 'Save' }).click();
```

The first depends on:

```text
Position
```

The second depends on:

```text
Meaning
```

---

# Why Coordinates Are Fragile

Coordinates can change because of:

```text
Window Size
Viewport
Responsive Layout
Browser Zoom
Page Scroll
Header Size
Font Rendering
Dynamic Content
Advertisements
Popups
```

Therefore coordinate-based automation should be exceptional.

---

# 4. `mouse.down()`

`mouse.down()` presses a mouse button.

Example:

```ts
await page.mouse.down();
```

Conceptually:

```text
Mouse
 ↓
Button pressed
 ↓
Button remains down
```

This is useful when constructing a manual gesture.

---

# 5. `mouse.up()`

`mouse.up()` releases the mouse button.

Example:

```ts
await page.mouse.up();
```

Combined:

```ts
await page.mouse.down();
await page.mouse.up();
```

This represents:

```text
Mouse Down
    ↓
Mouse Up
```

---

# Why `down()` and `up()` Matter

Some interactions are not simply:

```text
Click
```

They are gestures:

```text
Mouse Down
   ↓
Move
   ↓
Move
   ↓
Mouse Up
```

Examples:

```text
Drawing
Dragging
Resizing
Selecting
Canvas interaction
```

---

# 6. `mouse.click()`

You can directly click a coordinate:

```ts
await page.mouse.click(500, 300);
```

You can also specify a mouse button when needed.

Conceptually:

```text
left
right
middle
```

---

# High-Level Click vs Mouse Click

### High-Level

```ts
await locator.click();
```

Represents:

```text
Click this element.
```

### Low-Level

```ts
await page.mouse.click(x, y);
```

Represents:

```text
Click this location.
```

This distinction is critical.

---

# 7. `mouse.dblclick()`

You can perform a double-click at a coordinate.

```ts
await page.mouse.dblclick(500, 300);
```

Use this only when coordinate-based interaction is genuinely required.

If you have a locator:

```ts
await locator.dblclick();
```

is usually preferable.

---

# 8. `mouse.wheel()`

`mouse.wheel()` simulates wheel scrolling.

Example:

```ts
await page.mouse.wheel(0, 500);
```

Conceptually:

```text
Horizontal
    ↑
    0

Vertical
    ↓
   500
```

---

# Wheel Interaction

Wheel events can be useful for:

```text
Infinite scroll
Canvas
Map controls
Custom scroll containers
Virtualized components
```

---

# Wheel vs `scrollIntoViewIfNeeded()`

If your goal is simply:

```text
Make this element visible
```

prefer:

```text
locator-based scrolling
```

rather than:

```text
mouse.wheel()
```

For example:

```ts
await locator.scrollIntoViewIfNeeded();
```

This expresses the actual intent.

---

# Wheel for User Behavior

Use:

```ts
page.mouse.wheel()
```

when the test specifically needs to validate:

```text
User scroll behavior
```

rather than merely:

```text
Element visibility
```

---

# 9. Mouse Buttons

Mouse operations can distinguish:

```text
left
right
middle
```

Example:

```ts
await page.mouse.click(500, 300, {
  button: 'right'
});
```

This can trigger a context menu.

---

# 10. Modifier Keys

Mouse actions can be combined with:

```text
Shift
Control
Alt
Meta
```

Example conceptually:

```ts
await locator.click({
  modifiers: ['Control']
});
```

This is useful for:

```text
Multi-select
Opening links in new tabs
Keyboard-assisted selection
Desktop-like UI behavior
```

---

# 11. `page.keyboard`

Playwright also exposes:

```ts
page.keyboard
```

for low-level keyboard interaction.

Important APIs include:

```text
press()
down()
up()
insertText()
```

---

# 12. `keyboard.press()`

`press()` sends a key or key combination.

Example:

```ts
await page.keyboard.press('Enter');
```

---

# Keyboard Examples

```ts
await page.keyboard.press('Tab');

await page.keyboard.press('Escape');

await page.keyboard.press('ArrowDown');

await page.keyboard.press('Control+A');
```

---

# `press()` vs `down()` + `up()`

These are different levels.

### `press()`

Represents:

```text
Key Down
+
Key Up
```

as a combined key action.

### `down()`

Presses and holds a key.

### `up()`

Releases a key.

---

# 13. `keyboard.down()`

Example:

```ts
await page.keyboard.down('Shift');
```

Now:

```text
Shift
```

is held.

You can perform another action while it remains pressed.

---

# 14. `keyboard.up()`

Release the key:

```ts
await page.keyboard.up('Shift');
```

---

# Keyboard Combination

For a complex interaction:

```ts
await page.keyboard.down('Control');
await page.keyboard.press('a');
await page.keyboard.up('Control');
```

Conceptually:

```text
Hold Control
    ↓
Press A
    ↓
Release Control
```

This is equivalent in intent to:

```text
Ctrl + A
```

---

# Prefer `press()` When Possible

For a simple combination:

```ts
await page.keyboard.press('Control+A');
```

is usually easier to read.

Use:

```text
down()
+
up()
```

when you need the modifier to remain held across multiple operations.

---

# 15. `keyboard.insertText()`

`insertText()` inserts text directly through keyboard input semantics rather than simulating individual key presses.

Conceptually:

```ts
await page.keyboard.insertText('Hello World');
```

This can be useful for:

```text
Text editors
IME-related scenarios
Special character input
Rich text controls
```

---

# `insertText()` vs `type()`

Think:

```text
type()
   ↓
Character-oriented typing

insertText()
   ↓
Insert text directly
```

The exact event behavior differs, so choose based on the behavior being tested.

---

# 16. Keyboard Focus

Keyboard APIs operate against the currently focused context.

For example:

```ts
await page.getByLabel('Search').focus();
await page.keyboard.press('Enter');
```

This means:

```text
Search field
   ↓
Focused
   ↓
Keyboard action
```

---

# Important Rule

Before using:

```ts
page.keyboard
```

ask:

```text
Which element currently has focus?
```

Otherwise your keyboard action may affect:

```text
Body
Unexpected Control
Modal
Different Input
```

---

# Better When Possible

If the key is intended for a specific control:

```ts
await search.press('Enter');
```

is often clearer than:

```ts
await page.keyboard.press('Enter');
```

because the target is explicit.

---

# 17. Keyboard Navigation

Keyboard navigation is especially important for accessibility testing.

Example:

```text
Tab
 ↓
Search
 ↓
Tab
 ↓
Filter
 ↓
Tab
 ↓
Submit
```

A test may intentionally validate this flow.

---

# Keyboard Navigation Test

Conceptually:

```ts
await page.keyboard.press('Tab');
await expect(page.getByRole('textbox', { name: 'Search' })).toBeFocused();

await page.keyboard.press('Tab');
await expect(page.getByRole('button', { name: 'Filter' })).toBeFocused();
```

The important thing is:

```text
Focus Order
```

rather than mouse interaction.

---

# Accessibility Perspective

Keyboard navigation tests can validate:

```text
Focusable Controls
Focus Order
Keyboard Reachability
Escape Behavior
Arrow Navigation
Enter/Space Activation
Focus Traps
Dialog Navigation
```

---

# 18. `Tab`

`Tab` typically moves focus forward.

```ts
await page.keyboard.press('Tab');
```

---

# 19. `Shift+Tab`

Moves focus backward.

```ts
await page.keyboard.press('Shift+Tab');
```

This is especially useful for validating:

```text
Reverse keyboard navigation
```

---

# 20. `Escape`

Commonly used to close:

```text
Modal
Dropdown
Tooltip
Command Palette
Menu
```

Example:

```ts
await page.keyboard.press('Escape');
```

---

# 21. Arrow Keys

Common in:

```text
Dropdowns
Listboxes
Menus
Carousels
Date Pickers
Tree Views
Grids
```

Examples:

```ts
await page.keyboard.press('ArrowDown');
await page.keyboard.press('ArrowUp');
```

---

# 22. Enter and Space

Interactive controls may respond to:

```text
Enter
Space
```

For accessibility testing, it can be important to verify that keyboard users can activate controls.

Example:

```ts
await page.keyboard.press('Enter');
```

or:

```ts
await page.keyboard.press('Space');
```

depending on the control's expected behavior.

---

# 23. Keyboard Shortcuts

Applications may expose shortcuts such as:

```text
Ctrl+S
Ctrl+K
Ctrl+F
Escape
Alt+ArrowLeft
```

These can be tested directly.

Example:

```ts
await page.keyboard.press('Control+K');
```

---

# 24. Rich Text Editors

Rich text editors can be difficult because the editable region may not be a standard:

```html
<input>
```

or:

```html
<textarea>
```

It might use:

```html
<div contenteditable="true">
```

or a complex editor framework.

---

# Rich Text Interaction

Possible strategy:

```text
Locate editor
      ↓
Focus
      ↓
Keyboard actions
      ↓
Type / insert text
      ↓
Keyboard shortcuts
```

Example:

```ts
await editor.focus();
await page.keyboard.press('Control+B');
await page.keyboard.insertText('Important');
```

---

# 25. Canvas Applications

Canvas is one of the clearest use cases for low-level mouse APIs.

A canvas may not expose individual graphical objects as normal DOM elements.

For example:

```text
Canvas
 ┌──────────────────────┐
 │          ●           │
 │       /              │
 │      /               │
 │     ●                │
 └──────────────────────┘
```

There may be no:

```text
button
div
input
```

for the drawn objects.

---

# Canvas Interaction

You may need:

```ts
await page.mouse.move(100, 100);
await page.mouse.down();
await page.mouse.move(200, 150);
await page.mouse.up();
```

This models:

```text
Draw line
```

---

# 26. Resizing UI Components

A resizable panel may expose a drag handle.

Example:

```text
Panel       │
            │ ← Resize Handle
            │
```

You may need:

```text
Mouse move
Mouse down
Mouse move
Mouse up
```

---

# 27. Dragging Manually

A manual drag can conceptually be:

```ts
await page.mouse.move(startX, startY);
await page.mouse.down();
await page.mouse.move(endX, endY);
await page.mouse.up();
```

This is lower-level than:

```ts
source.dragTo(target);
```

---

# When Manual Drag Is Appropriate

Use it when:

```text
dragTo()
```

does not correctly represent the application's interaction model.

For example:

```text
Canvas
Custom gesture
Slider
Resize handle
Map interaction
```

---

# 28. Sliders

A slider can sometimes be controlled semantically.

If it exposes:

```text
role="slider"
```

prefer:

```text
Keyboard
ARIA semantics
Locator-based interaction
```

when appropriate.

If the application requires physical pointer movement:

```text
Mouse API
```

may be necessary.

---

# 29. Mouse Wheel and Infinite Scroll

An infinite scrolling page may load more data as the user scrolls.

A realistic test might perform:

```text
Scroll
 ↓
API request
 ↓
More content
 ↓
Scroll again
```

In such cases:

```ts
page.mouse.wheel()
```

can model actual scrolling behavior.

But if your goal is simply:

```text
Find an element
```

you should usually let locator actions scroll the target into view.

---

# 30. Mouse Movement and Hover

Sometimes a complex UI responds to exact pointer movement.

For example:

```text
Move to menu
   ↓
Submenu
   ↓
Move into submenu
   ↓
Submenu remains open
```

A high-level:

```ts
hover()
```

is usually preferable.

But specialized pointer choreography may require:

```text
mouse.move()
```

---

# 31. Mouse Coordinates and Viewport

Coordinate interaction depends heavily on:

```text
Viewport Size
Device Scale
Scrolling
Responsive Layout
```

Therefore tests using coordinates should establish a predictable environment.

---

# Coordinate-Based Test Risk

```text
Test
 ↓
Coordinate 500,300
 ↓
Viewport changes
 ↓
Wrong element
 ↓
Wrong action
```

This is why coordinate APIs should not be the default.

---

# 32. Mouse and Scroll Containers

Not all scrolling belongs to the page.

A component may have its own scroll container:

```text
Page
 └── Table Container
       └── Scrollable Area
```

In such cases, direct mouse wheel behavior may interact with the currently hovered or relevant scroll context.

The test should understand which container is supposed to scroll.

---

# 33. Mouse Actions and Actionability

High-level locator actions provide richer target semantics.

With:

```ts
locator.click()
```

Playwright knows:

```text
Which element
```

and can perform actionability checks.

With:

```ts
page.mouse.click(500, 300)
```

you provide:

```text
Coordinate
```

not:

```text
Logical target
```

Therefore low-level APIs shift more responsibility to the test author.

---

# Responsibility Shift

```text
High-Level API
    ↓
Playwright handles more interaction context

Low-Level API
    ↓
Test author handles more context
```

This is an important architectural tradeoff.

---

# 34. Low-Level APIs and Synchronization

Before using low-level coordinates, you may need to ensure:

```text
Page loaded
Correct viewport
Target rendered
Target position stable
Overlay absent
Scroll position correct
```

Otherwise coordinates may point to the wrong location.

---

# 35. High-Level vs Low-Level Example

## Preferred

```ts
await page.getByRole('button', { name: 'Save' }).click();
```

## Specialized

```ts
await page.mouse.click(512, 340);
```

The second should require a reason.

---

# 36. Example: Canvas Drawing

A legitimate low-level test:

```ts
await page.mouse.move(100, 100);
await page.mouse.down();
await page.mouse.move(150, 150);
await page.mouse.move(200, 100);
await page.mouse.up();
```

Business meaning:

```text
Draw a shape on canvas.
```

There may be no meaningful DOM locator for the drawn path.

---

# 37. Example: Keyboard Shortcut

A legitimate keyboard test:

```ts
await page.keyboard.press('Control+S');
```

Business meaning:

```text
Save document using keyboard shortcut.
```

This is much more appropriate than:

```text
Click Save
```

because the requirement is specifically keyboard-driven.

---

# 38. Example: Accessibility Navigation

```ts
await page.keyboard.press('Tab');
await expect(page.getByRole('button', { name: 'Save' })).toBeFocused();
```

This tests:

```text
Keyboard accessibility
```

rather than mouse behavior.

---

# 39. Example: Custom Slider

Suppose a custom slider responds to pointer movement.

Possible architecture:

```text
Slider
 ↓
Identify handle
 ↓
Determine target coordinates
 ↓
Mouse movement
 ↓
Mouse down
 ↓
Drag
 ↓
Mouse up
 ↓
Verify value
```

The implementation should be encapsulated in a component abstraction.

---

# 40. Encapsulate Low-Level Interactions

Do not scatter:

```ts
page.mouse.move(...)
page.mouse.down(...)
page.mouse.up(...)
```

through hundreds of tests.

Instead:

```text
SliderComponent
    ↓
setValue()
```

or:

```text
CanvasComponent
    ↓
drawLine()
```

This keeps tests business-readable.

---

# Component Abstraction

Instead of:

```ts
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.move(300, 200);
await page.mouse.up();
```

the test can say:

```ts
await canvas.drawLine(...);
```

The low-level implementation stays inside the component.

---

# 41. Enterprise Low-Level Interaction Pattern

```text
Test
 ↓
Business Action
 ↓
Component Method
 ↓
High-Level or Low-Level Playwright API
 ↓
Browser
```

This is preferable to exposing browser mechanics everywhere.

---

# 42. Keyboard Component Abstraction

A rich editor component might expose:

```ts
await editor.selectAll();
await editor.bold();
await editor.insertText('Hello');
```

Internally:

```text
keyboard.press()
keyboard.down()
keyboard.up()
insertText()
```

The test remains readable.

---

# 43. Why This Matters

Without abstraction:

```text
Tests
 ↓
Raw Coordinates
 ↓
Raw Keyboard Events
 ↓
Implementation Details
```

With abstraction:

```text
Tests
 ↓
Business Behavior
 ↓
Component API
 ↓
Implementation
```

---

# 44. Enterprise Governance

A mature framework can establish:

```text
Rule 1:
Prefer locator actions.

Rule 2:
Use mouse/keyboard APIs for genuine low-level requirements.

Rule 3:
Avoid hard-coded coordinates where semantic locators exist.

Rule 4:
Encapsulate coordinate interactions.

Rule 5:
Document viewport assumptions.

Rule 6:
Review low-level interactions.

Rule 7:
Prefer accessibility-oriented keyboard tests where relevant.
```

---

# 45. When Low-Level APIs Are Appropriate

Good candidates include:

```text
Canvas
Maps
Drawing tools
Complex drag gestures
Resize handles
Rich text editors
Keyboard shortcuts
Custom sliders
Specialized editors
Games
CAD-like interfaces
Pointer gesture testing
```

---

# 46. When Low-Level APIs Are NOT Appropriate

Avoid them for normal:

```text
Buttons
Links
Inputs
Checkboxes
Radio buttons
Native selects
Menus
Dialogs
Tables
Cards
```

when semantic locator actions can represent the interaction.

---

# 47. Anti-Pattern: Coordinate Everything

Bad:

```ts
page.mouse.click(120, 300);
page.mouse.click(240, 500);
page.mouse.click(800, 200);
```

This creates a test that depends on:

```text
Screen Geometry
```

rather than:

```text
Application Meaning
```

---

# 48. Anti-Pattern: Keyboard Everything

Do not replace:

```ts
await page.getByLabel('Email').fill(email);
```

with:

```text
Tab
Tab
Type
Tab
```

unless you are specifically testing keyboard navigation.

Otherwise the test becomes unnecessarily dependent on focus order.

---

# 49. Anti-Pattern: Manual Mouse Drag Everywhere

If:

```ts
source.dragTo(target)
```

works correctly, there is usually no reason to implement:

```text
move
down
move
move
up
```

manually.

---

# 50. Anti-Pattern: Using Low-Level APIs to Bypass Bad Locators

Bad reasoning:

```text
Locator is difficult
 ↓
Use coordinate click
```

This hides a locator problem.

The better approach is:

```text
Understand the UI
 ↓
Improve locator
```

unless the UI truly requires coordinates.

---

# 51. Decision Tree

```text
What interaction is required?
          │
          ▼
Can a semantic locator express it?
          │
     ┌────┴────┐
     ▼         ▼
    YES        NO
     │         │
     ▼         ▼
High-Level   Specialized
Action        Interaction
               │
       ┌───────┴────────┐
       ▼                ▼
    Mouse            Keyboard
       │                │
       ▼                ▼
page.mouse        page.keyboard
```

---

# 52. Enterprise Interaction Hierarchy

```text
Level 1
Semantic Locator + Action
        ↓
Level 2
Component Abstraction
        ↓
Level 3
Specialized High-Level Action
        ↓
Level 4
Mouse / Keyboard APIs
        ↓
Level 5
Direct DOM / Browser Evaluation
```

The lower the level:

```text
Higher maintenance responsibility
```

---

# 53. Real Interview Questions

## Junior-Level

### Q1. What is `page.mouse`?

**Answer:**

`page.mouse` provides lower-level mouse control for operations such as movement, button press/release, clicking, double-clicking, and wheel scrolling.

---

### Q2. What is `page.keyboard`?

**Answer:**

`page.keyboard` provides lower-level keyboard interaction such as pressing keys, holding keys, releasing keys, and inserting text.

---

### Q3. What does `mouse.move()` do?

**Answer:**

It moves the mouse pointer to a specified coordinate.

---

### Q4. What do `mouse.down()` and `mouse.up()` do?

**Answer:**

They press and release a mouse button, allowing construction of custom gestures such as dragging or drawing.

---

# Mid-Level

### Q5. Why are coordinate-based actions fragile?

**Answer:**

They depend on viewport size, layout, scrolling, responsive design, zoom, and element position rather than stable application semantics.

---

### Q6. When would you use `page.keyboard` instead of `locator.press()`?

**Answer:**

I would use `locator.press()` when the key belongs to a specific target element. I would use `page.keyboard` when I intentionally need page-level keyboard control or complex keyboard sequences.

---

### Q7. What is the difference between `keyboard.press()` and `keyboard.down()`?

**Answer:**

`press()` represents a key press and release, while `down()` holds a key down until an explicit `up()` is performed.

---

# Senior-Level

### Q8. Give examples where low-level mouse APIs are appropriate.

**Answer:**

Canvas drawing, custom drag gestures, resizing handles, map interactions, specialized sliders, and other interfaces where there is no reliable semantic DOM interaction.

---

### Q9. Why should low-level interactions be encapsulated?

**Answer:**

Because raw coordinates and event sequences are implementation details. Encapsulation keeps tests readable and prevents the same fragile interaction logic from being duplicated across many tests.

---

### Q10. Why is keyboard automation important for accessibility testing?

**Answer:**

It verifies that interactive controls can be reached and operated without a mouse, including focus order, keyboard activation, Escape behavior, arrow navigation, and keyboard shortcuts.

---

# Lead-Level

### Q11. How would you govern low-level APIs in an enterprise framework?

**Answer:**

I would establish semantic locator actions as the default, require justification for coordinate-based mouse operations, encapsulate low-level gestures inside component abstractions, document viewport assumptions, and review raw mouse/keyboard usage during code review.

---

### Q12. How would you test a canvas application?

**Answer:**

I would use semantic locators for surrounding controls where possible, and use `page.mouse` for canvas gestures that cannot be represented through DOM elements. The mouse implementation would be encapsulated inside a Canvas component abstraction, with assertions validating the resulting application state.

---

# Architect-Level

### Q13. What is the architectural trade-off of low-level interaction?

**Strong Answer:**

Low-level APIs provide precision but transfer more responsibility to the test author. The test becomes more coupled to coordinates, focus, viewport state, and event sequencing. Therefore they should be used only when higher-level semantic interactions cannot accurately represent the application's behavior.

---

### Q14. How should an enterprise framework handle coordinate-based automation?

**Strong Answer:**

Coordinate-based automation should be isolated inside specialized component abstractions, with explicit viewport assumptions and stable test environments. It should not be exposed as the default interaction mechanism. Semantic locator actions should remain the preferred strategy wherever possible.

---

# Knowledge Check

Answer these before continuing:

1. What is `page.mouse`?
2. What is `page.keyboard`?
3. What does `mouse.move()` do?
4. What does `mouse.down()` do?
5. What does `mouse.up()` do?
6. What does `mouse.click()` do?
7. What does `mouse.dblclick()` do?
8. What does `mouse.wheel()` do?
9. Why are coordinates fragile?
10. What is the difference between `locator.click()` and `page.mouse.click()`?
11. What does `keyboard.press()` do?
12. What does `keyboard.down()` do?
13. What does `keyboard.up()` do?
14. What does `keyboard.insertText()` do?
15. Why does focus matter for keyboard actions?
16. What is keyboard navigation testing?
17. Why are Tab and Shift+Tab important?
18. When would you use Escape?
19. When are arrow keys useful?
20. What are keyboard shortcuts?
21. When is canvas a good candidate for mouse APIs?
22. When is a rich text editor a good candidate for keyboard APIs?
23. Why should coordinate actions be encapsulated?
24. Why should you prefer `dragTo()` when it accurately models the requirement?
25. Why should low-level APIs not replace good locators?
26. What is the enterprise interaction hierarchy?
27. What is the difference between high-level and low-level interaction?
28. How do low-level APIs affect synchronization responsibility?
29. How would you test an accessibility keyboard flow?
30. When should you use low-level interaction in production automation?

---

# Step Summary

The most important principle from this lesson is:

> **Use the highest-level interaction that accurately represents the behavior under test.**

Your interaction hierarchy should be:

```text
Business Intent
      ↓
Semantic Locator
      ↓
High-Level Action
      ↓
Specialized Low-Level Action
      ↓
Browser Interaction
```

For ordinary controls:

```text
Button
   ↓
locator.click()

Input
   ↓
locator.fill()

Checkbox
   ↓
locator.check()

Native Select
   ↓
locator.selectOption()
```

For specialized interactions:

```text
Canvas
   ↓
page.mouse

Keyboard Shortcut
   ↓
page.keyboard

Complex Gesture
   ↓
page.mouse

Accessibility Navigation
   ↓
page.keyboard
```

And the enterprise principle remains:

```text
Semantic
   ↓
Preferred

Low-Level
   ↓
Specialized

Coordinate
   ↓
Exceptional
```

---

# Progress Milestone

✅ **Step 68 completed.**

You now understand:

- `page.mouse`
- `mouse.move()`
- `mouse.down()`
- `mouse.up()`
- `mouse.click()`
- `mouse.dblclick()`
- `mouse.wheel()`
- Mouse buttons
- Coordinates
- Modifier keys
- `page.keyboard`
- `keyboard.press()`
- `keyboard.down()`
- `keyboard.up()`
- `keyboard.insertText()`
- Keyboard shortcuts
- Keyboard navigation
- Tab navigation
- Shift+Tab
- Escape
- Arrow keys
- Canvas interaction
- Rich text editors
- Dragging
- Resizing
- Scrolling
- Coordinate-based automation
- High-level vs low-level APIs
- Component encapsulation
- Enterprise low-level interaction governance

Your complete action architecture is now:

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
             ┌──────────────┴──────────────┐
             ▼                             ▼
       High-Level                       Low-Level
             │                             │
     ┌───────┼────────┐             ┌──────┴──────┐
     ▼       ▼        ▼             ▼             ▼
   click()  fill()  check()       Mouse        Keyboard
                                  │             │
                                  ▼             ▼
                              Coordinates      Keys
                                  │             │
                                  └──────┬──────┘
                                         ▼
                                  Browser Events
                                         │
                                         ▼
                                  Application State
                                         │
                                         ▼
                                      Assert
```

---

# Next Step

# Step 69 — Mastering Playwright Dropdowns: Native `<select>`, `selectOption()`, Custom Dropdowns, Comboboxes, Listboxes, Autocomplete, Searchable Dropdowns, Multi-Select, Dynamic Options, Dependent Dropdowns, Async Data, Keyboard Selection, Hidden Options, Virtualized Lists, and Enterprise Dropdown Strategy

The next lesson will deeply cover one of the most common real-world automation problems:

> **"How do I reliably automate a dropdown?"**

We will distinguish:

```text
Native Select
        vs
Custom Dropdown
        vs
Combobox
        vs
Autocomplete
        vs
Searchable Dropdown
        vs
Multi-Select
```

and build a decision tree for choosing:

```text
selectOption()
click()
fill()
press()
getByRole()
filter()
keyboard
```

without falling back unnecessarily to:

```text
XPath
CSS
nth()
```
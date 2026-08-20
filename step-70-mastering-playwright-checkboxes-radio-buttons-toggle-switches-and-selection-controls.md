# Playwright with TypeScript Mastery Bootcamp

# Stage 31 — Advanced Form & Component Interaction

# Step 70 — Mastering Playwright Checkboxes, Radio Buttons, Toggle Switches & Selection Controls

## `check()`, `uncheck()`, `isChecked()`, `setChecked()`, Radio Groups, Toggle Buttons, ARIA Switches, Custom Checkboxes, Indeterminate State, Disabled Controls, Dynamic State, Forms, Accessibility Semantics, and Enterprise Selection Strategy

---

# Objective

In the previous lesson, you learned how to automate:

```text
Native Dropdowns
Custom Dropdowns
Comboboxes
Listboxes
Autocomplete
Searchable Dropdowns
Dependent Dropdowns
Virtualized Lists
```

Now we move to another major category of form controls:

```text
Checkbox
Radio Button
Toggle Switch
Custom Selection Control
```

These controls may look similar visually, but their behavior and semantics are different.

You will learn:

- `check()`
- `uncheck()`
- `isChecked()`
- `setChecked()`
- Checkbox semantics
- Radio button semantics
- Radio groups
- Toggle switches
- ARIA `switch`
- Custom checkboxes
- Indeterminate state
- Disabled controls
- Dynamic selection state
- Form validation
- Accessible roles
- `aria-checked`
- `aria-selected`
- State-based assertions
- Checkbox groups
- Select All / Deselect All
- Parent-child selection
- Permission matrices
- Enterprise selection-control strategy

---

# 1. Why Selection Controls Matter

Modern applications contain many controls that represent state.

Examples:

```text
☐ Remember me

☐ Accept Terms

○ Male
○ Female
○ Other

[ ON ] Email Notifications

☑ Select All
```

These controls are not interchangeable.

A professional automation engineer must understand:

```text
What does this control mean?
```

before deciding:

```text
How should I automate it?
```

---

# 2. Selection Control Architecture

```text
                 Selection Control
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
      Checkbox        Radio         Switch
          │             │             │
          ▼             ▼             ▼
    Independent      Exclusive      Binary
       State           Choice         State
```

---

# 3. Checkbox

A checkbox generally represents an independent boolean choice.

Example:

```text
☐ Remember Me
```

The user can change it independently.

Another checkbox can also be selected:

```text
☑ Remember Me
☑ Accept Terms
☐ Subscribe
```

Each state is independent.

---

# 4. Checkbox Mental Model

```text
Unchecked
    │
    ▼
 Checked
    │
    ▼
Unchecked
```

A checkbox normally has two primary states:

```text
checked
unchecked
```

Some implementations can also support:

```text
indeterminate
```

which we will cover later.

---

# 5. Native Checkbox

HTML:

```html
<input
  type="checkbox"
  id="remember"
>

<label for="remember">
  Remember me
</label>
```

Playwright:

```ts
const remember = page.getByRole('checkbox', {
  name: 'Remember me'
});
```

---

# 6. `check()`

Use:

```ts
await remember.check();
```

This expresses:

```text
Ensure checkbox is checked.
```

It does not merely mean:

```text
Click whatever state currently exists.
```

---

# 7. Why `check()` Is Better Than `click()`

Suppose:

```text
Checkbox currently checked
```

If you do:

```ts
await remember.click();
```

the checkbox becomes:

```text
unchecked
```

That may be the opposite of your intended state.

With:

```ts
await remember.check();
```

the desired state is explicit:

```text
Final state = checked
```

---

# 8. `uncheck()`

To ensure a checkbox is unchecked:

```ts
await remember.uncheck();
```

Mental model:

```text
Current State
     │
     ▼
uncheck()
     │
     ▼
Final State = Unchecked
```

---

# 9. `isChecked()`

To inspect checkbox state:

```ts
const checked = await remember.isChecked();
```

It returns:

```text
true
```

or:

```text
false
```

---

# Example

```ts
const remember = page.getByRole('checkbox', {
  name: 'Remember me'
});

await remember.check();

expect(await remember.isChecked()).toBe(true);
```

---

# 10. Prefer Assertions for Test Verification

Although:

```ts
isChecked()
```

is useful when program logic needs the state, Playwright assertions are usually better for test verification.

Example:

```ts
await expect(remember).toBeChecked();
```

This gives you:

```text
Assertion
+
Playwright waiting behavior
+
Better diagnostics
```

---

# 11. `toBeChecked()`

Example:

```ts
await expect(
  page.getByRole('checkbox', {
    name: 'Remember me'
  })
).toBeChecked();
```

This verifies:

```text
Checked = true
```

---

# 12. Verify Unchecked State

Depending on the assertion style, you can verify the opposite state.

Conceptually:

```ts
await expect(remember).not.toBeChecked();
```

This means:

```text
Checkbox should not be checked.
```

---

# 13. `setChecked()`

Playwright also provides:

```text
setChecked()
```

This is useful when you want to explicitly establish the desired state.

Example:

```ts
await remember.setChecked(true);
```

or:

```ts
await remember.setChecked(false);
```

---

# 14. `setChecked()` Mental Model

```text
setChecked(true)
        ↓
Final state = checked
```

and:

```text
setChecked(false)
        ↓
Final state = unchecked
```

This can be useful when the desired state is determined dynamically.

---

# 15. `check()` vs `setChecked()`

Both can establish checked state.

Conceptually:

```text
check()
 ↓
Ensure checked
```

while:

```text
setChecked(true)
 ↓
Explicit desired state
```

For normal test readability:

```text
check()
```

is often very clear.

For reusable abstractions:

```text
setChecked(desiredState)
```

can be convenient.

---

# 16. Dynamic Checkbox Helper

A reusable component may expose:

```ts
async setSelected(value: boolean) {
  await this.checkbox.setChecked(value);
}
```

Then the test can say:

```ts
await termsCheckbox.setSelected(true);
```

This is useful when building generic form components.

---

# 17. Radio Buttons

Radio buttons represent mutually exclusive choices.

Example:

```text
○ Credit Card
○ UPI
○ Net Banking
```

Only one option in the group should normally be selected.

---

# 18. Radio Group Mental Model

```text
Payment Method

○ Credit Card
○ UPI
○ Net Banking
```

Select:

```text
UPI
```

Result:

```text
○ Credit Card
● UPI
○ Net Banking
```

---

# 19. Radio vs Checkbox

| Property | Checkbox | Radio |
|---|---|---|
| Independent choices | Yes | No |
| Multiple selections | Usually | Usually no |
| Binary state | Yes | Selection within group |
| Typical meaning | Include/exclude | Choose one |
| Playwright action | `check()` | `check()` |

The action may be the same:

```text
check()
```

but the business semantics are different.

---

# 20. Radio Example

HTML:

```html
<input
  type="radio"
  name="payment"
  value="card"
>

<input
  type="radio"
  name="payment"
  value="upi"
>
```

Playwright:

```ts
await page.getByRole('radio', {
  name: 'UPI'
}).check();
```

---

# 21. Verify Radio Selection

```ts
const upi = page.getByRole('radio', {
  name: 'UPI'
});

await upi.check();

await expect(upi).toBeChecked();
```

---

# 22. Radio Group Validation

Selecting one option should generally result in:

```text
Selected Option = UPI
Other Options = Unselected
```

Example:

```ts
await upi.check();

await expect(upi).toBeChecked();

await expect(
  page.getByRole('radio', { name: 'Credit Card' })
).not.toBeChecked();
```

---

# 23. Radio Group Business Meaning

Suppose the requirement says:

```text
Select UPI as payment method.
```

A good test expresses:

```text
Payment Method
      ↓
UPI
```

rather than:

```text
Radio button #2
```

---

# 24. Avoid Positional Radio Selection

Bad:

```ts
await page.getByRole('radio').nth(1).check();
```

Why?

Because the order could change:

```text
Credit Card
UPI
Net Banking
```

becomes:

```text
UPI
Credit Card
Net Banking
```

Now:

```text
nth(1)
```

selects:

```text
Credit Card
```

---

# 25. Toggle Switch

A toggle switch represents an on/off state.

Example:

```text
Email Notifications

[ ON ]
```

or:

```text
[ OFF ]
```

---

# Toggle vs Checkbox

Visually they may look different:

```text
Checkbox:
☑

Switch:
[●────]
```

Semantically, both can represent binary state, but a switch often communicates:

```text
Immediate activation/deactivation
```

Examples:

```text
Dark Mode
Notifications
Auto Save
Location Services
Two-Factor Authentication
```

---

# 26. ARIA Switch

Accessible custom switches may expose:

```text
role="switch"
```

Example:

```html
<button
  role="switch"
  aria-checked="true"
>
  Notifications
</button>
```

Playwright can locate:

```ts
page.getByRole('switch', {
  name: 'Notifications'
});
```

---

# 27. Important Distinction

Do not assume:

```text
Visual Toggle
=
HTML checkbox
```

The actual implementation may be:

```text
button
div
input
custom component
ARIA switch
```

Inspect the semantics.

---

# 28. Testing an ARIA Switch

Depending on the component's supported interaction:

```ts
const notifications = page.getByRole('switch', {
  name: 'Notifications'
});

await notifications.click();
```

Then verify its state using the appropriate accessible state assertion.

The key principle is:

```text
Interaction
+
State Verification
```

---

# 29. `aria-checked`

Custom checkable controls may expose:

```text
aria-checked
```

Example:

```html
<div
  role="checkbox"
  aria-checked="true"
>
  Accept Terms
</div>
```

This communicates:

```text
Checked
```

to assistive technologies.

---

# 30. `aria-checked` States

For certain ARIA widgets, the state can be:

```text
true
false
mixed
```

`mixed` is particularly important for:

```text
Indeterminate selection
```

---

# 31. Indeterminate Checkbox

Consider:

```text
Select All
```

There are three child items:

```text
☑ Java
☐ TypeScript
☐ Python
```

The parent cannot honestly be:

```text
checked
```

because not everything is selected.

It also cannot simply be:

```text
unchecked
```

because something is selected.

Therefore:

```text
Indeterminate
```

is useful.

---

# 32. Indeterminate State

Mental model:

```text
None Selected
      ↓
Unchecked

Some Selected
      ↓
Indeterminate

All Selected
      ↓
Checked
```

---

# 33. Select-All Architecture

```text
Select All
    │
    ├── Java
    ├── TypeScript
    └── Python
```

Possible states:

```text
None
 ↓
Select All = Unchecked

Some
 ↓
Select All = Indeterminate

All
 ↓
Select All = Checked
```

---

# 34. Enterprise Example

Permission management:

```text
Users
 ├── View
 ├── Create
 ├── Edit
 └── Delete
```

Parent:

```text
Users
```

If only:

```text
View
Edit
```

are selected:

```text
Users = Indeterminate
```

---

# 35. Testing Select-All

A mature test should verify state transitions.

```text
Initial
 ↓
None selected
 ↓
Select All
 ↓
All selected
 ↓
Unselect one
 ↓
Select All becomes indeterminate
```

This is a powerful real-world scenario.

---

# 36. Disabled Checkbox

A checkbox may be disabled:

```text
☐ Subscribe
```

because another condition has not been satisfied.

Example:

```text
Account Type = Guest

☐ Premium Features
```

The checkbox may be:

```text
disabled
```

---

# 37. Testing Disabled State

Verify:

```text
Checkbox is disabled
```

rather than trying to force a selection.

The test should respect application state.

---

# 38. Enabled State

After changing the prerequisite:

```text
Account Type = Premium
```

the checkbox may become:

```text
enabled
```

Then:

```text
check()
```

can be performed.

---

# 39. Dynamic Checkbox

Example:

```text
☐ Subscribe to newsletter
```

When the user selects:

```text
Marketing preferences
```

the checkbox may become available.

This is another example of:

```text
Application State
```

controlling:

```text
Element State
```

---

# 40. Checkbox Dependencies

Consider:

```text
☐ Enable Advanced Settings
```

When checked:

```text
Advanced Settings
   ├── ☐ Logging
   ├── ☐ Metrics
   └── ☐ Debug Mode
```

The child controls may initially be disabled.

---

# 41. Testing Dependencies

```text
Verify child disabled
       ↓
Check parent
       ↓
Verify child enabled
       ↓
Select child
       ↓
Verify final state
```

This is more robust than simply clicking everything.

---

# 42. Checkbox Groups

Applications often contain groups:

```text
Skills

☐ Java
☐ TypeScript
☐ Angular
☐ Playwright
```

The test may need to select:

```text
Java
Playwright
```

---

# 43. Selecting Multiple Checkboxes

```ts
await page.getByRole('checkbox', {
  name: 'Java'
}).check();

await page.getByRole('checkbox', {
  name: 'Playwright'
}).check();
```

---

# 44. Checkbox Collection

You may also have a locator representing:

```text
All skill checkboxes
```

This is a collection.

Remember the strictness lesson:

```text
Collection
≠
Single Target
```

Multiple checkboxes are expected if your goal is to:

```text
Count
Inspect
Iterate
Filter
```

---

# 45. Filtering Checkbox Groups

Suppose:

```text
Permission
   │
   ├── Read
   ├── Write
   └── Delete
```

and multiple modules exist:

```text
Users
Orders
Products
```

You should scope:

```text
Orders
   ↓
Write
```

rather than:

```text
Write
```

globally.

---

# 46. Permission Matrix

Enterprise applications often contain:

```text
             Read   Write   Delete

Users         ☐      ☐       ☐
Orders        ☐      ☐       ☐
Products      ☐      ☐       ☐
```

The same checkbox labels repeat.

---

# Permission Matrix Strategy

Use:

```text
Row
 ↓
Permission
```

For example:

```text
Orders
   ↓
Write
```

This is another practical application of:

```text
Locator Scope
```

---

# 47. Component Scoping

Conceptually:

```text
Permission Row
      ↓
Order
      ↓
Write Checkbox
```

This is much stronger than:

```text
getByRole('checkbox', { name: 'Write' }).nth(2)
```

---

# 48. Checkbox State Assertion

Use state assertions:

```ts
await expect(
  page.getByRole('checkbox', {
    name: 'Accept Terms'
  })
).toBeChecked();
```

This expresses:

```text
Expected State
```

rather than:

```text
DOM Attribute
```

---

# 49. Why State Assertions Matter

Weak:

```ts
expect(await checkbox.getAttribute('checked')).toBe('');
```

This can be misleading for dynamic controls because the HTML attribute and current DOM property state are not necessarily the same concept.

Prefer:

```ts
await expect(checkbox).toBeChecked();
```

---

# 50. Current State vs Initial HTML

This is important.

For interactive form controls:

```text
HTML Attribute
```

and:

```text
Current State
```

are not always identical.

For example:

```html
<input checked>
```

establishes initial state.

But after user interaction:

```text
checked
```

can change without rewriting the original HTML attribute in the way a beginner might expect.

Therefore test:

```text
Current Control State
```

rather than relying on static markup.

---

# 51. Radio Button State

Use:

```ts
await expect(
  page.getByRole('radio', {
    name: 'UPI'
  })
).toBeChecked();
```

This is clearer than inspecting raw attributes.

---

# 52. Toggle State

For an ARIA switch, the state may be represented by:

```text
aria-checked
```

The appropriate assertion should verify the accessible state exposed by the component.

The exact assertion strategy depends on whether the component is:

```text
checkbox
switch
button
```

---

# 53. Custom Checkbox

Some applications visually render:

```text
☑
```

using:

```text
div
span
svg
```

while the actual semantics may come from:

```text
role="checkbox"
aria-checked
```

---

# 54. Custom Checkbox Strategy

First inspect:

```text
Role
Accessible Name
State
```

Then use:

```text
getByRole()
```

when the control exposes appropriate accessibility semantics.

---

# 55. Why Accessibility Semantics Matter

A strong custom component should expose:

```text
role
+
accessible name
+
state
```

This helps:

```text
Screen Readers
Keyboard Users
Automation
```

---

# 56. Accessibility and Testability

Good accessibility often produces better automation.

For example:

```html
<div
  role="checkbox"
  aria-checked="false"
>
  Accept Terms
</div>
```

is much easier to identify semantically than:

```html
<div class="checkbox-x-39482">
```

---

# 57. Keyboard Interaction with Checkboxes

Native checkboxes can often be activated through keyboard controls.

Accessibility tests may validate:

```text
Tab
 ↓
Focus checkbox
 ↓
Space
 ↓
Checked
```

---

# 58. Keyboard Checkbox Test

Conceptually:

```ts
const terms = page.getByRole('checkbox', {
  name: 'Accept Terms'
});

await terms.focus();
await terms.press('Space');

await expect(terms).toBeChecked();
```

The exact keyboard behavior depends on the control implementation.

---

# 59. Keyboard Radio Navigation

Radio groups often support:

```text
ArrowUp
ArrowDown
ArrowLeft
ArrowRight
```

depending on the widget.

Accessibility tests can validate:

```text
Focus
 ↓
Arrow
 ↓
Selection
```

---

# 60. Toggle Keyboard Interaction

A switch may be keyboard-operable using:

```text
Space
```

or another appropriate interaction according to its semantics.

This should be tested when keyboard accessibility is part of the requirement.

---

# 61. Disabled Radio Button

A radio option may be disabled:

```text
○ Credit Card
○ UPI
○ Net Banking (disabled)
```

The test should verify:

```text
Net Banking = disabled
```

and ensure it cannot become selected through normal interaction.

---

# 62. Default Radio Selection

Example:

```text
● Credit Card
○ UPI
○ Net Banking
```

The test may need to verify:

```text
Default = Credit Card
```

---

# 63. Required Radio Group

A form may require one selection:

```text
Payment Method *

○ Credit Card
○ UPI
○ Net Banking
```

If none is selected:

```text
Payment method is required
```

The test should verify the validation behavior.

---

# 64. Checkbox Validation

Example:

```text
☐ Accept Terms
```

User clicks Submit.

Application displays:

```text
You must accept the terms.
```

Test:

```text
Submit
 ↓
Validation
 ↓
Error Message
```

---

# 65. Checkbox Form Workflow

```text
Open Form
    ↓
Fill Fields
    ↓
Accept Terms
    ↓
Submit
    ↓
Verify Success
```

---

# 66. Negative Checkbox Workflow

```text
Open Form
    ↓
Do NOT accept Terms
    ↓
Submit
    ↓
Verify validation error
```

Negative state testing is important.

---

# 67. `check()` and Actionability

Remember the previous lesson.

When you call:

```ts
await checkbox.check();
```

Playwright does not simply execute a blind click.

It applies the appropriate interaction checks and waits as necessary.

Conceptually:

```text
Locator
 ↓
Strictness
 ↓
Actionability
 ↓
Checkbox Interaction
 ↓
State Change
```

---

# 68. `setChecked()` and State

`setChecked()` also establishes the desired checked state through Playwright's interaction model.

Conceptually:

```text
Desired State
     ↓
Current State
     ↓
Interaction if necessary
     ↓
Final State
```

---

# 69. Checkbox State Machine

A useful model:

```text
                 ┌──────────────┐
                 │  Unchecked   │
                 └──────┬───────┘
                        │
                      check
                        │
                        ▼
                 ┌──────────────┐
                 │   Checked    │
                 └──────┬───────┘
                        │
                     uncheck
                        │
                        ▼
                 ┌──────────────┐
                 │  Unchecked   │
                 └──────────────┘
```

For parent selection:

```text
None
 ↓
Unchecked

Some
 ↓
Indeterminate

All
 ↓
Checked
```

---

# 70. Enterprise Selection-Control Architecture

```text
                 Selection Control
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      Checkbox         Radio          Switch
          │              │              │
          ▼              ▼              ▼
     Independent      Exclusive       Binary
        State           Choice          State
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Semantic Locator
                         │
                         ▼
                  State-Based Action
                         │
                         ▼
                     Assertion
```

---

# 71. Best Practices

1. Use `check()` to ensure a checkbox is selected.
2. Use `uncheck()` to ensure it is deselected.
3. Use `setChecked()` when a generic desired-state API is useful.
4. Use `toBeChecked()` for state assertions.
5. Use semantic roles where available.
6. Distinguish checkbox and radio semantics.
7. Scope repeated checkbox groups.
8. Avoid `nth()` for repeated controls unless position is intentional.
9. Test disabled states.
10. Test default states.
11. Test validation.
12. Test dependent controls.
13. Test select-all behavior.
14. Test indeterminate state where applicable.
15. Test keyboard behavior when accessibility matters.
16. Prefer current state assertions over static HTML attributes.
17. Understand custom ARIA controls.
18. Verify final business state after selection.
19. Encapsulate reusable selection components.
20. Do not force disabled controls.

---

# Common Beginner Mistakes

## Mistake 1 — Clicking Instead of Checking

```ts
await checkbox.click();
```

can toggle the state unpredictably.

Prefer:

```ts
await checkbox.check();
```

when the goal is checked state.

---

## Mistake 2 — Using `nth()`

```ts
await page.getByRole('checkbox').nth(4).check();
```

This is fragile unless position is business-relevant.

---

## Mistake 3 — Inspecting `checked` HTML Attribute

Prefer:

```ts
toBeChecked()
```

for current state verification.

---

## Mistake 4 — Treating Radio Buttons as Independent Checkboxes

Radio buttons represent a mutually exclusive group.

---

## Mistake 5 — Assuming Every Toggle Is a Checkbox

Some toggles are:

```text
switch
button
custom ARIA widget
```

---

## Mistake 6 — Ignoring Indeterminate State

Select-all controls frequently have:

```text
checked
unchecked
indeterminate
```

---

## Mistake 7 — Ignoring Disabled State

Do not force interaction with a control that should be disabled.

---

## Mistake 8 — Ignoring Parent-Child Dependencies

Selecting a parent may enable child controls.

---

## Mistake 9 — Using Raw CSS Classes

Classes may be implementation details.

Prefer:

```text
Role
Label
Accessible Name
Test ID
Business Scope
```

---

# Professional Tips

When you encounter a selection control, ask:

```text
What semantic control is this?
        ↓
Checkbox?
Radio?
Switch?
Custom widget?
        ↓
What state should it represent?
        ↓
What is the accessible name?
        ↓
Is it enabled?
        ↓
Is it currently selected?
        ↓
What is the expected final state?
```

Then choose the action.

---

# Real Interview Discussion

## Junior-Level

### Q1. How do you select a checkbox?

**Answer:**

```ts
await checkbox.check();
```

---

### Q2. How do you deselect a checkbox?

**Answer:**

```ts
await checkbox.uncheck();
```

---

### Q3. How do you verify a checkbox is selected?

**Answer:**

```ts
await expect(checkbox).toBeChecked();
```

---

### Q4. What is the difference between checkbox and radio button?

**Answer:**

Checkboxes generally represent independent selections, while radio buttons represent mutually exclusive choices within a group.

---

# Mid-Level

### Q5. What is `setChecked()`?

**Answer:**

It establishes the checkbox or supported checkable control's desired checked state, such as:

```ts
await checkbox.setChecked(true);
```

or:

```ts
await checkbox.setChecked(false);
```

---

### Q6. Why is `check()` often better than `click()`?

**Answer:**

Because `check()` expresses the desired final state. A click can toggle the current state in either direction.

---

### Q7. What is an indeterminate checkbox?

**Answer:**

It represents a partial-selection state, commonly used for a Select All control when some but not all child items are selected.

---

# Senior-Level

### Q8. How would you automate a permission matrix?

**Answer:**

I would identify the correct permission row using business context, scope to that row, and then locate the required checkbox by its accessible name. I would avoid global checkbox locators and positional selectors.

---

### Q9. How would you test a Select All control?

**Answer:**

I would test the state transitions from none selected to checked, from all selected to unchecked, and from partially selected to indeterminate. I would also verify the corresponding child checkbox states.

---

### Q10. How do you automate a custom checkbox?

**Answer:**

I first inspect its accessibility semantics. If it exposes an appropriate `checkbox` role and accessible name, I use semantic locators. Otherwise, I identify the actual supported interaction while avoiding unnecessary dependence on implementation-specific CSS or DOM structure.

---

# Lead-Level

### Q11. How would you design a reusable selection-control component?

**Answer:**

I would expose business-level methods such as:

```text
select()
deselect()
isSelected()
setSelected()
```

Internally, the component would use Playwright's appropriate semantic action and state assertion. This hides implementation details from tests.

---

# Architect-Level

### Q12. What is the enterprise strategy for checkbox, radio, and switch automation?

**Strong Answer:**

The framework should classify the control by semantic behavior rather than visual appearance. Checkboxes represent independent state, radios represent mutually exclusive choices, and switches represent binary activation state. The framework should prefer semantic locators, state-oriented actions, and state assertions. Repeated controls should be scoped by business context, while positional selectors should be treated as exceptions.

---

### Q13. Why is accessibility important for automation architecture?

**Strong Answer:**

Accessible semantics provide stable information about the intended purpose and state of controls. Roles, accessible names, and states such as `aria-checked` make automation less dependent on visual styling and implementation-specific DOM details. They also align test automation with real user interaction and accessibility requirements.

---

# Knowledge Check

1. What is a checkbox?
2. What is `check()`?
3. What is `uncheck()`?
4. What is `isChecked()`?
5. What is `setChecked()`?
6. What is `toBeChecked()`?
7. Why is `check()` better than a blind click for setting state?
8. What is a radio button?
9. Why are radio buttons mutually exclusive?
10. What is a radio group?
11. What is a toggle switch?
12. What is an ARIA switch?
13. What is `aria-checked`?
14. What is an indeterminate checkbox?
15. When is Select All indeterminate?
16. How do you test disabled controls?
17. How do you test default selection?
18. How do you test checkbox validation?
19. How do you test dependent checkboxes?
20. What is a permission matrix?
21. Why should repeated checkboxes be scoped?
22. Why is `nth()` usually weak for repeated selection controls?
23. Why should current state be asserted rather than static HTML attributes?
24. How would you test keyboard checkbox interaction?
25. How would you test radio keyboard navigation?
26. How would you test a custom switch?
27. How do accessibility semantics improve automation?
28. What is the checkbox state machine?
29. What is the enterprise selection-control architecture?
30. How would you design a reusable checkbox component?

---

# Step Summary

The key principle of this lesson is:

> **Selection controls should be automated according to their semantic state, not merely by clicking their visual representation.**

Your mental model should be:

```text
Selection Control
       ↓
Identify Semantics
       │
 ┌─────┼──────┐
 ▼     ▼      ▼
Checkbox Radio Switch
 │       │      │
 ▼       ▼      ▼
Independent Exclusive Binary
 State     Choice   State
       │
       ▼
Semantic Locator
       │
       ▼
State-Oriented Action
       │
       ▼
State Assertion
```

For checkboxes:

```text
check()
uncheck()
setChecked()
isChecked()
toBeChecked()
```

For radio buttons:

```text
check()
toBeChecked()
```

For custom switches:

```text
Semantic Locator
+
Appropriate Interaction
+
State Verification
```

For Select All:

```text
None
 ↓
Unchecked

Some
 ↓
Indeterminate

All
 ↓
Checked
```

For enterprise permission matrices:

```text
Business Row
    ↓
Permission
    ↓
Checkbox
```

rather than:

```text
Global Checkbox
    ↓
nth()
```

---

# Progress Milestone

✅ **Step 70 completed.**

You now understand:

- Checkboxes
- `check()`
- `uncheck()`
- `isChecked()`
- `setChecked()`
- `toBeChecked()`
- Radio buttons
- Radio groups
- Toggle switches
- ARIA switches
- `aria-checked`
- Checked state
- Unchecked state
- Indeterminate state
- Disabled controls
- Enabled controls
- Dynamic controls
- Dependent controls
- Select All
- Deselect All
- Permission matrices
- Checkbox groups
- Keyboard selection
- Accessibility semantics
- State-based assertions
- Component abstractions
- Enterprise selection-control architecture

Your form-control architecture is now:

```text
                         FORM CONTROL
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
      Dropdown            Selection            Input
          │                   │                   │
          ▼                   ▼                   ▼
       Native /         Checkbox / Radio      Text / Date
       Custom /         Switch / Toggle       Number / etc.
       Combobox
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ▼
                       Semantic Locator
                              │
                              ▼
                       Actionability
                              │
                              ▼
                         User Action
                              │
                              ▼
                        State Change
                              │
                              ▼
                           Assertion
```

---

# Next Step

# Step 71 — Mastering Playwright Input Fields & Forms: `fill()`, `pressSequentially()`, Input Types, Textareas, Contenteditable, Passwords, Numbers, Dates, File Inputs, Clear/Replace Strategies, Input Validation, Dynamic Forms, Masked Inputs, Debounced Fields, Character-Level Events, Form Submission, Reset, and Enterprise Form Automation

The next lesson will deeply cover real-world form automation:

```text
Text Input
Textarea
Password
Number
Email
Date
Time
Search
URL
Contenteditable
Masked Input
Dynamic Input
```

You will learn when to use:

```text
fill()
pressSequentially()
press()
keyboard
```

and how to automate forms involving:

```text
Validation
Debouncing
Dynamic Fields
Required Fields
Character Limits
Input Masks
Auto-Formatting
Form Submit
Form Reset
```

without introducing unnecessary:

```text
waitForTimeout()
XPath
CSS position
nth()
```
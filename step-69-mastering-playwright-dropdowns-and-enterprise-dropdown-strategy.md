# Playwright with TypeScript Mastery Bootcamp

# Stage 31 — Advanced Form & Component Interaction

# Step 69 — Mastering Playwright Dropdowns: Native `<select>`, `selectOption()`, Custom Dropdowns, Comboboxes, Listboxes, Autocomplete, Searchable Dropdowns, Multi-Select, Dynamic Options, Dependent Dropdowns, Async Data, Keyboard Selection, Hidden Options, Virtualized Lists, and Enterprise Dropdown Strategy

---

# Objective

Dropdowns look simple to users:

```text
Click
 ↓
Choose option
```

But in automation, a dropdown can be implemented in many different ways.

It might be:

```html
<select>
```

or:

```text
button
 ↓
listbox
 ↓
option
```

or:

```text
input
 ↓
autocomplete
 ↓
API request
 ↓
suggestions
```

or:

```text
custom React component
```

Therefore:

> **The correct Playwright strategy depends on the actual implementation of the dropdown.**

You will learn:

- Native `<select>`
- `selectOption()`
- Selecting by value
- Selecting by label
- Selecting by index
- Multiple selection
- Custom dropdowns
- `role="combobox"`
- `role="listbox"`
- `role="option"`
- Autocomplete
- Searchable dropdowns
- Dynamic options
- Async options
- Dependent dropdowns
- Cascading dropdowns
- Virtualized lists
- Hidden options
- Keyboard selection
- Multi-select components
- Loading states
- Empty states
- No-results states
- Dropdown overlays
- Strictness
- Filtering
- Enterprise dropdown architecture

---

# 1. What Is a Dropdown?

A dropdown is a UI control that allows the user to choose one or more values from a collection of options.

Example:

```text
Country
┌───────────────────────┐
│ India              ▼  │
└───────────────────────┘
```

When opened:

```text
India
United States
United Kingdom
Australia
Canada
```

---

# 2. The First Question

Before writing Playwright code, ask:

> **Is this a native HTML `<select>` or a custom dropdown?**

This single question determines much of your automation strategy.

---

# 3. Native Dropdown

A native dropdown is usually:

```html
<select>
    <option value="IN">India</option>
    <option value="US">United States</option>
    <option value="UK">United Kingdom</option>
</select>
```

For this control, Playwright provides:

```text
selectOption()
```

---

# 4. Native Dropdown Example

HTML:

```html
<label for="country">Country</label>

<select id="country">
    <option value="IN">India</option>
    <option value="US">United States</option>
    <option value="UK">United Kingdom</option>
</select>
```

Playwright:

```ts
const country = page.getByLabel('Country');

await country.selectOption('IN');
```

---

# 5. Selecting by Value

Given:

```html
<option value="IN">India</option>
```

you can use:

```ts
await country.selectOption('IN');
```

This selects:

```text
India
```

---

# 6. Selecting by Label

You can also select using the visible option label.

Example:

```ts
await country.selectOption({
  label: 'India'
});
```

This is useful when:

```text
Business requirement
=
Select India
```

rather than:

```text
Select value IN
```

---

# 7. Selecting by Index

You can also select an option by index.

Conceptually:

```text
0 → India
1 → USA
2 → UK
```

Then:

```ts
await country.selectOption({
  index: 1
});
```

selects:

```text
USA
```

---

# Important Warning

Index-based selection should be used carefully.

Why?

Because option ordering can change.

Today:

```text
India
USA
UK
```

Tomorrow:

```text
USA
India
UK
```

Now:

```text
index: 1
```

selects a different business value.

Prefer:

```text
value
```

or:

```text
label
```

when stable.

---

# 8. Multiple Select

Native `<select>` can support multiple selections.

Example:

```html
<select multiple>
    <option value="java">Java</option>
    <option value="ts">TypeScript</option>
    <option value="python">Python</option>
</select>
```

Playwright can select multiple options.

Conceptually:

```ts
await skills.selectOption([
  'java',
  'ts'
]);
```

---

# Multiple Selection

Result:

```text
Java       ✓
TypeScript ✓
Python
```

This is useful for:

```text
Skills
Permissions
Categories
Tags
Roles
```

---

# 9. Deselecting Options

Native multi-select controls can also be manipulated to remove selections.

The exact operation should be based on the desired final state.

The important principle is:

```text
Test the desired selection state,
not merely a sequence of clicks.
```

---

# 10. Custom Dropdowns

Now consider:

```html
<button>
    India
</button>
```

When clicked:

```html
<div role="listbox">
    <div role="option">India</div>
    <div role="option">USA</div>
    <div role="option">UK</div>
</div>
```

This is not a native `<select>`.

Therefore:

```text
selectOption()
```

is not the correct API.

---

# Custom Dropdown Strategy

Typical flow:

```text
Open Dropdown
      ↓
Locate Option
      ↓
Select Option
```

Example:

```ts
await page.getByRole('button', { name: 'India' }).click();

await page.getByRole('option', { name: 'United States' }).click();
```

---

# 11. Why Roles Matter

Custom accessible dropdowns may expose:

```text
combobox
listbox
option
```

These roles communicate the component's semantics.

This is often better than:

```text
div:nth-child(3)
```

or:

```text
XPath
```

---

# 12. Combobox

A combobox is often an interactive control that allows:

```text
Input
+
Selection
```

Example:

```text
Country
[ Ind              ]
```

Suggestions:

```text
India
Indonesia
Ireland
```

The input may have:

```html
role="combobox"
```

---

# Combobox Workflow

```text
Locate Combobox
      ↓
Fill Search
      ↓
Wait for Suggestions
      ↓
Locate Option
      ↓
Select
```

Example:

```ts
const country = page.getByRole('combobox', {
  name: 'Country'
});

await country.fill('Ind');

await page.getByRole('option', {
  name: 'India'
}).click();
```

---

# 13. Autocomplete

Autocomplete is similar to a combobox.

The user types:

```text
Ind
```

and the application dynamically produces:

```text
India
Indonesia
Indiana
```

The options may come from:

```text
API
Database
Cached Data
Client-side Search
```

---

# Autocomplete Architecture

```text
User Input
    ↓
"Ind"
    ↓
API Request
    ↓
Response
    ↓
Suggestions
    ↓
Render Options
```

---

# 14. Dynamic Options

Dynamic dropdowns are common.

Initially:

```text
No options
```

Then:

```text
Loading...
```

Then:

```text
India
Indonesia
Ireland
```

Your test must synchronize with the actual UI state.

---

# Bad Strategy

```ts
await input.fill('Ind');

await page.waitForTimeout(2000);

await option.click();
```

This is a fixed-time synchronization strategy.

---

# Better Strategy

Use a meaningful locator and assertion:

```ts
await input.fill('Ind');

const option = page.getByRole('option', {
  name: 'India'
});

await expect(option).toBeVisible();
await option.click();
```

The test waits for:

```text
Actual State
```

rather than:

```text
Arbitrary Time
```

---

# 15. Searchable Dropdown

A searchable dropdown may look like:

```text
Country
┌─────────────────────────┐
│ Search country...       │
└─────────────────────────┘

India
Indonesia
Ireland
```

The workflow is:

```text
Open
 ↓
Search
 ↓
Results
 ↓
Select
```

---

# Searchable Dropdown Example

```ts
await page.getByRole('button', {
  name: 'Select country'
}).click();

const search = page.getByPlaceholder('Search country');

await search.fill('Ind');

await page.getByRole('option', {
  name: 'India'
}).click();
```

---

# 16. Dependent Dropdowns

A dependent dropdown changes based on another selection.

Example:

```text
Country
[ India ]

State
[ Select State ]
```

After:

```text
Country = India
```

states become:

```text
Madhya Pradesh
Maharashtra
Gujarat
Rajasthan
```

---

# Dependent Dropdown Architecture

```text
Country
   ↓
Select India
   ↓
API / State Update
   ↓
State Options
   ↓
Select Madhya Pradesh
```

---

# 17. Cascading Dropdowns

You may have:

```text
Country
   ↓
State
   ↓
City
   ↓
Branch
```

Each selection controls the next.

Example:

```text
India
 ↓
Madhya Pradesh
 ↓
Indore
 ↓
Branch A
```

---

# Testing Cascading Dropdowns

The test should verify each state transition.

Conceptually:

```text
Select Country
      ↓
Verify State becomes available
      ↓
Select State
      ↓
Verify City becomes available
      ↓
Select City
      ↓
Verify Branch becomes available
```

---

# 18. Disabled Dependent Dropdown

Initially:

```text
Country
[ Select Country ]

State
[ Disabled ]
```

After country selection:

```text
Country
[ India ]

State
[ Enabled ]
```

The test should understand:

```text
Disabled
```

is an expected state before the dependency is satisfied.

---

# 19. Loading State

A dropdown may show:

```text
Loading...
```

while fetching options.

Example:

```text
Country
[ India ]

State
[ Loading... ]
```

Then:

```text
State
[ Madhya Pradesh ▼ ]
```

---

# Test Strategy

Do not blindly click an option while:

```text
Loading...
```

Instead:

```text
Wait for desired option/state
```

Example:

```ts
await expect(
  page.getByRole('option', {
    name: 'Madhya Pradesh'
  })
).toBeVisible();
```

---

# 20. Empty State

A search may return no results.

Example:

```text
Search:
XYZ123

No results found
```

This is a valid business state.

Your test may need to verify:

```text
No results found
```

rather than attempting to click an option.

---

# 21. No-Results Testing

Example:

```ts
await search.fill('XYZ123');

await expect(
  page.getByText('No results found')
).toBeVisible();
```

This is a good example of why automation should model:

```text
Business State
```

not only:

```text
Successful Selection
```

---

# 22. Option Strictness

Suppose the page has:

```text
India
India
India
```

A broad locator:

```ts
getByRole('option', { name: 'India' })
```

could match multiple elements.

You need to understand:

```text
Which listbox?
Which dropdown?
Which component?
```

---

# Scope the Dropdown

Example:

```ts
const countryDropdown = page.getByRole('listbox', {
  name: 'Country'
});

await countryDropdown
  .getByRole('option', { name: 'India' })
  .click();
```

The exact structure depends on the application's accessible DOM.

The principle is:

```text
Dropdown Scope
      ↓
Option
```

rather than:

```text
Page
 ↓
Global Option
```

---

# 23. Filtering Options

Suppose a list contains:

```text
India
India - Remote
India - Office
```

You may need additional context.

Filtering can help:

```text
Option collection
 ↓
Meaningful condition
 ↓
Target option
```

---

# 24. Virtualized Dropdowns

Large dropdowns may contain thousands of options.

The application may render only:

```text
Visible options
```

instead of:

```text
All 10,000 options
```

This is called:

```text
Virtualization
```

---

# Virtualized List Architecture

```text
10,000 Items
     ↓
Only visible portion rendered
     ↓
Scroll
     ↓
More items rendered
```

---

# Why This Matters

You cannot always assume:

```text
All options exist in the DOM.
```

The desired option may need to be:

```text
Searched
Scrolled
Loaded
Rendered
```

---

# 25. Virtualized Dropdown Strategy

If the component supports search:

```text
Use Search
```

rather than manually scrolling through thousands of options.

Preferred:

```text
Search "Mumbai"
 ↓
Option rendered
 ↓
Select
```

instead of:

```text
Scroll 100 times
```

---

# 26. Keyboard Dropdown Selection

Dropdowns often support:

```text
ArrowDown
ArrowUp
Enter
Escape
```

Example:

```ts
await combo.press('ArrowDown');
await combo.press('Enter');
```

This may be useful when:

```text
Keyboard interaction
```

is explicitly part of the requirement.

---

# 27. Accessibility Dropdown Testing

A mature test suite should verify keyboard behavior when accessibility is important.

Example:

```text
Focus Combobox
 ↓
ArrowDown
 ↓
Option highlighted
 ↓
Enter
 ↓
Value selected
```

---

# 28. Escape Behavior

Many custom dropdowns close with:

```text
Escape
```

Example:

```ts
await combo.press('Escape');
```

Possible assertion:

```text
Listbox disappears
```

---

# 29. Multi-Select Custom Dropdown

Example:

```text
Skills
[ Java × ] [ TypeScript × ] ▼
```

Opening:

```text
☐ Java
☐ TypeScript
☐ Python
☐ Angular
```

You may need to:

```text
Open
 ↓
Select Java
 ↓
Select TypeScript
 ↓
Close
```

---

# Multi-Select Strategy

Prefer semantic state where possible.

For example:

```text
Checkbox options
```

may be represented by:

```text
role=checkbox
```

Then:

```ts
await page.getByRole('checkbox', {
  name: 'Java'
}).check();
```

---

# 30. Custom Dropdown vs Checkbox List

Not every multi-select should be treated as:

```text
dropdown
```

Some components are actually:

```text
Menu
+
Checkboxes
```

The DOM semantics should determine your Playwright strategy.

---

# 31. Dropdown Overlays

Custom dropdowns frequently render their option list outside the original component hierarchy.

For example:

```text
Dropdown Button
```

may be in:

```text
Form
```

while:

```text
Listbox
```

is mounted under:

```text
body
```

through a portal.

---

# Portal-Based Dropdown

Conceptually:

```text
Form
 └── Dropdown Button


Body
 └── Listbox
      ├── Option
      ├── Option
      └── Option
```

This can make naive parent-child scoping fail.

---

# Important Lesson

Do not assume:

```text
Visual Child
=
DOM Child
```

Modern frameworks often use:

```text
Portals
Overlays
CDK overlays
Popper
Floating UI
```

---

# 32. Portal Dropdown Strategy

The strategy may be:

```text
Locate trigger
 ↓
Open dropdown
 ↓
Locate visible listbox
 ↓
Locate option
 ↓
Select
```

The option may not be a descendant of the trigger's DOM node.

---

# 33. Hidden Options

Some applications keep dropdown options in the DOM while hiding them.

Example:

```text
Option A
displayed

Option B
hidden
```

A broad locator can potentially encounter multiple DOM matches.

Use the visible/current dropdown context appropriately.

---

# 34. Multiple Dropdowns

A page may contain:

```text
Country
State
City
Status
Category
```

and each may contain:

```text
Option
```

Global option locators can become ambiguous.

The test should scope according to:

```text
Dropdown identity
```

or:

```text
Visible listbox
```

---

# 35. Dropdown Identification

Good:

```text
Country
State
City
```

Weak:

```text
Dropdown 1
Dropdown 2
Dropdown 3
```

Tests should use:

```text
Accessible Name
Business Meaning
Test ID
```

when available.

---

# 36. Autocomplete Race Conditions

Consider:

```text
User types:
Ind
```

API request A:

```text
Ind
```

Then user types:

```text
India
```

API request B:

```text
India
```

Responses can arrive in different orders.

A well-designed application should handle this.

Automation should assert the final intended state.

---

# Race Condition Model

```text
Input A
 ↓
Request A
 ↓
Input B
 ↓
Request B
 ↓
Response B
 ↓
Response A
```

If the UI incorrectly renders A after B, that may be an application defect.

---

# 37. Test the Final Business State

Do not simply assert:

```text
API request happened
```

if the requirement is:

```text
Select India
```

Verify:

```text
India selected
```

---

# 38. Dropdown Validation

After selecting:

```text
India
```

verify the selected state.

For example:

```ts
await expect(
  page.getByRole('combobox', { name: 'Country' })
).toHaveValue('IN');
```

For custom controls, the appropriate assertion may instead inspect:

```text
visible selected text
aria-selected
checked state
button label
```

depending on the component implementation.

---

# 39. `aria-selected`

Accessible listbox options may use:

```text
aria-selected="true"
```

This can represent the selected option.

Example conceptual state:

```text
India
aria-selected="true"
```

This can be useful for accessibility-oriented assertions.

---

# 40. `aria-expanded`

A dropdown trigger may expose:

```text
aria-expanded="false"
```

Closed:

```text
aria-expanded="false"
```

Open:

```text
aria-expanded="true"
```

This can be a useful state signal.

---

# 41. Dropdown State Model

```text
Closed
  ↓
Open
  ↓
Loading
  ↓
Options Available
  ↓
Selection
  ↓
Closed
```

Not every dropdown follows exactly this sequence, but this is a useful mental model.

---

# 42. Dropdown Testing Matrix

A production dropdown should often be tested for:

```text
Open
Close
Select
Search
No Results
Loading
Keyboard Navigation
Escape
Disabled State
Default Value
Validation
Multiple Selection
Dependent Data
```

---

# 43. Native Dropdown Testing Matrix

For native `<select>`:

```text
Default Value
Select by Value
Select by Label
Multiple Select
Disabled State
Options
Selected State
Validation
```

---

# 44. Custom Dropdown Testing Matrix

For custom dropdowns:

```text
Open
Close
Visible Options
Option Selection
Keyboard Navigation
Search
Loading
No Results
Disabled State
Selected State
Outside Click
Escape
```

---

# 45. Outside Click

Many custom dropdowns close when clicking outside.

Example:

```text
Dropdown Open
     ↓
Click elsewhere
     ↓
Dropdown closes
```

This is an important behavioral test.

---

# 46. Outside Click Test

Conceptually:

```ts
await trigger.click();

await page.getByRole('heading', {
  name: 'Dashboard'
}).click();

await expect(listbox).toBeHidden();
```

The exact locator depends on the application.

---

# 47. Disabled Dropdown

A disabled dropdown should not open.

Example:

```text
Status
[ Select Status ▼ ]  disabled
```

Test:

```text
Verify disabled
```

rather than forcing it open.

---

# 48. Default Selection

Example:

```text
Country
[ India ▼ ]
```

The test may need to verify:

```text
Default = India
```

---

# 49. Validation

A required dropdown may produce:

```text
Country
[ Select Country ▼ ]

Country is required
```

The test should validate the error state.

---

# 50. Enterprise Dropdown Strategy

The first decision should always be:

```text
What type of dropdown is this?
```

Decision tree:

```text
Dropdown
   │
   ├── Native <select>?
   │       ↓
   │   selectOption()
   │
   └── Custom?
           ↓
       What semantics?
           │
      ┌────┼─────────┐
      ▼    ▼         ▼
   Combo  Listbox   Menu
      │    │         │
      └────┼─────────┘
           ▼
     Semantic Locators
           ↓
       User Actions
```

---

# 51. Enterprise Dropdown Decision Tree

```text
Can the control be represented by <select>?
              │
          ┌───┴───┐
          ▼       ▼
         YES      NO
          │       │
          ▼       ▼
   selectOption   Inspect semantics
                  │
                  ▼
          Combobox/Listbox/Menu
                  │
                  ▼
             Open control
                  │
                  ▼
             Locate option
                  │
                  ▼
               Select
```

---

# 52. Preferred Locator Order

For dropdowns, prefer:

```text
getByLabel()
getByRole()
getByTestId()
filter()
```

when appropriate.

Avoid:

```text
nth()
```

unless position is genuinely meaningful.

---

# 53. Native Dropdown Example

```ts
test('select country', async ({ page }) => {
  const country = page.getByLabel('Country');

  await country.selectOption({
    label: 'India'
  });

  await expect(country).toHaveValue('IN');
});
```

---

# 54. Custom Dropdown Example

```ts
test('select country from custom dropdown', async ({ page }) => {
  await page.getByRole('button', {
    name: 'Select country'
  }).click();

  const option = page.getByRole('option', {
    name: 'India'
  });

  await expect(option).toBeVisible();

  await option.click();
});
```

---

# 55. Searchable Dropdown Example

```ts
test('select searchable country', async ({ page }) => {
  const combo = page.getByRole('combobox', {
    name: 'Country'
  });

  await combo.fill('Ind');

  const option = page.getByRole('option', {
    name: 'India'
  });

  await expect(option).toBeVisible();
  await option.click();
});
```

---

# 56. Dependent Dropdown Example

```ts
test('select state after country', async ({ page }) => {
  const country = page.getByRole('combobox', {
    name: 'Country'
  });

  const state = page.getByRole('combobox', {
    name: 'State'
  });

  await country.selectOption('IN');

  await expect(state).toBeEnabled();

  await state.selectOption({
    label: 'Madhya Pradesh'
  });
});
```

If these are custom controls, use the component's actual interaction model rather than `selectOption()`.

---

# 57. Multi-Select Example

For a native multi-select:

```ts
await skills.selectOption([
  'java',
  'typescript'
]);
```

Then verify the expected selected values.

---

# 58. Keyboard Dropdown Example

If keyboard behavior is the requirement:

```ts
const combo = page.getByRole('combobox', {
  name: 'Country'
});

await combo.focus();
await combo.press('ArrowDown');
await combo.press('Enter');
```

The exact sequence depends on the component.

---

# 59. Avoid Fixed Sleeps

Bad:

```ts
await page.waitForTimeout(2000);
```

Better:

```ts
await expect(
  page.getByRole('option', { name: 'India' })
).toBeVisible();
```

---

# 60. Avoid Global Option Locators

Bad:

```ts
await page.getByRole('option', {
  name: 'India'
}).click();
```

if the page contains multiple dropdowns with the same option.

Better:

```text
Scope according to dropdown semantics
```

---

# 61. Avoid `nth()` for Dropdowns

Bad:

```ts
await page.getByRole('option').nth(3).click();
```

unless:

```text
"Select the fourth option"
```

is explicitly the requirement.

Better:

```text
Select option by business label
```

---

# 62. Dropdown Component Object

For enterprise frameworks, create reusable component abstractions.

Example:

```ts
export class DropdownComponent {
  constructor(
    private readonly page: Page,
    private readonly trigger: Locator
  ) {}

  async open() {
    await this.trigger.click();
  }

  async selectOption(name: string) {
    await this.open();

    await this.page.getByRole('option', {
      name
    }).click();
  }
}
```

---

# Important Improvement

For real enterprise applications, the component should often be scoped more carefully so that multiple dropdowns do not share a global option locator.

Conceptually:

```text
Dropdown Root
   ↓
Trigger
   ↓
Associated Listbox
   ↓
Option
```

---

# 63. Enterprise Component Architecture

```text
Page Object
    │
    ├── CountryDropdown
    │
    ├── StateDropdown
    │
    └── CityDropdown
```

Each component manages:

```text
Open
Search
Select
Validate
Close
```

---

# 64. Dropdown Failure Debugging

When a dropdown test fails, ask:

```text
Is it native or custom?
       ↓
Did it open?
       ↓
Is the option rendered?
       ↓
Is it loading?
       ↓
Is the option hidden?
       ↓
Is there a portal?
       ↓
Are multiple options matching?
       ↓
Is virtualization involved?
       ↓
Is the option disabled?
       ↓
Did selection actually change state?
```

---

# 65. Enterprise Best Practices

1. Identify the dropdown implementation first.
2. Use `selectOption()` for native `<select>`.
3. Prefer value or label over index.
4. Use semantic roles for custom controls.
5. Use `combobox` for appropriate searchable controls.
6. Use `listbox` and `option` semantics when exposed.
7. Scope options to the correct dropdown.
8. Handle async loading explicitly.
9. Test no-results states.
10. Test disabled states.
11. Test keyboard behavior when required.
12. Test Escape behavior.
13. Test outside-click behavior.
14. Avoid arbitrary sleeps.
15. Avoid `nth()` unless position is business-relevant.
16. Prefer search for virtualized large lists.
17. Verify the final selected state.
18. Encapsulate reusable dropdown behavior.
19. Account for portal-based overlays.
20. Treat dropdowns as stateful components, not just lists of text.

---

# Common Beginner Mistakes

## Mistake 1

Using:

```text
selectOption()
```

on a custom dropdown.

---

## Mistake 2

Using:

```text
click()
```

on a native `<select>` when `selectOption()` is available.

---

## Mistake 3

Selecting by index unnecessarily.

---

## Mistake 4

Using:

```text
waitForTimeout()
```

after every search.

---

## Mistake 5

Using a global option locator.

---

## Mistake 6

Ignoring loading states.

---

## Mistake 7

Ignoring no-results states.

---

## Mistake 8

Ignoring disabled dependent dropdowns.

---

## Mistake 9

Assuming the option is a DOM child of the trigger.

---

## Mistake 10

Trying to scroll through thousands of virtualized options manually instead of using search.

---

# Professional Tips

When you see a dropdown, do not immediately write:

```ts
selectOption()
```

First inspect:

```text
HTML structure
ARIA roles
Native select?
Custom component?
Combobox?
Listbox?
Portal?
Autocomplete?
```

Then choose the action.

---

# Real Interview Discussion

## Junior-Level

### Q1. How do you handle a native dropdown?

**Answer:**

Use `selectOption()`.

---

### Q2. Can `selectOption()` work with every dropdown?

**Answer:**

No. It is intended for native HTML `<select>` elements. Custom dropdowns require their actual interaction model to be automated.

---

### Q3. How do you select by visible text?

**Answer:**

For a native select, use the option label through `selectOption({ label: '...' })`. For a custom dropdown, locate the corresponding option using its semantic locator.

---

# Mid-Level

### Q4. How do you automate an autocomplete?

**Answer:**

Locate the combobox, enter the search text, wait for the desired option to become available, and select it using a semantic locator.

---

### Q5. How do you handle dependent dropdowns?

**Answer:**

Select the parent value, wait for the child dropdown to reach the expected state, then select the child value. Avoid fixed sleeps.

---

### Q6. How do you handle a virtualized dropdown?

**Answer:**

Prefer the component's search capability if available. Otherwise interact with the rendered portion and scroll as required while synchronizing with newly rendered options.

---

# Senior-Level

### Q7. How would you distinguish a native dropdown from a custom dropdown?

**Answer:**

I would inspect the DOM and accessibility semantics. A native dropdown is represented by `<select>`, while custom controls are commonly built from buttons, inputs, listboxes, menus, or other elements.

---

### Q8. Why should dropdown options be scoped?

**Answer:**

Because multiple dropdowns can contain identical option text. Scoping reduces ambiguity and makes the locator represent the correct business component.

---

### Q9. How would you test an asynchronous autocomplete?

**Answer:**

I would enter the search term, synchronize against the actual option or loading state, select the intended option, and verify the final selected state. I would avoid fixed sleeps.

---

# Lead-Level

### Q10. How would you build a reusable dropdown component?

**Answer:**

I would encapsulate the trigger, open/close behavior, option resolution, search behavior, selection, and state validation. The implementation would account for whether the dropdown is native, custom, portal-based, searchable, or virtualized.

---

### Q11. What dropdown behavior belongs in component objects rather than individual tests?

**Answer:**

Reusable mechanics such as opening the dropdown, finding options, searching, selecting, handling common loading behavior, and validating selection state should generally be encapsulated. Tests should express business intent.

---

# Architect-Level

### Q12. What is the enterprise strategy for dropdown automation?

**Strong Answer:**

First classify the control by implementation and accessibility semantics. Use `selectOption()` for native selects. For custom controls, prefer semantic roles such as combobox, listbox, and option, with appropriate scoping. Synchronize against actual UI states rather than fixed delays, encapsulate reusable behavior in component abstractions, and treat positional selection as an exception.

---

# Knowledge Check

1. What is a native dropdown?
2. What is `selectOption()`?
3. How do you select by value?
4. How do you select by label?
5. Why is index-based selection less stable?
6. How do you select multiple native options?
7. What is a custom dropdown?
8. What is a combobox?
9. What is a listbox?
10. What is an option?
11. What is an autocomplete?
12. How do you automate a searchable dropdown?
13. How do you handle dynamic options?
14. How do you handle loading states?
15. How do you test no-results states?
16. What is a dependent dropdown?
17. What is a cascading dropdown?
18. How do you synchronize dependent dropdowns?
19. What is a virtualized dropdown?
20. Why can virtualized options be missing from the DOM?
21. Why is search preferable for large lists?
22. What is a portal-based dropdown?
23. Why might the option not be a DOM child of the trigger?
24. How do `aria-selected` and `aria-expanded` help?
25. Why should dropdown options be scoped?
26. Why should you avoid unnecessary `nth()`?
27. How would you test keyboard selection?
28. How would you test Escape behavior?
29. How would you test outside-click behavior?
30. What is the enterprise dropdown strategy?

---

# Step Summary

The most important lesson is:

> **Automate the dropdown according to what it actually is, not what it visually looks like.**

The decision process is:

```text
Dropdown
   ↓
Inspect Implementation
   │
   ├── Native <select>
   │       ↓
   │   selectOption()
   │
   └── Custom
           ↓
      Inspect Semantics
           │
      ┌────┼─────────┐
      ▼    ▼         ▼
   Combobox Listbox Menu
      │      │       │
      └──────┼───────┘
             ▼
       Semantic Locator
             ↓
         User Action
             ↓
       State Assertion
```

For dynamic dropdowns:

```text
Open
 ↓
Loading
 ↓
Options
 ↓
Selection
 ↓
Selected State
```

For dependent dropdowns:

```text
Parent
 ↓
State Change
 ↓
Child Enabled
 ↓
Child Options
 ↓
Child Selection
```

For large virtualized lists:

```text
Search
 ↓
Render Desired Option
 ↓
Select
```

rather than:

```text
Scroll thousands of items
```

---

# Progress Milestone

✅ **Step 69 completed.**

You now understand:

- Native `<select>`
- `selectOption()`
- Selection by value
- Selection by label
- Selection by index
- Multi-select
- Custom dropdowns
- Comboboxes
- Listboxes
- Options
- Autocomplete
- Searchable dropdowns
- Dynamic options
- Async loading
- Dependent dropdowns
- Cascading dropdowns
- Disabled states
- Loading states
- Empty states
- No-results states
- Virtualized lists
- Portal-based dropdowns
- Hidden options
- Keyboard selection
- `ArrowDown`
- `ArrowUp`
- `Enter`
- `Escape`
- `aria-selected`
- `aria-expanded`
- Dropdown scoping
- Dropdown filtering
- Component abstractions
- Enterprise dropdown governance

Your dropdown automation architecture is now:

```text
                    DROPDOWN
                       │
                       ▼
              Identify Implementation
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
       Native                    Custom
          │                         │
          ▼                         ▼
  selectOption()             Inspect Semantics
                                    │
                      ┌─────────────┼─────────────┐
                      ▼             ▼             ▼
                  Combobox       Listbox        Menu
                      │             │             │
                      └─────────────┼─────────────┘
                                    ▼
                             Semantic Locator
                                    │
                                    ▼
                              User Action
                                    │
                                    ▼
                            Business State
                                    │
                                    ▼
                                Assertion
```

---

# Next Step

# Step 70 — Mastering Playwright Checkboxes, Radio Buttons, Toggle Switches & Selection Controls: `check()`, `uncheck()`, `isChecked()`, `setChecked()`, Radio Groups, Toggle Buttons, ARIA Switches, Custom Checkboxes, Indeterminate State, Disabled Controls, Dynamic State, Forms, Accessibility Semantics, and Enterprise Selection Strategy

The next lesson moves from dropdown selection to another major form-control category:

```text
Checkbox
Radio Button
Toggle Switch
Switch
Custom Checkbox
```

You will learn how to distinguish:

```text
Checkbox
vs
Radio
vs
Toggle
vs
Button
```

and how to test:

```text
Checked
Unchecked
Indeterminate
Disabled
Enabled
Selected
Deselected
```

while building reliable form automation without depending on:

```text
CSS classes
XPath
DOM position
nth()
```
# Playwright with TypeScript Mastery Bootcamp

# Stage 31 — Advanced Form & Component Interaction

# Step 72 — Mastering Playwright Date Pickers & Calendar Components

---

# Objective

In the previous lesson, you learned how to automate:

```text
Text Inputs
Textarea
Password Fields
Email Inputs
Number Inputs
Search Inputs
URL Inputs
Contenteditable
fill()
pressSequentially()
press()
Keyboard Interaction
Dynamic Forms
Validation
Masked Inputs
Debounced Inputs
Form Submission
Form Reset
```

Now we move to one of the most challenging form controls in real-world web automation:

```text
Date Pickers
Calendar Components
```

Date selection appears in:

```text
Booking Applications
Banking Applications
Healthcare Systems
Travel Applications
E-commerce
HR Systems
Attendance Systems
Insurance Applications
Financial Applications
Enterprise Administration
```

In this lesson, you will deeply understand:

* Native date inputs
* Custom date pickers
* Calendar widgets
* Month navigation
* Year navigation
* Previous month
* Next month
* Date selection
* Current date
* Past dates
* Future dates
* Disabled dates
* Minimum dates
* Maximum dates
* Date ranges
* Start dates
* End dates
* Booking calendars
* Multi-month calendars
* Dynamic calendars
* Date formatting
* Locale differences
* Timezone considerations
* Date validation
* Boundary dates
* Weekend restrictions
* Holiday restrictions
* Calendar accessibility
* Keyboard navigation
* Enterprise date automation strategy

The most important principle is:

> **Do not automate a date picker until you understand what type of date control you are dealing with.**

---

# 1. Why Date Pickers Matter

Dates are deceptively complicated.

A field may visually display:

```text
12/08/2026
```

but the application may internally represent it as:

```text
2026-08-12
```

Another application may display:

```text
12 Aug 2026
```

while storing:

```text
2026-08-12T00:00:00Z
```

Another application may interpret:

```text
08/12/2026
```

as:

```text
August 12, 2026
```

while another locale interprets it as:

```text
December 8, 2026
```

Therefore date automation requires more than simply clicking a number.

---

# 2. Date Picker Architecture

A useful mental model is:

```text
                         Date Control

                              │

             ┌────────────────┼────────────────┐

             ▼                ▼                ▼

          Native           Custom           Range Picker

             │                │                │

             ▼                ▼                ▼

       Browser UI        Application UI    Start + End

                              │

                              ▼

                         Calendar State

                              │

                              ▼

                    Date Selection Logic
```

The automation strategy depends heavily on which branch you are dealing with.

---

# 3. What Is a Date Picker?

A date picker is a user-interface control that allows users to select a date.

A simple example:

```text
┌─────────────────────────────┐
│ 12/08/2026              📅 │
└─────────────────────────────┘
```

The user may either:

```text
Type a date
```

or:

```text
Open Calendar
     ↓
Navigate
     ↓
Select Date
```

A date picker therefore combines:

```text
Input
+
Calendar
+
Navigation
+
Selection
+
Validation
```

---

# 4. Native Date Input

A native date input may be:

```html
<input type="date">
```

The browser provides date-specific behavior.

Conceptually:

```text
Date Input

      ↓

Browser Date Control

      ↓

Selected Date
```

The visual representation can vary by browser and operating system.

This is important.

The test should generally target the semantic control rather than relying on browser-specific visual details.

---

# 5. Native vs Custom Date Picker

This distinction is fundamental.

### Native

```text
<input type="date">
```

The browser provides the date UI.

### Custom

The application may build a calendar using:

```text
button
div
table
grid
ARIA widgets
```

For example:

```text
August 2026

Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 31
```

The automation strategies are very different.

---

# 6. Why Classification Comes First

Before writing automation, inspect:

```text
What element opens the calendar?

What role does it expose?

Is the input native?

Is the calendar custom?

Are dates buttons?

Are dates grid cells?

Are dates links?

Is ARIA used?

How is the selected date represented?

How are disabled dates represented?
```

Only after understanding this should you design the locator strategy.

---

# 7. Date Picker Mental Model

Think of a calendar as a state machine:

```text
Current Month
      ↓
Navigate
      ↓
Target Month
      ↓
Identify Target Date
      ↓
Select Date
      ↓
Input Updated
      ↓
Application State Updated
```

For example:

```text
August 2026

      ↓
Next Month

      ↓

September 2026

      ↓
Select 15

      ↓

15 September 2026
```

---

# 8. Calendar State

A calendar may maintain:

```text
Current Month
Current Year
Selected Date
Focused Date
Today
Minimum Date
Maximum Date
Disabled Dates
Hovered Date
Range Start
Range End
```

This is why date pickers can become complex.

---

# 9. Current Date

Many applications have a concept of:

```text
Today
```

For example:

```text
Today = August 20, 2026
```

The calendar may visually indicate:

```text
Today
```

while another date may be:

```text
Selected
```

These are not necessarily the same.

---

# 10. Today vs Selected Date

Consider:

```text
Today = August 20

Selected = August 15
```

The calendar may show:

```text
Today → August 20
Selected → August 15
```

Therefore:

```text
Today ≠ Selected
```

A test should distinguish between these concepts.

---

# 11. Selecting a Date

Suppose the requirement is:

> Select August 25, 2026.

The automation strategy should be:

```text
Open Calendar
      ↓
Determine Current Month
      ↓
Navigate if Necessary
      ↓
Identify August 25
      ↓
Select
      ↓
Verify Selected Date
```

Do not assume the calendar is already displaying the correct month.

---

# 12. Month Navigation

Most custom calendars provide:

```text
Previous Month
Next Month
```

Conceptually:

```text
August 2026

      ↓ Next

September 2026
```

and:

```text
August 2026

      ↓ Previous

July 2026
```

A test may need to navigate multiple months.

---

# 13. Month Navigation Architecture

```text
                    Calendar

                       │

              Current Month/Year

                       │

              ┌────────┴────────┐

              ▼                 ▼

        Previous Month       Next Month

              │                 │

              └────────┬────────┘

                       ↓

                 Updated Calendar

                       ↓

                 Target Date
```

---

# 14. Why Fixed Number of Clicks Is Weak

Suppose the target is:

```text
December 25, 2026
```

and the calendar currently displays:

```text
August 2026
```

A beginner may write:

```text
Click Next
Click Next
Click Next
Click Next
```

This is fragile because the initial calendar state may change.

A stronger strategy is:

```text
Read Current Month

      ↓

Compare With Target Month

      ↓

Calculate Navigation

      ↓

Navigate Until Target Month

      ↓

Select Date
```

This is state-driven automation.

---

# 15. Dynamic Month Navigation

Mental model:

```text
Current:
August 2026

Target:
December 2026

Difference:
4 months

        ↓

Next Month × 4

        ↓

December 2026
```

For a previous date:

```text
Current:
August 2026

Target:
May 2026

Difference:
3 months

        ↓

Previous Month × 3
```

The important principle is:

> Navigation should depend on calendar state, not hard-coded assumptions.

---

# 16. Year Navigation

Some calendars provide:

```text
Previous Year
Next Year
```

Others allow:

```text
Month Selector
Year Selector
```

Others open a separate:

```text
Year Picker
```

For example:

```text
2026

2027
2028
2029
2030
```

The automation strategy must follow the actual component behavior.

---

# 17. Month-Year Selector

Some enterprise applications expose:

```text
August 2026
```

as a clickable header.

Clicking it may produce:

```text
Months

January
February
March
...
December
```

Then:

```text
Select December

        ↓

Select Year 2026
```

This may be faster than clicking Next Month repeatedly.

---

# 18. Calendar Navigation Strategies

There are several possible strategies:

```text
Strategy 1
Previous / Next buttons

Strategy 2
Month selector

Strategy 3
Year selector

Strategy 4
Direct input

Strategy 5
Keyboard navigation
```

Choose the most stable strategy supported by the application.

---

# 19. Date Selection by Accessible Name

Well-designed calendars often expose accessible names.

For example:

```text
button:
"August 20, 2026"
```

This is excellent for semantic automation.

Conceptually:

```text
Calendar
   ↓
Date Button
   ↓
Accessible Name
   ↓
Target Date
```

A date locator should communicate:

> Select August 25, 2026.

rather than:

> Click the 25th element.

---

# 20. Avoid `nth()` for Calendar Dates

Bad strategy:

```text
Date #25
```

or:

```text
nth(24)
```

Why?

Because calendars can contain:

```text
Previous Month Dates
Current Month Dates
Next Month Dates
```

For example:

```text
July 26
July 27
July 28
July 29
July 30
July 31
August 1
```

The number `1` may not uniquely identify the target date.

---

# 21. Duplicate Day Numbers

A calendar can contain:

```text
July 25
August 25
September 25
```

depending on the calendar design.

Therefore:

```text
25
```

is not necessarily enough.

The test should identify:

```text
25 August 2026
```

rather than simply:

```text
25
```

---

# 22. Calendar Scope

A strong strategy is:

```text
Calendar
   ↓
Target Month
   ↓
Target Date
```

instead of:

```text
Entire Page
   ↓
Date Number
```

This follows the same locator-scoping principle you learned earlier.

---

# 23. Multi-Month Calendars

Some booking systems display:

```text
August 2026        September 2026

Su Mo Tu ...       Su Mo Tu ...
...
```

This is common in:

```text
Travel
Hotels
Flight Booking
Car Rental
Vacation Booking
```

The test must identify the correct calendar panel.

---

# 24. Multi-Month Architecture

```text
              Date Range Picker

                    │

          ┌─────────┴─────────┐

          ▼                   ▼

    August 2026          September 2026

          │                   │

          ▼                   ▼

       Start Date          End Date
```

A test should avoid accidentally selecting the same day number from the wrong month.

---

# 25. Date Range Picker

Range selection usually follows:

```text
Start Date
     ↓
End Date
```

For example:

```text
August 20
     ↓
August 25
```

The resulting range is:

```text
20 August → 25 August
```

---

# 26. Range Selection State

A range picker may have states:

```text
No Selection

      ↓

Start Selected

      ↓

End Selected

      ↓

Range Complete
```

Mental model:

```text
None
 ↓
Start
 ↓
Start + End
```

This is different from selecting a single date.

---

# 27. Date Range Validation

A test should consider:

```text
Start < End
Start = End
Start > End
```

depending on business rules.

For example:

```text
Check-in:
August 25

Check-out:
August 20
```

may be invalid.

Expected behavior may be:

```text
Validation Error
```

or:

```text
End Date Cannot Be Before Start Date
```

---

# 28. Same-Day Range

Some applications allow:

```text
Start = August 20
End = August 20
```

Others require:

```text
End > Start
```

Do not assume.

The business requirement determines the expected behavior.

---

# 29. Disabled Dates

Calendars often disable certain dates.

Examples:

```text
Past Dates
Fully Booked Dates
Public Holidays
Unavailable Dates
Maintenance Dates
Blackout Dates
```

A date may appear visually:

```text
20
```

but be non-selectable.

---

# 30. Disabled Date Architecture

```text
Calendar

 ├── Available Date
 │      ↓
 │   Selectable
 │
 └── Disabled Date
        ↓
     Not Selectable
```

The test should validate both:

```text
Disabled State
```

and:

```text
Business Reason
```

when relevant.

---

# 31. Past Dates

Booking systems frequently prohibit past dates.

For example:

```text
Today = August 20
```

Then:

```text
August 19 → Disabled
August 20 → Available
August 21 → Available
```

A test can verify:

```text
Past Date
   ↓
Disabled
```

and:

```text
Future Date
   ↓
Enabled
```

according to the business rule.

---

# 32. Future Date Restrictions

Some systems permit bookings only within a future window.

Example:

```text
Booking Window:
Next 90 Days
```

Therefore:

```text
Day 1     → Enabled
Day 30    → Enabled
Day 90    → Enabled
Day 91    → Disabled
```

This is an excellent boundary-testing scenario.

---

# 33. Minimum Date

A form may define:

```text
Minimum Date = August 20, 2026
```

Then:

```text
August 19 → Invalid
August 20 → Valid
August 21 → Valid
```

The important tests are:

```text
Minimum - 1
Minimum
Minimum + 1
```

---

# 34. Maximum Date

Suppose:

```text
Maximum Date = December 31, 2026
```

Test:

```text
December 30 → Valid
December 31 → Valid
January 1   → Invalid
```

This follows standard boundary-testing principles.

---

# 35. Date Boundary Testing

A general model is:

```text
            Invalid
               ↓
         Minimum - 1

               ↓

         Minimum Date

               ↓

          Valid Range

               ↓

         Maximum Date

               ↓

         Maximum + 1

               ↓
            Invalid
```

Boundary testing is particularly valuable for date controls.

---

# 36. Weekend Restrictions

Some applications allow only weekdays.

Example:

```text
Monday → Available
Tuesday → Available
Wednesday → Available
Thursday → Available
Friday → Available
Saturday → Disabled
Sunday → Disabled
```

A test should verify the rule.

But remember:

> Do not assume weekends are always invalid.

Different businesses have different rules.

---

# 37. Holiday Restrictions

Enterprise systems may disable holidays.

Examples:

```text
Company Holidays
Public Holidays
Bank Holidays
Maintenance Days
```

A calendar may therefore combine:

```text
Date Rules
+
Business Calendar
```

A test must use controlled and maintainable test data when validating such rules.

---

# 38. Date Formatting

Different systems display dates differently.

Examples:

```text
20/08/2026
08/20/2026
20-Aug-2026
Aug 20, 2026
2026-08-20
```

The automation should distinguish:

```text
Display Format
```

from:

```text
Internal Date Representation
```

---

# 39. Locale Differences

A major internationalization problem is:

```text
08/12/2026
```

This can mean:

```text
August 12
```

or:

```text
December 8
```

depending on locale.

Therefore tests should not blindly assume a date format.

The test environment may use:

```text
en-US
en-GB
en-IN
```

or another locale.

---

# 40. Locale-Aware Date Testing

A robust strategy is:

```text
Environment Locale
       ↓
Expected Date Format
       ↓
Input / Calendar
       ↓
Verification
```

The expected value should come from controlled test configuration or business rules rather than accidental machine locale.

---

# 41. Timezone Considerations

Dates become more complicated when timezones are involved.

Consider:

```text
UTC:
August 20, 23:30
```

In another timezone it may already be:

```text
August 21
```

Therefore:

```text
Date
```

may change depending on:

```text
Timezone
```

This is especially important for:

```text
Booking
Appointments
Transactions
Reports
Notifications
Scheduled Jobs
```

---

# 42. Date vs DateTime

Do not confuse:

```text
Date
```

with:

```text
Date + Time + Timezone
```

For example:

```text
August 20, 2026
```

is different from:

```text
August 20, 2026 23:30 UTC
```

An enterprise test must understand what the application actually stores and displays.

---

# 43. Calendar Accessibility

Accessible calendars may expose semantics such as:

```text
role="grid"
role="gridcell"
role="button"
aria-label
aria-selected
aria-disabled
```

This provides useful information for automation.

A well-designed calendar may communicate:

```text
Date:
August 20, 2026

State:
Selected
```

or:

```text
Date:
August 19, 2026

State:
Disabled
```

---

# 44. `aria-selected`

Some calendar implementations represent the selected date with:

```text
aria-selected="true"
```

Conceptually:

```text
Date Cell

     ↓

aria-selected

     ↓

true / false
```

This can provide semantic state information.

Do not blindly inspect attributes, however.

Use the appropriate semantic assertion strategy for the component.

---

# 45. `aria-disabled`

Disabled dates may expose:

```text
aria-disabled="true"
```

This allows the application to communicate disabled state to assistive technologies.

A robust test can use semantic state rather than depending only on CSS styling.

---

# 46. Keyboard Calendar Navigation

Accessible calendars may support:

```text
ArrowUp
ArrowDown
ArrowLeft
ArrowRight
Home
End
PageUp
PageDown
Enter
Space
```

The exact behavior depends on the component.

Keyboard navigation can be important for:

```text
Accessibility Testing
Keyboard-Only Users
Enterprise Compliance
```

---

# 47. Calendar Keyboard Mental Model

```text
Focus Date

    ↓

ArrowRight

    ↓

Next Day

    ↓

ArrowDown

    ↓

Next Week

    ↓

Enter

    ↓

Select
```

The actual keyboard behavior should be validated against the application's intended accessibility behavior.

---

# 48. Date Picker and Focus

A date picker may have separate concepts:

```text
Focused Date
Selected Date
Today
```

These can be different.

Example:

```text
Today:
August 20

Focused:
August 25

Selected:
August 15
```

Therefore a test should not assume:

```text
Focused = Selected
```

---

# 49. Current Date Navigation

Many calendars provide:

```text
Today
```

button.

Example:

```text
Today
```

Clicking it may navigate to the current month or select today's date.

These are not necessarily the same operation.

A test should determine whether:

```text
Today
```

means:

```text
Navigate to Today
```

or:

```text
Select Today
```

or both.

---

# 50. Calendar Year Boundaries

Month navigation becomes more interesting across years.

Example:

```text
December 2026
```

Next month becomes:

```text
January 2027
```

The test must verify both:

```text
Month
```

and:

```text
Year
```

This prevents false positives.

---

# 51. Previous-Year Boundary

Similarly:

```text
January 2027
```

Previous month:

```text
December 2026
```

The test should not only check:

```text
December
```

but:

```text
December 2026
```

---

# 52. Leap Years

Date testing must account for leap years where relevant.

For example:

```text
February 2028
```

contains:

```text
February 29
```

while:

```text
February 2027
```

does not.

A strong date test strategy may include:

```text
Leap Year
Non-Leap Year
February 28
February 29
March 1
```

when the business domain makes this important.

---

# 53. Month Length

Months have different numbers of days:

```text
January → 31
February → 28/29
April → 30
May → 31
```

This can expose calendar rendering bugs.

Important boundary scenarios include:

```text
Last Day of Month
First Day of Next Month
```

---

# 54. Cross-Month Selection

A date range may cross months:

```text
August 28
     ↓
September 5
```

The calendar must correctly represent:

```text
Start
Range
End
```

This is an important enterprise booking scenario.

---

# 55. Cross-Year Selection

Similarly:

```text
December 28, 2026
        ↓
January 5, 2027
```

This requires correct handling of:

```text
Year transition
Month transition
Range transition
```

---

# 56. Date Picker Workflow

A robust single-date workflow is:

```text
Open Form

   ↓

Open Date Picker

   ↓

Inspect Current Calendar State

   ↓

Determine Target Month/Year

   ↓

Navigate

   ↓

Locate Target Date

   ↓

Select

   ↓

Verify Selected State

   ↓

Verify Input Value

   ↓

Continue Business Flow
```

---

# 57. Date Range Workflow

```text
Open Date Picker

      ↓

Select Start Date

      ↓

Calendar Range State

      ↓

Select End Date

      ↓

Range Complete

      ↓

Verify Start

      ↓

Verify End

      ↓

Verify Displayed Range

      ↓

Continue
```

---

# 58. Native Date Input Strategy

For a native date input, the automation strategy may be significantly simpler than interacting with a custom calendar.

Conceptually:

```text
Locate Date Input

      ↓

Set Desired Date

      ↓

Verify Current Value
```

The exact interaction should respect the input's supported value format and the application's requirements.

Do not assume the displayed localized format is the same as the underlying input value.

---

# 59. Custom Date Picker Strategy

For a custom picker:

```text
Locate Date Field

      ↓

Open Calendar

      ↓

Identify Calendar Container

      ↓

Read Month/Year

      ↓

Navigate

      ↓

Locate Date

      ↓

Select

      ↓

Verify
```

This requires more state awareness.

---

# 60. Date Picker Component Abstraction

A reusable component can conceptually expose:

```text
DatePicker

    ↓

selectDate()

selectToday()

nextMonth()

previousMonth()

selectRange()

getSelectedDate()

isDateDisabled()
```

This hides low-level calendar interactions.

For example:

```text
Test

 ↓

DatePicker.selectDate()

 ↓

Calendar Locators

 ↓

Playwright
```

---

# 61. Business-Level Date API

An even stronger abstraction is:

```text
BookingForm

    ↓

setCheckInDate()

setCheckOutDate()

submit()
```

The test should ideally read like the business requirement:

```text
Set check-in date to August 20.

Set check-out date to August 25.

Submit booking.
```

The calendar mechanics remain inside the component.

---

# 62. Enterprise Calendar Architecture

```text
                         Test

                          ↓

                    Business Workflow

                          ↓

                    DatePicker Component

                          ↓

              ┌───────────┼───────────┐

              ▼           ▼           ▼

          Navigation   Date State   Selection

              │           │           │

              └───────────┼───────────┘

                          ↓

                       Playwright

                          ↓

                        Browser

                          ↓

                     Application
```

---

# 63. Date Test Data

Date automation should avoid uncontrolled hard-coded dates when the test depends on the current date.

For example:

```text
Today + 5 days
```

may be more meaningful than:

```text
August 25, 2026
```

for certain scenarios.

However, deterministic fixed dates may be better for historical regression scenarios.

The correct choice depends on the test purpose.

---

# 64. Relative Date Test Data

Examples:

```text
Today
Today + 1
Today + 7
Today + 30
Today - 1
End of Month
Beginning of Month
```

This can make date rules easier to express.

---

# 65. Deterministic Date Testing

A major enterprise concern is:

```text
Today changes
```

If a test depends on:

```text
Today
```

the result may change every day.

Therefore, enterprise frameworks may use:

```text
Controlled Clock
Fixed Test Data
Mocked Time
Environment Configuration
```

when appropriate.

The goal is:

```text
Predictable Test
```

rather than:

```text
Test Result Depends on Execution Date
```

---

# 66. Date Picker and API Integration

Some date pickers obtain availability from an API.

Example:

```text
Select Destination

      ↓

API Request

      ↓

Available Dates

      ↓

Calendar Updates

      ↓

User Selects Date
```

A date may be disabled because:

```text
Backend says unavailable
```

This is why enterprise automation sometimes requires understanding both:

```text
UI
+
API
```

---

# 67. Network-Driven Calendars

Architecture:

```text
Calendar Open

      ↓

API Request

      ↓

Availability Data

      ↓

Calendar Rendering

      ↓

Enabled / Disabled Dates

      ↓

Selection
```

A test failure could therefore originate from:

```text
UI
API
Test Data
Environment
Network
```

not necessarily the calendar itself.

---

# 68. Calendar Test Isolation

If availability depends on external data, stable automation may require controlled test data.

For example:

```text
Test Hotel
    ↓
Known Availability
    ↓
Known Dates
```

This is preferable to depending on unpredictable real-world inventory.

---

# 69. Date Picker Synchronization

Do not assume:

```text
Click Next Month
↓
Immediately Select Date
```

The application may need time to:

```text
Render
Fetch Data
Update State
Animate
```

Use meaningful synchronization.

Mental model:

```text
Navigation

   ↓

Calendar State Updated

   ↓

Target Month Visible

   ↓

Date Available

   ↓

Select
```

---

# 70. Date Picker Decision Tree

When encountering a date picker:

```text
Is it native?

      │
      ├── YES
      │    ↓
      │  Use native input strategy
      │
      └── NO
           ↓
       Custom Calendar
           ↓
       Identify semantics
           ↓
       Determine state
           ↓
       Navigate
           ↓
       Select
           ↓
       Verify
```

Then ask:

```text
Single Date?

Range?

Disabled Dates?

Dynamic Availability?

Locale?

Timezone?

Accessibility?

Keyboard Support?
```

---

# 71. Best Practices

1. Determine whether the date control is native or custom.

2. Prefer semantic locators.

3. Scope date selection to the correct calendar.

4. Identify month and year explicitly.

5. Avoid positional date selection.

6. Do not rely on day number alone.

7. Test disabled dates.

8. Test minimum and maximum dates.

9. Test past and future dates.

10. Test month/year transitions.

11. Test leap years where relevant.

12. Test date ranges.

13. Test cross-month ranges.

14. Test cross-year ranges.

15. Consider locale.

16. Consider timezone.

17. Distinguish focused and selected dates.

18. Verify the final input value.

19. Avoid arbitrary waits.

20. Build reusable date-picker components where justified.

---

# Common Beginner Mistakes

## Mistake 1 — Clicking `25`

The page may contain multiple `25` values.

Always identify the correct date context.

---

## Mistake 2 — Using `nth()`

Calendar DOM structures can change.

Positional selectors are fragile.

---

## Mistake 3 — Assuming Every Date Picker Is Native

Modern applications frequently use custom calendar components.

---

## Mistake 4 — Ignoring Year

```text
August 20
```

is incomplete when multiple years are involved.

---

## Mistake 5 — Hard-Coding Today's Date

Today's date changes.

---

## Mistake 6 — Ignoring Timezone

The date seen by the user may differ from a backend UTC date.

---

## Mistake 7 — Ignoring Disabled Dates

A visible date is not necessarily selectable.

---

## Mistake 8 — Using Fixed Navigation Counts

The starting calendar month can change.

---

## Mistake 9 — Confusing Display Format With Internal Value

For example:

```text
20/08/2026
```

may represent:

```text
2026-08-20
```

internally.

---

## Mistake 10 — Ignoring Range State

Selecting a start date does not necessarily mean the range is complete.

---

# Professional Tips

### Tip 1

Treat the calendar as a state machine.

Do not treat it as a collection of numbers.

---

### Tip 2

A date is a semantic entity:

```text
Day
+
Month
+
Year
```

not merely:

```text
25
```

---

### Tip 3

Always understand the application's date source.

Ask:

```text
Browser?
Frontend?
Backend?
Database?
UTC?
Local timezone?
```

---

### Tip 4

For enterprise booking systems, availability often comes from APIs.

Understand the API before blaming the UI.

---

### Tip 5

Use business-level abstractions such as:

```text
selectCheckInDate()
selectCheckOutDate()
```

when the same calendar logic is reused across many tests.

---

### Tip 6

Test boundaries aggressively.

Date bugs often occur around:

```text
Month End
Year End
Leap Day
Minimum Date
Maximum Date
Timezone Transition
```

---

# Real Interview Discussion

# Junior-Level

### Q1. What is a date picker?

**Answer:**

A date picker is a UI control that allows users to enter or select a date.

---

### Q2. What is the difference between a native and custom date picker?

**Answer:**

A native date picker is provided by the browser through controls such as `input type="date"`, while a custom date picker is implemented by the application using its own UI components and interaction logic.

---

### Q3. Why should you avoid using `nth()` for calendar dates?

**Answer:**

Because calendar structures can contain duplicate day numbers, dates from adjacent months, and changing DOM positions. Semantic date identification is more reliable.

---

# Mid-Level

### Q4. How would you select a date from a custom calendar?

**Answer:**

I would identify the calendar container, determine the displayed month and year, navigate to the required month if necessary, locate the target date using semantic information, select it, and verify the resulting state and input value.

---

### Q5. How would you test disabled dates?

**Answer:**

I would verify that dates prohibited by business rules are disabled or otherwise non-selectable and then verify that valid dates remain selectable.

---

### Q6. How would you test a date range?

**Answer:**

I would select the start date, verify the range state, select the end date, and verify both endpoints and the resulting displayed range. I would also cover invalid ranges and boundary conditions where required.

---

# Senior-Level

### Q7. How would you automate a calendar where availability comes from an API?

**Answer:**

I would understand the availability API and its test-data contract, then validate that the UI correctly represents the API-provided availability. Depending on the test objective, I might combine API setup or interception with UI validation.

---

### Q8. How would you handle a calendar that shows two months?

**Answer:**

I would scope the target date to the correct month panel and explicitly identify the month and year before selecting the date. I would never assume that a day number uniquely identifies a date.

---

# Lead-Level

### Q9. How would you design a reusable DatePicker component?

**Answer:**

I would encapsulate calendar navigation, date identification, selection, disabled-state handling, range selection, and date verification. The public API would expose business-level operations such as selecting a specific date or date range, while tests would remain independent of calendar DOM implementation.

---

# Architect-Level

### Q10. What are the major architectural challenges of enterprise date automation?

**Strong Answer:**

The major challenges include native versus custom implementations, dynamic availability, date/timezone representation, locale differences, deterministic test data, calendar state, disabled-date rules, range selection, cross-month and cross-year navigation, accessibility, synchronization, and maintainability. An enterprise framework should centralize reusable date-picker behavior while keeping business workflows readable and test data deterministic.

---

### Q11. How would you prevent date tests from becoming flaky?

**Strong Answer:**

I would avoid uncontrolled dependence on the current date, use deterministic or controlled test data where appropriate, synchronize with meaningful calendar state instead of fixed waits, explicitly handle timezone and locale assumptions, use semantic locators, and isolate external availability dependencies.

---

# Knowledge Check

1. What is a date picker?

2. What is the difference between native and custom date pickers?

3. Why should the control type be identified before automation?

4. What is calendar state?

5. What is the difference between today and selected date?

6. Why should month and year be identified?

7. Why is selecting `25` potentially unsafe?

8. Why is `nth()` fragile for calendars?

9. What is month navigation?

10. What is year navigation?

11. What is a multi-month calendar?

12. What is a date range?

13. What is the difference between start and end date?

14. What is a disabled date?

15. What is a minimum date?

16. What is a maximum date?

17. How would you test date boundaries?

18. Why are leap years important?

19. Why are month-end dates important?

20. What is locale-dependent date formatting?

21. Why is timezone important?

22. What is the difference between date and datetime?

23. What are `aria-selected` and `aria-disabled` used for?

24. What is calendar keyboard navigation?

25. What is the difference between focused and selected date?

26. How would you test a two-month calendar?

27. How would you test a booking calendar?

28. How would you test dynamic date availability?

29. Why should arbitrary waits be avoided?

30. How would you design an enterprise DatePicker component?

---

# Step Summary

The key principle of this lesson is:

> **A calendar is not simply a collection of numbers. It is a stateful date-selection component containing month, year, selection, availability, navigation, and validation rules.**

Your mental model should be:

```text
                    Date Requirement

                          ↓

                  Identify Control

                          ↓

             ┌────────────┴────────────┐

             ▼                         ▼

          Native                     Custom

             │                         │

             ▼                         ▼

       Input Strategy             Calendar State

                                       ↓

                              Month / Year

                                       ↓

                                  Navigation

                                       ↓

                                  Target Date

                                       ↓

                                   Selection

                                       ↓

                                  Validation

                                       ↓

                                  Assertion
```

For a single date:

```text
Target Date
    ↓
Target Month
    ↓
Navigate
    ↓
Select
    ↓
Verify
```

For a date range:

```text
Start Date
    ↓
Range State
    ↓
End Date
    ↓
Range Complete
    ↓
Verify
```

For enterprise automation:

```text
Date Requirement
       ↓
Date Rules
       ↓
Test Data
       ↓
Calendar Component
       ↓
Playwright
       ↓
Application
       ↓
Business Outcome
```

The professional mindset is:

> **Do not automate the visual calendar. Automate the semantic date-selection behavior.**

---

# Progress Milestone

✅ **Step 72 completed.**

You now understand:

* Native date inputs
* Custom date pickers
* Calendar components
* Calendar state
* Current date
* Selected date
* Focused date
* Month navigation
* Year navigation
* Previous month
* Next month
* Multi-month calendars
* Single-date selection
* Date ranges
* Start dates
* End dates
* Disabled dates
* Past-date restrictions
* Future-date restrictions
* Minimum dates
* Maximum dates
* Boundary testing
* Weekend restrictions
* Holiday restrictions
* Month-end boundaries
* Year-end boundaries
* Leap years
* Cross-month selection
* Cross-year selection
* Locale differences
* Date formatting
* Timezone considerations
* Date vs DateTime
* Calendar accessibility
* `aria-selected`
* `aria-disabled`
* Keyboard navigation
* Dynamic availability
* API-driven calendars
* Date synchronization
* Deterministic date data
* Reusable DatePicker components
* Enterprise date automation architecture

Your form-control architecture is now:

```text
                         FORM CONTROLS

                              │

       ┌──────────────────────┼──────────────────────┐

       ▼                      ▼                      ▼

    Selection              Input                  Date

       │                      │                      │

       ▼                      ▼                      ▼

 Checkbox / Radio       Text / Password       Native Date
 Switch / Toggle        Email / Number        Custom Calendar
                        Search / URL          Date Range

       │                      │                      │

       └──────────────────────┼──────────────────────┘

                              ▼

                     Semantic Locator

                              ↓

                     State Identification

                              ↓

                       User Interaction

                              ↓

                       Application State

                              ↓

                         Validation

                              ↓

                          Assertion
```

---

# Next Step

# Step 73 — Mastering Playwright Auto-Waiting, Actionability Checks & Synchronization

The next lesson will deeply cover one of the most important foundations of reliable Playwright automation:

```text
Auto-Waiting

Actionability Checks

Element Visibility

Element Stability

Enabled State

Editable State

Receives Events

Attached to DOM

Locator Resolution

Strictness

Assertions and Waiting

Navigation Waiting

Dynamic Rendering

AJAX

API-Driven UI

Animations

Loading States

Race Conditions

Synchronization Strategy

Why waitForTimeout() Is Weak

Explicit Waiting

waitFor()

waitForLoadState()

waitForURL()

waitForResponse()

waitForFunction()

Enterprise Synchronization Strategy
```

The architecture will move from:

```text
Locator
   ↓
Action
   ↓
Assertion
```

to:

```text
Locator
   ↓
Resolution
   ↓
Actionability
   ↓
Synchronization
   ↓
Action
   ↓
Application State
   ↓
Assertion
   ↓
Expected Outcome
```

This will establish the foundation required for reliable enterprise-grade Playwright automation.

```text
STEP 72 COMPLETE

        ↓

Date & Calendar Automation Mastered

        ↓

NEXT

STEP 73

Mastering Auto-Waiting,
Actionability & Synchronization
```

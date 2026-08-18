# DAX Pattern 004 — Ignore a Specific Filter

## Purpose

Calculate a measure while deliberately ignoring one specific filter but preserving the rest of the current report context.

This is useful when a calculation should remain responsive to most slicers and filters while being unaffected by one particular dimension or attribute.

---

## When to Use

Use this pattern when:

* A measure should ignore one slicer or visual grouping.
* A benchmark or baseline should remain constant across one dimension.
* A calculation should preserve filters such as Year, Region or Channel while ignoring Product Category.
* A comparison requires the current value and a reference value evaluated under different filter contexts.

---

## Business Example

Assume a report shows sales by Product Category:

| Category    |          Sales |
| ----------- | -------------: |
| Clothing    |       £500,000 |
| Footwear    |       £300,000 |
| Accessories |       £200,000 |
| **Total**   | **£1,000,000** |

The report also contains filters for:

```text
Year = 2026
Region = North
```

The requirement is to show total sales for the selected Year and Region while ignoring whichever Category is currently displayed.

The expected result is:

| Category    |    Sales | Sales Ignoring Category |
| ----------- | -------: | ----------------------: |
| Clothing    | £500,000 |              £1,000,000 |
| Footwear    | £300,000 |              £1,000,000 |
| Accessories | £200,000 |              £1,000,000 |

The reference value remains the same for each category because the Category filter is deliberately removed.

---

## Base Measure

Use an existing reusable base measure:

```DAX
Total Sales =
SUM ( Sales[Sales Amount] )
```

---

## DAX Pattern

```DAX
Sales Ignoring Category =
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

---

## How It Works

### 1. Start with the Current Filter Context

Suppose the current visual row is:

```text
Category = Clothing
```

and the report has:

```text
Year   = 2026
Region = North
```

The original filter context is therefore:

```text
Year     = 2026
Region   = North
Category = Clothing
```

`[Total Sales]` evaluated directly in this context returns:

```text
£500,000
```

---

## 2. Modify the Filter Context

The measure uses:

```DAX
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

`CALCULATE` evaluates `[Total Sales]` under a modified filter context.

`REMOVEFILTERS` removes:

```text
Category = Clothing
```

while leaving the other filters intact.

The resulting context is therefore:

```text
Year     = 2026
Region   = North
Category = ALL
```

The result becomes:

```text
£1,000,000
```

---

## Key Principle

This pattern does **not** mean:

> Ignore all report filters.

It means:

> Ignore this specific filter while retaining the remaining context.

That distinction is fundamental to reusable DAX design.

---

## Filter-Context Comparison

### Standard Measure

```DAX
[Total Sales]
```

For Clothing:

```text
Year     = 2026
Region   = North
Category = Clothing

Result = £500,000
```

### Filter-Removal Measure

```DAX
[Sales Ignoring Category]
```

For Clothing:

```text
Year     = 2026
Region   = North
Category = ALL

Result = £1,000,000
```

Only the intended filter has changed.

---

## Why Use `CALCULATE`?

`CALCULATE` is one of the most important DAX functions because it allows an expression to be evaluated under a modified filter context.

Conceptually:

```text
Existing filter context
        ↓
CALCULATE
        ↓
Apply filter changes
        ↓
Evaluate expression
```

For this pattern:

```text
Current context
        ↓
Remove Product[Category]
        ↓
Evaluate [Total Sales]
```

The aggregation itself has not changed.

The context in which the aggregation is evaluated has changed.

---

## Column-Level Filter Removal

This pattern deliberately uses:

```DAX
REMOVEFILTERS ( Product[Category] )
```

rather than:

```DAX
REMOVEFILTERS ( Product )
```

The difference is significant.

### Column-Level Removal

```DAX
REMOVEFILTERS ( Product[Category] )
```

removes only the Category filter.

Other Product-table filters may remain active.

### Table-Level Removal

```DAX
REMOVEFILTERS ( Product )
```

removes filters from the entire Product table.

That could unintentionally remove filters such as:

```text
Brand
Department
Product
Season
```

Use the narrowest filter-removal scope that satisfies the business requirement.

---

## Example with Multiple Filters

Assume the report context is:

```text
Year       = 2026
Region     = North
Department = Womenswear
Category   = Dresses
```

The measure:

```DAX
Sales Ignoring Category =
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

changes the evaluation context to:

```text
Year       = 2026
Region     = North
Department = Womenswear
Category   = ALL
```

The Department filter remains active.

The result therefore represents:

> Sales for all categories within Womenswear, North, during 2026.

---

## Common Use Cases

### Benchmark Against a Wider Group

Compare an individual category with the wider Department total.

### Fixed Reference Line

Create a reference value that remains constant across a visual axis.

### Ignore a Slicer

Allow a measure to disregard a specific selection while other measures continue to respond to it.

### Comparative Measures

Create a current-context value and a broader-context comparison value.

---

## Assumptions

This pattern assumes:

* A reusable base measure already exists.
* The filter to be removed can be clearly identified.
* Other report filters should remain active.
* The semantic model contains appropriate dimension relationships.
* Removing the specified filter produces the intended analytical context.

---

## Common Pitfalls

### Removing Too Much Context

Using:

```DAX
REMOVEFILTERS ()
```

or removing an entire table may produce a denominator or reference value that ignores important report selections.

### Removing the Wrong Column

The filter being removed must correspond to the business question.

For example:

```DAX
REMOVEFILTERS ( Product[Category] )
```

and:

```DAX
REMOVEFILTERS ( Product[Department] )
```

answer different analytical questions.

### Assuming Visual Layout Equals Filter Context

A column appearing in a visual does not necessarily mean it is the only filter affecting the measure.

Slicers, page filters, report filters and cross-filtering can all contribute to the current context.

### Repeating Aggregation Logic

Prefer:

```DAX
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

rather than repeating:

```DAX
SUM ( Sales[Sales Amount] )
```

inside every dependent measure.

---

## Performance Considerations

This pattern is generally inexpensive in a well-designed star schema.

Good practice includes:

* Removing filters from dimension columns rather than large fact tables.
* Reusing existing base measures.
* Keeping filter manipulation as narrow as possible.
* Avoiding unnecessary iterator functions.
* Testing behaviour in visuals containing multiple interacting filters.

The main consideration is usually analytical correctness rather than computational cost.

---

## Alternatives

Depending on the requirement, related functions include:

### `ALL`

Can also remove filters and is commonly encountered in existing DAX.

### `ALLSELECTED`

Useful when the calculation should preserve the user's wider selected set.

### `ALLEXCEPT`

Useful when most filters on a table should be removed while explicitly preserving selected columns.

### Explicit Filter Expressions

`CALCULATE` can also add or replace filters directly rather than only removing them.

These approaches should be chosen according to the required filter behaviour rather than syntax preference.

---

## Relationship to Percentage Patterns

Pattern 001 — Percentage of Total uses this same principle when calculating its denominator:

```DAX
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

The difference is that Pattern 004 isolates the filter-context technique itself.

This makes it reusable for many calculations that have nothing to do with percentages.

---

## Related Patterns

* [Percentage of Total](../percentages/percentage-of-total.md)
* Percentage of Selected Total
* Preserve Selected Filters
* Remove All Filters from a Dimension
* Keep Only Specific Filters
* Fixed Benchmark Measure

---

## Classification

| Attribute               | Value                |
| ----------------------- | -------------------- |
| Type                    | Measure Pattern      |
| Category                | Filter Context       |
| Difficulty              | Basic / Intermediate |
| Context Dependency      | High                 |
| Reusability             | High                 |
| Performance Sensitivity | Low                  |

---

## Notes

The key concept in this pattern is:

> DAX calculations are often changed not by altering the aggregation, but by changing the filter context in which that aggregation is evaluated.

Understanding deliberate filter removal is foundational to more advanced use of `CALCULATE`.

---

## Revision History

| Version | Date       | Change          |
| ------- | ---------- | --------------- |
| 1.0     | 2026-08-18 | Initial version |

---

[← Back to Filter Context Patterns](index.md)

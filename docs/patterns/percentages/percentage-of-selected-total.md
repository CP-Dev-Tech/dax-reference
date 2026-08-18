# DAX Pattern 002 — Percentage of Selected Total

## Purpose

Calculate the contribution of the current item to the total represented by the user's current selection.

This pattern is useful when slicers or report selections define the comparison set and the denominator should reflect only those selected values.

---

## When to Use

Use this pattern when you need to calculate:

* Percentage contribution within a slicer selection.
* Share of currently selected categories, products or regions.
* Dynamic percentage-of-total calculations that respond to user selections.
* Contribution within the visible selection rather than against the broader model total.

---

## Business Example

Assume the full model contains:

| Category    |          Sales |
| ----------- | -------------: |
| Clothing    |       £500,000 |
| Footwear    |       £300,000 |
| Accessories |       £200,000 |
| **Total**   | **£1,000,000** |

The report user selects only:

```text
Clothing
Footwear
```

The selected total is therefore:

```text
£800,000
```

The required result becomes:

| Category           |        Sales | % of Selected Total |
| ------------------ | -----------: | ------------------: |
| Clothing           |     £500,000 |               62.5% |
| Footwear           |     £300,000 |               37.5% |
| **Selected Total** | **£800,000** |            **100%** |

Accessories is not part of the selected comparison set.

---

## Base Measure

Use the existing reusable measure:

```DAX
Total Sales =
SUM ( Sales[Sales Amount] )
```

---

## DAX Pattern

```DAX
Sales % of Selected Total =
VAR CurrentSales =
    [Total Sales]

VAR SelectedCategorySales =
    CALCULATE (
        [Total Sales],
        ALLSELECTED ( Product[Category] )
    )

RETURN
    DIVIDE (
        CurrentSales,
        SelectedCategorySales
    )
```

---

## How It Works

### 1. Calculate the Current Value

```DAX
VAR CurrentSales =
    [Total Sales]
```

For the Clothing row, the current evaluation context includes:

```text
Product[Category] = "Clothing"
```

The result is therefore:

```text
CurrentSales = £500,000
```

---

### 2. Calculate the Selected Denominator

```DAX
VAR SelectedCategorySales =
    CALCULATE (
        [Total Sales],
        ALLSELECTED ( Product[Category] )
    )
```

`ALLSELECTED` removes the row-level category filter while retaining the broader selection made by the report user.

If the user has selected:

```text
Clothing
Footwear
```

then the denominator becomes:

```text
£500,000 + £300,000 = £800,000
```

Accessories remains excluded because it was not selected.

---

## Filter-Context Behaviour

The numerator and denominator are evaluated differently.

### Numerator

For Clothing:

```text
Current row:
Category = Clothing

User selection:
Clothing
Footwear
```

Result:

```text
£500,000
```

### Denominator

The row-level category restriction is removed, but the wider user selection is retained:

```text
Selected categories:
Clothing
Footwear
```

Result:

```text
£800,000
```

The final calculation is therefore:

```text
£500,000
────────── = 0.625 = 62.5%
 £800,000
```

---

## Difference from Pattern 001

Pattern 001 uses:

```DAX
REMOVEFILTERS ( Product[Category] )
```

Pattern 002 uses:

```DAX
ALLSELECTED ( Product[Category] )
```

These are not interchangeable.

### Pattern 001 — Percentage of Total

The denominator removes the current category filter.

Depending on the wider report context, this can evaluate against all available categories.

### Pattern 002 — Percentage of Selected Total

The denominator respects the broader selection made by the report user.

This allows the comparison set itself to change dynamically.

---

## Example Comparison

Assume the full dataset is:

```text
Clothing     £500,000
Footwear     £300,000
Accessories  £200,000
```

The user selects:

```text
Clothing
Footwear
```

For Clothing:

### Percentage of Total

```text
£500,000 / £1,000,000 = 50%
```

### Percentage of Selected Total

```text
£500,000 / £800,000 = 62.5%
```

The correct pattern therefore depends on the business question being asked.

---

## Business Questions

Pattern 001 answers:

> What percentage of the relevant total does this item represent?

Pattern 002 answers:

> What percentage of the currently selected comparison set does this item represent?

That distinction should be established before choosing the DAX implementation.

---

## Assumptions

This pattern assumes:

* `[Total Sales]` already exists.
* `Product[Category]` defines the members being compared.
* The report allows users to filter or select category values.
* The denominator should respect the user's selected comparison set.
* The semantic model relationships are correctly configured.

---

## Common Pitfalls

### Using `ALLSELECTED` Without a Clear Requirement

`ALLSELECTED` should not be used simply because a report contains slicers.

The analytical requirement must specifically call for the denominator to reflect the selected comparison set.

### Assuming `ALLSELECTED` Means "Everything Visible"

Its behaviour depends on evaluation context and can become less intuitive in complex visuals or nested calculations.

Always validate the result against the intended business meaning.

### Confusing Row Context with Selection Context

The current row in a visual and the broader user selection are separate contextual concepts.

This pattern deliberately removes the former while retaining the latter.

### Repeating Base Aggregation Logic

Continue to use measure branching:

```text
Total Sales
    ↓
Sales % of Selected Total
```

rather than reproducing aggregation logic in every dependent measure.

---

## Performance Considerations

For normal dimensional models, this pattern is generally inexpensive.

Consider:

* Keeping the filter target specific.
* Using established dimension columns for filtering.
* Avoiding unnecessary iteration.
* Testing `ALLSELECTED` carefully in complex report pages containing multiple interacting filters.

The main risk with this pattern is usually semantic correctness rather than raw performance.

---

## Alternatives

Depending on the requirement, alternatives include:

* `REMOVEFILTERS`
* `ALL`
* `ALLEXCEPT`
* Explicit filter expressions within `CALCULATE`

The appropriate function depends on which filters should be removed and which must remain active.

---

## Related Patterns

* [Percentage of Total](percentage-of-total.md)
* Percentage of Parent
* Percentage Within Group
* Dynamic Share of Total
* Top N Percentage Contribution

---

## Classification

| Attribute               | Value           |
| ----------------------- | --------------- |
| Type                    | Measure Pattern |
| Category                | Percentages     |
| Difficulty              | Intermediate    |
| Context Dependency      | High            |
| Reusability             | High            |
| Performance Sensitivity | Low             |

---

## Notes

The main lesson from this pattern is:

> `ALLSELECTED` is useful when the denominator should reflect the user's active comparison set rather than the broader available total.

The business meaning of the denominator should always be defined before selecting the DAX function.

---

## Revision History

| Version | Date       | Change          |
| ------- | ---------- | --------------- |
| 1.0     | 2026-08-18 | Initial version |

---

[← Back to Percentage Patterns](./README.md)

# DAX Pattern 001 — Percentage of Total

## Purpose

Calculate the contribution of the current item to a total while retaining the filter context that defines the required analysis.

A common example is calculating each product category's sales as a percentage of total sales.

---

## When to Use

Use this pattern when you need to calculate:

* Percentage of total sales by product, category or region.
* Percentage contribution to an overall value.
* Share of revenue, cost, quantity or another additive measure.
* Relative contribution of individual members within a dimension.

---

## Business Example

Assume a report contains sales by product category:

| Category    |          Sales |
| ----------- | -------------: |
| Clothing    |       £500,000 |
| Footwear    |       £300,000 |
| Accessories |       £200,000 |
| **Total**   | **£1,000,000** |

The required result is:

| Category    |          Sales | % of Total |
| ----------- | -------------: | ---------: |
| Clothing    |       £500,000 |        50% |
| Footwear    |       £300,000 |        30% |
| Accessories |       £200,000 |        20% |
| **Total**   | **£1,000,000** |   **100%** |

---

## Base Measure

Start with a reusable base measure:

```DAX
Total Sales =
SUM ( Sales[Sales Amount] )
```

This measure returns sales within the current filter context.

---

## DAX Pattern

```DAX
Sales % of Total =
VAR CurrentSales =
    [Total Sales]

VAR AllCategorySales =
    CALCULATE (
        [Total Sales],
        REMOVEFILTERS ( Product[Category] )
    )

RETURN
    DIVIDE (
        CurrentSales,
        AllCategorySales
    )
```

---

## How It Works

### 1. Calculate the Current Value

```DAX
VAR CurrentSales =
    [Total Sales]
```

When the measure is evaluated against a category such as **Clothing**, the current filter context includes:

```text
Product[Category] = "Clothing"
```

`[Total Sales]` therefore returns sales for Clothing.

In the example:

```text
CurrentSales = £500,000
```

---

### 2. Calculate the Denominator

```DAX
VAR AllCategorySales =
    CALCULATE (
        [Total Sales],
        REMOVEFILTERS ( Product[Category] )
    )
```

`CALCULATE` evaluates `[Total Sales]` using a modified filter context.

`REMOVEFILTERS` removes the filter specifically from:

```DAX
Product[Category]
```

The category currently being displayed therefore no longer restricts the calculation.

For Clothing:

```text
Current context:

Category = Clothing
        ↓
Total Sales = £500,000


Denominator context:

Category filter removed
        ↓
Total Sales = £1,000,000
```

Other report filters remain active unless they also affect the filter being explicitly removed.

---

### 3. Divide the Current Value by the Total

```DAX
DIVIDE (
    CurrentSales,
    AllCategorySales
)
```

For Clothing:

```text
£500,000
────────── = 0.50 = 50%
£1,000,000
```

`DIVIDE` is preferred to the `/` operator for this pattern because it safely handles a zero or blank denominator.

---

## Filter-Context Behaviour

The important principle is that the numerator and denominator are intentionally evaluated under different filter contexts.

### Numerator

Retains the current category filter:

```text
Category = Clothing
Region   = North
Year     = 2026
```

### Denominator

Removes only the category filter:

```text
Category = ALL
Region   = North
Year     = 2026
```

The result therefore answers:

> What percentage of sales within the currently selected Region and Year came from this Category?

This distinction is important. The denominator is not necessarily the unrestricted total for the entire model.

---

## Why Not Remove Every Filter?

A broader expression could be written using:

```DAX
REMOVEFILTERS ()
```

However, this removes filters much more widely and may cause the denominator to ignore report selections that should remain relevant.

For example, if the user selects:

```text
Year = 2026
Region = North
```

the required percentage may be the category's contribution **within North during 2026**, not its contribution to sales across every region and every year.

Filter removal should therefore be deliberate and as specific as the analytical requirement allows.

---

## Assumptions

This pattern assumes:

* `[Total Sales]` is an existing measure.
* `Product[Category]` is the dimension attribute whose contribution is being calculated.
* The data model contains appropriate relationships between Product and Sales.
* Other relevant report filters should remain active.
* The denominator represents the total across categories within the remaining filter context.

---

## Common Pitfalls

### Removing Too Many Filters

Using a broad filter-removal expression can unintentionally ignore slicers and other report selections.

### Removing the Wrong Filter

The denominator must remove the filter responsible for dividing the visual into the members being compared.

### Dividing Directly

Using:

```DAX
CurrentSales / AllCategorySales
```

works when the denominator is valid, but `DIVIDE` provides safer handling of zero and blank denominators.

### Recalculating Business Logic

Avoid repeating:

```DAX
SUM ( Sales[Sales Amount] )
```

throughout dependent measures when `[Total Sales]` already defines that business logic.

Prefer measure branching:

```text
Total Sales
    ↓
Sales % of Total
```

---

## Performance Considerations

This pattern is generally inexpensive when operating over a well-designed star schema.

Consider:

* Removing filters from the required dimension column rather than unnecessarily broad portions of the model.
* Reusing established base measures.
* Maintaining appropriate dimension-to-fact relationships.
* Avoiding unnecessary iterator functions when simple aggregation is sufficient.

---

## Alternatives

The precise implementation depends on the analytical requirement.

Related approaches may use:

* `REMOVEFILTERS`
* `ALL`
* `ALLSELECTED`

These functions can produce materially different results depending on the report's filter context.

`ALLSELECTED`, for example, may be appropriate when the required denominator is the total represented by the user's current selection rather than the broader total available after removing a specific filter.

These should be treated as related patterns rather than interchangeable syntax.

---

## Related Patterns

Future related entries:

* Percentage of Selected Total
* Percentage of Parent
* Percentage Variance
* Dynamic Share of Total
* Ranking by Percentage Contribution

---

## Classification

| Attribute               | Value                |
| ----------------------- | -------------------- |
| Type                    | Measure Pattern      |
| Category                | Percentages          |
| Difficulty              | Basic / Intermediate |
| Context Dependency      | High                 |
| Reusability             | High                 |
| Performance Sensitivity | Low                  |

---

## Notes

The key learning from this pattern is not the division itself.

The important DAX concept is:

> **Evaluate the numerator in the current filter context and deliberately modify the filter context for the denominator.**

Understanding that behaviour provides a foundation for many more advanced DAX calculations.

---

## Revision History

| Version | Date       | Change          |
| ------- | ---------- | --------------- |
| 1.0     | 2026-08-18 | Initial version |

---

[← Back to Percentage Patterns](./README.md)

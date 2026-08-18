# DAX Pattern 003 — Percentage of Parent

## Purpose

Calculate the contribution of the current item relative to its immediate parent within a hierarchy.

A common example is calculating:

* Product as a percentage of Category.
* Category as a percentage of Department.
* Department as a percentage of Total.

This pattern is useful when the denominator needs to change depending on the hierarchy level currently being displayed.

---

## When to Use

Use this pattern when:

* A matrix or hierarchical visual contains multiple levels.
* The denominator must represent the current item's immediate parent.
* Percentage contribution should be meaningful at each hierarchy level.
* A single measure needs to behave differently depending on drill level.

---

## Business Example

Assume the hierarchy is:

```text
Department
    ↓
Category
    ↓
Product
```

Example data:

| Department | Category | Product   |    Sales |
| ---------- | -------- | --------- | -------: |
| Womenswear | Dresses  | Product A | £200,000 |
| Womenswear | Dresses  | Product B | £300,000 |
| Womenswear | Knitwear | Product C | £250,000 |
| Womenswear | Knitwear | Product D | £250,000 |

Totals:

```text
Womenswear        £1,000,000

Dresses             £500,000
Knitwear            £500,000

Product A           £200,000
Product B           £300,000
Product C           £250,000
Product D           £250,000
```

The required percentages are:

```text
Product A = 40% of Dresses
Product B = 60% of Dresses

Dresses   = 50% of Womenswear
Knitwear  = 50% of Womenswear
```

The denominator therefore changes depending on the current hierarchy level.

---

## Base Measure

Use the existing base measure:

```DAX
Total Sales =
SUM ( Sales[Sales Amount] )
```

---

## DAX Pattern

```DAX
Sales % of Parent =
VAR CurrentSales =
    [Total Sales]

VAR ParentSales =
    SWITCH (
        TRUE (),

        ISINSCOPE ( Product[Product] ),
            CALCULATE (
                [Total Sales],
                REMOVEFILTERS ( Product[Product] )
            ),

        ISINSCOPE ( Product[Category] ),
            CALCULATE (
                [Total Sales],
                REMOVEFILTERS ( Product[Category] )
            ),

        ISINSCOPE ( Product[Department] ),
            CALCULATE (
                [Total Sales],
                REMOVEFILTERS ( Product[Department] )
            )
    )

RETURN
    DIVIDE (
        CurrentSales,
        ParentSales
    )
```

---

## How It Works

### 1. Determine the Current Hierarchy Level

The measure uses:

```DAX
ISINSCOPE ( ... )
```

to identify which hierarchy level is currently being evaluated.

For example:

```DAX
ISINSCOPE ( Product[Product] )
```

returns `TRUE` when the visual is currently evaluating an individual product row.

Likewise:

```DAX
ISINSCOPE ( Product[Category] )
```

returns `TRUE` when the current evaluation level is Category.

---

## 2. Product-Level Calculation

If the current row is:

```text
Product A
```

the measure evaluates:

```DAX
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Product] )
)
```

This removes the product-level filter while retaining the Category and Department filters.

The context therefore changes from:

```text
Department = Womenswear
Category   = Dresses
Product    = Product A
```

to:

```text
Department = Womenswear
Category   = Dresses
Product    = ALL
```

The denominator becomes total sales for Dresses:

```text
£500,000
```

Therefore:

```text
£200,000
────────── = 40%
£500,000
```

---

## 3. Category-Level Calculation

If the current row is:

```text
Dresses
```

the measure evaluates:

```DAX
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

The context changes from:

```text
Department = Womenswear
Category   = Dresses
```

to:

```text
Department = Womenswear
Category   = ALL
```

The denominator becomes:

```text
Womenswear sales = £1,000,000
```

Therefore:

```text
£500,000
────────── = 50%
£1,000,000
```

---

## 4. Department-Level Calculation

At Department level:

```DAX
REMOVEFILTERS ( Product[Department] )
```

removes the Department restriction.

The denominator therefore becomes the relevant higher-level total within the remaining filter context.

---

## Why `ISINSCOPE` Is Important

A hierarchy can produce different filter combinations depending on:

* Drill level.
* Expand/collapse state.
* Matrix structure.
* Subtotals.
* Report filters.

Simply checking whether a column is filtered is not always sufficient.

`ISINSCOPE` is designed to identify whether a hierarchy column is currently participating as the active grouping level in the visual.

This makes it particularly useful for hierarchy-aware measures.

---

## Evaluation Order

The order of the `SWITCH` conditions matters.

The measure checks the most detailed level first:

```text
Product
↓
Category
↓
Department
```

This is necessary because when Product is in scope, Category and Department may also form part of the wider filter context.

The measure should therefore detect the lowest active hierarchy level first.

---

## Filter-Context Behaviour

At Product level:

```text
Numerator:
Department = Womenswear
Category   = Dresses
Product    = Product A

Denominator:
Department = Womenswear
Category   = Dresses
Product    = ALL
```

At Category level:

```text
Numerator:
Department = Womenswear
Category   = Dresses

Denominator:
Department = Womenswear
Category   = ALL
```

The result is always:

> Current item ÷ immediate parent total

---

## Assumptions

This pattern assumes:

* A recognised hierarchy exists.
* The hierarchy levels are stored in dimension columns.
* `[Total Sales]` already exists.
* The hierarchy is evaluated from higher to lower levels.
* Removing the current-level filter produces the required immediate parent context.
* Model relationships are correctly configured.

---

## Common Pitfalls

### Checking Hierarchy Levels in the Wrong Order

This can cause the measure to return the wrong denominator.

Always test the most detailed level first.

### Removing Too Many Filters

For Product percentage of Category, removing Category as well as Product would calculate Product as a percentage of Department rather than Category.

### Using `HASONEVALUE` as a Substitute

`HASONEVALUE` answers whether one value exists in the current filter context.

That is not the same question as whether a column is the active hierarchy level.

For hierarchy-aware calculations, `ISINSCOPE` is usually the clearer choice.

### Ignoring Subtotals

Hierarchy measures should always be tested against:

* Detail rows.
* Parent rows.
* Subtotals.
* Grand totals.

The required subtotal behaviour should be explicitly defined.

---

## Grand Total Behaviour

At the grand-total level, none of the hierarchy columns may be in scope.

In that situation:

```DAX
ParentSales
```

may evaluate to `BLANK()`.

The resulting percentage will therefore also return blank.

This is often desirable because a grand total being "a percentage of its parent" may have no meaningful business interpretation.

If the requirement is to show:

```text
100%
```

at the grand total, that behaviour should be added deliberately rather than assumed.

---

## Optional Grand Total Version

If the requirement is for the grand total to display `100%`, the pattern can be extended:

```DAX
Sales % of Parent =
VAR CurrentSales =
    [Total Sales]

VAR ParentSales =
    SWITCH (
        TRUE (),

        ISINSCOPE ( Product[Product] ),
            CALCULATE (
                [Total Sales],
                REMOVEFILTERS ( Product[Product] )
            ),

        ISINSCOPE ( Product[Category] ),
            CALCULATE (
                [Total Sales],
                REMOVEFILTERS ( Product[Category] )
            ),

        ISINSCOPE ( Product[Department] ),
            CALCULATE (
                [Total Sales],
                REMOVEFILTERS ( Product[Department] )
            ),

        [Total Sales]
    )

RETURN
    DIVIDE (
        CurrentSales,
        ParentSales
    )
```

At grand total:

```text
Current Sales ÷ Current Sales = 100%
```

Use this variation only where the business presentation requires it.

---

## Performance Considerations

This pattern is generally inexpensive for standard dimensional models.

Consider:

* Keeping hierarchy columns within a proper dimension table.
* Using reusable base measures.
* Avoiding unnecessary iterators.
* Limiting filter removal to the relevant hierarchy column.
* Testing matrix behaviour where the hierarchy contains many members.

The complexity of this pattern is primarily contextual rather than computational.

---

## Alternatives

Depending on the model and hierarchy design, alternatives may include:

* Separate measures for each hierarchy level.
* Explicit `CALCULATE` logic tied to individual visuals.
* `ALLSELECTED` when the parent denominator must respect a user-defined selected set.
* Calculation groups for reusable hierarchy-aware behaviour.

A single `ISINSCOPE` measure is particularly useful when one measure needs to adapt dynamically across hierarchy levels.

---

## Related Patterns

* [Percentage of Total](percentage-of-total.md)
* [Percentage of Selected Total](percentage-of-selected-total.md)
* Percentage Within Group
* Hierarchy-Aware Variance
* Dynamic Matrix Measures

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

The key concept in this pattern is:

> The denominator is determined by the hierarchy level currently in scope.

`ISINSCOPE` identifies that level, while `CALCULATE` and `REMOVEFILTERS` create the appropriate parent context.

This allows one measure to behave intelligently across multiple drill levels.

---

## Revision History

| Version | Date       | Change          |
| ------- | ---------- | --------------- |
| 1.0     | 2026-08-18 | Initial version |

---

[← Back to Percentage Patterns](./README.md)

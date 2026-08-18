# DAX Pattern 005 — Keep Only Specific Filter

## Purpose

Evaluate a measure while removing most filters from a table but deliberately preserving one or more specified filters.

This pattern is useful when a calculation should ignore lower-level or unrelated filters within a dimension while retaining a defined grouping context.

---

## When to Use

Use this pattern when:

* A calculation should preserve one hierarchy level while ignoring lower levels.
* A benchmark should remain fixed within a Department, Region or other grouping.
* A measure should remove most filters from a dimension table but keep selected columns active.
* The intended comparison set is defined by a specific parent or grouping attribute.
* Repeating multiple `REMOVEFILTERS` expressions would be less clear than explicitly preserving the required filters.

---

## Business Example

Assume a Product dimension contains:

```text
Department
Category
Product
Brand
```

A report shows Product-level sales within Department.

Example:

| Department | Category | Product   |    Sales |
| ---------- | -------- | --------- | -------: |
| Womenswear | Dresses  | Product A | £200,000 |
| Womenswear | Dresses  | Product B | £300,000 |
| Womenswear | Knitwear | Product C | £250,000 |
| Womenswear | Knitwear | Product D | £250,000 |

The requirement is to calculate total Womenswear sales for every Product row while ignoring Category and Product.

The required reference value is:

```text
Womenswear Total Sales = £1,000,000
```

For each Product row:

| Product   |    Sales | Department Sales |
| --------- | -------: | ---------------: |
| Product A | £200,000 |       £1,000,000 |
| Product B | £300,000 |       £1,000,000 |
| Product C | £250,000 |       £1,000,000 |
| Product D | £250,000 |       £1,000,000 |

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
Sales Within Department =
CALCULATE (
    [Total Sales],
    ALLEXCEPT (
        Product,
        Product[Department]
    )
)
```

---

## How It Works

### 1. Start with the Current Filter Context

Suppose the current visual row is:

```text
Department = Womenswear
Category   = Dresses
Product    = Product A
```

The current measure:

```DAX
[Total Sales]
```

returns:

```text
£200,000
```

because all three filters contribute to the current context.

---

## 2. Preserve Only the Required Filter

The expression:

```DAX
ALLEXCEPT (
    Product,
    Product[Department]
)
```

removes filters from the `Product` table except for:

```text
Product[Department]
```

The context therefore changes from:

```text
Department = Womenswear
Category   = Dresses
Product    = Product A
```

to:

```text
Department = Womenswear
Category   = ALL
Product    = ALL
```

The calculation then returns:

```text
£1,000,000
```

for total Womenswear sales.

---

## Key Principle

`ALLEXCEPT` can be understood as:

> Remove filters from this table except for the filters I explicitly want to preserve.

Conceptually:

```text
Current Product filters
        ↓
Department = Womenswear
Category   = Dresses
Product    = Product A
        ↓
ALLEXCEPT
        ↓
Keep Department
Remove Category
Remove Product
        ↓
Department = Womenswear
```

---

## Why Use `ALLEXCEPT`?

The same business outcome could sometimes be achieved using multiple filter-removal expressions.

For example:

```DAX
Sales Within Department =
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] ),
    REMOVEFILTERS ( Product[Product] )
)
```

However, if the Product table contains many additional attributes such as:

```text
Brand
Season
Collection
Colour
Range
Supplier
```

the expression may become harder to maintain.

`ALLEXCEPT` expresses the intent from the opposite direction:

> Remove everything from Product except Department.

This can be clearer when the preserved context is simpler than the set of filters to remove.

---

## Filter-Context Behaviour

Assume the report context contains:

```text
Year       = 2026
Region     = North
Department = Womenswear
Category   = Dresses
Product    = Product A
```

The measure:

```DAX
Sales Within Department =
CALCULATE (
    [Total Sales],
    ALLEXCEPT (
        Product,
        Product[Department]
    )
)
```

produces:

```text
Year       = 2026
Region     = North
Department = Womenswear
Category   = ALL
Product    = ALL
```

Notice that:

```text
Year
Region
```

remain active because `ALLEXCEPT` is applied only to the `Product` table.

---

## Preserving More Than One Filter

`ALLEXCEPT` can preserve multiple columns from the same table.

For example:

```DAX
Sales Within Department and Brand =
CALCULATE (
    [Total Sales],
    ALLEXCEPT (
        Product,
        Product[Department],
        Product[Brand]
    )
)
```

This removes other Product filters while preserving:

```text
Department
Brand
```

The resulting measure answers:

> What are total sales within the current Department and Brand combination?

---

## Comparison with Pattern 004

Pattern 004 uses:

```DAX
REMOVEFILTERS ( Product[Category] )
```

and means:

> Remove this specific filter.

Pattern 005 uses:

```DAX
ALLEXCEPT (
    Product,
    Product[Department]
)
```

and means:

> Remove filters from this table except for this specific filter.

The choice depends on which description is simpler and more closely matches the business requirement.

---

## Example Comparison

Assume the current Product context is:

```text
Department = Womenswear
Category   = Dresses
Brand      = Brand A
Product    = Product A
```

### Remove Category Only

```DAX
CALCULATE (
    [Total Sales],
    REMOVEFILTERS ( Product[Category] )
)
```

Resulting context:

```text
Department = Womenswear
Category   = ALL
Brand      = Brand A
Product    = Product A
```

Other Product filters remain.

### Preserve Department Only

```DAX
CALCULATE (
    [Total Sales],
    ALLEXCEPT (
        Product,
        Product[Department]
    )
)
```

Resulting context:

```text
Department = Womenswear
Category   = ALL
Brand      = ALL
Product    = ALL
```

These two calculations answer very different business questions.

---

## Assumptions

This pattern assumes:

* The filters being manipulated belong to the same dimension table.
* The filter or filters to preserve are clearly defined.
* Other filters on that dimension should be removed.
* Filters from unrelated dimensions should remain active.
* The semantic model relationships are correctly configured.
* The preserved column provides a meaningful analytical grouping.

---

## Common Pitfalls

### Preserving Too Little Context

Using:

```DAX
ALLEXCEPT (
    Product,
    Product[Department]
)
```

will remove every other Product-table filter.

If Brand or Season should remain active, they must also be preserved.

---

### Assuming External Filters Are Removed

`ALLEXCEPT` only operates on the specified table.

For example:

```text
Date
Region
Customer
Channel
```

filters remain active unless explicitly changed elsewhere.

---

### Using `ALLEXCEPT` Automatically for Parent Totals

A parent-total requirement does not always mean `ALLEXCEPT` is appropriate.

Sometimes a more targeted:

```DAX
REMOVEFILTERS
```

expression is easier to understand and less likely to remove useful context.

Choose the function according to the required filter behaviour.

---

### Using It Across Unrelated Tables

The preserved columns passed to `ALLEXCEPT` must belong to the table being supplied as its first argument.

For example:

```DAX
ALLEXCEPT (
    Product,
    Product[Department],
    Region[Region]
)
```

is not a valid way to preserve filters from unrelated tables.

---

## When `REMOVEFILTERS` May Be Clearer

Suppose the only requirement is:

> Ignore Category but keep all other Product filters.

Then this is explicit:

```DAX
REMOVEFILTERS ( Product[Category] )
```

Using:

```DAX
ALLEXCEPT
```

would require identifying every Product filter that should remain active.

In this scenario, Pattern 004 is generally clearer.

---

## When `ALLEXCEPT` May Be Clearer

Suppose the requirement is:

> Ignore all Product-level detail but keep Department.

Then:

```DAX
ALLEXCEPT (
    Product,
    Product[Department]
)
```

expresses that requirement directly.

The preferred implementation should communicate the analytical intent as clearly as possible.

---

## Common Use Cases

### Department-Level Benchmark

Show a Department total alongside lower-level Category or Product rows.

### Regional Benchmark

Remove lower-level geographic filters while preserving Region.

### Customer-Segment Analysis

Preserve Customer Segment while ignoring individual Customer filters.

### Parent-Level Comparison

Create a reference value at a specified parent grouping.

### Stable Group Baseline

Create a baseline that remains constant within a defined group.

---

## Performance Considerations

This pattern is generally efficient in a well-designed dimensional model.

Consider:

* Applying `ALLEXCEPT` to dimension tables rather than large fact tables.
* Preserving only the columns required by the business rule.
* Reusing existing base measures.
* Avoiding unnecessary filter manipulation.
* Testing behaviour where the dimension contains many attributes and interacting slicers.

The main risk is usually unintended filter removal rather than computational cost.

---

## Alternatives

Depending on the requirement, alternatives include:

### `REMOVEFILTERS`

Prefer when only one or a small number of known filters need to be removed.

### `ALL`

Can remove filters from a table or column and is frequently encountered in existing DAX.

### `ALLSELECTED`

Prefer when the wider user-selected comparison set must be retained.

### Explicit `CALCULATE` Filters

Useful where filters need to be added, replaced or precisely controlled.

The correct implementation depends on the business meaning of the required evaluation context.

---

## Related Patterns

* [Ignore a Specific Filter](ignore-specific-filter.md)
* [Percentage of Total](../percentages/percentage-of-total.md)
* Percentage of Parent
* Preserve Selected Filters
* Remove All Filters from a Dimension
* Fixed Group Benchmark

---

## Classification

| Attribute               | Value           |
| ----------------------- | --------------- |
| Type                    | Measure Pattern |
| Category                | Filter Context  |
| Difficulty              | Intermediate    |
| Context Dependency      | High            |
| Reusability             | High            |
| Performance Sensitivity | Low             |

---

## Notes

The key concept in this pattern is:

> `ALLEXCEPT` defines filter context by identifying what should remain rather than listing everything that should be removed.

Use it when the preserved grouping is clearer and more stable than the set of filters that must be removed.

---

## Revision History

| Version | Date       | Change          |
| ------- | ---------- | --------------- |
| 1.0     | 2026-08-18 | Initial version |

---

[← Back to Filter Context Patterns](index.md)

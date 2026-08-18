# DAX Reference Library

A curated reference library of reusable DAX patterns for Power BI.

The purpose of this site is to provide concise, practical guidance for common analytical calculations while explaining the underlying DAX behaviour, particularly around filter context, hierarchy handling and reusable measure design.

---

## Pattern Catalogue

| ID  | Pattern                                                                              | Category       | Difficulty           |
| --- | ------------------------------------------------------------------------------------ | -------------- | -------------------- |
| 001 | [Percentage of Total](patterns/percentages/percentage-of-total.md)                   | Percentages    | Basic / Intermediate |
| 002 | [Percentage of Selected Total](patterns/percentages/percentage-of-selected-total.md) | Percentages    | Intermediate         |
| 003 | [Percentage of Parent](patterns/percentages/percentage-of-parent.md)                 | Percentages    | Intermediate         |
| 004 | [Ignore a Specific Filter](patterns/filter-context/ignore-specific-filter.md)        | Filter Context | Basic / Intermediate |
| 005 | [Keep Only Specific Filters](patterns/filter-context/keep-only-specific-filter.md)  | Filter Context | Intermediate         |

---

## Categories

### Percentages

Patterns for percentage-of-total, selected-total and hierarchical contribution calculations.

[Browse Percentage Patterns](patterns/percentages/index.md)

### Filter Context

Patterns for deliberately manipulating, preserving and removing filter context during measure evaluation.

[Browse Filter Context Patterns](patterns/filter-context/index.md)

---

## About This Reference

This is a curated public reference derived from a larger private working library.

Only mature, reusable patterns suitable for general use are published here.

Examples are intentionally generic and should be adapted to the semantic model and business requirements of the implementation in which they are used.

---

## Copyright and Use

Copyright © 2026 Carl Patten. All rights reserved.

Individual DAX examples may be adapted for use in Power BI solutions. Reproduction, republication or substantial redistribution of the library's original documentation is not permitted without permission.

[Copyright and permitted use](copyright-and-use.md)

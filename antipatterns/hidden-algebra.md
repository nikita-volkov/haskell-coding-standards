# Anti-pattern: Hidden Algebra

## Symptom

A module is labeled a [Language Module](../patterns/language-module.md), but its typeclasses encode operations with strong laws, and the module is used mainly for folding, evaluating, or equational reasoning.

## Problem

The center of gravity is operations and their laws, not the act of building domain expressions. Calling it a Language Module obscures the algebraic structure and misleads consumers about how to use it.

## Resolution

- Name it an Algebra or operation module.
- Document the laws explicitly.
- Split construction syntax from evaluation if both are needed.

## See Also

- [Language Module](../patterns/language-module.md)

# Anti-pattern: Hidden Algebra

## Symptom

A module is labeled a [Domain Language](../patterns/domain-language.md) or [Intermediate Representation](../patterns/intermediate-representation.md), but its typeclasses encode operations with strong laws, and the module is used mainly for folding, evaluating, or equational reasoning.

## Problem

The center of gravity is operations and their laws, not the act of building expressions in a vocabulary. Calling it a Domain Language or Intermediate Representation obscures the algebraic structure and misleads consumers about how to use it.

## Resolution

- Name it an Algebra or operation module.
- Document the laws explicitly.
- Split construction syntax from evaluation if both are needed.

## See Also

- [Domain Language](../patterns/domain-language.md)
- [Intermediate Representation](../patterns/intermediate-representation.md)

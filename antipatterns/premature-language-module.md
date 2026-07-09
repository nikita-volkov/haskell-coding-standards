# Anti-pattern: Premature Language Module

## Symptom

A module exports a few related types and one incidental typeclass. It is labeled a [Domain Language](../patterns/domain-language.md) or [Intermediate Representation](../patterns/intermediate-representation.md), but the typeclass does not create meaningful structure.

## Problem

The typeclass is doing local convenience work, not defining a grammar. Removing it would not change how consumers use the module. The "Language" label adds conceptual weight without a matching structure.

## Resolution

- If the types are the point and the typeclass is incidental, call the module a vocabulary or domain module.
- Move the typeclass to the module that actually uses it, or inline its operation.
- Promote to Domain Language or Intermediate Representation only when multiple typeclasses form real structure across the vocabulary.

## See Also

- [Domain Language](../patterns/domain-language.md)
- [Intermediate Representation](../patterns/intermediate-representation.md)
- [Stray Typeclass](stray-typeclass.md)

# Anti-pattern: Premature Language Module

## Symptom

A module exports a few related types and one incidental typeclass. It is labeled a [Language Module](../patterns/language-module.md), but the typeclass does not create meaningful structure.

## Problem

The typeclass is doing local convenience work, not defining a grammar. Removing it would not change how consumers use the module. The "Language" label adds conceptual weight without a matching structure.

## Resolution

- If the types are the point and the typeclass is incidental, call the module a vocabulary or domain module.
- Move the typeclass to the module that actually uses it, or inline its operation.
- Promote to Language Module only when multiple typeclasses form real structure across the vocabulary.

## See Also

- [Language Module](../patterns/language-module.md)
- [Stray Typeclass](stray-typeclass.md)

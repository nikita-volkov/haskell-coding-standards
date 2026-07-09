# Anti-pattern: Delayed DSL

## Symptom

A module provides a complete embedded language — syntax, combinators, and a clear interpreter — but is labeled a [Language Module](../patterns/language-module.md) to avoid the heavier term.

## Problem

"Language Module" implies a partial or evolving grammar. A full DSL has stronger obligations: a coherent syntax, evaluation semantics, and usually a clear boundary between construction and interpretation. The lighter name understates the commitment.

## Resolution

- If consumers write whole programs and there is a `runX` or `interpret` function, call it a DSL.
- Reserve Language Module for modules with a partial grammar or no central interpreter.

## See Also

- [Language Module](../patterns/language-module.md)

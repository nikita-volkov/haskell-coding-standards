# Anti-pattern: Delayed DSL

## Symptom

A module provides a complete embedded language — syntax, combinators, and a clear interpreter — but is labeled a [Domain Language](../patterns/domain-language.md) or [Intermediate Representation](../patterns/intermediate-representation.md) to avoid the heavier term.

## Problem

Those patterns imply a partial or evolving grammar. A full DSL has stronger obligations: a coherent syntax, evaluation semantics, and usually a clear boundary between construction and interpretation. The lighter name understates the commitment.

## Resolution

- If consumers write whole programs and there is a `runX` or `interpret` function, call it a DSL.
- Reserve Domain Language / Intermediate Representation for modules with a partial grammar or no central interpreter.

## See Also

- [Domain Language](../patterns/domain-language.md)
- [Intermediate Representation](../patterns/intermediate-representation.md)

# Anti-pattern: Open-World Drift

## Symptom

A [Language Module](../patterns/language-module.md) exposes its typeclasses as extension points for downstream modules to implement with their own types.

## Problem

A Language Module is a closed world: its types and typeclasses evolve together. Once external types implement its classes, the module becomes an open-world typeclass hierarchy. Changes to the classes risk breaking downstream code, and the boundary between language and library interface blurs.

## Resolution

- Keep typeclasses closed to the module's own types.
- If external extension is intentional, redesign as a Port, an Aggregator Namespace, or a public typeclass ecosystem.

## See Also

- [Language Module](../patterns/language-module.md)
- [Port](../patterns/port.md)
- [Aggregator Namespace](../patterns/aggregator-namespace.md)

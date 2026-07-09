# Anti-pattern: Stray Typeclass

## Symptom

A module contains a typeclass that does not belong to the module's primary role. It may be there for local convenience, future reuse, or because no better home was found.

## Problem

The typeclass creates false cohesion. Consumers importing the module for its main purpose inherit an unrelated abstraction. The module's contract becomes harder to reason about.

## Resolution

- Move the typeclass to the module whose topic it belongs to.
- If it has no clear home, create one.
- If it is truly incidental, inline its operation.

## See Also

- [Premature Language Module](premature-language-module.md)
- [Typeclass Dumping Ground](typeclass-dumping-ground.md)

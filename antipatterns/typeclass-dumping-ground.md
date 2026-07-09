# Anti-pattern: Typeclass Dumping Ground

## Symptom

A module collects unrelated typeclasses and instances from different domains. Names like `Classes`, `Typeclasses`, or `Interfaces` are warning signs.

## Problem

The module has no coherent topic. It becomes a dependency magnet: any module that needs one typeclass must import the whole pile. The typeclasses cannot be understood or maintained independently.

## Resolution

- Group typeclasses by domain or abstraction.
- Move each typeclass to the module that owns its topic.
- Delete modules whose only purpose is to gather unrelated classes.

## See Also

- [Stray Typeclass](stray-typeclass.md)

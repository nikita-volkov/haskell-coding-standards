# Anti-patterns

Misapplications of module roles and structures. Each anti-pattern names a common drift, explains why it fails, and points to the correct pattern.

## Language Module

Misclassifications of the [Language Module](../patterns/language-module.md) pattern:

- [Premature Language Module](premature-language-module.md) — a vocabulary module with one incidental typeclass
- [Hidden Algebra](hidden-algebra.md) — a module of law-governed operations labeled as a language
- [Delayed DSL](delayed-dsl.md) — a complete embedded language hiding behind a lighter name
- [Open-World Drift](open-world-drift.md) — a closed language that becomes an extension point
- [Capability Creep](capability-creep.md) — a language module that absorbs effectful operations

## General Module Hygiene

Anti-patterns that apply regardless of the specific module role:

- [Stray Typeclass](stray-typeclass.md) — a typeclass that does not justify its presence in the module
- [Typeclass Dumping Ground](typeclass-dumping-ground.md) — unrelated typeclasses collected in one module

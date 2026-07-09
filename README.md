# Haskell Coding Standards

A personal, evolving set of coding standards for consistent Haskell, shared with agents and collaborators across projects.

This is not a style guide in the sense of aesthetic preferences. It is a standards document: conventions, patterns, and architecture that remove repeated decisions. Where a rule exists, follow it. Where a rule is absent, the gap is a future contribution.

## Normative Language

- **MUST** — non-negotiable; violations are always wrong
- **SHOULD** — strong default; deviations require explicit justification
- **MAY** — permitted; no preference either way

## Conventions

Rules that apply at the statement or expression level across all modules.

- [Imports](conventions/imports.md) — what to qualify, how to alias, import ordering
- [Exports](conventions/exports.md) — explicit export lists, grouping, ordering
- [Naming](conventions/naming.md) — types, constructors, functions, fields, type classes, type variables
- [Errors](conventions/errors.md) — `Either`, exceptions, custom error types
- [Deriving](conventions/deriving.md) — strategy annotations (`stock`, `newtype`, `anyclass`, `via`)
- [Formatting](conventions/formatting.md) — formatter, `do` vs applicative, `where` vs `let`
- [Documentation](conventions/documentation.md) — Haddock rules and format
- [Language Extensions](conventions/language-extensions.md) — global defaults and per-file extensions

## Patterns

Named, reusable module structures. Each pattern is optional — apply it when the described situation matches; do not force it onto code that does not need it. Each pattern document is self-contained.

- [Aggregator Namespace](patterns/aggregator-namespace.md) — a namespace of implementors with a re-exporting root
- [Execution Capability](patterns/execution-capability.md) — type classes that lift an abstraction into arbitrary execution contexts
- [Port](patterns/port.md) — type classes as interfaces between logic and infrastructure
- [Preludes](patterns/preludes.md) — domain-specific prelude modules for aggregator namespaces
- [Language Module](patterns/language-module.md) — a closed set of domain types related by typeclasses

## Architecture

- [Logic Layer](architecture/logic-layer.md) — how to structure the pure logic layer of an application

## Glossary

**Topic of a module** — the abstraction a module primarily defines or implements. Imports whose names form the vocabulary of that abstraction are imported unqualified.

**Aggregator namespace** — a module namespace where each sub-module implements a type class and the root re-exports a summary of all sub-modules.

**Port** — a type class defined in the logic layer that specifies what the logic needs from infrastructure. Instances live in infrastructure, never in logic.

**Execution Capability** — a type class that grants an execution context the ability to run a specific abstraction (e.g., `RunsSession`).

**Component** — a cohesive unit in the logic DAG corresponding to one domain concept. May define types, Ports, and pure functions.

**Language Module** — a module that exports a closed set of domain types together with the typeclasses and instances that relate them.

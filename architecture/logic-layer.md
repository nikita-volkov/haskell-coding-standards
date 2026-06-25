# Architecture: Logic Layer

## Purpose

The logic layer contains the business logic of an application as a pure, infrastructure-free computation. It has no direct dependency on IO, databases, HTTP clients, or filesystems. Infrastructure depends on logic; logic does not depend on infrastructure.

## The DAG Model

The logic layer is a directed acyclic graph (DAG) of components. Each node in the DAG is a **component** — a cohesive module or small namespace corresponding to one domain concept.

Components may depend on other components, but the dependency graph MUST be acyclic. The Haskell module system enforces this.

## What a Component Contains

A component may define any combination of:

- **Domain types** — data structures that represent concepts in the domain
- **Ports** — type classes specifying what the component needs from infrastructure (see [Port pattern](../patterns/port.md))
- **Pure functions and combinators** — functions over the component's types with no monadic effects
- **Logic functions** — monadic programs constrained by Port type classes

There is no mandatory internal split between these: a component is a single module unless it grows large enough to warrant internal helper sub-modules (see [Module Organisation](#module-organisation) below).

## Component Boundaries

A component boundary corresponds to one **domain concept** — a cohesive idea from the problem domain with a name in the ubiquitous language.

When defining a new Port, treat it as a signal: the capability it abstracts may represent a domain concept that warrants its own component.

## Ports and Shared Capabilities

If two components need the same infrastructure capability, the Port belongs in a shared component that both depend on. The shared component is a lower node in the DAG.

```
Logic/
  Fs.hs            -- ManagesFiles port + FS utilities (shared)
  Reporting.hs     -- Warns port (owned here)
  Migrations.hs    -- depends on Fs, Reporting
  Generate.hs      -- depends on Fs, Reporting
```

## Pure vs. Monadic

The boundary between pure and monadic code is the type: if a function's signature contains `m` constrained by a Port, it is a logic function. If it is fully pure, it is a combinator.

Both belong in the same component. There is no prescribed split between them at the module level.

## Module Organisation

### One surface module per component

Each component has one public-facing module. That module is the component's entire API. Callers import only this module.

### Private helper sub-modules

When a component grows complex internally, implementation details MAY be extracted into private sub-modules. A private sub-module is used by exactly one module: its parent.

```
Logic/FileName.hs          -- public surface; all callers import this
Logic/FileName/Parsers.hs  -- private; only imported by Logic/FileName.hs
```

Seeing `Logic/FileName/Parsers` in an import is a violation — only `Logic/FileName` may import it. This is enforced by convention, not the build system.

## Relationship to Infrastructure

Infrastructure modules:
- Implement the logic layer's Port type classes for the application monad
- MUST NOT define domain types — those belong in logic
- MAY import any logic module; logic MUST NOT import infrastructure

```
Infrastructure/
  Fs.hs          -- implements ManagesFiles for AppM
  Reporting.hs   -- implements Warns for AppM
  Db.hs          -- implements database Ports for AppM
```

## What Does Not Belong Here

The logic layer MUST NOT contain:
- Direct `IO` actions (except as abstract effects through Ports)
- Imports of infrastructure libraries (`hasql`, `http-client`, `directory`)
- Application configuration or wiring

## Example Structure

```
Logic/
  Fs.hs                    -- ManagesFiles port, path utilities
  Reporting/
    Port.hs                -- Warns port
    Types/
      Report.hs            -- Report type
  Schema.hs                -- schema domain types and logic
  Migrations.hs            -- migration logic; depends on Fs, Reporting
  Generate.hs              -- code generation logic; depends on Fs, Reporting, Schema
```

## Related

- [Port Pattern](../patterns/port.md) — how infrastructure interfaces are defined
- [Execution Capability Pattern](../patterns/execution-capability.md) — how library abstractions are made context-polymorphic

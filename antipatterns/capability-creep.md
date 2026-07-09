# Anti-pattern: Capability Creep

## Symptom

A [Domain Language](../patterns/domain-language.md) or [Intermediate Representation](../patterns/intermediate-representation.md) module accumulates effectful functions: database access, HTTP calls, file I/O, or other runtime concerns.

## Problem

The module is supposed to define a language — pure expressions in a closed vocabulary. Effectful operations couple the language to a runtime and make it harder to test, interpret, or reuse the expressions.

## Resolution

- Move effectful operations to a capability, service, or Port module.
- Keep the module pure; it defines what can be said, not how it is executed.

## See Also

- [Domain Language](../patterns/domain-language.md)
- [Intermediate Representation](../patterns/intermediate-representation.md)
- [Port](../patterns/port.md)
- [Execution Capability](../patterns/execution-capability.md)

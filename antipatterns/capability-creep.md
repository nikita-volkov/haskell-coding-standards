# Anti-pattern: Capability Creep

## Symptom

A [Language Module](../patterns/language-module.md) accumulates effectful functions: database access, HTTP calls, file I/O, or other runtime concerns.

## Problem

The module is supposed to define a language — pure domain expressions. Effectful operations couple the language to a runtime and make it harder to test, interpret, or reuse the expressions.

## Resolution

- Move effectful operations to a capability, service, or Port module.
- Keep the Language Module pure; it defines what can be said, not how it is executed.

## See Also

- [Language Module](../patterns/language-module.md)
- [Port](../patterns/port.md)
- [Execution Capability](../patterns/execution-capability.md)

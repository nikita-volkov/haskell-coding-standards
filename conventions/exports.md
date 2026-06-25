# Exports

## Explicit Export Lists

Every public module MUST have an explicit export list. Implicit export of all definitions (`module Foo where` with no list) is prohibited for public modules.

A public module is one that forms part of the project's public API or is imported by callers outside its component.

Small internal modules — for example, private helper sub-modules used by exactly one parent — MAY omit the export list. For those modules, an explicit export list is still encouraged when it meaningfully clarifies the API.

An explicit export list is the module's public API declaration. It makes refactoring safe (internal helpers can be renamed freely), communicates intent to readers, and prevents accidental exposure.

## Grouping

Exports MUST be grouped by concept using blank lines between groups. Within a group, ordering is by logical relationship, not alphabetically.

There is no prescribed group structure — group by whatever clustering communicates the API most clearly to a reader.

```haskell
module MyApp.Session
  ( -- * Session type
    Session,
    runSession,

    -- * Configuration
    SessionConfig (..),
    defaultSessionConfig,
    withTimeout,

    -- * Errors
    SessionError (..),
  )
where
```

## What to Export

Export the minimum surface needed for callers to use the module correctly.

- Internal helpers, implementation details, and intermediate types MUST NOT be exported unless callers genuinely need them.
- Types whose constructors callers must not use directly SHOULD be exported without constructors (`Session`, not `Session (..)`).
- Types whose constructors callers need MUST export them explicitly (`SessionError (..)`, not just `SessionError`).

# Pattern: Port

## Intent

A type class defined in the logic layer that specifies what the logic needs from infrastructure. Instances live in infrastructure modules. Logic depends on the interface; infrastructure depends on logic. The dependency is inverted.

## When to Use

Use when logic needs to perform an effect that must not create a direct dependency on infrastructure: file I/O, database access, HTTP calls, logging, external services.

Do not confuse with [Execution Capability](execution-capability.md) — Ports define what logic needs from outside; Execution Capabilities connect concrete abstractions together within a library.

## Adoption

This pattern is optional. Use it when the dependency-inversion benefits outweigh the indirection. Do not force Ports into code that has no infrastructure boundary, and do not retrofit an entire existing codebase solely to satisfy this pattern.

## Naming

MUST follow the capability naming convention: third-person singular present tense verb phrase.

- Single-operation Ports: name the operation — `Warns`, `ExecutesMigrations`, `ReadsConfig`
- Multi-operation Ports: use `Manages` as the catch-all verb for broad access — `ManagesFiles`, `ManagesCache`, `ManagesConnections`

`Manages` signals "full access to this infrastructure boundary" without listing operations. Reserve specific verbs for genuinely single-operation Ports.

## Structure

Each Port is a module in the logic layer, named `<Component>.Port` or as a submodule when the component is large.

```haskell
-- Logic/Fs/Port.hs (or Logic/Fs.hs if the component is small)
module Logic.Fs.Port
  ( ManagesFiles (..)
  ) where

-- | Capability to read, write, and list filesystem paths.
class (Monad m) => ManagesFiles m where
  -- | Read the full contents of a file.
  readFile :: Path -> m Text
  -- | Write content to a file, creating or overwriting it.
  writeFile :: Path -> Text -> m ()
  -- | List immediate children of a directory.
  listDir :: Path -> m [Path]
```

## Grouping Operations

Operations belong in one Port when they share the same infrastructure boundary AND no meaningful partial implementation exists.

**Test:** can you imagine a real implementation (mock, test double, restricted environment) that provides some operations but not others? If yes, split into separate Ports.

```haskell
-- Split: read-only and read-write are meaningfully different
class (Monad m) => ReadsFiles m where
  readFile :: Path -> m Text
  listDir  :: Path -> m [Path]

class (ReadsFiles m) => WritesFiles m where
  writeFile :: Path -> Text -> m ()
```

```haskell
-- Grouped: all filesystem operations always come together in practice
class (Monad m) => ManagesFiles m where
  readFile  :: Path -> m Text
  writeFile :: Path -> Text -> m ()
  listDir   :: Path -> m [Path]
```

## Where Instances Live

Port instances MUST live in infrastructure modules, never in logic. The infrastructure module imports the Port type class from logic and provides a concrete implementation.

```haskell
-- Infrastructure/Fs.hs
module Infrastructure.Fs where

import Logic.Fs.Port (ManagesFiles (..))

instance ManagesFiles AppM where
  readFile path = liftIO (IO.readFile path)
  writeFile path content = liftIO (IO.writeFile path content)
  listDir path = liftIO (Directory.listDirectory path)
```

## Relationship to Logic Components

Ports belong to the logic component that defines the need. If two components need the same infrastructure capability, the Port belongs in a shared lower-level component that both depend on.

```
Logic/
  Fs/
    Port.hs        -- ManagesFiles — defined here, shared by any component needing FS
  Reporting/
    Port.hs        -- Warns — defined here, used only by reporting logic
  Migrations/
    Port.hs        -- ExecutesMigrations
```

See [Logic Layer Architecture](../architecture/logic-layer.md) for how components compose.

## Related

- [Execution Capability](execution-capability.md) — a different pattern for a different purpose
- [Logic Layer](../architecture/logic-layer.md) — where Ports fit in the overall architecture
- [Naming](../conventions/naming.md) — type class naming rules

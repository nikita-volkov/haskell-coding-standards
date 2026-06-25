# Pattern: Execution Capability

## Intent

A type class that grants an execution context the ability to run a specific abstraction. Allows writing context-polymorphic code that works across `Session`, `Transaction`, `Pipeline`, and custom application monads without duplicating logic.

## When to Use

Use when:
- You have a concrete abstraction (`Session`, `Pipeline`, `Transaction`) that needs to be executable in multiple different contexts
- You want to write code that is polymorphic over the execution context
- You need to integrate a library abstraction directly into a custom application monad

Do not confuse with the [Port](port.md) pattern — Execution Capabilities connect concrete abstractions together; Ports define what logic needs from infrastructure.

## Naming

MUST follow the capability naming convention: third-person singular present tense verb phrase.

```haskell
class (Functor f) => RunsSession f where ...
class (Functor f) => RunsPipeline f where ...
class (Functor f) => RunsStatement f where ...
class (Functor f) => RunsScript f where ...
```

## Structure

The type class and all its instances live in the same module — the module that owns the abstraction being lifted.

```haskell
module HasqlDev
  ( RunsSession (..)
  , RunsPipeline (..)
  , RunsStatement (..)
  , runStatementByParams
  ) where

-- | Capability of a functor to execute sessions.
class (Functor f) => RunsSession f where
  -- | Lift a session into the context of the functor.
  runSession :: Session.Session a -> f a

-- Concrete instances
instance RunsSession Session.Session where
  runSession = id

instance RunsSession (ReaderT Connection.Connection (ExceptT SessionError IO)) where
  runSession session = ReaderT \connection ->
    ExceptT (Connection.use connection session)

-- Transformer lifting instances
instance (RunsSession f) => RunsSession (ReaderT r f) where
  runSession session = ReaderT \_ -> runSession session

instance (RunsSession f) => RunsSession (StateT s f) where
  runSession session = StateT \s -> fmap (,s) (runSession session)
```

## Key Properties

**Concrete instances** enumerate the specific execution contexts the abstraction can run in: the abstraction itself (`id`), connection-based contexts, pool-based contexts.

**Transformer lifting instances** propagate the capability through standard monad transformer stacks (`ReaderT`, `StateT`, `ExceptT`, `WriterT`). These are mechanical — every capability class should include them.

**Utility functions** built on the class are defined in the same module and exported alongside it.

```haskell
-- | Execute a statement implicitly determined by its parameters.
runStatementByParams
  :: (RunsStatement f, IsStatement params)
  => params -> f (IsStatement.Result params)
runStatementByParams params =
  runStatement IsStatement.statement params
```

## Usage

Callers constrain on the capability rather than a concrete type:

```haskell
fetchUser :: (RunsStatement m) => UserId -> m (Maybe User)
fetchUser userId = runStatementByParams (GetUser userId)
```

This function works in any monad that has `RunsStatement` — a test monad, a production session, a transaction, or a custom application monad.

## Related

- [Port](port.md) — a different pattern; Ports define what logic needs from infrastructure, not how to lift a concrete abstraction
- [Naming](../conventions/naming.md) — type class naming rules

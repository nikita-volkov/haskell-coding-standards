# Imports

## Qualification

**MUST qualify** all imports except those that are the *topic* of the module.

The **topic** of a module is the abstraction it primarily defines or implements — the vocabulary you are working *in*, not a tool you reach *for*. If removing a module's imports would make the file's purpose unclear, it is the topic.

Examples of topic imports:
- A custom or standard prelude — always unqualified
- `Test.QuickCheck` in a module that defines `Arbitrary` instances — QuickCheck is the medium
- `MyApp.Domain.Types` in a module that works entirely with those domain types

Everything else MUST be imported qualified.

## Aliasing

### Local modules (same project)

The default alias is the module name with the project root namespace prefix stripped.

```haskell
-- Project root: MyApp
import qualified MyApp.Domain.Types as Domain.Types
import qualified MyApp.Domain.Types.Album as Domain.Types.Album
```

When importing a parent namespace and its sub-modules, the child alias SHOULD be derived from the parent alias by appending the child component. This preserves the hierarchy at call sites.

```haskell
import qualified MyApp.Statements as Statements
import qualified MyApp.Statements.InsertAlbum as Statements.InsertAlbum

-- Call site: the type and its module share the same prefix
album :: Statements.InsertAlbum
album = Statements.InsertAlbum.fromParams params
```

A shorter, conventional alias is permitted when the full name is long and the short form is unambiguous within the module.

```haskell
import qualified MyApp.PoloniexWsV3Protocol as Protocol
```

### External packages — well-named

Packages whose top-level component is a meaningful package name (e.g., `Hasql`, `Aeson`, `PostgresqlTypes`) MUST be imported with no alias — the full module path is used at call sites.

```haskell
import qualified Hasql.Statement
import qualified Hasql.Decoders
import qualified PostgresqlTypes.Int4

-- Call site
stmt :: Hasql.Statement.Statement a b
val = PostgresqlTypes.Int4.fromInt32 42
```

### External packages — legacy-prefixed

Modules under the legacy Haskell hierarchy prefixes (`Data`, `Control`, `System`, `Network`, `Text`, `Language`, `Numeric`, `Foreign`) SHOULD be aliased by their conventional short name. Often this is the remainder after stripping the prefix, but well-known modules MAY use an even shorter alias when it is unambiguous and conventional in the project.

```haskell
import qualified Data.Attoparsec.Text as Attoparsec.Text
import qualified Data.Map.Strict as Map
import qualified Data.Map.Lazy as Map.Lazy
import qualified Control.Concurrent.STM as STM
import qualified Data.Vector as Vector
import qualified Network.WebSockets as Ws
```

The legacy prefixes carry no semantic information — they are historical artefacts of pre-modern Hackage and SHOULD NOT appear at call sites. If both `Data.Map.Strict` and `Data.Map.Lazy` appear in the same module, use `Map.Strict` and `Map.Lazy` to keep them distinct.

## Individual Member Imports

No rule. Individual member imports (e.g., `import Foo (bar, baz)`) are neither required nor prohibited. Use judgment.

## Ordering

Imports SHOULD be ordered: unqualified first, then qualified, alphabetically within each group. A blank line MAY separate the groups.

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

MUST be aliased by stripping the project root namespace prefix.

```haskell
-- Project root: MyApp
import qualified MyApp.Domain.Types as Domain.Types
import qualified MyApp.Domain.Types.Album as Domain.Types.Album
```

When importing a parent namespace and its sub-modules, the child alias MUST be derived from the parent alias by appending the child component. This preserves the hierarchy at call sites.

```haskell
import qualified MyApp.Statements as Statements
import qualified MyApp.Statements.InsertAlbum as Statements.InsertAlbum

-- Call site: the type and its module share the same prefix
album :: Statements.InsertAlbum
album = Statements.InsertAlbum.fromParams params
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

Modules under the legacy Haskell hierarchy prefixes (`Data`, `Control`, `System`, `Network`, `Text`, `Language`, `Numeric`, `Foreign`) MUST be aliased by stripping the prefix.

```haskell
import qualified Data.Attoparsec.Text as Attoparsec.Text
import qualified Data.Map.Strict as Map.Strict
import qualified Control.Concurrent.STM as STM
import qualified Data.Vector as Vector
```

The legacy prefixes carry no semantic information — they are historical artefacts of pre-modern Hackage and MUST NOT appear at call sites.

## Individual Member Imports

No rule. Individual member imports (e.g., `import Foo (bar, baz)`) are neither required nor prohibited. Use judgment.

## Ordering

Imports SHOULD be ordered: unqualified first, then qualified, alphabetically within each group. A blank line MAY separate the groups.

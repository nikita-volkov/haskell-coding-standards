# Pattern: Aggregator Namespace

## Intent

A module namespace where each sub-module implements a specific type class and exports an identifying data structure named after the module. The root module re-exports a summary of all sub-modules.

## When to Use

Use when you have a set of uniform implementors of a single type class — each implementor is a distinct named entity, and callers need to refer to the collection as a whole.

Examples:
- A set of database statement modules, each implementing `IsStatement`
- A set of scalar type adapters, each implementing `IsScalar`
- A set of codec modules, each implementing `IsCodec`

## Structure

```
Namespace/           -- root module (aggregator)
Namespace/Thing1.hs  -- sub-module implementing the type class
Namespace/Thing2.hs  -- sub-module implementing the type class
Namespace/Thing3.hs  -- sub-module implementing the type class
```

## Variants

### Variant 1 — Types-Only Root

Use when sub-modules export an open-ended set of definitions determined by the subject of each module. Function names are short, type-scoped, and would collide if re-exported from the root.

**Root module:** re-exports type names only — no constructors, no functions.

**Sub-modules:** export the type plus whatever type-specific utilities are natural for that subject. Function names are short (`fromInt32`, `toText`) and are always used qualified.

**Example:** `postgresql-types` — each sub-module (`Int4`, `Text`, `Uuid`) exports type-specific conversion functions. The root exports only the type names so callers can write type signatures.

```haskell
-- Root: PostgresqlTypes
module PostgresqlTypes
  ( module PostgresqlTypes.Int4   -- re-exports Int4 type only
  , module PostgresqlTypes.Text
  , module PostgresqlTypes.Uuid
  ) where
```

Callers import the root for type signatures and sub-modules for functions:

```haskell
import qualified PostgresqlTypes
import qualified PostgresqlTypes.Int4

field :: PostgresqlTypes.Int4
field = PostgresqlTypes.Int4.fromInt32 42
```

### Variant 2 — Full Re-Export Root

Use when sub-modules export a **bounded, predictable set of definitions** determined by the type class contract rather than the subject. Names are globally unambiguous because they are prefixed with (or suffixed by) the module name.

**Root module:** re-exports everything from all sub-modules.

**Sub-modules:** become hidden modules — they are not part of the public API and callers never import them directly. They export a fixed, pattern-defined set of types and instances. All exported names MUST be globally unambiguous without module context.

**Primary type naming rule:** The primary type exported by each sub-module MUST be named after the module. Additional types follow the module name as a prefix.

```haskell
-- Sub-module: Statements/InsertAlbum.hs
module MyApp.Statements.InsertAlbum where

data InsertAlbum = InsertAlbum      -- named after the module
  { name :: Text
  , released :: Date
  }

type InsertAlbumResult = InsertAlbumResultRow   -- module name as prefix

newtype InsertAlbumResultRow = InsertAlbumResultRow
  { id :: Int64 }

instance IsStatement InsertAlbum where
  type Result InsertAlbum = InsertAlbumResult
  statement = ...
```

```haskell
-- Root: Statements.hs
module MyApp.Statements
  ( module MyApp.Statements.InsertAlbum
  , module MyApp.Statements.UpdateAlbum
  , module MyApp.Statements.DeleteAlbum
  ) where

import MyApp.Statements.DeleteAlbum
import MyApp.Statements.InsertAlbum
import MyApp.Statements.UpdateAlbum
```

Callers import only the root:

```haskell
import qualified MyApp.Statements as Statements

-- InsertAlbum type and its functions are available under Statements.*
params :: Statements.InsertAlbum
params = Statements.InsertAlbum { name = "Kind of Blue", released = ... }
```

## Choosing a Variant

| Question | Answer → Variant |
|---|---|
| Are sub-module function names short and type-scoped? | 1 |
| Are sub-module exports bounded by the type class contract? | 2 |
| Would re-exporting functions from the root cause name collisions? | 1 |
| Are all exported names already globally unambiguous? | 2 |

## Suffix Rules for Variant 2

When a specific Aggregator Namespace pattern is documented (e.g., the `IsStatement` pattern), the suffix scheme for additional types is defined per-pattern, not globally. The only global rule is: all exported names MUST use the module name as a prefix. See the pattern-specific documentation for suffix conventions.

## Related

- [Preludes](preludes.md) — when an Aggregator Namespace pattern exists, a dedicated prelude for its sub-modules is warranted
- [Imports](../conventions/imports.md) — aliasing rules for namespace root and sub-module imports

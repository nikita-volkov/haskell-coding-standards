# Pattern: Domain Preludes

## Intent

A dedicated prelude module that provides the complete unqualified vocabulary for a specific class of modules — typically the sub-modules of an [Aggregator Namespace](aggregator-namespace.md). Each module imports exactly one prelude.

## When to Use

A prelude is warranted when an Aggregator Namespace pattern exists and its sub-modules share a common vocabulary. The prelude captures that repeated pattern: the same unqualified imports that every sub-module would otherwise repeat.

If only one or two modules would benefit, use explicit imports instead. A prelude earns its existence through visible repetition.

## Adoption

This pattern is optional. Create a prelude only when an Aggregator Namespace has enough sub-modules that the repeated unqualified imports are clearly visible. Do not create a prelude for a handful of files, and do not retrofit existing code solely to introduce one.

## Naming

Preludes live in the `Preludes` namespace (plural). Each prelude is named after the pattern or namespace it serves.

```
Preludes/
Preludes/Statement.hs
Preludes/ScalarCodec.hs
Preludes/Test.hs
```

The base prelude (universal things) lives at `Preludes` or a project-specific root.

## Structure

A prelude re-exports everything that sub-modules need unqualified: the base prelude, the type class being implemented, its supporting types, and pervasive utilities for that domain.

```haskell
-- Preludes/Statement.hs
module Preludes.Statement
  ( module Preludes
  , module Hasql.Statement
  , module Hasql.Mapping.IsStatement
  , IsScalar.encoder
  , IsScalar.decoder
  ) where

import Preludes
import qualified Hasql.Mapping.IsScalar as IsScalar
import Hasql.Mapping.IsStatement
import Hasql.Statement
```

## One Prelude Per Module

Each module MUST import exactly one prelude. Importing multiple preludes is a signal that the module spans multiple domains and should be split.

```haskell
-- Correct: one prelude
module MyApp.Statements.InsertAlbum where

import MyApp.Preludes.Statement

-- Prohibited: multiple preludes
import MyApp.Preludes.Statement
import MyApp.Preludes.Http
```

## What Belongs in a Prelude

Include in a domain prelude:
- The base prelude (re-exported)
- The type class the namespace implements (unqualified — it is the topic)
- Types that appear in every sub-module's signatures
- Pervasive utilities for that domain

Exclude:
- Anything that would be used qualified at call sites
- Infrastructure-specific modules
- Anything used by fewer than most sub-modules

## Relationship to Aggregator Namespace

The prelude and the aggregator namespace are paired: when you create an Aggregator Namespace, consider creating a companion prelude. The prelude serves the sub-modules; the aggregator root serves the callers.

```
Statements/         ← aggregator namespace (serves callers)
Statements/InsertAlbum.hs
Statements/UpdateAlbum.hs
Preludes/Statement.hs  ← domain prelude (serves sub-module authors)
```

## Related

- [Aggregator Namespace](aggregator-namespace.md) — the pattern that triggers prelude creation
- [Imports](../conventions/imports.md) — topic imports and the unqualified rule

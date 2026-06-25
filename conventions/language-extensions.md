# Language Extensions

## Global Default Extensions

The following extensions MUST be enabled in `default-extensions` in the Cabal file for all projects.

```cabal
default-extensions:
  ApplicativeDo
  Arrows
  BangPatterns
  BinaryLiterals
  BlockArguments
  ConstraintKinds
  DataKinds
  DefaultSignatures
  DeriveDataTypeable
  DeriveFoldable
  DeriveFunctor
  DeriveGeneric
  DeriveTraversable
  DerivingVia
  DuplicateRecordFields
  EmptyDataDecls
  FlexibleContexts
  FlexibleInstances
  FunctionalDependencies
  GADTs
  GeneralizedNewtypeDeriving
  HexFloatLiterals
  ImportQualifiedPost
  LambdaCase
  LiberalTypeSynonyms
  MultiParamTypeClasses
  MultiWayIf
  NamedFieldPuns
  NoFieldSelectors
  NoMonomorphismRestriction
  NumericUnderscores
  OverloadedLabels
  OverloadedRecordDot
  OverloadedStrings
  ParallelListComp
  PatternGuards
  PatternSynonyms
  RankNTypes
  RecordWildCards
  ScopedTypeVariables
  StandaloneDeriving
  StrictData
  TupleSections
  TypeApplications
  TypeFamilies
  TypeOperators
  ViewPatterns
```

### Notable semantics

**`NoFieldSelectors`** — record fields are not injected into the global namespace as selector functions. Fields are accessed via `OverloadedRecordDot` (`.` syntax) and used in record construction/update syntax. Eliminates field name collisions without prefixing.

**`OverloadedRecordDot`** — enables `value.field` syntax for record access. Required companion to `NoFieldSelectors`.

**`StrictData`** — all record fields are strict by default. Lazy fields (`~`) MUST include a comment explaining why laziness is required.

**`DuplicateRecordFields`** — permits the same field name in multiple record types. Kept alongside `NoFieldSelectors` for compatibility.

## Per-File Extensions

The following extensions MUST be enabled per-file only, using `{-# LANGUAGE ... #-}` pragmas. They MUST NOT be in `default-extensions`.

```haskell
{-# LANGUAGE TemplateHaskell #-}
{-# LANGUAGE QuasiQuotes #-}
{-# LANGUAGE UndecidableInstances #-}
```

Their presence in a file is a visible, deliberate signal: `TemplateHaskell` and `QuasiQuotes` affect compile times and enable code generation; `UndecidableInstances` relaxes type checker termination checks. Both warrant explicit opt-in at the file level.

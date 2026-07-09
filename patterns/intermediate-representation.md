# Pattern: Intermediate Representation

## Intent

A module that exports a closed set of types together with the typeclasses and instances that relate them, representing one stage of a data-conversion pipeline — the compiler sense of "intermediate representation" (AST, Core, ANF, and similar). The types form the vocabulary of that stage; the typeclasses form a partial grammar for constructing, transforming, and categorizing expressions within it.

This is the same shape as [Domain Language](domain-language.md) — closed world, partial grammar, build-first usage — applied to a pipeline stage instead of to the business domain. See that pattern for the shared structural properties; this document covers what's specific to pipeline stages.

## When to Use

Use when:

- Your system is a pipeline of conversions between representations (e.g. a compiler, a parser, a codegen tool).
- Each stage has its own closed set of concepts, distinct from the stages before and after it.
- Consumers of a given stage primarily **build** or **traverse** expressions in that stage's vocabulary before lowering to the next stage.

Do not use for the domain types business logic operates on — that's [Domain Language](domain-language.md). Do not use for wire-facing types at the boundary of the system — that's [Transport Representation](transport-representation.md).

## Structure

Each stage is its own module — a self-contained closed vocabulary, structured exactly like a Domain Language module:

```
Compiler/
  Ast.hs     -- surface syntax; what the parser produces
  Core.hs    -- desugared, name-resolved; what typechecking operates on
  Anf.hs     -- administrative normal form; what codegen consumes
```

```haskell
-- Compiler/Core.hs
module Compiler.Core
  ( Expr (..)
  , Decl (..)
  , fromAst
  ) where

data Expr
  = Var Name
  | App Expr Expr
  | Lam Name Expr
  | Lit Literal

data Decl = Decl
  { name :: Name
  , body :: Expr
  }

-- | Lower from the previous stage.
fromAst :: Ast.Expr -> Expr
fromAst = \case
  Ast.Var name -> Var name
  Ast.App f x -> App (fromAst f) (fromAst x)
  Ast.Lam name body -> Lam name (fromAst body)
  Ast.Lit lit -> Lit lit
  Ast.Paren inner -> fromAst inner  -- parens are dropped: lossy
```

The lowering function lives in the module that owns the **target** stage — `Core.hs` depends on `Ast.hs`, not the other way around, keeping the pipeline a DAG with no back-edges.

## Stage-to-Stage Conversion

Lowering is usually a **total, lossy** conversion: every well-formed input at stage N produces a stage-N+1 value (lowering doesn't fail — a separate typechecking or validation pass does), but round-tripping back to stage N loses information (source spans, parens, surface sugar). This is the [Lawful Conversion](lawful-conversion.md) `IsMany` shape, not `IsSome` or `Is`.

Not every lowering function needs the typeclass — a plain `fromAst :: Ast.Expr -> Core.Expr` is fine for a one-off pipeline. Reach for `IsMany` when multiple stages, or multiple front-ends feeding the same stage, make the relationship worth naming generically. See [Lawful Conversion](lawful-conversion.md) for the full guidance.

## Key Properties

Inherits all of [Domain Language](domain-language.md)'s key properties (closed world, partial grammar, build-first usage, no effects) applied per-stage, plus:

**One direction of flow.** Stages form a DAG: later stages depend on earlier ones for their lowering functions, never the reverse.

**Lossy by default.** Unlike Domain Language, where types are typically expected to round-trip, IR stages routinely discard information on the way down the pipeline. Don't force losslessness where the domain doesn't need it.

## Misclassifications

Applies the same misclassifications as Domain Language — see [Anti-patterns: Domain Language & Intermediate Representation](../antipatterns/README.md#domain-language--intermediate-representation). Additionally:

- Splitting a stage into a module before its vocabulary is large enough to need one is premature — small pipelines with 2-3 tightly-coupled stages may keep them in one module until that stops being true.

## Related

- [Domain Language](domain-language.md) — the same shape, applied to the business domain instead of a pipeline stage
- [Transport Representation](transport-representation.md) — for wire-facing types, a different and simpler pattern
- [Lawful Conversion](lawful-conversion.md) — for the stage-to-stage lowering relation

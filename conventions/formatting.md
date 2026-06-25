# Formatting

## Formatter

MUST use [Ormolu](https://github.com/tweag/ormolu) with zero configuration. What Ormolu produces is the style. No exceptions, no overrides.

Run Ormolu on all `.hs` files before committing. CI MUST fail on unformatted files.

## `do` Notation vs Applicative Style

Use **applicative style** (`<$>`, `<*>`) for short expressions that fit on one or two lines with no dependencies between effects.

Use **`do` notation** for anything spanning multiple lines.

```haskell
-- Applicative — short, no inter-effect dependencies
User <$> getName <*> getAge <*> getEmail

-- do — multiple lines or inter-effect dependencies
do
  config <- loadConfig path
  let adjusted = adjustConfig config
  user <- fetchUser adjusted.userId
  pure (config, user)
```

The `ApplicativeDo` extension is enabled globally, so `do` notation desugars to applicative automatically where possible.

## `where` vs `let`

**`where`** — for named sub-computations that support the main expression: helpers, deconstructed pieces, named constants. The main expression appears first; details follow below.

```haskell
runQuery conn =
  execute conn sql params
  where
    sql = "SELECT ..."
    params = encodeParams conn
```

**`let` in `do` blocks** — for intermediate values in a sequence of effectful steps.

```haskell
do
  raw <- fetchRaw url
  let parsed = parse raw
  store parsed
```

**`let` expressions** — use sparingly. Prefer `where` for anything that would create a nested `let ... in` chain. In `do` blocks, `let x = ...` (no `in`) is the standard form.

MUST NOT use `where` inside a `do` block for values that are logically sequential — use `let` instead.

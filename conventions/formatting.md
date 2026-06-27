# Formatting

## Formatter

SHOULD use a formatter. [Ormolu](https://github.com/tweag/ormolu) is the preferred formatter and is recommended for new projects. If a project already uses a different formatter, continue using it consistently. What the chosen formatter produces is the style for that project.

Run the project's formatter on changed `.hs` files before committing. CI MAY enforce formatting.

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

## Helper Functions

Every top-level definition joins the module's namespace — visible to imports and implicitly claiming to be a standalone concept. Helper functions used by only one caller do not earn that status and MUST be placed in that caller's `where` block instead.

This keeps the module's real surface area apparent and prevents the namespace from filling with names that are meaningless outside their one use site.

```haskell
-- Bad: top-level helpers that belong to one caller
renderPage :: Config -> Html
renderPage config = header <> body <> footer
  where
    header = renderHeader config
    body = renderBody config
    footer = renderFooter config

renderHeader :: Config -> Html  -- top-level, but only used by renderPage
renderHeader config = ...

renderBody :: Config -> Html
renderBody config = ...

renderFooter :: Config -> Html
renderFooter config = ...

-- Good: helpers scoped to their caller
renderPage :: Config -> Html
renderPage config = renderHeader config <> renderBody config <> renderFooter config
  where
    renderHeader cfg = ...
    renderBody cfg = ...
    renderFooter cfg = ...
```

Promote a helper to the top level only when it is used by more than one top-level definition or is exported.

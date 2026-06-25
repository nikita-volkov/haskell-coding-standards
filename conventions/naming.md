# Naming

## Types

MUST use `PascalCase`. Names are descriptive noun phrases — no role suffixes (`Enum`, `Record`, `Type`).

Acronyms MUST capitalise only the first letter: `JsonAst`, `HttpRequest`, `HtmlParser`. Adjacent acronyms (`JSONAST`) are ambiguous and prohibited.

## Constructors

### Product types

The constructor of a type with exactly one constructor MUST share the type name exactly.

```haskell
data Album = Album { ... }
newtype AlbumId = AlbumId Int64
data Wrapped = Wrapped Something OtherThing
```

The `Mk` prefix and all variants (`mkAlbum`, `MkAlbum`) are prohibited.

### Sum types

Constructors of types with multiple constructors MUST be suffixed with the full type name. The discriminating part leads; the type name closes.

```haskell
data SessionError
  = AcquisitionTimeoutSessionError
  | ClientSessionError
  | QuerySessionError
```

Shortened suffixes are prohibited. The full type name MUST always be used.

## Record Fields

MUST be unprefixed. Field names MUST NOT be prefixed with the type name or any abbreviation.

Requires `NoFieldSelectors` and `OverloadedRecordDot` — see [Language Extensions](language-extensions.md).

```haskell
data Album = Album
  { name :: Text
  , released :: Date
  , format :: AlbumFormat
  }

-- Access via record dot syntax
album.name
```

## Functions

MUST use `camelCase`.

### Conversion functions

Functions that convert *to* or *from* the module's subject type MUST use `from`/`to` relative to that subject. The module context provides the implicit subject.

```haskell
-- In PostgresqlTypes.Int4:
fromInt32 :: Int32 -> Int4
toInt32   :: Int4 -> Int32
```

Used at call sites always qualified (`PostgresqlTypes.Int4.fromInt32`), making the direction unambiguous.

When converting between two types neither of which owns the function, MUST use the directional form `fooToBar`. SHOULD instead place the function in one of the types' own modules.

### Predicates

Boolean functions MUST use `is` or `has` as a prefix. No other prefix is permitted for boolean-returning functions (`check`, `test`, bare adjectives).

- `is` — tests a property or state of the value itself: `isEmpty`, `isValid`
- `has` — tests for presence of a component or child: `hasResults`, `hasTracklist`

### Scoped-resource functions

Functions following the bracket pattern (acquire → use → release) MUST use the `with` prefix: `withConnection`, `withTimeout`, `withFile`.

`with` is reserved exclusively for this pattern. MUST NOT be used for record modifiers or other purposes.

## Type Variables

- Use community-standard single letters for structural/abstract roles: `f` (functor/applicative), `m` (monad), `a`/`b` (generic values), `e` (error), `r` (reader environment)
- Use descriptive names when the type variable carries domain meaning: `params`, `result`, `key`, `value`

## Type Classes

Type class names MUST be predicates — they qualify a type, they do not name a concept.

**Prohibited:** noun names (`Vector`, `Container`), `-able` suffix (`Serializable`).

### Identity and nature predicates

MUST use the `Is` prefix: `IsStatement`, `IsScalar`, `IsValid`.

Reads as: "this type is a statement."

### Capability predicates

MUST use third-person singular present tense verb phrases: `RunsSession`, `ExecutesMigrations`, `ManagesFiles`, `Warns`.

Reads as: "this monad runs sessions / warns / manages files."

Standard library type classes (`Monad`, `Functor`, `Traversable`, etc.) are exempt — their names are inherited and not subject to this rule.

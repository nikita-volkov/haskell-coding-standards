# Deriving

## Always Annotate the Strategy

Every `deriving` clause MUST include an explicit strategy keyword. Bare `deriving (Eq, Show)` without a strategy is prohibited.

```haskell
-- Correct
deriving stock (Eq, Show, Generic)
deriving newtype (Hashable, NFData)
deriving anyclass (SomeClass)
deriving (ToJSON) via SomeWrapper Foo

-- Prohibited
deriving (Eq, Show)
```

The strategy is semantic information: `deriving stock (Eq)` and `deriving newtype (Eq)` generate different code. Bare deriving forces the compiler to infer the strategy, making intent opaque.

## When to Use Each Strategy

**`stock`** — for GHC-native structural derivations: `Eq`, `Ord`, `Show`, `Read`, `Bounded`, `Enum`, `Generic`, `Functor`, `Foldable`, `Traversable`, `Data`.

**`newtype`** — for newtypes lifting the underlying type's instance. The derived instance is identical to the wrapped type's instance.

```haskell
newtype AlbumId = AlbumId Int64
  deriving stock (Show, Eq, Ord)
  deriving newtype (Hashable, NFData, ToJSON, FromJSON)
```

**`anyclass`** — for type classes with `DefaultSignatures` implementations or empty instances. Use sparingly.

**`via`** — for deriving through a representationally equal type. Use when a newtype wrapper provides the instance logic.

```haskell
newtype Name = Name Text
  deriving (IsString) via Text
```

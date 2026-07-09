# Pattern: Language Module

## Intent

A module that exports a closed set of domain types together with the typeclasses and instances that relate them. The types form the vocabulary; the typeclasses form a partial grammar for constructing, transforming, and categorizing expressions in that vocabulary.

## When to Use

Use when:

- The domain has a closed set of closely related concepts.
- The concepts are connected by structural relationships: construction, transformation, or categorization.
- Consumers primarily **build** domain expressions, then **write** generic functions constrained by the module's typeclasses, then **read** the types.

Do not use when the module is really a [Vocabulary module](#vocabulary-module), an [Algebra](#algebra), or a full [DSL](#dsl).

## Naming

Name the module after the domain it models. Use the suffix `Language` only when the module is intentionally a language surface.

Good:

- `Orders`
- `Orders.Language`
- `Documents`

Avoid:

- `Orders.Types` — too weak; the module is not just a bag of types.
- `Orders.Model` — overloaded; the module does more than model.

## Structure

```haskell
module Orders
  ( Order (..)
  , LineItem (..)
  , Discount (..)
  , Customer (..)
  , Priced (..)
  , Discountable (..)
  , Taxable (..)
  ) where

data Order = Order
  { customer :: Customer
  , items :: [LineItem]
  , discounts :: [Discount]
  }

data LineItem = LineItem
  { product :: Text
  , quantity :: Int
  , unitPrice :: Money
  }

data Discount = Discount
  { code :: Text
  , amount :: Money
  }

data Customer = Customer
  { name :: Text
  , address :: Address
  }

-- Categorization
class Priced a where
  price :: a -> Money

-- Transformation
class Discountable a where
  applyDiscount :: Discount -> a -> a

-- Categorization
class Taxable a where
  taxRate :: a -> Percent
```

Instances are partial: not every type needs every class.

```haskell
instance Priced LineItem where
  price item = fromIntegral item.quantity * item.unitPrice

instance Priced Order where
  price order = sum (map price order.items)

instance Discountable Order where
  applyDiscount discount order =
    order { discounts = discount : order.discounts }
```

## Key Properties

**Closed world.** The types and typeclasses evolve together. The module is not designed for external types to implement its typeclasses.

**Partial grammar.** Typeclasses are optional. A new type may be added without instances for every class.

**Build-first usage.** Consumers build expressions from the types, then write functions constrained by the typeclasses, then read the types.

**No effects.** The module defines a language, not a runtime. Effectful operations belong in a capability or service module.

## Misclassifications

See [Anti-patterns: Language Module](../antipatterns/README.md#language-module).

## Related

- [Aggregator Namespace](aggregator-namespace.md) — a different pattern for uniform implementors of a single typeclass
- [Port](port.md) — for interfaces between logic and infrastructure
- [Execution Capability](execution-capability.md) — for lifting abstractions into execution contexts

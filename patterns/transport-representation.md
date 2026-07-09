# Pattern: Transport Representation

## Intent

A closed set of DTO types that model the wire shape of a boundary format (JSON, Binary, ...), each related to its Domain counterpart by a single [Lawful Conversion](lawful-conversion.md) instance. Unlike [Domain Language](domain-language.md) or [Intermediate Representation](intermediate-representation.md), a Transport Representation has no internal grammar: nobody hand-builds a wire value or writes generic functions constrained by typeclasses over the wire vocabulary. It exists solely to be encoded to and decoded from.

## When to Use

Use when:

- A Domain type needs to cross a system boundary (HTTP, a message queue, a file format, a wire protocol).
- The wire shape and the Domain shape are not identical — the wire format has its own constraints (JSON allows nulls and nesting; a binary layout is flat and positional; a CSV row is a fixed tuple of text).

Do not use when the Domain type already has no additional invariants beyond what the wire format represents — in that case a direct `deriving` instance on the Domain type is simpler, at the cost of coupling the Domain module to a specific wire format. Whether that trade-off is worth it is a per-type judgment call, not a rule.

## One DTO Per Format, Kept Separate

Each format gets its own DTO type, and formats are kept in separate modules — often separate libraries or layers entirely, not just separate files in one package.

```
Orders/            -- Domain Language module: Order, LineItem, ...

OrdersJson/         -- Transport Representation: JSON format
  OrderJson.hs      -- OrderJson type + ToJSON/FromJSON + IsSome instance

OrdersBinary/       -- Transport Representation: Binary format
  OrderBinary.hs    -- OrderBinary type + Binary instance + IsSome instance
```

Resist collapsing `OrderJson` and `OrderBinary` into one shared `OrderWire` type with two codec instances attached. Formats diverge in shape often enough (nesting vs. flat layout, optional-via-null vs. optional-via-length-prefix) that a shared type either compromises to fit both or silently only really fits one. The test: if the two format shapes would be structurally identical field-for-field, sharing one type is fine; if either format needs its own layout, split.

## Structure

```haskell
-- OrdersJson/OrderJson.hs
module OrdersJson.OrderJson
  ( OrderJson (..)
  ) where

data OrderJson = OrderJson
  { customer :: CustomerJson
  , items :: [LineItemJson]
  , discountCodes :: [Text]
  }
  deriving (ToJSON, FromJSON)

instance IsSome OrderJson Order where
  to order =
    OrderJson
      { customer = to order.customer
      , items = map to order.items
      , discountCodes = map (.code) order.discounts
      }
  maybeFrom orderJson = do
    customer <- maybeFrom orderJson.customer
    items <- traverse maybeFrom orderJson.items
    pure Order { customer, items, discounts = [] }  -- discounts resolved elsewhere
```

## Conversion Direction

The wire type is usually the **superset** — anything that parses structurally, including values that violate business invariants — and the Domain type is the **subset** — values that satisfy them. That makes `IsSome Wire Domain` the common shape: `to :: Domain -> Wire` (encode, total — a valid Domain value always has a wire representation) and `maybeFrom :: Wire -> Maybe Domain` (decode and validate, partial — not every wire value is valid).

Treat this as the common case, not a mandate. When a Domain type has no extra invariants beyond the wire shape, the relation degenerates to `Is` (total both ways). See [Lawful Conversion](lawful-conversion.md) for the full typeclass hierarchy and when each applies.

## Key Properties

**No internal grammar.** A Transport Representation is a bag of DTOs plus one conversion relation each. It is not a Domain Language or Intermediate Representation in miniature — resist adding construction/transformation/categorization typeclasses over the wire vocabulary itself. If a real need for that emerges (e.g. a JSON-AST-like DTO set with its own traversal typeclasses), that's a sign that particular vocabulary has outgrown this pattern.

**Kept at arm's length from Domain.** Domain modules do not import Transport modules. Transport modules import Domain to define the conversion.

## Related

- [Lawful Conversion](lawful-conversion.md) — the Domain <-> Wire conversion relation
- [Domain Language](domain-language.md) — what Transport Representation converts to/from
- [Aggregator Namespace](aggregator-namespace.md) — if a single format ends up with many DTO sub-modules, consider pairing with this pattern for the format's root module
- [Preludes](preludes.md) — a companion prelude may be warranted once a format's DTOs are numerous enough

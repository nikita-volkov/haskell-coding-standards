# Pattern: Lawful Conversion

## Intent

A small typeclass vocabulary for expressing conversions between two related types, graded by how much the conversion preserves. Replaces ad-hoc `toX`/`fromX` function pairs with a shape whose laws are known in advance, so callers and instance authors don't have to reverse-engineer what a given conversion promises.

Implemented by [`lawful-conversions`](https://hackage.haskell.org/package/lawful-conversions). Use it rather than hand-rolling an equivalent typeclass — the laws and the property tests that check them already exist.

## When to Use

Use when two types are related well enough that a generic conversion relation is worth naming:

- [Transport Representation](transport-representation.md) <-> [Domain Language](domain-language.md): decoding validates, encoding doesn't fail.
- [Intermediate Representation](intermediate-representation.md) stage N <-> stage N+1: lowering is total but usually loses information.
- A newtype and its underlying representation, or two representations of the same concept at different precision/strictness (e.g. `Text` <-> strict `ByteString`).

Don't reach for this when a conversion is truly one-off and local — a single unexported helper function doesn't need a typeclass. The pattern earns its cost when the same shape of conversion recurs across multiple type pairs, or when the relation is part of a documented pattern (Transport, IR) that other code is expected to rely on.

## The Three Relations

| Typeclass | Meaning | Methods | Direction |
|---|---|---|---|
| `IsSome a b` | `b` is a subset of `a` | `to :: b -> a` (total), `maybeFrom :: a -> Maybe b` (partial) | one type embeds into the other; going back may fail |
| `IsMany a b` | `a` canonicalizes to `b` | `from :: a -> b` (total) | total but may be lossy (surjective, not injective) |
| `Is a b` | `a` and `b` are isomorphic | both of the above, total in both directions | no information lost either way |

`Is` is the strictest (lossless, total, reversible) and implies both `IsSome` and `IsMany`. Pick the weakest relation that's actually true of the pair — claiming `Is` when the conversion can fail or lose information breaks the laws and misleads callers who rely on them.

## Choosing a Relation

Ask two questions about the pair `(a, b)`:

1. **Can converting fail?** If yes, you need the partial direction (`IsSome`'s `maybeFrom`). If no, both directions are total.
2. **Is the conversion reversible without loss?** If yes and both directions are total, use `Is`. If one direction is total but lossy and there's no meaningful partial inverse, use `IsMany`. If one direction is total (embedding) and the other is partial (validating), use `IsSome`.

Common instantiations:

- Wire format `a` is a superset of Domain `b`, and decoding validates: `IsSome Wire Domain`.
- Domain type has no invariants beyond the wire shape: `Is Wire Domain`.
- IR stage `a` lowers to stage `b`, discarding surface syntax, and lowering never fails: `IsMany StageA StageB`.

These are common cases, not mandates — a given pair may genuinely need something else. Judge each pair on its own two questions above rather than pattern-matching on the type names.

## Testing

Instances should be checked against the laws with property tests (the library ships `quickcheck-classes`-style law tests) rather than trusted by inspection — a broken `IsSome` instance (e.g. `maybeFrom . to /= Just`) silently corrupts round-trips at exactly the boundary this pattern exists to make safe.

## Related

- [Transport Representation](transport-representation.md) — the primary consumer of `IsSome`/`Is` for Domain <-> Wire
- [Intermediate Representation](intermediate-representation.md) — the primary consumer of `IsMany` for stage lowering

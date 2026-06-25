# Documentation

## Coverage

Every exported definition MUST have a Haddock comment. Undocumented exports are a style violation.

This includes: functions, types, type class methods, type aliases, and pattern synonyms. Exception: re-exports from another module where the original documentation is authoritative.

## Module Header

Every module MUST have a module-level Haddock comment stating its purpose in one paragraph.

```haskell
-- |
-- Encoding and decoding of album data for PostgreSQL storage.
module MyApp.Album.Codecs where
```

## Inline Documentation

Document the *why* and the *contract*, not the *what*. The name and type signature already say what the function does.

```haskell
-- Correct — documents the contract
-- | Validates an album record. Fails if the name is empty or the
-- release date is in the future.
validateAlbum :: RawAlbum -> Either AlbumValidationError Album

-- Prohibited — restates the name
-- | Validates an album.
validateAlbum :: RawAlbum -> Either AlbumValidationError Album
```

## Type Class Laws

Type class laws MUST be documented using `prop>` Haddock syntax in the type class comment.

```haskell
-- |
-- Conversion between the Haskell type and its PostgreSQL scalar representation.
--
-- Laws:
--
-- prop> decode (encode x) == Right x
class IsScalar a where
  encode :: a -> Value
  decode :: Value -> Either Text a
```

## Format

Use plain prose. Avoid elaborate Haddock markup (`@`, `<`, `>`) unless it meaningfully aids comprehension. Section headers (`== Section`) MAY be used in large module headers.

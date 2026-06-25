# Errors

## `Either` for Expected Failure

MUST use `Either e a` for any failure a caller should handle: validation, parsing, business rule violations, missing resources.

The error type `e` MUST be a custom ADT specific to the operation or domain. It MUST enumerate the possible failure modes as distinct constructors.

```haskell
data AlbumValidationError
  = MissingNameAlbumValidationError
  | InvalidReleasedDateAlbumValidationError Text
  | UnknownFormatAlbumValidationError Text

validateAlbum :: RawAlbum -> Either AlbumValidationError Album
```

## Exceptions for Unexpected Failure

MUST use exceptions (`throwIO`) only for conditions that indicate programmer error or unrecoverable state: broken invariants, resource exhaustion, missing configuration that should have been validated at startup.

MUST NOT catch exceptions in normal logic paths. Exception handling belongs at process boundaries.

## Prohibited Patterns

**`Maybe` for errors** — `Maybe` means "optionally present", not "succeeded or failed". If a failure has a cause, use `Either`. `Maybe` is appropriate only for genuine optionality (a value that may or may not exist).

**String error types** — `Either String a` and `Either Text a` as final error types are prohibited. They are unstructured, untestable, and prevent callers from handling specific cases. Use a custom ADT.

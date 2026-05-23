# LeanSha256

A pure-Lean SHA-256 reference implementation. NIST CAVP-validated,
kernel-reducible (no FFI / no opaque calls), independent of any SSZ
machinery.

## Status

Stable. NIST CAVP byte-oriented short-message and long-message
test suites pass via build-time `native_decide` gates inside
`LeanSha256/Nist.lean`.

## Dependencies

None.

## Modules

* `LeanSha256.Core` — the SHA-256 algorithm (initial state, message
  schedule, compression function, padding, multi-block driver).
  Kernel-reducible.
* `LeanSha256.Nist` — NIST CAVP byte-oriented test vectors as
  `native_decide` gates over `LeanSha256.Core`.

## Build / test

```bash
# From the monorepo root:
lake build LeanSha256
# (NIST CAVP gates fire at build time via native_decide inside
# LeanSha256.Nist; if any case regresses, the build fails.)

# Or from this subpackage's directory:
cd packages/LeanSha256 && lake build
```

## Requiring this package

TODO: publication URL.

```toml
[[require]]
name = "LeanSha256"
git = "TODO"
```

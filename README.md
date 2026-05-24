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
# (3 anchor FIPS 180-4 §B gates fire at build time via
# native_decide inside LeanSha256.Nist; if any case regresses,
# the build fails.)

# Full NIST CAVP byte-oriented suite — 129 short-message +
# long-message cases via native_decide, ~108 s. The `Justfile`
# at the umbrella root wraps this:
just test-sha256
# …which is exactly `lake build LeanSha256Tests`.

# Or from this subpackage's directory:
cd packages/LeanSha256 && lake build
```

The CAVP vector files (`SHA256ShortMsg.rsp`, `SHA256LongMsg.rsp`,
`SHA256Monte.rsp`) live under `cavp/` in this subpackage and are
the upstream NIST artefacts unmodified — a stranger can verify
the byte-for-byte match against the
[NIST CSRC CAVP archive](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/secure-hashing).

## Requiring this package

`LeanSha256` lives as a subpackage of the [`etheorem` umbrella
repository](https://github.com/etheorem/etheorem). To depend on
it from another Lake project, add a `[[require]]` block to your
`lakefile.toml`:

```toml
[[require]]
name = "LeanSha256"
git = "https://github.com/etheorem/etheorem"
subDir = "packages/LeanSha256"
rev = "main"  # pin to a specific commit hash for reproducible builds
```

Then run `lake update` to refresh `lake-manifest.json`. Per the
umbrella's [`CLAUDE.md`](../../CLAUDE.md) dependency policy,
prefer pinning `rev` to a specific commit hash over tracking a
branch once you've validated a working pair.

`LeanSha256` has no dependencies outside Lean core.

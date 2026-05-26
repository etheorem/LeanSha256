# LeanSha256

A pure-Lean SHA-256 reference implementation. Two functions in the
public surface (`hash` and `combine`); NIST CAVP-validated against
the FIPS 180-4 published vectors; kernel-reducible (no FFI, no
opaque calls, no SSZ coupling) so kernel `decide` closes goals about
it without trusting the compiler.

## Status

Stable. NIST CAVP byte-oriented short-message and long-message
test suites pass via build-time `native_decide` gates inside
`LeanSha256Tests/Nist.lean`. Version `0.1.0` — the two-function
public API is intended to remain stable; internal modules under
`LeanSha256.Core` (round functions, message schedule, helper
lemmas) may evolve.

## Dependencies

None. Pure Lean 4 + the standard library — no `mathlib`, no
`batteries`, no native shims.

## Modules

* `LeanSha256.Core` — the SHA-256 algorithm (initial state, message
  schedule, compression function, padding, multi-block driver).
  Kernel-reducible.
* `LeanSha256Tests.Nist` — NIST CAVP byte-oriented test vectors as
  `native_decide` gates over `LeanSha256.Core`. Lives in the
  separate `LeanSha256Tests` library so the ~108 s build cost is
  opt-in.

## Public API

```lean
namespace LeanSha256

/-- Single SHA-256 digest. Equivalent to FIPS 180-4 §6.2. -/
def hash    : ByteArray → ByteArray

/-- Sibling combine: `hash (a ++ b)`. Exported separately because
    Merkle-tree consumers (e.g. SSZ `hash_tree_root`) call it on the
    hot path and want a direct, allocation-free entry point. -/
def combine : ByteArray → ByteArray → ByteArray

end LeanSha256
```

Both functions are total, structurally terminating, and return a
32-byte `ByteArray`. The `Core` module also exposes a handful of
structural conformance lemmas (e.g. that `hash` agrees with a
single-block compression on inputs ≤ 55 bytes) — see its module
docstring for the full list.

## Trust assumptions

* Kernel-reducible: every definition reduces in Lean's trusted
  kernel. Closing a goal with `decide` against `LeanSha256.hash`
  adds **no** compiler-trust axiom — only Lean's core kernel is
  trusted.
* For speed, consumers normally close hash-equality goals with
  `native_decide`, which trusts the Lean compiler via
  `Lean.ofReduceBool`. The full `LeanSha256Tests` library
  (`LeanSha256Tests.Nist`) does this — see its `#axioms` output.
* No `sorry`, no `axiom` declarations, no FFI calls anywhere in
  the library.

## Build / test

```bash
# From the umbrella repo root:
lake build LeanSha256
# (3 anchor FIPS 180-4 §B gates fire at build time via
# native_decide inside LeanSha256.Core; if any case regresses,
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

To regenerate `LeanSha256Tests/Nist.lean` after refreshing the
`.rsp` files, run the in-package script (or its umbrella wrapper):

```bash
python3 packages/LeanSha256/scripts/gen_sha256_cavp.py
# or, from the umbrella root:
just gen-cavp
```

## Requiring this package

`LeanSha256` is published from the [`etheorem` umbrella
repository](https://github.com/etheorem/etheorem). It can be
required either from inside the umbrella or as a standalone
package — both forms are supported because the package is
self-contained (no FFI, no shared sources outside its own
directory).

**From another Lake project, via git** (typical):

```toml
[[require]]
name = "LeanSha256"
git = "https://github.com/etheorem/etheorem"
subDir = "packages/LeanSha256"
rev = "<commit-sha>"  # pin to a commit for reproducible builds
```

**From a local copy, via path** (vendoring or development):

```toml
[[require]]
name = "LeanSha256"
path = "vendor/LeanSha256"
```

Then run `lake update` to refresh `lake-manifest.json`. Per the
umbrella's [`CLAUDE.md`](../../CLAUDE.md) dependency policy,
prefer pinning `rev` to a specific commit hash over tracking a
branch once you've validated a working pair.

## Licence

LGPL-3.0-only. The licence text is the umbrella repo's
[`LICENSE`](../../LICENSE), referenced by this package's
`lakefile.toml` via `licenseFiles = ["../../LICENSE"]`.

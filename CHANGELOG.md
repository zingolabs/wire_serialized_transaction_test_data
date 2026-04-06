# Changelog

## 1.0.0 — 2026-04-06

Initial public release.

- 26 wire-format serialized Zcash transaction test vectors covering Sprout v1,
  Sprout v2, Overwinter, Sapling, and NU5.
- `TestVector` struct, `TEST_VECTORS` static, and `get_test_vectors()` accessor.
- Zero external dependencies (`LazyLock` from std).

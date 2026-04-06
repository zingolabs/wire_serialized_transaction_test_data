# wire_serialized_transaction_test_data

Wire-format serialized Zcash transaction test data covering v1-v4 transaction formats.

Publishing this data as a standalone, zero-dependency crate on crates.io gives the Zcash
ecosystem a stable calibration point. Versions are never yanked and data is never removed,
so any crate — whether inside the zaino workspace or maintained by a third party — can pin
a known version and trust that its transaction-parsing tests are running against the same
reference bytes. Embedding the same data inside each downstream project leads to drift; a
shared crate on the registry eliminates that.

Zero-dependency Rust crate providing 26 test vectors across five Zcash transaction eras:
Sprout v1, Sprout v2, Overwinter, Sapling, and NU5. The vectors are generated from
[zingolabs/zcash-test-vectors](https://github.com/zingolabs/zcash-test-vectors).

## Public interface

All items are exported from the `transactions` module.

### `TestVector`

```rust
pub struct TestVector {
    pub description: &'static str,
    pub version: u32,
    pub lock_time: u32,
    pub expiry_height: u32,
    pub txid: [u8; 32],
    pub is_coinbase: u8,
    pub has_sapling: u8,
    pub has_orchard: u8,
    pub transparent_inputs: usize,
    pub transparent_outputs: usize,
    pub tx: Vec<u8>,
}
```

### `TEST_VECTORS`

```rust
pub static TEST_VECTORS: LazyLock<Vec<TestVector>>
```

Lazily-initialized vector containing all 26 test vectors.

### `get_test_vectors()`

```rust
pub fn get_test_vectors() -> &'static Vec<TestVector>
```

Returns a reference to `TEST_VECTORS`.

## Usage in zaino

### zaino-fetch (direct dev-dependency)

`zaino-fetch` depends on this crate directly as a dev-dependency:

```toml
[dev-dependencies]
wire_serialized_transaction_test_data = { workspace = true }
```

Its unit tests in `zaino-fetch/src/chain/transaction.rs` filter vectors by version and feed
the raw bytes into `FullTransaction::parse_from_slice`, then assert that:

- the transaction parses successfully
- the parsed txid matches `vector.txid`
- the version field is correct
- transparent input/output counts match
- no trailing bytes remain after parsing

### zaino-testutils (via zaino-testvectors)

`zaino-testutils` depends on the in-tree `zaino-testvectors` crate, which contains the same
test vector data as this crate (using `lazy_static` rather than `LazyLock`).
`zaino-testutils` re-exports it as a convenience module:

```rust
pub mod test_vectors {
    pub use zaino_testvectors::*;
}
```

The integration test suite (`integration-tests/tests/test_vectors.rs`) consumes the vectors
through this re-export:

```rust
use zaino_testutils::test_vectors::transactions::get_test_vectors;
```

The `pre_v4_txs_parsing` test iterates over all vectors with `version < 4`, parses each with
`FullTransaction::parse_from_slice`, and validates version, txid, shielded spend/orchard
action presence, and transparent input/output presence.

This crate is the standalone, externally-published replacement for the in-tree
`zaino-testvectors` crate.

## License

Apache-2.0

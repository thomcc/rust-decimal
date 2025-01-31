# Decimal &emsp; [![Build Status]][actions] [![Latest Version]][crates.io] [![Docs Badge]][docs] 

[Build Status]: https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fpaupino%2Frust-decimal%2Fbadge&label=build&logo=none
[actions]: https://actions-badge.atrox.dev/paupino/rust-decimal/goto
[Latest Version]: https://img.shields.io/crates/v/rust-decimal.svg
[crates.io]: https://crates.io/crates/rust-decimal
[Docs Badge]: https://docs.rs/rust_decimal/badge.svg
[docs]: https://docs.rs/rust_decimal

A Decimal implementation written in pure Rust suitable for financial calculations that require significant integral and fractional digits with no round-off errors.

The binary representation consists of a 96 bit integer number, a scaling factor used to specify the decimal fraction and a 1 bit sign. Because of this representation, trailing zeros are preserved and may be exposed when in string form. These can be truncated using the `normalize` or `round_dp` functions.

[Documentation](https://docs.rs/rust_decimal/)

## Usage

Decimal numbers can be created in a few distinct ways. The easiest and most optimal method of creating a Decimal is to use the procedural macro within the `rust_decimal_macros` crate:

```rust
// Procedural macros need importing directly
use rust_decimal_macros::dec;

let number = dec!(-1.23);
assert_eq!("-1.23", number.to_string());
```

Alternatively you can also use one of the Decimal number convenience functions:

```rust
// Using the prelude can help importing trait based functions (e.g. core::str::FromStr).
use rust_decimal::prelude::*;

// Using an integer followed by the decimal points
let scaled = Decimal::new(202, 2);
assert_eq!("2.02", scaled.to_string());

// From a string representation
let from_string = Decimal::from_str("2.02").unwrap();
assert_eq!("2.02", from_string.to_string());

// From a string representation in a different base
let from_string_base16 = Decimal::from_str_radix("ffff", 16).unwrap();
assert_eq!("65535", from_string_base16.to_string());

// Using the `Into` trait
let my_int: Decimal = 3i32.into();
assert_eq!("3", my_int.to_string());

// Using the raw decimal representation
let pi = Decimal::from_parts(1102470952, 185874565, 1703060790, false, 28);
assert_eq!("3.1415926535897932384626433832", pi.to_string());
```

## Features

* [db-postgres](#db-postgres)
* [db-tokio-postgres](#db-tokio-postgres)
* [db-diesel-postgres](#db-diesel-postgres)
* [legacy-ops](#legacy-ops)
* [maths](#maths)
* [rust-fuzz](#rust-fuzz)
* [serde-float](#serde-float)
* [serde-str](#serde-str)
* [std](#std)

## `db-postgres`

This feature enables a PostgreSQL communication module. It allows for reading and writing the `Decimal`
type by transparently serializing/deserializing into the `NUMERIC` data type within PostgreSQL.

## `db-tokio-postgres`

Enables the tokio postgres module allowing for async communication with PostgreSQL.

## `db-diesel-postgres`

Enable `diesel` PostgreSQL support. 

## `legacy-ops`

As of `1.10` the algorithms used to perform basic operations have changed which has benefits of significant speed improvements. 
To maintain backwards compatibility this can be opted out of by enabling the `legacy-ops` feature.

## `maths`

This feature enables mathematical functionality such as `pow`, `ln`, `enf` etc.

## `rust-fuzz`

Enable `rust-fuzz` support by implementing the `Arbitrary` trait.

## `serde-float`

Enable this so that JSON serialization of Decimal types are sent as a float instead of a string (default).

e.g. with this turned on, JSON serialization would output:
```json
{
  "value": 1.234
}
```

## `serde-str`

This is typically useful for `bincode` or `csv` like implementations.

Since `bincode` does not specify type information, we need to ensure that a type hint is provided in order to 
correctly be able to deserialize. Enabling this feature on it's own will force deserialization to use `deserialize_str` 
instead of `deserialize_any`. 

If, for some reason, you also have `serde-float` enabled then this will use `deserialize_f64` as a type hint. Because
converting to `f64` _loses_ precision, it's highly recommended that you do NOT enable this feature when working with 
`bincode`. That being said, this will only use 8 bytes so is slightly more efficient in regards to storage size.

## `serde-arbitrary-precision`

This is used primarily with `serde_json` and consequently adds it as a "weak dependency". This supports the 
`arbitrary_precision` feature inside `serde_json` when parsing decimals. 

This is recommended when parsing "float" looking data as it will prevent data loss.

## `std`

Enable `std` library support. This is enabled by default, however in the future will be opt in. For now, to support `no_std`
libraries, this crate can be compiled with `--no-default-features`.

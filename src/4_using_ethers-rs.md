# Using ethers-rs

`ethers-rs` provides extensive tooling to generate type safe bindings for smart contracts. Under the hood, it relies on
the [`ethabi` crate's](https://github.com/rust-ethereum/ethabi) data model for contracts and all core encoding/decoding
related traits. `ethers-rs` provides additional types, traits and macros that help with

* [ABI encoding/decoding](./4_1_abi_encoding_decoding.md) - encode/decode custom types and events
* [Contract Bindings with `abigen!`](./4_2_abigen_contract_bindings.md) - generate type safe contract bindings 

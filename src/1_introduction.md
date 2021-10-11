# Introduction

Welcome to the Rust Ethereum book, where we'll go through examples of how to interact with Ethereum
using [ethers-rs](https://github.com/gakonst/ethers-rs/).

We assume familiarity with Ethereum's mechanics such as:

* Deploying a contract & interacting with it
* Sending funds across accounts
* Signing messages and recovering their sender

## Installing Ethers-rs

First create a new Rust project:

```console
$ cargo new ethers-hello-world && cd ethers-hello-world
```

Edit your `Cargo.toml` and insert the following dependency:

```toml
[dependencies]

ethers = { git = "https://github.com/gakonst/ethers-rs" }
```

## Recommended additional tooling

Tests require the following installed:

1. [`solc`](https://solidity.readthedocs.io/en/latest/installing-solidity.html). We also recommend
   using [solc-select](https://github.com/crytic/solc-select) for more flexibility.
2. [`ganache-cli`](https://github.com/trufflesuite/ganache-cli#installation)

In addition, it is recommended that you set the `ETHERSCAN_API_KEY` environment variable
for [the abigen via Etherscan](https://github.com/gakonst/ethers-rs/blob/master/ethers/tests/major_contracts.rs) tests.
You can get one [here](https://etherscan.io/apis).
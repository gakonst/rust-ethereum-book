# ABI Encoding & Decoding

The [solidity docs](https://docs.soliditylang.org/en/v0.8.9/abi-spec.html#abi) cover the contract ABI specification at
great length. `ethabi` provides the [`Token`](https://docs.rs/ethabi/15.0.0/ethabi/token/enum.Token.html) enum which
represents the fixed set of ABI types,

```rust
pub enum Token {
    Address(Address),
    FixedBytes(FixedBytes),
    Bytes(Bytes),
    Int(Int),
    Uint(Uint),
    Bool(bool),
    String(String),
    FixedArray(Vec<Token>),
    Array(Vec<Token>),
    Tuple(Vec<Token>),
}
```

An encoded payload is always represented as `Vec<Token>`, a series of encoded values:

```rust
pub fn encode(tokens: &[Token]) -> Bytes { ... }
```

To make this generally applicable, `ethers-rs` provides additional helper traits for tokenizing (encoding) and
detokenizable (decoding), most
importantly [`Tokenizable`](https://docs.rs/ethers-core/0.5.4/ethers_core/abi/trait.Tokenizable.html). All primitive
types and commonly used data structures, such as tuples are `Tokenizable`.

### Implement ABI encoding/decoding

Contract function input can be very verbose and can consist of quite a few parameters, so it may be convenient to bundle
them into user-defined data that also supports ABI compliant encoding/decoding.

A custom ABI type might look like this:

```rust
#[derive(Debug, Clone)]
struct ValueChanged {
    old_author: Address,
    new_author: Address,
    old_value: String,
    new_value: String,
}
```

### Implement `Tokenizable` manually

In order to make custom types __tokenizable__ you'd need to implement the `Tokenizable` trait:

```rust

impl ethers_core::abi::Tokenizable for ValueChanged
{
    /// Tries to convert a `Token` into an instance of the type 
    fn from_token(
        token: ethers_core::abi::Token,
    ) -> Result<Self, ethers_core::abi::InvalidOutputType>
        where
            Self: Sized,
    {
        if let ethers_core::abi::Token::Tuple(tokens) = token {
            // -- snip --
        }
    }

    /// Encodes the type into a `Token`
    fn into_token(self) -> ethers_core::abi::Token {
        ethers_core::abi::Token::Tuple(vec![
            self.old_author.into_token(),
            self.new_author.into_token(),
            self.old_value.into_token(),
            self.new_value.into_token(),
        ])
    }
}
```

### Implement `Tokenizable` using derive macro `EthAbiType`

Implementing `Tokenizable` can be very verbose itself and error-prone, therefore `ethers-rs` provides
the [`EthAbiType`](https://docs.rs/ethers-contract/0.5.3/ethers_contract/derive.EthAbiType.html) derive proc macro that
automatically generates all the necessary `from` and `into` token code.

By adding a `EthAbiType` the code above will be automatically generated

```rust
use ethers::abi::EthAbiType;
use ethers::types::*;

#[derive(Debug, Clone, EthAbiType)]
struct ValueChanged {
    old_author: Address,
    new_author: Address,
    old_value: String,
    new_value: String,
}
```

This will work for every type that is `Tokenizable`, so it's possible to nest them:

```rust
#[derive(Debug, Clone, EthAbiType)]
struct ValueChangedOuter {
    inner: ValueChanged,
    msg: String,
}
```

### Implement Event bindings

Ethereum Events (aka logs) work similarly, but their encoding is dependent on their indexed parameters,
see [solidity docs](https://docs.soliditylang.org/en/v0.8.9/abi-spec.html#events) for more details.

In `ethers-rs` _Event_ types are abstracted via
the [`EthEvent`](https://docs.rs/ethers-contract/0.5.3/ethers_contract/trait.EthEvent.html) trait, which provides
necessary functions to identify and decode events.

To easily create event bindings, `ethers-rs` provides the `EthEvent` as derive proc macro as well.

In order to turn a custom type into an `EthEvent` you only need to add it as derive:

```rust
use ethers::abi::EthEvent;

#[derive(Debug, Clone, EthEvent)]
struct ValueChangedEvent {
    old_author: Address,
    new_author: Address,
    old_value: String,
    new_value: String,
}

assert_eq!(
    "ValueChangedEvent(address,address,string,string)",
    ValueChangedEvent::abi_signature()
);
```

An _event_ can be further customized via the `ethevent` attribute.

__Rename the event__

```rust
#[derive(Debug, Clone, EthEvent)]
#[ethevent(
name = "MyEvent",
)]
struct ValueChangedEvent {
    old_author: Address,
    new_author: Address,
    old_value: String,
    new_value: String,
}

assert_eq!(
    "MyEvent(address,address,string,string)",
    ValueChangedEvent::abi_signature()
);
```

__Mark fields as _indexed___

```rust
#[derive(Debug, Clone, EthEvent)]
struct ValueChangedEvent {
    old_author: Address,
    #[ethevent(indexed, name = "newAuthor")]
    new_author: Address,
    old_value: String,
    new_value: String,
}

assert_eq!(
    "ValueChangedEvent(address,address,string,string)",
    ValueChangedEvent::abi_signature()
);
```

__Mark event as anonymous__

```rust
#[derive(Debug, Clone, EthEvent)]
#[ethevent(anonymous)]
struct ValueChangedEvent {
    old_author: Address,
    new_author: Address,
    old_value: String,
    new_value: String,
}

assert_eq!(
    "ValueChangedEvent(address,address,string,string) anonymous",
    ValueChangedEvent::abi_signature()
);
```

__Rename and override the ABI as well__

```rust

#[derive(Debug, Clone, EthAbiType)]
struct SomeType {
    inner: Address,
    msg: String,
}

#[ethevent(
    name = "MyEvent",
    abi = "MyEvent(address,(address,string),string)"
)]
struct ValueChangedEvent {
    old_author: Address,
    val: SomeType,
    new_value: String,
}

assert_eq!(
    "MyEvent(address,(address,string),string)",
    ValueChangedEvent::abi_signature()
);

```


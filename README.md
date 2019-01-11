# branca

|Crate|Documentation|License|Travis
|:---:|:-----------:|:-----------:|:-----------:|
|[![Crates.io][crates-badge]][crates-url]|[![Docs][doc-badge]][doc-url]|[![License][license-badge]][license-url]|[![Travis-CI][travis-badge]][travis-url]

[crates-badge]: https://img.shields.io/crates/v/branca.svg
[crates-url]: https://crates.io/crates/branca
[doc-badge]: https://docs.rs/branca/badge.svg
[doc-url]: https://docs.rs/branca
[license-badge]: https://img.shields.io/badge/License-MIT-brightgreen.svg
[license-url]: https://github.com/return/branca/blob/master/LICENSE
[travis-badge]: https://api.travis-ci.org/return/branca.svg?branch=master
[travis-url]: https://travis-ci.org/return/branca

Branca is a secure alternative token format to JWT. This implementation is written in pure Rust and uses the XChaCha20-Poly1305 AEAD (Authenticated Encryption with Associated Data) stream cipher for generating authenticated and encrypted tamper-proof tokens. More information about the branca token specification can be found here in [branca-spec.](
https://github.com/tuupola/branca-spec/blob/master/README.md)

# Requirements

* Rust 1.18+
* Cargo

# Installation

Add this line to your Cargo.toml under the dependencies section:

```toml
[dependencies]
branca = "^0.5.0"
```

Then you can import the crate into your project with these lines:
```rust
extern crate branca;
use branca::{Branca, encode, decode};
```

# Example Usage

The simplest way to use this crate is to use `Branca::new()` in this example below:

```rust
    let key = b"supersecretkeyyoushouldnotcommit".to_vec();
    let token = Branca::new(&key).unwrap();
    let ciphertext = token.encode("Hello World!").unwrap();

    let payload = token.decode(ciphertext.as_str(), 0).unwrap();
    println("{}", payload); // "Hello World!"
```

See more examples of setting fields in the [Branca struct](https://docs.rs/branca/) and in the [Documentation section.](https://docs.rs/branca/struct.Branca.html)

## Direct usage without Branca builder.
### Encoding:
```rust
let key = b"supersecretkeyyoushouldnotcommit".to_vec();
let nonce = *b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c";

let message = "Hello world!";
let timestamp = 123206400;
let branca_token = encode(message,&key,&nonce,timestamp).unwrap();

// branca_token = 875GH233T7IYrxtgXxlQBYiFobZMQdHAT51vChKsAIYCFxZtL1evV54vYqLyZtQ0ekPHt8kJHQp0a
```

### Decoding:
```rust
let ciphertext = branca_token.as_str();
let key = b"supersecretkeyyoushouldnotcommit".to_vec();
let ttl = 0; // The ttl can be used to determine if the supplied token has expired or not.
let decoded = decode(ciphertext, &key, ttl);

if decoded.is_err() {
    // Error
} else {
    let msg = decoded.unwrap(); 
    // msg = "Hello world!"
}
```
## Encode/Decode arbitrary data structures with Serde.
Since Branca is able to work with any format of data in the payload, it is possible for the payload to be anything from a JSON object, plaintext, raw bytes, protocol buffers or even a JWT.

Here is a example of using Branca to encode/decode a typical JSON object with serde_json.

Add the following into your Cargo.toml file:
```toml
[dependencies]
branca = "^0.5.0"
serde_json = "^1.0"
serde_derive = "1.0.83"
ring = "0.13.5"
```

```rust
#[macro_use]
extern crate serde_json;
#[macro_use]
extern crate serde_derive;
extern crate branca;

use branca::{encode, decode};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    user: String,
    scope: Vec<String>,
}

fn main(){

    let message = json!({
        "user" : "someone@example.com",
        "scope":["read", "write", "delete"],
    }).to_string();

    let key = b"supersecretkeyyoushouldnotcommit".to_vec();
    let token = Branca::new(&key).unwrap();
    
    // Encode Message
    let branca_token = token.encode(message.as_str()).unwrap();
    
    // Decode Message
    let payload = token.decode(branca_token.as_str(), 0).unwrap();

    let json: User = serde_json::from_str(payload.as_str()).unwrap();

    println!("{}", branca_token);
    println!("{}", payload);
    println!("{:?}", json);
}
```

Branca uses the [Ring crate](https://github.com/briansmith/ring) as to generate secure random nonces. You can still use Ring's SecureRandom or sodiumoxide's aead gen_nonce() or gen_key() for generating secure nonces and keys for example. 

But do note that the nonce **must be 24 bytes in length.** Keys **must be 32 bytes in length.**

# Building
`cargo build`

# Testing
`cargo test --examples`

# Contributing
Contributions and patches are welcome! Fork this repository, add your changes and send a PR.

Before you send a PR, make sure you run `cargo test --examples` first to check if your changes pass the tests.

If you would like to fix a bug or add a enhancement, please do so in the issues section and provide a short description about your changes.

# License
MIT
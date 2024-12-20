<h1 align="center">Rust async-osc</h1>
<div align="center">
  <strong>
    Async Rust library for the Open Sound Control (OSC) protocol
  </strong>
</div>

<br />

<div align="center">
  <!-- Crates version -->
  <a href="https://crates.io/crates/async-osc">
    <img src="https://img.shields.io/crates/v/async-osc.svg?style=flat-square"
    alt="Crates.io version" />
  </a>
  <!-- Downloads -->
  <a href="https://crates.io/crates/async-osc">
    <img src="https://img.shields.io/crates/d/async-osc.svg?style=flat-square"
      alt="Download" />
  </a>
  <!-- docs.rs docs -->
  <a href="https://docs.rs/async-osc">
    <img src="https://img.shields.io/badge/docs-latest-blue.svg?style=flat-square"
      alt="docs.rs docs" />
  </a>
</div>

<div align="center">
  <h3>
    <a href="https://docs.rs/async-osc">
      API Docs
    </a>
    <span> | </span>
    <a href="https://github.com/Frando/async-osc/releases">
      Releases
    </a>
    <span> | </span>
    <a href="https://github.com/Frando/async-osc/blob/master.github/CONTRIBUTING.md">
      Contributing
    </a>
  </h3>
</div>

This crate implements an async interface on the [OSC 1.0](https://web.archive.org/web/20201211193930/http://opensoundcontrol.org/spec-1_0) protocol. It uses [`async-std`](https://docs.rs/async-std) for async networking and [`rosc`](https://docs.rs/rosc) for OSC encoding and decoding.

## Installation
```sh
$ cargo add async-osc
```

## Example
```rust
use async_osc::{prelude::*, Error, OscPacket, OscSocket, OscType, Result};
use async_std::stream::StreamExt;

#[async_std::main]
async fn main() -> Result<()> {
    let mut socket = OscSocket::bind("localhost:5050").await?;

    // Open a second socket to send a test message.
    async_std::task::spawn(async move {
        let socket = OscSocket::bind("localhost:0").await?;
        socket.connect("localhost:5050").await?;
        socket
            .send(("/volume", (0.9f32, "foo".to_string())))
            .await?;
        Ok::<(), Error>(())
    });

    // Listen for incoming packets on the first socket.
    while let Some(packet) = socket.next().await {
        let (packet, peer_addr) = packet?;
        eprintln!("Receive from {}: {:?}", peer_addr, packet);
        match packet {
            OscPacket::Bundle(_) => {}
            OscPacket::Message(message) => match &message.as_tuple() {
                ("/volume", &[OscType::Float(vol), OscType::String(ref s)]) => {
                    eprintln!("Set volume: {} {}", vol, s);
                }
                _ => {}
            },
        }
    }
    Ok(())
}
```

## Safety
This crate uses ``#![deny(unsafe_code)]`` to ensure everything is implemented in
100% Safe Rust.

## Contributing
Want to join us? Check out our ["Contributing" guide][contributing] and take a
look at some of these issues:

- [Issues labeled "good first issue"][good-first-issue]
- [Issues labeled "help wanted"][help-wanted]

[contributing]: https://github.com/Frando/async-osc/blob/master.github/CONTRIBUTING.md
[good-first-issue]: https://github.com/Frando/async-osc/labels/good%20first%20issue
[help-wanted]: https://github.com/Frando/async-osc/labels/help%20wanted

## License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br/>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>

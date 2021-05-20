# rust-utp

[![Crate Version](https://img.shields.io/crates/v/utp.svg?style=flat)](https://crates.io/crates/tokio-utp)
[![Build Status](https://img.shields.io/travis/meqif/rust-utp.svg?style=flat)](http://travis-ci.org/meqif/rust-utp)
[![Windows Build Status](https://ci.appveyor.com/api/projects/status/q38b38fendqat8o6?svg=true)](https://ci.appveyor.com/project/meqif/rust-utp)
[![codecov](https://codecov.io/gh/meqif/rust-utp/branch/master/graph/badge.svg)](https://codecov.io/gh/meqif/rust-utp)

An async [Micro Transport Protocol](http://www.bittorrent.org/beps/bep_0029.html) library implemented in Rust.

[API documentation](http://meqif.github.io/rust-utp/)

## Overview

The Micro Transport Protocol is a reliable transport protocol built over
UDP. Its congestion control algorithm is
[LEDBAT](http://tools.ietf.org/html/rfc6817), which tries to use as much unused
bandwidth as it can but readily yields to competing flows, making it useful for
bulk transfers without introducing congestion in the network.

The current implementation is somewhat incomplete, lacking a complete implementation of congestion
control. However, it does support packet loss detection (except by timeout) the
Selective Acknowledgment extension, handles unordered and duplicate packets and
presents a stream interface (`UtpStream`).

## Stability

This crate is experimental and contains many bugs. We strongly advice against using it in production environments.
Contributions and fixes are welcome :)

## Usage

To use `tokio-utp`, add this to your `Cargo.toml`:

```toml
[dependencies]
tokio-utp = "*"
```

## Examples

The simplest example program would be:

```rust
use tokio-utp::UtpStream;
use tokio::task;
use std::io::Write;

fn main() {
    // Connect to an hypothetical local server running on port 8080
    let addr = "127.0.0.1:8080";
    let (mut stream, driver) = UtpStream::connect(addr).await.expect("Error connecting to remote peer");
    
    task::spawn(driver);

    // Send a string
    stream.write("Hi there!".as_bytes()).await.expect("Write failed");

    // Close the stream
    stream.close().await.expect("Error closing connection");
}
```

Check out the files under the "examples" directory for more example programs, or run them with `cargo run --example <example_name>`.

## Roadmap

- [x] congestion control
- [x] proper connection closing
    - [x] handle both RST and FIN
    - [x] send FIN on close
    - [x] automatically send FIN on `drop` if not already closed
- [x] sending RST on mismatch
- [x] setters and getters that hide header field endianness conversion
- [x] SACK extension
- [x] handle packet loss
    - [x] send triple-ACK to re-request lost packet (fast resend request)
    - [x] rewind send window and resend in reply to triple-ACK (fast resend)
    - [x] resend packet on ACK timeout
- [x] stream interface
- [x] handle unordered packets
- [x] duplicate packet handling
- [x] listener abstraction
- [x] incoming connections iterator
- [x] time out connection after too many retransmissions
- [ ] path MTU discovery

## License

This library is distributed under similar terms to Rust: dual licensed under the MIT license and the Apache license (version 2.0).

See LICENSE-APACHE, LICENSE-MIT, and COPYRIGHT for details.

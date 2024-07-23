# Tokio-IPC

This is a fork of [parity-tokio-ipc](https://github.com/paritytech/parity-tokio-ipc) -> [tipsy](https://github.com/aschey/tipsy).

[Tokio-IPC](https://github.com/akahan/tokio-ipc) is a library for cross-platform async IPC using Tokio.
It utilizes unix sockets on UNIX (via [`tokio::net::UnixStream`](https://docs.rs/tokio/latest/tokio/net/struct.UnixStream.html))
and named pipes on windows (via [`tokio::net::windows::named_pipe`](https://docs.rs/tokio/latest/tokio/net/windows/named_pipe/index.html)).

## Server

```rust,no_run
use tokio_ipc::{Endpoint, ServerId};
use futures::stream::StreamExt;
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    #[cfg(not(windows))]
    let options = Some(tokio_ipc::EndpointOptions {
        on_conflict: tokio_ipc::OnConflict::Overwrite,
    });
    #[cfg(windows)]
    let options = None;

    Endpoint::new(ServerId::new("my-server"), options)?
        .incoming()?
        .for_each(|conn| async {
            match conn {
                Ok(stream) => println!("Got connection!"),
                Err(e) => eprintln!("Error when receiving connection: {:?}", e),
            }
        });
    Ok(())
}
```

## Client

```rust,no_run
use tokio_ipc::{Endpoint, ServerId};
use tokio::io::AsyncWriteExt;
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let mut client = Endpoint::connect(ServerId::new("my-server")).await?;
    client.write_all(b"ping").await?;
    Ok(())
}
```

## Examples

See [examples](https://github.com/akahan/tokio-ipc/tree/main/examples).

## Supported Rust Versions

The MSRV is currently `1.75.0`.

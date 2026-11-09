# Tutorial: Rust

Before you start, have `rustorb-idl` on `PATH`, the naming service running
and `Echo.idl` written ([Tutorial](index.md), steps 1–3).

## Create the project

```sh
cargo new echo-rs
cd echo-rs
cargo add rustorb-core@0.4 rustorb-naming@0.4
cargo add --build rustorb-codegen@0.2
mkdir idl && cp ../Echo.idl idl/
rm src/main.rs && mkdir src/bin
```

The project will have a library with the generated code, and two programs:

```text
echo-rs/
├── Cargo.toml
├── build.rs
├── idl/Echo.idl
└── src/
    ├── lib.rs
    └── bin/
        ├── server.rs
        └── client.rs
```

## Generate the code

`build.rs` runs `rustorb-idl -brust` into `$OUT_DIR/echo.rs`, and `lib.rs`
includes it:

```rust title="build.rs"
fn main() {
    rustorb_codegen::Config::new()
        .compile(&["idl/Echo.idl"])
        .unwrap();
}
```

```rust title="src/lib.rs"
include!(concat!(env!("OUT_DIR"), "/echo.rs"));
```

The IDL module `Example` becomes the Rust module `example`, with

- `Echo`, the object reference type, with a method per operation;
- `EchoServant`, the trait a server implements;
- `EchoPoa`, the wrapper that activates a servant in a POA.

Operations return `Result<_, SystemException>` (or an error enum for
operations with `raises`).

## The server

```rust title="src/bin/server.rs"
use std::error::Error;

use rustorb_core::{Orb, SystemException};
use echo_rs::example::{EchoPoa, EchoServant};
use rustorb_naming::{NamingContext, to_name};

struct EchoImpl {
    orb: Orb,
}

impl EchoServant for EchoImpl {
    fn echo(&self, message: &str) -> Result<String, SystemException> {
        println!("server: {message}");
        Ok(message.to_owned())
    }

    fn shutdown(&self) -> Result<(), SystemException> {
        self.orb.shutdown(false)
    }
}

fn main() -> Result<(), Box<dyn Error>> {
    let mut args: Vec<String> = std::env::args().skip(1).collect();
    let orb = Orb::init(&mut args)?; // takes the -ORB options out of args
    let poa = orb.root_poa()?;
    poa.the_poa_manager().activate()?;
    let (_, echo) = EchoPoa(EchoImpl { orb: orb.clone() }).activate(&poa)?;

    let naming = orb.resolve_initial_references("NameService")?;
    let root = NamingContext::narrow(naming)?.ok_or("NameService is not a NamingContext")?;
    root.rebind(&to_name("Echo")?, echo.object())?;
    println!("server: ready");

    orb.run();
    orb.destroy()?;
    Ok(())
}
```

`Orb::init` reads and removes the `-ORB...` options from `args`. The ORB
calls servants on its own threads, so a servant with state keeps it in a
`Mutex` or the like. `orb.run()` returns after `shutdown`.

## The client

```rust title="src/bin/client.rs"
use std::error::Error;

use rustorb_core::Orb;
use echo_rs::example::Echo;
use rustorb_naming::NamingContextExt;

fn main() -> Result<(), Box<dyn Error>> {
    let mut args: Vec<String> = std::env::args().skip(1).collect();
    let orb = Orb::init(&mut args)?;
    let shutdown = args.iter().any(|a| a == "--shutdown");

    let naming = orb.resolve_initial_references("NameService")?;
    let root = NamingContextExt::narrow(naming)?.ok_or("NameService is not a NamingContextExt")?;
    let obj = root.resolve_str("Echo")?.ok_or("Echo is nil")?;
    let echo = Echo::narrow(&obj)?.ok_or("Echo is not an Example::Echo")?;

    let reply = echo.echo("hello from Rust")?;
    println!("client: {reply}");
    if shutdown {
        echo.shutdown()?;
    }
    orb.destroy()?;
    Ok(())
}
```

`narrow` returns `Ok(None)` when the object is not of that type.

## Build and run

```sh
cargo build
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
./target/debug/server $NS &
./target/debug/client $NS --shutdown
```

```text
server: ready
server: hello from Rust
client: hello from Rust
```

## Next

- [Run them together](together.md) with clients and servers in other languages.
- The Rust mapping in full: [rust-mapping.md](https://github.com/rustorb/rustorb/blob/main/docs/rust-mapping.md);
  the async API: [async-api.md](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md);
  the API on [docs.rs](https://docs.rs/rustorb-core).

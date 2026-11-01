# Install for Rust

rustorb is a set of crates on crates.io. It needs **Rust 1.85** or later
(edition 2024).

| Crate | What it is |
| --- | --- |
| [`rustorb-core`](https://crates.io/crates/rustorb-core) | the ORB: `Orb`, the POA, Any, TypeCode, valuetypes, DII/DSI |
| [`rustorb-naming`](https://crates.io/crates/rustorb-naming) | CosNaming stubs (`NamingContext`, `NamingContextExt`) and the `rustorb-names` daemon |
| [`rustorb-codegen`](https://crates.io/crates/rustorb-codegen) | runs `rustorb-idl -brust` from a build script |
| `rustorb-cdr`, `rustorb-giop` | CDR and GIOP, used by the core |
| `rustorb-ffi` | the C ABI that the C++ and Java bindings use |

## Add the crates

```sh
cargo add rustorb-core@0.1 rustorb-naming@0.1
cargo add --build rustorb-codegen@0.1
```

or by hand in `Cargo.toml`:

```toml
[dependencies]
rustorb-core = "0.1"
rustorb-naming = "0.1"     # if you use the naming service

[build-dependencies]
rustorb-codegen = "0.1"
```

## Install the IDL compiler

`rustorb-codegen` calls the `rustorb-idl` command, so install it once
(see [Install](index.md#the-idl-compiler-rustorb-idl)):

```sh
pip install rustorb-idl     # or: pipx install rustorb-idl
```

If it is not on `PATH`, set `RUSTORB_IDL` to its path when you build.

## Generate code from IDL

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

The output file is named after the IDL file. The [Rust tutorial](../tutorial/rust.md)
shows the whole program.

## Check it

```sh
cargo build
```

The first build compiles tokio and the ORB; later builds only regenerate
the code when the IDL changes.

## Notes

- The API is synchronous: the ORB runs its own tokio runtime inside, and
  calls servants on its own threads. An async API is available too
  ([async-api.md](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md)).
- The API documentation is on [docs.rs](https://docs.rs/rustorb-core).

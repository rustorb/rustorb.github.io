# Naming service

`rustorb-names` is a CosNaming server, a replacement for omniNames. It
serves rustorb, omniORB, omniORBpy and JacORB clients alike.

```sh
cargo install rustorb-naming
```

```text
usage: rustorb-names [-start [PORT]] [-logdir DIR] [-memory] [-ignoreport]
                     [-checkpoint N] [ORB options]

  -start [PORT]   first run: listen on PORT (default 2809) and remember it
  -logdir DIR     where the naming data is kept
                  (default $RUSTORB_NAMES_LOGDIR, else /var/lib/rustorb-names)
  -memory         keep nothing: start empty and forget everything at exit
  -ignoreport     listen only on the -ORBendPoint addresses
  -checkpoint N   write a checkpoint every N updates (default 1000)
```

## Typical use

```sh
rustorb-names -start 2809 -logdir /tmp/rustorb-names   # first run
rustorb-names -logdir /tmp/rustorb-names               # later runs: same port and bindings
rustorb-names -memory -start 2809                      # nothing kept (tests, tutorials)
```

With `-logdir`, every change is appended to a log and a checkpoint is
written every `-checkpoint` updates, so the bindings survive a restart, as
with omniNames.

Programs reach it with

```sh
-ORBInitRef NameService=corbaloc::localhost:2809/NameService
```

## From programs

| Language | Client stubs |
| --- | --- |
| Rust | `rustorb_naming::{NamingContext, NamingContextExt, to_name}` |
| Python | `rustorb-idl -bpython "$(rustorb-idl --idl-dir)/CosNaming.idl"` (or `rustorb-omniorb-compat`) |
| C++ | `rustorb_idl_generate(... "${RUSTORB_IDL_DIR}/CosNaming.idl")` |
| Java | `org.omg.CosNaming` in the jar |

A naming server can also be embedded in a Rust program with the
`rustorb-naming` crate ([docs.rs](https://docs.rs/rustorb-naming)).

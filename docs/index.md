# rustorb

**A CORBA ORB with its core written in Rust, and bindings for Rust, Python,
C++ and Java.**

rustorb speaks GIOP 1.0 to 1.2 over IIOP and SSLIOP, and is tested against
omniORB, omniORBpy and JacORB in every combination of client, server and
naming service. It aims to replace omniORB and omniORBpy in existing
systems: the Python binding generates the same API as `omniidl -bpython`,
and a compatibility package runs omniORBpy applications such as
OpenRTM-aist-Python unchanged.

<div class="grid cards" markdown>

-   :material-rocket-launch: **[Quick start](quickstart.md)**

    A server and a client in Python, in five minutes, with `pip` or with
    Docker.

-   :material-download: **[Install](install/index.md)**

    The packages for Rust, Python, C++ and Java, the IDL compiler and the
    naming service.

-   :material-school: **[Tutorial](tutorial/index.md)**

    One IDL interface, a server and a client in each language, found
    through the naming service, all talking to each other.

-   :material-book-open-variant: **[Reference](reference/orb-options.md)**

    ORB options, the naming service, and the design documents.

</div>

## What is in it

- **Protocol**: CDR, GIOP 1.0–1.2 with fragments, IORs, `corbaloc:` and
  `corbaname:`, code set negotiation, SSLIOP (TLS with rustls)
- **Server side**: the POA with the standard policies, servant managers,
  default servants, adapter activators and the three thread models
- **Types**: Any, TypeCode, DynAny, `fixed`, `long double`, valuetypes
  (custom, truncatable, boxed, `supports`), abstract interfaces
- **Naming Service**: CosNaming client and server, and the `rustorb-names`
  daemon (a replacement for omniNames, with persistence)
- **More**: DII / DSI, policies and call timeouts, an optional async API
  for Rust

## Languages

| Language | Mapping | Package | IDL compiler |
| --- | --- | --- | --- |
| Rust | rustorb's own ([mapping](https://github.com/rustorb/rustorb/blob/main/docs/rust-mapping.md)) | `rustorb-core`, `rustorb-naming` on crates.io | `rustorb-codegen` in `build.rs` |
| Python 3.9+ | the standard IDL to Python mapping, omniORBpy layout | `pip install rustorb` | `rustorb-idl -bpython` |
| C++17 | IDL to C++11 | `rustorb-native` from the GitHub release | `rustorb-idl -bc++11` |
| Java 22+ | IDL to Java (`org.omg.*`), through the FFM API | `rustorb-<version>.jar` from the GitHub release | `rustorb-idl -bjava` |

The current version is **0.1.0** ([release notes](https://github.com/rustorb/rustorb/releases/tag/v0.1.0),
[changelog](https://github.com/rustorb/rustorb/blob/main/CHANGELOG.md)).
Before 1.0, a new minor version may change the API.

## License

rustorb is licensed under either of the Apache License, Version 2.0 or the
MIT license, at your option. The IDL compiler (`rustorb-idl` and the
`rustorb-codegen` crate) is GPL-3.0-or-later, because it builds on
[idl_parser](https://github.com/sugarsweetrobotics/idl_parser); the code it
generates is not covered by that license.

# More documentation

The design documents live in the
[rustorb repository](https://github.com/rustorb/rustorb/tree/main/docs)
(most of them in Japanese).

| Topic | Document |
| --- | --- |
| Rust mapping | [rust-mapping.md](https://github.com/rustorb/rustorb/blob/main/docs/rust-mapping.md) |
| Async API (Rust) | [async-api.md](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md) |
| Python binding | [python-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/python-binding.md), [generator](https://github.com/rustorb/rustorb-idl/blob/main/docs/python-codegen.md) |
| omniORBpy compatibility | [omniorb-compat.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-compat.md) |
| C++ binding | [cpp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/cpp-binding.md) |
| Java binding | [java-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/java-binding.md), [FFM](https://github.com/rustorb/rustorb/blob/main/docs/java-ffm.md) |
| POA | [poa-design.md](https://github.com/rustorb/rustorb/blob/main/docs/poa-design.md) |
| Any and TypeCode | [any-typecode.md](https://github.com/rustorb/rustorb/blob/main/docs/any-typecode.md), [DynAny](https://github.com/rustorb/rustorb/blob/main/docs/dynany.md) |
| Valuetypes | [valuetype.md](https://github.com/rustorb/rustorb/blob/main/docs/valuetype.md) |
| DII / DSI | [dii-dsi.md](https://github.com/rustorb/rustorb/blob/main/docs/dii-dsi.md) |
| SSLIOP | [ssliop.md](https://github.com/rustorb/rustorb/blob/main/docs/ssliop.md) |
| Timeouts | [timeouts.md](https://github.com/rustorb/rustorb/blob/main/docs/timeouts.md) |
| Interoperability tests | [interop/](https://github.com/rustorb/rustorb/tree/main/interop) |
| Releases | [CHANGELOG.md](https://github.com/rustorb/rustorb/blob/main/CHANGELOG.md), [release.md](https://github.com/rustorb/rustorb/blob/main/docs/release.md) |

## What is not there

The CORBA Component Model, the Event and Notification services, the
Interface Repository, CSIv2 and the C++98 mapping are out of scope. IDL 4
`map` and `@optional` have no GIOP encoding and are rejected. The known
differences from other ORBs are listed in
[interop/known-failures.txt](https://github.com/rustorb/rustorb/blob/main/interop/known-failures.txt).

## Help

Questions and bug reports: [GitHub issues](https://github.com/rustorb/rustorb/issues).

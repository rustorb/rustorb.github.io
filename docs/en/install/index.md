# Install

Every language uses the same two tools: the IDL compiler, and (if you use
names rather than IORs) a naming service. Then install the package for
your language.

| Language | Install | Needs |
| --- | --- | --- |
| [Rust](rust.md) | `rustorb-core`, `rustorb-naming`, `rustorb-codegen` from crates.io | Rust 1.85+ |
| [Python](python.md) | `pip install rustorb` | CPython 3.9+ |
| [C++](cpp.md) | `rustorb-native` archive from the GitHub release | a C++17 compiler, CMake 3.16+ |
| [Java](java.md) | `rustorb-<version>.jar` and `rustorb-native` from the GitHub release | Java 22+ |
| [.NET](csharp.md) | `dotnet add package Rustorb` | .NET 8+ |
| [C](c.md) | `rustorb-c` archive from the GitHub release | a C99 compiler |
| [Ruby](ruby.md) | `gem install rustorb` | Ruby 3.0+ with its headers, a C compiler |

## The IDL compiler: rustorb-idl

```sh
pip install rustorb-idl
rustorb-idl --version
```

`rustorb-idl` is developed in [rustorb/rustorb-idl](https://github.com/rustorb/rustorb-idl) and released
on its own (GPL-3.0-or-later; the code it generates is not covered).
It needs Python 3.9 or later and installs
[idl_parser](https://github.com/sugarsweetrobotics/idl_parser). It has one
backend per language:

| Backend | Output |
| --- | --- |
| `-brust` | Rust (usually run for you by `rustorb-codegen` in `build.rs`) |
| `-bpython` | Python, in the layout of `omniidl -bpython` |
| `-bc++11` | C++ (IDL to C++11 mapping) |
| `-bjava` | Java (IDL to Java mapping) |
| `-bcsharp` | C# (IDL4 to C# mapping) |
| `-bc` | C (OMG C mapping) |
| `-bruby` | Ruby (OMG Ruby mapping) |

`-C DIR` writes the output into `DIR` (create it first), and `-I DIR` adds
an include directory. The IDL of the naming service, `CosNaming.idl`,
comes with it: `rustorb-idl --idl-dir` prints the directory it is in, which
is also searched for `#include "CosNaming.idl"`.

!!! tip "Keep it in a virtual environment"
    For C++, Java, .NET, C, Ruby and Rust projects, `rustorb-idl` only has to be on `PATH`
    at build time. `pipx install rustorb-idl` is a convenient way to get it
    there without a virtual environment. Build tools also accept its path
    in the `RUSTORB_IDL` environment variable.

## The naming service: rustorb-names

```sh
cargo install rustorb-naming       # installs the rustorb-names binary
rustorb-names -memory -start 2809  # a naming service on port 2809
```

`cargo install` needs a Rust toolchain ([rustup](https://rustup.rs/)).
Without one, take the prebuilt binary from the
[GitHub release](https://github.com/rustorb/rustorb/releases/latest)
(`rustorb-names-<version>-<platform>`, for Linux, macOS and Windows) and
put it on `PATH`:

```sh
curl -LO https://github.com/rustorb/rustorb/releases/download/v0.7.1/rustorb-names-0.7.1-linux-x86_64.tar.gz
tar xzf rustorb-names-0.7.1-linux-x86_64.tar.gz
./rustorb-names-0.7.1-linux-x86_64/rustorb-names -memory -start 2809
```

Any
CORBA naming service works in place of `rustorb-names`, `omniNames`
included, and `rustorb-names` serves omniORB, omniORBpy and JacORB clients
as well. See [Naming service](../reference/naming.md) for persistence and
options.

## Release files

The C++, Java and C packages, and copies of the Python wheels, crates, NuGet package and gem, are
attached to each [GitHub release](https://github.com/rustorb/rustorb/releases)
with a `SHA256SUMS` file. For 0.7.1:

| File | Contents |
| --- | --- |
| `rustorb-native-0.7.1-linux-x86_64.tar.gz` | `librustorb_ffi` (static and shared), C++ headers, CMake package, `bin/rustorb-names` |
| `rustorb-native-0.7.1-linux-aarch64.tar.gz` | the same, for Linux on 64-bit ARM |
| `rustorb-native-0.7.1-macos-aarch64.tar.gz` | the same, for macOS on Apple silicon |
| `rustorb-native-0.7.1-windows-x86_64.zip` | the same, for Windows x64 |
| `rustorb-names-0.7.1-<platform>.tar.gz` / `.zip` | the `rustorb-names` naming service alone, for the same four platforms |
| `rustorb-0.7.1.jar` | the Java mapping |
| `rustorb-c-0.7.1-<platform>.tar.gz` | the C mapping: headers, `librustorb_c.a`, `librustorb_ffi.a` (Linux x86_64 and aarch64, macOS) |
| `rustorb-c-0.7.1-windows-x86_64.zip` | the C mapping for Windows x64: headers, `rustorb_c.lib`, `rustorb_ffi.lib` (MSVC) |
| `Rustorb.0.7.1.nupkg` | the .NET package, also on nuget.org |
| `Rustorb.Types.0.7.1.nupkg` | the IDL types (`Omg.Types`) that `Rustorb` depends on, also on nuget.org |
| `rustorb-0.7.1.gem` | the Ruby gem, also on rubygems.org |

## From source

Everything can also be built from the [repository](https://github.com/rustorb/rustorb):

```sh
git clone https://github.com/rustorb/rustorb
cd rustorb
cargo build --workspace --release
```

The per-language pages say how to build their package from the tree.

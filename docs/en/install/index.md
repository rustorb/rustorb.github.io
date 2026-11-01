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

## The IDL compiler: rustorb-idl

```sh
pip install rustorb-idl
rustorb-idl --version
```

`rustorb-idl` needs Python 3.9 or later and installs
[idl_parser](https://github.com/sugarsweetrobotics/idl_parser). It has one
backend per language:

| Backend | Output |
| --- | --- |
| `-brust` | Rust (usually run for you by `rustorb-codegen` in `build.rs`) |
| `-bpython` | Python, in the layout of `omniidl -bpython` |
| `-bc++11` | C++ (IDL to C++11 mapping) |
| `-bjava` | Java (IDL to Java mapping) |

`-C DIR` writes the output into `DIR` (create it first), and `-I DIR` adds
an include directory. The IDL of the naming service, `CosNaming.idl`,
comes with it: `rustorb-idl --idl-dir` prints the directory it is in, which
is also searched for `#include "CosNaming.idl"`.

!!! tip "Keep it in a virtual environment"
    For C++, Java and Rust projects, `rustorb-idl` only has to be on `PATH`
    at build time. `pipx install rustorb-idl` is a convenient way to get it
    there without a virtual environment. Build tools also accept its path
    in the `RUSTORB_IDL` environment variable.

## The naming service: rustorb-names

```sh
cargo install rustorb-naming       # installs the rustorb-names binary
rustorb-names -memory -start 2809  # a naming service on port 2809
```

`cargo install` needs a Rust toolchain ([rustup](https://rustup.rs/)). Any
CORBA naming service works in place of `rustorb-names`, `omniNames`
included, and `rustorb-names` serves omniORB, omniORBpy and JacORB clients
as well. See [Naming service](../reference/naming.md) for persistence and
options.

## Release files

The C++ and Java packages, and copies of the Python wheels and crates, are
attached to each [GitHub release](https://github.com/rustorb/rustorb/releases)
with a `SHA256SUMS` file. For 0.1.0:

| File | Contents |
| --- | --- |
| `rustorb-native-0.1.0-linux-x86_64.tar.gz` | `librustorb_ffi` (static and shared), C++ headers, CMake package |
| `rustorb-native-0.1.0-linux-aarch64.tar.gz` | the same, for Linux on 64-bit ARM |
| `rustorb-native-0.1.0-macos-aarch64.tar.gz` | the same, for macOS on Apple silicon |
| `rustorb-native-0.1.0-windows-x86_64.zip` | the same, for Windows x64 |
| `rustorb-0.1.0.jar` | the Java mapping |

## From source

Everything can also be built from the [repository](https://github.com/rustorb/rustorb):

```sh
git clone https://github.com/rustorb/rustorb
cd rustorb
cargo build --workspace --release
```

The per-language pages say how to build their package from the tree.

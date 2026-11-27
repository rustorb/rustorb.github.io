# Install rustorb-dds

Install the package for your language, and the IDL compiler
`rustorb-idl` to make the types of your topics from IDL.

| Language | Install | Needs |
| --- | --- | --- |
| [Rust](#rust) | `rustorb-dds` and `rustorb-typesupport` from crates.io | Rust 1.89+ |
| [Python](#python) | `pip install rustorb-dds` | CPython 3.9+ |
| [C++](#c) | build and install `cpp/` from the repository | a C++17 compiler, CMake, Rust |
| [Java](#java) | build `java/` from the repository with Maven | Java 22+, Maven, Rust |
| [.NET](#net) | `dotnet add package Rustorb.Dds` | .NET 8+ |
| [C](#c_1) | `rustorb-dds-native` archive from the GitHub release | a C11 compiler |
| [Ruby](#ruby) | `gem install rustorb-dds` | Ruby 3.0+ with its headers, a C compiler (Linux, macOS) |

## The IDL compiler: rustorb-idl

```sh
pip install rustorb-idl      # or: pipx install rustorb-idl
rustorb-idl --version
```

It is the same compiler as for the ORB (see
[Install](../install/index.md#the-idl-compiler-rustorb-idl)). For DDS, add
`-Wbdds_only` to the language's backend: the output then has only the
DDS types, and does not need the ORB.

| Language | Command | Output |
| --- | --- | --- |
| Rust | `rustorb_codegen::Config::new().dds_only()` in `build.rs` | `$OUT_DIR/<name>.rs` |
| Python | `rustorb-idl -bpython -Wbdds_only Foo.idl` | `Foo_dds.py` |
| C++ | `rustorb-idl -bc++11 -Wbdds_only Foo.idl` | `Foo.hh`, `Foo_dds.hh` |
| Java | `rustorb-idl -bjava -Wbdds_only Foo.idl` | a class and a `<Name>Codec` per type |
| C# | `rustorb-idl -bcsharp -Wbdds_only Foo.idl` | `Foo.cs`, `Foo_dds.cs` |
| Ruby | `rustorb-idl -bruby -Wbdds_only Foo.idl` | `Foo_dds.rb` |

With `-Wbdds` instead, the same types also get the CORBA code, so that one
type can travel over both CORBA and DDS (as OpenRTM-aist does).

## Rust

```toml
[dependencies]
rustorb-dds = "0.3"
rustorb-typesupport = "0.6"   # used by the generated code

[build-dependencies]
rustorb-codegen = "0.6"       # runs rustorb-idl from build.rs
```

Cargo features of `rustorb-dds`:

| Feature | Default | What it adds |
| --- | --- | --- |
| `shm` | on | the shared memory transport between participants of one host |
| `security` | off | DDS Security 1.2 with the builtin plugins (`rustorb-dds-security`, on RustCrypto) |

The other crates are `rustorb-rtps` (the wire protocol), `rustorb-xtypes`
(the type system and DynamicData), `rustorb-dds-security`,
`rustorb-dds-ffi` (the C ABI) and `rustorb-dds-corba` (CORBA TypeCodes and
anys as XTypes types).

## Python

```sh
pip install rustorb-dds
python -c "import rustorb_dds; print(rustorb_dds.__version__)"
```

Wheels are built for Linux x86_64 and aarch64 (manylinux 2.28), macOS
x86_64 and arm64, and Windows x64; one abi3 wheel serves CPython 3.9 and
later. The module is `rustorb_dds`. Types can come from
`rustorb-idl -bpython -Wbdds_only`, or be declared in Python with
`@dds.struct` (see the
[package's README](https://github.com/rustorb/rustorb-dds/tree/main/python)).

## C++

The C++ binding is header only, on the C library. It is not packaged
yet: build and install it from the repository (this builds the C library
with Cargo too).

```sh
git clone https://github.com/rustorb/rustorb-dds
cmake -S rustorb-dds/cpp -B build/cpp -DCMAKE_BUILD_TYPE=Release
cmake --build build/cpp
cmake --install build/cpp --prefix "$HOME/.local"
```

In your `CMakeLists.txt`:

```cmake
find_package(rustorb_dds REQUIRED)
target_link_libraries(app PRIVATE rustorb_dds::rustorb_dds)
```

## Java

The Java binding uses the FFM API (`java.lang.foreign`), so it needs
Java 22 or later, and no JNI. It is not on Maven Central yet: build it
from the repository (Maven builds the C library with Cargo).

```sh
git clone https://github.com/rustorb/rustorb-dds
mvn -f rustorb-dds/java/pom.xml install
```

This makes `java/target/rustorb-dds-0.3.0.jar` and
`target/release/librustorb_dds_ffi.so` (`.dylib`, `.dll`), and installs
the jar in the local Maven repository as `org.rustorb:rustorb-dds:0.3.0`.
Run your program with `--enable-native-access=ALL-UNNAMED`, and tell it
where `librustorb_dds_ffi` is: the system property
`rustorb.dds.ffi.library`, the environment variable
`RUSTORB_DDS_FFI_LIBRARY`, or `java.library.path`. The shared library
from the [C archive](#c_1) works too.

## .NET

```sh
dotnet add package Rustorb.Dds
```

The package carries the native library for `linux-x64`, `linux-arm64`,
`osx-arm64`, `osx-x64` and `win-x64`. Its sequence types come from the
small package `Rustorb.Types`, which it depends on.

## C

Download `rustorb-dds-native-<version>-<platform>` from the
[GitHub release](https://github.com/rustorb/rustorb-dds/releases/latest)
(`.tar.gz`, or `.zip` on Windows). It has `include/rustorb/dds.h` and
`librustorb_dds_ffi`, shared and static, for Linux x86_64 and aarch64
(glibc 2.28), macOS arm64 and x86_64, and Windows x64.

```sh
curl -LO https://github.com/rustorb/rustorb-dds/releases/download/v0.3.0/rustorb-dds-native-0.3.0-linux-x86_64.tar.gz
tar xzf rustorb-dds-native-0.3.0-linux-x86_64.tar.gz
cc -std=c11 -Irustorb-dds-native-0.3.0-linux-x86_64/include main.c \
   rustorb-dds-native-0.3.0-linux-x86_64/lib/librustorb_dds_ffi.a -lpthread -ldl -lm
```

Every function starts with `rdds_`; the rules for callers are at the top
of the header. The C ABI is described in
[docs/c-abi.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/c-abi.md)
(in Japanese).

## Ruby

```sh
gem install rustorb-dds
ruby -e 'require "rustorb/dds"; puts DDS::VERSION'
```

The gem carries the static C library for Linux x86_64 and aarch64 and
macOS arm64 and x86_64, and builds its C extension when it is installed.
Windows is not supported yet.

## Release files

The wheels, the sdist, the crates, the NuGet package, the gem and the C
archives are attached to each
[GitHub release](https://github.com/rustorb/rustorb-dds/releases) with a
`SHA256SUMS` file.

# rustorb-dds のインストール

使う言語のパッケージと、トピックの型を IDL から作る IDL コンパイラ `rustorb-idl` をインストールします。

| 言語 | インストール | 必要なもの |
| --- | --- | --- |
| [Rust](#rust) | crates.io の `rustorb-dds` と `rustorb-typesupport` | Rust 1.89+ |
| [Python](#python) | `pip install rustorb-dds` | CPython 3.9+ |
| [C++](#c) | リポジトリの `cpp/` をビルドしてインストール | C++17 コンパイラ、CMake、Rust |
| [Java](#java) | リポジトリの `java/` を Maven でビルド | Java 22+、Maven、Rust |
| [.NET](#net) | `dotnet add package Rustorb.Dds` | .NET 8+ |
| [C](#c_1) | GitHub Release の `rustorb-dds-native` アーカイブ | C11 コンパイラ |
| [Ruby](#ruby) | `gem install rustorb-dds` | Ruby 3.0+ とそのヘッダ、C コンパイラ（Linux、macOS） |

## IDL コンパイラ: rustorb-idl

```sh
pip install rustorb-idl      # または pipx install rustorb-idl
rustorb-idl --version
```

ORB で使うものと同じコンパイラです（[インストール](../install/index.md#the-idl-compiler-rustorb-idl)を参照）。DDS では、言語のバックエンドに `-Wbdds_only` を付けます。出力は DDS の型だけになり、ORB は要りません。

| 言語 | コマンド | 出力 |
| --- | --- | --- |
| Rust | `build.rs` で `rustorb_codegen::Config::new().dds_only()` | `$OUT_DIR/<name>.rs` |
| Python | `rustorb-idl -bpython -Wbdds_only Foo.idl` | `Foo_dds.py` |
| C++ | `rustorb-idl -bc++11 -Wbdds_only Foo.idl` | `Foo.hh`、`Foo_dds.hh` |
| Java | `rustorb-idl -bjava -Wbdds_only Foo.idl` | 型ごとのクラスと `<Name>Codec` |
| C# | `rustorb-idl -bcsharp -Wbdds_only Foo.idl` | `Foo.cs`、`Foo_dds.cs` |
| Ruby | `rustorb-idl -bruby -Wbdds_only Foo.idl` | `Foo_dds.rb` |

`-Wbdds` にすると、同じ型に CORBA のコードも付きます。1 つの型を CORBA と DDS の両方で使えます（OpenRTM-aist のように）。

## Rust

```toml
[dependencies]
rustorb-dds = "0.3"
rustorb-typesupport = "0.6"   # 生成コードが使う

[build-dependencies]
rustorb-codegen = "0.6"       # build.rs から rustorb-idl を実行する
```

`rustorb-dds` の Cargo の feature:

| feature | 既定 | 加わるもの |
| --- | --- | --- |
| `shm` | 有効 | 同じホストの参加者どうしの共有メモリ転送 |
| `security` | 無効 | 組み込みプラグインによる DDS Security 1.2（`rustorb-dds-security`、RustCrypto を使用） |

ほかのクレートは `rustorb-rtps`（通信プロトコル）、`rustorb-xtypes`（型システムと DynamicData）、`rustorb-dds-security`、`rustorb-dds-ffi`（C ABI）、`rustorb-dds-corba`（CORBA の TypeCode と any を XTypes の型として扱う）です。

## Python

```sh
pip install rustorb-dds
python -c "import rustorb_dds; print(rustorb_dds.__version__)"
```

wheel は Linux x86_64・aarch64（manylinux 2.28）、macOS x86_64・arm64、Windows x64 向けにあり、1 つの abi3 wheel で CPython 3.9 以降に対応します。モジュール名は `rustorb_dds` です。型は `rustorb-idl -bpython -Wbdds_only` で生成するか、Python で `@dds.struct` を使って宣言します（[パッケージの README](https://github.com/rustorb/rustorb-dds/tree/main/python) を参照）。

## C++

C++ バインディングは C ライブラリの上のヘッダだけのライブラリです。まだパッケージにしていないので、リポジトリからビルドしてインストールします（C ライブラリも Cargo でビルドされます）。

```sh
git clone https://github.com/rustorb/rustorb-dds
cmake -S rustorb-dds/cpp -B build/cpp -DCMAKE_BUILD_TYPE=Release
cmake --build build/cpp
cmake --install build/cpp --prefix "$HOME/.local"
```

`CMakeLists.txt` では:

```cmake
find_package(rustorb_dds REQUIRED)
target_link_libraries(app PRIVATE rustorb_dds::rustorb_dds)
```

## Java

Java バインディングは FFM API（`java.lang.foreign`）を使うので Java 22 以降が必要です。JNI は使いません。まだ Maven Central にはないので、リポジトリからビルドします（Maven が C ライブラリを Cargo でビルドします）。

```sh
git clone https://github.com/rustorb/rustorb-dds
mvn -f rustorb-dds/java/pom.xml install
```

`java/target/rustorb-dds-0.3.0.jar` と `target/release/librustorb_dds_ffi.so`（`.dylib`、`.dll`）ができ、jar はローカルの Maven リポジトリに `org.rustorb:rustorb-dds:0.3.0` として入ります。プログラムは `--enable-native-access=ALL-UNNAMED` を付けて実行し、`librustorb_dds_ffi` の場所を、システムプロパティ `rustorb.dds.ffi.library`、環境変数 `RUSTORB_DDS_FFI_LIBRARY`、`java.library.path` のどれかで教えます。[C のアーカイブ](#c_1)の共有ライブラリも使えます。

## .NET

```sh
dotnet add package Rustorb.Dds
```

パッケージには `linux-x64`、`linux-arm64`、`osx-arm64`、`osx-x64`、`win-x64` のネイティブライブラリが入っています。列の型は、依存先の小さなパッケージ `Rustorb.Types` にあります。

## C

[GitHub Release](https://github.com/rustorb/rustorb-dds/releases/latest) から `rustorb-dds-native-<version>-<platform>`（`.tar.gz`、Windows は `.zip`）をダウンロードします。`include/rustorb/dds.h` と、共有・静的の `librustorb_dds_ffi` が入っています。Linux x86_64・aarch64（glibc 2.28）、macOS arm64・x86_64、Windows x64 向けです。

```sh
curl -LO https://github.com/rustorb/rustorb-dds/releases/download/v0.3.0/rustorb-dds-native-0.3.0-linux-x86_64.tar.gz
tar xzf rustorb-dds-native-0.3.0-linux-x86_64.tar.gz
cc -std=c11 -Irustorb-dds-native-0.3.0-linux-x86_64/include main.c \
   rustorb-dds-native-0.3.0-linux-x86_64/lib/librustorb_dds_ffi.a -lpthread -ldl -lm
```

関数はすべて `rdds_` で始まり、呼び出し側の決まりはヘッダの先頭にあります。C ABI の設計は [docs/c-abi.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/c-abi.md) にあります。

## Ruby

```sh
gem install rustorb-dds
ruby -e 'require "rustorb/dds"; puts DDS::VERSION'
```

gem には Linux x86_64・aarch64、macOS arm64・x86_64 の静的 C ライブラリが入っていて、インストール時に C 拡張をビルドします。Windows にはまだ対応していません。

## リリースのファイル

wheel、sdist、クレート、NuGet パッケージ、gem、C のアーカイブは、`SHA256SUMS` と一緒に各 [GitHub Release](https://github.com/rustorb/rustorb-dds/releases) に付いています。

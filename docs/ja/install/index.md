# インストール

どの言語でも共通して使うツールが 2 つあります。IDL コンパイラと、IOR ではなく名前でオブジェクトを探す場合のネーミングサービスです。そのうえで、使う言語のパッケージを入れます。

| 言語 | 入れるもの | 必要なもの |
| --- | --- | --- |
| [Rust](rust.md) | crates.io の `rustorb-core`、`rustorb-naming`、`rustorb-codegen` | Rust 1.85 以降 |
| [Python](python.md) | `pip install rustorb` | CPython 3.9 以降 |
| [C++](cpp.md) | GitHub Release の `rustorb-native` アーカイブ | C++17 コンパイラ、CMake 3.16 以降 |
| [Java](java.md) | GitHub Release の `rustorb-<版>.jar` と `rustorb-native` | Java 22 以降 |

## IDL コンパイラ: rustorb-idl {#the-idl-compiler-rustorb-idl}

```sh
pip install rustorb-idl
rustorb-idl --version
```

`rustorb-idl` は Python 3.9 以降で動き、[idl_parser](https://github.com/sugarsweetrobotics/idl_parser) も一緒に入ります。言語ごとにバックエンドがあります。

| バックエンド | 出力 |
| --- | --- |
| `-brust` | Rust（ふつうは `build.rs` の `rustorb-codegen` が呼びます） |
| `-bpython` | Python（`omniidl -bpython` と同じ配置） |
| `-bc++11` | C++（IDL to C++11 マッピング） |
| `-bjava` | Java（IDL to Java マッピング） |

`-C DIR` で出力先を `DIR` にします（先にディレクトリを作っておいてください）。`-I DIR` でインクルードディレクトリを足せます。ネーミングサービスの IDL `CosNaming.idl` は rustorb-idl に同梱されています。`rustorb-idl --idl-dir` でそのディレクトリがわかり、`#include "CosNaming.idl"` もそこから探されます。

!!! tip "仮想環境に入れておく"
    C++、Java、Rust のプロジェクトでは、`rustorb-idl` はビルドのときに `PATH` にあれば十分です。`pipx install rustorb-idl` を使うと、仮想環境なしで `PATH` に置けて便利です。ビルドツールは環境変数 `RUSTORB_IDL` でパスを指定することもできます。

## ネーミングサービス: rustorb-names

```sh
cargo install rustorb-naming       # rustorb-names コマンドが入ります
rustorb-names -memory -start 2809  # ポート 2809 でネーミングサービスを起動
```

`cargo install` には Rust のツールチェイン（[rustup](https://rustup.rs/)）が要ります。`rustorb-names` の代わりに `omniNames` など他の CORBA ネーミングサービスを使っても構いません。`rustorb-names` は omniORB、omniORBpy、JacORB のクライアントにも使えます。永続化とオプションは[ネーミングサービス](../reference/naming.md)を見てください。

## リリースのファイル

C++ と Java のパッケージ、そして Python の wheel とクレートのコピーは、各[GitHub Release](https://github.com/rustorb/rustorb/releases) に `SHA256SUMS` と一緒に付いています。0.1.0 では次のとおりです。

| ファイル | 中身 |
| --- | --- |
| `rustorb-native-0.1.0-linux-x86_64.tar.gz` | `librustorb_ffi`（静的・共有）、C++ ヘッダ、CMake パッケージ |
| `rustorb-native-0.1.0-linux-aarch64.tar.gz` | 同上、64 ビット ARM の Linux 用 |
| `rustorb-native-0.1.0-macos-aarch64.tar.gz` | 同上、Apple シリコンの macOS 用 |
| `rustorb-native-0.1.0-windows-x86_64.zip` | 同上、Windows x64 用 |
| `rustorb-0.1.0.jar` | Java のマッピング |

## ソースから

すべて[リポジトリ](https://github.com/rustorb/rustorb)からビルドすることもできます。

```sh
git clone https://github.com/rustorb/rustorb
cd rustorb
cargo build --workspace --release
```

各言語のページに、ソースツリーからパッケージを作る方法を書いています。

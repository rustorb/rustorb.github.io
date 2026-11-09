# Rust へのインストール

rustorb は crates.io のクレート群です。**Rust 1.85** 以降（edition 2024）が必要です。

| クレート | 中身 |
| --- | --- |
| [`rustorb-core`](https://crates.io/crates/rustorb-core) | ORB 本体: `Orb`、POA、Any、TypeCode、valuetype、DII/DSI |
| [`rustorb-naming`](https://crates.io/crates/rustorb-naming) | CosNaming のスタブ（`NamingContext`、`NamingContextExt`）と `rustorb-names` デーモン |
| [`rustorb-codegen`](https://crates.io/crates/rustorb-codegen) | ビルドスクリプトから `rustorb-idl -brust` を実行する |
| `rustorb-cdr`、`rustorb-giop` | CDR と GIOP（コアが使う） |
| `rustorb-ffi` | C++ と Java のバインディングが使う C ABI |

## クレートを追加する

```sh
cargo add rustorb-core@0.4 rustorb-naming@0.4
cargo add --build rustorb-codegen@0.2
```

`Cargo.toml` に直接書く場合:

```toml
[dependencies]
rustorb-core = "0.4"
rustorb-naming = "0.4"     # ネーミングサービスを使う場合

[build-dependencies]
rustorb-codegen = "0.2"
```

## IDL コンパイラを入れる

`rustorb-codegen` は `rustorb-idl` コマンドを呼ぶので、一度入れておきます（[インストール](index.md#the-idl-compiler-rustorb-idl)を参照）。

```sh
pip install rustorb-idl     # または pipx install rustorb-idl
```

`PATH` にない場合は、ビルド時に `RUSTORB_IDL` でパスを指定します。

## IDL からコードを生成する

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

出力ファイルの名前は IDL ファイルの名前から決まります。プログラム全体は[Rust のチュートリアル](../tutorial/rust.md)にあります。

## 確かめる

```sh
cargo build
```

最初のビルドでは tokio と ORB をコンパイルします。以降は IDL を変えたときだけコードを生成し直します。

## 補足

- API は同期です。ORB は内部で自分の tokio ランタイムを動かし、自分のスレッドでサーバントを呼びます。非同期 API もあります（[async-api.md](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md)）。
- API ドキュメントは [docs.rs](https://docs.rs/rustorb-core) にあります。

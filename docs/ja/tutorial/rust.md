# チュートリアル: Rust

始める前に、`rustorb-idl` を `PATH` に置き、ネーミングサービスを起動し、`Echo.idl` を書いておきます（[チュートリアル](index.md)の手順 1〜3）。

## プロジェクトを作る

```sh
cargo new echo-rs
cd echo-rs
cargo add rustorb-core@0.1 rustorb-naming@0.1
cargo add --build rustorb-codegen@0.1
mkdir idl && cp ../Echo.idl idl/
rm src/main.rs && mkdir src/bin
```

生成したコードを持つライブラリと、2 つのプログラムからなるプロジェクトです。

```text
echo-rs/
├── Cargo.toml
├── build.rs
├── idl/Echo.idl
└── src/
    ├── lib.rs
    └── bin/
        ├── server.rs
        └── client.rs
```

## コードを生成する

`build.rs` が `rustorb-idl -brust` を実行して `$OUT_DIR/echo.rs` を作り、`lib.rs` がそれを取り込みます。

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

IDL のモジュール `Example` は Rust のモジュール `example` になり、次のものができます。

- `Echo`: オブジェクト参照の型。オペレーションごとにメソッドを持ちます
- `EchoServant`: サーバが実装するトレイト
- `EchoPoa`: サーバントを POA で活性化するためのラッパ

オペレーションは `Result<_, SystemException>` を返します（`raises` のあるオペレーションはエラーの enum を返します）。

## サーバ

```rust title="src/bin/server.rs"
use std::error::Error;

use rustorb_core::{Orb, SystemException};
use echo_rs::example::{EchoPoa, EchoServant};
use rustorb_naming::{NamingContext, to_name};

struct EchoImpl {
    orb: Orb,
}

impl EchoServant for EchoImpl {
    fn echo(&self, message: &str) -> Result<String, SystemException> {
        println!("server: {message}");
        Ok(message.to_owned())
    }

    fn shutdown(&self) -> Result<(), SystemException> {
        self.orb.shutdown(false)
    }
}

fn main() -> Result<(), Box<dyn Error>> {
    let mut args: Vec<String> = std::env::args().skip(1).collect();
    let orb = Orb::init(&mut args)?; // takes the -ORB options out of args
    let poa = orb.root_poa()?;
    poa.the_poa_manager().activate()?;
    let (_, echo) = EchoPoa(EchoImpl { orb: orb.clone() }).activate(&poa)?;

    let naming = orb.resolve_initial_references("NameService")?;
    let root = NamingContext::narrow(naming)?.ok_or("NameService is not a NamingContext")?;
    root.rebind(&to_name("Echo")?, echo.object())?;
    println!("server: ready");

    orb.run();
    orb.destroy()?;
    Ok(())
}
```

`Orb::init` は `args` から `-ORB...` のオプションを読み取って取り除きます。ORB は自分のスレッドでサーバントを呼ぶので、状態を持つサーバントは `Mutex` などに入れます。`orb.run()` は `shutdown` のあとに戻ります。

## クライアント

```rust title="src/bin/client.rs"
use std::error::Error;

use rustorb_core::Orb;
use echo_rs::example::Echo;
use rustorb_naming::NamingContextExt;

fn main() -> Result<(), Box<dyn Error>> {
    let mut args: Vec<String> = std::env::args().skip(1).collect();
    let orb = Orb::init(&mut args)?;
    let shutdown = args.iter().any(|a| a == "--shutdown");

    let naming = orb.resolve_initial_references("NameService")?;
    let root = NamingContextExt::narrow(naming)?.ok_or("NameService is not a NamingContextExt")?;
    let obj = root.resolve_str("Echo")?.ok_or("Echo is nil")?;
    let echo = Echo::narrow(&obj)?.ok_or("Echo is not an Example::Echo")?;

    let reply = echo.echo("hello from Rust")?;
    println!("client: {reply}");
    if shutdown {
        echo.shutdown()?;
    }
    orb.destroy()?;
    Ok(())
}
```

`narrow` はオブジェクトがその型でないとき `Ok(None)` を返します。

## ビルドして動かす

```sh
cargo build
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
./target/debug/server $NS &
./target/debug/client $NS --shutdown
```

```text
server: ready
server: hello from Rust
client: hello from Rust
```

## 次に

- 他の言語のクライアントやサーバと[つないで動かす](together.md)。
- Rust マッピングの全体は [rust-mapping.md](https://github.com/rustorb/rustorb/blob/main/docs/rust-mapping.md)、非同期 API は [async-api.md](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md)、API は [docs.rs](https://docs.rs/rustorb-core) にあります。

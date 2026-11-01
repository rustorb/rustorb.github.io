# チュートリアル

このチュートリアルでは、小さなシステムを各言語で作ります。IDL のインタフェースを実装してネーミングサービスに `Echo` という名前で登録する**サーバ**と、`Echo` を探して呼び出す**クライアント**です。4 つのサーバと 4 つのクライアントは入れ替えが利き、どのクライアントもどのサーバとつながります。omniORB、omniORBpy、JacORB のものともつながります。

同じプログラムがリポジトリの[examples/](https://github.com/rustorb/rustorb/tree/main/examples) にあり、CI ですべてのサーバとすべてのクライアントの組み合わせを動かしています。

## 1. ツールを入れる

IDL コンパイラとネーミングサービスが要ります（[インストール](../install/index.md)を参照）。

```sh
pip install rustorb-idl
cargo install rustorb-naming
```

## 2. ネーミングサービスを起動する

専用のターミナルで:

```sh
rustorb-names -memory -start 2809
```

これでルートのネーミングコンテキストが `corbaloc::localhost:2809/NameService` になります。このチュートリアルのプログラムはどれも、標準の ORB オプション

```sh
-ORBInitRef NameService=corbaloc::localhost:2809/NameService
```

でこれを見つけます。どのバインディングもコマンドラインからこのオプションを受け取ります。（`-memory` は終了すると名前を忘れます。残したい場合は[ネーミングサービス](../reference/naming.md)を見てください。）

## 3. IDL を書く

どの言語でも同じ `Echo.idl` を使います。

```idl title="Echo.idl"
module Example {
  interface Echo {
    string echo(in string message);
    oneway void shutdown();
  };
};
```

`echo` は引数をそのまま返し、`shutdown` はサーバに停止を頼みます。`shutdown` は `oneway` なので、クライアントは応答を待ちません。

## 4. 言語を選ぶ

<div class="grid cards" markdown>

-   **[Rust](rust.md)**: crates.io、`build.rs` でコードを生成
-   **[Python](python.md)**: `pip install rustorb`、omniORBpy と同じ配置
-   **[C++](cpp.md)**: IDL to C++11 マッピング、CMake
-   **[Java](java.md)**: IDL to Java マッピング、Java 22 以降

</div>

最後に[つないで動かし](together.md)ます。

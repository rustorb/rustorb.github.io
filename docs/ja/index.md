<p class="rustorb-hero" markdown>
![rustorb](../assets/images/rustorb-wordmark-light.svg#only-light){ width="480" }
![rustorb](../assets/images/rustorb-wordmark-dark.svg#only-dark){ width="480" }
</p>

# rustorb { .rustorb-sr-only }

**Rust でコアを書いた CORBA ORB です。Rust、Python、C++、Java から使えます。**

rustorb は IIOP と SSLIOP の上で GIOP 1.0〜1.2 を話し、omniORB、omniORBpy、JacORB とのあいだで、クライアント・サーバ・ネーミングサービスのすべての組み合わせをテストしています。既存システムの omniORB / omniORBpy を置き換えることを目指しており、Python バインディングは `omniidl -bpython` と同じ API を生成します。互換パッケージを使えば、OpenRTM-aist-Python のような omniORBpy のアプリケーションをそのまま動かせます。

<div class="grid cards" markdown>

-   :material-rocket-launch: **[クイックスタート](quickstart.md)**

    `pip` か Docker で、Python のサーバとクライアントを 5 分で動かします。

-   :material-download: **[インストール](install/index.md)**

    Rust、Python、C++、Java のパッケージと、IDL コンパイラ、ネーミングサービス。

-   :material-school: **[チュートリアル](tutorial/index.md)**

    1 つの IDL インタフェースについて、各言語でサーバとクライアントを書き、ネーミングサービス経由でつなぎます。

-   :material-book-open-variant: **[リファレンス](reference/orb-options.md)**

    ORB オプション、ネーミングサービス、設計ドキュメント。

</div>

## できること

- **プロトコル**: CDR、フラグメントを含む GIOP 1.0〜1.2、IOR、`corbaloc:` と `corbaname:`、コードセットのネゴシエーション、SSLIOP（rustls による TLS）
- **サーバ側**: 標準ポリシーを備えた POA、サーバントマネージャ、デフォルトサーバント、アダプタアクティベータ、3 つのスレッドモデル
- **型**: Any、TypeCode、DynAny、`fixed`、`long double`、valuetype（custom、truncatable、box、`supports`）、abstract interface
- **ネーミングサービス**: CosNaming のクライアントとサーバ、`rustorb-names` デーモン（永続化つきの omniNames の代替）
- **その他**: DII / DSI、ポリシーと呼び出しタイムアウト、Rust の非同期 API

## 対応言語

| 言語 | マッピング | パッケージ | IDL コンパイラ |
| --- | --- | --- | --- |
| Rust | rustorb 独自（[マッピング](https://github.com/rustorb/rustorb/blob/main/docs/rust-mapping.md)） | crates.io の `rustorb-core`、`rustorb-naming` | `build.rs` で `rustorb-codegen` |
| Python 3.9+ | 標準の IDL to Python マッピング、omniORBpy と同じ配置 | `pip install rustorb` | `rustorb-idl -bpython` |
| C++17 | IDL to C++11 | GitHub Release の `rustorb-native` | `rustorb-idl -bc++11` |
| Java 22+ | IDL to Java（`org.omg.*`）、FFM API 経由 | GitHub Release の `rustorb-<版>.jar` | `rustorb-idl -bjava` |

現在の版は **0.1.0** です（[リリースノート](https://github.com/rustorb/rustorb/releases/tag/v0.1.0)、[変更履歴](https://github.com/rustorb/rustorb/blob/main/CHANGELOG.md)）。1.0 までは、マイナーバージョンが上がると API が変わることがあります。

## ライセンス

rustorb は Apache License 2.0 と MIT ライセンスのデュアルライセンスで、どちらかを選べます。IDL コンパイラ（`rustorb-idl` と `rustorb-codegen` クレート）だけは[idl_parser](https://github.com/sugarsweetrobotics/idl_parser) を使うため GPL-3.0-or-later です。生成されたコードはこのライセンスの対象になりません。

# その他のドキュメント

機能ごとの使い方は[ガイド](../guides/index.md)にあります。

設計ドキュメントは [rustorb のリポジトリ](https://github.com/rustorb/rustorb/tree/main/docs)にあります（ほとんどが日本語です）。

| 内容 | ドキュメント |
| --- | --- |
| Rust マッピング | [rust-mapping.md](https://github.com/rustorb/rustorb/blob/main/docs/rust-mapping.md) |
| 非同期 API（Rust） | [async-api.md](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md) |
| Python バインディング | [python-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/python-binding.md), [ジェネレータ](https://github.com/rustorb/rustorb-idl/blob/main/docs/python-codegen.md) |
| omniORBpy 互換 | [omniorb-compat.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-compat.md) |
| C++ バインディング | [cpp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/cpp-binding.md) |
| Java バインディング | [java-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/java-binding.md), [FFM](https://github.com/rustorb/rustorb/blob/main/docs/java-ffm.md) |
| POA | [poa-design.md](https://github.com/rustorb/rustorb/blob/main/docs/poa-design.md) |
| Any と TypeCode | [any-typecode.md](https://github.com/rustorb/rustorb/blob/main/docs/any-typecode.md), [DynAny](https://github.com/rustorb/rustorb/blob/main/docs/dynany.md) |
| valuetype | [valuetype.md](https://github.com/rustorb/rustorb/blob/main/docs/valuetype.md) |
| DII / DSI | [dii-dsi.md](https://github.com/rustorb/rustorb/blob/main/docs/dii-dsi.md) |
| SSLIOP | [ssliop.md](https://github.com/rustorb/rustorb/blob/main/docs/ssliop.md) |
| Unix ドメインソケット（`giop:unix`） | [unix-transport.md](https://github.com/rustorb/rustorb/blob/main/docs/unix-transport.md) |
| 双方向 GIOP | [bidir-giop.md](https://github.com/rustorb/rustorb/blob/main/docs/bidir-giop.md) |
| 接続 | [connections.md](https://github.com/rustorb/rustorb/blob/main/docs/connections.md) |
| Portable Interceptors | [portable-interceptors.md](https://github.com/rustorb/rustorb/blob/main/docs/portable-interceptors.md) |
| トレース | [tracing.md](https://github.com/rustorb/rustorb/blob/main/docs/tracing.md) |
| タイムアウト | [timeouts.md](https://github.com/rustorb/rustorb/blob/main/docs/timeouts.md) |
| SyncScope（oneway 呼び出し） | [sync-scope.md](https://github.com/rustorb/rustorb/blob/main/docs/sync-scope.md) |
| コールバック型 AMI（C++・Java・Python） | [ami.md](https://github.com/rustorb/rustorb/blob/main/docs/ami.md) |
| 相互接続テスト | [interop/](https://github.com/rustorb/rustorb/tree/main/interop) |
| リリース | [CHANGELOG.md](https://github.com/rustorb/rustorb/blob/main/CHANGELOG.md), [release.md](https://github.com/rustorb/rustorb/blob/main/docs/release.md) |

## 対象外のもの

CORBA Component Model、イベントサービスと通知サービス、インタフェースリポジトリ、CSIv2、C++98 マッピングは対象外です。IDL 4 の `map` と `@optional` は GIOP でのエンコードが決まっていないため、エラーになります。他の ORB との既知の違いは[interop/known-failures.txt](https://github.com/rustorb/rustorb/blob/main/interop/known-failures.txt)にまとめています。

## 質問・不具合

[GitHub の issue](https://github.com/rustorb/rustorb/issues) にお願いします。

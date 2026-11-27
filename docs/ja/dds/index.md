# rustorb-dds

**Rust でコアを書いた DDS（Data Distribution Service）の実装です。Rust、Python、C++、Java、C#、C、Ruby から使えます。**

rustorb-dds は rustorb プロジェクトの DDS 側です。UDPv4 の上で DDSI-RTPS 2.5 を話し、Cyclone DDS、Fast DDS、OpenDDS、ROS 2、OpenRTM-aist とのあいだで、両方向の送受信を CI でテストしています。OMG の仕様（DDS 1.4、DDSI-RTPS 2.5、DDS-XTypes 1.3、DDS Security 1.2、DDS-RPC 1.0）と、ほかの実装との黒箱の相互接続試験だけをもとに書いており、ほかの実装のソースコードは読んでいません。

ソースは [rustorb/rustorb-dds](https://github.com/rustorb/rustorb-dds) にあります。CDR の符号化（`rustorb-cdr`）、型サポートのトレイト（`rustorb-typesupport`）、IDL コンパイラ（`rustorb-idl`）を rustorb の ORB と共有していますが、ORB 自体は必要ありません。

<div class="grid cards" markdown>

-   :material-download: **[インストール](install.md)**

    Rust、Python、.NET、Ruby のパッケージ、C のライブラリ、C++ と Java のバインディング。

-   :material-rocket-launch: **[はじめに](getting-started.md)**

    1 つの IDL の型で、各言語の publisher と subscriber を動かします。

-   :material-swap-horizontal: **[相互接続と規格](interop.md)**

    Cyclone DDS、Fast DDS、OpenDDS、ROS 2、OpenRTM-aist と確かめていることと、OMG 仕様への対応状況。

-   :material-speedometer: **[性能](performance.md)**

    Cyclone DDS、Fast DDS、OpenDDS、RustDDS と並べた、レイテンシ、スループット、discovery の時間。

</div>

## できること

- **通信プロトコル**: UDPv4 上の RTPS 2.5。SPDP/SEDP による discovery、best-effort と reliable の writer・reader、大きなサンプル（DATA_FRAG）、key 付きのインスタンス、同じホストの rustorb-dds の参加者どうしの共有メモリ転送（Rust ではゼロコピーの貸し出しも）
- **DCPS（DDS 1.4）**: 参加者、トピック、content filtered topic、パーティション付きの publisher と subscriber、writer と reader、listener、WaitSet と条件。QoS は RELIABILITY、HISTORY、RESOURCE_LIMITS、DURABILITY（VOLATILE から PERSISTENT まで）、DURABILITY_SERVICE、DEADLINE、LIVELINESS、USER_DATA、PROPERTY など
- **型（DDS-XTypes 1.3）**: XCDR1 と XCDR2、final・appendable・mutable の型、`@key`、`@optional`、TypeObject と TypeInformation、assignability、TypeLookup サービス、DynamicData、ネットワークから型を知ったトピックの読み書き（Rust と Python）
- **セキュリティ（DDS Security 1.2）**: 組み込みプラグインの PKI-DH、Permissions、AES-GCM-GMAC と、Logging、Data Tagging、PSK。Cargo の feature `security` で有効になります
- **リクエストとリプライ（DDS-RPC 1.0）**: 全言語の Requester と Replier、ROS 2 のサービス
- **ROS 2 と OpenRTM-aist**: ROS 2 ノードとの `std_msgs` のトピックと `example_interfaces` のサービス、OpenRTM-aist 2.0 のデータポート
- **CORBA との橋渡し**: `rustorb-dds-corba` が CORBA の TypeCode と any を XTypes の型と DynamicData に変換します（逆も）

規格ごとの詳しい表は [docs/compliance.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/compliance.md) にあります。概要は[相互接続と規格](interop.md)にまとめました。

## 言語

| 言語 | API | パッケージ | IDL からの型 |
| --- | --- | --- | --- |
| Rust 1.89+ | rustorb-dds 独自（DDS 1.4 にならう） | crates.io の `rustorb-dds` | `build.rs` で `rustorb-codegen`（`.dds_only()`） |
| Python 3.9+ | Python らしい DCPS（`rustorb_dds`） | `pip install rustorb-dds` | `rustorb-idl -bpython -Wbdds_only` か、`@dds.struct` のクラス |
| C++17 | ISO/IEC C++ PSM for DDS（`<dds/dds.hpp>`）、ヘッダだけ | リポジトリからビルド（`cpp/`） | `rustorb-idl -bc++11 -Wbdds_only` |
| Java 22+ | Java 5 PSM for DDS（`org.omg.dds`）、FFM API 経由 | リポジトリからビルド（`java/`） | `rustorb-idl -bjava -Wbdds_only` |
| C#（.NET 8+） | Java 5 PSM の構造を C# に移したもの（`Omg.Dds`） | NuGet の `Rustorb.Dds` | `rustorb-idl -bcsharp -Wbdds_only` |
| C11 | ほかのバインディングの土台の C ABI（`rustorb/dds.h`） | GitHub Release の `rustorb-dds-native` | `rdds_*_builder` で型を組み立てる |
| Ruby 3.0+ | C++ PSM の構造を Ruby に移したもの（`DDS::`） | `gem install rustorb-dds` | `rustorb-idl -bruby -Wbdds_only` |

どのバインディングも型を同じように記述するので、TypeObject は同じ IDL からほかの DDS 実装が作るものと一致します。

現在のバージョンは **0.3.0** です（[リリース](https://github.com/rustorb/rustorb-dds/releases/tag/v0.3.0)、[変更履歴](https://github.com/rustorb/rustorb-dds/blob/main/CHANGELOG.md)）。1.0 までは、マイナーバージョンが上がると API が変わることがあります。

## まだないもの

QoS の OWNERSHIP、DESTINATION_ORDER、LIFESPAN、TIME_BASED_FILTER、PRESENTATION、READER_DATA_LIFECYCLE、作成後の QoS の変更、組み込みトピックの reader、QueryCondition と MultiTopic、UDPv6、ユニキャストのピアだけでの discovery。ほかは [docs/compliance.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/compliance.md#未対応のおもな項目) の最後にあります。

## ライセンス

rustorb-dds は Apache License 2.0 と MIT ライセンスのどちらかを選んで使えます。クレートが rustorb から依存するのは `rustorb-cdr` と `rustorb-typesupport` だけです（`rustorb-dds-corba` だけは `rustorb-core` にも依存します）。IDL コンパイラ `rustorb-idl` は GPL-3.0-or-later ですが、生成したコードにはこのライセンスはかかりません。

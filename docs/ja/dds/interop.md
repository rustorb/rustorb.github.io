# 相互接続と規格

## ほかの DDS 実装

rustorb-dds は、ほかの実装を黒箱として、配布されたパッケージと公開 API、同じ IDL からその実装自身の生成器が作ったコードだけを使って試験しています。CI の `interop`、`opendds`、`ros2`、`openrtm` の各ジョブが、変更のたびに両方向で走ります。

| 相手 | 版 | 確かめていること |
| --- | --- | --- |
| Cyclone DDS | 0.10.4 | best-effort、reliable（失われた datagram の修復も）、TRANSIENT_LOCAL、TRANSIENT・PERSISTENT（rustorb-dds から）、LIVELINESS 3 種、大きなサンプル（DATA_FRAG）、key 付きインスタンスの dispose と unregister、content filtered topic、XCDR2、TypeLookup による型の進化、DynamicData、DDS Security（RTPS の保護も） |
| Fast DDS | 2.11.2 | 同上。ただし TypeLookup と型の進化はなし（Fast DDS 2.11 に TypeLookup がない）。TRANSIENT は TRANSIENT_LOCAL の reader で |
| OpenDDS | 3.31 | best-effort、reliable、TRANSIENT_LOCAL、DATA_FRAG、key 付きインスタンス、XCDR2、TypeLookup による型の進化、再帰型 |
| ROS 2 | Jazzy（`rmw_fastrtps_cpp`、`rmw_cyclonedds_cpp`） | demo の talker・listener との `std_msgs/msg/String`、サービス `add_two_ints` |
| OpenRTM-aist | 2.0.2（FastRTPSTransport） | `RTC::TimedLong` のデータポート（OutPort と InPort）、Rust・C++・Python から |
| RustDDS | 0.15.1 | [性能測定](performance.md)での、あらゆる大きさの reliable と best-effort のサンプル |

各バインディング（Rust、Python、C++、Java、.NET、Ruby）の例も、Cyclone DDS、Fast DDS、OpenDDS と試験しています。試験していないもの: RTI Connext DDS、新しい版の Cyclone DDS（0.11 以降）と Fast DDS（2.12 以降、3.x）、OpenDDS との DDS Security。

試験の動かし方と、相手ごとに分かったことは [interop/README.md](https://github.com/rustorb/rustorb-dds/blob/main/interop/README.md) にあります。

### ROS 2

ROS 2 のノードは DDS の参加者なので、rustorb-dds のプログラムは ROS 2 をインストールしなくても、そのトピックに直接 publish・subscribe し、サービスを呼べます。トピック名と型名は ROS 2 の決まり（`rt/chatter`、`std_msgs::msg::dds_::String_`）に従います。例の [`ros2`](https://github.com/rustorb/rustorb-dds/tree/main/crates/rustorb-dds/examples/ros2) は `ros2 run demo_nodes_cpp talker` と `listener` と、[`ros2_service`](https://github.com/rustorb/rustorb-dds/tree/main/crates/rustorb-dds/examples/ros2_service) は `add_two_ints_server` と `add_two_ints_client` とやりとりします。

### OpenRTM-aist

OpenRTM-aist 2.0 のデータポートを Fast DDS の通信路でつなぐと、同じ型と QoS の rustorb-dds の writer・reader とやりとりできます。設定は [docs/openrtm.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/openrtm.md) にあります。

## OMG の仕様

| 仕様 | 版 | 状況 | 場所 |
| --- | --- | --- | --- |
| DDS（DCPS） | 1.4 | 主要な部分 | `rustorb-dds` |
| DDSI-RTPS | 2.5 | UDPv4 の範囲で実装済み | `rustorb-rtps` |
| DDS-XTypes | 1.3 | 主要な部分 | `rustorb-xtypes`、`rustorb-cdr` |
| DDS Security | 1.2 | 組み込みプラグイン 3 つ、Logging、Data Tagging、PSK | `rustorb-dds-security` |
| DDS-RPC | 1.0 | request/reply 形式は全言語、関数呼び出し形式は Rust | `rustorb_dds::rpc` |
| ISO/IEC C++ PSM for DDS | 1.0 | 部分集合 | `cpp/` |
| Java 5 PSM for DDS | 1.0 | 部分集合 | `java/` |
| IDL | 4.2 | DDS の型と XTypes の annotation | rustorb-idl |

できていることの主なもの:

- **DCPS**: すべての entity とその操作、content filtered topic、すべての通信状態、上位の entity への伝搬を含む listener、WaitSet と StatusCondition・GuardCondition・ReadCondition。QoS は USER_DATA、PARTITION、RELIABILITY、HISTORY、RESOURCE_LIMITS、DURABILITY（4 種すべて）、DURABILITY_SERVICE、DEADLINE、LIVELINESS、WRITER_DATA_LIFECYCLE、PROPERTY、DATA_REPRESENTATION、TYPE_CONSISTENCY_ENFORCEMENT。
- **RTPS**: すべての submessage、SPDP と SEDP、writer の liveliness プロトコル、断片を含む reliable の writer・reader、独自の共有メモリ転送。
- **XTypes**: 型システムのすべて（bitset と annotation 型は一部）、XCDR1 と XCDR2、MINIMAL と COMPLETE の TypeObject、再帰型（強連結成分）、assignability、TypeLookup、Rust と Python の DynamicData。
- **Security**: PKI-DH（EC P-256、RSA 2048）、署名付きの governance・permissions による Permissions、メッセージ・submessage・payload の AES-GCM-GMAC、secure discovery、CRL と OCSP。

まだないもの: OWNERSHIP、DESTINATION_ORDER、LIFESPAN、TIME_BASED_FILTER、PRESENTATION と coherent access、READER_DATA_LIFECYCLE、作成後の QoS の変更、組み込みトピックの reader、`ignore_*`、QueryCondition、MultiTopic、UDPv6、ユニキャストだけの discovery、XML の型表現とデータ表現、C++ と Java の DynamicData、PKCS #11。

項目ごとの詳しい表は [docs/compliance.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/compliance.md) にあります。

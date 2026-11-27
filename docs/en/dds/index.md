# rustorb-dds

**A DDS (Data Distribution Service) implementation with its core in Rust,
and bindings for Rust, Python, C++, Java, C#, C and Ruby.**

rustorb-dds is the DDS side of the rustorb project. It speaks DDSI-RTPS 2.5
over UDPv4, and is tested in CI against Cyclone DDS, Fast DDS, OpenDDS,
ROS 2 and OpenRTM-aist, in both directions. It is written from the OMG
specifications (DDS 1.4, DDSI-RTPS 2.5, DDS-XTypes 1.3, DDS Security 1.2,
DDS-RPC 1.0) and black-box tests with other implementations, without
reading their source code.

The source is in [rustorb/rustorb-dds](https://github.com/rustorb/rustorb-dds).
It shares the CDR encoder (`rustorb-cdr`), the type support traits
(`rustorb-typesupport`) and the IDL compiler (`rustorb-idl`) with the
rustorb ORB, but does not need the ORB.

<div class="grid cards" markdown>

-   :material-download: **[Install](install.md)**

    The packages for Rust, Python, .NET and Ruby, the C library, and the
    C++ and Java bindings.

-   :material-rocket-launch: **[Getting started](getting-started.md)**

    One IDL type, a publisher and a subscriber, in each language.

-   :material-swap-horizontal: **[Interoperability](interop.md)**

    What is tested with Cyclone DDS, Fast DDS, OpenDDS, ROS 2 and
    OpenRTM-aist, and which parts of the OMG specifications are done.

-   :material-speedometer: **[Performance](performance.md)**

    Latency, throughput and discovery time next to Cyclone DDS, Fast DDS,
    OpenDDS and RustDDS.

</div>

## What is in it

- **Wire protocol**: RTPS 2.5 over UDPv4 with SPDP/SEDP discovery,
  best-effort and reliable writers and readers, large samples
  (DATA_FRAG), keyed instances, and a shared memory transport between
  rustorb-dds participants on one host (with zero-copy loans in Rust)
- **DCPS (DDS 1.4)**: participants, topics, content filtered topics,
  publishers and subscribers with partitions, writers and readers,
  listeners, WaitSets and conditions, and the QoS policies RELIABILITY,
  HISTORY, RESOURCE_LIMITS, DURABILITY (VOLATILE to PERSISTENT),
  DURABILITY_SERVICE, DEADLINE, LIVELINESS, USER_DATA, PROPERTY and more
- **Types (DDS-XTypes 1.3)**: XCDR1 and XCDR2, final, appendable and
  mutable types, `@key`, `@optional`, TypeObject and TypeInformation,
  assignability, the TypeLookup service, DynamicData, and reading a topic
  whose type you only learn from the network (Rust and Python)
- **Security (DDS Security 1.2)**: the builtin plugins PKI-DH,
  Permissions and AES-GCM-GMAC, logging, data tagging and PSK, under the
  Cargo feature `security`
- **Requests and replies (DDS-RPC 1.0)**: Requester and Replier in every
  language, and ROS 2 services
- **ROS 2 and OpenRTM-aist**: `std_msgs` topics and `example_interfaces`
  services with ROS 2 nodes, and data ports of OpenRTM-aist 2.0
- **CORBA bridge**: `rustorb-dds-corba` turns CORBA TypeCodes and anys
  into XTypes types and DynamicData, and back

The full table, standard by standard, is
[docs/compliance.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/compliance.md)
(in Japanese); [Interoperability](interop.md) has a summary.

## Languages

| Language | API | Package | Types from IDL |
| --- | --- | --- | --- |
| Rust 1.89+ | rustorb-dds's own, after DDS 1.4 | `rustorb-dds` on crates.io | `rustorb-codegen` (`.dds_only()`) in `build.rs` |
| Python 3.9+ | Pythonic DCPS (`rustorb_dds`) | `pip install rustorb-dds` | `rustorb-idl -bpython -Wbdds_only`, or `@dds.struct` classes |
| C++17 | ISO/IEC C++ PSM for DDS (`<dds/dds.hpp>`), header only | build from the repository (`cpp/`) | `rustorb-idl -bc++11 -Wbdds_only` |
| Java 22+ | Java 5 PSM for DDS (`org.omg.dds`), through the FFM API | build from the repository (`java/`) | `rustorb-idl -bjava -Wbdds_only` |
| C# (.NET 8+) | the Java 5 PSM's structure in C# (`Omg.Dds`) | NuGet `Rustorb.Dds` | `rustorb-idl -bcsharp -Wbdds_only` |
| C11 | the C ABI the other bindings are built on (`rustorb/dds.h`) | `rustorb-dds-native` from the GitHub release | types built with `rdds_*_builder` |
| Ruby 3.0+ | the C++ PSM's structure in Ruby (`DDS::`) | `gem install rustorb-dds` | `rustorb-idl -bruby -Wbdds_only` |

Every binding describes its types the same way, so the TypeObjects match
those other DDS implementations make from the same IDL.

The current version is **0.3.0**
([release](https://github.com/rustorb/rustorb-dds/releases/tag/v0.3.0),
[changelog](https://github.com/rustorb/rustorb-dds/blob/main/CHANGELOG.md)).
Before 1.0, a new minor version may change the API.

## Not done yet

The QoS policies OWNERSHIP, DESTINATION_ORDER, LIFESPAN,
TIME_BASED_FILTER, PRESENTATION and READER_DATA_LIFECYCLE; changing QoS
after creation; builtin topic readers; QueryCondition and MultiTopic;
UDPv6; discovery with unicast peers only. See the end of
[docs/compliance.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/compliance.md#未対応のおもな項目)
for the rest.

## License

rustorb-dds is licensed under either of the Apache License, Version 2.0 or
the MIT license, at your option. Its crates depend only on `rustorb-cdr`
and `rustorb-typesupport` from rustorb (and `rustorb-dds-corba` on
`rustorb-core`). The IDL compiler `rustorb-idl` is GPL-3.0-or-later; the
code it generates is not covered by that license.

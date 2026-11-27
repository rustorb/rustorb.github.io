# Interoperability and standards

## Other DDS implementations

rustorb-dds is tested with other implementations as black boxes: their
released packages and public APIs, and the code their own generators make
from the same IDL. The `interop`, `opendds`, `ros2` and `openrtm` jobs of
CI run these tests on every change, in both directions.

| Peer | Version | What is tested |
| --- | --- | --- |
| Cyclone DDS | 0.10.4 | best-effort, reliable (with lost datagrams repaired), TRANSIENT_LOCAL, TRANSIENT and PERSISTENT (from rustorb-dds), the three LIVELINESS kinds, large samples (DATA_FRAG), keyed instances with dispose and unregister, content filtered topics, XCDR2, type evolution with TypeLookup, DynamicData, DDS Security (with RTPS protection) |
| Fast DDS | 2.11.2 | the same, but no TypeLookup or type evolution (Fast DDS 2.11 has no TypeLookup); TRANSIENT with a TRANSIENT_LOCAL reader |
| OpenDDS | 3.31 | best-effort, reliable, TRANSIENT_LOCAL, DATA_FRAG, keyed instances, XCDR2, type evolution with TypeLookup, recursive types |
| ROS 2 | Jazzy (`rmw_fastrtps_cpp`, `rmw_cyclonedds_cpp`) | `std_msgs/msg/String` with the demo talker and listener, and the `add_two_ints` service |
| OpenRTM-aist | 2.0.2 (FastRTPSTransport) | `RTC::TimedLong` data ports (OutPort and InPort), from Rust, C++ and Python |
| RustDDS | 0.15.1 | reliable and best-effort samples of every size, in the [benchmarks](performance.md) |

The examples of each binding (Rust, Python, C++, Java, .NET, Ruby) are run
against Cyclone DDS, Fast DDS and OpenDDS. Not tested: RTI Connext DDS,
newer Cyclone DDS (0.11+) and Fast DDS (2.12+, 3.x), and DDS Security
with OpenDDS.

How to run the tests, and what was learned about each peer, is in
[interop/README.md](https://github.com/rustorb/rustorb-dds/blob/main/interop/README.md).

### ROS 2

ROS 2 nodes are DDS participants, so a rustorb-dds program can publish
and subscribe to their topics and call their services directly, with no
ROS 2 installation. The topic names and types follow the ROS 2
conventions (`rt/chatter`, `std_msgs::msg::dds_::String_`). The
[`ros2`](https://github.com/rustorb/rustorb-dds/tree/main/crates/rustorb-dds/examples/ros2)
example talks with `ros2 run demo_nodes_cpp talker` and `listener`, and
[`ros2_service`](https://github.com/rustorb/rustorb-dds/tree/main/crates/rustorb-dds/examples/ros2_service)
with `add_two_ints_server` and `add_two_ints_client`.

### OpenRTM-aist

OpenRTM-aist 2.0's data ports, connected with its Fast DDS transport, talk
with rustorb-dds writers and readers of the same type and QoS. See
[docs/openrtm.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/openrtm.md)
(in Japanese) for the settings.

## OMG specifications

| Specification | Version | Status | Where |
| --- | --- | --- | --- |
| DDS (DCPS) | 1.4 | most of it | `rustorb-dds` |
| DDSI-RTPS | 2.5 | done for UDPv4 | `rustorb-rtps` |
| DDS-XTypes | 1.3 | most of it | `rustorb-xtypes`, `rustorb-cdr` |
| DDS Security | 1.2 | the three builtin plugins, logging, data tagging, PSK | `rustorb-dds-security` |
| DDS-RPC | 1.0 | request/reply in every language; function-call style in Rust | `rustorb_dds::rpc` |
| ISO/IEC C++ PSM for DDS | 1.0 | a subset | `cpp/` |
| Java 5 PSM for DDS | 1.0 | a subset | `java/` |
| IDL | 4.2 | the DDS types and XTypes annotations | rustorb-idl |

Highlights of what is done:

- **DCPS**: all entities and their operations, content filtered topics,
  every communication status, listeners with propagation to the parent
  entities, WaitSets and StatusCondition, GuardCondition, ReadCondition.
  QoS: USER_DATA, PARTITION, RELIABILITY, HISTORY, RESOURCE_LIMITS,
  DURABILITY (all four kinds), DURABILITY_SERVICE, DEADLINE, LIVELINESS,
  WRITER_DATA_LIFECYCLE, PROPERTY, DATA_REPRESENTATION,
  TYPE_CONSISTENCY_ENFORCEMENT.
- **RTPS**: every submessage, SPDP and SEDP, the writer liveliness
  protocol, reliable writers and readers with fragments, and a shared
  memory transport of its own.
- **XTypes**: the whole type system (bitsets and annotation types in
  part), XCDR1 and XCDR2, MINIMAL and COMPLETE TypeObjects, recursive
  types (strongly connected components), assignability, TypeLookup, and
  DynamicData in Rust and Python.
- **Security**: PKI-DH (EC P-256 and RSA 2048), Permissions with signed
  governance and permissions, AES-GCM-GMAC for messages, submessages and
  payloads, secure discovery, CRL and OCSP.

Not done yet: OWNERSHIP, DESTINATION_ORDER, LIFESPAN, TIME_BASED_FILTER,
PRESENTATION and coherent access, READER_DATA_LIFECYCLE, changing QoS
after creation, builtin topic readers, `ignore_*`, QueryCondition,
MultiTopic, UDPv6, unicast-only discovery, XML type and data
representations, DynamicData in C++ and Java, PKCS #11.

The full tables, item by item, are in
[docs/compliance.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/compliance.md)
(in Japanese).

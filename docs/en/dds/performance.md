# Performance

rustorb-dds is measured next to Cyclone DDS and Fast DDS with the same
program in each implementation (same type, QoS and way of waiting for
samples), and next to OpenDDS and RustDDS. The method, every table and the
open issues are in
[docs/benchmark.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/benchmark.md)
(in Japanese), and the tools in
[bench/](https://github.com/rustorb/rustorb-dds/tree/main/bench).

!!! warning "Relative numbers"
    These numbers come from a shared 4-CPU container (Intel Xeon 2.1 GHz,
    Linux 6.18) where neither the CPU frequency nor interrupts can be
    pinned, so they move from run to run. Compare them within one run;
    do not set them next to numbers from another machine. Measuring on a
    dedicated machine is planned.

## Round trip

Reliable, UDP, the median round trip of a ping and a pong in two
processes (KEEP_LAST 1), in µs, from the 2026-11-26 run:

| Size | rustorb-dds | Cyclone DDS 0.10.4 | Fast DDS 2.11.2 |
| ---: | ---: | ---: | ---: |
| 32 B | 79 | 66 | 89 |
| 16 KiB | 143 | 85 | |
| 256 KiB | 800 | 460 | |

Small samples are close to Cyclone DDS, and faster than Fast DDS. Large
samples still take longer than with Cyclone DDS; this is being worked on.
Answering in a listener, as Cyclone DDS's `ddsperf` does, takes 32 B down
to 51 µs. With the shared memory transport and zero-copy loans (Rust), a
1 MiB sample makes the round trip in about 0.1 ms, a thirtieth of UDP.

## Throughput

Reliable, UDP, payload received in 5 s:

- 256 B: rustorb-dds 256 Mbit/s, Cyclone DDS 256 Mbit/s, Fast DDS
  149 Mbit/s.
- 1 KiB: rustorb-dds 918 Mbit/s, Cyclone DDS 870 Mbit/s.
- 16 KiB to 1 MiB: rustorb-dds 8.7 to 10 Gbit/s, Cyclone DDS 5.3 to
  8.7 Gbit/s.

## Discovery

N processes start together, each with one participant, writer and reader
on the same topic; the time until the last one has matched all the
others, in ms (median of 3 runs):

| Participants | rustorb-dds | Cyclone DDS | Fast DDS |
| ---: | ---: | ---: | ---: |
| 2 | 6 | 9 | 81 |
| 10 | 23 | 42 | 203 |
| 50 | 501 | 747 | 3,792 |

The largest resident set of one process was 5 to 6 MiB for rustorb-dds,
7 to 9 MiB for Cyclone DDS and 15 to 17 MiB for Fast DDS.

## DDS Security

Median round trip in µs with data and metadata signed (AES-GMAC) or
encrypted (AES-GCM), each implementation with its own security plugins:

| Size | rustorb-dds sign | rustorb-dds encrypt | Cyclone DDS sign | Cyclone DDS encrypt | Fast DDS sign | Fast DDS encrypt |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 32 B | 90 | 92 | 100 | 92 | 210 | 201 |
| 4 KiB | 100 | 112 | 106 | 103 | 207 | 208 |
| 64 KiB | 360 | 551 | 452 | 450 | 490 | 542 |

With authentication, 10 participants all matched in about 90 ms
(Cyclone DDS 530 to 710 ms).

## Language bindings

The round trip from a ping in each binding to a Rust pong (shared memory,
reliable), in µs:

| Size | Rust | C++ | Java | .NET | Python | Ruby |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 32 B | 85 | 96 | 100 | 92 | 120 | 103 |
| 4 KiB | 94 | 98 | 103 | 92 | 118 | 193 |
| 64 KiB | 195 | 215 | 251 | 248 | 270 | 1,245 |

The bindings add 10 to 15 µs (C++, Java, .NET) or 20 to 35 µs (Python,
Ruby) to a small sample.

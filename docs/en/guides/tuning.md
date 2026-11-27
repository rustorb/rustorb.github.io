# Speed and tuning

rustorb 0.8 is much faster than 0.7 for small calls, many threads, calls in
one process and large data. Most of it needs nothing from you. This page
lists what is automatic, the options that change it, and the two choices
that change the language mapping. The measurements are in
[omniorb-comparison.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-comparison.md)
and the plan behind them in
[perf-plan.md](https://github.com/rustorb/rustorb/blob/main/docs/perf-plan.md).

## What is automatic

- **Fewer thread switches per call.** A blocking caller that read its own
  reply keeps reading the connection for its next call, and an upcall
  thread that answered a request reads the next request of the same
  connection itself (Linux and macOS; Windows keeps the I/O task reading).
- **Several connections per server.** Calls made at the same time from
  several threads to one server use up to 4 connections instead of one.
- **Calls in the same process skip GIOP.** The arguments are marshalled
  once, the servant runs on the calling thread and the reply comes back as
  its body. A POA manager that is not active, a `SINGLE_THREAD_MODEL` or
  `MAIN_THREAD_MODEL` POA and server interceptors take the general path.
- **Large messages wake the reader less** (Linux, TCP without TLS).
- **C and Ruby** copy, free and check sequences of numbers at once.

## Options

| Option | Default | Meaning |
| --- | --- | --- |
| `-ORBmaxConnectionsPerServer N` | 4 | the most connections a client opens to one server endpoint (also omniORB's `-ORBmaxGIOPConnectionPerServer`). `1` puts every call on one connection, as before 0.8 |
| `-ORBfollowLinger US` | 1000 | how long (µs) a thread that finished a call or a request keeps reading the connection. `0` lets the I/O task read everything, as before 0.8 |
| `-ORBfollowLimit N` | 64 | how many requests of one connection an upcall thread reads in a row |
| `-ORBmaxServerThreadPoolSize N` | 100 | the most upcall threads running at once |
| `-ORBstatistics 1` | off | counts what the ORB does inside (wake-ups, reads, connections), read with `rustorb_core::stats` in Rust |

## Choices that change the mapping

These are off by default, because code written for the standard mapping
would see different types.

### Rust: shared octet sequences

`rustorb-idl -brust -Wbshared_octets` maps an unbounded `sequence<octet>`
to `rustorb_core::Octets` instead of `Vec<u8>` (`&Octets` for `in`). One
of 4 KiB or more is received without being copied out of the message and
sent without being copied into it. The bytes on the wire are the same, so
the other side may use `Vec<u8>`.

### Ruby: octet sequences as Strings

```ruby
Rustorb.octets_as_string = true
data = target.get_octets(1 << 20)   # a binary String, not an Array of Integer
```

Sequences and arrays of `octet` are then read as binary Strings
(`Encoding::BINARY`), which for 1 MiB is more than ten times faster than an
Array. A String was already accepted when sending. The setting is for the
whole process and also applies to values in an `any`.

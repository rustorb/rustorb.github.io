# Tutorial

This tutorial builds one small system in each language: a **server** that
implements an IDL interface and registers it in the naming service under
the name `Echo`, and a **client** that looks `Echo` up and calls it. The
four servers and four clients are interchangeable: every client works with
every server, and with omniORB, omniORBpy and JacORB ones.

The same programs are in the repository under
[examples/](https://github.com/rustorb/rustorb/tree/main/examples), where
CI runs every server with every client.

## 1. Install the tools

You need the IDL compiler and a naming service (see [Install](../install/index.md)):

```sh
pip install rustorb-idl
cargo install rustorb-naming
```

## 2. Start the naming service

In a terminal of its own:

```sh
rustorb-names -memory -start 2809
```

The root naming context is now `corbaloc::localhost:2809/NameService`.
Every program in this tutorial finds it with the standard ORB option

```sh
-ORBInitRef NameService=corbaloc::localhost:2809/NameService
```

which every binding takes from its command line. (`-memory` forgets the
names when it exits; see [Naming service](../reference/naming.md) to keep
them.)

## 3. Write the IDL

Every language uses the same `Echo.idl`:

```idl title="Echo.idl"
module Example {
  interface Echo {
    string echo(in string message);
    oneway void shutdown();
  };
};
```

`echo` returns its argument; `shutdown` asks the server to stop. It is
`oneway`, so the client does not wait for a reply.

## 4. Pick a language

<div class="grid cards" markdown>

-   **[Rust](rust.md)**: crates.io, `build.rs` generates the code
-   **[Python](python.md)**: `pip install rustorb`, the omniORBpy layout
-   **[C++](cpp.md)**: the IDL to C++11 mapping, CMake
-   **[Java](java.md)**: the IDL to Java mapping, Java 22+

</div>

Then [run them together](together.md).

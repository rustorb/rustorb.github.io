# Install for C

The C binding is the OMG C Language Mapping 1.0 on the rustorb library:
`librustorb_c`, the `CORBA_*`, `PortableServer_*` and `CosNaming_*` APIs in
C99. It comes as an archive per platform on the
[release](https://github.com/rustorb/rustorb/releases/tag/v0.7.0), with the
headers and two static libraries, `librustorb_c.a` and `librustorb_ffi.a`.
It needs a C99 compiler.

You also need [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0
or later to generate code from IDL.

## Download

```sh
curl -LO https://github.com/rustorb/rustorb/releases/download/v0.7.0/rustorb-c-0.7.0-linux-x86_64.tar.gz
tar xzf rustorb-c-0.7.0-linux-x86_64.tar.gz
PREFIX=$PWD/rustorb-c-0.7.0-linux-x86_64
```

| Platform | Archive |
| --- | --- |
| Linux x86_64 | `rustorb-c-0.7.0-linux-x86_64.tar.gz` |
| Linux aarch64 | `rustorb-c-0.7.0-linux-aarch64.tar.gz` |
| macOS (Apple silicon) | `rustorb-c-0.7.0-macos-aarch64.tar.gz` |

Each has `include/rustorb/` (`orb.h`, `poa.h`, `CosNaming.h`, `dii.h`,
`dynany.h`, `messaging.h`, `pi.h` and the C ABI's `rustorb.h`) and
`lib/` with the two libraries.

## Generate code and build

```sh
rustorb-idl -bc -C gen Echo.idl         # gen/Echo.h and gen/Echo.c
cc -std=c99 -I$PREFIX/include -Igen server.c gen/Echo.c \
   $PREFIX/lib/librustorb_c.a $PREFIX/lib/librustorb_ffi.a -lpthread -ldl -lm -o server
```

On macOS, link `-framework CoreFoundation -framework Security -lpthread -lm`
instead. Memory follows the mapping: free what an operation returns with
`CORBA_free`, and release references with `CORBA_Object_release`.

## From source

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cmake -S c -B build/c -DRUSTORB_BUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=Release   # runs cargo too
cmake --build build/c
cmake --install build/c --prefix $HOME/rustorb-c    # include/ and lib/, as in the archive
```

The mapping is described in
[c-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/c-binding.md).

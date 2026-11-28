# Install for C

The C binding is the OMG C Language Mapping 1.0 on the rustorb library:
`librustorb_c`, the `CORBA_*`, `PortableServer_*` and `CosNaming_*` APIs in
C99. It comes as an archive per platform on the
[release](https://github.com/rustorb/rustorb/releases/tag/v0.8.0), with the
headers and two static libraries, `librustorb_c.a` and `librustorb_ffi.a`
(`rustorb_c.lib` and `rustorb_ffi.lib` on Windows). It needs a C99
compiler; on Windows, MSVC.

You also need [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0
or later to generate code from IDL.

## Download

```sh
curl -LO https://github.com/rustorb/rustorb/releases/download/v0.8.0/rustorb-c-0.8.0-linux-x86_64.tar.gz
tar xzf rustorb-c-0.8.0-linux-x86_64.tar.gz
PREFIX=$PWD/rustorb-c-0.8.0-linux-x86_64
```

| Platform | Archive |
| --- | --- |
| Linux x86_64 | `rustorb-c-0.8.0-linux-x86_64.tar.gz` |
| Linux aarch64 | `rustorb-c-0.8.0-linux-aarch64.tar.gz` |
| macOS (Apple silicon) | `rustorb-c-0.8.0-macos-aarch64.tar.gz` |
| Windows x64 (MSVC) | `rustorb-c-0.8.0-windows-x86_64.zip` (since 0.8.0) |

Each has `include/rustorb/` (`orb.h`, `poa.h`, `CosNaming.h`, `dii.h`,
`dynany.h`, `messaging.h`, `pi.h` and the C ABI's `rustorb.h`) and
`lib/` with the two libraries.
The headers are the same on every platform.

## Generate code and build

```sh
rustorb-idl -bc -C gen Echo.idl         # gen/Echo.h and gen/Echo.c
cc -std=c99 -I$PREFIX/include -Igen server.c gen/Echo.c \
   $PREFIX/lib/librustorb_c.a $PREFIX/lib/librustorb_ffi.a -lpthread -ldl -lm -o server
```

On macOS, link `-framework CoreFoundation -framework Security -lpthread -lm`
instead.

On Windows, build with MSVC and link the two `.lib` files with the system
libraries that `rustorb_ffi.lib` needs, for example with CMake:

```cmake
cmake_minimum_required(VERSION 3.18)
project(echo LANGUAGES C)
set(RUSTORB_C_PREFIX "" CACHE PATH "the unpacked rustorb-c archive")
add_executable(server server.c gen/Echo.c)
target_include_directories(server PRIVATE "${RUSTORB_C_PREFIX}/include" gen)
target_link_libraries(server PRIVATE
  "${RUSTORB_C_PREFIX}/lib/rustorb_c.lib" "${RUSTORB_C_PREFIX}/lib/rustorb_ffi.lib"
  ws2_32 userenv ntdll bcrypt)
```

```powershell
cmake -S . -B build -DRUSTORB_C_PREFIX="$PWD\rustorb-c-0.8.0-windows-x86_64"
cmake --build build --config Release        # build\Release\server.exe
```

Memory follows the mapping: free what an operation returns with
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

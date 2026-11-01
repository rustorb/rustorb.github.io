# Install for C++

The C++ binding is the IDL to C++11 mapping, compiled as **C++17**, on a
library written in Rust (`librustorb_ffi`). It is distributed as the
`rustorb-native` archive of each GitHub release, which holds the library,
the headers and a CMake package.

You need a C++17 compiler (GCC 9+, Clang 10+ or MSVC 2019+), CMake 3.16 or
later, and [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl).

## Download

Pick the archive for your platform from the
[release](https://github.com/rustorb/rustorb/releases/tag/v0.1.1) and unpack
it where you like, for example `$HOME/rustorb`:

=== "Linux x86_64"

    ```sh
    curl -LO https://github.com/rustorb/rustorb/releases/download/v0.1.1/rustorb-native-0.1.1-linux-x86_64.tar.gz
    tar xzf rustorb-native-0.1.1-linux-x86_64.tar.gz
    mv rustorb-native-0.1.1-linux-x86_64 $HOME/rustorb
    ```

=== "Linux aarch64"

    ```sh
    curl -LO https://github.com/rustorb/rustorb/releases/download/v0.1.1/rustorb-native-0.1.1-linux-aarch64.tar.gz
    tar xzf rustorb-native-0.1.1-linux-aarch64.tar.gz
    mv rustorb-native-0.1.1-linux-aarch64 $HOME/rustorb
    ```

=== "macOS (Apple silicon)"

    ```sh
    curl -LO https://github.com/rustorb/rustorb/releases/download/v0.1.1/rustorb-native-0.1.1-macos-aarch64.tar.gz
    tar xzf rustorb-native-0.1.1-macos-aarch64.tar.gz
    mv rustorb-native-0.1.1-macos-aarch64 $HOME/rustorb
    ```

=== "Windows x64"

    Download `rustorb-native-0.1.1-windows-x86_64.zip` from the release
    and extract it, for example to `C:\rustorb`.

The archive contains:

```text
include/rustorb/...          the C++ headers
lib/librustorb_ffi.a         the static library (rustorb_ffi.lib on Windows)
lib/librustorb_ffi.so        the shared library (.dylib on macOS, bin/rustorb_ffi.dll on Windows)
lib/cmake/rustorb/           the CMake package
```

## Use it from CMake

Point CMake at the directory with `-DCMAKE_PREFIX_PATH=$HOME/rustorb`, and:

```cmake
find_package(rustorb 0.1 REQUIRED)

# runs rustorb-idl -bc++11; the output goes to ${CMAKE_CURRENT_BINARY_DIR}/idl
rustorb_idl_generate(SOURCES Echo.idl)

add_executable(app main.cc ${SOURCES})
target_include_directories(app PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/idl")
target_link_libraries(app PRIVATE rustorb::rustorb)
```

`rustorb::rustorb` links the static library and the system libraries it
needs. The package finds `rustorb-idl` on `PATH` (or in the `RUSTORB_IDL`
variable), and sets `RUSTORB_IDL_DIR` to the directory of the standard IDL
files, so the naming service is
`rustorb_idl_generate(SOURCES Echo.idl "${RUSTORB_IDL_DIR}/CosNaming.idl")`.

## Without CMake

```sh
rustorb-idl -bc++11 -C gen Echo.idl
c++ -std=c++17 -I$HOME/rustorb/include -Igen main.cc gen/Echo.cc \
    $HOME/rustorb/lib/librustorb_ffi.a -lpthread -ldl -lm -o app
```

On macOS link `-framework CoreFoundation -framework Security` instead of
`-ldl -lm`.

## From source

The same package can be built and installed from the repository (cargo
builds the library):

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cmake -S cpp -B build/cpp -DRUSTORB_BUILD_TESTS=OFF
cmake --build build/cpp
cmake --install build/cpp --prefix $HOME/rustorb
```

The mapping, and where it differs from the standard, is described in
[cpp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/cpp-binding.md).

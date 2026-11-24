# C へのインストール

C バインディングは OMG の C Language Mapping 1.0 で、rustorb のライブラリの上に C99 で書いた `librustorb_c` です。`CORBA_*`、`PortableServer_*`、`CosNaming_*` の API があります。[リリース](https://github.com/rustorb/rustorb/releases/tag/v0.7.1)にプラットフォームごとのアーカイブがあり、ヘッダと 2 つの静的ライブラリ `librustorb_c.a`・`librustorb_ffi.a`（Windows では `rustorb_c.lib`・`rustorb_ffi.lib`）が入っています。C99 のコンパイラが必要です（Windows では MSVC）。

IDL からコードを生成するために [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0 以降も使います。

## ダウンロード

```sh
curl -LO https://github.com/rustorb/rustorb/releases/download/v0.7.1/rustorb-c-0.7.1-linux-x86_64.tar.gz
tar xzf rustorb-c-0.7.1-linux-x86_64.tar.gz
PREFIX=$PWD/rustorb-c-0.7.1-linux-x86_64
```

| プラットフォーム | アーカイブ |
| --- | --- |
| Linux x86_64 | `rustorb-c-0.7.1-linux-x86_64.tar.gz` |
| Linux aarch64 | `rustorb-c-0.7.1-linux-aarch64.tar.gz` |
| macOS（Apple シリコン） | `rustorb-c-0.7.1-macos-aarch64.tar.gz` |
| Windows x64（MSVC） | `rustorb-c-0.7.1-windows-x86_64.zip`（0.7.1 から） |

どれにも `include/rustorb/`（`orb.h`、`poa.h`、`CosNaming.h`、`dii.h`、`dynany.h`、`messaging.h`、`pi.h` と C ABI の `rustorb.h`）と、2 つのライブラリの入った `lib/` があります。ヘッダはどのプラットフォームでも同じです。

## コードを生成してビルドする

```sh
rustorb-idl -bc -C gen Echo.idl         # gen/Echo.h と gen/Echo.c
cc -std=c99 -I$PREFIX/include -Igen server.c gen/Echo.c \
   $PREFIX/lib/librustorb_c.a $PREFIX/lib/librustorb_ffi.a -lpthread -ldl -lm -o server
```

macOS では代わりに `-framework CoreFoundation -framework Security -lpthread -lm` をリンクします。

Windows では MSVC でビルドし、2 つの `.lib` と、`rustorb_ffi.lib` が使うシステムのライブラリをリンクします。CMake なら次のとおりです。

```cmake
cmake_minimum_required(VERSION 3.18)
project(echo LANGUAGES C)
set(RUSTORB_C_PREFIX "" CACHE PATH "展開した rustorb-c のアーカイブ")
add_executable(server server.c gen/Echo.c)
target_include_directories(server PRIVATE "${RUSTORB_C_PREFIX}/include" gen)
target_link_libraries(server PRIVATE
  "${RUSTORB_C_PREFIX}/lib/rustorb_c.lib" "${RUSTORB_C_PREFIX}/lib/rustorb_ffi.lib"
  ws2_32 userenv ntdll bcrypt)
```

```powershell
cmake -S . -B build -DRUSTORB_C_PREFIX="$PWD\rustorb-c-0.7.1-windows-x86_64"
cmake --build build --config Release        # build\Release\server.exe
```

メモリはマッピングのとおりに扱います。操作が返したものは `CORBA_free` で解放し、参照は `CORBA_Object_release` で手放します。

## ソースから

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cmake -S c -B build/c -DRUSTORB_BUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=Release   # cargo も動かす
cmake --build build/c
cmake --install build/c --prefix $HOME/rustorb-c    # アーカイブと同じ include/ と lib/
```

マッピングは [c-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/c-binding.md) にあります。

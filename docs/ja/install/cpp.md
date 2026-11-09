# C++ へのインストール

C++ バインディングは IDL to C++11 マッピングで、**C++17** としてコンパイルします。土台は Rust で書いたライブラリ（`librustorb_ffi`）です。各 GitHub Release の `rustorb-native` アーカイブとして配布しており、ライブラリ、ヘッダ、CMake パッケージが入っています。

C++17 コンパイラ（GCC 9 以降、Clang 10 以降、MSVC 2019 以降）、CMake 3.16 以降、[`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) が必要です。

## ダウンロード

[リリース](https://github.com/rustorb/rustorb/releases/tag/v0.4.0)からプラットフォームに合うアーカイブを選び、好きな場所に展開します。ここでは `$HOME/rustorb` にします。

=== "Linux x86_64"

    ```sh
    curl -LO https://github.com/rustorb/rustorb/releases/download/v0.4.0/rustorb-native-0.4.0-linux-x86_64.tar.gz
    tar xzf rustorb-native-0.4.0-linux-x86_64.tar.gz
    mv rustorb-native-0.4.0-linux-x86_64 $HOME/rustorb
    ```

=== "Linux aarch64"

    ```sh
    curl -LO https://github.com/rustorb/rustorb/releases/download/v0.4.0/rustorb-native-0.4.0-linux-aarch64.tar.gz
    tar xzf rustorb-native-0.4.0-linux-aarch64.tar.gz
    mv rustorb-native-0.4.0-linux-aarch64 $HOME/rustorb
    ```

=== "macOS（Apple シリコン）"

    ```sh
    curl -LO https://github.com/rustorb/rustorb/releases/download/v0.4.0/rustorb-native-0.4.0-macos-aarch64.tar.gz
    tar xzf rustorb-native-0.4.0-macos-aarch64.tar.gz
    mv rustorb-native-0.4.0-macos-aarch64 $HOME/rustorb
    ```

=== "Windows x64"

    リリースから `rustorb-native-0.4.0-windows-x86_64.zip` をダウンロードし、`C:\rustorb` などに展開します。

アーカイブの中身:

```text
include/rustorb/...          C++ のヘッダ
lib/librustorb_ffi.a         静的ライブラリ（Windows では rustorb_ffi.lib）
lib/librustorb_ffi.so        共有ライブラリ（macOS は .dylib、Windows は bin/rustorb_ffi.dll）
lib/cmake/rustorb/           CMake パッケージ
```

## CMake から使う

`-DCMAKE_PREFIX_PATH=$HOME/rustorb` でディレクトリを CMake に教え、次のように書きます。

```cmake
find_package(rustorb 0.4 REQUIRED)

# rustorb-idl -bc++11 を実行し、${CMAKE_CURRENT_BINARY_DIR}/idl に出力する
rustorb_idl_generate(SOURCES Echo.idl)

add_executable(app main.cc ${SOURCES})
target_include_directories(app PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/idl")
target_link_libraries(app PRIVATE rustorb::rustorb)
```

`rustorb::rustorb` は静的ライブラリと、それが必要とするシステムライブラリをリンクします。パッケージは `rustorb-idl` を `PATH`（または変数 `RUSTORB_IDL`）から探し、標準 IDL ファイルのディレクトリを `RUSTORB_IDL_DIR` に設定します。ネーミングサービスを使う場合は `rustorb_idl_generate(SOURCES Echo.idl "${RUSTORB_IDL_DIR}/CosNaming.idl")` とします。

## CMake を使わない場合

```sh
rustorb-idl -bc++11 -C gen Echo.idl
c++ -std=c++17 -I$HOME/rustorb/include -Igen main.cc gen/Echo.cc \
    $HOME/rustorb/lib/librustorb_ffi.a -lpthread -ldl -lm -o app
```

macOS では `-ldl -lm` の代わりに `-framework CoreFoundation -framework Security` をリンクします。

## ソースから

同じパッケージをリポジトリからビルドしてインストールすることもできます（ライブラリは cargo がビルドします）。

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cmake -S cpp -B build/cpp -DRUSTORB_BUILD_TESTS=OFF
cmake --build build/cpp
cmake --install build/cpp --prefix $HOME/rustorb
```

マッピングと、標準との違いは[cpp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/cpp-binding.md) にあります。

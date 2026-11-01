# チュートリアル: C++

始める前に、`rustorb-native` アーカイブを展開し（ここでは `$HOME/rustorb`。[C++ へのインストール](../install/cpp.md)を参照）、`rustorb-idl` を `PATH` に置き、ネーミングサービスを起動し、`Echo.idl` を書いておきます（[チュートリアル](index.md)の手順 1〜3）。

## プロジェクト

```text
echo-cpp/
├── CMakeLists.txt
├── Echo.idl
├── server.cc
└── client.cc
```

```cmake title="CMakeLists.txt"
cmake_minimum_required(VERSION 3.16)
project(echo_cpp CXX)

find_package(rustorb 0.1 REQUIRED)

# Echo.idl, and CosNaming.idl from rustorb-idl's standard IDL files
rustorb_idl_generate(SOURCES Echo.idl "${RUSTORB_IDL_DIR}/CosNaming.idl")
add_library(echo_idl STATIC ${SOURCES})
target_include_directories(echo_idl PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/idl")
target_link_libraries(echo_idl PUBLIC rustorb::rustorb)

add_executable(server server.cc)
target_link_libraries(server PRIVATE echo_idl)
add_executable(client client.cc)
target_link_libraries(client PRIVATE echo_idl)
```

`rustorb_idl_generate` はビルド時に `rustorb-idl -bc++11` を実行し、`build/idl` に `Echo.hh`、`Echo.cc`、`CosNaming.hh`、`CosNaming.cc` を出力します。マッピングは IDL to C++11 で、参照は `IDL::traits<T>::ref_type`（参照カウント）、文字列は `std::string`、シーケンスは `std::vector` です。

## サーバ

```cpp title="server.cc"
#include <iostream>
#include <string>

#include "CosNaming.hh"
#include "Echo.hh"

class EchoImpl : public CORBA::servant_traits<Example::Echo>::base_type {
 public:
  explicit EchoImpl(IDL::traits<CORBA::ORB>::ref_type orb) : orb_(std::move(orb)) {}

  std::string echo(const std::string& message) override {
    std::cout << "server: " << message << std::endl;
    return message;
  }

  void shutdown() override { orb_->shutdown(false); }

 private:
  IDL::traits<CORBA::ORB>::ref_type orb_;
};

int main(int argc, char* argv[]) {
  try {
    auto orb = CORBA::ORB_init(argc, argv);
    auto poa = IDL::traits<PortableServer::POA>::narrow(
        orb->resolve_initial_references("RootPOA"));
    poa->the_POAManager()->activate();
    auto echo = CORBA::make_reference<EchoImpl>(orb)->_this();

    auto root = IDL::traits<CosNaming::NamingContext>::narrow(
        orb->resolve_initial_references("NameService"));
    root->rebind(CosNaming::Name{CosNaming::NameComponent("Echo", "")}, echo);
    std::cout << "server: ready" << std::endl;

    orb->run();
    orb->destroy();
  } catch (const CORBA::Exception& e) {
    std::cerr << "server: " << e << std::endl;
    return 1;
  }
  return 0;
}
```

## クライアント

```cpp title="client.cc"
#include <iostream>
#include <string>

#include "CosNaming.hh"
#include "Echo.hh"

int main(int argc, char* argv[]) {
  try {
    auto orb = CORBA::ORB_init(argc, argv);
    bool shutdown = false;
    for (int i = 1; i < argc; ++i) {
      if (std::string(argv[i]) == "--shutdown") shutdown = true;
    }

    auto root = IDL::traits<CosNaming::NamingContextExt>::narrow(
        orb->resolve_initial_references("NameService"));
    auto echo = IDL::traits<Example::Echo>::narrow(root->resolve_str("Echo"));

    std::cout << "client: " << echo->echo("hello from C++") << std::endl;
    if (shutdown) echo->shutdown();
    orb->destroy();
  } catch (const CORBA::Exception& e) {
    std::cerr << "client: " << e << std::endl;
    return 1;
  }
  return 0;
}
```

## ビルドして動かす

```sh
cmake -S . -B build -DCMAKE_PREFIX_PATH=$HOME/rustorb
cmake --build build
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
./build/server $NS &
./build/client $NS --shutdown
```

```text
server: ready
server: hello from C++
client: hello from C++
```

## 次に

- 他の言語のクライアントやサーバと[つないで動かす](together.md)。
- バインディングの全体は [cpp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/cpp-binding.md) にあります。

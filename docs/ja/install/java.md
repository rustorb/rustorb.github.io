# Java へのインストール

Java バインディングは IDL to Java マッピング（`org.omg.CORBA`、`org.omg.PortableServer`、`org.omg.CosNaming`）で、rustorb のライブラリを Foreign Function & Memory API で呼び出します。**Java 22 以降**と、[リリース](https://github.com/rustorb/rustorb/releases/tag/v0.3.0)の次の 2 つが必要です。

- `rustorb-0.3.0.jar`: Java のクラス（依存なし）
- プラットフォームに合う `rustorb-native` アーカイブに入っている共有ライブラリ `rustorb_ffi`

IDL からコードを生成するために [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) も使います。

## ダウンロード

```sh
curl -LO https://github.com/rustorb/rustorb/releases/download/v0.3.0/rustorb-0.3.0.jar
curl -LO https://github.com/rustorb/rustorb/releases/download/v0.3.0/rustorb-native-0.3.0-linux-x86_64.tar.gz
tar xzf rustorb-native-0.3.0-linux-x86_64.tar.gz
```

| プラットフォーム | アーカイブ | 中の共有ライブラリ |
| --- | --- | --- |
| Linux x86_64 | `rustorb-native-0.3.0-linux-x86_64.tar.gz` | `lib/librustorb_ffi.so` |
| Linux aarch64 | `rustorb-native-0.3.0-linux-aarch64.tar.gz` | `lib/librustorb_ffi.so` |
| macOS（Apple シリコン） | `rustorb-native-0.3.0-macos-aarch64.tar.gz` | `lib/librustorb_ffi.dylib` |
| Windows x64 | `rustorb-native-0.3.0-windows-x86_64.zip` | `bin/rustorb_ffi.dll` |

jar はまだ Maven Central にありません。Maven や Gradle から使う場合は、ローカルリポジトリにインストールします。

```sh
mvn install:install-file -Dfile=rustorb-0.3.0.jar \
    -DgroupId=org.rustorb -DartifactId=rustorb -Dversion=0.3.0 -Dpackaging=jar
```

## 実行する

JVM にネイティブコードの呼び出しを許可し、ライブラリを見つけられるようにします。ライブラリは次の順に探されます。

1. システムプロパティ `rustorb.ffi.library`（ファイルのパス）
2. 環境変数 `RUSTORB_FFI_LIBRARY`（同上）
3. `java.library.path`
4. システムの動的ローダのパス（`LD_LIBRARY_PATH` など）

```sh
java --enable-native-access=ALL-UNNAMED \
     -Drustorb.ffi.library=$PWD/rustorb-native-0.3.0-linux-x86_64/lib/librustorb_ffi.so \
     -cp rustorb-0.3.0.jar:classes Server
```

`org.omg.CORBA.ORBClass` を設定しなくても、`ORB.init(args, props)` は rustorb の ORB を返します。

## IDL からコードを生成する

```sh
rustorb-idl -bjava -C gen Echo.idl
javac --release 22 -cp rustorb-0.3.0.jar -d classes $(find gen -name '*.java') *.java
```

CosNaming のクラスは jar に入っています。他の IDL コンパイラ（JacORB など）で生成したスタブも、ポータブルスタブとして使えます。

## ソースから

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cargo build --release -p rustorb-ffi       # target/release/librustorb_ffi.so
mvn -f java/pom.xml package                # java/target/rustorb-0.3.0.jar
```

マッピングは [java-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/java-binding.md) にあります。

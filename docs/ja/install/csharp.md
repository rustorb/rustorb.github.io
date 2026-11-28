# .NET へのインストール

.NET バインディングは IDL4 to C# マッピング（OMG formal/21-07-01）で、rustorb のライブラリを P/Invoke で呼び出します。`Corba`、`PortableServer`、`CosNaming` などのモジュールがあります。**.NET 8 以降**が必要です。NuGet パッケージ [`Rustorb`](https://www.nuget.org/packages/Rustorb) で配布していて、Linux（x64、arm64）、macOS（Apple シリコン）、Windows（x64）のネイティブライブラリが入っています。

IDL からコードを生成するために [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0 以降も使います。

## パッケージを追加する

```sh
dotnet new console -o EchoServer && cd EchoServer
dotnet add package Rustorb --version 0.8.0
```

ネイティブライブラリはパッケージの `runtimes/<rid>/native` に入っているので、`dotnet run` や `dotnet publish -r <rid>` で見つかります。別にビルドしたものを使うときは、環境変数 `RUSTORB_FFI_LIBRARY` にファイルのパスを入れます。ライブラリは次の順に探されます。

1. 環境変数 `RUSTORB_FFI_LIBRARY`
2. アセンブリの隣（パッケージの `runtimes/<rid>/native`）
3. OS のローダが探す場所（`LD_LIBRARY_PATH`、`PATH` など）

## IDL からコードを生成する

```sh
rustorb-idl -bcsharp -C gen Echo.idl
```

IDL ファイルごとに `.cs` ファイルが 1 つできます。たとえば次のようにしてプロジェクトに加えます。

```xml
<ItemGroup>
  <Compile Include="gen/*.cs" />
</ItemGroup>
```

生成される名前は、マッピングのとおり IDL の綴りのままです（`echo`）。`-Wbdotnet_naming` を付けると .NET の命名規則（型とメンバーは PascalCase、パラメーターは camelCase）になります。CosNaming の型はパッケージに入っています。

## ソースから

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cargo build --release -p rustorb-ffi       # target/release/librustorb_ffi.so
dotnet build -c Release dotnet/src/Rustorb.Corba
```

プロジェクトから `dotnet/src/Rustorb.Corba/Rustorb.Corba.csproj` を参照し、`RUSTORB_FFI_LIBRARY` にライブラリのパスを入れます。マッピングは [csharp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/csharp-binding.md) にあります。

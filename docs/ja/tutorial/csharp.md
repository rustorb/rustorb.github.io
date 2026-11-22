# チュートリアル: C#

始める前に、.NET 8 SDK と `rustorb-idl` を `PATH` に置き、ネーミングサービスを起動し、`Echo.idl` を書いておきます（[チュートリアル](index.md)の手順 1〜3）。パッケージについては [.NET へのインストール](../install/csharp.md) を見てください。

## プロジェクトを作る

サーバとクライアントを、`Rustorb` パッケージを使うコンソールのプロジェクトとして作ります。

```sh
dotnet new console -o server && dotnet add server package Rustorb --version 0.7.0
dotnet new console -o client && dotnet add client package Rustorb --version 0.7.0
rustorb-idl -bcsharp -C gen Echo.idl
```

`gen/Echo.cs` には `Example::Echo` の IDL4 to C# マッピングが入っています。インタフェース `IEcho`、`EchoHelper`（`narrow` を持つ）、スタブ、そしてサーバが継承するスケルトン `EchoPOA` です。これを両方のプロジェクトに加えます。`server.csproj` と `client.csproj` に次を書きます。

```xml
<ItemGroup>
  <Compile Include="../gen/*.cs" />
</ItemGroup>
```

## サーバ

```csharp title="server/Program.cs"
using System;
using CosNaming;
using Example;

sealed class EchoImpl : EchoPOA
{
    private readonly Corba.ORB orb;

    public EchoImpl(Corba.ORB orb)
    {
        this.orb = orb;
    }

    public override string echo(string message)
    {
        Console.WriteLine("server: " + message);
        return message;
    }

    public override void shutdown() => orb.shutdown(false);
}

static class Server
{
    static void Main(string[] args)
    {
        Corba.ORB orb = Corba.ORB.init(args);
        var poa = PortableServer.POAHelper.narrow(orb.resolve_initial_references("RootPOA"));
        poa.the_POAManager.activate();
        IEcho echo = EchoHelper.narrow(poa.servant_to_reference(new EchoImpl(orb)));

        INamingContext root = NamingContextHelper.narrow(orb.resolve_initial_references("NameService"));
        root.rebind(new Omg.Types.Sequence<NameComponent>(new[] { new NameComponent("Echo", "") }), echo);
        Console.WriteLine("server: ready");

        orb.run();
        orb.destroy();
    }
}
```

## クライアント

```csharp title="client/Program.cs"
using System;
using System.Linq;
using CosNaming;
using Example;

static class Client
{
    static void Main(string[] args)
    {
        Corba.ORB orb = Corba.ORB.init(args, out string[] rest);
        bool shutdown = rest.Contains("--shutdown");

        INamingContextExt root = NamingContextExtHelper.narrow(orb.resolve_initial_references("NameService"));
        IEcho echo = EchoHelper.narrow(root.resolve_str("Echo"));

        Console.WriteLine("client: " + echo.echo("hello from C#"));
        if (shutdown)
        {
            echo.shutdown();
        }
        orb.destroy();
    }
}
```

`Corba.ORB.init(args, out rest)` は `-ORB` のオプションを取り、残りの引数を返します。

## 動かす

```sh
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
dotnet run --project server -- $NS &
dotnet run --project client -- $NS --shutdown
```

```text
server: ready
server: hello from C#
client: hello from C#
```

## 次に

- ほかの言語のクライアントやサーバと[つないで動かす](together.md)。
- バインディングの全体：[csharp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/csharp-binding.md)。

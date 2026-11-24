# Tutorial: C#

Before you start, have the .NET 8 SDK and `rustorb-idl` on `PATH`, the
naming service running and `Echo.idl` written ([Tutorial](index.md), steps
1–3). See [Install for .NET](../install/csharp.md) for the package.

## Make the projects

A server and a client, each a console project with the `Rustorb` package:

```sh
dotnet new console -o server && dotnet add server package Rustorb --version 0.7.1
dotnet new console -o client && dotnet add client package Rustorb --version 0.7.1
rustorb-idl -bcsharp -C gen Echo.idl
```

`gen/Echo.cs` has the IDL4 to C# mapping of `Example::Echo`: the interface
`IEcho`, `EchoHelper` (with `narrow`), the stub and the skeleton `EchoPOA`
that a server derives from. Add it to both projects, in `server.csproj` and
`client.csproj`:

```xml
<ItemGroup>
  <Compile Include="../gen/*.cs" />
</ItemGroup>
```

## The server

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

## The client

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

`Corba.ORB.init(args, out rest)` takes the `-ORB` options and gives back
the other arguments.

## Run

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

## Next

- [Run them together](together.md) with clients and servers in other languages.
- The binding in full: [csharp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/csharp-binding.md).

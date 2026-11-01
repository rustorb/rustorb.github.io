# Tutorial: Java

Before you start, have Java 22 or later, `rustorb-0.1.1.jar` and the
unpacked `rustorb-native` archive in the current directory
([Install for Java](../install/java.md)), `rustorb-idl` on `PATH`, the
naming service running and `Echo.idl` written ([Tutorial](index.md),
steps 1–3).

## Generate the code

```sh
rustorb-idl -bjava -C gen Echo.idl
```

This writes the standard IDL to Java mapping into `gen/Example/`: the
interface `Echo`, `EchoHelper` (with `narrow`), `EchoHolder`, the stub and
the skeleton `EchoPOA` that a server extends. The CosNaming classes are
already in the jar.

## The server

```java title="Server.java"
import Example.Echo;
import Example.EchoHelper;
import Example.EchoPOA;
import org.omg.CORBA.ORB;
import org.omg.CosNaming.NameComponent;
import org.omg.CosNaming.NamingContext;
import org.omg.CosNaming.NamingContextHelper;
import org.omg.PortableServer.POA;
import org.omg.PortableServer.POAHelper;

public final class Server {
    private Server() {}

    static final class EchoImpl extends EchoPOA {
        private final ORB orb;

        EchoImpl(ORB orb) {
            this.orb = orb;
        }

        @Override
        public String echo(String message) {
            System.out.println("server: " + message);
            return message;
        }

        @Override
        public void shutdown() {
            orb.shutdown(false);
        }
    }

    public static void main(String[] args) throws Exception {
        ORB orb = ORB.init(args, null);
        POA poa = POAHelper.narrow(orb.resolve_initial_references("RootPOA"));
        poa.the_POAManager().activate();
        Echo echo = EchoHelper.narrow(poa.servant_to_reference(new EchoImpl(orb)));

        NamingContext root = NamingContextHelper.narrow(orb.resolve_initial_references("NameService"));
        root.rebind(new NameComponent[] {new NameComponent("Echo", "")}, echo);
        System.out.println("server: ready");

        orb.run();
        orb.destroy();
    }
}
```

## The client

```java title="Client.java"
import Example.Echo;
import Example.EchoHelper;
import java.util.Arrays;
import org.omg.CORBA.ORB;
import org.omg.CosNaming.NamingContextExt;
import org.omg.CosNaming.NamingContextExtHelper;

public final class Client {
    private Client() {}

    public static void main(String[] args) throws Exception {
        ORB orb = ORB.init(args, null);
        boolean shutdown = Arrays.asList(args).contains("--shutdown");

        NamingContextExt root = NamingContextExtHelper.narrow(orb.resolve_initial_references("NameService"));
        Echo echo = EchoHelper.narrow(root.resolve_str("Echo"));

        System.out.println("client: " + echo.echo("hello from Java"));
        if (shutdown) {
            echo.shutdown();
        }
        orb.destroy();
    }
}
```

## Build and run

```sh
javac --release 22 -cp rustorb-0.1.1.jar -d classes Server.java Client.java $(find gen -name '*.java')

LIB=$PWD/rustorb-native-0.1.1-linux-x86_64/lib/librustorb_ffi.so
JAVA="java --enable-native-access=ALL-UNNAMED -Drustorb.ffi.library=$LIB -cp rustorb-0.1.1.jar:classes"
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
$JAVA Server $NS &
$JAVA Client $NS --shutdown
```

On Windows the class path separator is `;` and the library is
`bin\rustorb_ffi.dll`; on macOS it is `lib/librustorb_ffi.dylib`.

```text
server: ready
server: hello from Java
client: hello from Java
```

## Next

- [Run them together](together.md) with clients and servers in other languages.
- The binding in full: [java-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/java-binding.md).

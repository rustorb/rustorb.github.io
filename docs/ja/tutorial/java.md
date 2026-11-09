# チュートリアル: Java

始める前に、Java 22 以降を用意し、`rustorb-0.4.0.jar` と展開した `rustorb-native` アーカイブをカレントディレクトリに置きます（[Java へのインストール](../install/java.md)）。さらに `rustorb-idl` を `PATH` に置き、ネーミングサービスを起動し、`Echo.idl` を書いておきます（[チュートリアル](index.md)の手順 1〜3）。

## コードを生成する

```sh
rustorb-idl -bjava -C gen Echo.idl
```

標準の IDL to Java マッピングが `gen/Example/` に出力されます。インタフェース `Echo`、`EchoHelper`（`narrow` を持つ）、`EchoHolder`、スタブ、そしてサーバが継承するスケルトン `EchoPOA` です。CosNaming のクラスは jar に入っています。

## サーバ

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

## クライアント

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

## ビルドして動かす

```sh
javac --release 22 -cp rustorb-0.4.0.jar -d classes Server.java Client.java $(find gen -name '*.java')

LIB=$PWD/rustorb-native-0.4.0-linux-x86_64/lib/librustorb_ffi.so
JAVA="java --enable-native-access=ALL-UNNAMED -Drustorb.ffi.library=$LIB -cp rustorb-0.4.0.jar:classes"
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
$JAVA Server $NS &
$JAVA Client $NS --shutdown
```

Windows ではクラスパスの区切りが `;` で、ライブラリは `bin\rustorb_ffi.dll` です。macOS では `lib/librustorb_ffi.dylib` です。

```text
server: ready
server: hello from Java
client: hello from Java
```

## 次に

- 他の言語のクライアントやサーバと[つないで動かす](together.md)。
- バインディングの全体は [java-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/java-binding.md) にあります。

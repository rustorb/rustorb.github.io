# チュートリアル: Python

始める前に、`rustorb` と `rustorb-idl` を入れ（`pip install rustorb rustorb-idl`）、ネーミングサービスを起動し、`Echo.idl` を書いておきます（[チュートリアル](index.md)の手順 1〜3）。

## コードを生成する

```sh
rustorb-idl -bpython -C gen Echo.idl "$(rustorb-idl --idl-dir)/CosNaming.idl"
export PYTHONPATH=gen
```

`Echo.idl` とネーミングサービスのスタブが生成されます。コードは標準の IDL to Python マッピングで、配置は `omniidl -bpython` と同じです。

- `Example`（IDL のモジュール）にスタブのクラス `Example.Echo` があります
- `Example__POA` に、サーバが継承するスケルトン `Example__POA.Echo` があります
- `CosNaming` にネーミングサービスの型があります

## サーバ

```python title="server.py"
import sys

from rustorb import CORBA, PortableServer
import CosNaming
import Example__POA


class EchoImpl(Example__POA.Echo):
    def __init__(self, orb):
        self.orb = orb

    def echo(self, message):
        print("server:", message, flush=True)
        return message

    def shutdown(self):
        self.orb.shutdown(False)


def main():
    orb = CORBA.ORB_init(sys.argv)
    poa = orb.resolve_initial_references("RootPOA")
    poa._get_the_POAManager().activate()
    echo = EchoImpl(orb)._this()

    root = orb.resolve_initial_references("NameService")._narrow(CosNaming.NamingContext)
    root.rebind([CosNaming.NameComponent("Echo", "")], echo)
    print("server: ready", flush=True)

    orb.run()
    orb.destroy()


if __name__ == "__main__":
    main()
```

`CORBA.ORB_init(sys.argv)` が `-ORB...` のオプションを読み取ります。`_this()` はサーバントをルート POA で活性化し、その参照を返します。ORB は自分のスレッドでサーバントを呼びます。

## クライアント

```python title="client.py"
import sys

from rustorb import CORBA
import CosNaming
import Example


def main():
    orb = CORBA.ORB_init(sys.argv)
    shutdown = "--shutdown" in sys.argv[1:]

    root = orb.resolve_initial_references("NameService")._narrow(CosNaming.NamingContextExt)
    echo = root.resolve_str("Echo")._narrow(Example.Echo)

    print("client:", echo.echo("hello from Python"))
    if shutdown:
        echo.shutdown()
    orb.destroy()


if __name__ == "__main__":
    main()
```

## 動かす

```sh
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
python server.py $NS &
python client.py $NS --shutdown
```

```text
server: ready
server: hello from Python
client: hello from Python
```

## 次に

- 他の言語のクライアントやサーバと[つないで動かす](together.md)。
- omniORBpy のアプリケーションを移行する: [Python へのインストール](../install/python.md#coming-from-omniorbpy)。
- バインディングの全体は [python-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/python-binding.md) にあります。

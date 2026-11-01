# Tutorial: Python

Before you start, have `rustorb` and `rustorb-idl` installed
(`pip install rustorb rustorb-idl`), the naming service running and
`Echo.idl` written ([Tutorial](index.md), steps 1–3).

## Generate the code

```sh
rustorb-idl -bpython -C gen Echo.idl "$(rustorb-idl --idl-dir)/CosNaming.idl"
export PYTHONPATH=gen
```

This generates the stubs of `Echo.idl` and of the naming service. The
code is the standard IDL to Python mapping, in the same layout as
`omniidl -bpython`:

- `Example` (the IDL module) holds the stub class `Example.Echo`;
- `Example__POA` holds the skeleton `Example__POA.Echo` that a server
  subclasses;
- `CosNaming` holds the naming service's types.

## The server

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

`CORBA.ORB_init(sys.argv)` reads the `-ORB...` options. `_this()` activates
the servant in the root POA and returns its reference. The ORB calls
servants on its own threads.

## The client

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

## Run

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

## Next

- [Run them together](together.md) with clients and servers in other languages.
- Moving an omniORBpy application: [Install for Python](../install/python.md#coming-from-omniorbpy).
- The binding in full: [python-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/python-binding.md).

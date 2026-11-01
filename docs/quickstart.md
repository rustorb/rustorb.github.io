# Quick start

This page runs a CORBA server and client written in Python. It needs
nothing but Python 3.9 or later, or Docker. The [tutorial](tutorial/index.md)
then does the same in every language, with a naming service.

## 1. Get a shell with rustorb

=== "pip"

    In a new directory and virtual environment:

    ```sh
    mkdir rustorb-quickstart && cd rustorb-quickstart
    python3 -m venv .venv
    . .venv/bin/activate
    pip install rustorb rustorb-idl
    ```

    On Windows, activate it with `.venv\Scripts\activate`.

=== "Docker"

    Nothing is installed on your machine. Start a Python container with a
    new directory of yours mounted as `/work`, so you can write the files
    below with your own editor (the container is removed when you leave
    it):

    ```sh
    mkdir rustorb-quickstart && cd rustorb-quickstart
    docker run --rm -it -v "$PWD":/work -w /work python:3.12-slim bash
    ```

    and inside it:

    ```sh
    pip install rustorb rustorb-idl
    ```

    Run the commands of the following steps in this shell.

`rustorb` is the ORB (wheels with the Rust core compiled in, for Linux,
macOS and Windows), and `rustorb-idl` is the IDL compiler.

## 2. Write the interface

Save this as `Echo.idl`:

```idl
module Example {
  interface Echo {
    string echo(in string message);
  };
};
```

and generate the Python code from it:

```sh
rustorb-idl -bpython Echo.idl
```

This writes `Echo_idl.py` and the packages `Example` (the client stubs)
and `Example__POA` (the server skeletons).

## 3. Write the server

`server.py` implements the interface, activates the object and writes its
reference (an IOR) to a file:

```python
import sys
from rustorb import CORBA, PortableServer
import Example, Example__POA

class EchoImpl(Example__POA.Echo):
    def echo(self, message):
        print("server got:", message, flush=True)
        return message

orb = CORBA.ORB_init(sys.argv)
poa = orb.resolve_initial_references("RootPOA")
poa._get_the_POAManager().activate()
ref = EchoImpl()._this()
with open("echo.ior", "w") as f:
    f.write(orb.object_to_string(ref))
print("server ready, IOR in echo.ior", flush=True)
orb.run()
```

## 4. Write the client

`client.py` reads the reference and calls the object:

```python
import sys
from rustorb import CORBA
import Example

orb = CORBA.ORB_init(sys.argv)
ior = open("echo.ior").read()
echo = orb.string_to_object(ior)._narrow(Example.Echo)
print(echo.echo("hello, rustorb"))
orb.destroy()
```

## 5. Run them

```sh
python server.py &
python client.py
```

```text
server ready, IOR in echo.ior
server got: hello, rustorb
hello, rustorb
```

Stop the server with `kill %1` (or `fg` and ++ctrl+c++).

The two processes talk GIOP over TCP, so the server works with any CORBA
client, and the client with any CORBA server: omniORB, omniORBpy, JacORB,
or rustorb in another language.

## Next

- [Install](install/index.md) rustorb for the language you use.
- Follow the [tutorial](tutorial/index.md): the same `Echo` in Rust,
  Python, C++ and Java, found by name through the naming service instead of
  an IOR file.
- Coming from omniORBpy? See [Python](install/python.md#coming-from-omniorbpy).

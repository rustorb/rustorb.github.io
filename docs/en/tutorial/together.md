# Run them together

All the servers register themselves as `Echo` in the same naming service,
and all the clients look `Echo` up there, so any client works with any
server. With the naming service from the [tutorial](index.md) running:

```sh
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"

python server.py $NS &                   # a Python server ...
./echo-rs/target/debug/client $NS        # ... called from Rust
./echo-cpp/build/client $NS --shutdown   # ... and from C++, which stops it

./echo-rs/target/debug/server $NS &      # a Rust server ...
python client.py $NS --shutdown          # ... called from Python
```

```text
client: hello from Rust
client: hello from C++
client: hello from Python
```

The wire protocol is plain GIOP 1.2 over TCP, so omniORB, omniORBpy and
JacORB programs join in the same way, and `omniNames` can take the place of
`rustorb-names`.

## All sixteen pairs

The repository has a script that builds the four examples and runs every
server with every client against a fresh `rustorb-names`:

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
examples/run-all.sh                      # needs cargo, cmake, a C++ compiler, Python 3.9+, Java 22+ and Maven
EXAMPLES="rust python" examples/run-all.sh
```

## Without a naming service

A reference can also be passed as a string. A server prints
`orb.object_to_string(ref)` (an `IOR:...`), and a client turns it back with
`orb.string_to_object(...)`, as in the [quick start](../quickstart.md).

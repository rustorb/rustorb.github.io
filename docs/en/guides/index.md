# Guides

The [tutorial](../tutorial/index.md) builds one server and one client. These
guides show the rest of what rustorb does, one feature at a time:

<div class="grid cards" markdown>

-   **[The POA](poa.md)**: policies, object ids, persistent references,
    POA managers and the `POAManagerFactory`
-   **[Any and valuetypes](any-valuetype.md)**: values of any type, and
    objects passed by value, shared or not
-   **[DII and DSI](dii-dsi.md)**: calling and serving operations without
    generated code
-   **[Callback AMI](ami.md)**: calls that return at once, with the reply
    delivered to a handler
-   **[Transports](transports.md)**: TLS, Unix domain sockets, callbacks
    over the client's connection, addresses behind NAT
-   **[Timeouts and SyncScope](timeouts.md)**: how long a call waits, and
    when a oneway call returns
-   **[Coming from omniORBpy](omniorbpy.md)**: running omniORBpy
    applications on rustorb

</div>

## The examples

The examples are in Python, because they are short and run in one file,
and each guide lists the names of the same API in Rust, C++ and Java. The
design documents linked from each guide (in Japanese) have the details.

They share one IDL file:

```idl title="Demo.idl"
module Demo {
  valuetype Point {
    public long x;
    public long y;
  };
  struct Pair {
    Point first;
    Point second;
  };
  exception Overflow { long limit; };
  interface Calc {
    long add(in long a, in long b) raises (Overflow);
    Pair echo_pair(in Pair p);
    any echo_any(in any a);
    oneway void post(in string message);
  };
};
```

and one servant, `calc_impl.py`:

```python title="calc_impl.py"
import Demo, Demo__POA


class CalcImpl(Demo__POA.Calc):
    def add(self, a, b):
        if a + b > 1000:
            raise Demo.Overflow(1000)
        return a + b

    def echo_pair(self, p):
        return p

    def echo_any(self, a):
        return a

    def post(self, message):
        print("posted:", message)
```

Generate the code (`-Wbami` adds what [callback AMI](ami.md) needs) and run
an example with the generated code on the path:

```sh
pip install rustorb rustorb-idl
rustorb-idl -bpython -Wbami -C gen Demo.idl
PYTHONPATH=gen:. python3 poa_example.py
```

Each example starts its server and its client in the same process; the
calls go through the ORB just as they would between two processes.

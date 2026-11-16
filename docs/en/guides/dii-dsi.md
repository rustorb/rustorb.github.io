# DII and DSI

The **Dynamic Invocation Interface** (DII) calls an operation that the
client has no generated code for: it names the operation and passes the
arguments as anys. The **Dynamic Skeleton Interface** (DSI) is the server
side: one servant receives every request as a `ServerRequest` and reads
the arguments itself. Bridges, gateways, scripting tools and test drivers
use them. A DII client and a DSI server talk to ordinary stubs and
servants of any ORB; the wire does not show the difference.

rustorb has both in Rust, Python, C++ and Java.

## Example

```python title="dii_example.py"
import sys

from rustorb import CORBA, PortableServer
import Demo
from calc_impl import CalcImpl

orb = CORBA.ORB_init(sys.argv)
poa = orb.resolve_initial_references("RootPOA")
poa._get_the_POAManager().activate()
calc = CalcImpl()._this()

# DII: call add without the stub.
req = calc._request("add")
req.add_in_arg(CORBA.Any(CORBA.TC_long, 20))
req.add_in_arg(CORBA.Any(CORBA.TC_long, 22))
req.set_return_type(CORBA.TC_long)
req.invoke()
print(req.return_value().value())       # 42

# Deferred: send now, collect the reply later.
req = calc._request("add")
req.add_in_arg(CORBA.Any(CORBA.TC_long, 1))
req.add_in_arg(CORBA.Any(CORBA.TC_long, 2))
req.set_return_type(CORBA.TC_long)
req.send_deferred()
# ... other work ...
req.get_response()
print(req.return_value().value())       # 3


# DSI: a servant that reads its requests itself.
class Adder(PortableServer.DynamicImplementation):
    def _primary_interface(self, oid, poa):
        return "IDL:Demo/Calc:1.0"

    def invoke(self, request):
        if request.operation() != "add":
            raise CORBA.BAD_OPERATION(0, CORBA.COMPLETED_NO)
        args = CORBA.NVList()
        args.add_value("a", CORBA.Any(CORBA.TC_long, None), CORBA.ARG_IN)
        args.add_value("b", CORBA.Any(CORBA.TC_long, None), CORBA.ARG_IN)
        request.arguments(args)            # reads the arguments into args
        a, b = (args.item(i).argument.value() for i in range(2))
        request.set_result(CORBA.Any(CORBA.TC_long, a + b))


adder = Adder()._this()._narrow(Demo.Calc)
print(adder.add(5, 6))                  # 11, through the ordinary stub

orb.destroy()
```

A user exception the operation can raise is declared with
`req.add_exception(Demo._tc_Overflow)` (C++ and Java:
`exceptions()->add(tc)`, `exceptions().add(tc)`). Python then raises it
as `Demo.Overflow`; C++ and Java give it as `CORBA::UnknownUserException`
with the exception in an any. An exception that is not declared arrives as
`UNKNOWN`. `orb.send_multiple_requests_deferred` and `get_next_response` handle many
requests at once.

## In other languages

| | Rust | C++ | Java |
| --- | --- | --- | --- |
| New request | `ObjectRef::create_request(op)` | `obj->_request(op)` | `obj._request(op)` |
| Arguments | `DiiRequest` builder | `add_in_arg()` returns an `Any&` | `add_in_arg()` returns an `Any` |
| Send | sync, async, oneway, deferred | `invoke`, `send_oneway`, `send_deferred` | `invoke`, `send_oneway`, `send_deferred` |
| Errors | `Result` | thrown; user exceptions as `CORBA::UnknownUserException` | put into `req.env()`, not thrown (as JacORB does) |
| DSI servant | implement `DynamicImplementation` | derive from `PortableServer::DynamicImplementation` | extend `org.omg.PortableServer.DynamicImplementation` |

In C++ and Java, the anys of a request hold valuetypes by reference, so
valuetypes shared among the arguments stay shared
([Any and valuetypes](any-valuetype.md#valuetypes-in-an-any)).

Not supported: contexts (`ctx`, `ctxlist`, `get_default_context`) raise
`NO_IMPLEMENT`, and `create_operation_list` needs an interface repository,
which rustorb does not have.

More: [dii-dsi.md](https://github.com/rustorb/rustorb/blob/main/docs/dii-dsi.md).

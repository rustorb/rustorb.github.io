# The POA

A server makes its objects reachable through a **POA** (Portable Object
Adapter). The RootPOA from the [tutorial](../tutorial/index.md) is enough
for objects that live as long as the process; a POA of your own chooses
how objects are named, how long their references stay valid, and how
requests reach the servants.

## Policies

`create_POA(name, manager, policies)` makes a child POA. Each policy has a
default, the value the RootPOA uses, except where noted:

| Policy | Values (default first) |
| --- | --- |
| Thread | `ORB_CTRL_MODEL`, `SINGLE_THREAD_MODEL`, `MAIN_THREAD_MODEL` |
| Lifespan | `TRANSIENT`, `PERSISTENT` |
| IdUniqueness | `UNIQUE_ID`, `MULTIPLE_ID` |
| IdAssignment | `SYSTEM_ID`, `USER_ID` |
| ImplicitActivation | `NO_IMPLICIT_ACTIVATION` (the RootPOA has `IMPLICIT_ACTIVATION`), `IMPLICIT_ACTIVATION` |
| ServantRetention | `RETAIN`, `NON_RETAIN` |
| RequestProcessing | `USE_ACTIVE_OBJECT_MAP_ONLY`, `USE_DEFAULT_SERVANT`, `USE_SERVANT_MANAGER` |

- `ORB_CTRL_MODEL` runs requests on a pool of threads, several at once;
  `SINGLE_THREAD_MODEL` runs one request of the POA at a time;
  `MAIN_THREAD_MODEL` runs them in the thread that calls `orb.run()` or
  `orb.perform_work()`.
- `PERSISTENT` references stay valid when the server restarts, if it comes
  back on the same port with the same POA names and object ids. Give such a
  server a fixed port (`-ORBendPoint giop:tcp::2810`).
- `USER_ID` lets the application choose the object ids
  (`activate_object_with_id`).
- Servant managers (`ServantActivator`, `ServantLocator`), default servants
  and adapter activators are all there.

Combinations that contradict each other (`NON_RETAIN` with
`USE_ACTIVE_OBJECT_MAP_ONLY`, `IMPLICIT_ACTIVATION` with `USER_ID`, ...)
raise `InvalidPolicy`.

## POA managers

A POA manager switches the request processing of its POAs: `holding`
(the initial state: requests wait), `active`, `discarding` (requests fail
with `TRANSIENT`) and `inactive`. Several POAs can share a manager.

The `POAManagerFactory` (CORBA 3) creates managers with a name of your
choice, and finds and lists them. Every manager has an id: the RootPOA's
is `RootPOAManager`, the one `create_POA` makes when it is given none is
`POAManager<n>`. A manager ends with the last of its POAs.

## Example

```python title="poa_example.py"
import sys

from rustorb import CORBA, PortableServer
import Demo
from calc_impl import CalcImpl

orb = CORBA.ORB_init(sys.argv)
root = orb.resolve_initial_references("RootPOA")

# A manager of its own, from the POAManagerFactory.
factory = root._get_the_POAManagerFactory()
manager = factory.create_POAManager("calc-manager", [])

# A POA with persistent references and ids chosen by the application.
policies = [
    root.create_lifespan_policy(PortableServer.PERSISTENT),
    root.create_id_assignment_policy(PortableServer.USER_ID),
]
poa = root.create_POA("calc", manager, policies)

poa.activate_object_with_id(b"calc-1", CalcImpl())
calc = poa.id_to_reference(b"calc-1")._narrow(Demo.Calc)
manager.activate()

print(manager.get_id())                         # calc-manager
print([m.get_id() for m in factory.list()])     # ['RootPOAManager', 'calc-manager']
print(calc.add(1, 2))                           # 3
try:
    calc.add(999, 2)
except Demo.Overflow as e:
    print("overflow at", e.limit)               # overflow at 1000

orb.destroy()
```

```sh
PYTHONPATH=gen:. python3 poa_example.py -ORBendPoint giop:tcp::2810
```

The reference of `calc-1` (`orb.object_to_string(calc)`) now works across
restarts of this program.

## In other languages

| | Rust | C++ | Java |
| --- | --- | --- | --- |
| Child POA | `Poa::create_poa(name, Some(&manager), &[Policy])` | `poa->create_POA(name, manager, policies)` | `poa.create_POA(name, manager, policies)` |
| Policies | `Policy::Lifespan(Lifespan::Persistent)` and so on | `poa->create_lifespan_policy(...)` and so on | `poa.create_lifespan_policy(...)` and so on |
| Factory | `Poa::the_poa_manager_factory()` | `poa->the_POAManagerFactory()` | `poa.the_POAManagerFactory()` |
| New manager | `PoaManagerFactory::create_poa_manager(id, policies)` | `create_POAManager(id, policies)` | `create_POAManager(id, policies)` |
| Find, list | `find(id)`, `list()` | `find(id)`, `list()` | `find(id)`, `list()` |
| Manager id | `PoaManager::get_id()` | `manager->get_id()` | `manager.get_id()` |

More: [poa-design.md](https://github.com/rustorb/rustorb/blob/main/docs/poa-design.md)
(threads, servant managers, how the POA compares with omniORB's).

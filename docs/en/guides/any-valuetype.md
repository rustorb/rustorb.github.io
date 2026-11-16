# Any and valuetypes

## Any

An `any` holds a value together with its **TypeCode**, the description of
its type. A receiver that does not know the type in advance can still look
at it, pass it on, or take it apart with DynAny.

```python title="any_example.py"
import sys

from rustorb import CORBA
import Demo
from calc_impl import CalcImpl

orb = CORBA.ORB_init(sys.argv)
poa = orb.resolve_initial_references("RootPOA")
poa._get_the_POAManager().activate()
calc = CalcImpl()._this()

# A basic type: the TypeCodes of the basic types are in CORBA.
a = calc.echo_any(CORBA.Any(CORBA.TC_long, 42))
print(a.typecode().kind() == CORBA.tk_long, a.value())    # True 42

# A type from IDL: the generated code has its TypeCode (_tc_<name>).
p = Demo.Point(1, 2)
a = calc.echo_any(CORBA.Any(Demo._tc_Point, p))
print(a.typecode().id(), a.value().x)                     # IDL:Demo/Point:1.0 1

orb.destroy()
```

A value of a type the receiver's generated code does not have still
arrives: rustorb builds its TypeCode and a class with the member names on
the fly. `orb.create_struct_tc` and the other `create_*_tc` operations build
TypeCodes by hand.

| | Rust | C++ | Java |
| --- | --- | --- | --- |
| Make | `Any::new(tc, Value::...)` | `a <<= value` | `Helper.insert(a, value)`, `a.insert_long(...)` |
| Read | `any.value()` | `a >>= value` (false if the type differs) | `Helper.extract(a)`, `a.extract_long()` |
| TypeCode | `TypeCode::struct_tc(...)`, generated code | `_tc_<name>`, `CORBA::_tc_long` | `Helper.type()`, `orb.get_primitive_tc(...)` |

## Valuetypes

A **valuetype** is an object passed by value: the receiver gets a copy of
its state, made by a value factory (the generated code registers one for a
valuetype with no operations). Unlike a struct, a valuetype can be null,
can inherit, and keeps sharing: one instance in two places of a request
arrives as one instance, and a graph with cycles arrives with its cycles.

```python
p = Demo.Point(1, 2)
pair = calc.echo_pair(Demo.Pair(p, p))
print(pair.first is pair.second)    # True
```

Custom marshalling, truncatable valuetypes, value boxes, `supports` and
abstract interfaces are supported in every language.

### Valuetypes in an any {#valuetypes-in-an-any}

In C++ and Java, an any holds a valuetype **by reference**, as the
IDL to C++11 and IDL to Java mappings say (since 0.6.0):

- One instance put into several anys stays one instance: the DII and the
  DSI keep valuetypes shared among the arguments and the result of a
  request, as the generated stubs and skeletons do.
- Extracting from the same any gives the same instance each time.
- An any keeps the state the instance had when it was inserted. Inserting
  the instance again replaces that state, in every any that holds it.

Python's anys hold Python objects, so they share without further rules.

More: [any-typecode.md](https://github.com/rustorb/rustorb/blob/main/docs/any-typecode.md),
[dynany.md](https://github.com/rustorb/rustorb/blob/main/docs/dynany.md),
[valuetype.md](https://github.com/rustorb/rustorb/blob/main/docs/valuetype.md).

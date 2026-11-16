# Coming from omniORBpy

rustorb's Python binding follows the standard IDL to Python mapping in the
layout of `omniidl -bpython`, so code written for omniORBpy mostly runs
as it is. The package `rustorb-omniorb-compat` adds the `omniORB` module
and the omniORBpy extensions that applications use, enough to run
OpenRTM-aist-Python and rtshell unchanged.

## Switching

```sh
pip install rustorb rustorb-omniorb-compat rustorb-idl
```

1. Regenerate the stubs with `rustorb-idl -bpython` instead of
   `omniidl -bpython`. The generated modules have the same names and the
   same API, but omniidl's output calls omniORBpy's internals and does not
   work with rustorb. `-Wbpackage=` works as with omniidl.
2. Keep the imports: `from omniORB import CORBA, PortableServer`,
   `import CosNaming` and the top-level `CORBA` and `PortableServer` modules
   now give rustorb's.
3. Keep the command lines: the `-ORB...` options have omniORB's names
   ([ORB options](../reference/orb-options.md)).

Without the compatibility package, write `from rustorb import CORBA,
PortableServer` instead.

## What the compatibility package has

| omniORBpy API | Notes |
| --- | --- |
| `omniORB.CORBA`, `omniORB.PortableServer`, `CORBA`, `PortableServer`, `CosNaming` | rustorb's modules |
| `omniORB.cdrMarshal`, `cdrUnmarshal` | same encoding as omniORB (strings in ISO 8859-1), so the bytes match |
| `omniORB.any.to_any`, `from_any` | same choice of TypeCodes as omniORBpy |
| `omniORB.findType`, `omniORB.tcInternal` | the type descriptors OpenRTM-aist reads |
| `resolve_initial_references("omniINSPOA")` | object ids as `corbaloc` keys (`corbaloc::host:2810/manager`) |
| `omniORB.setClientCallTimeout` and the other timeout functions | [Timeouts](timeouts.md) |
| `omniORB.URI`: `stringToName`, `nameToString`, `addrAndNameToURI` | omniORBpy 4.3.4's rules, measured |
| Minor codes such as `TRANSIENT_ConnectFailed` | set to the codes rustorb raises |

```python title="compat_example.py"
import sys

from omniORB import CORBA, URI
import omniORB

orb = CORBA.ORB_init(sys.argv, CORBA.ORB_ID)

name = URI.stringToName("apps/calc.service")
print([(c.id, c.kind) for c in name])     # [('apps', ''), ('calc', 'service')]
print(URI.nameToString(name))             # apps/calc.service
print(URI.addrAndNameToURI("localhost:2809", "apps/calc.service"))
                                          # corbaname:localhost:2809#apps/calc.service
omniORB.setClientCallTimeout(2000)        # 2 s for every call of this ORB

orb.destroy()
```

## Differences to know

- Minor codes of system exceptions are rustorb's, not omniORB's; the
  constants of the compatibility package follow rustorb.
- `omniORB.setClientThreadCallDeadline`, `installTimeoutExceptionHandler`
  and the `sslTP` / `httpTP` modules are not there. TLS is set with the ORB
  options instead ([Transports](transports.md#tls-ssliop)).
- omniNames works with rustorb, and `rustorb-names` (a replacement, with
  persistence) works with omniORBpy: see [Naming service](../reference/naming.md).

More: [omniorb-compat.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-compat.md),
[python-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/python-binding.md).

# ORB options

Every binding takes the same `-ORB...` options: from the command line
(`Orb::init(&mut args)` in Rust, `CORBA.ORB_init(sys.argv)` in Python,
`CORBA::ORB_init(argc, argv)` in C++, `ORB.init(args, props)` in Java,
where they can also be properties). The ORB removes the ones it reads.

| Option | Meaning |
| --- | --- |
| `-ORBInitRef NAME=URL` | an initial reference (`corbaloc:`, `corbaname:` or an IOR), e.g. `NameService=corbaloc::localhost:2809/NameService` |
| `-ORBDefaultInitRef URL` | where other initial references are looked up |
| `-ORBendPoint giop:tcp:HOST:PORT` | where the server listens (default: any port on all addresses) |
| `-ORBendPointPublish giop:tcp:HOST:PORT` | the address written into IORs (behind NAT, in containers) |
| `-ORBmaxGIOPVersion 1.1` | the highest GIOP version to send |
| `-ORBclientCallTimeOutPeriod MS` | a timeout for every call ([timeouts.md](https://github.com/rustorb/rustorb/blob/main/docs/timeouts.md)) |
| `-ORBendPoint giop:ssl:HOST:PORT`, `-ORBsslCAFile FILE`, `-ORBsslKeyFile FILE` | SSLIOP, IIOP over TLS ([ssliop.md](https://github.com/rustorb/rustorb/blob/main/docs/ssliop.md)) |

The option names are those of omniORB, so existing command lines and
scripts keep working.

## URLs

| Form | Example |
| --- | --- |
| stringified IOR | `IOR:010000001...` |
| `corbaloc:` | `corbaloc::host:2809/NameService`, `corbaloc:iiop:1.2@host:2809/key` |
| `corbaname:` | `corbaname::host:2809#path/to/Echo` (looks the name up in that naming service) |

`orb.string_to_object()` accepts all three.

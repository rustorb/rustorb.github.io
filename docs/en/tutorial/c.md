# Tutorial: C

Before you start, have a C99 compiler, the C archive unpacked with
`PREFIX` set to its directory ([Install for C](../install/c.md)),
`rustorb-idl` on `PATH`, the naming service running and `Echo.idl` written
([Tutorial](index.md), steps 1–3).

## Generate the code

```sh
rustorb-idl -bc -C gen Echo.idl
```

`gen/Echo.h` and `gen/Echo.c` have the C mapping of `Example::Echo`: the
object type `Example_Echo`, the client functions `Example_Echo_echo` and
`Example_Echo_shutdown`, and for a server the entry point vectors
`POA_Example_Echo__epv` and `__vepv` and `POA_Example_Echo__init`.

## The server

A servant is a `POA_Example_Echo` whose `vepv` points at the functions that
implement the operations.

```c title="server.c"
#include <stdio.h>
#include <stdlib.h>

#include <rustorb/CosNaming.h>

#include "Echo.h"

static CORBA_ORB orb;

static void check(CORBA_Environment* ev, const char* what) {
  if (ev->_major == CORBA_NO_EXCEPTION) return;
  fprintf(stderr, "server: %s: %s\n", what, CORBA_exception_id(ev));
  exit(1);
}

static CORBA_char* echo(PortableServer_Servant self, const CORBA_char* message,
                        CORBA_Environment* ev) {
  (void)self;
  (void)ev;
  printf("server: %s\n", message);
  fflush(stdout);
  return CORBA_string_dup(message);
}

static void shutdown_orb(PortableServer_Servant self, CORBA_Environment* ev) {
  (void)self;
  CORBA_ORB_shutdown(orb, CORBA_FALSE, ev);
}

static PortableServer_ServantBase__epv base_epv = {NULL, NULL, NULL};
static POA_Example_Echo__epv echo_epv = {NULL, echo, shutdown_orb};
static POA_Example_Echo__vepv echo_vepv = {&base_epv, &echo_epv};
static POA_Example_Echo servant = {NULL, &echo_vepv};

int main(int argc, char** argv) {
  CORBA_Environment ev = {CORBA_NO_EXCEPTION, NULL, NULL};
  PortableServer_POA poa;
  PortableServer_POAManager mgr;
  CosNaming_NamingContext root;
  CosNaming_NameComponent component;
  CosNaming_Name name;
  CORBA_Object echo_ref;

  orb = CORBA_ORB_init(&argc, argv, "", &ev);
  check(&ev, "ORB_init");
  poa = (PortableServer_POA)CORBA_ORB_resolve_initial_references(orb, "RootPOA", &ev);
  check(&ev, "RootPOA");
  mgr = PortableServer_POA__get_the_POAManager(poa, &ev);
  PortableServer_POAManager_activate(mgr, &ev);
  check(&ev, "activate");
  POA_Example_Echo__init(&servant, &ev);
  echo_ref = PortableServer_POA_servant_to_reference(poa, &servant, &ev);
  check(&ev, "servant_to_reference");

  root = (CosNaming_NamingContext)CORBA_ORB_resolve_initial_references(orb, "NameService", &ev);
  check(&ev, "NameService");
  component.id = "Echo";
  component.kind = "";
  name._maximum = name._length = 1;
  name._buffer = &component;
  name._release = CORBA_FALSE;
  CosNaming_NamingContext_rebind(root, &name, echo_ref, &ev);
  check(&ev, "rebind");
  printf("server: ready\n");
  fflush(stdout);

  CORBA_ORB_run(orb, &ev);
  check(&ev, "run");
  CORBA_Object_release(echo_ref, &ev);
  CORBA_Object_release(root, &ev);
  CORBA_Object_release(mgr, &ev);
  CORBA_Object_release(poa, &ev);
  CORBA_ORB_destroy(orb, &ev);
  CORBA_Object_release(orb, &ev);
  return 0;
}
```

## The client

```c title="client.c"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <rustorb/CosNaming.h>

#include "Echo.h"

static void check(CORBA_Environment* ev, const char* what) {
  if (ev->_major == CORBA_NO_EXCEPTION) return;
  fprintf(stderr, "client: %s: %s\n", what, CORBA_exception_id(ev));
  exit(1);
}

int main(int argc, char** argv) {
  CORBA_Environment ev = {CORBA_NO_EXCEPTION, NULL, NULL};
  CORBA_ORB orb;
  CosNaming_NamingContextExt root;
  Example_Echo echo;
  CORBA_char* reply;
  int shutdown = 0, i;

  orb = CORBA_ORB_init(&argc, argv, "", &ev);
  check(&ev, "ORB_init");
  for (i = 1; i < argc; i++)
    if (strcmp(argv[i], "--shutdown") == 0) shutdown = 1;

  root = (CosNaming_NamingContextExt)CORBA_ORB_resolve_initial_references(orb, "NameService", &ev);
  check(&ev, "NameService");
  echo = (Example_Echo)CosNaming_NamingContextExt_resolve_str(root, "Echo", &ev);
  check(&ev, "resolve_str");

  reply = Example_Echo_echo(echo, "hello from C", &ev);
  check(&ev, "echo");
  printf("client: %s\n", reply);
  CORBA_free(reply);
  if (shutdown) {
    Example_Echo_shutdown(echo, &ev);
    check(&ev, "shutdown");
  }
  CORBA_Object_release(echo, &ev);
  CORBA_Object_release(root, &ev);
  CORBA_ORB_destroy(orb, &ev);
  CORBA_Object_release(orb, &ev);
  return 0;
}
```

What an operation returns belongs to the caller: the string with
`CORBA_free`, references with `CORBA_Object_release`. Errors come back in
the `CORBA_Environment`.

## Build and run

```sh
LIBS="$PREFIX/lib/librustorb_c.a $PREFIX/lib/librustorb_ffi.a -lpthread -ldl -lm"
cc -std=c99 -I$PREFIX/include -Igen server.c gen/Echo.c $LIBS -o server
cc -std=c99 -I$PREFIX/include -Igen client.c gen/Echo.c $LIBS -o client

NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
./server $NS &
./client $NS --shutdown
```

On macOS, use `-framework CoreFoundation -framework Security -lpthread -lm`
in place of `-lpthread -ldl -lm`.

```text
server: ready
server: hello from C
client: hello from C
```

## Next

- [Run them together](together.md) with clients and servers in other languages.
- The binding in full: [c-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/c-binding.md).

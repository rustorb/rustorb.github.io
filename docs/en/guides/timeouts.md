# Timeouts and SyncScope

## Call timeouts

A call waits for its reply until a timeout, if one is set; then it raises
`CORBA.TIMEOUT`. A timeout can be set at four levels, and a call uses the
first that is set:

1. the request (Rust only),
2. the object reference (shared with the references narrowed from it),
3. the calling thread,
4. the ORB (initially `-ORBclientCallTimeOutPeriod MS`).

The reference comes before the thread, as in omniORB. The connect timeout
(`-ORBclientConnectTimeOutPeriod MS`) limits how long a new connection may
take.

| When the time runs out | Exception | What the server gets |
| --- | --- | --- |
| while connecting | `TIMEOUT`, `COMPLETED_NO` | nothing |
| before the request is sent | `TIMEOUT`, `COMPLETED_NO` | nothing |
| after the request is sent | `TIMEOUT`, `COMPLETED_MAYBE` | a GIOP `CancelRequest`; the connection stays open |

### The rustorb API and the omniORB one

| Level | Python | C++ | Java | omniORBpy compatibility |
| --- | --- | --- | --- | --- |
| Reference | `rustorb.timeouts.set_object_call_timeout(obj, s)` | `rustorb::setClientCallTimeout(obj, ms)` | `Timeouts.setClientCallTimeout(obj, ms)` | `omniORB.setClientCallTimeout(obj, ms)` |
| Thread | `rustorb.timeouts.set_thread_call_timeout(s)` | `rustorb::setClientThreadCallTimeout(ms)` | `Timeouts.setClientThreadCallTimeout(ms)` | `omniORB.setClientThreadCallTimeout(ms)` |
| ORB | `rustorb.timeouts.set_call_timeout(s)` | `rustorb::setClientCallTimeout(ms)` | `Timeouts.setClientCallTimeout(ms)` | `omniORB.setClientCallTimeout(ms)` |
| Connect | `rustorb.timeouts.set_connect_timeout(s)` | `rustorb::setClientConnectTimeout(ms)` | `Timeouts.setClientConnectTimeout(ms)` | `omniORB.setClientConnectTimeout(ms)` |

Rust has `Request::timeout`, `ObjectRef::set_call_timeout`,
`rustorb_core::set_thread_call_timeout` and `Orb::set_call_timeout`.
`rustorb.timeouts` takes seconds; the omniORB-style functions take
milliseconds.

### The standard way: `RelativeRoundtripTimeoutPolicy`

The same timeouts can be set with CORBA Messaging policies: on a
reference with `_set_policy_overrides`, on the thread through
`PolicyCurrent`, on the ORB through `ORBPolicyManager`. The value is a
`TimeBase::TimeT`, in units of 100 ns.

## SyncScope: when a oneway call returns

A oneway call has no reply, so by default it returns once the request is
written to the connection. The `SyncScopePolicy` changes that:

| `Messaging::SyncScope` | The call returns |
| --- | --- |
| `SYNC_NONE` | at once; the ORB writes the request later |
| `SYNC_WITH_TRANSPORT` (default) | once the request is written |
| `SYNC_WITH_SERVER` | once the server has found the object, before the servant runs |
| `SYNC_WITH_TARGET` | once the servant has run |

With `SYNC_WITH_SERVER` and `SYNC_WITH_TARGET` the server answers, so the
call raises the system exceptions of a two-way call (`OBJECT_NOT_EXIST`,
`TRANSIENT`, `TIMEOUT`, ...). The policy is set like the timeout policy, on
a reference, the thread or the ORB, and works with omniORB and other ORBs'
servers (it is the `response_flags` of GIOP 1.2).

## Example

```python title="timeout_example.py"
import sys
import time

from rustorb import CORBA, Messaging, TimeBase, timeouts
import Demo, Demo__POA


class SlowCalc(Demo__POA.Calc):
    def add(self, a, b):
        time.sleep(1.0)
        return a + b

    def post(self, message):
        time.sleep(1.0)


# The server: its own ORB, listening on TCP.
server = CORBA.ORB_init(["-ORBendPoint", "giop:tcp:127.0.0.1:"], "server")
server.resolve_initial_references("RootPOA")._get_the_POAManager().activate()
ior = server.object_to_string(SlowCalc()._this())

orb = CORBA.ORB_init(sys.argv, "client")
calc = orb.string_to_object(ior)._narrow(Demo.Calc)

# A timeout of 0.3 s on this reference.
timeouts.set_object_call_timeout(calc, 0.3)
try:
    calc.add(1, 2)
except CORBA.TIMEOUT as e:
    print("timed out", e.completed == CORBA.COMPLETED_MAYBE)   # timed out True

# The same with the standard policy.
rt = orb.create_policy(
    Messaging.RELATIVE_RT_TIMEOUT_POLICY_TYPE,
    CORBA.Any(TimeBase._tc_TimeT, 3_000_000),                 # 0.3 s
)
obj = orb.string_to_object(ior)._set_policy_overrides([rt], CORBA.SET_OVERRIDE)
try:
    obj._narrow(Demo.Calc).add(1, 2)
except CORBA.TIMEOUT:
    print("timed out again")


# SyncScope: a oneway call that returns once the servant has run.
def seconds(calc):
    start = time.monotonic()
    calc.post("hello")
    return round(time.monotonic() - start, 1)


plain = orb.string_to_object(ior)._narrow(Demo.Calc)
target = orb.create_policy(
    Messaging.SYNC_SCOPE_POLICY_TYPE,
    CORBA.Any(Messaging._tc_SyncScope, Messaging.SYNC_WITH_TARGET),
)
waiting = plain._set_policy_overrides([target], CORBA.SET_OVERRIDE)._narrow(Demo.Calc)
print(seconds(plain), seconds(waiting))                       # 0.0 1.0

orb.destroy()
server.destroy()
```

More: [timeouts.md](https://github.com/rustorb/rustorb/blob/main/docs/timeouts.md),
[sync-scope.md](https://github.com/rustorb/rustorb/blob/main/docs/sync-scope.md),
[orb-operations.md](https://github.com/rustorb/rustorb/blob/main/docs/orb-operations.md).

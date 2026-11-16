# タイムアウトと SyncScope

## 呼び出しのタイムアウト

タイムアウトを設定すると、呼び出しはその時間まで返事を待ち、過ぎると `CORBA.TIMEOUT` になります。タイムアウトは 4 つの範囲で設定でき、呼び出しは次の順で最初に設定されているものを使います。

1. 要求（Rust だけ）
2. オブジェクト参照（そこから narrow した参照と共有）
3. 呼び出したスレッド
4. ORB（初期値は `-ORBclientCallTimeOutPeriod MS`）

omniORB と同じく、参照がスレッドより先です。接続のタイムアウト（`-ORBclientConnectTimeOutPeriod MS`）は、新しい接続にかけてよい時間です。

| 時間が切れたところ | 例外 | サーバに届くもの |
| --- | --- | --- |
| 接続中 | `TIMEOUT`、`COMPLETED_NO` | なし |
| 要求を送る前 | `TIMEOUT`、`COMPLETED_NO` | なし |
| 要求を送った後 | `TIMEOUT`、`COMPLETED_MAYBE` | GIOP の `CancelRequest`。接続はそのまま使います |

### rustorb の API と omniORB の API

| 範囲 | Python | C++ | Java | omniORBpy 互換 |
| --- | --- | --- | --- | --- |
| 参照 | `rustorb.timeouts.set_object_call_timeout(obj, s)` | `rustorb::setClientCallTimeout(obj, ms)` | `Timeouts.setClientCallTimeout(obj, ms)` | `omniORB.setClientCallTimeout(obj, ms)` |
| スレッド | `rustorb.timeouts.set_thread_call_timeout(s)` | `rustorb::setClientThreadCallTimeout(ms)` | `Timeouts.setClientThreadCallTimeout(ms)` | `omniORB.setClientThreadCallTimeout(ms)` |
| ORB | `rustorb.timeouts.set_call_timeout(s)` | `rustorb::setClientCallTimeout(ms)` | `Timeouts.setClientCallTimeout(ms)` | `omniORB.setClientCallTimeout(ms)` |
| 接続 | `rustorb.timeouts.set_connect_timeout(s)` | `rustorb::setClientConnectTimeout(ms)` | `Timeouts.setClientConnectTimeout(ms)` | `omniORB.setClientConnectTimeout(ms)` |

Rust には `Request::timeout`、`ObjectRef::set_call_timeout`、`rustorb_core::set_thread_call_timeout`、`Orb::set_call_timeout` があります。`rustorb.timeouts` は秒、omniORB 形式の関数はミリ秒で指定します。

### 標準の書き方：`RelativeRoundtripTimeoutPolicy`

同じタイムアウトを CORBA Messaging のポリシーでも設定できます。参照には `_set_policy_overrides`、スレッドには `PolicyCurrent`、ORB には `ORBPolicyManager` を使います。値は `TimeBase::TimeT` で、単位は 100 ns です。

## SyncScope：oneway 呼び出しが戻るとき

oneway 呼び出しには返事がないので、既定では要求を接続に書き込んだところで戻ります。`SyncScopePolicy` でこれを変えられます。

| `Messaging::SyncScope` | 呼び出しが戻るとき |
| --- | --- |
| `SYNC_NONE` | すぐ。書き込みは ORB が後で行います |
| `SYNC_WITH_TRANSPORT`（既定） | 要求を書き込んだとき |
| `SYNC_WITH_SERVER` | サーバがオブジェクトを見つけたとき（サーバントが動く前） |
| `SYNC_WITH_TARGET` | サーバントが終わったとき |

`SYNC_WITH_SERVER` と `SYNC_WITH_TARGET` ではサーバが返事をするので、二方向の呼び出しと同じシステム例外（`OBJECT_NOT_EXIST`、`TRANSIENT`、`TIMEOUT` など）が上がります。ポリシーはタイムアウトのポリシーと同じく、参照、スレッド、ORB に設定します。GIOP 1.2 の `response_flags` なので、omniORB など他の ORB のサーバとも使えます。

## 例

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

詳しくは [timeouts.md](https://github.com/rustorb/rustorb/blob/main/docs/timeouts.md)、[sync-scope.md](https://github.com/rustorb/rustorb/blob/main/docs/sync-scope.md)、[orb-operations.md](https://github.com/rustorb/rustorb/blob/main/docs/orb-operations.md) を見てください。

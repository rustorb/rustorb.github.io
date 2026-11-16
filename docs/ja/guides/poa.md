# POA

サーバは **POA**（Portable Object Adapter）を通してオブジェクトを公開します。プロセスと同じだけ生きるオブジェクトなら、[チュートリアル](../tutorial/index.md)の RootPOA で足ります。自分で POA を作ると、オブジェクトの名前の付け方、参照が有効な期間、要求がサーバントに届く方法を選べます。

## ポリシー

`create_POA(name, manager, policies)` で子の POA を作ります。各ポリシーの既定値（表の最初の値）は、注記したものを除いて RootPOA と同じです。

| ポリシー | 値（最初が既定） |
| --- | --- |
| Thread | `ORB_CTRL_MODEL`、`SINGLE_THREAD_MODEL`、`MAIN_THREAD_MODEL` |
| Lifespan | `TRANSIENT`、`PERSISTENT` |
| IdUniqueness | `UNIQUE_ID`、`MULTIPLE_ID` |
| IdAssignment | `SYSTEM_ID`、`USER_ID` |
| ImplicitActivation | `NO_IMPLICIT_ACTIVATION`（RootPOA は `IMPLICIT_ACTIVATION`）、`IMPLICIT_ACTIVATION` |
| ServantRetention | `RETAIN`、`NON_RETAIN` |
| RequestProcessing | `USE_ACTIVE_OBJECT_MAP_ONLY`、`USE_DEFAULT_SERVANT`、`USE_SERVANT_MANAGER` |

- `ORB_CTRL_MODEL` は要求をスレッドプールで並行に動かします。`SINGLE_THREAD_MODEL` はその POA の要求を 1 つずつ動かします。`MAIN_THREAD_MODEL` は `orb.run()` か `orb.perform_work()` を呼んだスレッドで動かします。
- `PERSISTENT` の参照は、サーバが同じポート、同じ POA の名前、同じオブジェクト ID で起動し直せば有効なままです。こうしたサーバにはポートを固定してください（`-ORBendPoint giop:tcp::2810`）。
- `USER_ID` にすると、アプリケーションがオブジェクト ID を決めます（`activate_object_with_id`）。
- サーバントマネージャ（`ServantActivator`、`ServantLocator`）、デフォルトサーバント、アダプタアクティベータも使えます。

矛盾する組み合わせ（`NON_RETAIN` と `USE_ACTIVE_OBJECT_MAP_ONLY`、`IMPLICIT_ACTIVATION` と `USER_ID` など）は `InvalidPolicy` になります。

## POA マネージャ

POA マネージャは、それに属する POA の要求の扱いを切り替えます。状態は `holding`（初期状態。要求は待たされます）、`active`、`discarding`（要求は `TRANSIENT` で失敗します）、`inactive` です。複数の POA で 1 つのマネージャを共有できます。

`POAManagerFactory`（CORBA 3）は、名前を付けたマネージャを作り、探し、一覧にします。マネージャにはすべて ID があり、RootPOA のものは `RootPOAManager`、`create_POA` にマネージャを渡さなかったときに作られるものは `POAManager<n>` です。マネージャは最後の POA がなくなると消えます。

## 例

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

`calc-1` の参照（`orb.object_to_string(calc)`）は、このプログラムを起動し直しても使えます。

## 他の言語では

| | Rust | C++ | Java |
| --- | --- | --- | --- |
| 子の POA | `Poa::create_poa(name, Some(&manager), &[Policy])` | `poa->create_POA(name, manager, policies)` | `poa.create_POA(name, manager, policies)` |
| ポリシー | `Policy::Lifespan(Lifespan::Persistent)` など | `poa->create_lifespan_policy(...)` など | `poa.create_lifespan_policy(...)` など |
| ファクトリ | `Poa::the_poa_manager_factory()` | `poa->the_POAManagerFactory()` | `poa.the_POAManagerFactory()` |
| マネージャを作る | `PoaManagerFactory::create_poa_manager(id, policies)` | `create_POAManager(id, policies)` | `create_POAManager(id, policies)` |
| 探す・一覧 | `find(id)`、`list()` | `find(id)`、`list()` | `find(id)`、`list()` |
| マネージャの ID | `PoaManager::get_id()` | `manager->get_id()` | `manager.get_id()` |

詳しくは [poa-design.md](https://github.com/rustorb/rustorb/blob/main/docs/poa-design.md)（スレッド、サーバントマネージャ、omniORB との比較）を見てください。

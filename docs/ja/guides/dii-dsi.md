# DII と DSI

**DII**（Dynamic Invocation Interface）は、クライアントが生成コードを持たない操作を呼び出します。操作の名前を指定し、引数を any で渡します。**DSI**（Dynamic Skeleton Interface）はそのサーバ側で、1 つのサーバントがすべての要求を `ServerRequest` として受け取り、引数を自分で読みます。ブリッジ、ゲートウェイ、スクリプトのツール、テストドライバなどで使います。DII のクライアントと DSI のサーバは、どの ORB の普通のスタブやサーバントとも話せます。線上では違いがありません。

rustorb では Rust、Python、C++、Java のすべてで使えます。

## 例

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

操作が投げるユーザー例外は `req.add_exception(Demo._tc_Overflow)` で宣言します（C++ は `exceptions()->add(tc)`、Java は `exceptions().add(tc)`）。Python ではその例外（`Demo.Overflow`）が上がり、C++ と Java では any に例外を入れた `CORBA::UnknownUserException` になります。宣言していない例外は `UNKNOWN` になります。`orb.send_multiple_requests_deferred` と `get_next_response` で、多くの要求をまとめて扱えます。

## 他の言語では

| | Rust | C++ | Java |
| --- | --- | --- | --- |
| 要求を作る | `ObjectRef::create_request(op)` | `obj->_request(op)` | `obj._request(op)` |
| 引数 | `DiiRequest` のビルダー | `add_in_arg()` が `Any&` を返す | `add_in_arg()` が `Any` を返す |
| 送る | 同期、async、oneway、deferred | `invoke`、`send_oneway`、`send_deferred` | `invoke`、`send_oneway`、`send_deferred` |
| エラー | `Result` | 投げる。ユーザー例外は `CORBA::UnknownUserException` | 投げずに `req.env()` に入る（JacORB と同じ） |
| DSI のサーバント | `DynamicImplementation` を実装 | `PortableServer::DynamicImplementation` から派生 | `org.omg.PortableServer.DynamicImplementation` を継承 |

C++ と Java では要求の any が valuetype を参照で持つので、引数の間で共有した valuetype は共有のまま届きます（[Any と valuetype](any-valuetype.md#valuetypes-in-an-any)）。

使えないもの：コンテキスト（`ctx`、`ctxlist`、`get_default_context`）は `NO_IMPLEMENT` です。`create_operation_list` はインタフェースリポジトリが要るので、ありません。

詳しくは [dii-dsi.md](https://github.com/rustorb/rustorb/blob/main/docs/dii-dsi.md) を見てください。

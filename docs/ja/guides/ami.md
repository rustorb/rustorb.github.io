# コールバック AMI

**コールバック型の AMI**（Asynchronous Method Invocation、CORBA Messaging の機能）では、クライアントは `<op>` の代わりに `sendc_<op>` を呼びます。呼び出しはすぐに戻り、返事が届くと、ORB がクライアントの実装した**応答ハンドラ**の同じ名前の操作を呼びます。サーバは関わりません。サーバから見れば普通の要求なので、AMI のクライアントはどのサーバとも動きます。

rustorb では Python、C++、Java で、`rustorb-idl -Wbami`（rustorb-idl 0.3.0 以降）で生成したコードと一緒に使えます。Rust には代わりに [`async fn <op>_async`](https://github.com/rustorb/rustorb/blob/main/docs/async-api.md) があります。

## `-Wbami` が生成するもの

インタフェース `I` ごとに、ハンドラのインタフェース `AMI_IHandler` と、`I` のスタブに各操作・属性の `sendc_` 操作を生成します。`Demo::Calc` では次のとおりです。

```idl
// Implied IDL: what -Wbami adds for Calc.
interface AMI_CalcHandler : Messaging::ReplyHandler {
  void add(in long ami_return_val);
  void add_excep(in Messaging::ExceptionHolder excep_holder);
  void echo_pair(in Pair ami_return_val);
  void echo_pair_excep(in Messaging::ExceptionHolder excep_holder);
  // echo_any likewise; nothing for the oneway post
};
// In the stub of Calc:
//   void sendc_add(in AMI_CalcHandler ami_handler, in long a, in long b);
//   void sendc_echo_pair(in AMI_CalcHandler ami_handler, in Pair p);
```

- `sendc_<op>` は、ハンドラと、`in` と `inout` の引数を取ります。
- ハンドラの `<op>` は、戻り値（`ami_return_val`）と、`inout` と `out` の引数を取ります。
- `<op>_excep` は、ユーザー例外もシステム例外も `ExceptionHolder` で受け取ります。`raise_exception()` で投げ直せます。
- 属性には `get_<attr>` と `set_<attr>` ができます。

## 例

```python title="ami_example.py"
import sys
import threading

from rustorb import CORBA
import Demo, Demo__POA
from calc_impl import CalcImpl


class AddHandler(Demo__POA.AMI_CalcHandler):
    def __init__(self):
        self.done = threading.Event()

    def add(self, ami_return_val):
        print("result:", ami_return_val)
        self.done.set()

    def add_excep(self, excep_holder):
        try:
            excep_holder.raise_exception()
        except Demo.Overflow as e:
            print("overflow at", e.limit)
        except CORBA.SystemException as e:
            print("failed:", e)
        self.done.set()


orb = CORBA.ORB_init(sys.argv)
poa = orb.resolve_initial_references("RootPOA")
poa._get_the_POAManager().activate()
calc = CalcImpl()._this()

handler = AddHandler()
calc.sendc_add(handler._this(), 20, 22)   # returns at once
handler.done.wait()                       # result: 42

handler = AddHandler()
calc.sendc_add(handler._this(), 999, 2)
handler.done.wait()                       # overflow at 1000

orb.destroy()
```

## 返事の届き方

- ハンドラは ORB の返事用のスレッドプールで動きます。クライアントが `orb.run()` を呼ばなくても届きます。`MAIN_THREAD_MODEL` の POA にあるハンドラだけは、メインスレッドが `run` か `perform_work` を呼ぶまで待ちます。
- 返事は届いた順に、複数のスレッドで同時に呼ばれることがあります。
- `sendc_` が戻った後に起きた失敗（接続できない、`TRANSIENT`、[タイムアウト](timeouts.md)など）は、すべて `<op>_excep` に届きます。`sendc_` が自分で投げるのは、ORB が破棄されているなど、送る前に分かるものだけです。
- ハンドラに nil を渡すこともできます。要求は送られ、返事は捨てられます。
- ハンドラは普通のオブジェクトなので、別のプロセスにあっても構いません。

## 他の言語では

| | C++ | Java |
| --- | --- | --- |
| 生成 | `rustorb-idl -bc++11 -Wbami` | `rustorb-idl -bjava -Wbami` |
| ハンドラのサーバント | `CORBA::servant_traits<Demo::AMI_CalcHandler>::base_type` から派生 | `Demo.AMI_CalcHandlerPOA` を継承 |
| 呼び出し | `calc->sendc_add(handler, 20, 22)` | `calc.sendc_add(handler, 20, 22)` |
| 例外 | `Messaging::ExceptionHolder`（`<rustorb/messaging.h>`） | `org.omg.Messaging.ExceptionHolder` |

使えないもの：ポーリング型（`sendp_` とポーラー）と、時間に依存しない呼び出し（ルーター）。

詳しくは [ami.md](https://github.com/rustorb/rustorb/blob/main/docs/ami.md) を見てください。

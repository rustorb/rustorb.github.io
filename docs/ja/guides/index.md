# ガイド

[チュートリアル](../tutorial/index.md)ではサーバとクライアントを 1 つずつ作りました。このガイドでは、rustorb のそれ以外の機能を 1 つずつ紹介します。

<div class="grid cards" markdown>

-   **[POA](poa.md)**: ポリシー、オブジェクト ID、永続的な参照、POA マネージャと `POAManagerFactory`
-   **[Any と valuetype](any-valuetype.md)**: 任意の型の値と、値渡しのオブジェクト（共有されるかどうか）
-   **[DII と DSI](dii-dsi.md)**: 生成コードなしで操作を呼び出し、受け付ける
-   **[コールバック AMI](ami.md)**: すぐに戻り、返事をハンドラで受け取る呼び出し
-   **[通信路](transports.md)**: TLS、Unix ドメインソケット、クライアントの接続を使ったコールバック、NAT の内側のアドレス
-   **[タイムアウトと SyncScope](timeouts.md)**: 呼び出しが待つ時間と、oneway 呼び出しが戻るとき
-   **[omniORBpy からの移行](omniorbpy.md)**: omniORBpy のアプリケーションを rustorb で動かす
-   **[速さと調整](tuning.md)**: 呼び出しを速くしている仕組み、それを変えるオプション、octet の列を `Octets` や String で受け取る方法

</div>

## 例について

例は Python で書いています。短く、1 つのファイルで動くからです。各ガイドには、同じ API の Rust、C++、Java での名前の表があります。C#・C・Ruby はそれぞれのマッピングに従い、名前はバインディングのドキュメント（[.NET](https://github.com/rustorb/rustorb/blob/main/docs/csharp-binding.md)、[C](https://github.com/rustorb/rustorb/blob/main/docs/c-binding.md)、[Ruby](https://github.com/rustorb/rustorb/blob/main/docs/ruby-binding.md)）にあります。詳しいことは各ガイドからリンクしている設計ドキュメントにあります。

例はすべて次の IDL を使います。

```idl title="Demo.idl"
module Demo {
  valuetype Point {
    public long x;
    public long y;
  };
  struct Pair {
    Point first;
    Point second;
  };
  exception Overflow { long limit; };
  interface Calc {
    long add(in long a, in long b) raises (Overflow);
    Pair echo_pair(in Pair p);
    any echo_any(in any a);
    oneway void post(in string message);
  };
};
```

サーバントも共通で、`calc_impl.py` に置きます。

```python title="calc_impl.py"
import Demo, Demo__POA


class CalcImpl(Demo__POA.Calc):
    def add(self, a, b):
        if a + b > 1000:
            raise Demo.Overflow(1000)
        return a + b

    def echo_pair(self, p):
        return p

    def echo_any(self, a):
        return a

    def post(self, message):
        print("posted:", message)
```

コードを生成し（`-Wbami` は[コールバック AMI](ami.md) に要るものを足します）、生成したコードをパスに入れて例を動かします。

```sh
pip install rustorb rustorb-idl
rustorb-idl -bpython -Wbami -C gen Demo.idl
PYTHONPATH=gen:. python3 poa_example.py
```

どの例も、サーバとクライアントを同じプロセスで動かします。呼び出しは 2 つのプロセスの間と同じく ORB を通ります。

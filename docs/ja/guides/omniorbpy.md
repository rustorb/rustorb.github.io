# omniORBpy からの移行

rustorb の Python バインディングは、標準の IDL to Python マッピングに `omniidl -bpython` と同じ配置で従うので、omniORBpy 向けに書いたコードはたいていそのまま動きます。パッケージ `rustorb-omniorb-compat` は `omniORB` モジュールと、アプリケーションが使う omniORBpy の拡張を足します。OpenRTM-aist-Python と rtshell がそのまま動く範囲です。

## 切り替え方

```sh
pip install rustorb rustorb-omniorb-compat rustorb-idl
```

1. スタブを `omniidl -bpython` ではなく `rustorb-idl -bpython` で生成し直します。生成されるモジュールの名前と API は同じですが、omniidl の生成物は omniORBpy の内部を呼ぶので rustorb では動きません。`-Wbpackage=` は omniidl と同じく使えます。
2. import はそのままです。`from omniORB import CORBA, PortableServer`、`import CosNaming`、トップレベルの `CORBA` と `PortableServer` が rustorb のモジュールになります。
3. コマンドラインもそのままです。`-ORB...` のオプションは omniORB と同じ名前です（[ORB オプション](../reference/orb-options.md)）。

互換パッケージを使わないときは `from rustorb import CORBA, PortableServer` と書きます。

## 互換パッケージにあるもの

| omniORBpy の API | 備考 |
| --- | --- |
| `omniORB.CORBA`、`omniORB.PortableServer`、`CORBA`、`PortableServer`、`CosNaming` | rustorb のモジュール |
| `omniORB.cdrMarshal`、`cdrUnmarshal` | omniORB と同じ符号化（文字列は ISO 8859-1）なので、バイト列も同じです |
| `omniORB.any.to_any`、`from_any` | TypeCode の選び方も omniORBpy と同じです |
| `omniORB.findType`、`omniORB.tcInternal` | OpenRTM-aist が読む型記述子 |
| `resolve_initial_references("omniINSPOA")` | オブジェクト ID を `corbaloc` のキーにします（`corbaloc::host:2810/manager`） |
| `omniORB.setClientCallTimeout` などのタイムアウトの関数 | [タイムアウト](timeouts.md) |
| `omniORB.URI`：`stringToName`、`nameToString`、`addrAndNameToURI` | omniORBpy 4.3.4 で実測した規則 |
| `TRANSIENT_ConnectFailed` などのマイナーコード | rustorb が上げる値にしてあります |

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

## 知っておくとよい違い

- システム例外のマイナーコードは omniORB のものではなく rustorb のものです。互換パッケージの定数も rustorb に合わせています。
- `omniORB.setClientThreadCallDeadline`、`installTimeoutExceptionHandler`、`sslTP` / `httpTP` モジュールはありません。TLS は ORB オプションで設定します（[通信路](transports.md#tls-ssliop)）。
- omniNames は rustorb から使えますし、`rustorb-names`（永続化つきの代替）は omniORBpy から使えます。[ネーミングサービス](../reference/naming.md)を見てください。

詳しくは [omniorb-compat.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-compat.md)、[python-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/python-binding.md) を見てください。

# クイックスタート

Python で書いた CORBA のサーバとクライアントを動かします。必要なのは Python 3.9 以降か Docker だけです。そのあと[チュートリアル](tutorial/index.md)で、同じことをすべての言語で、ネーミングサービスを使って行います。

## 1. rustorb の入ったシェルを用意する

=== "pip"

    新しいディレクトリと仮想環境を作ります。

    ```sh
    mkdir rustorb-quickstart && cd rustorb-quickstart
    python3 -m venv .venv
    . .venv/bin/activate
    pip install rustorb rustorb-idl
    ```

    Windows では `.venv\Scripts\activate` で有効にします。

=== "Docker"

    手元のマシンには何も入れません。新しいディレクトリを `/work` にマウントして Python のコンテナを起動するので、以下のファイルは手元のエディタで書けます（コンテナは抜けると消えます）。

    ```sh
    mkdir rustorb-quickstart && cd rustorb-quickstart
    docker run --rm -it -v "$PWD":/work -w /work python:3.12-slim bash
    ```

    コンテナの中で:

    ```sh
    pip install rustorb rustorb-idl
    ```

    以降の手順のコマンドは、このシェルで実行します。

`rustorb` が ORB 本体（Rust のコアをビルド済みの wheel。Linux、macOS、Windows 用）、`rustorb-idl` が IDL コンパイラです。

## 2. インタフェースを書く

次の内容を `Echo.idl` として保存します。

```idl
module Example {
  interface Echo {
    string echo(in string message);
  };
};
```

ここから Python のコードを生成します。

```sh
rustorb-idl -bpython Echo.idl
```

`Echo_idl.py` と、パッケージ `Example`（クライアント用スタブ）、`Example__POA`（サーバ用スケルトン）ができます。

## 3. サーバを書く

`server.py` はインタフェースを実装し、オブジェクトを活性化して、その参照（IOR）をファイルに書きます。

```python
import sys
from rustorb import CORBA, PortableServer
import Example, Example__POA

class EchoImpl(Example__POA.Echo):
    def echo(self, message):
        print("server got:", message, flush=True)
        return message

orb = CORBA.ORB_init(sys.argv)
poa = orb.resolve_initial_references("RootPOA")
poa._get_the_POAManager().activate()
ref = EchoImpl()._this()
with open("echo.ior", "w") as f:
    f.write(orb.object_to_string(ref))
print("server ready, IOR in echo.ior", flush=True)
orb.run()
```

## 4. クライアントを書く

`client.py` は参照を読み込んで、オブジェクトを呼び出します。

```python
import sys
from rustorb import CORBA
import Example

orb = CORBA.ORB_init(sys.argv)
ior = open("echo.ior").read()
echo = orb.string_to_object(ior)._narrow(Example.Echo)
print(echo.echo("hello, rustorb"))
orb.destroy()
```

## 5. 動かす

```sh
python server.py &
python client.py
```

```text
server ready, IOR in echo.ior
server got: hello, rustorb
hello, rustorb
```

サーバは `kill %1`（または `fg` して ++ctrl+c++）で止めます。

2 つのプロセスは TCP 上の GIOP で通信しています。そのため、このサーバはどの CORBA クライアントからも呼べ、このクライアントはどの CORBA サーバでも呼べます。omniORB、omniORBpy、JacORB、別の言語の rustorb のどれでも構いません。

## 次に

- 使う言語向けに rustorb を[インストール](install/index.md)します。
- [チュートリアル](tutorial/index.md)では、同じ `Echo` を Rust、Python、C++、Java で書き、IOR ファイルではなくネーミングサービスで名前から探します。
- omniORBpy から移行する場合は [Python](install/python.md#coming-from-omniorbpy) を見てください。

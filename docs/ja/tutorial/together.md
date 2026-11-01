# つないで動かす

どのサーバも同じネーミングサービスに `Echo` として自分を登録し、どのクライアントもそこで `Echo` を探すので、どのクライアントもどのサーバとつながります。[チュートリアル](index.md)のネーミングサービスを起動したまま:

```sh
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"

python server.py $NS &                   # Python のサーバを
./echo-rs/target/debug/client $NS        # Rust から呼び
./echo-cpp/build/client $NS --shutdown   # C++ からも呼んで止める

./echo-rs/target/debug/server $NS &      # Rust のサーバを
python client.py $NS --shutdown          # Python から呼ぶ
```

```text
client: hello from Rust
client: hello from C++
client: hello from Python
```

通信は TCP 上の素の GIOP 1.2 なので、omniORB、omniORBpy、JacORB のプログラムも同じように加われます。`rustorb-names` の代わりに `omniNames` を使うこともできます。

## 16 通りすべて

リポジトリには、4 つの例をビルドし、新しく起動した `rustorb-names` に対してすべてのサーバとすべてのクライアントを動かすスクリプトがあります。

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
examples/run-all.sh                      # cargo、cmake、C++ コンパイラ、Python 3.9+、Java 22+、Maven が必要
EXAMPLES="rust python" examples/run-all.sh
```

## ネーミングサービスを使わない場合

参照は文字列として渡すこともできます。サーバが `orb.object_to_string(ref)`（`IOR:...`）を出力し、クライアントが `orb.string_to_object(...)` で参照に戻します。[クイックスタート](../quickstart.md)がこの方法です。

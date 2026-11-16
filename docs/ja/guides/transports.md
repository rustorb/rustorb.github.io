# 通信路

どのバインディングも、ここにある設定を ORB オプションとして、コマンドラインか `ORB_init` の引数で受け取ります（Java はプロパティでも）。オプションの名前は omniORB と同じです。一覧は [ORB オプション](../reference/orb-options.md)にあります。

## アドレス：サーバが待ち受ける場所と、公開する場所

```sh
server -ORBendPoint giop:tcp::2810                          # all addresses, port 2810
server -ORBendPoint giop:tcp:10.0.0.5:2810                  # one address
server -ORBendPoint giop:tcp::2810 -ORBendPointPublish giop:tcp:public.example.org:2810
```

IOR には、クライアントが使うアドレスが入ります。NAT の内側、コンテナの中、ポートフォワードを使うときは、`-ORBendPointPublish` でクライアントから届くアドレスを公開してください。`-ORBendPoint` がなければ、サーバはすべてのアドレスの空いているポートで待ち受けます。

## TLS（SSLIOP） {#tls-ssliop}

TLS の上の IIOP です。omniORB の SSL トランスポートとつながり、rustls で作っています（OpenSSL は使いません）。

```sh
server -ORBendPoint giop:ssl::2811 \
       -ORBsslCAFile ca.pem -ORBsslKeyFile server.pem
client -ORBsslCAFile ca.pem
```

| オプション | 意味 |
| --- | --- |
| `-ORBendPoint giop:ssl:HOST:PORT` | TLS で待ち受けます。`giop:tcp` の待ち受けがなければ、平文の TCP では待ち受けません |
| `-ORBsslCAFile PATH` | 相手を確かめる CA（PEM）。これがないクライアントは TLS を使いません |
| `-ORBsslKeyFile PATH`、`-ORBsslCertFile PATH` | 自分の鍵と証明書（PEM。omniORB と同じく、鍵のファイルに両方入れても構いません） |
| `-ORBsslVerifyClient none\|optional\|required` | サーバがクライアントの証明書を求めるか（既定は `none`） |
| `-ORBsslVerifyHost 0\|1` | クライアントがサーバの証明書のホスト名を確かめるか（既定は 1） |

CA を持つクライアントは、TLS のアドレスを持つサーバにはそのアドレスだけを使い、平文の TCP には落としません。CSIv2 は使えません。

## Unix ドメインソケット（`giop:unix`）

omniORB の、同じホストのプロセスの間の通信路です。Linux と macOS で使えます。

```sh
server -ORBendPoint giop:unix:/run/myapp/calc.sock
server -ORBendPoint giop:unix:                  # a new socket in /tmp/omni-USER/
server -ORBendPoint giop:unix:/run/myapp/       # a new socket in that directory
server -ORBendPoint giop:unix: -ORBendPoint giop:tcp::2810   # both
```

IOR にはソケットとそのホストが入ります。同じホストのクライアントは TCP や TLS より先にソケットを使い、つながらなければ他のアドレスを使います。別のホストのクライアントはソケットを無視します。サーバは終了するときにソケットのファイルを消します。omniORB と違い、ソケットでないファイルや、他のプロセスが待ち受けているソケットがある場所では起動しません。

## 双方向 GIOP

外からつなげないクライアント（NAT やファイアウォールの内側）でも、自分がサーバに開いた接続でコールバックを受けられます。両側で有効にします。

```sh
server -ORBacceptBiDirectionalGIOP 1
client -ORBofferBiDirectionalGIOP 1
```

クライアントのコールバックのオブジェクトは、`BidirectionalPolicy` が `BOTH` の POA に置きます。

```python
from rustorb import BiDirPolicy, CORBA

both = orb.create_policy(
    BiDirPolicy.BIDIRECTIONAL_POLICY_TYPE,
    CORBA.Any(BiDirPolicy._tc_BidirectionalPolicyValue, BiDirPolicy.BOTH),
)
poa = root_poa.create_POA("callbacks", root_poa._get_the_POAManager(), [both])
```

## 接続

ORB はサーバのアドレスごとに 1 本の接続を持ち、すべての呼び出しで共有します。使われていない接続はしばらくすると閉じます（`-ORBoutConScanPeriod`、`-ORBinConScanPeriod`、秒）。`-ORBmaxServerConnections` でサーバが受け付ける接続の数を制限できます。

詳しくは [ssliop.md](https://github.com/rustorb/rustorb/blob/main/docs/ssliop.md)、[unix-transport.md](https://github.com/rustorb/rustorb/blob/main/docs/unix-transport.md)、[bidir-giop.md](https://github.com/rustorb/rustorb/blob/main/docs/bidir-giop.md)、[connections.md](https://github.com/rustorb/rustorb/blob/main/docs/connections.md) を見てください。

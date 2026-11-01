# ORB オプション

どのバインディングも同じ `-ORB...` オプションを受け取ります。Rust は `Orb::init(&mut args)`、Python は `CORBA.ORB_init(sys.argv)`、C++ は `CORBA::ORB_init(argc, argv)`、Java は `ORB.init(args, props)` で、コマンドラインから渡します（Java ではプロパティでも渡せます）。ORB は読み取ったオプションを引数から取り除きます。

| オプション | 意味 |
| --- | --- |
| `-ORBInitRef NAME=URL` | 初期参照（`corbaloc:`、`corbaname:`、IOR）。例: `NameService=corbaloc::localhost:2809/NameService` |
| `-ORBDefaultInitRef URL` | その他の初期参照を探す場所 |
| `-ORBendPoint giop:tcp:HOST:PORT` | サーバが待ち受ける場所（既定: すべてのアドレスの任意のポート） |
| `-ORBendPointPublish giop:tcp:HOST:PORT` | IOR に書くアドレス（NAT の内側やコンテナの中で使う） |
| `-ORBmaxGIOPVersion 1.1` | 送信する GIOP の最大の版 |
| `-ORBclientCallTimeOutPeriod MS` | すべての呼び出しのタイムアウト（[timeouts.md](https://github.com/rustorb/rustorb/blob/main/docs/timeouts.md)） |
| `-ORBendPoint giop:ssl:HOST:PORT`、`-ORBsslCAFile FILE`、`-ORBsslKeyFile FILE` | SSLIOP（TLS 上の IIOP。[ssliop.md](https://github.com/rustorb/rustorb/blob/main/docs/ssliop.md)） |

オプション名は omniORB と同じなので、既存のコマンドラインやスクリプトはそのまま使えます。

## URL

| 形式 | 例 |
| --- | --- |
| 文字列化した IOR | `IOR:010000001...` |
| `corbaloc:` | `corbaloc::host:2809/NameService`、`corbaloc:iiop:1.2@host:2809/key` |
| `corbaname:` | `corbaname::host:2809#path/to/Echo`（そのネーミングサービスで名前を引く） |

`orb.string_to_object()` は 3 つとも受け付けます。

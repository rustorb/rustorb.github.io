# ネーミングサービス

`rustorb-names` は CosNaming のサーバで、omniNames の代わりに使えます。rustorb、omniORB、omniORBpy、JacORB のクライアントのどれにも使えます。

```sh
cargo install rustorb-naming
```

```text
usage: rustorb-names [-start [PORT]] [-logdir DIR] [-memory] [-ignoreport]
                     [-checkpoint N] [ORB options]

  -start [PORT]   first run: listen on PORT (default 2809) and remember it
  -logdir DIR     where the naming data is kept
                  (default $RUSTORB_NAMES_LOGDIR, else /var/lib/rustorb-names)
  -memory         keep nothing: start empty and forget everything at exit
  -ignoreport     listen only on the -ORBendPoint addresses
  -checkpoint N   write a checkpoint every N updates (default 1000)
```

| オプション | 意味 |
| --- | --- |
| `-start [PORT]` | 初回の起動: PORT（既定 2809）で待ち受け、そのポートを記録する |
| `-logdir DIR` | データを置く場所（既定は `$RUSTORB_NAMES_LOGDIR`、なければ `/var/lib/rustorb-names`） |
| `-memory` | 何も残さない: 空で起動し、終了時にすべて忘れる |
| `-ignoreport` | `-ORBendPoint` のアドレスだけで待ち受ける |
| `-checkpoint N` | N 回の更新ごとにチェックポイントを書く（既定 1000） |

## よくある使い方

```sh
rustorb-names -start 2809 -logdir /tmp/rustorb-names   # 初回
rustorb-names -logdir /tmp/rustorb-names               # 2 回目以降: 同じポートと登録内容
rustorb-names -memory -start 2809                      # 何も残さない（テストやチュートリアル向け）
```

`-logdir` を指定すると、変更はすべてログに追記され、`-checkpoint` 回の更新ごとにチェックポイントが書かれます。そのため omniNames と同じく、再起動しても登録内容が残ります。

プログラムからは次のオプションで参照します。

```sh
-ORBInitRef NameService=corbaloc::localhost:2809/NameService
```

## プログラムから

| 言語 | クライアントのスタブ |
| --- | --- |
| Rust | `rustorb_naming::{NamingContext, NamingContextExt, to_name}` |
| Python | `rustorb-idl -bpython "$(rustorb-idl --idl-dir)/CosNaming.idl"`（または `rustorb-omniorb-compat`） |
| C++ | `rustorb_idl_generate(... "${RUSTORB_IDL_DIR}/CosNaming.idl")` |
| Java | jar の `org.omg.CosNaming` |

`rustorb-naming` クレートを使えば、Rust のプログラムにネーミングサーバを組み込むこともできます（[docs.rs](https://docs.rs/rustorb-naming)）。

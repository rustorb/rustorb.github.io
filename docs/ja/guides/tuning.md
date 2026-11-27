# 速さと調整

rustorb 0.8 は 0.7 より、小さな呼び出し、多くのスレッドからの呼び出し、同じプロセスの中の呼び出し、大きなデータで大きく速くなりました。ほとんどは何もしなくても効きます。このページでは、自動で効くこと、それを変えるオプション、言語のマッピングが変わる 2 つの選択肢を説明します。計測の結果は [omniorb-comparison.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-comparison.md) に、計画は [perf-plan.md](https://github.com/rustorb/rustorb/blob/main/docs/perf-plan.md) にあります。

## 自動で効くこと

- **呼び出しごとのスレッドの切り替えが減りました。** 自分で返事を読んだ呼び出し元は、次の呼び出しまでその接続を読み続けます。要求に答えたアップコールのスレッドは、同じ接続の次の要求を自分で読みます（Linux と macOS。Windows は今までどおり I/O タスクが読みます）
- **1 つのサーバーに複数の接続を張ります。** 複数のスレッドから同じサーバーを同時に呼ぶと、1 本ではなく 4 本まで接続を使います
- **同じプロセスの中の呼び出しは GIOP を通りません。** 引数を 1 度だけマーシャルし、サーバントは呼び出したスレッドで動き、返事はその本体として返ります。POA マネージャーが active でないとき、`SINGLE_THREAD_MODEL`・`MAIN_THREAD_MODEL` の POA、サーバーのインターセプターがあるときは、今までの経路を通ります
- **大きなメッセージを受け取るときに、読む側が起きる回数が減りました**（Linux、TLS なしの TCP）
- **C と Ruby** は数値の列のコピー・解放・確認をまとめて行います

## オプション

| オプション | 既定 | 意味 |
| --- | --- | --- |
| `-ORBmaxConnectionsPerServer N` | 4 | クライアントが 1 つのサーバーのエンドポイントに張る接続の上限（omniORB の `-ORBmaxGIOPConnectionPerServer` も受け付けます）。`1` にすると、0.8 より前と同じく 1 本にまとめます |
| `-ORBfollowLinger US` | 1000 | 呼び出しや要求を終えたスレッドが、その接続を読み続ける時間（µs）。`0` にすると、0.8 より前と同じく I/O タスクがすべて読みます |
| `-ORBfollowLimit N` | 64 | アップコールのスレッドが、1 つの接続の要求を続けて読む数 |
| `-ORBmaxServerThreadPoolSize N` | 100 | 同時に動くアップコールのスレッドの上限 |
| `-ORBstatistics 1` | オフ | ORB の中の動き（起床、読み込み、接続）を数えます。Rust では `rustorb_core::stats` で読めます |

## マッピングが変わる選択肢

標準のマッピングに合わせて書いたコードからは型が違って見えるので、どちらも既定ではオフです。

### Rust：共有の octet の列

`rustorb-idl -brust -Wbshared_octets` は、上限のない `sequence<octet>` を `Vec<u8>` ではなく `rustorb_core::Octets`（`in` は `&Octets`）にします。4 KiB 以上のものは、受け取るときにメッセージから写さず、送るときにメッセージに写しません。ワイヤ上のバイトは同じなので、相手側は `Vec<u8>` のままでかまいません。

### Ruby：octet の列を String で

```ruby
Rustorb.octets_as_string = true
data = target.get_octets(1 << 20)   # Integer の Array ではなく、バイナリの String
```

octet の列と配列を、バイナリの String（`Encoding::BINARY`）で受け取ります。1 MiB では Array より 10 倍以上速くなります。送るときは前から String も受け付けています。設定はプロセス全体に効き、`any` の中の値にも効きます。

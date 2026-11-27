# 性能

rustorb-dds を、各実装で同じ手順（同じ型、QoS、サンプルの待ち方）のプログラムを使って Cyclone DDS、Fast DDS と比べ、OpenDDS と RustDDS とも並べて測っています。測り方、すべての表、残っている課題は [docs/benchmark.md](https://github.com/rustorb/rustorb-dds/blob/main/docs/benchmark.md) に、道具は [bench/](https://github.com/rustorb/rustorb-dds/tree/main/bench) にあります。

!!! warning "相対的な数値です"
    ここの数値は、ほかと共有する CPU 4 個のコンテナ（Intel Xeon 2.1 GHz、Linux 6.18）で測りました。CPU の周波数も割り込みも固定できないので、回ごとに揺れます。同じ回の中で比べてください。別の機械の数値と並べないでください。専用の機械での測定は今後の予定です。

## 往復時間

reliable、UDP、2 つのプロセスの ping と pong の往復時間の中央値（KEEP_LAST 1）、µs、2026-11-26 の測定:

| 大きさ | rustorb-dds | Cyclone DDS 0.10.4 | Fast DDS 2.11.2 |
| ---: | ---: | ---: | ---: |
| 32 B | 79 | 66 | 89 |
| 16 KiB | 143 | 85 | |
| 256 KiB | 800 | 460 | |

小さなサンプルでは Cyclone DDS に近く、Fast DDS より速くなっています。大きなサンプルはまだ Cyclone DDS より時間がかかり、改善を進めています。Cyclone DDS の `ddsperf` のように listener の中で返事を書くと、32 B で 51 µs になります。共有メモリ転送とゼロコピーの貸し出し（Rust）では、1 MiB のサンプルの往復が約 0.1 ms で、UDP の 1/30 です。

## スループット

reliable、UDP、5 秒間に受け取った payload:

- 256 B: rustorb-dds 256 Mbit/s、Cyclone DDS 256 Mbit/s、Fast DDS 149 Mbit/s。
- 1 KiB: rustorb-dds 918 Mbit/s、Cyclone DDS 870 Mbit/s。
- 16 KiB〜1 MiB: rustorb-dds 8.7〜10 Gbit/s、Cyclone DDS 5.3〜8.7 Gbit/s。

## Discovery

N 個のプロセスを同時に起動し、それぞれが同じトピックの参加者・writer・reader を 1 つずつ作ります。最後のプロセスがほかのすべてとマッチするまでの時間（ms、3 回の中央値）:

| 参加者数 | rustorb-dds | Cyclone DDS | Fast DDS |
| ---: | ---: | ---: | ---: |
| 2 | 6 | 9 | 81 |
| 10 | 23 | 42 | 203 |
| 50 | 501 | 747 | 3,792 |

1 プロセスの最大 RSS は、rustorb-dds が 5〜6 MiB、Cyclone DDS が 7〜9 MiB、Fast DDS が 15〜17 MiB でした。

## DDS Security

data と metadata を署名（AES-GMAC）または暗号化（AES-GCM）したときの往復時間の中央値（µs）。各実装はそれぞれ自分の Security プラグインを使います:

| 大きさ | rustorb-dds 署名 | rustorb-dds 暗号化 | Cyclone DDS 署名 | Cyclone DDS 暗号化 | Fast DDS 署名 | Fast DDS 暗号化 |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 32 B | 90 | 92 | 100 | 92 | 210 | 201 |
| 4 KiB | 100 | 112 | 106 | 103 | 207 | 208 |
| 64 KiB | 360 | 551 | 452 | 450 | 490 | 542 |

認証ありで 10 個の参加者がすべてマッチするまで約 90 ms でした（Cyclone DDS は 530〜710 ms）。

## 言語バインディング

各バインディングの ping から Rust の pong への往復時間（共有メモリ、reliable）、µs:

| 大きさ | Rust | C++ | Java | .NET | Python | Ruby |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 32 B | 85 | 96 | 100 | 92 | 120 | 103 |
| 4 KiB | 94 | 98 | 103 | 92 | 118 | 193 |
| 64 KiB | 195 | 215 | 251 | 248 | 270 | 1,245 |

小さなサンプルでバインディングが加える時間は、C++・Java・.NET で 10〜15 µs、Python・Ruby で 20〜35 µs です。

# Python へのインストール

```sh
pip install rustorb rustorb-idl
```

| パッケージ | 中身 |
| --- | --- |
| [`rustorb`](https://pypi.org/project/rustorb/) | ORB 本体: `from rustorb import CORBA, PortableServer` |
| [`rustorb-idl`](https://pypi.org/project/rustorb-idl/) | IDL コンパイラ（`rustorb-idl -bpython`） |
| [`rustorb-omniorb-compat`](https://pypi.org/project/rustorb-omniorb-compat/) | 任意: omniORBpy のアプリケーション向けの `from omniORB import ...` |

`rustorb` は Rust のコアをビルド済みの abi3 wheel で、プラットフォームごとに 1 つです。Rust のツールチェインなしで **CPython 3.9 以降**で動きます。

- Linux x86_64 と aarch64（manylinux 2.28。Ubuntu 20.04 以降、Debian 11 以降、RHEL 8 以降など）
- macOS x86_64 と arm64
- Windows x64

それ以外のプラットフォームでは pip が sdist からビルドします。その場合は Rust 1.85 以降が要ります。

## 確かめる

```sh
python -c "from rustorb import CORBA; print(CORBA.ORB_init([]))"
rustorb-idl --version
```

## IDL からコードを生成する

```sh
rustorb-idl -bpython -C gen Echo.idl
export PYTHONPATH=gen
```

出力は標準の IDL to Python マッピングで、配置は `omniidl -bpython` と同じです。IDL のモジュールごとにスタブのパッケージ（`Example`）と、`__POA` の付いたスケルトンのパッケージ（`Example__POA`）、そして `Echo_idl.py` ができます。ネーミングサービスを使う場合は、そのスタブも生成します。

```sh
rustorb-idl -bpython -C gen "$(rustorb-idl --idl-dir)/CosNaming.idl"
```

## omniORBpy から移行する {#coming-from-omniorbpy}

omniORBpy 向けに書いたコードは、import を変えるだけで rustorb で動きます。

```python
from rustorb import CORBA, PortableServer     # 元は from omniORB import CORBA, PortableServer
```

**そのまま**動かしたい場合（OpenRTM-aist-Python、rtshell など）は、互換パッケージを omniORBpy の*代わりに*入れます（並べて入れないでください）。

```sh
pip uninstall omniORBpy        # pip で入れていた場合
pip install rustorb rustorb-omniorb-compat
```

互換パッケージは `omniORB` パッケージ、トップレベルの `CORBA` と `PortableServer` モジュール、`CosNaming` のスタブを提供します。`omniidl -bpython` で生成したスタブはそのまま使えますが、`rustorb-idl -bpython` で生成し直すことをおすすめします。対応している omniORBpy の拡張は [omniorb-compat.md](https://github.com/rustorb/rustorb/blob/main/docs/omniorb-compat.md)にまとめています。

## ソースから

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
pip install ./python               # maturin でネイティブモジュールをビルド。Rust が必要
pip install ./crates/rustorb-codegen/python
```

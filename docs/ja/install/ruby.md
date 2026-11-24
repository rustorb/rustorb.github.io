# Ruby へのインストール

Ruby バインディングは OMG の Ruby Language Mapping 1.2（`CORBA`、`PortableServer`、`CosNaming`）で、rustorb のライブラリを C 拡張から呼び出します。Linux か macOS の **Ruby 3.0 以降**と、Ruby のヘッダ、C コンパイラが必要です。gem はインストールのときに拡張をビルドします。そのとき、gem に入っている Linux（x86_64、aarch64）と macOS（Apple シリコン）向けの `librustorb_ffi` をリンクします。

IDL からコードを生成するために [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0 以降も使います。

## gem を入れる

```sh
gem install rustorb -v 0.7.1
```

Bundler なら `Gemfile` に `gem "rustorb", "~> 0.7"` と書きます。Debian と Ubuntu では、ヘッダは `ruby-dev`、コンパイラは `build-essential` に入っています。ほかのプラットフォームでは、ソースから `librustorb_ffi.a` をビルドし（下記）、そのパスを渡します。

```sh
RUSTORB_FFI_LIBRARY=/path/to/librustorb_ffi.a \
RUSTORB_FFI_INCLUDE=/path/to/rustorb/crates/rustorb-ffi/include gem install rustorb
```

## IDL からコードを生成する

```sh
rustorb-idl -bruby -C gen Echo.idl      # gen/EchoC.rb（型とスタブ）、gen/EchoS.rb（スケルトン）
ruby -Igen server.rb
```

プログラムは `require "corba"` のあとで、使う生成コードを require します。CosNaming のスタブは gem に入っています（`require "CosNamingC"`）。

## ソースから

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cd ruby && rake compile                 # cargo を動かし、拡張を lib/ にビルドする
ruby -I$PWD/lib -Igen server.rb
```

マッピングは [ruby-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/ruby-binding.md) にあります。

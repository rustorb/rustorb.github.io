# チュートリアル: Ruby

始める前に、Ruby 3.0 以降と `rustorb` gem を用意します（[Ruby へのインストール](../install/ruby.md)）。さらに `rustorb-idl` を `PATH` に置き、ネーミングサービスを起動し、`Echo.idl` を書いておきます（[チュートリアル](index.md)の手順 1〜3）。

## コードを生成する

```sh
rustorb-idl -bruby -C gen Echo.idl
```

`Example::Echo` の Ruby Language Mapping が 2 つのファイルに出力されます。`gen/EchoC.rb` はモジュール `Example::Echo`（`_narrow` とスタブのメソッドを持つ）、`gen/EchoS.rb` はサーバが継承するスケルトンのクラス `POA::Example::Echo` です。

## サーバ

```ruby title="server.rb"
require "corba"
require "CosNamingC"
require "EchoS"

class EchoImpl < POA::Example::Echo
  def initialize(orb)
    super()
    @orb = orb
  end

  def echo(message)
    puts "server: #{message}"
    $stdout.flush
    message
  end

  def shutdown
    @orb.shutdown(false)
  end
end

orb = CORBA.ORB_init(ARGV)
poa = orb.resolve_initial_references("RootPOA")
poa.the_POAManager.activate
echo = EchoImpl.new(orb)._this

root = CosNaming::NamingContext._narrow(orb.resolve_initial_references("NameService"))
root.rebind([CosNaming::NameComponent.new("Echo", "")], echo)
puts "server: ready"
$stdout.flush

orb.run
orb.destroy
```

`_this` はサーバントを RootPOA で活性化し、その参照を返します。

## クライアント

```ruby title="client.rb"
require "corba"
require "CosNamingC"
require "EchoC"

orb = CORBA.ORB_init(ARGV)
shutdown = ARGV.include?("--shutdown")

root = CosNaming::NamingContextExt._narrow(orb.resolve_initial_references("NameService"))
echo = Example::Echo._narrow(root.resolve_str("Echo"))

puts "client: #{echo.echo('hello from Ruby')}"
echo.shutdown if shutdown
orb.destroy
```

## 動かす

```sh
NS="-ORBInitRef NameService=corbaloc::localhost:2809/NameService"
ruby -Igen server.rb $NS &
ruby -Igen client.rb $NS --shutdown
```

```text
server: ready
server: hello from Ruby
client: hello from Ruby
```

要求はバインディングの Ruby スレッドで、GVL を取って動きます。`orb.run` が待っているあいだも、ほかの Ruby スレッドは動き続けます。

## 次に

- ほかの言語のクライアントやサーバと[つないで動かす](together.md)。
- バインディングの全体：[ruby-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/ruby-binding.md)。

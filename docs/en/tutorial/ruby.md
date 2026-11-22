# Tutorial: Ruby

Before you start, have Ruby 3.0 or later with the `rustorb` gem
([Install for Ruby](../install/ruby.md)), `rustorb-idl` on `PATH`, the
naming service running and `Echo.idl` written ([Tutorial](index.md), steps
1–3).

## Generate the code

```sh
rustorb-idl -bruby -C gen Echo.idl
```

This writes the Ruby Language Mapping of `Example::Echo` in two files:
`gen/EchoC.rb`, the module `Example::Echo` with `_narrow` and the stub
methods, and `gen/EchoS.rb`, the skeleton class `POA::Example::Echo` that
a server derives from.

## The server

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

`_this` activates the servant in the RootPOA and gives its reference.

## The client

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

## Run

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

Requests run on Ruby threads of the binding, with the GVL; other Ruby
threads go on running while `orb.run` waits.

## Next

- [Run them together](together.md) with clients and servers in other languages.
- The binding in full: [ruby-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/ruby-binding.md).

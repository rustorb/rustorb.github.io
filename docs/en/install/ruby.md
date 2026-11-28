# Install for Ruby

The Ruby binding is the OMG Ruby Language Mapping 1.2 (`CORBA`,
`PortableServer`, `CosNaming`) on the rustorb library, through a C
extension. It needs **Ruby 3.0 or later** on Linux or macOS, with the Ruby
headers and a C compiler: the gem builds its extension when it is
installed, against a copy of `librustorb_ffi` it carries for Linux (x86_64,
aarch64) and macOS (Apple silicon).

You also need [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0
or later to generate code from IDL.

## Install the gem

```sh
gem install rustorb -v 0.8.0
```

or, with Bundler, `gem "rustorb", "~> 0.8"` in the `Gemfile`. On Debian and
Ubuntu the headers are in `ruby-dev`; `build-essential` has the compiler.
On another platform, build `librustorb_ffi.a` from source (below) and point
the build at it:

```sh
RUSTORB_FFI_LIBRARY=/path/to/librustorb_ffi.a \
RUSTORB_FFI_INCLUDE=/path/to/rustorb/crates/rustorb-ffi/include gem install rustorb
```

## Generate code from IDL

```sh
rustorb-idl -bruby -C gen Echo.idl      # gen/EchoC.rb (types and stubs), gen/EchoS.rb (skeletons)
ruby -Igen server.rb
```

A program does `require "corba"`, then the generated files it uses. The
CosNaming stubs come with the gem (`require "CosNamingC"`).

## From source

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cd ruby && rake compile                 # runs cargo, builds the extension into lib/
ruby -I$PWD/lib -Igen server.rb
```

The mapping is described in
[ruby-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/ruby-binding.md).

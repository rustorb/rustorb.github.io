# Install for .NET

The .NET binding is the IDL4 to C# mapping (OMG formal/21-07-01) on the
rustorb library: `Corba`, `PortableServer`, `CosNaming` and the other
modules, called through P/Invoke. It needs **.NET 8 or later** and comes
as the NuGet package [`Rustorb`](https://www.nuget.org/packages/Rustorb),
which carries the native library for Linux (x64, arm64), macOS (Apple
silicon) and Windows (x64).

You also need [`rustorb-idl`](index.md#the-idl-compiler-rustorb-idl) 0.5.0
or later to generate code from IDL.

## Add the package

```sh
dotnet new console -o EchoServer && cd EchoServer
dotnet add package Rustorb --version 0.7.0
```

The native library is in the package under `runtimes/<rid>/native`, so
`dotnet run` and `dotnet publish -r <rid>` find it. To use another build of
it, set `RUSTORB_FFI_LIBRARY` to the path of the file. It is looked up, in
this order:

1. the environment variable `RUSTORB_FFI_LIBRARY`
2. next to the assembly (`runtimes/<rid>/native` of the package)
3. where the operating system's loader looks (`LD_LIBRARY_PATH`, `PATH`, ...)

## Generate code from IDL

```sh
rustorb-idl -bcsharp -C gen Echo.idl
```

This writes one `.cs` file per IDL file. Add them to the project, for
example with

```xml
<ItemGroup>
  <Compile Include="gen/*.cs" />
</ItemGroup>
```

Generated names keep their IDL spelling (`echo`), as the mapping says;
`-Wbdotnet_naming` gives .NET-style names (PascalCase types and members,
camelCase parameters) instead. The CosNaming types are in the package.

## From source

```sh
git clone https://github.com/rustorb/rustorb && cd rustorb
cargo build --release -p rustorb-ffi       # target/release/librustorb_ffi.so
dotnet build -c Release dotnet/src/Rustorb.Corba
```

Reference `dotnet/src/Rustorb.Corba/Rustorb.Corba.csproj` from your project
and set `RUSTORB_FFI_LIBRARY` to the library. The mapping is described in
[csharp-binding.md](https://github.com/rustorb/rustorb/blob/main/docs/csharp-binding.md).

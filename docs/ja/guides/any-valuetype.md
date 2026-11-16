# Any と valuetype

## Any

`any` は値とその **TypeCode**（型の記述）を一緒に持ちます。受け取る側は、型を前もって知らなくても、値を調べたり、そのまま渡したり、DynAny で分解したりできます。

```python title="any_example.py"
import sys

from rustorb import CORBA
import Demo
from calc_impl import CalcImpl

orb = CORBA.ORB_init(sys.argv)
poa = orb.resolve_initial_references("RootPOA")
poa._get_the_POAManager().activate()
calc = CalcImpl()._this()

# A basic type: the TypeCodes of the basic types are in CORBA.
a = calc.echo_any(CORBA.Any(CORBA.TC_long, 42))
print(a.typecode().kind() == CORBA.tk_long, a.value())    # True 42

# A type from IDL: the generated code has its TypeCode (_tc_<name>).
p = Demo.Point(1, 2)
a = calc.echo_any(CORBA.Any(Demo._tc_Point, p))
print(a.typecode().id(), a.value().x)                     # IDL:Demo/Point:1.0 1

orb.destroy()
```

受け取る側の生成コードにない型の値も届きます。rustorb がその場で TypeCode と、メンバー名の属性を持つクラスを作ります。`orb.create_struct_tc` などの `create_*_tc` で TypeCode を手で組み立てることもできます。

| | Rust | C++ | Java |
| --- | --- | --- | --- |
| 作る | `Any::new(tc, Value::...)` | `a <<= value` | `Helper.insert(a, value)`、`a.insert_long(...)` |
| 読む | `any.value()` | `a >>= value`（型が違えば false） | `Helper.extract(a)`、`a.extract_long()` |
| TypeCode | `TypeCode::struct_tc(...)`、生成コード | `_tc_<名前>`、`CORBA::_tc_long` | `Helper.type()`、`orb.get_primitive_tc(...)` |

## valuetype

**valuetype** は値渡しのオブジェクトです。受け取る側は状態の写しを受け取り、値ファクトリがインスタンスを作ります（操作のない valuetype には生成コードがファクトリを登録します）。struct と違って、valuetype は null にでき、継承でき、共有が保たれます。1 つの要求の中で 2 か所に置いた同じインスタンスは 1 つのインスタンスとして届き、循環した値も循環したまま届きます。

```python
p = Demo.Point(1, 2)
pair = calc.echo_pair(Demo.Pair(p, p))
print(pair.first is pair.second)    # True
```

custom のマーシャル、truncatable、value box、`supports`、abstract interface は、どの言語でも使えます。

### Any の中の valuetype {#valuetypes-in-an-any}

C++ と Java の any は、IDL to C++11 と IDL to Java のマッピングのとおり、valuetype を**参照で**持ちます（0.6.0 から）。

- 同じインスタンスを複数の any に入れても 1 つのインスタンスのままです。DII と DSI でも、生成したスタブ・スケルトンと同じく、要求の引数と戻り値の間で valuetype の共有が保たれます。
- 同じ any から取り出すと、毎回同じインスタンスが返ります。
- any が持つのは、挿入したときのインスタンスの状態です。同じインスタンスを入れ直すと、それを持つすべての any の状態が置き換わります。

Python の any は Python のオブジェクトを持つので、特別な規則なしに共有されます。

詳しくは [any-typecode.md](https://github.com/rustorb/rustorb/blob/main/docs/any-typecode.md)、[dynany.md](https://github.com/rustorb/rustorb/blob/main/docs/dynany.md)、[valuetype.md](https://github.com/rustorb/rustorb/blob/main/docs/valuetype.md) を見てください。

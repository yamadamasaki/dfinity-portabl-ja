# Local objects and classes

In Motoko, an `object` may encapsulate local state (`var`-bound variables) by packaging this state with `public` methods that access and update it.

Motoko では, オブジェクトはローカルな状態 (`var-` 束縛された変数) とそれにアクセスして更新する `public` なメソッドを一つのパッケージに入れて, カプセル化したものです.

As in other typed languages, Motoko programs benefit from the ability to encapsulate state as objects with abstract types.

他の型付け言語同様, Motoko のプログラムでオブジェクトとしてカプセル化できることは抽象型としての恩恵があります.

However, Motoko objects that include mutable state are *not shareable*, and this is a critical security-oriented design decision.

しかし可変状態を持つ Motoko オブジェクトは _共有可能ではありません_. これはセキュリティを指向する重要な設計上の判断です.

If they were shareable, that would mean either conceptually moving a mobile object’s code among actors and executing it remotely, a security risk, or sharing state with remote logic, another security risk. (Notably, as a subcase, objects may be pure records and those *are* shareable, since they are free from mutable state.)

もし共有可能, つまり概念上可搬なオブジェクトのコードをアクタ間で移動でき, 遠隔で実行できるのだとしたら, それはセキュリティ上のリスクになりますし, 遠隔のロジックと状態を共有できるとしても, それはまた別のリスクとなります. (特別な副条件としてオブジェクトが純粋なレコード型ならば, 可変状態をもたないので _共有可能_ になります)

To compensate for this necessary limitation, `actor` objects *are shareable*, but always execute remotely. They communicate with shareable Motoko data only. Local objects interact in less restricted ways with themselves, and can pass any Motoko data to each other’s methods, including other objects. In most other ways, local objects (and classes) are non-shareable counterparts to actor objects (and classes).

この必要な制約を克服するために, `アクタ` ・オブジェクトは _共有可能_ ではあるけれど, 実行は常に遠隔で行うこととします. アクタ・オブジェクトは Motoko の共有可能なデータとのみ通信します。ローカル・オブジェクト同士はより制約の少ない方法で相互作用し, Motoko のデータを (他のオブジェクトを含む) それぞれのオブジェクトのメソッドに渡すことができます. 他のほとんどすべての点で, ローカル・オブジェクト (とクラス) はアクタ・オブジェクト (とクラス) から共有性を取り除いたものと考えられます.

The [Mutable state](mutable-state) introduced declarations of private mutable state, in the form of `var`-bound variables and (mutable) array allocation. In this chapter, we use mutable state to implement simple objects, much like how we would implement simple objects in object-oriented programming.

[可変状態](mutable-state) では,  `var-` 束縛された変数と (可変な) 配列割り付けの形式で, プライベートな可変状態の宣言を導入しました. この章では, 可変状態を用いた簡単なオブジェクトを実装しますが, それはオブジェクト指向プログラミングにおけるオブジェクトの実装とだいたい同じです.

We illustrate this support via a running example, which continues in the next chapter. The following example illustrates a general evolution path for Motoko programs. Each *object*, if important enough, has the potential to be refactored into an Internet *service*, by refactoring this *(local) object* into an *actor object*.

続く章では, 例題を動かしながら説明していきます. その説明では, Motoko のプログラムの進化する過程をお見せします. オブジェクトはどれも (重要性に応じて) インタネット上の _サービス_ に (ローカル・オブジェクトから _アクタ_・オブジェクトへと) リファクタリングされる可能性があります.

**Object classes**. Frequently, one needs *a family* of related objects to perform a task. When objects exhibit similar behavior, it makes sense to fabricate them according to the same blueprint, but with customizable initial state. To this end, Motoko provides a syntactical construct, called a `class` definition, which simplifies building objects of the same type and implementation. We introduce these after discussing objects.

**オブジェクト・クラス**. あるタスクを実施するために複数のオブジェクトを _ファミリ_ としたいことがしばしばあります. 似たような振る舞いをするオブジェクトが複数あるのなら, それらを初期状態が設定可能な一つの青写真から作り出すのには理に適っています. このために Motoko は `class` 定義と呼ばれる, 構文上の構成子を提供しています. これによって同じ型と実装を持つオブジェクトを構築するのが簡単になります. オブジェクトに引き続いてクラスについて説明します.

**Actor classes**. When an object class exposes a *[service](actors-async.md)* (asynchronous behavior), the corresponding Motoko construct is an [actor class](actors-async.md), which follows a similar (but distinct) design.

**アクタ・クラス**. オブジェクト・クラスが *[サービス](actors-async.md)* (非同期的な振る舞い) を公開している場合は, それに相当する Motoko 構成子は  [アクタ・クラス](actors-async.md) で, オブジェクト・クラスに似た (異なる点もあるけど) 設計になっています.

## Example: The `counter` object

Consider the following *object declaration* of the object value `counter`:

以下はオブジェクト値 `counter` のオブジェクト宣言です:

``` motoko
object counter {
  var count = 0;
  public func inc() { count += 1 };
  public func read() : Nat { count };
  public func bump() : Nat {
    inc();
    read()
  };
};
```

This declaration introduces a single object instance named `counter`, whose entire implementation is given above.

この宣言は, `counter` という名前の単一のオブジェクト・インスタンス (上がその完全な実装) を導入します.

In this example, the developer exposes three *public* functions `inc`, `read` and `bump` using keyword `public` to declare each in the object body. The body of the object, like a block expression, consists of a list of declarations.

この例では, 開発者は `public` キーワードを使ってオブジェクト本体の中でそれぞれ宣言した三つの _パブリックな_ 関数 `inc`, `read`, `dump` を公開しています. オブジェクトの本体は, ブロック式と同様, 宣言の列から成ります.

In addition to these three functions, the object has one (private) mutable variable `count`, which holds the current count, initially zero.

この三つの関数に加えて, このオブジェクトには一つの (プライベートな) 可変変数 `count` があります. `count` は現在のカウント値を保持し, 初期値は 0 です.

## Object types

This object `counter` has the following *object type*, written as a list of field-type pairs, enclosed in braces (`{` and `}`):

この `counter` オブジェクトは, フィールドとその型の対のリストを中括弧 `{}` で括った, 次のようなオブジェクト型を持ちます:

``` motoko
{
  inc  : () -> () ;
  read : () -> Nat ;
  bump : () -> Nat ;
}
```

Each field type consists of an identifier, a colon `:`, and a type for the field content. Here, each field is a function, and thus has an *arrow* type form (`_ → _`).

それぞれのフィールド型は, 識別子, コロン `:`, そのフィールドの中身の型から成ります. ここでは各フィールドは関数なので, 射型形式 `(_ → _)` になっています.

In the declaration of `object`, the variable `count` was explicitly declared neither as `public` nor as `private`.

`object`  の宣言中で, `count` 変数は `public` とも `private` とも明示的には宣言されていません.

By default, all declarations in an object block are `private`, as is `count` here. Consequently, the type for `count` does not appear in the type of the object, *and* its name and presence are both inaccessible from the outside.

オブジェクト・ブロックの中での宣言はすべて, デフォルトで `private` になります (ここでの `count` も同様). そのためにこのオブジェクトの型には `count` の型は現れず, その名前にも存在自体にも外部からはアクセスできません.

The inaccessibility of this field comes with a powerful benefit: By not exposing this implementation detail, the object has a *more general* type (fewer fields), and as a result, is interchangeable with objects that implement the same counter object type differently, without using such a field.

このフィールドがアクセス不可であることには強力な利点があります. それは, この実装の詳細を公開しないことで, オブジェクトの型はより汎用的になり (フィールドが少ない), その結果として同じ counter オブジェクト型を (このフィールドを使わないような) 異なる実装で置き換えることができる, ということです.

## Example: The `byteCounter` object

To illustrate the point just above, consider this variation of the `counter` declaration above, of `byteCounter`:

今述べた点を説明するために, 上で宣言した `counter` の変種 `byteCounter` を見てみましょう:

``` motoko
import Nat8 "mo:base/Nat8";
object byteCounter {
  var count : Nat8 = 0;
  public func inc() { count += 1 };
  public func read() : Nat { Nat8.toNat(count) };
  public func bump() : Nat { inc(); read() };
};
```

This object has the same type as the previous one, and thus from the standpoint of type checking, this object is interchangeable with the prior one:

このオブジェクトは前のものと同じ型を持っているので, 型検査の視点からは前のオブジェクトと相互に交換可能です:

``` motoko
{
  inc  : () -> () ;
  read : () -> Nat ;
  bump : () -> Nat ;
}
```

Unlike the first version, however, this version does not use the same implementation of the counter field. Rather than use an ordinary natural `Nat` that never overflows, but may also grow without bound, this version uses a byte-sized natural number (type `Nat8`) whose size is always eight bits.

前のバージョンと違うのは, このバージョンでは count フィールドの実装が違っていることです. 通常の自然数 `Nat` (決して溢れないが, どんどん大きくなっていく) の代わりにこのバージョンではバイト長の自然数 (型は `Nat8`, 大きさは常に 8bit)  を使っています. __NOTE:__ counter フィールドではなく, count フィールドだろう

As such, the `inc` operation may fail with an overflow for this object, but never the prior one, which may instead (eventually) fill the program’s memory, a different kind of application failure.

と言うわけで, このオブジェクトでは `inc` 演算は溢れで失敗するかもしれませんが, 前のオブジェクトでは溢れるようなことはありません (が, 結局プログラムのメモリがいっぱいになるとか, それ以外のアプリケーションの失敗はあり得ます).

Neither implementation of a counter comes without some complexity, but in this case, they share a common type.

どちらのカウンタの実装も大して複雑なものではありませんでしたが, この場合は同じ型を共有しています.

In general, a common type shared among two implementations (of an object or service) affords the potential for the internal implementation complexity to be factored away from the rest of the application that uses it. Here, the common type abstracts over the simple choice of a number’s representation. In general, the implementation choices would each be more complex, and more interesting.

一般的に, オブジェクトやサービスで, 二つの実装が共通の型を共有することは, 実装の内部的な複雑さを, それを利用するアプリケーションの他の部分から取り除くことができる可能性をもたらします. ここでは, 共通する型によって, 数の表現の (簡単な) 選択を抽象化しています. 一般的に, 実装の選択はもっと複雑に, と同時にもっと興味深いものになり得ます.

## Object subtyping

To illustrate the role and use of object subtyping in Motoko, consider implementing a simpler counter with a more general type (fewer public operations):

Motoko に置けるオブジェクトの部分型付けの役割と使い方を説明するために, もっと汎用的な型を持つ簡単なカウンタの実装を考えてみましょう:

``` motoko
object bumpCounter {
  var c = 0;
  public func bump() : Nat {
    c += 1;
    c
  };
};
```

The object `bumpCounter` has the following object type, exposing exactly one operation, `bump`:

この `bumpCounter` オブジェクトでは, 以下のオブジェクト型を持ち, ひとつの演算 `bump` だけを公開しています:

``` motoko
{
  bump : () -> Nat ;
 }
```

This type exposes the most common operation, and one that only permits certain behavior. For instance, the counter can only ever increase, and can never decrease or be set to an arbitrary value.

この型は特定の振る舞いだけを許す, もっともありふれた演算だけを公開しています. 例えばこのカウンタは値を増やすことも減らすことも適当な値に設定することもできません.

In other parts of a system, we may in fact implement and use a *less general* version, with *more* operations:

システムの他の部分では, よりたくさんの演算を持つ, より汎用性の低いバージョンを使って実装をしているかもしれません.

``` motoko
fullCounter : {
  inc   : () -> () ;
  read  : () -> Nat ;
  bump  : () -> Nat ;
  write : Nat -> () ;
}
```

Here, we consider a counter named `fullCounter` with a less general type than any given above. In addition to `inc`, `read` and `bump`, it additionally includes `write`, which permits the caller to change the current count value to an arbitrary one, such as back to `0`.

ここでは, `fullCounter` という名前の今まで見てきたものよりも汎用性の低い型を示しています. `inc`, `read`,  `bump` に加えて, 現在の count の値を任意の値に変更する (`0` に戻すとか) ための `write` もあります.

**Object subtyping.** In Motoko, objects have types that may relate by subtyping, as the various types of counters do above. As is standard, types with *more fields* are *less general* (are ***sub**types* of) types with *fewer fields*. For instance, we can summarize the types given in the examples above as being related in the following subtyping order:

**オブジェクトの部分型付け**. Motoko では, 上のさまざまなカウンタで見てきたとおり, オブジェクトは部分型付けに依って関連付けられる型を持ち得ます. 規範に則り, フィールドが多い型ほどフィールドが少ない型より汎用性が低く, 前者を後者の _**部分**_ 型と呼びます. 例えば, 上の例で与えた型は以下のような部分型に順序づけられます:

-   Most general:
-   いちばん汎用的:

``` motoko
{ bump : () -> Nat }
```

-   Middle generality:
-   その次に汎用的:

``` motoko
{
  inc  : () -> () ;
  read : () -> Nat ;
  bump : () -> Nat ;
}
```

-   Least generality:
-   汎用性がいちばん低い:

``` motoko
{
  inc  : () -> () ;
  read : () -> Nat ;
  bump : () -> Nat ;
  write : Nat -> () ;
}
```

If a function expects to receive an object of the first type (`{ bump: () → Nat }`), *any* of the types given above will suffice, since they are each equal to, or a subtype of, this (most general) type.

ある関数が引数として最初の型 (`{ bump: () → Nat }`) を受け付けるようになっているとしたら, 上で与えたいくつかの型は一番上の (もっとも汎用性の高い) 型と同じか, あるいは部分型になっているので, どの型も受け付けられることになります.

However, if a function expects to receive an object of the last, least general type, the other two will *not* suffice, since they each lack the needed `write` operation, to which this function rightfully expects to have access.

反対に, ある関数が最後のオブジェクト (もっとも汎用性の低い型) を受け付けるようになっていたとしたら, 他の型はこの関数が当然あると思っている `write` 演算を持っていないので, 残り二つはこの関数に受け付けられません.

## Object classes

In Motoko, an object encapsulates state, and an object `class` is a package of two entities that share a common name.

Motoko では, オブジェクトは状態をカプセル化し, オブジェクト・`クラス` は同じ名前を共有する複数の実体をパッケージ化します.

Consider this example `class` for counters that start at zero:

この例の `クラス` は, ゼロから始まるカウンタです:

``` motoko
class Counter() {
  var c = 0;
  public func inc() : Nat {
    c += 1;
    return c;
  }
};
```

The value of this definition is that we can *construct* new counters, each starting with their own unique state, initially at zero:

この定義のいいところは, 新しいカウントを複数個作成し, 初期値 0 からそれぞれが独自の状態をもって開始できることです:

``` motoko
let c1 = Counter();
let c2 = Counter();
```

Each is independent:

それぞれは独立しています:

``` motoko
let x = c1.inc();
let y = c2.inc();
(x, y)
```

We could achieve the same results by writing a function that returns an object:

オブジェクトを返す関数を書けば, 同じ結果が得られます:

``` motoko
func Counter() : { inc : () -> Nat } =
  object {
    var c = 0;
    public func inc() : Nat { c += 1; c }
  };
```

Notice the return type of this *constructor function* (an object type):

この構成子関数が返す型 (オブジェクト型) に注意してください:

``` motoko
{ inc : () -> Nat }
```

We may want to name this type, for example, `Counter`, as follows, for use in further type declarations:

この型に名前を付けて, 将来的に型宣言で使えるようにするとしたら, 例えば以下のように `Counter` とするでしょうか:

``` motoko
type Counter = { inc : () -> Nat };
```

In fact, the `class` keyword syntax shown above is nothing but a shorthand for these two definitions for `Counter`: a factory function `Counter` that constructs objects, and the type `Counter` of these objects. Classes do not provide any new functionality beyond this convenience.

実際, 上で示した `class` キーワードは, 単にこれら二つの `Counter` 定義の省略形, つまりオブジェクトとその型 `Counter` を作成するファクトリ関数なのです. クラスは, この手軽さ以上の何か新しい機能性を提供しているわけではありません.

### Class constructor

An object class defines a constructor function that may carry zero or more data arguments and zero or more type arguments.

オブジェクト・クラスは 0 個以上のデータ引数と 0 個以上の型引数を取る構成子関数を定義します.

The `Counter` example above has zero of each.

上の `Counter` の例では, どちらの数も 0 です.

The type arguments, if any, parameterize both the type and the constructor function for the class.

型引数がもしあれば, そのクラスの型と構成子関数をパラメタ化します.

The data arguments, if any, parameterize (only) the constructor function for the class.

データ引数がもしあれば, そのクラスの構成子関数 (だけ) をパラメタ化します.

#### Data arguments

Suppose we want to initialize the counter with some non-zero value. We can supply that value as a data argument to the `class` constructor:

ここでは, 0 以外の数で初期化するカウンタが欲しいとしましょう. その値をデータ引数として `class` の構成子に与えることができます:

``` motoko
class Counter(init : Nat) {
  var c = init;
  public func inc() : Nat { c += 1; c };
};
```

This parameter is available to all methods.

このパラメタはすべてのメソッドで参照できます.

For instance, we can `reset` the `Counter` to its initial value, a parameter:

例えば, `Counter` をその初期値 (パラメタの値) に `reset` できます:

``` motoko
class Counter(init : Nat) {
  var c = init;
  public func inc() : Nat { c += 1; c };
  public func reset() { c := init };
};
```

#### Type arguments

Suppose we want the counter to actually carry data that it counts (like a specialized `Buffer`).

ここでは, 実際にカウントするデータを担う (特殊な `Buffer` とか) カウンタが欲しいとしましょう.

When classes use or contain data of arbitrary type, they carry a type argument (or equivalently, *type parameter*) for that unknown type, just as with functions.

クラスが任意で型のデータを使ったり, 持ったりするときには, その未だ何になるか分からない型を型引数 (同じことだけど, _型パラメタ_ とも言う) に入れます.

The scope of this type parameter covers the entire `class`, just as with data parameters. As such, the methods of the class can use (and *need not re-introduce*) these type parameters.

この型パラメタのスコープは, その `class` 全体です (データ・パラメタと同じ). 同様にそのクラスのメソッドは, その型パラメタを (再導入することなく) 使えます.

``` motoko
import Buffer "mo:base/Buffer";

class Counter<X>(init : Buffer.Buffer<X>) {
  var buffer = init.clone();
  public func add(x : X) : Nat {
    buffer.add(x);
    buffer.size()
  };

  public func reset() {
    buffer := init.clone()
  };
};
```

#### Type annotation

Optionally, the class constructor may also carry a type annotation for its "return type" (the type of objects that it produces). When supplied, Motoko checks that this type annotation is compatible with the body of the class (an object definition). This check ensures that each object produced by the constructor meets the supplied specification.

オプションとして, クラス構成子では "返値型" (作成するオブジェクトの型) を表す型注釈を付けることができます. これを付けると, Motoko はこの型注釈がクラス本体 (オブジェクト定義) と互換性があるかどうかを検査します. この検査によって, その構成子が作成するオブジェクトが与えられた仕様を満たしていることが保証されます.

For example, we repeat the `Counter` as a buffer, and annotate it with a more general type `Accum<X>` that permits adding, but not resetting the counter. This annotation ensures that the objects are compatible with the type `Accum<X>`.

例えば, 上と同じくバッファとしての `Counter` を考えて, それをより汎用的な型 `Accum<X>` (加えることはできるけど, リセットはできない) で注釈付けしましょう. この注釈は作成されるオブジェクトが, 型 `Accum<X>` と互換であることを保証します.

``` motoko
import Buffer "mo:base/Buffer";

type Accum<X> = { add : X -> Nat };

class Counter<X>(init : Buffer.Buffer<X>) : Accum<X> {
  var buffer = init.clone();
  public func add(x : X) : Nat { buffer.add(x); buffer.size() };
  public func reset() { buffer := init.clone() };
};
```

#### Full syntax

In full, classes are defined by the keyword `class`, followed by: - a name for the constructor and type being defined (for example, `Counter`) - optional type arguments (for example, omitted, or `<X>`, or `<X, Y>`) - an argument list (for example, `()`, or `(init : Nat)`, etc.) - an optional type annotation for the constructed objects (for example, omitted, or `Accum<X>`), - the class "body" is an object definition, parameterized by the type and value arguments (if any).

完全に言うと, 

- クラスとは `class` キーワードによって定義され,
- それに続いて構成子の名前と定義される型 (例えば `Counter`),
- オプションの型引数 (例えば, なし, `<x>`, `<x, y>`),
- 引数リスト (例えば `()`, `(init: Nat)`, など),
- 作成されるオブジェクトに対するオプションの型注釈 (例えば, なし, `Accum<X>`),
- クラスの「本体」 ((もしあれば型引数, データ引数によってパラメタ化された) オブジェクト定義)が並ぶ

ものです.

The constituents of the body marked `public` contribute to the resulting objects' type and these types compared against the (optional) annotation, if given.

`public`  の付いた本体の構成要素は, 作られるオブジェクトの型に寄与し, (もしあればオプションの) 注釈と対照比較されます.

##### Another example: `Bits`

As another example, let’s consider the task of walking the bits of a natural number (type `Nat`). For this example, we could define the following:

別の例として, 自然数 (型は `Nat`) のビットを「歩く」タスク (next() を送るたびに, 最下位ビットが 1 か (true) どうかを答えて, 右にシフトする. 設けたが残っていなければ null を返す) を考えてみましょう.  この例では, 次のように定義してみました:

``` motoko
class Bits(n : Nat) {
  var state = n;
  public func next() : ?Bool {
    if (state == 0) { return null };
    let prev = state;
    state /= 2;
    ?(state * 2 != prev)
  }
}
```

The above class definition is equivalent to the simultaneous definition of a structural type synonym and a factory function, both named `Bits`:

上記のクラス定義は, 構造型の名付けとファクトリ関数を同時に定義したものと同値です:

``` motoko
type Bits = {next : () -> ?Bool}
let Bits : Nat -> Bits =
func Bits(n : Nat) : Bits = object {
  // class body
};
```

## Structural subtyping

Object subtyping in Motoko uses *structural subtyping*, not *nominal subtyping*.

Motoko でのオブジェクトの部分型付けは _構造に基づく部分型付け_ で _名前による部分型付け_ ではありません.

Recall that in nominal typing, the question of two types equality depends on choosing consistent, globally-unique type names (across projects and time).

名前による型付けでは, 二つの型の同等性は矛盾のない, 大域的に (プロジェクトと時間を跨いで) 一意な型名に依存しているのでした.

In Motoko, the question of two types' equality is based on their *structure*, not their names.

Motoko では, 二つの型の同等性は名前ではなく, その _構造_ に基づいて判定されます.

Due to structural typing, naming the class type provides a convenient abbreviation.

構造的型付けのお陰で, クラス型の名前付けは便利な短縮形になります.

For typing purposes, however, all that matters is the *structure* of the corresponding object type: two classes with different names but equivalent definitions produce type-compatible objects.

型付けのために重要なのはすべて対応するオブジェクト型の _構造_ です. 名前は異なるけれど, 定義が同等な二つのクラスは, 型互換性のあるオブジェクトを作成します.

When the optional type annotation is supplied in a class declaration, conformance is checked: the object type must be a subtype of the annotation. The annotation does not affect the type of the class, however, even if it only describes a proper super-type of the object type.

クラス宣言にオプショナルな型注釈が付いている場合には, 適合性が検査されます. したがって, オブジェクト型は注釈の部分型でなければなりません. とは言え, 注釈は (たとえ, そのオブジェクト型の正当な上位型を記述しているだけだとしても) クラス自身の型には影響しません.

Formally, subtyping relationships in Motoko extend to all types, not just object types.

形式的には, Motoko における部分型関係は, オブジェクト型だけではなく, すべての型に拡張されます.

Most cases are standard, and follow conventional programming language theory (for *structural* subtyping, specifically).

ほとんどの場合は規範的なもので, 従来のプログラミング言語理論 (特に構造的部分型付けの) に従っています.

Other notable cases in Motoko for new programmers include array, options, variants and number type inter-relationships.

新規のプログラマにとって, Motoko で他に気を付けないといけないのは, 配列, オプション, ヴァリアント, 数値型の間の相互関係などです.

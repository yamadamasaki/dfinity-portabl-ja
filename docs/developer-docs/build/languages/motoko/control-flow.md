# Imperative control flow

There are two key categories of control flow:

制御フローには, 二つの重要なカテゴリがあります:

-   *declarative*, when the structure of some value guides control and the selection of the next expression to evaluate, like in `if` and `switch` expressions;

-   *imperative* where control changes abruptly according to a programmer’s command, abondoning regular control flow; examples are `break` and `continue`, but also `return` and `throw`.



-   _宣言的_, その構造の何かの値が制御と次に評価する式の選択を導く. 例えば `if` 式, `switch` 式など
-   _手続き的_, 正規の制御フロープロを放棄して, プログラマの唐突な指令にしたがって制御が変わる. 例えば `break`, `continue` など. `return` と `throw` も.

Imperative control flow often goes hand-in-hand with state changes and other flavors of side-effects, such as error handling and input/output.

手続き的な制御フローはしばしば状態変化やエラー処理や I/O など副作用の匂いと密接に絡み合っています.

## Early `return` from `func`

Normally, the result of a function is the value of its body. Sometimes, during evaluation of the body, the result is available before the end of evaluation. In such situations the `return <exp>` construct can be used to abandon the rest of the computation and immediately exit the function with a result. Similarly, where permitted, `throw` may be used to abandon a computation with an error.

通常, 関数の結果はその本体の値です. たまに本体の評価中, 評価を完了する前に結果が得られることがあります. このような状況では, 計算の残りを放棄して, その結果を持って直ちに関数を抜けるのに, `return (exp)` 構成子を使えます. 許される場所では, 同様に `throw` がエラーを持って計算を放棄するのに使われます.

When a function has unit result type, the shorthand `return` may be used instead of the equivalent `return ()`.

関数で返値が単位型のときには, 省略して `reutrn` を使うことができますが, これは `return ()` と同値です.

## Loops and labels

Motoko provides several kinds of repetition constructs, including:

Motoko では何種類かの繰り返し構成子を

-   `for` expressions for iterating over members of structured data.

-   `loop` expressions for programmatic repetition (optionally with termination condition).

-   `while` loops for programmatic repetition with entry condition.



-   `for` 式は構造化データのメンバ上を繰り返す
-   `loop` 式はプログラム上の繰り返し (終了条件はオプション)

-   `while` は入場条件のあるプログラム上の繰り返し

Any of these can be prefixed with a `label <name>` qualifier to give the loop a symbolic name. Named loops are useful for imperatively changing control flow to continue from the entry or exit of the named loop.

これらはどれも `label (name)` 限定子を前置して, ループにシンボル名を付けることができます. 名前の付いたループは制御フローを変更して入り口から続けるとか, 抜けるとかには便利です.

-   re-entering the loop with `continue <name>`, or

-   exiting the loop altogether with `break <name>`.



-   `continue <name>` でループに再突入する
-   `break <name>` でループ全体を抜ける

In the following example, the `for` expression loops over characters of some text and abandons iteration as soon as an exclamation sign is encountered.

下の例は, あるテキストの文字上をループする `for` 式で, 感嘆符にぶつかったら即繰り返しを中断します.

``` motoko
import Debug "mo:base/Debug";
label letters for (c in "ran!!dom".chars()) {
  Debug.print(debug_show(c));
  if (c == '!') { break letters };
  // ...
}
```

### Labeled expressions

There are two other facets to `label`​s that are less mainstream, but come in handy in certain situations:

それほどよく使われるわけではありませんが, 特定の状況では便利な `label` の使い途がもう二つあります.

-   `label`​s can be typed

-   *any* expression (not just loops) can be named by prefixing it with a label; `break` allows one to short-circuit the expression’s evaluation by providing an immediate value for its result. (This is similar to exiting a function early using `return`, but without the overhead of declaring and calling a function.)



-   `label` は型付けできる
-   (ループに限らず) 任意の式もラベルを前置することで名前を付けることができる; `break` すると式の評価を短絡して, その結果を即値で与えられる (これは関数を `reutrn` で途中で抜けるのに似ている. しかし, 宣言と関数呼び出しのオーバヘッドはない) 

The syntax for type-annotated labels is `label <name> : <type> <expr>`, signifying that any expression can be exited using a `break <name> <alt-expr>` construct that returns the value of `<alt-expr>` as the value of `<expr>`, short-circuiting evaluation of `<expr>`.

型注釈付きラベルの構文は `label <name> : <type> <expr>` で,  `<expr>` の値として  `<alt-expr>` の値を返す `break <name> <alt-expr>` 構成子を使って任意の式から抜け出すことを意味しています.

Judicious use of these constructs allows the programmer to focus on the primary program logic and handle exceptional case via `break`

これらの構成子を賢く使えば, プログラマはもっとも重要なプログラム・ロジックに焦点を当てて, 例外的な状況は `break` を使って処理する, と言うようなことができます.

``` motoko
import Text "mo:base/Text";
import Iter "mo:base/Iter";

type Host = Text;
let formInput = "us@dfn";

let address = label exit : ?(Text, Host) {
  let splitted = Text.split(formInput, #char '@');
  let array = Iter.toArray<Text>(splitted);
  if (array.size() != 2) { break exit(null) };
  let account = array[0];
  let host = array[1];
  // if (not (parseHost(host))) { break exit(null) };
  ?(account, host)
}
```

Naturally, labeled common expressions don’t allow `continue`. In terms of typing, both `<expr>` and `<alt-expr>`​'s types must conform with the label’s declared `<type>`. If a label is only given a `<name>`, then its `<type>` defaults to unit (`()`). Similarly a `break` without an `<alt-expr>` is shorthand for the value unit (`()`).

当然, 普通のラベル付きの式は `continue` はできません. 型付けの面から言って, `<expr>` と `<akt-expr>` の型はラベルで宣言した `<type>` を充たさなければなりません. ラベルが `<name>` だけ与えられた場合にはその `<type>` はデフォルトで単位型 (`()`) になります. 同様に `<alt-expr>` のない `break` は単位値 (`()`) の省略形です.

## Option blocks and null breaks

Like many other high-level languages, Motoko lets you opt in to `null` values, tracking possible occurrences of `null` values using option types of the form `?T`. This is to both to encourage you to avoid using `null` values when possible, and to consider the possibility of `null` values when necessary.

他の高級言語と同様, Motoko では `null` 値が起こりうる可能性を `?T` のかたちのオプション型を使って追跡しながら, `null` 値の使用を許すことができます. それと同時に, 可能ならば `null` 値の使用を避け, 必要なときだけ `null` 値の可能性を検討することを推奨するものでもあります.

The latter could be cumbersome, if the only way to test a value for `null` were with a verbose `switch` expression, but Motoko simplifies the handling of option types with some dedicated syntax: *option blocks* and *null breaks*.

もし値が `null` かどうか検査する方法が回りくどい `switch` 式だけだったら, 後者は面倒くさいな, と思うかもしれませんが, Motoko ではオプション型の扱いを簡略化するために, 専用の構文 `オプション・ブロック` と `null ブレーク` を用意しています.

The option block, `do ? <block>`, produces a value of type `?T`, when block `<block>` has type `T` and, importantly, introduces the possibility of a break from `<block>`. Within a `do ? <block>`, the null break `<exp> !`, tests whether the result of the expression, `<exp>`, of unrelated option type, `?U`, is `null`. If the result `<exp>` is `null`, control immediately exits the `do ? <block>` with value `null`. Otherwise, the result of `<exp>` must be an option value `?v`, and evaluation of `<exp> !` proceeds with its contents, `v` (of type `U`).

オプション・ブロック `do ? <block>` はブロック `<block>` が型 `T` を持ち,  `<block>` から `break` する可能性があるとき (これが重要) に型 `?T` の値を作成します. __TODO__

As a realistic example, we give the definition of a simple function evaluating numeric Expressions built from natural numbers, division and a zero test, encoded as a variant type:

現実味のある例として, 自然数, 除算, ゼロ検査から成るヴァリアント型として作成された数値式を評価する簡単な関数の定義を見てもらいましょう:

``` motoko
type Exp = {
  #Lit : Nat;
  #Div : (Exp, Exp);
  #IfZero : (Exp, Exp, Exp);
};

func eval(e : Exp) : ? Nat {
  do ? {
    switch e {
      case (#Lit n) { n };
      case (#Div (e1, e2)) {
        let v1 = eval e1 !;
        let v2 = eval e2 !;
        if (v2 == 0)
          null !
        else v1 / v2
      };
      case (#IfZero (e1, e2, e3)) {
        if (eval e1 ! == 0)
          eval e2 !
        else
          eval e3 !
      };
    };
  };
}
```

To guard against division by `0` without trapping, the `eval` function returns an option result, using `null` to indicate failure.

`0` 除算でトラップしないようにするために, `eval` 関数はオプション型の結果 (`null` は失敗を示す) を返します.

Each recursive call is checked for `null` using `!`, immediately exiting the outer `do ? block`, and thus the function itself, with `null`, when a result is `null`.

再帰的な呼び出しではそれぞれ `!` を使って `null` かどうかをチェックし, すぐに外側の `do ? block` を出ますが, 結果が `null` のときには関数自体が `null` になります. 

(As an exercise that illustrates the concision of option blocks, you might want to try rewriting `eval` using a labeled expression and explicit switches for each null break.)

(オプション・ブロックの簡潔さを納得するための演習として, null ブレークをラベル付き式と明示的な switch で `eval` を書き直してみるのもいいでしょう) 

## Repetition with `loop`

The simplest way to indefinitely repeat a sequence of imperative expressions is by using a `loop` construct

手続き的な式を無限に繰り返す, 最も簡単なやり方は, `loop` 構成子を使うことです

``` motoko
loop { <expr1>; <expr2>; ... }
```

The loop can only be abandoned with a `return` or `break` construct.

このループは, `return` 構成子と `break` 構成子を使って脱出できます.

A re-entry condition can be affixed to allow a conditional repetition of the loop with `loop <body> while <cond>`.

再突入条件を付ければ, `loop <body> while <cond>` で条件付き繰り返しのループを作ることができます.

The body of such a loop is always executed at least once.

このようなループの本体は, 必ず少なくとも一度は実行されます.

## `while` loops with precondition

Sometimes an entry condition is needed to guard the first execution of a loop. For this kind of repetition the `while <cond> <body>`-flavor is available

突入条件を付けて, ループの最初の実行を防がなければならない場合もあるでしょう. このような繰り返しとして `while <cond> <body>` 風味もあります.

``` motoko
while (earned < need) { earned += earn() };
```

Unlike a `loop`, the body of a `while` loop may never be executed.

`loop` とは違って, `while` ループはまったく実行されない場合もあります.

## `for` loops for iteration

An iteration over elements of some homogeneous collection can be performed using a `for` loop. The values are drawn from an iterator and bound to the loop pattern in turn.

内容が同質な要素の集まりの上の繰り返しは `for` ループで実行できます. 値は繰り返し子から取られ, ループ・パタンで順に束縛されます.

``` motoko
let carsInStock = [
  ("Buick", 2020, 23.000),
  ("Toyota", 2019, 17.500),
  ("Audi", 2020, 34.900)
];
var inventory : { var value : Float } = { var value = 0.0 };
for ((model, year, price) in carsInStock.vals()) {
  inventory.value += price;
};
inventory
```

## Using `range` with a `for` loop

The `range` function produces an iterator (of type `Iter<Nat>`) with the given lower and upper bound, inclusive.

`range` 関数は与えられた下限/上限 (境界を含む) 間の繰り返し子 (型は `Iter<Nat>`) を作成します.

The following loop example prints the numbers `0` through `10` over its *eleven* iterations:

``` motoko
import Iter "mo:base/Iter";
import Debug "mo:base/Debug";
var i = 0;
for (j in Iter.range(0, 10)) {
  Debug.print(debug_show(j));
  assert(j == i);
  i += 1;
};
assert(i == 11);
```

More generally, the function `range` is a `class` that constructs iterators over sequences of natural numbers. Each such iterator has type `Iter<Nat>`.

より一般的には, 関数 `range` は自然数の列の上を走る繰り返し子を作成する `クラス` です.

As a constructor function, `range` has a function type:

作成関数として, `range` は下の関数型を持ちます:

``` motoko
(lower : Nat, upper : Int) -> Iter<Nat>
```

Where `Iter<Nat>` is an iterator object type with a `next` method that produces optional elements, each of type `?Nat`:

`Iter<Nat>` をオプショナルな要素を生成する `next` メソッドを持つオブジェクト型だとすると:

``` motoko
type Iter<A> = {next : () -> ?A};
```

For each invocation, `next` returns an optional element (of type `?Nat`).

`next` を起動するたびに (型 `?Nat` をもつ) オプショナルな要素を返します.

The value `null` indicates that the iteration sequence has terminated.

値 `null` は繰り返し列が終了したことを示します.

Until reaching `null`, each non-`null` value, of the form `?`*n* for some number *n*, contains the next successive element in the iteration sequence.

`null` に到達するまで, ある数値 _n_ に対する `?`_n_ 形式の非 `null` 値毎に繰り返し対象の列のつぎの後続要素を返します.

## Using `revRange`

Like `range`, the function `revRange` is a `class` that constructs iterators (each of type `Iter<Int>`). As a constructor function, it has a function type:

`range` と同じく, 関数 `revRange` は繰り返し子 (型は `Iter<Int>`) を作成する `クラス` です.構成子として, 以下の関数型を持ちます:

``` motoko
(upper : Int, lower : Int) -> Iter<Int>
```

Unlike `range`, the `revRange` function *descends* in its iteration sequence, from an initial *upper* bound to a final *lower* bound.

`range` と異なり, `revRange` 関数は繰り返し列の _上限_ から始めて _下限_ に向かって減らしていきます.

## Using iterators of specific data structures

Many built-in data structures come with pre-defined iterators. Below table lists them

あらかじめ定義された繰り返し子が, たくさんの組込みデータ構造にあります.

| Type      | Name                  | Iterator | Elements                  | Element type |
|-----------|-----------------------|----------|---------------------------|--------------|
| `[T]`     | array of `T`​s         | `vals`   | the array’s members       | `T`          |
| `[T]`     | array of `T`​s         | `keys`   | the array’s valid indices | `Nat`        |
| `[var T]` | mutable array of `T`​s | `vals`   | the array’s members       | `T`          |
| `[var T]` | mutable array of `T`​s | `keys`   | the array’s valid indices | `Nat`        |
| `Text`    | text                  | `chars`  | the text’s characters     | `Char`       |
| `Blob`    | blob                  | `vals`   | the blob’s bytes          | `Nat8`       |

Iterators for data structures

User-defined data structures can define their own iterators. As long they conform with the `Iter<A>` type for some element type `A`, these behave like the built-in ones and can be consumed with ordinary `for`-loops.

ユーザが定義したデータ構造にもそれ用の繰り返し子を定義することができます. 何らかの型 `A` に対する型 `Iter<A>` を充たしてさえいれば, 組込みの繰り返し子と同様に振る舞うので, 通常の `for` ループで使うことができます.

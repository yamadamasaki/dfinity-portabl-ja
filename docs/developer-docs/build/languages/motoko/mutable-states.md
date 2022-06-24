# Mutable states

Each actor in Motoko may use, but may *never directly share*, internal mutable state.

Motoko の各アクタは内部的に可変な状態を使いますが, それを _直接共有する_ ことは決してありません.

Later, we discuss [sharing among actors](sharing), where actors send and receive *immutable* data, and also handles to each others external entry points, which serve as *shareable functions*. Unlike those cases of shareable data, a key Motoko design invariant is that ***mutable data** is kept internal (private) to the actor that allocates it, and **is never shared remotely***.

後ほど [アクタ間での共有](sharing) を議論しますが, アクタは不変データを送受し, 互いの外部のエントリ・ポイント (それが _共有可能関数_ として働く) を扱います. 共有可能なデータを扱う他の多くの場合とは異なり, Motoko の設計の変わらぬ重要点は _**可変なデータ** はそのデータを割り付けたアクタの内部に留め置かれ (プライベート), **決して遠隔から共有されることはない **_ ということです.

In this chapter, we continue using minimal examples to show how to introduce (private) actor state, and use mutation operations to change it over time.

この章では最小限の例題を使って, アクタの (プライベートな) 状態をどうやって取り入れるか, 時間が経つにつれて変化させる変更演算をどう使うかを示します.

In [local objects and classes](local-objects-classes), we introduce the syntax for local objects, and a minimal `counter` actor with a single mutable variable. In the [following chapter](actors-async), we show an actor with the same behavior, exposing the counter variable indirectly behind an associated service interface for using it remotely.

[ローカルなオブジェクトとクラス](local-objects-classes) では, ローカル・オブジェクトの構文と, ひとつの可変変数を持つだけの最小の `counter` アクタを紹介します. [続く章](actors-async) では, 同じ振る舞いを持ちながら, 対応するサービス・インタフェイス越しに遠隔から間接的に counter 変数にアクセスできるようにします.

## Immutable versus mutable variables

The `var` syntax declares mutable variables in a declaration block:

宣言ブロックの `var` 構文は, 可変な変数を宣言します:

``` motoko
let text  : Text = "abc";
let num  : Nat = 30;

var pair : (Text, Nat) = (text, num);
var text2 : Text = text;
```

The declaration list above declares four variables. The first two variables (`text` and `num`) are lexically-scoped, *immutable variables*. The final two variables (`pair` and `text2`) are lexically-scoped, ***mutable*** variables.

上の宣言リストでは, 四つの変数を宣言しています. 最初の二つの変数 (`text` と `num`) は静的スコープで, 不変な変数です. 後の二つの変数 (`pair` と `text2`) は静的スコープですが, **可変** な変数です.

## Assignment to mutable memory

Mutable variables permit assignment, and immutable variables do not.

可変変数には代入ができますが, 不変変数には代入はできません.

If we try to assign new values to either `text` or `num` above, we will get static type errors; these variables are immutable.

上で `text` と `num` の両方に新しい値を代入しようとすると, 「この変数は不変である」であるという静的な型エラーになります.

However, we may freely update the value of mutable variables `pair` and `text2` using the syntax for assignment, written as `:=`, as follows:

でも, 可変変数である `pair` と `text2` に下で `:=` と書いたような代入構文で値を代入して変更するのは問題ありません:

``` motoko
text2 := text2 # "xyz";
pair := (text2, pair.1);
pair
```

Above, we update each variable based on applying a simple “update rule” to their current values (for example, we *update* `text2` by appending string constant `"xyz"` to its suffix). Likewise, an actor processes some calls by performing *updates* on its internal (private) mutable variables, using the same assignment syntax as above.

上では, 各変数を現在の値に対する単純な「更新規則」を適用して変更しました (例えば `text2` には最後に文字列定数 `"xyz"` を付け加える). 同様に, アクタも上と同じ代入構文を使って, 内部の (プライベートな) 可変変数を更新して, 何らかの呼び出しを処理します.

### Special assignment operations

The assignment operation `:=` is general, and works for all types.

代入演算子 `:=` は汎用的で, どんな型にも使えます.

Motoko also includes special assignment operations that combine assignment with a binary operation. The assigned value uses the binary operation on a given operand and the current contents of the assigned variable.

Motoko にはそれ以外に代入と二項演算を組み合わせた, 特殊な代入演算があります. 代入された値は, 代入先の変数の現在の値と, 与えられた被演算子に二項演算を施したものになります.

For example, numbers permit a combination of assignment and addition:

例えば, 数値は代入と加算を組み合わせることができます:

``` motoko
var num2 = 2;
num2 += 40;
num2
```

After the second line, the variable `num2` holds `42`, as one would expect.

二行目の実行後, 変数 `num2` は (ご期待通り) `42` になります.

Motoko includes other combinations as well. For example, we can rewrite the line above that updates `text2` more concisely as:

Motoko では他の組合せでも大丈夫です. 例えば上で `text2` の更新をもっと簡潔に書くことができます:

``` motoko
text2 #= "xyz";
text2
```

As with `+=`, this combined form avoids repeating the assigned variable’s name on the right hand side of the (special) assignment operator `#=`.

`+=` と同じですが, この組合せ形式では (特殊な) 代入演算子 `#=` の左辺の代入される変数名を二度書かずに済みます. __NOTE:__ 原文は「右辺」になっているが, 「左辺」のはず

The [full list of assignment operations](language-manual#syntax-ops-assignment) lists numerical, logical, and textual operations over appropriate types (number, boolean and text values, respectively).

[代入演算の全一覧](language-manual#syntax-ops-assignment) には, それぞれ適切な型 (数値, ブール値, テキスト値) に対応する数値演算, 論理演算, テキスト演算が載っています.

## Reading from mutable memory

When we updated each variable, we also first *read* from the mutable contents, with no special syntax.

各変数を更新するとき, まず最初にその変更対象の内容を読み出します (特別な構文はありません).

This illustrates a subtle point: Each use of a mutable variable *looks like* the use of an immutable variable, but does not *act like* one. In fact, its meaning is more complex. As in many languages (JavaScript, Java, C#, etc.), but not all, the syntax of each use hides the *memory effect* that accesses the memory cell identified by that variable, and gets its current value. Other languages from functional traditions (SML, OCaml, Haskell, etc), generally expose these effects syntactically.

ここには微妙な点があります. 可変変数の利用は不変変数の利用と _似ている_ が, 振る舞いは異なっている, ということです. 実際にはその意味はもっと込み入っています. 多く (とは言えすべてではない) の言語 (JavaScript, Java, C#, など) と同様に, 変数利用の構文はその変数によって指定されるメモリ・セルにアクセスして現在の値を取得するという __メモリ作用__ を隠してしまっているのです. 関数型の伝統に連なる他の言語 (SML, OCaml, Haskell, など) では, 一般的にこれらの作用を構文上も明らかにしています.

Below, we explore this point in detail.

この点の詳細を以下で見てみましょう.

## Understanding `var`- versus `let`-bound variables

Consider the following two variable declarations, which look similar:

次の二つの変数宣言は同じように見えます:

``` motoko
let x : Nat = 0
```

and:

``` motoko
var x : Nat = 0
```

The only difference in their syntax is the use of keyword `let` versus `var` to define the variable `x`, which in each case the program initializes to `0`.

構文上の違いと言えば, 変数 `x` の定義に使っているのが `let` キーワードと `var` キーワードというだけで, どちらもそれを `0` に初期化しています.

However, these programs carry different meanings, and in the context of larger programs, the difference in meanings will impact the meaning of each occurrence of `x`.

しかし, この二つのプログラムの意味は違うものです. より上位のプログラムの文脈では, この意味の相違は `x` の出現の意味に影響を与えます.

For the first program, which uses `let`, each such occurrence *means* `0`. Replacing each occurrence with `0` will not change the meaning of the program.

`let` を使っている最初のプログラムでは, どこに出現してもそれは `0` です. 出現場所をすべて `0` で置き換えたとしてもプログラムの意味は変わりません.

For the second program, which uses `var`, each occurrence *means*: “read and produce the current value of the mutable memory cell named `x`.” In this case, each occurrence’s value is determined by dynamic state: the contents of the mutable memory cell named `x`.

`var` を使っている二番目のプログラムでは, その出現は「`x` という名前の可変メモリ・セルを読んで現在値を作成する」ということを意味しています. この場合, 出現場所での値は動的な状態, つまり `x` という名前の可変メモリ・セルの内容によって決まります.

As one can see from the definitions above, there is a fundamental contrast between the meanings of `let`-bound and `var`-bound variables.

上の定義から分かるように, `let-` 束縛変数と `var- `束縛変数の意味には根本的な対照性があります.

In large programs, both kinds of variables can be useful, and neither kind serves as a good replacement for the other.

大きなプログラムでは, どちらの種類の変数も役に立ちますが, 入れ替えて使うことはできません.

However, `let`-bound variables *are* more fundamental.

しかし, より根本的な概念は `let-` 束縛変数です.

To see why, consider encoding a `var`-bound variable using a one-element, mutable array, itself bound using a `let`-bound variable.

なぜなら `var-` 束縛変数はそれ自身を `let-` 束縛変数に束縛された一要素の可変配列だと考えることができるからです.

For instance, instead of declaring `x` as a mutable variable initially holding `0`, we could instead use `y`, an immutable variable that denotes a mutable array with one entry, holding `0`:

例えば `x` を最初に 0 をもっている可変変数として宣言する代わりに, 値 `0` をもつ一つの要素だけからなる可変配列を表す不変変数 `y` を考えてみましょう:

``` motoko
var x : Nat       = 0 ;
let y : [var Nat] = [var 0] ;
```

We explain mutable arrays in more detail [below](#mutable-arrays).

可変配列についての詳細は [下](#mutable-arrays) に書きます.

Unfortunately, the read and write syntax required for this encoding reuses that of mutable arrays, which is not as readable as that of `var`-bound variables. As such, the reads and writes of variable `x` will be easier to read than those of variable `y`.

残念ながら, このようにコード化した読み書きの構文は可変配列のものを使うことになるので, `var-` で束縛された変数のようには読み易くはありません. 変数 `x` の読み書きは変数 `y`  よりは易しいでしょう.

For this practical reason, and others, `var`-bound variables are a core aspect of the language design.

このような実用上の理由などから, `let-` で束縛された変数がこの言語設計の核となる観点なのです. __NOTE:__ `var-` になっているけど `let-` だと思うのだが. それにこの論法で行くと, [var Nat] の方はどうなるのだ?

## Immutable arrays

Before discussing [mutable arrays](#mutable-arrays), we introduce immutable arrays, which share the same projection syntax, but do not permit mutable updates (assignments) after allocation.

[可変配列](#mutable-arrays) について話を進める前に, 射影構文は同じですが, いったん割り付けた後は変更更新 (代入) ができない, 不変配列というものを導入しましょう.

### Allocate an immutable array of constants

``` motoko
let a : [Nat] = [1, 2, 3] ;
```

The array `a` above holds three natural numbers, and has type `[Nat]`. In general, the type of an immutable array is `[_]`, using square brackets around the type of the array’s elements, which must share a single common type, in this case `Nat`.

上の `a` という配列には三つの自然数が入っており, 型は `[Nat]` です. 一般的に不変配列の型は `[_]` のように表現し, 配列の要素の型 (この場合は `Nat` ですが,全要素に共通するひとつの型) を角括弧で囲みます.

### Project from (read from) an array index

We can project from (*read from*) an array using the usual bracket syntax (`[` and `]`) around the index we want to access:

配列からは, よくあるような角括弧構文 (`[` と `]` でアクセスしたいインデクスを囲う) を使って射影する (読み出す) ことができます:

``` motoko
let x : Nat = a[2] + a[0] ;
```

Every array access in Motoko is safe. Accesses that are out of bounds will not access memory unsafely, but instead will cause the program to trap, as with an [assertion failure](basic-concepts#overview-traps).

Motoko における配列へのアクセスは常に安全です. 境界を越えたアクセスはメモリの境界も越えてしまいますが, その場合には [表明の不成立](basic-concepts#overview-traps) としてプログラムがトラップします.

## The Array module

The Motoko standard library provides basic operations for immutable and mutable arrays. It can be imported as follows,

Motoko の標準ライブラリでは, 不変/可変配列に対する基本的な演算を用意しており, 次のようにしてインポートできます:

``` motoko
import Array "mo:base/Array";
```

In this section, we discuss some of the most frequently used array operations. For more information about using arrays, see the [Array](../../../../references/motoko-ref/array.md) library descriptions.

この節では, もっともよく使われる配列演算について説明します. 配列についてのそれ以上の情報はライブラリの記述 [配列](../../../../references/motoko-ref/array.md) をご覧下さい.

### Allocate an immutable array with varying content

Above, we showed a limited way of creating immutable arrays.

上では, 不変配列を作るやり方を一つだけ見てみました.

In general, each new array allocated by a program will contain a varying number of varying elements. Without mutation, we need a way to specify this family of elements "all at once", in the argument to allocation.

一般的に, プログラム内で割り付けられる新しい配列には, さまざまな数のさまざまな要素が含まれます. 変更がない場合には, この一群の要素を割り付けの引数として一挙に指定することができます.

To accommodate this need, the Motoko language provides *the higher-order* array-allocation function `Array.tabulate`, which allocates a new array by consulting a user-provided "generation function" `gen` for each element.

この必要性に応じて, Motoko 言語では _高階_ 配列割り付け関数 `Array.tabulate` を提供しています. これは各要素を "生成する関数" `gen` をユーザが定義し, それを用いて新しい配列を割り付けるものです.

``` motoko
func tabulate<T>(size : Nat,  gen : Nat -> T) : [T]
```

Function `gen` specifies the array *as a function value* of arrow type `Nat → T`, where `T` is the final array element type.

関数 `gen` は, 射型 `Nat → T` (ここで `T` は結果となる配列の要素の型) の関数値が配列を決めます.

The function `gen` actually *functions* as the array during its initialization: It receives the index of the array element, and it produces the element (of type `T`) that should reside at that index in the array. The allocated output array populates itself based on this specification.

関数 `gen` は初期化中に実際配列のように _機能_ します. つまり, 配列要素のインデクスを受け取って, 配列の index 番目に入れるべき要素 (型は `T`) を返します.

For instance, we can first allocate `array1` consisting of some initial constants, and then functionally-update *some* of the indices by "changing" them (in a pure, functional way), to produce `array2`, a second array that does not destroy the first.

例えば, まず `array1` をいくつかの初期定数から作ります. 次にインデクスの _いくつか_ を (純粋な関数的なやり方で)  "変換する" ことで関数的更新をします. そして最初の配列は壊さずに, 二番目の配列 `array2` を作り出しています.

``` motoko
let array1 : [Nat] = [1, 2, 3, 4, 6, 7, 8] ;

let array2 : [Nat] = Array.tabulate<Nat>(7, func(i:Nat) : Nat {
    if ( i == 2 or i == 5 ) { array1[i] * i } // change 3rd and 6th entries
    else { array1[i] } // no change to other entries
  }) ;
```

Even though we "changed" `array1` into `array2` in a functional sense, notice that both arrays and both variables are immutable.

`array1` を `array2` に関数型的な意味で変換したわけですが, 配列自体もそれを指す変数も不変であることに注意してください.

Next, we consider *mutable* arrays, which are fundamentally distinct.

次に可変配列を見てみますが, こちらは根本的に違うものです.

## Mutable arrays

Above, we introduced *immutable* arrays, which share the same projection syntax as mutable arrays, but do not permit mutable updates (assignments) after allocation. Unlike immutable arrays, each mutable array in Motoko introduces (private) mutable actor state.

上では _不変_ 配列を紹介しました. これは射影構文を用いることは可変配列と同じですが, いったん割り付けた後は変更更新 (代入) はできません.

Because Motoko’s type system enforces that remote actors do not share their mutable state, the Motoko type system introduces a firm distinction between mutable and immutable arrays that impacts typing, subtyping and the language abstractions for asynchronous communication.

Motoko の型システムでは遠隔のアクタとは可変な状態を共有させないようにしていますから, 型システムとして可変配列と不変配列をきちんと区別しておかないと, 型付けや部分型付け, 言語における非同期通信の抽象化が破綻します.

Locally, the mutable arrays can not be used in places that expect immutable ones, since Motoko’s definition of [subtyping](language-manual#subtyping) for arrays (correctly) distinguishes those cases for the purposes of type soundness. Additionally, in terms of actor communication, immutable arrays are safe to send and share, while mutable arrays can not be shared or otherwise sent in messages. Unlike immutable arrays, mutable arrays have *non-shareable types*.

ローカルでは不変配列が必要なところに可変配列を使うことはできません. Motoko の配列の [部分型付け](language-manual#subtyping) の定義では, 型の健全性を守るためにこの二つを (正しくも) 区別しています. さらにアクタ間通信の用語で言えば, 不変配列は送ったり共有しても安全ですが, 可変配列は共有したり, ましてやメッセージで送ることはできません. 不変配列とは異なり, 可変配列は _共有不可な型_ なのです.

### Allocate a mutable array of constants

To indicate allocation of *mutable* arrays (in contrast to the forms above, for immutable ones), the mutable array syntax `[var _]` uses the `var` keyword, in both the expression and type forms:

_可変_ 配列の割り付けであることを明確にするために (上でやった不変配列の形式とは対照的に), 可変配列の構文 `[var _]` において, 式形式でも型形式でも `var` というキーワードを用います:

``` motoko
let a : [var Nat] = [var 1, 2, 3] ;
```

As above, the array `a` above holds three natural numbers, but has type `[var Nat]`.

配列 `a` には上でやったのと同じく三つの自然数が入っていますが, 型は `[var Nat]` です.

### Allocate a mutable array with dynamic size

To allocate mutable arrays of non-constant size, use the `Array_init` primitive, and supply an initial value:

非定長の可変配列を割り付けるときは `Array_init` プリミティヴを使って, 初期値を与えます:

``` motoko
func init<T>(size : Nat,  x : T) : [var T]
```

For example:

例えば:

``` motoko
var size : Nat = 42 ;
let x : [var Nat] = Array.init<Nat>(size, 3);
```

The variable `size` need not be constant here; the array will have `size` number of entries, each holding the initial value `3`.

変数 `size` はここでは定数でなくても構いません. したがって, 配列には `size` 個分の要素を入れられ, 初期値はそれぞれ `3` になります.

### Mutable updates

Mutable arrays, each with type form `[var _]`, permit mutable updates via assignment to an individual element, in this case element index `2` gets updated from holding `3` to instead hold value `42`:

型形式 `[var _]` であるような可変配列は, 個々の要素への代入による更新が可能で, この場合だとインデクス `2` の要素は `3` から `42` に更新されます:

``` motoko
let a : [var Nat] = [var 1, 2, 3];
a[2] := 42;
a
```

### Subtyping does not permit *mutable* to be used as *immutable*

Subtyping in Motoko does not permit us to use a mutable array of type `[var Nat]` in places that expect an immutable one of type `[Nat]`.

Motoko の部分付けでは, 型 `[Nat]` である不変配列が必要な場所に型 `[var Nat]`であるような可変配列を使うことは許されていません.

There are two reasons for this. First, as with all mutable state, mutable arrays require different rules for sound subtyping. In particular, mutable arrays have a less flexible subtyping definition, necessarily. Second, Motoko forbids uses of mutable arrays across [asynchronous communication](actors-async), where mutable state is never shared.

理由は二つあります. 第一の理由は, すべての可変状態に対して可変配列は部分型付けの異なる規則が必要になるからです. 特に可変配列には柔軟性の低い部分型付けの定義が必要となります. 第二に, Motoko では非同期通信を通じて, 可変配列の利用を禁じているからです. 可変状態は決して共有されません.

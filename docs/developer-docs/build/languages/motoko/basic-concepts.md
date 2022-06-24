# Basic concepts and terms

Motoko is designed for distributed programming with actors.

Motoko はアクタを用いた分散プログラミングを前提として設計されました.

When programming on the Internet Computer in Motoko, each **actor** represents an **Internet Computer canister smart contract** with a Candid interface, whether written in Motoko, Rust, Wasm or some other language that compiles to Wasm. Within Motoko, we use the term **actor** to refer to any canister smart contract, authored in any language that deploys to the Internet Computer. The role of Motoko is to make these actors easy to author, and easy to use programmatically, once deployed.

インタネット・コンピュータ上 Motoko でプログラムを書く場合, **アクタ** は Candid インタフェイスの **Internet Computer キャニスタ・スマート・コントラクト** を表します (Mototo, Rust, Wasm その他の Wasm にコンパイルされる言語はどれもそうです). Motoko では Internet Computer に配備される, 任意の言語で記述されたキャニスタ・スマート・コントラクトを **アクタ** と呼ぶことにします. Motoko の役割はこれらアクタを書き易くすること, いったん配備された後も容易にプログラムから操れるようにすることです.

Before you begin writing distributed applications using actors, you should be familiar with a few of the basic building blocks of any programming language and with Motoko in particular. To get you started, this section introduces the following key concepts and terms that are used throughout the remainder of the documentation and that are essential to learning to program in Motoko:

分散アプリケーションを書く前から, Motoko や他のいろんなプログラミング言語の基本的な構成要素のいくつかには親しんでいると思います. この節では, 以後のドキュメントで使われる, また Motoko で書かれたプログラムの学習に欠かせない, 以下の主要概念/用語を紹介します:

-   program
-   declaration
-   expression
-   value
-   variable
-   type



-   プログラム
-   宣言
-   式
-   値
-   変数
-   型

If you have experience programming in other languages or are familiar with modern programming language theory, you are probably already comfortable with these terms and how they are used. There’s nothing unique in how these terms are used in Motoko. If you are new to programming, however, this guide introduces each of these terms gradually and by using simplified example programs that eschew any use of actors or distributed programming. After you have the basic terminology as a foundation to build on, you can explore more advanced aspects of the language. More advanced features are illustrated with correspondingly more complex examples.

もし他の言語を使ったことがあるとか, モダンなプログラミング言語理論が分かっているならば, これらの用語を耳にしたことがあり, 使い方も分かっているでしょう. これらの用語の使われ方は Motoko でも他の言語と特に変わりはありません. でも, もしプログラミング自体が初めてだとしたら, このガイドでこれらの用語をアクタや分散プログラミングを意識せずに簡単な例題プログラムから徐々に学んでいけると思います. いったん基本的な用語の使われ方を基礎から学んでしまえば, この言語のもっと先の観点を探索していけるでしょう. この先のフィーチャは, それに応じた難しさの例題を使って解説します.

The following topics are covered in the section:

この節では以下の題目について述べます:

-   [Motoko program syntax](#intro-progs)

-   [Printing numbers and text](#intro-printing), and [using the base library](#intro-stdlib)

-   [Declarations versus expressions](#intro-decls-vs-exps)

-   [Lexical scoping of variables](#intro-lexical-scoping)

-   [Values and evaluation](#intro-values)

-   [Type annotations variables](#intro-type-anno)

-   [Type soundness and type-safe evaluation](#intro-type-soundness)



-   [Motoko プログラムの構文](#intro-progs)

-   [数や文を出力する](#intro-printing) と [基本ライブラリの使い方](#intro-stdlib)

-   [宣言と式](#intro-decls-vs-exps)
-   [変数の静的スコープ](#intro-lexical-scoping)

-   [値とその評価](#intro-values)

-   [型アノテーション変数](#intro-type-anno)

-   [型の健全性と型安全な評価](#intro-type-soundness)

## Motoko program syntax

Each Motoko *program* is a free mix of declarations and expressions, whose syntactic classes are distinct, but related (see the [language quick reference guide](language-manual) for precise program syntax).

Motoko のプログラムは, 宣言と式 (この二つのシンタクスのクラスは異なるが, 関連はしている (プログラムの詳細な構文については [language quick reference guide](language-manual) を参照のこと)) を自由に並べたものです.

For programs that we deploy on the Internet Computer, a valid program consists of an *actor expression*, introduced with specific syntax (keyword `actor`) that we discuss in [Actors and async data](actors-async).

インタネット・コンピュータに配備するプログラムは, ひとつの _アクタ式_ から成ります (アクタ式の基本的な構文 (`actor` キーワード) については [アクタと非同期データ](actors-async) で述べます).

In preparing for that discussion, we discuss programs in this chapter and in [Mutable state](mutable-state) that are not meant to be Internet Computer services. Rather, these tiny programs illustrate snippets of Motoko for writing those services, and each can (usually) be run on its own as a (non-service) Motoko program, possibly with some printed terminal output.

この先に進む前にまず, この章と [可変状態](mutable-state) でインタネット・コンピュータ上でのサービスに限らないプログラムについて述べます.

The examples in this section illustrate basic principles using simple expressions, such as arithmetic. For an overview of the full expression syntax of Motoko, see the [Language quick reference](language-manual).

この節での例題では, (算術のような) 単純な式を用いた基本的な原理をお見せします. Motoko の式の構文全体の概要については [Language quick reference](language-manual) をご覧下さい.

As a starting point, the following code snippet consists of two declarations — for the variables `x` and `y` — followed by an expression to form a single program:

次のコード片, 二つの宣言 (変数 `x` と `y`) と一つの式から成る一つのプログラムから始めましょう:

``` motoko
let x = 1;
let y = x + 1;
x * y + x;
```

We will use variations of this small program in our discussion below.

以下では, この小さなプログラムをちょっとずつ変えながら説明していきます.

First, this program’s type is `Nat` (natural number), and when run, it evaluates to the (natural number) value of `3`.

まず, このプログラムの型は `Nat` (自然数) であり, 実行すると (自然数の) 値 3 に評価されます.

Introducing a block with enclosing braces (`do {` and `}`) and another variable (`z`), we can amend our original program as follows:

次に中括弧 (`do {}`) で囲まれたブロックと, 新たな変数 (`z`) を導入して. 元のプログラムを次のように変更します:

``` motoko
let z = do {
  let x = 1;
  let y = x + 1;
  x * y + x
};
```

## Declarations and expressions

Declarations introduce immutable variables, mutable state, actors, objects, classes and other types. Expressions describe computations that involve these notions.

宣言では, 不変変数 (おかしいですね:-), 可変状態, アクタ, オブジェクト, クラスなどの型を導入します. 式ではこれらの記法を含む計算を記述します.

For now, we use example programs that declare immutable variables, and compute simple arithmetic.

では, 不変変数を宣言して, 簡単な算術計算をするプログラムを例題として使いましょう.

### Declarations versus expressions

[Recall](#intro-progs) that each Motoko *program* is a free mix of declarations and expressions, whose syntactic classes are distinct, but related. In this section, we use examples to illustrate their distinctions and accommodate their intermixing.

Motoko の _プログラム_ は, 宣言と式の混在であり, そのシンタクス・クラスは別ではあるが, 互いに関係しているのだと言うことを [思い出して](#intro-progs) ください. この節では, その区別を説明し, 両方を雑ぜるような例題を使います.

Recall our example program, first introduced above:

上で最初に紹介した例題プログラムは以下のようなものでした:

``` motoko
let x = 1;
let y = x + 1;
x * y + x;
```

In reality, this program is a *declaration list* that consists of *three* declarations:

実際には, このプログラムは _三つの_ 宣言から成る _宣言の列_ なのです.

1.  immutable variable `x`, via declaration `let x = 1;`,

2.  immutable variable `y`, via declaration `let y = x + 1;`,

3.  and an *unnamed, implicit variable* holding the final expression’s value, `x * y + x`.



1.   宣言 `let x = 1;` では不変変数 `x` を
2.   宣言 `let y = x + 1;` では不変変数 `y` を
3.   そして最後の式の値 `x * y + x` をもつ無名の暗黙変数を

This expression `x * y + x` illustrates a more general principle: Each expression can be thought of as a declaration where necessary since the language implicitly declares an unnamed variable with that expression’s result value.

式 `x * y + x` は, より一般的な原則を示しています. それは, それぞれの式は必要ならば, その式の結果の値を持つ無名の変数を言語が暗黙のうちに宣言していると考えてもよい, というものです.

When the expression appears as the final declaration, this expression may have any type. Here, the expression `x * y + x` has type `Nat`.

式が最後の宣言として現れる場合には, この式は任意の型を持つことができます. ここでは式  `x * y + x` は型 `Nat` を持ちます.

Expressions that do not appear at the end, but rather *within* the list of declarations must have unit type `()`.

式が最後ではなく宣言の列の _途中で_ 現れる場合には, 単位型 `()` を持たなければなりません.

### Ignoring non-unit-typed expressions in declaration lists

We can always overcome this unit-type restriction by explicitly using `ignore` to ignore any unused result values. For example:

この単位型の制約を乗り越えるには, どんな場合でも明示的に `ignore` を用いて, 使われることのない結果の値を無視するようにします. 例えば:

``` motoko
let x = 1;
ignore(x + 42);
let y = x + 1;
ignore(y * 42);
x * y + x;
```

### Declarations and variable substitution

Declarations can be mutually recursive, but in cases where they are not, they permit substitution semantics. (that is, replacing equals for equals, as familiar from high-school algebraic simplification).

宣言は相互再帰していても構いませんが, そうでない場合には代入意味論が許されます (つまり, 高校の代数でやる簡略化のように, 等号を等号で置き換えられる).

Recall our original example:

元の例題に戻ってみましょう:

``` motoko
let x = 1;
let y = x + 1;
x * y + x;
```

We can manually rewrite the program above by *substituting* the variables' declared values for each of their respective occurrences.

上のプログラムでは, 宣言された変数の値をそれぞれの出現場所に代入するように書き換えることができます.

In so doing, we produce the following expression, which is also a program:

実際にやってみると, 以下のような式になります (これもプログラムです):

``` motoko
1 * (1 + 1) + 1
```

This is also a valid program — of the same type and with the same behavior (result value `3`) — as the original program.

これも, 元のプログラムと同じ型, 同じ振る舞い (結果の値は `3` になる) を持つ, 妥当なプログラムなのです. 

We can also form a single expression using a block.

ブロックを使って一つの式にまとめることもできます.

### From declarations to block expressions

Many of the programs above each consist of a list of declarations, as with this example, just above:

上で見てきたプログラムはだいたい, 一連の宣言からできていました. この例は, 上のものと同じですが:

``` motoko
let x = 1;
let y = x + 1;
x * y + x
```

A declaration list is not itself (immediately) an *expression*, so we cannot (immediately) declare another variable with its final value (`3`).

宣言の列は, それ自身 (そのままでは) 式ではないので,  (値が `3` になる) 別の変数を (そのまま) 宣言することはできません.

**Block expressions.** We can form a *block expression* from this list of declarations by enclosing it with matching *curly braces*. Blocks are only allowed as sub-expressions of control flow expressions like `if`, `loop`, `case`, etc. In all other places, we use `do { …​ }` to represent block expression, to distinguish blocks from object literals. For example, `do {}` is the empty block of type `()`, while `{}` is an empty record of record type `{}`.

**ブロック式**. この宣言の列を中括弧で括るとブロック式を作ることができる. ブロックが使えるのは, `if`, `loop`, `case` のような制御フロー式の副式としてだけである. その他の場所で使うときには `do {...}` でブロック式を表現して, ブロックとオブジェクト・リテラルが区別できるようにする. 例えば, `do {}` は型 `()` の空ブロック, `{}` はレコード型 `{}` の空レコードと言うことになる.

``` motoko
do {
  let x = 1;
  let y = x + 1;
  x * y + x
}
```

This is also program, but one where the declared variables `x` and `y` are privately scoped to the block we introduced.

これもまた一つのプログラムになりますが, ここで宣言された変数 `x` と `y` は, ここで導入したブロックにプライベートなスコープを持ちます.

This block form preserves the autonomy of the declaration list and its *choice of variable names*.

このブロックは, 宣言列の自律性と変数名の選択を保持します.

### Declarations follow **lexical scoping**

Above, we saw that nesting blocks preserves the autonomy of each separate declaration list and its *choice of variable names*. Language theorists call this idea *lexical scoping*. It means that variables' scopes may nest, but they may not interfere as they nest.

上では, 入れ子になったブロックにおいて, 分離された宣言列の自律性と変数名の選択が保持されることを見てきました. 言語論ではこれを _レキシカル・スコープ_ と呼びます. つまり, 変数のスコープは入れ子になってもいいけど, 入れ子の外には干渉しない, と言うことです.

For instance, the following (larger, enclosing) program evaluates to `42`, *not* `2`, since the final occurrences of `x` and `y`, on the final line, refer to the *very first* definitions, *not* the later ones within the enclosed block:

例えば, 次のプログラムの全体を評価すると `42` になります. なぜならば, `x` と `y` の最後の出現は最後の行になり, それは囲みのブロックの中ではなく, いちばん最初の行の定義を参照しているからです:

``` motoko
let x = 40; let y = 2;
ignore do {
  let x = 1;
  let y = x + 1;
  x * y + x
};
x + y
```

Other languages that lack lexical scoping may give a different meaning to this program. However, modern languages universally favor lexical scoping, the meaning given here.

レキシカル・スコープを採用していない言語では, このプログラムの意味は異なるものになるでしょう. しかし, モダンな言語ではほぼレキシカル・スコープが好まれており, ここで与えたのと同じ意味論になります.

Aside from mathematical clarity, the chief practical benefit of lexical scoping is *security*, and its use in building compositionally-secure systems. Specifically, Motoko gives very strong composition properties. For example, nesting your program within a program you do not trust cannot arbitrarily redefine your variables with different meanings.

## Values and evaluation

Once a Motoko expression receives the program’s (single) thread of control, it evaluates eagerly until it reduces to a *result value*.

Motoko の式がプログラムの (単一) 制御スレッドを受け取ると, 最終的な _結果値_ が出るまで先行評価を行います.

In so doing, it will generally pass control to sub-expressions, and to sub-routines before it gives up control from the *ambient control stack*.

多くの場合, その過程で副式やサブルーチンに制御を渡し, 最終的に _ambient control stack_ からの制御を手放します (__TODO__).

If this expression never reaches a value form, the expression evaluates indefinitely. Later we introduce recursive functions and imperative control flow, which each permit non-termination. For now, we only consider terminating programs that result in values.

もし式が値形式に帰着しない場合には, その式は無限に評価を続けます. 後ほど再帰関数と手続き的な制御フローを導入しますが, どちらも終了しない可能性があります. 今のところは, プログラムは最終的には値を得て終了するものとしておきましょう.

In the material above, we focused on expressions that produced natural numbers. As a broader language overview, however, we briefly summarize the other value forms below:

ここまでの材料では, 自然数を生成する式だけを見てきました. 言語のより広範な概要を知るために, ここで他の値形式を手短にまとめておきます:

### Primitive values

Motoko permits the following primitive value forms:

Motoko には以下の原始値形式があります:

-   Boolean values (`true` and `false`).
-   Integers (…​,`-2`, `-1`, `0`, `1`, `2`, …​) - bounded and *unbounded* variants.
-   Natural numbers (`0`, `1`, `2`, …​) - bounded and *unbounded* variants.
-   Text values - strings of unicode characters.



-   ブール値 (`true`, `false`)
-   整数 (…,`-2`, `-1`, `0`, `1`, `2`, …) - 上下限あり, 上下限無しのバリエーション
-   自然数 (`0`, `1`, `2`, …) - 上限あり, 上限無しのバリエーション
-   テキスト値 - ユニコード文字列

By default, **integers** and **natural numbers** are *unbounded* and do not overflow. Instead, they use representations that grow to accommodate any finite number.

**整数** と **自然数** は, デフォルトでは上下限無しで溢れはありません. したがって, 任意の有限数を大きさに合わせて入れられるような表現を用いています.

For practical reasons, Motoko also includes *bounded* types for integers and natural numbers, distinct from the default versions. Each bounded variant has a fixed width (one of `8`, `16`, `32`, `64`) and each carries the potential for “overflow”. If and when this event occurs, it is an error and causes the [program to trap](#overview-traps). There are no unchecked, uncaught overflows in Motoko, except in well-defined situations, for explicitly *wrapping* operations (indicated by a `%` character in the operator). The language provides primitive built-ins to convert between these various number representations.

実用上の要請から, Motoko ではデフォルトとは異なる, 上 (下) 限ありの整数型, 自然数型も用います. 上 (下) 限ありの場合には, 大きさ (ビット数) は固定 (`8`, `16`, `32`, `64` のいずれか) で, 溢れる可能性があります. 溢れが起きた場合には, エラーとなり, [プログラム・トラップ](#overview-traps) を引き起こします. Motoko ではちゃんと定義された状況で明示的に _ラッピング_ 演算 (演算子に `%` 文字を付けて示す) をする場合を除き, すべての溢れは検査され, 捕捉されます. さまざまな種類の数値表現の間の変換は, 言語に組み込んで提供されています.

The [language quick reference](language-manual) contains a complete list of [primitive types](language-manual#primitive-types).

[language quick reference](language-manual) に [原始型](language-manual#primitive-types) の完全なリストがあります.

### Non-primitive values

Building on the primitive values and types above, the language permits user-defined types, and each of the following non-primitive value forms and associated types:

上で見た原始値, 原始型を用いてユーザが型を定義できるように, 以下の非原始的な値とそれに対応する型を用意しています:

-   [Tuples](language-manual#exp-tuple), including the unit value (the "empty tuple")

-   [Arrays](language-manual#exp-arrays), with both *immutable* and *mutable* variants.

-   [Objects](language-manual#exp-object), with named, unordered fields and methods

-   [Variants](language-manual#variant-types), with named constructors and optional payload values

-   [Function values](language-manual#exp-func), including [shareable functions](sharing).

-   [Async values](language-manual#exp-async), also known as *promises* or *futures*.

-   [Error values](language-manual#type-Error) carry the payload of exceptions and system failures



-   [タプル](language-manual#exp-tuple), 単位値 (「空タプル」) を含む

-   [配列](language-manual#exp-arrays), 不変と可変のバリエーション

-   [オブジェクト](language-manual#exp-object), 名前付きフィールド/メソッドがある (その順序は不定)

-   [ヴァリアント](language-manual#variant-types), 名前付きコンストラクタがある (値を載っけることもできる)

-   [関数値](language-manual#exp-func), [共有関数](sharing) を含む

-   [非同期値](language-manual#exp-async), いわゆるプロマイズあるいはフューチャ

-   [エラー値](language-manual#type-Error), 例外やシステム・エラーの情報を持つ

We discuss the use of these forms in the succeeding chapters. For precise language definitions of primitive and non-primitive values, see the [language quick reference](language-manual#exp-error).

これらの値形式, 型の使い方については, これ以降の章で述べます. 原始値/非原始値についての詳細な言語定義は [language quick reference](language-manual#exp-error) をご覧下さい.

### The **unit type** versus the `void` type

Motoko has no type named `void`. In many cases where readers may think of return types being “void” from using languages like Java or C++, we encourage them to think instead of the *unit type*, written `()`.

Motoko には `void` という名前の型はありません. Java や C++ などの言語を使った経験から, 読者が戻り値の型を "void" にしたいと思ったときには, 代わりに _単位型_ `()` を使うのだと考えてください.

In practical terms, like `void`, the unit value usually carries zero representation cost.

実用の観点から言えば, `void` と同様, 単位値の表現にかかるコストは通常ゼロです.

Unlike the `void` type, there *is* a unit value, but like the `void` return value, the unit value carries no values internally, and as such, it always carries zero *information*.

 `void` 型とは異なり, 単位型には単位値というものが _あります_. とは言え, `void` 返値と同様に単位値は内部的には値を持っているわけではありません. 単位値が担う _情報_ はゼロです.

Another mathematical way to think of the unit value is as a tuple with no elements - the nullary (“zero-ary”) tuple. There is only one value with these properties, so it is mathematically unique, and thus need not be represented at runtime.

別の数学的な考え方としては, 単位値は要素のないタプル (無項タプル, nullary) と考えられます. このような性質を持つ値はひとつしかないので, 数学的にも一意ということになり, したがって実行時にそれを表現する必要がないのです.

### Natural numbers

The members of this type consist of the usual values - `0`, `1`, `2`, …​ - but, as in mathematics, the members of `Nat` are not bound to a special maximum size. Rather, the runtime representation of these values accommodates arbitrary-sized numbers, making their "overflow" (nearly) impossible. (*nearly* because it is the same event as running out of program memory, which can always happen for some programs in extreme situations).

この型の要素は, 普通の値 - `0`, `1`, `2`, … - から成ります. しかし, 数学的には `Nat` に特別な最大値というものがあるわけではありません. そこで, これらの値の実行時表現は任意長の数で, 「溢れ」が (ほぼ) 起こりません. (_ほぼ_ というのは, プログラムのメモリを使い切ってしまったというようなイベントもあり得るからです. 極端な状況ではいつでも起こりうる問題ですが). 

Motoko permits the usual arithmetic operations one would expect. As an illustrative example, consider the following program:

Motoko では, だれもが期待するような通常の算術演算ができます. わかりやすい例として, 次のプログラムを見てください:

``` motoko
let x = 42 + (1 * 37) / 12: Nat
```

This program evaluates to the value `45`, also of type `Nat`.

このプログラムを評価すると, 型 `Nat` の  `45` になります.

## Type soundness

Each Motoko expression that type-checks we call *well-typed*. The *type* of a Motoko expression serves as a promise from the language to the developer about the future behavior of the program, if executed.

Motoko の型検査された式を _正しく型付けされた_ (well-typed) 式と呼びます. Motoko の式の _型_ は言語から開発者へのプログラムを将来実行したら何が起こるかという約束です.

First, each well-typed program will evaluate without undefined behavior. That is, the phrase **“well-typed programs don’t go wrong”** applies here. For those unfamiliar with the deeper implications of that phrase, it means that there is a precise space of meaningful (unambiguous) programs, and the type system enforces that we stay within it, and that all well-typed programs have a precise (unambiguous) meaning.

まず, 正しく型付けされたプログラムは, 評価されて未定義の振る舞いをすることはありません. つまり, **"正しく型付けされたプログラムは誤りがない"** という文句はここで当てはまるのです. この文句の深い意味を捉えきれていない人には, __TODO__

Furthermore, the types make a precise prediction over the program’s result. If it yields control, the program will generate a *result value* that agrees with that of the original program.

__TODO__

In either case, the static and dynamic views of the program are linked by and agree with the static type system. This agreement is the central principle of a static type system, and is delivered by Motoko as a core aspect of its design.

いずれの場合も, 静的型システムがプログラムの静的な側面と動的な側面を関係づけ, 齟齬がないようにしています. この取り決めこそが静的な型システムの中心的な原理であり, Motoko がその言語設計の核となる観点として提供するものです.

The same type system also enforces that asynchronous interactions agree between static and dynamic views of the program, and that the resulting messages generated "under the hood" never mismatch at runtime. This agreement is similar in spirit to the caller/callee argument type and return type agreements that one ordinarily expects in a typed language.

この型システムは同時に, 非同期な相互作用がプログラムの静的な側面と動的な側面で齟齬がないこと, 「裏側の仕組み」が生成する結果メッセージが実行時に必ず一致することを保証しています. この一致は本質的には型付き言語で呼び出し側と呼ばれた側で引数の型と返値の型が一致するという, 誰もが期待することと同じです.

## Type annotations and variables

Variables relate (static) names and (static) types with (dynamic) values that are present only at runtime.

変数は (静的な) 名前と, (静的な) 型と, (実行時にのみ存在する, 動的な) 値とを関連付けます.

In this sense, Motoko types provide a form of *trusted, compiler-verified documentation* in the program source code.

その意味で, Motoko の型は _信頼性のある, コンパイラが検証済のドキュメント_ がプログラムのソースコード中にあるのと同じです.

Consider this very short program:

次の短いプログラムを見てみましょう:

``` motoko
let x : Nat = 1
```

In this example, the compiler infers that the expression `1` has type `Nat`, and that `x` has the same type.

この例では, コンパイラが式 `1` が型 `Nat` であること, また `x` も同じ型であると推論しています.

In this case, we can omit this annotation without changing the meaning of the program:

この場合には, この注釈部分を削除してしまってもプログラムの意味は変わりません:

``` motoko
let x = 1
```

Except for some esoteric situations involving operator overloading, type annotations do not (typically) affect the meaning of the program as it runs.

演算子の多重定義のような特殊な状況を除けば, 型注釈は実行時にプログラムの意味に (おおよそ) 影響を与えることはありません.

If they are omitted and the compiler accepts the program, as is the case above, the program has the same meaning (same *behavior*) as it did originally.

上の場合のように, 型注釈を省略して, コンパイラがそのプログラムを受け付けてくれたとしたら, そのプログラムは最初の場合と同じように, 同じ意味 (そして同じ振る舞い) をもち, 同じ振る舞いをする, と言うことです.

However, sometimes type annotations are required by the compiler to infer other assumptions, and to check the program as a whole.

ただし, コンパイラが他の前提を推論したり, プログラム全体を検査するために型注釈を必要とする場合もあります.

When they are added and the compiler still accepts the program, we know that the added annotations are *consistent* with the existing ones.

そのような型注釈を付け加えてなおコンパイラがプログラムを受け付けたならば, その型注釈は元のものと _無矛盾_ であると言うことが分かります.

For instance, we can add additional (not required) annotations, and the compiler checks that all annotations and other inferred facts agree as a whole:

例えば, 次のような型注釈を (この場合は必須ではありませんが) 付け加えると, コンパイラはすべての型注釈と, そこから推論される事実が全体として辻褄が合っているかどうかを検査します:

``` motoko
let x : Nat = 1 : Nat
```

If we were to try to do something *inconsistent* with our annotation type, however, the type checker will signal an error.

もし何か矛盾を起こすような型注釈を付けようとすると, 型検査器はエラーを知らせます.

Consider this program, which is not well-typed:

この (正しく型付けされていない) プログラムを見てみましょう:

``` motoko
let x : Text = 1 + 1
```

The type annotation `Text` does not agree with the rest of the program, since the type of `1 + 1` is `Nat` and not `Text`, and these types are unrelated by subtyping. Consequently, this program is not well-typed, and the compiler will signal an error (with a message and location) and will not compile or execute it.

`1 + 1` の型は `Nat` であって `Text` ではありませんし, 部分型の関係でもありませんから, `Text` という型注釈はプログラムの他の部分と辻褄が合ってないことになります.

## Type errors and messages

Mathematically, the type system of Motoko is *declarative*, meaning that it exists independently of any implementation, as a concept entirely in formal logic. Likewise, the other key aspects of the language definition (for example, its execution semantics) exist outside of an implementation.

数学的には, Motoko の型システムは _宣言的_ であると言えます. これは, その概念が形式的な論理に基づいたものであり, どんな実装に依存していないと言うことです. Motoko の言語定義の他の鍵となる側面 (例えばその実行セマンティクス) もどんな実装とも離れたところにあります.

However, to design this logical definition, to experiment with it, and to practice making mistakes, we want to interact with this type system, and to make lots of harmless mistakes along the way.

しかし, この論理的な定義を設計する, それを実際に動かす, また間違いから学ぶときにはこの型システムと対話し, その過程で害のない間違いをたくさんしていこうと思っています.

The error messages of the *type checker* attempt to help the developer when they misunderstand or otherwise misapply the logic of the type system, which is explained indirectly in this book.

型検査器のエラー・メッセージは, 開発者が勘違いしていたり, 型システムの論理を間違って適用したりしたときの助けになるようになっていますが, この本では間接的に説明してあります.

These error messages will evolve over time, and for this reason, we will not include particular error messages in this text. Instead, we will attempt to explain each code example in its surrounding prose.

エラー・メッセージは時を経るにしたがって進化していきます. なので, このテキストには特定のエラー・メッセージは直接書いてしまうのではなく, その部分を例題のコードで説明するようにしました.

### Using the Motoko base library

For various practical language engineering reasons, the design of Motoko strives to minimize builtin types and operations.

さまざまな言語工学の経験に基づいて,  Motoko では組込みの型や演算は最小限にとどめるような設計にしました.

Instead, whenever possible, the Motoko base library provides the types and operations that make the language feel complete. ***However**, this base library is still under development, and is still incomplete*.

その代わりに可能な限り, 言語を完全なものにするための型や演算を Motoko ベース・ライブラリとして提供しています. _ただし_ このベース・ライブラリはまだ開発中で, 完全なものではありません.

The [Motoko Base Library](../../../../references/motoko-ref/stdlib-intro) lists a *selection* of modules from the Motoko base library, focusing on core features used in the examples that are unlikely to change radically. However, all of these base library APIs will certainly change over time (to varying degrees), and in particular, they will grow in size and number.

 [Motoko ベース・ライブラリ](../../../../references/motoko-ref/stdlib-intro) には, Motoko ベース・ライブラリからそうそうは変わらないだろうと思われる, この本の例題で使っている核となるフィーチャに絞って, 選び抜いたモジュール群について書いてあります.

To import from the base library, use the `import` keyword. Give a local module name to introduce, in this example `D` for “**D**ebug”, and a URL where the `import` declaration may locate the imported module:

ベース・ライブラリからインポートするには, `import` と言うキーワードを使います. また, ローカルに用いられるモジュール名 (この例では "**D**ebug" の `D`) と `import` 宣言でインポートするモジュールの場所が分かるような URL を与えます:

``` motoko
import D "mo:base/Debug";
D.print("hello world");
```

In this case, we import Motoko code (not some other module form) with the `mo:` prefix. We specify the `base/` path, followed by the module’s file name `Debug.mo` minus its extension.

ここでは, `mo:`という前置子を付けて  (何か別のモジュール形式ではなく) Motoko コードそのものをインポートしています. さらに `base/` パス にモジュールのファイル名 `Debug.mo` から拡張子を除いたものを付けています.

### Printing using `Debug.print` and `debug_show`

Above, we print the text string using the function `print` in library `Debug.mo`:

上で, ライブラリ `Debug.mo` の `print`関数を使ってテキスト文字列を表示しました:

``` motoko
print: Text -> ()
```

The function `print` accepts a text string (of type `Text`) as input, and produces the *unit value* (of *unit type*, or `()`) as its output.

関数 `print` はテキスト文字列 (型は `Text`) を一つ入力に取り, _単位値_ (単位型, つまり `()`) を出力として作り出します.

Because unit values carry no information, all values of type unit are identical, so the `print` function doesn’t actually produce an interesting result. Instead of a result, it has a *side effect*. The function `print` has the effect of emitting the text string in a human-readable form to the output terminal. Functions that have side effects, such as emitting output, or modifying state, are often called *impure*. Functions that just return values, without further side-effects, are called *pure*. We discuss the return value (the unit value) [in detail below](#intro-unit-type), and relate it to the `void` type for readers more familiar with that concept.

単位値は何の情報もになっていないので, この型の値はすべて同一になり, `print` 関数は実際には何ら意味のある結果は作り出しません. 結果は出さないのですが, _副作用_ があります. 関数 `print` の副作用は, ひとが読みやすいような形式でテキスト文字列を出力ターミナルに出すというものです. 何かを出力したり, 状態を変えるような副作用を持つ関数は _不純_  と呼ばれます. 何ら副作用なく, 単に値を返すだけの関数は _純粋_ と呼ばれます. 返値 (単位値) については [以下で詳細に](#intro-unit-type) 述べますが, `void` に馴染みのある読者はそれと似たものだと考えてください.

Finally, we can transform most Motoko values into human-readable text strings for debugging purposes, *without* having to write those transformations by hand.

結局 Motoko の値のほとんどは, このような変換を _人手で書かなくても_  デバグ時にひとが読みやすいテキスト文字列に変換できます.

The `debug_show` primitive permits converting a large class of values into values of type `Text`.

`debug-show` というプリミティヴが, 多くの種類の値を 型 `Text` の値に変換してくれるのです.

For instance, we can convert a triple (of type `(Text, Nat, Text)`) into debugging text without writing a custom conversion function ourselves:

例えばある三つ組み (型は `(Text, Nat, Text)`) をいちいち専用の変換関数を作らなくてもデバグ用のテキストに変換できます:

``` motoko
import D "mo:base/Debug";
D.print(debug_show(("hello", 42, "world")))
```

Using these text transformations, we can print most Motoko data as we experiment with our programs.

このようなテキスト変換を使って, Motoko ではデータのほとんどを上のプログラムのように出力できるのです.

### Accommodating incomplete code

Sometimes, in the midst of writing a program, we want to run an incomplete version, or a version where one or more execution paths are either missing or simply invalid.

プログラムを書いている途中で, まだ不完全なバージョンや実行パスの何カ所かがまだ書いていなかったり, 単に動かなかったりする場合でも, とりあえず動かしてみたくなることがあります.

To accommodate these situations, we use the `xxx`, `nyi` and `unreachable` functions from the base `Prelude` library, explained below. Each wraps a [general trap mechanism](#overview-traps), explained further below.

このような状況に配慮して, `xxx`, `nyi` and `unreachable` という関数を `Prelude` ベース・ライブラリに用意しましたので紹介しておきます. これら関数は以下に説明するように, どれも [汎用的なトラップの仕組み](#overview-traps) をラップしたものです.

### Use short-term holes

Short-term holes are never committed to a source repository, and only ever exist in a single development session, for a developer that is still writing the program.

「短期的な穴」は, ソース・リポジトリには決してコミットされないような類のもので, 単独の開発セッション中に, 開発者がまだプログラムを書いている途中にだけ存在します.

Assuming that earlier, one has imported the prelude as follows:

まず, プレリュード (静的型を持つプログラミング言語の一部のライブラリで, 実行の前提となるような型や関数の集合を prelude と呼ぶことがあります) を次のようにインポートしておきます:

``` motoko
import P "mo:base/Prelude";
```

The developer can fill *any missing expression* with the following one:

開発者は _まだ書いていない, どんな式_ でも, 以下のように書いておくことができます:

``` motoko
P.xxx()
```

The result will *always* type check at compile time, and *will always* trap at run time, if and when this expression ever executes.

こうしておくとコンパイル時の型検査はすべて通りますが, 実行すれば常にトラップに落ちるので, この式が実行されたかどうかが分かるようになっています.

### Document longer-term holes

By convention, longer-term holes can be considered "not yet implemented" (`nyi`) features, and marked as such with a similar function from the Prelude module:

「長期的な穴」とは, よく使う "not yet implemented (未実装)" (`nyi`) と書かれるようなフィーチャのことで, プレリュード・モジュールの同じような関数で印付けします:

``` motoko
P.nyi()
```

### Document `unreachable` code paths

In contrast to the situations above, sometimes code will *never* be filled, since it will *never* be evaluated, assuming the coherence of the internal logic of the programs' invariants.

上で述べたようなふたつの状況とは対照的に, プログラムの不変条件を表す内部論理の一貫性を前提として, 決して書かれない (決して評価されることがないので) コードがあり得ます.

To document a code path as logically impossible, or *unreachable*, use the base library function `unreachable`:

論理的に不可能, つまり _到達不可能_ なコード・パスをドキュメント化するために, ベース・ライブラリの関数 `unreachable` を使います:

``` motoko
P.unreachable()
```

As in the situations above, this function type-checks in all contexts, and when evaluated, traps in all contexts.

このような状況では, この関数はどのような文脈でも型検査を通り, 評価されるとどんな文脈であろうとトラップに落ちます.

### Traps due to execution failure

Some errors, such as division by zero, out-of-bounds array indexing, and pattern match failure are not prevented by the type system, but can cause dynamic failures called *traps*.

ゼロ除算や配列の範囲外インデクス, パタン・マッチ失敗のような, ある種のエラーは型システムでは防ぎようがありません. このような動的な失敗は _トラップ_ を起こします.

``` motoko
1/0; // traps due to division by 0
```

``` motoko
let a = ["hello", "world"];
a[2]; // traps due to out-of-bounds indexing
```

``` motoko
let true = false; // pattern match failure
```

We say that code *traps* when its exection causes a *trap*.

実行すると _トラップ_ を起こすことを, コードがトラップすると言ったりします.

Execution of code is aborted at the first trap and makes no further progress.

コードの実行は最初のトラップで中断され, その先に進むことはありません.

:::note

Traps that occur within actor messages are more subtle: they don’t abort the entire actor, but prevent that particular message from proceeding, rolling back any yet uncommitted state changes. Other messages on the actor will continue execution.

アクタのメッセージングの最中でトラップが起こったような場合は, もっと微妙です. つまり, その場合, アクタ全体が中断するのではなく, そのメッセージ処理が中断し, その時点でまだコミットされていない状態変化はロールバックします. そのアクタの他のメッセージは実行を継続します.

:::

### Explicit traps

Occasionally it can be useful to force an unconditional trap, with a user-defined message.

ユーザが定義したメッセージで, 無条件にトラップが起こるようにしておくと役に立つこともたまにはあります.

The `Debug` library provides the function `trap(t)` for this purpose, which can be used in any context.

`Debug` ライブラリでは `trap(t)` 関数を提供しており, この関数は任意の文脈で使うことができます.

``` motoko
import Debug "mo:base/Debug";

Debug.trap("oops!");
```

``` motoko
import Debug "mo:base/Debug";

let swear : Text = Debug.trap("oh my!");
```

(The `Prelude` functions `nyi()`, `unreachable()` and `xxx()` discussed above are simple wrappers around `Debug.trap`.)

(上で議論した`nyi()`, `unreachable()` and `xxx()` など `Prelude` の関数は, この `Debug.trap` を単にラップしたものです)

### Assertions

Assertions allow you to conditionally trap when some Boolean test fails to hold, but continue execution otherwise. For example,

表明とは, 何らかの真偽値を返すような検査が失敗したときに限ってトラップさせるもので, それ以外の場合には実行は継続します. 例えば:

``` motoko
let n = 65535;
assert n % 2 == 0; // traps when n not even
```

``` motoko
assert false; // unconditionally traps
```

``` motoko
import Debug "mo:base/Debug";

assert 1 > 0; // never traps
Debug.print "bingo!";
```

Because an assertion may succeed, and thus proceed with execution, it may only be used in context where a value of type `()` is expected.

表明が成功した場合には実行は続くので, 値が型 `()` になるような文脈でしか使えません.

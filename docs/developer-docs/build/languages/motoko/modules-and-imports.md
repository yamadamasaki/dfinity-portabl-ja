# Modules and imports

This section provides examples of different scenarios for using the `module` and `import` keywords.

この節では, `module` と `import` キーワードを使ういくつかのシナリオの例を用意しています.

To illustrate how these keywords are used, let’s step through some sample code.

これらのキーワードをどうやって使うかを説明しますので, 先のサンプル・コードに進んでいってください.

## Importing from the Motoko base library

One of the most common import scenarios is one that you see illustrated in the examples in this guide, in the Motoko projects in the examples repository, and in the tutorials involves importing modules from the Motoko base library. Importing modules from the base library enables you to re-use the values, functions and types defined in those modules rather than writing similar ones from scratch.

import のもっとも良くあるシナリオの一つは, このガイドの例の中や例題リポジトリの Motoko プロジェクト, Motoko ベース・ライブラリからモジュールをインポートしているチュートリアルで説明しています. ベース・ライブラリからモジュールをインポートすると, ゼロから似たようなものを書き直さずに, モジュールに定義されている値, 関数, 型を再利用することができます.

The following two lines import functions from the `Array` and `Result` modules:

次の二行では, `Array` モジュールと `Result` モジュールから関数をインポートしています:

``` motoko
import Array "mo:base/Array";
import Result "mo:base/Result";
```

Notice that the import declaration includes the `mo:` prefix to identify the module as a Motoko module and that the declaration does not include the `.mo` file type extension.

import 宣言を見ると, `mo:` 前置子を付けることで, Motoko モジュールとして識別していること, 宣言にはファイル拡張子の `.mo` は付けていないことが分かりますね.

Above example uses an identifier pattern to import modules wholesale, but you can also selectively import a subset of symbols from a module by resorting to the object pattern syntax:

上の例では, 識別子パタンを用いてモジュール全体をインポートしていますが, オブジェクト・パタンの構文を用いれば, シンボルの部分集合だけを選択的にインポートすることもできます:

``` motoko
import { map, find, foldLeft = fold } = "mo:base/Array";
```

In this example, the functions `map` and `find` are imported unaltered, while the `foldLeft` function is renamed to `fold`.

この例では, 関数 `map` と `find` を名前を変えずにインポートし, `foldLeft` 関数を `fold` としてインポートしています.

## Importing local files

Another common approach to writing programs in Motoko involves splitting up the source code into different modules. For example, you might design an application to use the following model:

Motoko でプログラムを書くときの別のやり方として, ソース・コードを複数のモジュールに分割するやり方があります. 例えばアプリケーションを次のようなモデルを用いて設計します:

-   a `main.mo` file to contain the actor and functions that change state.

-   a `types.mo` file for all of your custom type definitions.

-   a `utils.mo` file for functions that do work outside of the actor.




-   `main.mo` ファイルにはアクタと状態を変える関数を入れる
-   `types.mo` ファイルには自前の型定義を全部入れる
-   `utils.mo` ファイルにはアクタの外で使う関数を入れる

In this scenario, you might place all three files in the same directory and use a local import to make the functions available where they are needed.

このシナリオでは, この三つのファイルを全部同じディレクトリに置いて, 必要なときに使えるようにローカル・インポートします.

For example, the `main.mo` contains the following lines to reference the modules in the same directory:

例えば, `main.mo` には以下の行を入れて, 同じディレクトリ内のモジュールを参照します:

``` motoko
import Types "types";
import Utils "utils";
```

Because these lines import modules from the local project instead of the Motoko library, these import declarations don’t use the `mo:` prefix.

これらの行では, モジュールは Motoko ライブラリからではなく, ローカル・プロジェクトからインポートするので, インポート宣言では `mo:` 前置子は使いません.

In this example, both the `types.mo` and `utils.mo` files are in the same directory as the `main.mo` file. Once again, import does not use the `.mo` file suffix.

この例では, `type.mo` ファイルも `utils.mo ` ファイルも `main.mo` ファイルと同じディレクトリにあります. 個々でも `.mo` 後置子は使っていません.

## Importing from another package or directory

You can also import modules from other packages or from directories other than the local directory.

他のパッケージやローカル・ディレクトリ以外のディレクトリからインポートすることもできます.

For example, the following lines import modules from a `redraw` package that is defined as a dependency:

例えば, 次の行では依存性として定義されている `redraw` パッケージからモジュールをインポートしています:

``` motoko
import Render "mo:redraw/Render";
import Mono5x5 "mo:redraw/glyph/Mono5x5";
```

You can define dependencies for a project using the Vessel package manager or in the project `dfx.json` configuration file.

依存性は, プロジェクトで Vessel パッケージ・マネジャを使うか, プロジェクトの `dfx.json` 構成ファイルで定義することができます

In this example, the `Render` module is in the default location for source code in the `redraw` package and the `Mono5x5` module is in a `redraw` package subdirectory called `glyph`.

この例では `Render` モジュールは `redraw` パッケージ内のソース・コードのデフォルト位置にあり, `Mono5x5` モジュールは, `redraw` パッケージの `glyph` というサブディレクトリにあります.

## Importing actor classes

While module imports are typically used to import libraries of local functions and values, they can also be used to import actor classes. When an imported file consists of a named actor class, the client of the imported field sees a module containing the actor class.

モジュールのインポートは, 典型的にはローカルの関数や型のライブラリをインポートするのに用いられますが, アクタ・クラスをインポートするときには使うことができます. インポートしたファイルが名前の付いたアクタ・クラスを含んでいた場合, インポートされたフィールドのクライアントは, アクタ・クラスの入ったモジュールを見ることになります.

This module has two components, both named after the actor class:

このモジュールには二つのコンポネントがあり, モジュール名はアクタ・クラスにちなんで付けています.

-   a type definition, describing the interface of the class, and

-   an asynchronous function, that takes the class parameters as arguments an asynchronously returns a fresh instance of the class.



-   そのクラスのインタフェイスを記述した型の定義
-   クラスのパラメタを引数に取り,  そのクラスの新しいインスタンスを非同期に返す非同期関数

For example, a Motoko actor can import and instantiate the `Counter` class described in [Actors and async data](actors-async#actor_class) as follows:

例えば, Motoko のアクタは Counter クラスをインポートして, [アクタと非同期データ](actors-async#actor_class) にも書いてありますが, その後のようにしてインスタンス化できます:

**Counters.mo**

</div>

``` motoko
actor class Counter(init : Nat) {
  var count = init;

  public func inc() : async () { count += 1 };

  public func read() : async Nat { count };

  public func bump() : async Nat {
    count += 1;
    count;
  };
};
```

<div class="formalpara-title">

**CountToTen.mo**

</div>

``` motoko
import Counters "Counters";
import Debug "mo:base/Debug";
import Nat "mo:base/Nat";

actor CountToTen {
  public func countToTen() : async () {
    let C : Counters.Counter = await Counters.Counter(1);
    while ((await C.read()) < 10) {
      Debug.print(Nat.toText(await C.read()));
      await C.inc();
    };
  }
};
await CountToTen.countToTen()
```

The call to `Counters.Counter(1)` installs a fresh counter on the network. Installation is asynchronous, so the caller must `await` the result.

`Counters.Counter(1)`  を呼び出すと, ネットワーク上に新しいカウンタをインストールします. インストールは非同期なので, 呼び出し側は結果を`await` しなければなりません.

The type annotation `: Counters.Counter` is redundant here. It’s included only to illustrate that the type of the actor class is available when required.

型注釈 `: Counters.Counter` はここでは冗長です. アクタ・クラスの型が要求時に用意されるのを記述するときにのみ必要です.

## Importing from another canister smart contract

In addition to the examples above that import Motoko modules, you can also import actors (and their shared functions) from canister smart contract smart constracts by using the `canister smart contract:` prefix in place of the `mo:` prefix.

Motoko のモジュールをインポートする上記の例に加えて, `mo:` 前置子の代わりに `canister smart contract:` を使ってキャニスタ・スマート・コントラクト・スマート・コントラクタからアクタ (とその共有関数) をインポートすることもできます.

:::note

Unlike a Motoko library, an imported canister smart contract can be implemented in any other Internet Computer language that emits Candid interfaces for its canister smart contracts (for instance Rust). It could even be an older or newer version of Motoko.

Mokoto ライブラリとは違って, インポートされたキャニスタ・スマート・コントラクトは, そのキャニスタ・スマート・コントラクトの Candid インタフェイスを伴う Internet Computer の任意の言語 (例えば Rust) で記述されています. Motoko のバージョンさえ異なっている場合もあります.

:::

For example, you might have a project that produces the following three canister smart contracts:

例えば, 以下の三つのキャニスタ・スマート・コントラクトを作り出すプロジェクトがあるとしましょう:

-   BigMap (implemented in Rust)

-   Connectd (implemented in Motoko)

-   LinkedUp (implemented in Motoko)




-   BigMap (Rust で実装)
-   Connected (Motoko で実装)
-   LinkedUp (Motoko で実装)

These three canister smart contracts are declared in the project’s `dfx.json` configuration file and compiled by running `dfx build`.

この三つのキャニスタ・スマート・コントラクトはそのプロジェクトの `dfx.json` 構成ファイルに宣言され, `dfx build` を走らせるとコンパイルされます.

You can then use the following lines to import the `BigMap` and `Connectd` canister smart contracts as actors in the Motoko LinkedUp actor:

このとき, Motoko LinkedUp アクタで `BigMap` と `Connected` キャニスタ・スマート・コントラクトをアクタとしてインポートする以下の行を使うことができます.

``` motoko
import BigMap "canister:BigMap";
import Connectd "canister:connectd";
```

When importing canister smart contracts, it is important to note that the type for the imported canister smart contract corresponds to a **Motoko actor** instead of a **Motoko module**. This distinction can affect how some data structures are typed.

キャニスタ・スマート・コントラクトをインポートしたときには, インポートしたキャニスタ・スマート・コントラクトの型が, Motoko モジュールではなく, Motoko アクタに対応すると言うことに気を付けるのが重要です.

For the imported canister smart contract actor, types are derived from the Candid file — the *project-name*.did file — for the canister smart contract rather than from Motoko itself.

インポートされたキャニスタ・スマート・コントラクト・アクタの型は Motoko 自体ではなく, そのキャニスタ・スマート・コントラクトの Candid ファイル (_プロジェクト名_.did ファイル)  から導かれます.

The translation from Motoko actor type to Candid service type is mostly, but not entirely, one-to-one, and there are some distinct Motoko types that map to the same Candid type. For example, the Motoko `Nat32` and `Char` types both exported as Candid type `nat32`, but `nat32` is canonically imported as Motoko `Nat32`, not `Char`.

Motoko アクタ型から Candid サービス型への翻訳は, ほとんど (完全ではない) 一対一に対応していますが, Motoko の異なる型が Candid の一つの型に対応付けられることもあります. 例えば Motoko の `Nat32` と `Char` 型はともに Candid の `nat32` 型としてエクスポートされますが, `nat32` は公式には Motoko の `char` ではなく `Nat32` としてインポートされます.

The type of an imported canister smart contract function, therefore, might differ from the type of the original Motoko code that implements it. For example, if the Motoko function had type `shared Nat32 -> async Char` in the implementation, its exported Candid type would be `(nat32) -> (nat32)` but the Motoko type imported from this Candid type will actually be the correct—but perhaps unexpected—type `shared Nat32 -> async Nat32`.

したがって, インポートされたキャニスタ・スマート・コントラクト関数の型も, それを実装したもともとの Motoko コードの型とは異なっている場合があります. 例えば実装時に `shared Nat32 -> async Char` という型の Motoko の関数は, Candid の型としては `(nat32) -> (nat32)`として公開されます. しかし, この Candid 型からインポートされた Motoko 型は実際には型 `shared Nat32 -> async Nat32` となって正しいのです (多分期待したものではないでしょうが).

## Naming imported modules

Although the most common convention is to identify imported modules by the module name as illustrated in the examples above, there’s no requirement for you to do so. For example, you might want to use different names to avoid naming conflicts or to simplify the naming scheme.

インポートされたモジュールを上の例題で説明したようなモジュール名で識別するのはほとんどのよく使われる習慣でしょうが, そうしなければならない必要があるわけではありません. 例えば, 名前の衝突を避けたり, 名前付けの図式を単純化するために別の名前を使っても構いません.

The following examples illustrate different names you might use when importing the `List` base library module, avoiding a clash with another `List` library from a fictional `collections` package.

以下の例では, ベース・ライブラリの  `List` モジュールをインポートするときに `collection` パッケージ (このパッケージは実在しません) の別の `List` ライブラリとぶつかるのを避けるために別の名前を使っています.

``` motoko
import List "mo:base/List:";
import Sequence "mo:collections/List";
import L "mo:base/List";
```

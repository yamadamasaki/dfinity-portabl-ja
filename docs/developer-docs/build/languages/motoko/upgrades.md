# Stable variables and upgrade methods

One key feature of the Internet Computer is its ability to persist canister smart contract state using WebAssembly memory and globals rather than a traditional database. This means that that the entire state of a canister smart contract is magically restored before, and saved after, each message, without explicit user instruction. This automatic and user-transparent preservation of state is called *orthogonal persistence*.

Internet Computer の鍵となるフィーチャの一つは, キャニスタ・スマート・コントラクトの状態を伝統的なデータベースにではなく,  WebAssembly のメモリとグローバルを使って永続化する能力です. つまり, 明示的にユーザが指示しなくても魔法のように, メッセージを受け取る前には状態が復元され, メッセージを処理した後には保存されるのです. 自動的でユーザ透過な (永続的な) 状態は _直交永続性_ とも呼ばれています.

Though convenient, orthogonal persistence poses a challenge when it comes to upgrading the code of a canister smart contract. Without an explicit representation of the canister smart contract’s state, how does one tranfer any application data from the retired canister smart contract to its replacement?

直交性属性は便利ではありますが, いざキャニスタ・スマート・コントラクトのコードをアップグレードしようとすると, ある挑戦が待ち構えています. キャニスタ・スマート・コントラクトの状態の明示的な表現がないのに, どうやって任意のアプリケーション・データを古いキャニスタ・スマート・コントラクトから新しいキャニスタ・スマート・コントラクトに移動すれば良いのでしょうか?

Accommodating upgrades without data loss requires some new facility to *migrate* a canister smart contract’s crucial data to the upgraded canister smart contract. For example, if you want to deploy a new version of a user-registration canister smart contract to fix an issue or add functionality, you need to ensure that existing registrations survive the upgrade process.

データ損失なくアップグレードを取り込むには, キャニスタ・スマート・コントラクトの極めて重要なデータをアップグレードしたキャニスタ・スマート・コントラクトに _移行_ するための何らかの新しい仕組みが必要になります. 例えば, あなたがユーザを登録するキャニスタ・スマート・コントラクトの問題点を解決したり, 機能を付け加えるために新しいヴァージョンを配備しようとしたとしましょう. このとき, アップグレードの過程で既存の登録がちゃんと存続することを保証しなければなりません.

The Internet Computer's persistence model allows a canister smart contract to save and restore such data to dedicated *stable memory* that, unlike ordinary canister smart contract memory, is retained across an upgrade, allowing a canister smart contract to transfer data in bulk to its replacement canister smart contract.

Internet Computer の永続化モデルでは, キャニスタ・スマート・コントラクトがそのようなデータを (通常のキャニスタ・スマート・コントラクトのメモリとは別の) 専用の _安定メモリ_ に保存/読み出しできる, すなわちアップグレードを越えて保存され, キャニスタ・スマート・コントラクトがデータをまとめて新たに置き換えられるキャニスタ・スマート・コントラクトに移動できるようになっています. 

For applications written in Motoko, the language provides high-level support for preserving state that leverages Internet Computer stable memory. This higher-level feature, called *stable storage*, is designed to accommodate changes to both the application data and to the Motoko compiler used to produce the application code.

Motoko で記述されたアプリケーションでは, 状態を Internet Computer の安定メモリを利用して保存できるような高レベルのサポートを言語が提供しています.

Utilizing stable storage depends on you — as the application programmer — anticipating and indicating the data you want to retain after an upgrade. Depending on the application, the data you decide to persist might be some, all, or none of a given actor’s state.

安定メモリを活用し, アップグレード後も残しておきたいデータを見越して, 指定するかどうかは (アプリケーション・プログラマである) あなた次第です. あなたが永続化を決めるデータがアクタの状態の一部なのか, 全部なのか, なくていいのかは, アプリケーションによって異なります.

## Declaring stable variables

In an actor, you can nominate a variable for stable storage (in Internet Computer stable memory) by using the `stable` keyword as a modifier in the variable’s declaration.

アクタの変数を安定ストレージ (Internet Computer の安定メモリ) に入れたいときには, その変数宣言の修飾子に `stable` キーワードを使います.

More precisely, every `let` and `var` variable declaration in an actor can specify whether the variable is `stable` or `flexible`. If you don’t provide a modifier, the variable is declared as `flexible` by default.

もっと正確に言うと, アクタのすべての `let` , `var` 変数宣言では, その変数が `stable` か `flexible` であるかを指定することができます. 修飾子を付けなければ, その変数はデフォルトで `flexible` であるということになります.

The following is a simple example of how to declare a stable counter that can be upgraded while preserving the counter’s value:

次の簡単な例題では, カウンタの値をアップグレード中も保存できるような安定カウンタを宣言しています:

``` motoko
actor Counter {

  stable var value = 0;

  public func inc() : async Nat {
    value += 1;
    return value;
  };
}
```

:::note

You can only use the `stable` or `flexible` modifier on `let` and `var` declarations that are **actor fields**. You cannot use these modifiers anywhere else in your program.

`stable`  `flexible` 修飾子を付けられるのは, **アクタのフィールド** の `let` `var` 宣言のみです. プログラムの他の部分で使うことはできません.

:::

## Typing

Because the compiler must ensure that stable variables are both compatible with and meaningful in the replacement program after an upgrade, the following type restrictions apply to stable state:

コンパイラは, 安定変数がアップグレード後も新しいプログラムと互換性があり, 有意味であることを保証しなければなりません. そのために安定状態には以下の型制約が課せられます.

-   every `stable` variable must have a *stable* type



-   すべての `stable` 変数は stable な型を持たなければならない

where a type is *stable* if the type obtained by ignoring any `var` modifiers within it is *shared*.

ここで, ある型が _stable_ であるとは, その型の中のすべての `var` 修飾子を無視したときに _shared_ になることです.

Thus the only difference between stable types and shared types is the former’s support for mutation. Like shared types, stable types are restricted to first-order data, excluding local functions and structures built from local functions (such as objects). This exclusion of functions is required because the meaning of a function value — consisting of both data and code — cannot easily be preserved across an upgrade, while the meaning of plain data — mutable or not — can be.

したがって, 安定型と共有型の違いは, 安定型では可変性をサポートしていることになります. 共有型と同様, 安定型は (ローカル関数とローカル関数から構成される構造 (オブジェクトなど) を除いて) 一級データに限定されます. 関数を除外する必要があるのは, (データとコードから成る) 関数値の意味をアップグレードを跨いで保存するのは簡単なことではないからです  (単なるデータの意味を保存するのは (可変であれ不変であれ) 可能なのですが).

:::note

In general, object types are not stable because they can contain local functions. However, a plain record of stable data is a special case of object types that is stable. Moreover, references to actors and shared functions are also stable, allowing you to preserve their values across upgrades. For example, you can preserve state recording a set of actors or shared function callbacks subscribing to a service.

オブジェクト型は一般にローカル関数を含むので安定ではありません. しかし, 安定データから成る普通のレコードは, オブジェクト型が安定になる特殊ケースです. さらにアクタや共有型への参照も安定なので, アップグレードを越えて変数値を保存します. 例えば, あるサービスを購読するコールバック共有関数やアクタの集合を記録する状態は保存可能です.

:::

## How stable variables are upgraded

When you first compile and deploy a canister smart contract, all flexible and stable variables in the actor are initialized in sequence. When you deploy a canister smart contract using the `upgrade` mode, all stable variables that existed in the previous version of the actor are pre-initialized with their old values. After the stable variables are initialized with their previous values, the remaining flexible and newly-added stable variables are initialized in sequence.

最初にあるキャニスタ・スマート・コントラクトをコンパイルし, 配備するときには, そのアクタのすべての柔軟/安定変数は順に初期化されます. あるキャニスタ・スマート・コントラクトを `upgrade` モードで配備するときには, そのアクタの前のヴァージョンに存在するすべての安定変数はその古い値 (最初の初期値でもなく, 動いていたときの最後の値) で (前) 初期化されます. 安定変数が古い値で初期化された後に, 柔軟変数と新しく追加された安定変数が順に初期化されます.

## Preupgrade and postupgrade system methods

Declaring a variable to be `stable` requires its type to be stable too. Since not all types are stable, some variables cannot be declared `stable`.

ある変数を `stable` と宣言するには, その型も安定でなければ成りません. すべての型が安定ではないので, `stable` と宣言できない変数も出てきます.

As a simple example, consider the Registry\` actor from the discussion of [orthogonal persistence](./index.md#orthogonal-persistence).

例として, [直交永続性](./index.md#orthogonal-persistence) にも出てきた `Registry` アクタを見てみましょう.

``` motoko
import Text "mo:base/Text";
import Map "mo:base/HashMap";

actor Registry {

  let map = Map.HashMap<Text, Nat>(10, Text.equal, Text.hash);

  public func register(name : Text) : async () {
    switch (map.get(name)) {
      case null {
        map.put(name, map.size());
      };
      case (?id) { };
    }
  };

  public func lookup(name : Text) : async ?Nat {
    map.get(name);
  };
};

await Registry.register("hello");
(await Registry.lookup("hello"), await Registry.lookup("world"))
```

This actor assigns sequential identifiers to `Text` values, using the size of the underlying `map` object to determine the next identifier. Like other actors, it relies on *orthogonal persistence* to maintain the state of the hashmap between calls.

このアクタでは, Text 値に順に (`map` オブジェクトの大きさで次の id を決めつつ) id を振っています. 他のアクタと同様, これは HashMap の状態が呼び出しを跨いで保持されるという _直交永続性_ に依存しています.

We’d like to make the `Register` upgradable, without the upgrade losing any existing registrations.

この `Registry` を, 既存の登録を残したままアップグレード可能にしたいとしましょう. __NOTE: __多分 Register ではなく Registry

Unfortunately, its state, `map`, has a proper object type that contains member functions (for example, `map.get`), so the `map` variable cannot, itself, be declared `stable`.

残念なことにこの状態 `map` はメンバ関数 (例えば `map.get`) を持つ本来のオブジェクト型です. したがって, `map` 変数は `stable` 宣言することができません.

For scenarios like this that can’t be solved using stable variables alone, Motoko supports user-defined upgrade hooks that, when provided, run immediately before and after upgrade. These upgrade hooks allow you to migrate state between unrestricted flexible variables to more restricted stable variables. These hooks are declared as `system` functions with special names, `preugrade` and `postupgrade`. Both functions must have type `: () → ()`.

このようなシナリオでは安定変数を使うだけでは問題を解決できませんが, Motoko ではユーザがアップグレードのフック (引っ掛け) を定義して, (もしあれば) アップグレードの前後でそのフックを動かすようにできます. このアップグレード・フックでは, 制約のない柔軟変数を制約のある安定変数に移行することもできます. これらのフックは `preupgrade` と `postupgrade` という特別な名前をもつ `system` 関数として宣言されています. どちらの関数も型は `: () → ()` です.

The `preupgrade` method lets you make a final update to stable variables, before the runtime commits their values to Internet Computer stable memory, and performs an upgrade. The `postupgrade` method is run after an upgrade has initialized the replacement actor, including its stable variables, but before executing any shared function call (or message) on that actor.

`preupgrade` メソッドでは, ランタイムが安定変数の値を Internet Computer の安定メモリにコミットし, アップグレードを実施する前に安定変数に最後の変更をかけられます. `postupgrade` メソッドは, アップグレードが新しくなったアクタを初期化 (安定変数の前初期化を含む) した後, ただしそのアクタに対する共有関数呼び出し (メッセージ送信) を実行する前に走ります.

Here, we introduce a new stable variable, `entries`, to save and restore the entries of the unstable hash table.

ここでは, 新しい安定変数 `entries` を導入して,  安定ではないハッシュテーブルのエントリをここに保存して, アップグレード後に戻すようにしましょう.

``` motoko
import Text "mo:base/Text";
import Map "mo:base/HashMap";
import Array "mo:base/Array";
import Iter "mo:base/Iter";

actor Registry {

  stable var entries : [(Text, Nat)] = [];

  let map = Map.fromIter<Text,Nat>(
    entries.vals(), 10, Text.equal, Text.hash);

  public func register(name : Text) : async () {
    switch (map.get(name)) {
      case null  {
        map.put(name, map.size());
      };
      case (?id) { };
    }
  };

  public func lookup(name : Text) : async ?Nat {
    map.get(name);
  };

  system func preupgrade() {
    entries := Iter.toArray(map.entries());
  };

  system func postupgrade() {
    entries := [];
  };
}
```

Note that the type of `entries`, being just an array of `Text` and `Nat` pairs, is indeed a stable type.

`entries` の型は `Text` と `Nat` の二つ組みの配列で, まさに安定型になっていることに注意してください.

In this example, the `preupgrade` system method simply writes the current `map` entries to `entries` before `entries` is saved to stable memory. The `postupgrade` system method resets `entries` to the empty array after `map` has been populated from `entries` to free space.

この例では, `preupgrade` システム・メソッドで `entries` が安定メモリに保存される前に現在の `map` の各項目を `entries` に書き込んでいます. また `postupgrade` システム・メソッドでは, `entries`  を `map` に書き戻した後, `entries` をリセットして空き空間にしています.

## Stable type signatures

The collection of stable variable declarations in an actor can be summarized in a *stable signature*.

アクタの安定変数宣言の集まりは, _安定シグネチャ_ としてまとめることができます.

The textual representation of an actor’s stable signature resembles the internals of a Motoko actor type:

アクタの安定シグネチャのテキスト表現は, Motoko アクタ型の内部に似ています:

``` motoko
actor {
  stable x : Nat;
  stable var y : Int;
  stable z : [var Nat];
};
```

It specifies the names, types and mutability of the actor’s stable fields, possibly preceded by relevant Motoko type declarations.

アクタの安定フィールドの名前, 型, 可変性を指定し, 場合によって前に Motoko の関連する型宣言を付けます.

:::tip

You can emit the stable signature of the main actor or actor class to a `.most` file using `moc` compiler option `--stable-types`. You should never need to author your own `.most` file.

`moc` コンパイラのオプション `--stable-types` を使って, メインのアクタ, あるいはアクタ・クラスの安定シグネチャを `.most` ファイルに書き出すことができます.

:::

A stable signature `<stab-sig1>` is *stable-compatible* with signature `<stab-sig2>`, if, and only,

安定シグネチャ `<stab-sig1>` がシグネチャ `<stab-sig2>` と _安定互換_ であるとは以下のとき, またそのときのみである

-   every immutable field `stable <id> : T` in `<stab-sig1>` has a matching field `stable <id> : U` in `<stab-sig2>` with `T <: U`.

-   every mutable field `stable var <id> : T` in `<stab-sig1>` has a matching field `stable var <id> : U` in `<stab-sig2>` with `T <: U`.




-   `<stab-sig1>` の各不変フィールド `stable <id> : T` が `<stab-sig2>` に対応するフィールド `stable <id> : U` を持ち, `T <: U` である
-   `<stab-sig1>` の各可変フィールド `stable var <id> : T` が `<stab-sig2>` に対応するフィールド `stable var <id> : U` を持ち, `T <: U` である

Note that `<stab-sig2>` may contain additional fields. Typically, `<stab-sig1>` is the signature of an older version while `<stab-sig2>` is the signature of a newer version.

`<stab-sig2>` には, `<stab-sig1>` にないフィールドもあり得ることに注意. 典型的には `<stab-sig1>` が古いヴァージョンのシグネチャで, `<stab-sig2>` が新しいヴァージョンのシグネチャと考えられます.

The subtyping condition on stable fields ensures that the final value of some field can be consumed as the initial value of that field in the upgraded code.

安定フィールドの副型条件は, あるフィールドの最終値がアップグレードされたコードのそのフィールドの初期値になることを保証しています.

:::tip

You can check the stable-compatiblity of two `.most` files, `cur.most` and `nxt.most` (containing stable signatures), using `moc` compiler option `--stable-compatible cur.most nxt.most`.

`moc` コンパイラのオプション `--stable-compatible cur.most nxt.most` を使えば  ` cur.most` と `next.most` という二つの `.most` ファイル (安定シグネチャが入っている) の安定互換性をチェックできます.

:::

:::note

The *stable-compatible* relation is quite conservative. In the future, it may be relaxed to accommodate a change in field mutability and/or abandoning fields from `<stab-sig1>` (but with a warning).

この _安定互換_ 関係はとても保守的になっています. 将来的には, フィールドの可変性と `<stab-sig1>` のフィールドの削除 (警告は出す) の変更は取り込みたいと思っています.

:::

## Upgrade safety

Before upgrading a deployed canister smart contract, you should ensure that the upgrade is safe and will not

配備されているキャニスタ・スマート・コントラクトをアップグレードする前に, アップグレードが安全であることを保証するためには以下のことをしてはならない

-   break existing clients (due to a Candid interface change); or

-   discard Motoko stable state (due to an incompatible change in stable declarations).



-   既存のクライアントを壊す (Candid インタフェイスが変わってしまうため) ; あるいは
-   Motoko の安定状態を棄てる (安定宣言が非互換的に変わってしまうため)

A Motoko canister smart contract upgrade is safe provided:

Motoko キャニスタ・スマート・コントラクトを安全にアップグレードするには:

-   the canister smart contract’s Candid interface evolves to a Candid subtype; and

-   the canister smart contract’s Motoko stable signature evolves to a *stable-compatible* one.



-   キャニスタ・スマート・コントラクトの Candid インタフェイスを副型にする方向で進化させる; かつ
-   キャニスタ・スマート・コントラクトの Motoko 安全シグネチャを _安全互換_ なように進化させる

Upgrade safety does not guarantee that the upgrade process will succeed (it can still fail due to resource constraints). However, it should at least ensure that a successful upgrade will not break Candid type compatibility with existing clients or unexpectedly lose data that was marked `stable`.

アップグレードの安全性はアップグレード過程がうまく行くことを保証するものではありません. 資源制約で失敗することはあります. しかし, アップグレードがうまく行ったとしたら少なくとも, Candid 型は既存のクライアントと互換性を壊していないこと, `stable` の印を付けたデータが予期せず失われてしまうようなことはないことは保証されます.

:::tip

You can check valid Candid subtyping between two services described in `.did` files, `cur.did` and `nxt.did` (containing Candid types), using the `didc` tool with argument `check nxt.did cur.did`. The `didc` tool is available at <https://github.com/dfinity/candid>.

`cur.did` と `nxt.did` という `.did` ファイル (Candid 型が入っている) で記述された二つのサービス間の Candid 副型付けが妥当かどうかは, `didc` ツールに引数 `check nxt.did cur.did` を与えると確かめることができます. `didc` ツールは <https://github.com/dfinity/candid> にあります.

:::

## Metadata sections

The Motoko compiler embeds the Candid interface and stable signature of a canister smart contract as canister smart contract metadata, recorded in additional Wasm custom sections of a compiled binary.

Motoko コンパイラは, Candid インタフェイスとキャニスタ・スマート・コントラクトの安定シグネチャをキャニスタ・スマート・コントラクトのメタデータとして埋め込み, コンパイルされた Wasm のカスタム・セクションを追加して記録します.

This metadata can be selectively exposed by the Internet Computer and used by tools such as `dfx` to verify upgrade compatibility.

Internet Computer はこのメタデータを選択的に公開でき, `dfx` のようなツールを使ってアップグレードの互換性をチェックできます.

## Upgrading a deployed actor or canister smart contract

After you have deployed a Motoko actor with the appropriate `stable` variables or `preupgrade` and `postupgrade` system methods, you can use the `dfx canister smart contract install` command with the `--mode=upgrade` option to upgrade an already deployed version. For information about upgrading a deployed canister smart contract, see [Upgrade a canister smart contract](../../project-setup/manage-canister smart contracts.md#upgrade-a-canister smart contract).

Motoko アクタを適切な `stable` 変数, システム・メソッドの `preupgrade`, `postupgrade` を使って配備した後, `--mode=upgrade` オプションを付けた `dfx canister install` コマンドを使えば , 既に配備されているヴァージョンをにアップグレードすることができます. 配備されているキャニスタ・スマート・コントラクトのアップグレードのやり方は [キャニスタ・スマート・コントラクトのアップグレード](../../project-setup/manage-canisters.md#upgrade-a-canister) にあります

An upcoming version of `dfx` will, if appropriate, check the safety of an upgrade by comparing the Candid and (for Motoko canister smart contracts only) the stable signatures embedded in the deployed binary and upgrade binary, and abort the upgrade request when unsafe.

`dfx` の次のヴァージョンでは, Candid と (Motoko キャニスタ・スマート・コントラクトの場合は) 配備されたバイナリとアップグレードするバイナリにそれぞれ埋め込まれた安定シグネチャを比較して, アップグレードの安全性をチェックし, 安全でない場合にはアップグレード要求を中断することができるようになります.

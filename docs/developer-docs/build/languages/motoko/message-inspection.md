# Message inspection

On the Internet Computer, a canister can selectively *inspect* and *accept* or *decline* ingress messages submitted through the HTTP interface:

Internet Computer では, キャニスタが HTTP インタフェイス経由で入ってくるメッセージを選択的に監査して,  _受諾/拒絶_ できます.

> A canister can inspect ingress messages before executing them. When the IC receives an update call from a user, the IC will use the canister method `canister_inspect_message` to determine whether the message shall be accepted. If the canister is empty (i.e. does not have a Wasm module), then the ingress message will be rejected. If the canister is not empty and does not implement `canister_inspect_message`, then the ingress message will be accepted.
>
> キャニスタは, 実行前に入ってくるメッセージを監査することができます. IC がユーザから更新呼び出しを受け取ると, IC はキャニスタのメソッド `canister_inspect_message` を使って, メッセージを受け取るかどうかを決めます. キャニスタが空の (すなわち Wasm モジュールを持っていない) ときには, 入って来たメッセージは拒絶されます. キャニスタが空でなく, `canister_inspect_message` を実装していなければ, 入って来たメッセージは受け入れられます.
>
> In `canister_inspect_message`, the canister can accept the message by invoking `ic0.accept_message : () → ()`. This function traps if invoked twice. If the canister traps in `canister_inspect_message` or does not call `ic0.accept_message`, then the access is denied.
>
> キャニスタは `canister_inspect_message` 内で  `ic0.accept_message : () → ()` を起動することでメッセージを受け入れることができます. 二度起動されるとこの関数はトラップを起こします. キャニスタが `canister_inspect_message` でトラップしたり, `ic0.accept_message` を呼び出さなければ, アクセスは拒否されます.
>
> The `canister_inspect_message` is *not* invoked for HTTP query calls, inter-canister calls or calls to the management canister.
>
> `canister_inspect_message` は HTTP の問い合わせ呼び出し, キャニスタ間呼び出し, 管理キャニスタに対する呼び出しに対しては起動され _ません_.
>
> —  [IC Interface Specification](https://smartcontracts.org/docs/current/references/ic-interface-spec/#ingress-message-inspection)

Message inspection mitigates some denial of service attacks, designed to drain canisters of cycles by placing unsolicited free calls.

メッセージ監査は, 望まざる無料呼び出しによってキャニスタのサイクルを枯渇させるようなサービス攻撃を軽減します.

:::note

You can think of method inspection as providing the "Collect call from *name*. Do you accept charges?" prologue of an old-fashioned, operator-assisted, collect phone call.

メソッド監査は「_誰々_ さんからコレクトコールです. お出になりますか」という, 昔風の交換手が出てくる電話のコレクト・コールの前置きのようなものだと考えてください.

:::

In Motoko, actors can elect to inspect and accept or decline ingress messages by declaring a particular `system` function called `inspect`. Given a record of message attributes, this function produces a `Bool` that indicates whether to accept or decline the message by returning `true` or `false`. The function is invoked (by the system) on each ingress message. Similar to a query, any side-effects of an invocation are discarded and transient. A call that traps due to some fault has the same result as returning `false` (message declination).

Motoko では, アクタは `inspect` という特別な `system` 関数を宣言することで, 入ってくるメッセージを監査して, そのメッセージを受けれるかどうかを選択することができます. 与えられたメッセージ属性のレコードに対して, この関数は `Bool` 値  `true` か `false` を返すことで, そのメッセージを許諾するか, 拒絶するかを示します. この関数は, 内向きのメッセージ毎に (システムによって) 起動されます. 問い合わせと同じように, その起動による副作用はすべて棄てられ, 一時的なものになります. 何らかの失敗によってトラップした呼び出しは, `false` を返すのと同じ結果 (メッセージ拒絶) になります.

Unlike other system functions, that have a fixed argument type, the argument type of `inspect` depends on the interface of the enclosing actor. In particular, the formal argument of `inspect` is a record of fields of the following types:

他のシステム関数とは異なり, この関数は決まった引数型を持ちますが, `inspect` の引数の型はそのアクタを囲むインタフェイスによって変わってきます.

-   `caller : Principal`: the principal, possibly anonymous, of the caller of the message;

-   `arg : Blob`: the raw, binary content of the message argument;

-   `msg : <variant>`: a variant of *decoding* functions, where `<variant> == {…​; #<id>: () → T; …​}` contains one variant per shared function, `<id>`, of the actor. The variant’s tag identifies the function to be called; The variant’s argument is a function that, when applied, returns the (decoded) argument of the call as a value of type `T`.




-   `caller : Principal`: そのメッセージの呼び出し元のプリンシパル (匿名の場合もある)
-   `arg : Blob`: メッセージ引数の生なバイナリ表現
-   `msg: <variant>`: `復号` 関数 (たち) のヴァリアント (`<variant> == {…; #<id>: () → T; …}`) で, アクタの共有関数 (`<id>`) 毎に一つのヴァリアントがある. ヴァリアントのタグで呼ばれるべき関数を同定する. ヴァリアントの引数は関数で, 適用するとその呼び出しの (復号された) 引数の値 (型 `T`) を返す

Using a variant, tagged with `#<id>`, allows the return type, `T`, of the decoding function to vary with the argument type (also `T`) of the shared function `<id>`.

`#<id>` でタグ付けされたヴァリアントを用いて, 復号関数は型 `T` を返すことができますが, この型は共有関数 `<id>` の引数の型 (これも `T`) に応じて変わります.

The variant’s argument is a function so that one can avoid the expense of message decoding (when appropriate).

このヴァリアントの引数は関数なので, (それが適切であれば) メッセージ復号の手間を避けることもできます.

Exploiting subtyping, the formal argument can omit record fields it does not require, or selectively ignore the arguments of particular shared functions, for example, in order to simply dispatch on the name of a function without inspecting its actual argument.

副型付けを活用し, 仮引数で必要ではないレコードを省いたり, 特定の共有関数の引数を選んで無視したりして, 関数を監査せずにその関数の名前を処理しても構いません.

:::note

Confusingly, a `shared query` function *can* be called using a regular HTTP call to obtain a certified response: this is why the variant type also includes `shared query` functions.

紛らわしいですが, `共有問い合わせ` 関数でも, 認証された応答を得るために (敢えて) 通常の HTTP 経由で呼び出しすることもできます. ヴァリアント型に `共有問い合わせ` 間も含まれているのはそのためです.

:::

:::danger

An actor that fails to declare system field `inspect` will simply accept all ingress messages.

`inspect` システム・フィールドを宣言し損なっていると, すべての内向きのメッセージを受け取ることになってしまいます.

:::

:::danger

System function `inspect` should **not** be used for definitive access control. This is because `inspect` is executed by a single replica, without full consensus, and its result could be spoofed by a malicious boundary node. Reliable access control checks can only be performed within the `shared` functions guarded by `inspect`.

システム関数 `inspect` を最終的なアクセス制御に用いては **いけません**. なぜならば `inspect` は完全な同意ではなく一つのレプリカで実行され, その結果は悪意のある境界ノードによってなりすまされる可能性があるからです. 信頼のおけるアクセス制御の検査は `inspect` によって保護された `共有` 関数内部でのみ実行可能です.

:::

## Example

A simple, contrived example of method inspection is a counter actor, that inspects some of its messages in detail, and others only superficially:

カウンタ・アクタの簡単で不自然なメソッド監査の例です. この例ではいくつかのメッセージは詳細に監査し, 他は表面的な監査をしています.

``` motoko
import Principal = "mo:base/Principal";

actor {

   var c = 0;

   public func inc() : async () { c += 1 };
   public func set(n : Nat) : async () { c := n };
   public query func read() : async Nat { c };
   public func reset() : () { c := 0 }; // oneway

   system func inspect(
     {
       caller : Principal;
       arg : Blob;
       msg : {
         #inc : () -> ();
         #set : () -> Nat;
         #read : () -> ();
         #reset : () -> ();
       }
     }) : Bool {
    if (Principal.isAnonymous(caller)) return false;
    if (arg.size() > 512) return false;
    switch (msg) {
      case (#inc _) { true };
      case (#set n) { n() != 13 };
      case (#read _) { true };
      case (#reset _) { false };
    }
  }
};
```

Note that, due to subtyping, all of the following variations, in order of increasing argument specificity, are legal definitions of `inspect`.

副型付けのせいで, 以下のヴァリエーションでは, 引数の特異性が増す順になっており, これは `inspect` の定義としては問題ありません (__NOTE:__ `case _` のパタン・マッチングの出現順のことだと思われます).

Blanket denial of all ingress messages, ignoring further information:

すべてのメッセージを, その情報に拘わらず一律拒絶:

``` motoko no-repl
   system func inspect({}) : Bool { false }
```

Declining anonymous calls:

匿名呼び出しを:

``` motoko no-repl
   system func inspect({ caller : Principal }) : Bool {
     not (Principal.isAnonymous(caller));
   }
```

Declining large messages, based on \`arg’s size (in bytes).

`引数` の大きさ (単位はバイト) に基づいて, 大きなメッセージを拒絶:

``` motoko no-repl
   system func inspect({ arg : Blob }) : Bool {
     arg.size() <= 512;
   }
```

Declining messages by name only, ignoring message arguments (note the use of type `Any` as message argument variants):

メッセージを, 引数を無視して名前だけで拒絶 (メッセージの引数のヴァリアントとして, 型 `Any` を利用していることに注意):

``` motoko no-repl
   system func inspect(
     {
       msg : {
         #inc : Any;
         #set : Any;
         #read : Any;
         #reset : Any;
       }
     }) : Bool {
    switch (msg) {
      case ((#set _) or (#reset _)) { false };
      case _ { true }; // allow inc and read
    }
  }
```

A combination of the previous three, specifying the argument types of some variants while ignoring others at type `Any` and using pattern matching to conflate identical cases.

ここまでの三つの組み合わせ. いくつかは引数型を指定. 他は型を `Any` として, 同様の場合をパタン・マッチングでまとめている.

``` motoko no-repl
   system func inspect(
     {
       caller : Principal;
       arg : Blob;
       msg : {
         #inc : Any;
         #set : () -> Nat;
         #read : Any;
         #reset : Any;
       }
     }) : Bool {
    if (Principal.isAnonymous(caller)) return false;
    if (arg.size() > 512) return false;
    switch (msg) {
      case (#set n) { n() != 13 };
      case (#reset _) { false };
      case _ { true }; // allow inc and read
    }
  }
```

## Tips on authoring `inspect`

Implementing `inspect` after the fact, once all shared functions of an actor have already been implemented, can be tedious, since you’ll need to declare a correctly typed variant for each shared function. A simple trick is to first implement the function *incorrectly*, with a `()` argument, compile the code, then use the compiler’s error message to obtain the required argument type.

アクタのすべての共有関数をいったん実装し終えてから, さらに `inspect` を実装するのは面倒に思うかもしれません. しかし, 共有関数毎に正しい型付けのヴァリアントを宣言する必要があります. 簡単な裏技として, まず関数を `()` 引数で間違った実装をし, コードをコンパイルし, コンパイラのエラー・メッセージを利用して必要な引数型を取得する, というやり方もあります.

For example, in the actor from the previous section, incorrectly declaring:

例えば, 前の節のアクタで, このように (間違って) 宣言し:

``` motoko no-repl
   system func inspect() : Bool {
     false
   }
```

forces the compiler to report the expected type below:

コンパイラに正しい型を報告させ:

``` motoko no-repl
Inspect.mo:13.4-15.5: type error [M0127], system function inspect is declared with type
  () -> Bool
instead of expected type
  {
    arg : Blob;
    caller : Principal;
    msg :
      {
        #inc : () -> ();
        #read : () -> ();
        #reset : () -> ();
        #set : () -> Nat
      }
  } -> Bool
```

which you can now cut-and-paste into your code.

これを自分のコードに切り貼りするのもありですね.
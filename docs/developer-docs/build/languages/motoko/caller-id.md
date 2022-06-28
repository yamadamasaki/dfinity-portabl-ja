# Principals and caller identification

Motoko’s shared functions support a simple form of caller identification that allows you to inspect the Internet Computer **principal** associated with the caller of a function. The principal associated with a call is a value that identifies a unique user or canister smart contract.

Motoko の共有関数は, 簡単なかたちの呼び出し元識別 (関数の呼び出し元に関連付けられている Internet Computer の **principal** を調べられるようにする) をサポートしています. 呼び出しに関連付けられたプリンシパルは 一意なユーザ, あるいはキャニスタ・スマート・コントラクトを識別する値です.

You can use the **principal** associated with the caller of a function to implement a basic form of *access-control* in your program.

関数の呼び出し元に関連付けられた **principal** を使えば, プログラム中で基本的なかたちの _アクセス制御_ を実装することができます.

In Motoko, the `shared` keyword is used to declare a shared function. The shared function can also declare an optional parameter of type `{caller : Principal}`.

Motoko では `shared` キーワードを用いて共有関数を宣言します. 共有関数では, オプションとして型 {caller : Principal} のパラメタを宣言することもできます.

To illustrate how to access the caller of a shared function, consider the following:

共有関数の呼び出し元にどうやってアクセスするかを説明するので, 以下を見てください:

``` motoko
shared(msg) func inc() : async () {
  // ... msg.caller ...
}
```

In this example, the shared function `inc()` specifies a `msg` parameter, a record, and the `msg.caller` accesses the principal field of `msg`.

この例では, 共有関数 `inc()` は `msg` パラメタ (レコード) を指定し, `msg.caller` は `msg` の principal フィールドにアクセスします.

The calls to the `inc()` function do not change — at each call site, the caller’s principal is provided by the system, not the user — so the principal cannot be forged or spoofed by a malicious user.

`inc()` 関数への呼びだし方は変わりません. 呼び出し毎に, (ユーザではなく) システムが呼び出し元のプリンシパルを用意するので, 悪意のあるユーザがプリンシパルを偽造したり, 改変したりすることはできません.

To access the caller of an actor class constructor, you use the same (optional) syntax on the actor class declaration. For example:

アクタ・クラスの構成子の呼び出し元にアクセスするには, アクタ・クラス宣言の同様の構文を使います (オプションです). 例えば:

``` motoko
shared(msg) actor class Counter(init : Nat) {
  // ... msg.caller ...
}
```

To extend this example, assume you want to restrict the `Counter` actor so it can only be modified by the installer of the `Counter`. To do this, you can record the principal that installed the actor by binding it to an `owner` variable. You can then check that the caller of each method is equal to `owner` like this:

この例題を拡張して, `Counter` をインストールしたひとだけがカウンタを更新できるように `Counter` アクタを制限したいと思ったとしましょう. そのためには, このアクタをインストールしたプリンシパルを, `onwer` 変数に束縛して記録しておきます. そうすれば各メソッドが呼ばれたときに, 呼び出したひとが `owner` と同じかどうかをチェックできます. こんな風に:

``` motoko
shared(msg) actor class Counter(init : Nat) {

  let owner = msg.caller;

  var count = init;

  public shared(msg) func inc() : async () {
    assert (owner == msg.caller);
    count += 1;
  };

  public func read() : async Nat {
    count
  };

  public shared(msg) func bump() : async Nat {
    assert (owner == msg.caller);
    count := 1;
    count;
  };
}
```

In this example, the `assert (owner == msg.caller)` expression causes the functions `inc()` and `bump()` to trap if the call is unauthorized, preventing any modification of the `count` variable while the `read()` function permits any caller.

この例では, 呼び出しが権限のないものだった場合には `assert (owner == msg.caller)` 式が関数 `inc()`, `bump()` でトラップを起こし, `count` 変数が変更されるのを防止します (`read()` 関数は誰でも呼べます).

The argument to `shared` is just a pattern, so, if you prefer, you can also rewrite the above to use pattern matching:

`shared` への引数はただのパタンで, もししたければ, 上の例をパタン・マッチングを使って書き直すこともできます:

``` motoko
shared({caller = owner}) actor class Counter(init : Nat) {

  var count : Nat = init;

  public shared({caller}) func inc() : async () {
    assert (owner == caller);
    count += 1;
  };

  // ...
}
```

:::note

Simple actor declarations do not let you access their installer. If you need access to the installer of an actor, rewrite the actor declaration as a zero-argument actor class instead.

単純なアクタ宣言ではインストールしたひとにアクセスすることはできません. アクタのインストーラにアクセスしたければ, アクタ宣言を書き直して, 引数無しのアクタ・クラスにします.

:::

Principals support equality, ordering, and hashing, so you can efficiently store principals in containers, for example, to maintain an allow or deny list. More operations on principals are available in [Principal](../../../../references/motoko-ref/principal) base library.

プリンシパルは, 同等性, 順序づけ, ハッシングをサポートしているので, プリンシパルをコンテナに効率よく格納して, (例えば) 許可/不許可リストを管理することができます.

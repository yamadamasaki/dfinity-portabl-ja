# Actors and async data

The programming model of the Internet Computer consists of memory-isolated canister smart contracts communicating by asynchronous message passing of binary data encoding Candid values. A canister smart contract processes its messages one-at-a-time, preventing race conditions. A canister smart contract uses call-backs to register what needs to be done with the result of any inter-canister smart contract messages it issues.

Internet Computer のプログラミング・モデルは, Candid の値をコード化したバイナリ・データの非同期メッセージ交換によって通信する, メモリ隔離されたキャニスタ・スマート・コントラクトから成っています. 一つのキャニスタ・スマート・コントラクトは, 一度に1つのメッセージを処理し, 競合状態を回避します. キャニスタ・スマート・コントラクトは, 自分が発したキャニスタ・スマート・コントラクト間メッセージの戻り値を元に次に何をすべきかをコールバックとして登録します.

Motoko abstracts the complexity of the Internet Computer with a well known, higher-level abstraction: the *actor model*. Each canister smart contract is represented as a typed actor. The type of an actor lists the messages it can handle. Each message is abstracted as a typed, asynchronous function. A translation from actor types to Candid types imposes structure on the raw binary data of the underlying Internet Computer. An actor is similar to an object, but is different in that its state is completely isolated, its interactions with the world are entirely through asynchronous messaging, and its messages are processed one-at-a-time, even when issued in parallel by concurrent actors.

Motoko は Internet Computer の持つ複雑さを _アクタ・モデル_ という, よく知られた高度な抽象を用いて抽象化しています. 各キャニスタ・スマート・コントラクトは型付けされたアクタとして表現されます. アクタの型は, 自分が扱えるメッセージの一覧です. 各メッセージは, 型付けられた非同期関数として抽象化されています.  アクタ型から Condid の型への翻訳によって, 下位層の Internet Computer の生のバイナリ・データ上の構造が決められます. アクタはオブジェクトに似ていますが, 異なるのは, 状態が完全に隔離されていること, 世界との相互作用が完全に非同期メッセージ交換によること, たとえ並行に動作するアクタが並列にメッセージを送ってきたとしても, メッセージは一度に1つずつ処理されることです.

In Motoko, sending a message to an actor is a function call, but instead of blocking the caller until the call has returned, the message is enqueued on the callee, and a *future* representing that pending request immediately returned to the caller. The future is a placeholder for the eventual result of the request, that the caller can later query. Between issuing the request, and deciding to wait for the result, the caller is free to do other work, including issuing more requests to the same or other actors. Once the callee has processed the request, the future is completed and its result made available to the caller. If the caller is waiting on the future, its execution can resume with the result, otherwise the result is simply stored in the future for later use.

Motoko では, アクタへのメッセージ送信は関数呼び出しですが, 

1.   呼び出しが戻るまで呼び出し元がブロックされるのではなく, メッセージは呼び出し先のキューに入れられ, 保留中のリクエストを表す _フューチャ_ が呼び出し元に即時に返されます.フューチャは, 呼び出し元が後で (必要になったときに) 問い合わせできるような, リクエストの将来の結果の仮置場です.
2.   リクエストを送信してから, 結果が必要になるまでの間, 呼び出し元は自由に他の仕事 (別のリクエストを同じ, あるいは別のアクタに送るなど) ができます.
3.   いったん呼び出し先がリクエストを処理してしまえば, フューチャは完了し, その結果は呼び出し元が利用可能になります. 呼び出し元がフューチャを待っているとしたら, その実行はその結果と共に回復しますし, そうでなければ, その結果は後で使えるようにフューチャの中に格納されます.

In Motoko, actors have dedicated syntax and types; messaging is handled by so called *shared* functions returning futures (shared because they are available to remote actors); a future, `f`, is a value of the special type `async T` for some type `T`; waiting on `f` to be completed is expressed using `await f` to obtain a value of type `T`. To avoid introducing shared state through messaging, for example, by sending an object or mutable array, the data that can be transmitted through shared functions is restricted to immutable, *shared* types.

Motoko では, アクタには専用の構文と型があります. それは,

-   メッセージ交換はフューチャを返すいわゆる _共有_ 関数に依って取り扱われます (共有という理由は, そのフューチャが遠隔のアクタにもアクセスできるため)
-   フューチャ `f` は, ある型 `T` に対して `async T` という特殊な型の値になります
-   `f` が完了するのを待っていることは型 `T` の値を得るための `await f` を使って表現されます.

 (例えば) オブジェクトや可変配列を送るなどメッセージによって共有状態が紛れ込むのを避けるために, 共有関数を通じて送信されるデータは不変な _共有_ 型に制限されます.

To start, we consider the simplest stateful service: a `Counter` actor, the distributed version of our previous, local `counter` object.

まず始めに, いちばん簡単な状態を持つサービス - 以前にローカルで動かした `counter` オブジェクトの分散ヴァージョンである `Counter` アクタから始めましょう.

## Example: a Counter service

Consider the following actor declaration:

次のアクタ宣言をご覧下さい:

``` motoko
actor Counter {

  var count = 0;

  public shared func inc() : async () { count += 1 };

  public shared func read() : async Nat { count };

  public shared func bump() : async Nat {
    count += 1;
    count;
  };
};
```

The `Counter` actor declares one field and three public, *shared* functions:

`Counter` アクタはひとつのフィールドと3つのパブリックな _共有_ 関数を宣言しています:

-   the field `count` is mutable, initialized to zero and implicitly `private`.

-   function `inc()` asynchronously increments the counter and returns a future of type `async ()` for synchronization.

-   function `read()` asynchronously reads the counter value and returns a future of type `async Nat` containing its value.

-   function `bump()` asynchronously increments and reads the counter.



-   フィールド `count` は可変で, ゼロに初期化され, 暗黙に `private` です.
-   関数 `inc()` は非同期にカウンタを増やし, 同期のために  `async ()` 型のフューチャを返します.
-   関数 `read()` は非同期にカウンタの値を読んで, その値を持つ `async Nat` 型のフューチャを返します.
-   関数 `bump()` は非同期にカウンタを増やし, カウンタを読み出します.

Shared functions, unlike local functions, are accessible to remote callers and have additional restrictions: their arguments and return value must be *shared* types - a subset of types that includes immutable data, actor references, and shared function references, but excludes references to local functions and mutable data. Because all interaction with actors is asynchronous, an actor’s functions must return futures, that is, types of the form `async T`, for some type `T`.

共有関数はローカル関数とは異なり, 遠隔の呼び出し元にアクセスできますが, 引数と返値は _共有_ 型でなければならないという独自の制約もあります. 共有型には, 不変データ, アクタ参照, 共有関数参照などがありますが, ローカル関数への参照や可変データは含みません. アクタによる通信はすべて非同期なので, アクタの関数はフューチャ, つまりある型 `T` に対して `async T` 形式の型を返す必要があります.

The only way to read or modify the state (`count`) of the `Counter` actor is through its shared functions.

`Counter` アクタの状態 (`count`) を読んだり, 変更したりするには, その共有関数を通じて行うしかありません.

A value of type `async T` is a future. The producer of the future completes the future when it returns a result, either a value or error.

型 `async T` の値がフューチャです. フューチャの生成側は結果 (値かエラーか) を返すときにフューチャを完了します.

Unlike objects and modules, actors can only expose functions, and these functions must be `shared`. For this reason, Motoko allows you to omit the `shared` modifier on public actor functions, allowing the more concise, but equivalent, actor declaration:

オブジェクトやモジュールとは異なり, アクタが公開できるのは関数だけで, しかもこの関数は `共有` できるものでなければなりません. この理由から, Motoko はパブリックなアクタ関数に `shared` 修飾子を付けなくてもより簡潔に同等なアクタ宣言ができるようにしています:

``` motoko
actor Counter {

  var count = 0;

  public func inc() : async () { count += 1 };

  public func read() : async Nat { count };

  public func bump() : async Nat {
    count += 1;
    count;
  };
};
```

For now, the only place shared functions can be declared is in the body of an actor or actor class. Despite this restriction, shared functions are still first-class values in Motoko and can be passed as arguments or results, and stored in data structures.

今のところ, 共有関数を宣言できるのは, アクタかアクタ・クラスの本体の中だけです. この制約にも拘わらず, 共有関数は Motoko の中では第一級の値で, 引数や返値として渡したり, データ構造の中に格納したりすることができます.

The type of a shared function is specified using a shared function type. For example, the value `inc` has type `shared () → async Nat` and could be supplied as a standalone callback to some other service (see [publish-subscribe](sharing) for an example).

共有関数の型は, 共有関数型を用いて指定されます. 例えば, 値 `inc` の型は `shared () → async Nat` になり, これを他のサービスへの単独のコールバックとして用いることができます (例は[公開-購読](sharing) を見てね).

## Actor types

Just as objects have object types, actors have *actor types*. The `Counter` actor has the following type:

オブジェクトがオブジェクト型を持つのと同様に, アクタは _アクタ型_ を持ちます. `Counter` アクタは次のような型を持っています:

``` motoko
actor {
  inc  : shared () -> async ();
  read : shared () -> async Nat;
  bump : shared () -> async Nat;
}
```

Again, because the `shared` modifier is required on every member of an actor, Motoko both elides them on display, and allows you to omit them when authoring an actor type.

前にも言ったとおりアクタのメンバ毎に `shared` 修飾子が必要ですが, Motoko では表示時に省略したり, アクタ型の記述時に付けなくていいようにしています.

Thus the previous type can be expressed more succinctly as:

したがって, 上の型をもっと簡潔に書くとすると:

``` motoko
actor {
  inc  : () -> async ();
  read : () -> async Nat;
  bump : () -> async Nat;
}
```

Like object types, actor types support subtyping: an actor type is a subtype of a more general one that offers fewer functions with more general types.

オブジェクト型と同様, アクタ型は部分型付けができます. つまり, あるアクタ型はより汎用性の高い型を持ち, より少ない関数で汎用的なアクタの部分型になります.

## Using `await` to consume async futures

The caller of a shared function typically receives a future, a value of type `async T` for some T.

共有関数の呼び出し元は, 通常 ある T に対する 型 `async T` の値となるようなフューチャを受け取ります.

The only thing the caller, a consumer, can do with this future is wait for it to be completed by the producer, throw it away, or store it for later use.

呼び出し元 (利用側) がこのフューチャに関してできるのは, 作成側に依って完了とされるのを待つこと, それを放棄してしまうこと, あるいは後で使えるように格納することだけです.

To access the result of an `async` value, the receiver of the future use an `await` expression.

`async` な値の結果にアクセスするには, フューチャの受け手が `await` 式を使わなければなりません.

For example, to use the result of `Counter.read()` above, we can first bind the future to an identifier `a`, and then `await a` to retrieve the underlying `Nat`, `n`:

例えば, 上の `Counter.read()` の結果を利用するためには, まずフューチャを識別子 `a` に束縛し, 次に中にある `Nat n` を取り出すために `await a` します:

``` motoko
let a : async Nat = Counter.read();
let n : Nat = await a;
```

The first line immediately receives *a future of the counter value*, but does not wait for it, and thus cannot (yet) use it as a natural number.

最初の行では, _カウンタの値のフューチャ_ をすぐに受け取りますが, 待っていないので, それを自然数として使うのは (まだ) できません.

The second line `await`s this future and extracts the result, a natural number. This line may suspend execution until the future has been completed.

二行目では, フューチャを `await` し, 結果の自然数を取り出しています. この行は, フューチャが完了するまで実行が中断するかもしれません.

Typically, one rolls the two steps into one and one just awaits an asynchronous call directly:

通常はこの二行を一行にまとめて, 非同期呼び出しを直接待つことが多いでしょう:

``` motoko
let n : Nat = await Counter.read();
```

Unlike a local function call, which blocks the caller until the callee has returned a result, a shared function call immediately returns a future, `f`, without blocking. Instead of blocking, a later call to `await f` suspends the current computation until `f` is complete. Once the future is completed (by the producer), execution of `await p` resumes with its result. If the result is a value, `await f` returns that value. Otherwise the result is some error, and `await f` propagates the error to the consumer of `await f`.

呼び出し先が結果を返すまで呼び出し元がブロックされるローカル関数の呼び出しとは異なり, 共有関数はブロック無しで即座にフューチャ `f` を返します. ブロックされない分, 後での `await f` 呼び出しが `f` を完了するまでの間, 現在の計算を中断します.いったん (作成側によって) フューチャが完了すれば, `await f` の実行はその結果を持って回復します. その結果が値であれば `await f` はその値を返します. そうではなく結果が何らかのエラーであれば, `await f` はエラーを `await f` の利用側に広めます. __NOTE:__ `await p` とあるのは `await f` の間違い

Awaiting a future a second time will just produce the same result, including re-throwing any error stored in the future. Suspension occurs even if the future is already complete; this ensures state changes and message sends prior to *every* `await` are committed.

フューチャを二度待っても, (フューチャに格納されたエラーの再スローまで含めて) 同じ結果が戻ることになっています. フューチャが既に完了している場合でも中断は起こり得ます. それは, 毎回 `await` がコミットされるのに先立って状態変化とメッセージ送信が確実に行われるようにするためです.

:::danger

A function that does not `await` in its body is guaranteed to execute atomically - in particular, the environment cannot change the state of the actor while the function is executing. If a function performs an `await`, however, atomicity is no longer guaranteed. Between suspension and resumption around the `await`, the state of the enclosing actor may change due to concurrent processing of other incoming actor messages. It is the programmer’s responsibility to guard against non-synchronized state changes. A programmer may, however, rely on any state change prior to the await being committed.

本体内で `await` しない関数は, 原子的に実行すること (特に関数の実行中に, 環境がそのアクタの状態を変更しないこと) が保証されています. ただし, 関数が `await` を一度でも実行したら原子性はもう保証されません. `await` を囲む中断と回復の間, その外側のアクタの状態は他に入ってきたアクタのメッセージの並列処理のせいで変わってしまうかもしれません. 非同期の状態変更に対して保護をするのはプログラマの責務です. しかし, プログラマは待ちがコミットされる以前の状態変化に依存してしまっているかもしれません.

:::

For example, the implementation of `bump()` above is guaranteed to increment and read the value of `count`, in one atomic step. The alternative implementation:

例えば上の `bump()` の実装では, `count` の値の増加と読み出しが1つの原子的なステップで行われることは保証されています:

``` motoko
  public shared func bump() : async Nat {
    await inc();
    await read();
  };
```

does *not* have the same semantics and allows another client of the actor to interfere with its operation: each `await` suspends execution, allowing an interloper to change the state of the actor. By design, the explicit `await`s make the potential points of interference clear to the reader.

しかし, こちらは同じ意味ではなく, この演算にこのアクタの他のクライアントが干渉できてしまいます. つまり, 各 `await` が実行を中断している間に侵入者がアクタの状態を変えてしまうことがあります. 明示的な `await` が干渉の潜在的なポイントになり得ることを読者に明確にするのは設計上意図してのことです.

## Traps and Commit Points

A trap is a non-recoverable runtime failure caused by, for example, division-by-zero, out-of-bounds array indexing, numeric overflow, cycle exhaustion or assertion failure.

トラップとは, ゼロ除算, 配列インデクスの境界越え, 数値溢れ,  サイクル枯渇, 表明の失敗などが引き起こす, 回復不可能な実行時故障のことです.

A shared function call that executes without executing an `await` expression never suspends and executes atomically. A shared function that contains no `await` expression is syntactically atomic.

`await` 式を実行しないまま実行されている共有関数呼び出しは決して中断せずに, 原子的に実行されます.

An atomic shared function whose execution traps has no visible effect on the state of the enclosing actor or its environment - any state change is reverted, and any message that it has sent is revoked. In fact, all state changes and message sends are tentative during execution: they are committed only after a successful *commit point* is reached.

原子的な共有関数では, 実行時トラップが上位のアクタや実行環境の状態に目に見える影響を与えません. つまり, 状態の変化は戻され, 送信されたメッセージは取り消されます. 実際, 実行中のすべての状態変化やメッセージ送信は暫定的なものです. つまり _コミット点_ に到達するまでそれらはコミットされません.

The points at which tentative state changes and message sends are irrevocably committed are:

暫定的な状態変化やメッセージ送信が取り消しできない状態にコミットされる時点は, 以下のいずれかです:

-   implicit exit from a shared function by producing a result,

-   explict exit via `return` or `throw` expressions, and

-   explicit `await` expressions.



-   結果が出て, 共有関数から暗黙に終了するときか,

-   `return` 式や `throw` 式で明示的に終了するときか,

-   明示的に `await` するとき.

A trap will only revoke changes made since the last commit point. In particular, in a non-atomic function that does multiple awaits, a trap will only revoke changes attempted since the last await - all preceding effects will have been committed and cannot be undone.

トラップは, その前のコミット点以降に行われた変更のみを取り消します. 特に, 複数回 `await` した, 原子的ではない関数では, トラップが取り消す変更は最後の `await` 以降の変更だけです. それ以前の影響は既にコミットされており, 取り消すことはできません.

For example, consider the following (contrived) stateful `Atomicity` actor:

例として, 次の (不自然ですが) 状態を持つアクタ `Atomicity` を見てください:

``` motoko
actor Atomicity {

  var s = 0;
  var pinged = false;

  public func ping() : async () {
    pinged := true;
  };

  // an atomic method
  public func atomic() : async () {
    s := 1;
    ignore ping();
    ignore 0/0; // trap!
  };

  // a non-atomic method
  public func nonAtomic() : async () {
    s := 1;
    let f = ping(); // this will not be rolled back!
    s := 2;
    await f;
    s := 3; // this will not be rolled back!
    await f;
    ignore 0/0; // trap!
  };

};
```

Calling (shared) function `atomic()` will fail with an error, since the last statement causes a trap. However, the trap leaves the mutable variable `s` with value `0`, not `1`, and variable `pinged` with value `false`, not `true`. This is because the trap happens *before* method `atomic` has executed an `await`, or exited with a result. Even though `atomic` calls `ping()`, `ping()` is tentative (queued) until the next commit point, so never delivered.

共有関数 `atomic()` の呼び出しは, 最後の文がトラップするためエラーになって失敗します. しかし, そのトラップは, 可変変数 `s` の値を `0` のままにするので, `1` にはなりませんし, 変数 `pinged` の値は `true` ではなく, `false` のままです. なぜかと言えば, `atomic` メソッドが `await` を実行する, あるいは結果を伴って終了する _前に_ トラップしたからです. `atomic` が `ping()` を呼んでいたとしても, `ping()`  は次のコミット点までは暫定的 (キューに入っている) なので, 決して送信されないのです.

Calling (shared) function `nonAtomic()` will fail with an error, since the last statement causes a trap. However, the trap leaves the variable `s` with value `3`, not `0`, and variable `pinged` with value `true`, not `false`. This is because each `await` commits its preceding side-effects, including message sends. Even though `f` is complete by the second await on `f`, this await also forces a commit of the state, suspends execution and allows for interleaved processing of other messages to this actor.

一方, 共有関数 `nonAtomic()` の呼び出しも, 最後の文がトラップするためにエラーで失敗します. しかしトラップによって, 変数 `s` の値は `0` ではなく `3` になり, 変数 `pinged` の値は `false` ではなく `true` になります. その理由は, `await` するごとにそれに先立つメッセージ送信のような副作用をコミットするからです. たとえ, `f` の二番目の `await` が完了したとしても, この `await` が状態をコミットしてしまい, 実行は中断し, このアクタに対する他のメッセージの処理が割り込んでしまうでしょう.

## Query functions

In Internet Computer terminology, all three `Counter` functions are *update* messages that can alter the state of the canister smart contract when called. Effecting a state change requires agreement amongst the distributed replicas before the Internet Computer can commit the change and return a result. Reaching consensus is an expensive process with relatively high latency.

Internet Computer の用語では, `Counter` の3つの関数はどれも呼ばれるとこのキャニスタ・スマート・コントラクトの状態を変えるような _更新_ メッセージと呼ばれます. 状態変化を起こすには, Internet Computer がその変化をコミットして結果を返す前に, 分散したレプリカ間での合意が必要になります. 合意を成立させるのは大変高くつくプロセスであり, 相当高いレイテンシが発生します.

For the parts of applications that don’t require the guarantees of consensus, the Internet Computer supports more efficient *query* operations. These are able to read the state of a canister smart contract from a single replica, modify a snapshot during their execution and return a result, but cannot permanently alter the state or send further Internet Computer messages.

アプリケーションの中でも, 合意の保証を必要としない部分もあります. そのような部分に対して, Internet Computer ではもっと効率の高い _問い合わせ_ 演算というものを用意しています. それはキャニスタ・スマート・コントラクトの状態をレプリカのうちの1つだけから読み, 実行して結果を返すまでの間はスナップショットの方を更新して, 状態を永続的に変更したり, それ以上の Internet Computer メッセージを送信しないようにするものです.

Motoko supports the implementation of Internet Computer queries using `query` functions. The `query` keyword modifies the declaration of a (shared) actor function so that it executes with non-committing, and faster, Internet Computer query semantics.

Motoko では `query` 関数を用いて, Internet Computer の `問い合わせ` 実装をサポートしています. `query` キーワードでアクタの共有関数の宣言を修飾すると, コミットがなくて, 高速な Internet Computer の問い合わせ意味論を実行します.

For example, we can extend the `Counter` actor with a fast-and-loose variant of the trustworthy `read` function, called `peek`:

例題として `Counte` アクタを拡張し, 信頼性の高い `read` 関数の代わりに「速いけど緩い」変種である `peek` を作ってみましょう:

``` motoko
actor Counter {

  var count = 0;

  // ...

  public shared query func peek() : async Nat {
    count
  };

}
```

The `peek()` function might be used by a `Counter` frontend offering a quick, but less trustworthy, display of the current counter value.

`peek()` 関数は, 多少信頼性は低くても素早く反応し, カウンタの現在値を表示する `Counter` のフロントエンドで使えるでしょう.

It is a compile-time error for a query method to call an actor function since this would violate dynamic restrictions imposed by the Internet Computer. Calls to ordinary functions are permitted.

問い合わせメソッドの中からアクタの関数を呼ぶのとコンパイル時エラーになります. Internet Computer が課す動的な制限を超えることになってしまうからです. 通常の関数を呼ぶのは問題ありません.

Query functions can be called from non-query functions. Because those nested calls require consensus, the efficiency gains of nested query calls will be modest at best.

問い合わせ関数は, 問い合わせ関数以外の関数から呼ぶこともできます. ただし, 入れ子の呼び出しは合意の成立を必要とするため, 入れ子になった問い合わせ呼び出しが得られる効率は良くても控えめなものになるでしょう.

The `query` modifier is reflected in the type of a query function:

`query` 修飾子は, 問い合わせ関数の型にも反映されます.

``` motoko
  peek : shared query () -> async Nat
```

As before, in `query` declarations and actor types the `shared` keyword can be omitted.

以前と同様, `query` 宣言においても, アクタ型の `shared` キーワードは省略できます.

## Messaging Restrictions

The Internet Computer places restrictions on when and how canister smart contracts are allowed to communicate. These restrictions are enforced dynamically on the Internet Computer but prevented statically in Motoko, ruling out a class of dynamic execution errors. Two examples are:

Internet Computer では, キャニスタ・スマート・コントラクト間の通信のタイミングと方法について制約を設けています. これらの制約は Internet Computer 上では動的に課せられますが, Motoko 上では動的な実行エラーの種類を排除することによって静的に防止することができます. 2つの例を挙げます:

-   canister smart contract installation can execute code, but not send messages.

-   a canister smart contract query method cannot send messages.



-   キャニスタ・スマート・コントラクトのインストール時にはコードは実行できるが, メッセージ送信はできない.
-   キャニスタ・スマート・コントラクトの問い合わせメソッドからはメッセージ送信できない.

These restrictions are surfaced in Motoko as restrictions on the context in which certain expressions can be used.

それらの制約は Motoko においては, その文脈である式が使えるかどうかと言う制約に置き換えられます.

In Motoko, an expression occurs in an *asynchronous context* if it appears in the body of an `async` expression, which may be the body of a (shared or local) function or a stand-alone expression. The only exception are `query` functions, whose body is not considered to open an asynchronous context.

Motoko では, ある式が `async` 式の本体 (共有/ローカル関数あるいは独立式の本体) に現れるならば, その式は非同期の文脈にあることになります. 唯一の例外は `query` 関数で, その本体は非同期の文脈にあるとは考えません.

In Motoko calling a shared function is an error unless the function is called in an asynchronouus context. In addition, calling a shared function from an actor class constructor is also an error.

Motoko で非同期の文脈以外で, 共有関数を呼び出すとエラーになります.さらにアクタ・クラスの構成子からの共有関数の呼び出しもエラーです.

The `await` construct is only allowed in an asynchronous context.

`await` 構成子は非同期の文脈でのみ許されます.

The `async` construct is only allowed in an asynchronous context.

`async` 構成子は非同期の文脈でのみ許されます.

It is only possible to `throw` or `try/catch` errors in an asynchronous context. This is because structured error handling is supported for messaging errors only and, like messaging itself, confined to asynchronous contexts.

エラーを `throw` あるいは `try/catch` できるのは非同期の文脈内だけです. それは, 構造化されたエラーの取り扱いがサポートされているのは通信エラーだけであるから, またメッセージ自身と同様, 非同期の文脈内に置かれているからです.

These rules also mean that local functions cannot, in general, directly call shared functions or `await` futures. This limitation can sometimes be awkward: we hope to extend the type system to be more permissive in future.

この規則はまた, ローカル関数は, 一般的に共有関数の呼び出しやフューチャの `await` を直接にはできないことを意味しています. 、この制約はときに問題になることもあります. 将来はもっと寛大な型システムに拡張したいと思っています.

## Actor classes generalize actors

An actor *class* generalizes a single actor declaration to the declaration of family of actors satisfying the same interface. An actor class declares a type, naming the interface of its actors, and a function that constructs a fresh actor of that type each time it is supplied with an argument. An actor class thus serves as a factory for manufacturing actors. Because canister smart contract installation is asynchronous on the Internet Computer, the constructor function is asynchronous too, and returns its actor in a future.

アクタ・`クラス` は, ひとつだけのアクタ宣言を同じインタフェイスを充たすアクタのファミリの宣言に拡張するものです. アクタ・クラスは型を宣言し, そのアクタのインタフェイスに名前を付け, 引数を渡される毎に新しいアクタを作成する関数を定義します. したがって, アクタ・クラスは, アクタを製造する工場のようなものです. キャニスタ・スマート・コントラクトのインストールは, Internet Computer では非同期なので, 構成子関数も非同期であり, 作成したアクタをフューチャとして返します.

For example, we can generalize `Counter` given above to `Counter(init)` below, by introducing a constructor parameter, variable `init` of type `Nat`:

上で作った `Counter` を下では `Counter(init)` として一般化し, 構成子パラメタ (型 `Nat` の変数 `init`) を導入する例を見てみましょう.

<div class="formalpara-title">

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

If this class is stored in file `Counters.mo`, then we can import the file as a module and use it to create several actors with different initial values:

このクラスを `Counters.mo` というファイルに入れたら, そのファイルをモジュールとしてインポートし, 異なる初期値を使って複数のアクタを作るのに使うことができます:

``` motoko
import Counters "Counters";

let C1 = await Counters.Counter(1);
let C2 = await Counters.Counter(2);
(await C1.read(), await C2.read())
```

The last two lines above *instantiate* the actor class twice. The first invocation uses the initial value `1`, where the second uses initial value `2`. Because actor class instantiation is asynchronous, each call to `Counter(init)` returns a future that can be `await`ed for the resulting actor value. Both `C1` and `C2` have the same type, `Counters.Counter` and can be used interchangeably.

最後より前の二行では, アクタ・クラスを二度インスタンス化しています. 最初のインスタンス化では初期値として `1` を, 二度目では初期値として `2` を使っています. アクタ・クラスのインスタンス化は非同期なので, `Counter(init)` の呼び出しはそれぞれフューチャを返し, それを `await` することで欲しいアクタを値として得ることができます. `C1`, `C2` のいずれも同じ型 `Counters.Counter` を持つので, 入れ替えて使うこともできます.

:::note

For now, the Motoko compiler gives an error when compiling programs that do not consist of a single actor or actor class. Compiled programs may still, however, reference imported actor classes. For more information, see [Importing actor classes](modules-and-imports#importing_actor_classes) and [Actor classes](actor-classes#actor_classes).

現時点では, Motoko のコンパイラは, アクタあるいはアクタ・クラスを1つも含まないようなプログラムをコンパイルしようとするとエラーになります. しかし,  コンパイルされたプログラムはインポートしたアクタ・クラスへの参照を維持している場合があります. 詳細は [アクタ・クラスをインポートする](modules-and-imports#importing_actor_classes) および [アクタ・クラス](actor-classes#actor_classes) をご覧下さい.

:::

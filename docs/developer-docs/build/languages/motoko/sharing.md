# Sharing data and behavior

Recall that in Motoko, mutable state is always private to an actor.

Motoko では, 可変状態は常にアクタにプライベートであることを思い出して下さい.

However, two actors can share message data, and those messages can refer to actors, including themselves and one another. Additionally, messages can refer to individual functions, if those functions are `shared`.

しかし, 二つのアクタはメッセージ・データを共有することはできますし, メッセージがアクタ (自分自身と相手) を参照することもできます. さらにメッセージは個々の関数も参照できます (`shared` 関数ならば) .

Through these mechanisms, two actors can coordinate their behavior through asynchronous message passing.

このような機構を通じて, 二つのアクタの非同期メッセージ交換による振る舞いの協調が可能になります.

## Publisher-subscriber pattern with actors

The examples in this section illustrate how actors share their functions by focusing on variations of the [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish-subscribe_pattern). In the publish-subscribe pattern, a **publishing** actor records a list of **subscriber** actors to notify when something notable occurs in the publisher’s state. For example, if the publisher actor publishes a new article, the subscriber actors are notified that a new article is available.

この節の例題では, [公開-購読パタン](https://en.wikipedia.org/wiki/Publish-subscribe_pattern) のさまざまな変種に焦点を当てて, アクタがどうやって関数を共有するのかを説明します.

The example below uses two actors in Motoko to build variations of the publisher-subscriber relationship.

下の例題は Motoko の二つのアクタを使って,  公開者と購読者の関係の変種を作成します.

To see the complete code for a working project that uses this pattern, see the [pubsub](https://github.com/dfinity/examples/tree/master/motoko/pubsub) example in the [examples repository](https://github.com/dfinity/examples).

このパタンを使って動いているプロジェクトの完全なコードは, [examples repository](https://github.com/dfinity/examples) にある [pubsub](https://github.com/dfinity/examples/tree/master/motoko/pubsub の例を見てください.

### Subscriber actor

The following `Subscriber` actor type provides a possible interface for the subscriber actor and the publisher actor to expose and to call, respectively:

下の `Subscriber` アクタ型は, 購読側アクタと公開側アクタのそれぞれ公開と呼び出しのインタフェイスとして考えられるものです.

``` motoko
type Subscriber = actor {
  notify : () -> ()
};
```

-   The `Publisher` uses this type to define a data structure to store its subscribers as data.

-   Each `Subscriber` actor exposes a `notify` update function as described in the `Subscriber` actor type signature above.



-   `Publisher` はこの型を使って, 購読者をデータとして保存するためのデータ構造を定義する
-   各 `SUbscriber` アクタは `Subscriber` アクタ型のシグネチャに定義された `notify` 更新関数を公開する

Note that sub-typing enables the `Subscriber` actor to include additional methods that are not listed in this type definition.

副型付けをすれば, `Subscriber` アクタはこの型定義には含まれていないようなメソッドでも追加して入れられることを付記しておきます.

For simplicity, assume that the `notify` function accepts relevant notification data and returns some new status message about the subscriber to the publisher. For example, the subscriber might return a change to its subscription settings based on the notification data.

簡単にするために, `notify` 関数は関連する通知データを受け付け, 購読者側の何らかの新しい状態メッセージを公開者側に返すものとします. 例えば, 購読者側は通知データに基づいて, 購読の設定の変更を返したりするかもしれません.

### Publisher actor

The publisher side of the code stores an array of subscribers. For simplicity, assume that each subscriber only subscribes itself once using a `subscribe` function.

コードの公開者側では, 購読者の配列を保存します. 簡単にするために, 各購読者は一度 `subscribe` 関数を使ったら購読に登録されると言うことにします.

``` motoko
import Array "mo:base/Array";

actor Publisher {
    var subs: [Subscriber] = [];

    public func subscribe(sub: Subscriber) {
        subs := Array.append<Subscriber>(subs, [sub]);
    };

    public func publish() {
        for (sub in subs.vals()) {
          sub.notify();
        };
    };
};
```

Later, when some unspecified external agent invokes the `publish` function, all of the subscribers receive the `notify` message, as defined in the `Subscriber` type given above.

この後, 何らかの不特定の外部エージェントが `publish` 関数を起動すると, 購読者全員に上の `Subscriber` 型で定義したとおり `notify` メッセージが届くことになります.

### Subscriber methods

In the simplest case, the subscriber actor has the following methods:

もっとも単純な場合には, 購読者アクタは以下のメソッドを持ちます:

-   Subscribe to notifications from the publisher using the `init` method.

-   Receive notification as one of the subscribed actors as specified by the `notify` function in the `Subscriber` type given above).

-   Permit queries to the accumulated state, which in this sample code is simply a `get` method for the number of notifications received and stored in the `count` variable.




-   `init` メソッドで, 公開者からの通知を購読する
-   購読登録しているアクタの一つとして, 上の `Subscriber` 型の `notiry` 関数で定義したとおり, 通知を受け取る
-   蓄積された状態への問い合わせ (下のサンプル・コードでは, 受け取って `count` 変数に格納した通知の数を返す `get` メソッド) を許す

The following code illustrates implementing these methods:

次のコードはこれらのメソッドを実装したものです:

``` motoko
actor Subscriber {
  var count: Nat = 0;
  public func init() {
    Publisher.subscribe(Subscriber);
  };
  public func notify() {
    count += 1;
  };
  public func get() : async Nat {
    count
  };
}
```

The actor assumes, but does not enforce, that its `init` function is only ever called once. In the `init` function, the `Subscriber` actor passes a reference to itself, of type `actor { notify : () → () };` (locally called `Subscriber` above).

このアクタでは, 自分の `init` 関数が一度だけ呼ばれることを前提としています (強制はしていないけど). `init` 関数では, `Subscriber` アクタが自分自身への参照 (型は `actor { notify : () → ()};`, 上でローカルには `Subscriber` と呼ばれている) を渡しています.

If called more than once, the actor will subscribe itself multiple times, and will receive multiple (duplicate) notifications from the publisher. This fragility is the consequence of the basic publisher-subscriber design we show above. With more care, a more advanced publisher actor could check for duplicate subscriber actors and ignore them, for instance.

もし二度以上呼ばれることがあると, このアクタは自分を複数回購読 (登録) することになり, 公開者からは複数の, 重複した通知を受け取ることになります. この脆弱性は, 上で見てもらった公開者-行動者の基本的な設計の結末です. もっと手を入れれば, 公開者アクタが購読者アクタの重複をチェックして, もしあれば無視するなどのようにできるでしょう.

## Sharing functions among actors

In Motoko, a `shared` actor function can be sent in a message to another actor, and then later called by that actor, or by another actor.

Motoko では アクタの `shared` 関数をメッセージに入れて別のアクタに送ることができ, そのアクタや別のアクタから呼ばれることもあります.

The code shown above has been simplified for illustrative purposes. The full version offers additional features to the publisher-subscriber relationship, and uses shared functions to make this relationship more flexible.

上で示したコードは説明を目的として簡略化しています. 完全版では, 公開者-購読者関係についてフィーチャを追加し, 共有関数を利用して, この関係をもっと柔軟にしています.

For instance, the notification function is *always* designated as `notify`. A more flexible design would only fix the type of `notify`, and permit the subscriber to choose any of its `shared` functions, specified in a `subscribe` message in place of (just) the actor that is subscribing.

例えば, 通知関数は _常に_  `notify` が指定されています. もっと柔軟な設計にすると, `notify` の型だけを固定すれば良く, 購読者が任意の `shared` 関数を選択して, 購読しているアクタに替わって `subscribe` メッセージで指定できるようになります.

See the [the full example](https://github.com/dfinity/examples/tree/master/motoko/pub-sub) for details.

[完全版](https://github.com/dfinity/examples/tree/master/motoko/pub-sub)の例題の詳細をご覧下さい.

In particular, suppose that the subscriber wants to avoid being locked into a certain naming scheme for its interface. What really matters is that the publisher can call *some* function that the subscriber chooses.

特に購読者がインタフェイスの特定の名前付け図式にはめ込まれてしまうのを避けたい場合を考えてみてください. 真に重要なのは, 公開者が購読者が選択した _いずれかの_ 関数を呼べる, と言うことなのです.

### The `shared` keyword

To permit this flexibility, an actor needs to share a single *function* that permits remote invocation from another actor, not merely a reference to itself.

この柔軟性を許すようにするためには, あるアクタが一つの _関数_ を共有して, 別のアクタから (単に参照するだけではなく) 遠隔起動できるようにする必要があります.

The ability to share a function requires that it be pre-designated as `shared`, and the type system enforces that these functions follow certain rules around the types of data that these functions accept, return, and over which their closures close.

ある関数を共有できるようにするには, 前もってその関数を `shared` に指定し, そのような関数が受け付ける/返すデータの型が一定の規則に従い, クロージャがそれに関して閉じていることを型システムが遵守させるようにしなければなりません.

Motoko lets you omit this keyword for *public* actor methods since, implicitly, *any public function of an actor must be \`shared\`*, whether marked explicitly or not.

Motoko ではアクタのパブリックなメソッドについては _shared_ と書かなくても良いようにしていますが, それはアクタのすべての関数は明示的にそう書かれているかどうかに拘わらず, 暗黙の裡に`shared` でなければならないからなのです.

Using the `shared` function type, we can extend the example above to be more flexible. For example:

`shared` 関数型を使って, 上の例題を拡張し, もっと柔軟なものにしてみましょう. 例えば:

``` motoko
type SubscribeMessage = { callback: shared () -> (); };
```

This type differs from the original, in that it describes *a message* record type with a single field called `callback`, and the original type first shown above describes *an actor* type with a single method called `notify`:

この型が元の型と違うのは, _message_ のレコード型が `callback` と呼ばれる一つのフィールドを記述するものであるのに対して, 最初に見てもらった最初の型は `notify` と呼ばれる一つのメソッドを持つ _アクタ_ 型を記述していると言うことです.

``` motoko
type Subscriber = actor { notify : () -> () };
```

Notably, the `actor` keyword means that this latter type is not an ordinary record with fields, but rather, an actor with at least one method, which *must* be called `notify`.

お分かりのように, `actor` キーワードはこの後者の型がフィールドを持つ普通のレコード型ではなく, 少なくとも一つのメソッド (その名前は `nofity` でなければならない) を持つアクタであることを意味しています.

By using the `SubscribeMessage` type instead, the `Subscriber` actor can choose another name for their `notify` method:

代わりに `SubscribeMessage` 型を使うことで, `Subscriber` アクタは `notify` 以外の別の名前を選べるようになります:

``` motoko
actor Subscriber {
  var count: Nat = 0;
  public func init() {
    Publisher.subscribe({callback = incr;});
  };
  public func incr() {
    count += 1;
  };
  public query func get(): async Nat {
    count
  };
};
```

Compared to the original version, the only lines that change are those that rename `notify` to `incr`, and form the new `subscribe` message payload, using the expression `{callback = incr}`.

オリジナル版に較べて 変更したのは `notify` を `incr` に換えたことと,  `subscribe` メッセージに式 `{callback = incr}` を載せたことです.

Likewise, we can update the publisher to have a matching interface:

このインタフェイスに合わせて, 公開側も更新しておきましょう:

``` motoko
import Array "mo:base/Array";
actor Publisher {
  var subs: [SubscribeMessage] = [];
  public func subscribe(sub: SubscribeMessage) {
    subs := Array.append<SubscribeMessage>(subs, [sub]);
  };
  public func publish() {
    for (sub in subs.vals()) {
      sub.callback();
    };
  };
};
```
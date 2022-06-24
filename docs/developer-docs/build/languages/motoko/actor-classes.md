# Actor classes

Actor classes enable you to create networks of actors (canister smart contracts) *programmatically*. Currently, actor classes have to be defined in a separate source file. To illustrate how to define and import actor classes, the following example implements a distributed map of keys of type `Nat` to values of type `Text`. It provides simple insert and lookup functions, `put(k, v)` and `get(k)`, for working with these keys and values.

アクタ・クラスを使うとアクタ (キャニスタ・スマート・コントラクト) のネットワークをプログラム的に作ることができます. 今のところ, アクタ・クラスは別のソース・ファイルで定義しなければなりません. アクタ・クラスをどうやって定義し, インポートするかを説明するために、次の例では, キーの型が  `Nat` で, 値の型が `Text` であるような分散マップを実装してみます. これは `put(k, v)` と `get(k)` で簡単な挿入と探索をするものです.

To distribute the data for this example, the set of keys is partitioned into `n` buckets. For now, we just fix `n = 8`. The bucket, `i`, of a key, `k`, is determined by the remainder of `k` divided by `n`, that is, `i = k % n`. The `i`th bucket (`i` in `[0..n)`) receives a dedicated actor to store text values assigned to keys in that bucket.

この例ではデータを分散させるために, キーの集合を `n` 個のバケットに分割します. 今は `n = 8` に固定しておきます. キーが `k` のバケット `i` は, `k` を `n` で割った余り, つまり `i = k % n` で決まります. ここで `i` 番目のバケット (`i`) in `[0..n)]`) はそれぞれ決まったアクタから, 保存するテキスト値を受け取り, そのバケットでキーを割り当てます.

The actor responsible for bucket `i` is obtained as an instance of the actor class `Bucket(i)`, defined in the sample `Buckets.mo` file, as follows:

バケット `i` を担当するアクタはアクタ・クラス `Bucket(i)` のインスタンスとして得ることができます. これを例の `Buckets.mo` ファイル (以下) で定義しています.

<div class="formalpara-title">

**Buckets.mo**

</div>

``` motoko
import Nat "mo:base/Nat";
import Map "mo:base/RBTree";

actor class Bucket(n : Nat, i : Nat) {

  type Key = Nat;
  type Value = Text;

  let map = Map.RBTree<Key, Value>(Nat.compare);

  public func get(k : Key) : async ?Value {
    assert((k % n) == i);
    map.get(k);
  };

  public func put(k : Key, v : Value) : async () {
    assert((k % n) == i);
    map.put(k,v);
  };

};
```

A bucket stores the current mapping of keys to values in a mutable `map` variable containing an imperative RedBlack tree, `map`, that is initially empty.

バケットでは, 現在のキーから値への対応付けを可変の `map` 変数 (中身は手続き的な RedBlack ツリー. 最初は空) に保存します.

On `get(k)`, the bucket actor simply returns any value stored at `k`, returning `map.get(k)`.

`get(k)` で, バケット・アクタは単に `k` に保存されている値を `map.get(k)` で返します.

On `put(k, v)`, the bucket actor updates the current `map` to map `k` to `?v` by calling `map.put(k, v)`.

`put(k, v)` では, バケット・アクタは現在の `map` を `map.put(k, v)` で `k` を `?v` に対応付けるように更新します.

Both functions use the class parameters `n` and `i` to verify that the key is appropriate for the bucket by asserting `((k % n) == i)`.

どちらの関数もクラスのパラメタ `n` と `i` を用いて, キーがそのバケットに適切かどうかを表明 `((k % n) == i)` で検証しています.

Clients of the map can then communicate with a coordinating `Map` actor, implemented as follows:

このマップのクライアントは, 調整役の `Map` アクタ (以下の実装) と通信します:

``` motoko
import Array "mo:base/Array";
import Buckets "Buckets";

actor Map {

  let n = 8; // number of buckets

  type Key = Nat;
  type Value = Text;

  type Bucket = Buckets.Bucket;

  let buckets : [var ?Bucket] = Array.init(n, null);

  public func get(k : Key) : async ?Value {
    switch (buckets[k % n]) {
      case null null;
      case (?bucket) await bucket.get(k);
    };
  };

  public func put(k : Key, v : Value) : async () {
    let i = k % n;
    let bucket = switch (buckets[i]) {
      case null {
        let b = await Buckets.Bucket(n, i); // dynamically install a new Bucket
        buckets[i] := ?b;
        b;
      };
      case (?bucket) bucket;
    };
    await bucket.put(k, v);
  };

};
```

As this example illustrates, the `Map` code imports the `Bucket` actor class as module `Buckets`.

この例で書かれているように, `Map` のコードで `Bucket` アクタ・クラスをモジュール `Buckets` としてインポートしています.

The actor maintains an array of `n` allocated buckets, with all entries initially `null`. Entries are populated with `Bucket` actors on demand.

アクタは割り当てられた `n` 個のバケットの配列 (初期値は `null`) の面倒を見ます. エントリは必要なときに `Bucket` アクタが設定します.

On `get(k, v)`, the `Map` actor:

`Map` アクタの `get(k)` では: __NOTE:__ 引数は (k, v) ではなく (k) だね

-   uses the remainder of key `k` divided by `n` to determine the index `i` of the bucket responsible for that key

-   returns `null` if the `i`th bucket does not exist, or

-   delegates to that bucket by calling `bucket.get(k, v)` if it does.




-   キー `k` を `n` で割った余りを使ってそのキーの責務を担うバケットのインデクス `i` を決める
-   もし バケット `i` が存在しなければ `null` を返す
-   もしあれば `bucket.get(k, v)` を呼んでバケットに委譲する

On `put(k, v)`, the `Map` actor:

`Map` アクタの `put(k, v)` では:

-   uses the remainder of key `k` divided by `n` to determine the index `i` of the bucket responsible for that key

-   installs bucket `i` if the bucket does not exist by using an asynchronous call to the constructor, `Buckets.Bucket(i)`, and, after awaiting the result, records it in the array `buckets`

-   delegates the insertion to that bucket by calling `bucket.put(k, v)`.




-   キー `k` を `n` で割った余りを使ってそのキーの責務を担うバケットのインデクス `i` を決める
-   もしそのバケットが存在していなければ, 構成子 `Buckets.Bucket(i)` への非同期呼び出しを使ってバケット `i` をインストールし, その結果を待って, それを配列 `buckets` に記録する
-   `bucket.put(k, v)` を呼んで, そのバケットへの挿入を委譲する

While this example sets the number of buckets to `8`, you can easily generalize the example by making the `Map` actor an actor *class*, adding a parameter `(n : Nat)` and omitting the declaration `let n = 8;`. For example:

この例題ではバケットの数を `8` に設定していますが, `Map` アクタをアクタ・クラスにしてパラメタ `(n : Nat)` を付け加え, 宣言 `len n = 8` を削除すれば, 簡単に一般化できます. 例えば:

``` motoko
actor class Map(n : Nat) {

  type Key = Nat
  ...
}
```

Clients of actor *class* `Map` are now free to determine the (maximum) number of buckets in the network by passing an argument on construction.

こうすればアクタ・クラス `Map` のクライアントは, 構築時に引数を渡して, ネットワーク中のバケットの (最大) 数を自由に決めることができます.

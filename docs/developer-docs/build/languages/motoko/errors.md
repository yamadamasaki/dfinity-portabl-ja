# Errors and Options

There are three primary ways to represent and handle errors values in Motoko:

Motoko では, エラーの値を表現し, 取り扱う方法がおもに三つあります:

-   Option values (with a non-informative `null` indicated *some* error);

-   `Result` variants (with a descriptive `#err value` providing more information about the error); and

-   `Error` values (that, in an asynchronous context, can be thrown and caught - similar to exceptions - and contain a numeric code and message).



-   Option の値 (_何らかの_ エラーを示すが, 情報はない `null`);
-   `Result` ヴァリアント (それよりはエラーに関する情報が多い `#err value`)
-   `Error` の値 (非同期の文脈で投げたり, 捕まえたりできる (例外に似ている), 数値コードとメッセージ) 

## Our Example API

Let’s assume we’re building an API for a Todo application and want to expose a function that lets a user mark one of their Todo’s as **Done**. To keep it simple we’ll accept a `TodoId` and return an `Int` that represents how many seconds the Todo has been open. We’re also assuming we’re running in our own actor so we return an async value. If nothing would ever go wrong that would leave us with the following API:

ここでわたしたちは, Todo アプリケーションのための API を構築しており, ユーザが自分の Todo の中の一つに **Done** の印を付けられるような関数を公開しようとしているとしましょう. 簡単のため, `TodoId` を受け取り, その Todo が何秒間オープンになっていたかを示す `Int` を返すことにします. また, この API はアクタの中で動いているものとするので, 非同期値を返します. 特に悪いことが起きないとすれば, 次のような API になります:

``` motoko
func markDone(id : TodoId) : async Int
```

The full definition of all types and helpers we’ll use in this document is included for reference:

このドキュメントで用いるすべての型とヘルパを参考までに載せておきます:

<div class="informalexample">

``` motoko
import Int "mo:base/Int";
import Hash "mo:base/Hash";
import Map "mo:base/HashMap";
import Time "mo:base/Time";
import Result "mo:base/Result";
import Error "mo:base/Error";
```

``` motoko
type Time = Int;
type Seconds = Int;

func secondsBetween(start : Time, end : Time) : Seconds =
  (end - start) / 1_000_000_000;

public type TodoId = Nat;

type Todo = { #todo : { text : Text; opened : Time }; #done : Time };
type TodoMap = Map.HashMap<TodoId, Todo>;

var idGen : TodoId = 0;
let todos : TodoMap = Map.HashMap(32, Int.equal, Hash.hash);

private func nextId() : TodoId {
  let id = idGen;
  idGen += 1;
  id
};

/// Creates a new todo and returns its id
public shared func newTodo(txt : Text) : async TodoId {
  let id = nextId();
  let now = Time.now();
  todos.put(id, #todo({ text = txt; opened = now }));
  id
};
```

</div>

## When things go wrong

We now realize that there are conditions under which marking a Todo as done fails.

Todo に done の印を付けるのが失敗するような条件にはどんなものがあるでしょうか.

-   The `id` could reference a non-existing Todo

-   The Todo might already be marked as done




-   `id` が存在しない Todo を参照している
-   その Todo には既に done の印が付いていた

We’ll now talk about the different ways to communicate these errors in Motoko and slowly improve our solution.

Motoko でこれらのエラーをやり取りするいろいろなやり方について述べるので, 我々のソリューションを少しずつ良くしていきましょう.

## What error type to prefer

### How *not* to do things

One particularly easy and *bad* way of reporting errors is through the use of a *sentinel* value. For example, for our `markDone` function we might decide to use the value `-1` to signal that something failed. The callsite then has to check the return value against this special value and report the error. But it’s way too easy to not check for that error condition and continue to work with that value in our code. This can lead to delayed or even missing error detection and is strongly discouraged.

エラーを参照報告する, とりわけ簡単ではあるけれど _良くない_ やり方として, _番兵_ と呼ばれるような特別な値を使うやり方があります. 例えば `markDone` 関数では, 値 `-1` を使って何か間違いが起きたことを知らせることにしましょう. でもこの場合に起こりがちなのは, そのエラー条件をチェックせず, コード中でその値のまま仕事を続けることです. これにより, エラー検出の遅延や欠落につながる可能性があり, まったくお奨めできません.

Definition:

``` motoko
public shared func markDoneBad(id : TodoId) : async Seconds {
  switch (todos.get(id)) {
    case (?(#todo(todo))) {
      let now = Time.now();
      todos.put(id, #done(now));
      secondsBetween(todo.opened, now)
    };
    case _ { -1 };
  }
};
```

Callsite:

``` motoko
public shared func doneTodo1(id : Todo.TodoId) : async Text {
  let seconds = await Todo.markDoneBad(id);
  if (seconds != -1) {
    "Congrats! That took " # Int.toText(seconds) # " seconds.";
  } else {
    "Something went wrong.";
  };
};
```

### Prefer Option/Result over Exceptions where possible

Using `Option` or `Result` is the preferred way of signaling errors in Motoko. They work in both synchronous and asynchronous contexts and make your APIs safer to use (by encouraging clients to consider the error cases as well as the success cases. Exceptions should only be used to signal unexpected error states.

`Option` と `Result` を使うのは Motoko でエラーを伝えるのによりお奨めのやり方です. これは同期/非同期どちらの文脈でも動き, クライアントに成功した場合と同様にエラーの場合も考慮に入れることを推奨するので, あなたの API をより安全にします. 例外は, 予期せぬエラー状態を伝えるときにのみ使うようにします.

### Error reporting with Option

A function that wants to return a value of type `A` or signal an error can return a value of *option* type `?A` and use the `null` value to designate the error. In our example this means having our `markDone` function return an `async ?Seconds`.

型 `A` の値を返すか, エラーを伝えるかしたい関数ならばオプション型 `?A`の値を返し, エラーを指定したい場合には `null` 値を使うということもできます. この例では, `markDone` 関数が `async ?Secods` を返すことに相当します.

Here’s what that looks like for our `markDone` function:

`markDone` 関数は次のようになります:

Definition:

``` motoko
public shared func markDoneOption(id : TodoId) : async ?Seconds {
  switch (todos.get(id)) {
    case (?(#todo(todo))) {
      let now = Time.now();
      todos.put(id, #done(now));
      ?(secondsBetween(todo.opened, now))
    };
    case _ { null };
  }
};
```

Callsite:

``` motoko
public shared func doneTodo2(id : Todo.TodoId) : async Text {
  switch (await Todo.markDoneOption(id)) {
    case null {
      "Something went wrong."
    };
    case (?seconds) {
      "Congrats! That took " # Int.toText(seconds) # " seconds."
    };
  };
};
```

The main drawback of this approach is that it conflates all possible errors with a single, non-informative `null` value. Our callsite might be interested in why marking a `Todo` as done has failed, but that information is lost by then, which means we can only tell the user that `"Something went wrong."`. Returning option values to signal errors should only be used if there just one possible reason for the failure, and that reason can be easily determined at the callsite. One example of a good usecase for this is a HashMap lookup failing.

このやり方のおもな欠点は, いろいろあるエラーの可能性を一つの, 意味のない `null` 値で知らせていることです. 呼び出し側では「なぜ `Todo` を done できなかったのか」が知りたいのに, その情報はその時点では失われてしまっているので, ユーザには `「何かまずいことが起こりました」` しか伝えられないのです. エラーを伝えるためにオプション値を返すのは, 失敗の原因の可能性がひとつしかない場合, その理由が呼び出し側から簡単に決められる場合のみにすべきです. このやり方の良いユースケースの一例は, HashMap 探索の失敗です.

### Error reporting with `Result` types

To address the shortcomings of using option types to signal errors we’ll now look at the richer `Result` type. While options are a built-in type, the `Result` is defined as a variant type like so:

エラーを伝えるのにオプション型を使う欠点を解決するのに, より高機能な `Result` 型を見てみましょう. オプションは組込みの型でしたが, `Result` はヴァリアント型として定義されています:

``` motoko
type Result<Ok, Err> = { #ok : Ok; #err : Err }
```

Because of the second type parameter, `Err`, the `Result` type lets us select the type we use to describe errors. So we’ll define a `TodoError` type our `markDone` function will use to signal errors.

二つ目の型パラメタ `Err` があるので, `Result` 型ではエラーを記述するのに使う型を我々が選択できます. では, `markDone` 関数がエラーを伝えるのに使う `TodoError` 型を定義しましょう:

``` motoko
public type TodoError = { #notFound; #alreadyDone : Time };
```

This lets us now write the third version of `markDone`:

これを使って, `markDone` の三番目のヴァージョンを書いてみます:

Definition:

``` motoko
public shared func markDoneResult(id : TodoId) : async Result.Result<Seconds, TodoError> {
  switch (todos.get(id)) {
    case (?(#todo(todo))) {
      let now = Time.now();
      todos.put(id, #done(now));
      #ok(secondsBetween(todo.opened, now))
    };
    case (?(#done(time))) {
      #err(#alreadyDone(time))
    };
    case null {
      #err(#notFound)
    };
  }
};
```

Callsite:

``` motoko
public shared func doneTodo3(id : Todo.TodoId) : async Text {
  switch (await Todo.markDoneResult(id)) {
    case (#err(#notFound)) {
      "There is no Todo with that ID."
    };
    case (#err(#alreadyDone(at))) {
      let doneAgo = secondsBetween(at, Time.now());
      "You've already completed this todo " # Int.toText(doneAgo) # " seconds ago."
    };
    case (#ok(seconds)) {
      "Congrats! That took " # Int.toText(seconds) # " seconds."
    };
  };
};
```

And as we can see we can now give the user a useful error message.

さて, ようやくユーザに役に立つエラー・メッセージを渡すことができるようになりました.

## Working with Option/Result

`Option`s and `Results`s are a different way of thinking about errors, especially if you come from a language with pervasive exceptions. In this chapter we’ll look at the different ways to create, destructure, convert, and combine `Option`s and `Results` in different ways.

どこにでも例外を使うような言語から来た人にとっては, この `Options` と `Results` を使うやり方は, エラーに関する考え方に大きな違いを感じるでしょう. この章では, `Options` と`Results` を今までとは違うやり方で作成し, 再構築し, 変換し, 組み合わせるやり方を見てみます.

### Pattern matching

The first and most common way of working with `Option` and `Result` is to use 'pattern matching'. If we have a value of type `?Text` we can use the `switch` keyword to access the potential `Text` contents:

 `Options` と `Results` を使う第一の, そしてもっともよくあるやり方は, 'パタン・マッチング' を使うことです. 型 `?Text` の値があったとしたら, `Text` に入っている内容に `switch` キーワードを使ってアクセスすることができます:

``` motoko
func greetOptional(optionalName : ?Text) : Text {
  switch (optionalName) {
    case (null) { "No name to be found." };
    case (?name) { "Hello, " # name # "!" };
  }
};
assert(greetOptional(?"Dominic") == "Hello, Dominic!");
assert(greetOptional(null) ==  "No name to be found");
```

The important thing to understand here is that Motoko does not let you access the optional value without also considering the case that it is missing.

ここで理解して欲しい, 重要なことは, Motoko では, オプショナルな値にアクセスする場合には開発者にすべての場合分けを考慮させるようにしている, ということです.

In the case of a `Result` we can also use pattern matching, with the difference that we also get an informative value (not just `null`) in the `#err` case.

`Restul` の場合にもパタン・マッチングを使うことができますが, `#err` からは (単なる `null` ではない) 意味のある値を得ることができる点が異なっています.

``` motoko
func greetResult(resultName : Result<Text, Text>) : Text {
  switch (resultName) {
    case (#err(error)) { "No name: " # error };
    case (#ok(name)) { "Hello, " # name };
  }
};
assert(greetResult(#ok("Dominic")) == "Hello, Dominic!");
assert(greetResult(#err("404 Not Found")) == "No name: 404 Not Found");
```

### Higher-Order functions

Pattern matching can become tedious and verbose, especially when dealing with multiple optional values. The [base](https://github.com/dfinity/motoko-base) library exposes a collection of higher-order functions from the `Optional` and `Result` modules to improve the ergonomics of error handling.

パタン・マッチングは, 特に複数のオプション値を扱う場合には, 退屈で, 面倒なものです. [base](https://github.com/dfinity/motoko-base) ライブラリでは エラー処理の人間工学を改善するような高階関数の集まりを `Optional` と `Result` から公開しています.

### Converting back and forth between Option/Result

Sometimes you’ll want to move between Options and Results. A Hashmap lookup returns `null` on failure and that’s fine, but maybe the caller has more context and can turn that lookup failure into a meaningful `Result`. At other times you don’t need the additional information a `Result` provides and just want to convert all `#err` cases into `null`. For these situations [base](https://github.com/dfinity/motoko-base) provides the `fromOption` and `toOption` functions in the `Result` module.

開発中には `Options` と `Results` の間を行ったり来たりすることがあるかもしれません. HashMap の探索で, 失敗したときに `null` を返すのは大丈夫, 問題ありません. しかし, 呼び出し元にはもっと別の文脈があり, 探索の失敗をもっと意味のある `Result` に変えたいかもしれない. また別のときには, `Result` が用意するような付加的な情報は不要で, `#err` をすべて `null` に変換したくなるかもしれない. このような状況に対して, [base](https://github.com/dfinity/motoko-base) では `Result` モジュールに `fromOption`, `toOption` 関数を用意してあります.

## Asynchronous Errors

The last way of dealing with errors in Motoko is to use asynchronous `Error` handling, a restricted form of the exception handling familiar from other languages. Unlike the exceptions of other languages, Motoko *errors* values, can only be thrown and caught in asynchronous contexts, typically the body of a `shared` function or `async` expression. Non-`shared` functions cannot employ structured error handling. This means you can exit a shared function by `throw`ing an `Error` value and `try` some code calling a shared function on another actor, `catch`ing its failure as a result of type `Error`, but you can’t use these error handling constructs in regular code, outside of an asynchronous context.

Motoko でエラーを扱う最後の方法は, 非同期な `Error` 処理 (他の言語ではなじみ深い例外処理の制限版) を使うことです. 他の言語の例外とは異なり, Motoko の _errors_ 値は非同期の文脈 (通常 `shared` 関数か, `async` 式の本体) でだけ投げたり, 捕まえたりできます. 非 `shared` 関数は構造的なエラー処理を利用できません. これは `Error` 値を投げて, `try` のコードで他のアクタの共有関数を呼び, そこでこの失敗を `Error` 型の結果として `catch` すれば, 共有関数を抜けられる, ということを意味しています. しかし, このようなエラー処理の構成子は, 非同期の文脈外である通常のコードで使うことはできません.

Asynchronous `Error`s should generally only be used to signal unexpected failures that you cannot recover from, and that you don’t expect many consumers of your API to handle. If a failure should be handled by your caller you should make it explicit in your signature by returning a `Result` instead. For completeness here is the `markDone` example with exceptions:

非同期の `Error` は, 一般的に自分では回復できない予期せぬ失敗を伝えたり, 自分の API の多くのクライアントがこれを取り扱えないという場合にのみ利用します. ある失敗を自分の呼び出し元が扱うべきときには, そのことを `Result` を返すようなシグネチャとして明示するべきです. 完全性を期すために, 例外を用いた `markDone` の例題を示します.

Definition:

``` motoko
public shared func markDoneException(id : TodoId) : async Seconds {
  switch (todos.get(id)) {
    case (?(#todo(todo))) {
      let now = Time.now();
      todos.put(id, #done(now));
      secondsBetween(todo.opened, now)
    };
    case (?(#done(time))) {
      throw Error.reject("Already done")
    };
    case null {
      throw Error.reject("Not Found")
    };
  }
};
```

Callsite:

``` motoko
public shared func doneTodo4(id : Todo.TodoId) : async Text {
  try {
    let seconds = await Todo.markDoneException(id);
    "Congrats! That took " # Int.toText(seconds) # " seconds.";
  } catch (e) {
    "Something went wrong.";
  }
};
```

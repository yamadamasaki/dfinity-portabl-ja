# Introduction

Motoko is a modern, general-purpose programming language you can use specifically to author [Internet Computer](../../../../concepts/what-is-ic) canister smart contracts. Although aimed squarely at the Internet Computer, its design is general enough to support future compilation to other targets.

Motoko はモダンな汎用プログラミング言語で, 特に [インタネット・コンピュータ](../../../../concepts/what-is-ic) のキャニスタ・スマート・コントラクトを記述するのに使えます. Internet Computer での利用を直接の目標としていますが, 汎用性のあるデザインになっているので, 将来的には他のターゲットへのコンパイルも十分可能です.

## Approachability

Motoko is a modern language designed to be approachable for programmers who have some basic familiarity with modern object-oriented and/or functional programming idioms in either JavaScript, or another modern programming language, such as Rust, Swift, TypeScript, C#, or Java.

Motokoは, JavaScript のようなモダンなプログラミング言語 (Rust, Swift, TypeScript, C#, Java など) でのオブジェクト指向, あるいは関数型のプログラミング・イディオムにある程度馴染んでいるプログラマが取り組みやすいように考えて設計した, モダンな言語です.

## Asynchronous messaging and type sound execution

Motoko permits modern programming idioms, including special programming abstractions for distributed applications (dapps). Each dapp consists of one or more *actors* that communicate solely by *asynchronous message passing*. The state of an actor is isolated from all other actors, supporting distribution. There is no way to share state between several actors. The actor-based programming abstractions of Motoko permit human-readable message-passing patterns, and they enforce that each network interaction obeys certain rules and avoids certain common mistakes.

Motoko はモダンなプログラミング・イディオムを用意していますが, 中でも分散型アプリケーション (dapp) 向けの特別なプログラミング抽象化を用意しています. Dapp のひとつひとつは, もっぱら _非同期なメッセージ・パッシング_ によって通信するひとつ, あるいは複数の _アクタ_ から成ります. あるアクタの状態は他のアクタからは完全に切り離されているので, 分散性が成立します. 複数のアクタ間で状態を共有することはできません. Motoko のアクタ・ベースのプログラミング抽象化によって, ひとにも読みやすいメッセージ・パッシングのパタンを記述できるので, それぞれのネットワーク上の相互作用が一定のルールに則り, ありがちな誤りを回避することが可能になります.

Specifically, Motoko programs are *type sound* since Motoko includes a practical, practical type system that checks each one before it executes. The Motoko type system statically checks that each Motoko program will execute safely, without dynamic type errors, on all possible inputs. Consequently, entire classes of common programming pitfalls that are common in other languages, and web programming languages in particular, are ruled out. This includes null reference errors, mis-matched argument or result types, missing field errors and many others.

特に, Motoko には実用的かつ実用的な型システムがあり, 実行前に型を検査できるので, Motoko で書かれたプログラムは  _型健全_ であると言えます. Motoko の型システムは, Motoko で書かれたプログラムが, あり得るどんな入力に対しても動的な型エラーなしに安全に実行できるかどうかを静的に検査します. これによって, 他の言語 (特に Web 向けプログラミング言語ね) でよくあるプログラミング上の落し穴 (null 参照, 引数や返値の型の不適合, 存在しないフィールドへのアクセス, などなど) の類をすべて排除できます.

To execute, Motoko statically compiles to [WebAssembly](about-this-guide#wasm), a portable binary format that abstracts cleanly over modern computer hardware, and thus permits its execution broadly on the Internet, and the [Internet Computer](../../../../concepts/what-is-Internet Computer).

Motoko は実行のために [WebAssembly](about-this-guide#wasm) (モダンなコンピュータ・ハードウェアとして明確に抽象化された, 可搬性のあるバイナリ形式) にコンパイルされるので, インタネット上の広い範囲で, もちろん [Internet Computer](../../../../concepts/what-is-Internet Computer) 上でも実行できます.

## Each canister smart contract as an *actor*

Motoko provides an **actor-based** programming model to developers to express *services*, including those of canister smart contracts on the [Internet Computer](../../../../concepts/what-is-Internet Computer).

Motoko は開発者が [Internet Computer](../../../../concepts/what-is-Internet Computer) 上のキャニスタ・スマート・コントラクトを含む _サービス_ を記述するために **アクタ・ベース** のプログラミング・モデルを提供します.

An actor is similar to an object, but is special in that its state is completely isolated, and all its interactions with the world are by *asynchronous* messaging.

アクタはオブジェクトに似ていますが, その状態が完全に分離されている点と, その世界の中では _非同期_ なメッセージングによってすべての相互作用を行う点が特徴です.

All communication with and between actors involves passing messages asynchronously over the network using the Internet Computer’s messaging protocol. An actor’s messages are processed in sequence, so state modifications never admit race conditions (unless explicitly allowed by punctuating `await` expressions).

アクタ間のすべての通信には, インタネット・コンピュータのメッセージング・プロトコルを用いたネットワーク上の非同期のメッセージ・パッシングが伴います. ひとつのアクタ内ではメッセージは順次処理され, 状態の変更では (`await` 式によって明示的に中断されない限りは) 競合は起こりません .

The Internet Computer ensures that each message that is sent receives a response. The response is either success with some value, or an error. An error can be the explicit rejection of the message by the receiving canister smart contract, a trap due to an illegal instruction such as division by zero, or a system error due to distribution or resource constraints. For example, a system error might be the transient or permanent unavailability of the receiver (either because the receiving actor is oversubscribed or has been deleted).

インタネット・コンピュータでは, メッセージを送るごとにリスポンスを受け取ることが保証されます. リスポンスは成功 (何らかの値がある) か, エラーか,です. エラーは, 受け取ったキャニスタ・スマート・コントラクトがそのメッセージを明示的に拒絶したこと, あるいは, ゼロ除算のような不正命令でトラップしたこと, メッセージの配送や資源の制約に起因するシステム・エラーが生じたことを示しています.

### Asynchronous actors

Like other *modern* programming languages, Motoko permits an ergonomic syntax for *asynchronous* communication among components.

他のモダンなプログラミング言語と同じように, Motoko でもコンポネント間の _非同期_ な通信用のひとにやさしい構文を使えます.

In the case of Motoko, each communicating component is an actor.

Motoko の場合は, コミュニケートするコンポネントとはアクタのことです.

As an example of *using* actors, perhaps as an actor ourselves, consider this three-line program:

アクタを _利用した_ 例題として, 次の三行のプログラム (多分これ自身がアクタ) を見てみましょう:

``` motoko
let result1 = service1.computeAnswer(params);
let result2 = service2.computeAnswer(params);
finalStep(await result1, await result2)
```

We can summarize the program’s behavior with three steps:

このプログラムの振る舞いをまとめると, 次の 3 ステップになります:

1.  The program makes two requests (lines 1 and 2) to two distinct services, each implemented as a Motoko actor or canister smart contract implemented in some other language.

2.  The program waits for each result to be ready (line 3) using the keyword `await` on each result value.

3.  The program uses both results in the final step (line 3) by calling the `finalStep` function.



1.   このプログラムでは, 二つの異なるサービス (それぞれが Motoko のアクタ, あるいは別の言語で実装されたキャニスタ・スマート・コントラクトとして) に二つのリクエストを送る (行 1, 2).
2.   それぞれの返値ごとに `await` キーワードを用いて, 結果が戻って来るのを待つ (行 3).
3.   最後のステップ (行 3) での二つの結果を用いて `finalStep` 関数を呼び出す.

Generally-speaking, the services *interleave* their executions rather than wait for one another, since doing so reduces overall latency. However, if we try to reduce latency this way *without* special language support, such interleaving will quickly sacrifice clarity and simplicity.

一般的には, 複数のサービスの実行はひとつずつ順に行われるのではなく, _インタリーヴして_ 行われることになります. その方が全体的な待ち時間を減らせるからです. ただし, このような待ち時間の短縮を言語の特別なサポート無しで実現しようとすると, 明解さと単純さを大きく犠牲にせざるを得ないでしょう.

Even in cases where there are *no* interleaving executions (for example, if there were only one call above, not two), the programming abstractions still permit clarity and simplicity, for the same reason. Namely, they signal to the compiler where to transform the program, freeing the programmer from contorting the program’s logic in order to interleave its execution with the underlying system’s message-passing loop.

このようなプロミング抽象化があれば,  _インタリーヴした_ 実行がない場合 (例えば, 上記のような二つの呼び出しではなく, ひとつだけの場合) でさえ, 明解さと単純さがもたらされます. と言うのも, このようなプロミング抽象化によって, プログラムをどのように変換するかの指示をコンパイラに与え, 下層のシステムのメッセージ・パッシングのループを使って実行をインタリーヴするためにプログラムのロジックをいじくる手間からプログラマを解放できるからです.

Here, the program uses `await` in line 3 to express that interleaving behavior in a simple fashion, with human-readable syntax that is provided by Motoko.

このプログラムでは, `await` を使って (行 3), Motoko が提供する, ひとが読みやすい構文でインタリーヴする振る舞いを単純な見た目に表現できています.

In language settings that lack these abstractions, developers would not merely call these two functions directly, but would instead employ very advanced programming patterns, possibly registering developer-provided “callback functions” within system-provided “event handlers”. Each callback would handle an asynchronous event that arises when an answer is ready. This kind of systems-level programming is powerful, but very error-prone, since it decomposes a high-level data flow into low-level system events that communicate through shared state. Sometimes this style is necessary, but here it is not.

このような抽象化を持たない言語では, 開発者はこの二つの関数を単純に直接呼び出すのでは済まず, とても高度なプログラミング・パタン (例えば開発者が書いた「コールバック関数」をシステムが提供する「イベント・ハンドラ」に登録する, というような) を駆使しなければなりません. このように高レベルのデータ・フローを低レベルの共有状態に用いて通信するシステム・イベントに分解しているので, この種のシステム・レベルのプログラミングは強力ではありますが, 間違いを呼び込みやすいものになりがちです. このようなやり方が必要な場合もありますが, 今はそうではありません.

Our program instead eschews that more cumbersome programming style for this more natural, *direct* style, where each request resembles an ordinary function call. This simpler, stylized programming form has become increasingly popular for expressing practical systems that interact with an *external environment*, as most modern software does today. However, it requires special compiler and type-system support, as we discuss in more detail below.

我々のプログラムでは込み入ったプログラミング・スタイルを避けて, より自然で _直接的_ なやり方, つまりそれぞれのリクエストが通常の関数呼び出しのように見えるやり方を採用します. 今日のほとんどのモダンなソフトウェアのように _外部環境_ と相互作用する実用的なシステムを表現するための, このような単純で様式化されたプログラミング形式は急速に広まっています. しかしそれには, 特別なコンパイラと型システムの支援が必要となります. その詳細についてはこれから見ていきましょう.

### Support for *asynchronous* behavior

In an *asynchronous* computing setting, a program and its running environment are permitted to perform *internal computations* that occur *concurrently* with one another.

_非同期_ な計算環境では, プログラムとその実行環境は _並列_ に起きる _内部計算_ を実行できるようになっています.

Specifically, asynchronous programs are ones where the program’s requests of its environment do not (necessarily) require the program to wait for the environment. In the meantime, the program is permitted to make internal progress within this environment while the environment proceeds to complete the request. In the example, above, the program issues the second request before waiting for the first request to complete.

特に, 非同期なプログラムはプログラムから環境へのリクエストで, プログラムを待たせなくてもいいようになっています. 一方, 環境がリクエストの処理を行っている間にも, プログラムが内部計算を続行できるようになっています. 上の例では, プログラムは最初のリクエストが完了するのを待たずに二番目のリクエストを送っています.

Symmetrically, the environment’s requests of the program do not (necessarily) require the environment to wait for the program’s answer: the environment can make external progress while the answer is produced.

対称的に, 環境からプログラムへのリクエストもプログラムの答えを環境が待たなくてもいいはずです. つまり答えを用意している間も環境は外部とのやり取りを続けられるのです.

We do not show an example of this “notify” pattern above, since it uses callbacks (and *higher-order* functions and control flow) and is thus more complex.

ここではこの "通知" パタンの例は取り上げません (コールバック (と高階関数/制御フロー) を使うために複雑になるので) .

### Syntactic forms `async` and `await`

To address the need for clarity and simplicity, Motoko adopts the increasingly-common program constructs `async` and `await`, which afford the programmer a *structured* language for describing potentially-complex asynchronous dependency graphs.

明解さと単純さの必要性を解決するために, Motoko は複雑になりがちな非同期性の依存グラフをプログラマが _構造的_ に書くためによく使われるようになってきた二つのプログラム構成子 `async` と `await` を提供しています.

The [async](language-manual#exp-async) syntax introduces futures. A future value represents a *promise* of a result *that will be delivered, asynchronously, sometime in the future* (not shown in the first example above). You’ll learn more about futures when we introduce actors in [Actors and async data](actors-async).

[async](language-manual#exp-async) 構文はフューチャを導入します. フューチャ値とは _未来のいつか非同期的に届くはず_ の結果を _約束_ することを表します (これは最初の例では使っていません). フューチャについては, この先 [アクタと非同期データ](actors-async) でアクタを紹介するときに学ぶことにしましょう.

Here, we merely use the ones that arise from calling `service1.computeAnswer(params)` and `service2.computeAnswer(params)`.

ここでは単に `service1.computeAnswer(params)` と `service2.computeAnswer(params)` を使うことします.

The syntax `await` synchronizes on a future, and suspends computation until the future is completed by its producer. We see two uses of `await` in the example above, to obtain the results from two calls to services.

`await` 構文はフューチャに同期して, そのフューチャがその生産者側で完了するまで計算を待つというものです. 上の例題では `await` を二つ使った二つのサービス呼び出しから結果を得ましたね.

When the developer uses these keywords, the compiler transforms the program as necessary, often doing complex transformations to the program’s control- and data-flow that would be tedious to perform by hand in a purely synchronous language. Meanwhile, the type system of Motoko enforces certain correct usage patterns for these constructs, including that types flowing between consumers and producers always agree, and that the types of data sent among services are permitted to flow there, and do not (for example) contain [private mutable state](mutable-state).

開発者がこれらのキーワードを使うとコンパイラはプログラムを必要に応じて変換しますが, たいていの場合, プログラムの制御フローとデータフローを複雑に変換することになり, 純粋な同期型言語ではとてもプログラマの手に負えるものではありません. Motoko の型システムはこれらの構成子に対して正しい利用パタンを当てはめて, 消費者と生産者の間で型について常に一致すること, サービス間で受け渡される値の型が合っていること, その中に (例えば) [そのアクタ内部の変化する状態](mutable-state) が含まれていないことなどを保証します.

### Types are static

Like other modern programming languages, Motoko permits each variable to carry the value of a function, object, or a primitive datum (for example, a string, word, or integer). Other [types of values](basic-concepts#intro-values) exist too, including records, tuples, and “tagged data” called *variants*.

他のモダンなプログラミング言語と同じく, Mokto は変数には値として, 関数, オブジェクト, 原始値 (文字列, ワード, 整数など) を入れることができます. 他の [値の型](basic-concepts#intro-values) (レコード, タプル, タグ値 (ヴァリアントと呼ぶ)) もあります.

Motoko enjoys the formal property of type safety, also known as *type soundness*. We often summarize this idea with the phrase: [Well-typed Motoko programs don’t go wrong](basic-concepts#intro-type-soundness), meaning that the only operations that will be performed on data are those permitted by its static type.

Motoko には (_型の健全性_ としても知られる) 型の安全性という悦ばしい形式的性質もあります. 我々はこの概念を一言で次のように表すこともあります: [正しく型付けられた Motoko のプログラムには誤りはない](basic-concepts#intro-type-soundness) . つまり, あるデータに対して実行できる演算は, その静的に付けられた型で許されたものに限られると言うことです.

For example, each variable in a Motoko program carries an associated *type*, and this type is known *statically*, before the program executes. Each use of each variable is checked by the compiler to prevent runtime type errors, including null reference errors, invalid field access and the like.

例えば, Motoko のプログラムでは各変数には _型_ が指定され, その型は _静的に_ (プログラムを実行する前に) 決まります. 各値は使用するごとにコンパイラが検査して, 実行時の型エラーがないこと (null 参照エラーや存在しないフィールドへのアクセスなどを含む) を確認します.

In this sense, Motoko types provide a form of *trustworthy, **compiler-verified** documentation* in the program source code.

その意味では, Motoko の型はプログラムのソースコードの中の _信頼に足る, コンパイラに依って検証済のドキュメンテーション_ なのです.

As usual, dynamic testing can check properties that are beyond the reach of the Motoko type system. While modern, the Motoko type system is intentionally *not* “advanced” or particularly exotic. Rather, the type system of Motoko integrates standard concepts from modern, but well-understood, [実用的な型システム](about-this-guide#modern-types) to provide an approachable, expressive yet safe language for programming general-purpose, distributed applications.

もちろん, Motoko の型システムの値からが及ばない性質についての動的なテストはあり得ます. Motoko の型システムはモダンなものではありますが, あえて「先端的」だったり, 特別に変わったものにはして _いません_. むしろ Motoko の型システムは, 汎用の分散アプリケーションをプログラムする, 近付きやすく, 表現力も高く, かつ安全な言語を提供するためのモダンではあっても, 広く理解されている [実用的な型システム](about-this-guide#modern-types) 由来の標準的な考え方を統合したものになっています.

# Managing cycles

Usage of the Internet Computer is measured, and paid for, in *cycles*. The Internet Computer maintains a balance of cycles per canister smart contract. In addition, cycles can be transferred between canister smart contracts.

Internet Computer の使用量は測定されており, _サイクル_ で支払われます. Internet Computer はキャニスタ・スマート・コントラクト毎にサイクルの収支を管理します. さらにサイクルはキャニスタ・スマート・コントラクト間で振り替えることができます.

In Motoko programs targeting the Internet Computer, each actor represents an Internet Computer canister smart contract, and has an associated balance of cycles. The ownership of cycles can be transferred between actors. Cycles are selectively sent and received through messages, that is, shared function calls. A caller can choose to transfer cycles with a call, and a callee can choose to accept cycles that are made available by the caller. Unless explicitly instructed, no cycles are transferred by callers or accepted by callees.

Motoko プログラムは Internet Computer をターゲットとしているので, 各アクタは Internet Computer のキャニスタ・スマート・コントラクトを現しており, サイクル収支に紐付いています. サイクルの所有権はアクタ間で移転できます. サイクルはメッセージを経由して, すなわち共有関数呼び出しで, 選択的に送受信されます. 呼び出し元は呼び出し時にサイクルの転送を選択でき, 呼び出し先は呼び出し元が用意したサイクルを受け取るかどうかを選択できます. 明示的に指定しない限り, 呼び出し元からサイクルを転送し, 呼び出し先がそれを受け取ることはありません.

Callees can accept all, some or none of the available cycles up to limit determined by their actor’s current balance. Any remaining cycles are refunded to the caller. If a call traps, all its accompanying cycles are automatically refunded to the caller, without loss.

呼び出し先は用意されたサイクルのうち, アクタの現在の収支によって決められる限度内で, すべてを受け取る, 一部を受け取る, あるいはまったく受け取らないということができます. もし呼び出しがトラップしたら, それに付随するすべてのサイクルは自動的に呼び出し元に払い戻され, 損失はありません.

In future, we may see Motoko adopt dedicated syntax and types to support safer programming with cycles. For now, we provide a temporary way to manage cycles through a low-level imperative API provided by the [ExperimentalCycles](../../../../references/motoko-ref/experimentalcycles) library in package `base`.

将来的には, サイクルをより安全にプログラミングできるように Motoko 専用の構文と型を採用するかもしれません. 今のところは, 一時的ではありますが, サイクルを `base` パッケージの [ExperimentalCycles](../../../../references/motoko-ref/experimentalcycles) ライブラリが提供する低レベルで手続き的な API を通じて管理します.

:::note

This library is subject to change and likely to be replaced by more high-level support for cycles in later versions of Motoko.

このライブラリは変わることがあり, Motoko の将来のヴァージョンではより高いレベルのサイクルのサポートで置き換えられる可能性があります.

:::

## The `ExperimentalCycles` Library

The `ExperimentalCycles` library provides imperative operations for observing an actor’s current balance of cycles, transferring cycles and observing refunds.

`ExperimentalCycles` ライブラリは, あるアクタのサイクルの現在の収支を監視したり, サイクルを振り替えたり, 払い戻しを監視したりする手続き的な操作を提供しています.

The library provides the following operations:

このライブラリが提供するのは, 以下の操作です:

``` motoko
func balance() : (amount : Nat)

func available() : (amount : Nat)

func accept(amount : Nat) : (accepted : Nat)

func add(amount : Nat) : ()

func refunded() : (amount : Nat)
```

Function `balance()` returns the actor’s current balance of cycles as `amount`. Function `balance()` is stateful and may return different values after calls to `accept(n)`, calling a function after `add`ing cycles, or resuming from await (reflecting a refund).

`banace()` 関数はそのアクタの現在の収支を `amount` として返します. `balance()` 関数は状態を持ち, `accept(n)` を呼び出した後, サイクルを `追加` して何かの関数を呼び出したとき, await から戻ったとき (払い戻しが反映される) には異なる値を返すでしょう.

:::danger

Since cycles measure computational resources spent, the value of `balance()` generally decreases from one shared function call to the next.

サイクルは消費された計算資源を測るものなので, 共有関数のある呼び出しから次の呼び出しの間に`balance()` の値は減少するのが一般的です.

:::

Function `available()`, returns the currently available `amount` of cycles. This is the amount received from the current caller, minus the cumulative amount `accept`ed so far by this call. On exit from the current shared function or `async` expression via `return` or `throw` any remaining available amount is automatically refunded to the caller.

`available()` 関数はサイクルの現在残っている `amount` (量) を返します. これは現在の呼び出し元から受け取った量からこの呼び出しによってこれまでに `accept` された (受け取られた) 量の積算量を引いたものです. この共有関数や `非同期` 式から `return` か `throw` によって抜けるときに残っている量は自動的に呼び出し元に払い戻されます.

Function `accept` transfers `amount` from `available()` to `balance()`. It returns the amount actually transferred, which may be less than requested, for example, if less is available, or if canister smart contract balance limits are reached.

`accept` 関数は, `amount` を `available()` から `balance()` に移します. 実際に移された量を返しますが, これは例えば用意できた量が足りなかったとか, キャニスタ・スマート・コントラクトの収支上限に達したとかの場合には要求した量より少なくなります.

Function `add(amount)` indicates the additional amount of cycles to be transferred in the next remote call, i.e. evaluation of a shared function call or `async` expression. Upon the call, but not before, the total amount of units `add`ed since the last call is deducted from `balance()`. If this total exceeds `balance()`, the caller traps, aborting the call.

`add(amont)` 関数は, 次の遠隔呼び出し, つまり共有関す呼び出しや`async` 式の評価で転送されるサイクルの追加量を示します.  (事前にではなく) 呼び出し時に, 最後の呼び出し以降に `add` された総量が, `balance()` から差し引かれます.

:::note

the implicit register of added amounts, incremented on each `add`, is reset to zero on entry to a shared function, and after each shared function call or on resume from an await.

追加された量 (`add` 毎に増えていく) の暗黙の登録が, 共有関数に入るとき, 各共有関数の呼び出しや await からの回復毎にゼロにリセットされます.

:::

Function `refunded()` reports the `amount` of cycles refunded in the last `await` of the current context, or zero if no await has occurred yet. Calling `refunded()` is solely informational and does not affect `balance()`. Instead, refunds are automatically added to the current balance, whether or not `refunded` is used to observe them.

`refunded()` 関数は, 現在の文脈での最後の `await` で払い戻されたサイクルの量 (まだ await されていない場合はゼロ) を報告します. `refunded()` の呼び出しは完全に情報を得るためのもので, `balance()` には影響しません. ただし, 払い戻しは自動的に現在の収支に追加され, 払い戻しを監視するために `refunded` を呼んだかどうかは関係ありません.

### Example

To illustrate, we will now use the `ExperimentalCycles` library to implement a toy *piggy bank* for saving cycles.

説明のために `ExperimentalCycles` ライブラリを使って, サイクルをやり取りする _子豚ブーブー銀行_ の実装をしてみましょう.

Our piggy bank has an implicit owner, a `benefit` callback and a fixed `capacity`, all supplied at time of construction. The callback is used to transfer *withdrawn* amounts.

子豚ブーブー銀行には, 暗黙的な `owner`, `benefit` コールバック, 固定値の `capacity` があり, これらは作成時に指定されます. このコールバックは _引き落とし_ 額を送金するのに使われます.

``` motoko
import Cycles "mo:base/ExperimentalCycles";

shared(msg) actor class PiggyBank(
  benefit : shared () -> async (),
  capacity: Nat
  ) {

  let owner = msg.caller;

  var savings = 0;

  public shared(msg) func getSavings() : async Nat {
    assert (msg.caller == owner);
    return savings;
  };

  public func deposit() : async () {
    let amount = Cycles.available();
    let limit : Nat = capacity - savings;
    let acceptable =
      if (amount <= limit) amount
      else limit;
    let accepted = Cycles.accept(acceptable);
    assert (accepted == acceptable);
    savings += acceptable;
  };

  public shared(msg) func withdraw(amount : Nat)
    : async () {
    assert (msg.caller == owner);
    assert (amount <= savings);
    Cycles.add(amount);
    await benefit();
    let refund = Cycles.refunded();
    savings -= amount - refund;
  };

}
```

The owner of the bank is identified with the (implicit) caller of constructor `PiggyBank()`, using the shared pattern, `shared(msg)`. Field `msg.caller` is a `Principal` and is stored in private variable `owner` (for future reference). See [Principals and caller identification](caller-id) for more explanation of this syntax.

この銀行の所有者は, `PiggyBank()` の構成子を呼んだひとと (暗黙裡に) 識別されます (共有パタン `shared(msg)` を使っています). `msg.caller` フィールドは `Principal` でプライベート変数 `owner` に (後での参照のために) 保存します. この構文についての詳細は [プリンシパルと呼び出し者の識別](caller-id) をご覧下さい.

The piggy bank is initially empty, with zero current `savings`.

この銀行は最初は空っぽで, `savings` はこの時点でゼロです.

Only calls from `owner` may:

`owner` が呼ぶのは:

-   query the current `savings` of the piggy bank (function `getSavings()`), or

-   withdraw amounts from the savings (function `withdraw(amount)`).



-   現在の `savings` (残高) を問い合わせる (関数 `getSavings()`)
-   残高からある量を引き下ろす (関数 withdraw(amount))

The restriction on the caller is enforced by the statements `assert (msg.caller ==
owner)`, whose failure causes the enclosing function to trap, without revealing the balance or moving any cycles.

呼び出し元の制約は `assert (msg.caller == owner)` 文に依って課せられ, 制約に失敗するとそれを囲んでいる関数がトラップして, 残高の参照やサイクルの移動のないまま失敗します.

Any caller may `deposit` an amount of cycles, provided the savings will not exceed `capacity`, breaking the piggy bank. Because the deposit function only accepts a portion of the available amount, a caller whose deposit exceeds the limit will receive an implicit refund of any unaccepted cycles. Refunds are automatic and ensured by the Internet Computer infrastructure.

呼び出し元はある量 (savings が  `capacity` を越えない範囲) のサイクルを `deposit` (預け入れ) できます (`capacity` を越えてしまったら銀行は破綻です). deposit 関数は手元にある量の一部だけしかを受け入れられないので, この限界を超えた預け入れをしたひとは, 受け入れられなかったサイクルを暗黙に払い戻されます. 払い戻しは自動的で, Internet Computer のインフラストラクチャによって保証されます.

Since transfer of cycles is one-directional (from caller to callee), retrieving cycles requires the use of an explicit callback (the `benefit` function, taken by the constructor as an argument). Here, `benefit` is called by the `withdraw` function, but only after authenticating the caller as `owner`. Invoking `benefit` in `withdraw` inverts the caller/caller relationship, allowing cycles to flow "upstream".

サイクルの移動は一方通行 (呼び出し元から呼び出し先へ) なので, サイクルを取り出すときには明示的なコールバック (構成子が引数として取る `benefit` 関数) の使用が必要になります. ここでは `benefit` は `withdraw` 関数で, 呼び出し元が  `owner` であることを確認した後に呼ばれています.

Note that the owner of the `PiggyBank` could, in fact, supply a callback that rewards a beneficiary distinct from `owner`.

この銀行の所有者が実際には, 所有者とは異なる受益者に報酬を与えるコールバックを提供できることに注意してください.

Here’s how an owner, `Alice`, might use an instance of `PiggyBank`:

ここで所有者 `Alice` が `ブーブー子豚銀行` のインスタンスをどう使っているでしょうか:

``` motoko
import Cycles = "mo:base/ExperimentalCycles";
import Lib = "PiggyBank";

actor Alice {

  public func test() : async () {

    Cycles.add(10_000_000_000_000);
    let porky = await Lib.PiggyBank(Alice.credit, 1_000_000_000);

    assert (0 == (await porky.getSavings()));

    Cycles.add(1_000_000);
    await porky.deposit();
    assert (1_000_000 == (await porky.getSavings()));

    await porky.withdraw(500_000);
    assert (500_000 == (await porky.getSavings()));

    await porky.withdraw(500_000);
    assert (0 == (await porky.getSavings()));

    Cycles.add(2_000_000_000);
    await porky.deposit();
    let refund = Cycles.refunded();
    assert (1_000_000_000 == refund);
    assert (1_000_000_000 == (await porky.getSavings()));

  };

  // Callback for accepting cycles from PiggyBank
  public func credit() : async () {
    let available = Cycles.available();
    let accepted = Cycles.accept(available);
    assert (accepted == available);
  }

}
```

Let’s dissect `Alice`'s code.

`Alice` のコードを細かく見てみましょう.

`Alice` imports the `PiggyBank` actor class as a library, so she can create a new `PiggyBank` actor on demand.

`Alice` は`PiggyBank` アクタ・クラスをライブラリとしてインポートしているので, 必要に応じて `PiggyBank` アクタを新たに作ることができます

Most of the action occurs in `Alice`'s `test()` function:

`Alice` の `test()` 関数で何が起きているかというと:

Alice dedicates `10_000_000_000_000` of her own cycles for running the piggy bank, by calling `Cycles.add(10_000_000_000_000)` just before creating a new instance, `porky`, of the `PiggyBank`, passing callback `Alice.credit` and capacity (`1_000_000_000`). Passing `Alice.credit` nominates `Alice` as the beneficiary of withdrawals. The `10_000_000_000_000` cycles, minus a small installation fee, are credited to `porky`'s balance without any further action by `porky` initialization code. You can think of this as an electric piggy bank, that consumes its own resources as its used. Since constructing a `PiggyBank` is asynchronous, `Alice` needs to `await` the result.

Alice は `Cycles.add(10_000_000_000_000)` を呼んで, 銀行を運営するのに彼女自身のサイクル `10_000_000_000_000` を納めた後に, コールバック `Alice.credit` と capacity (`1_000_000_000`) を渡して `PiggyBank` の新しいインスタンス `porky` を作ります. `Alice.credit` を渡したことで, `Alice` は報酬の受取手の候補になりました. `porky` の初期化コード以外は特に何もしなくても `10_000_000_000_000` サイクルから少額のインストール手数料を引いたものが `porky` の残高に付いています. 自分で使った分だけ中身が減るので, これは電気仕掛けの豚の貯金箱だと思うかもしれません.`PiggyBank` の作成は非同期なので, `Alice` はその結果を `await` する必要があります.

After creating `porky`, she first verifies that the `porky.getSavings()` is zero using an `assert`.

`porky` を作った後, 彼女は最初に `assert` を使って `porky.getSavings()` がゼロであることを確認しています.

`Alice` dedicates `1_000_000` of her cycles (`Cycles.add(1_000_000)`) to transfer to `porky` with the next call to `porky.deposit()`. The cycles are only consumed from Alice’s balance if the call to `porky.deposit()` succeeds (which it should).

`Alice` は彼女のサイクルのうち `1_000_000` を使って (`Cycles.add(1_000_000)`), 次の `porky.deposit()` 呼び出しで, `porky` に送金します.

`Alice` now withdraws half the amount, `500_000`, and verifies that `porky`'s savings have halved. `Alice` eventually receives the cycles via a callback to `Alice.credit()`, initiated in `porky.withdraw()`. Note the received cycles are precisely the cycles `add`ed in `porky.withdraw()`, before it invokes its `benefit` callback, that is, `Alice.credit`.

`Alice` は, 今度は半分の `500_000` を引き下ろし, `porky` の残高が半分になったことを確認します. `Alice` は `porky.withdraw()` の中で起動された `Alice.credit()` コールバック経由でサイクルを実際に受け取ります. 受け取ったサイクルは `porky.withdraw()` で `benefit` を起動する前に `add` したサイクル, すなわち `Alice.credit` とちょうど同じであることに注意してください.

`Alice` withdraws another `500_000` cycles to wipe out her savings.

`Alice` はまた `500_000` サイクルを引き下ろして, 残高を引き払います.

`Alice` vainly tries to deposit `2_000_000_000` cycles into `porky` but this exceeds `porky`'s capacity by half, so `porky` accepts `1_000_000_000` and refunds the remaining `1_000_000_000` to `Alice`. `Alice` verifies the refund amount (`Cycles.refunded()`), which has (already) been automatically restored to her balance. She also verifies `porky`'s adjusted savings.

`Alice` は`porky` に無謀にも `2_000_000_000` サイクルを預け入れようとしますが, これは `porky` の capacity を超えているので, `porky` は `1_000_000_000` だけを受け付けて, 残りの `1_000_000_000` を `Alice`に払い戻します. `Alice` は払い戻しの量を確認します (`Cycles.refunded()`) が, それはもう既に自動的に彼女の口座に戻されています. 彼女は `porky` の残高も確認します.

`Alice`'s `credit()` function simply accepts all available cycles by calling `Cycles.accept(available)`, checking the actually `accepted` amount with an assert.

`Alice` の `credit()` 関数は単純に全額のサイクルを`Cycles.accept(available)` の呼び出しで渡しており, 実際に `accepted` 額は assert でチェックしています.

:::note

For this example, Alice is using her (readily available) cycles, that she already owns.

この例では, Alice が自分が既に所有している (利用可能な) サイクルを使っています.

:::

:::danger

Because `porky` consumes cycles in its operation, it is possible for `porky` to spend some or even all of Alice’s cycle savings before she has a chance to retrieve them.

`porky` の操作ではサイクルを消費するので, Alice は貯金を取り出す前にいくらか, あるいは全部のサイクルを `porky` のために使い切ってしまう可能性があります.

:::

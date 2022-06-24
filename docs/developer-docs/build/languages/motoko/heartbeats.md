# Heartbeats

Internet Computer canister smart contracts can elect to receive regular heartbeat messages by exposing a particular `canister smart contract_heartbeat` function (see [heartbeat](https://smartcontracts.org/docs/interface-spec/index.html#_heartbeat)).

Internet Computer キャニスタ・スマート・コントラクトは, 特定の `canister smart contract_heartbeat` 関数を公開することで定期的にハートビート・メッセージを受け取るように設定できます.

In Motoko, an actor can receive heartbeat messages by declaring a `system` function, named `heartbeat`, with no arguments, returning a future of unit type (`async ()`).

Motoko では, `heartbeat` という名前で, 引数無し,  単位型のフューチャ (async ()) を返す `system` 関数を宣言するとアクタはハートビート・メッセージを受け取れるようになります.

A simple, contrived example is a recurrent alarm, that sends a message to itself every `n`-th heartbeat:

単純であまり意味のない例ですが, ハートビート `n` 回ごとに自分宛にメッセージを送る繰り返し警報です.

``` motoko
import Debug "mo:base/Debug";

actor Alarm {

  let n = 5;
  var count = 0;

  public shared func ring() : async () {
    Debug.print("Ring!");
  };

  system func heartbeat() : async () {
    if (count % n == 0) {
      await ring();
    };
    count += 1;
  }
}
```

The `heartbeat` function, when declared, is called on every Internet Computer subnet *heartbeat*, by scheduling an asynchronous call to the `heartbeat` function. Due to its `async` return type, a heartbeat function may send further messages and await results. The result of a heartbeat call, including any trap or thrown error, is ignored. The implicit context switch inherent to calling every Motoko async function, means that the time the `heartbeat` body is executed may be later than the time the heartbeat was issued by the subnet.

`heartbeat` 関数が宣言され, `heartbeat` 関数への非同期呼び出しがスケジュールされると, Internet Computer サブネットの _ハートビート_ 毎に呼ばれるようになります. 返値の型が `async` なので, ハートビート関数は返りを待つ間に他にメッセージを送ることができます. ハートビート呼び出しの結果は (トラップやエラーの場合も含めて) 無視されます. Motoko 非同期関数の呼び出し毎について回る暗黙のコンテキスト・スイッチがあるので, `heartbeat` 本体の実行はサブネットによって発行されるハートビートの時刻より遅れる可能性もあります.

As an `async` function, `Alarm`'s `hearbeat` function is free to call other asynchronous functions (the inner call to `ring()` above is an example), as well as shared functions of other canister smart contracts.

`Alarm` の `heartbeat` 関数は `async` なので, 他のキャニスタ・スマート・コントラクトの共有関数と同じく他の非同期関数を呼ぶことができます (上の例では, 内部で `ring()` を呼んでいる).

# Structural equality

Equality (`==`) — and by extension inequality (`!=`) — is **structural**: two values `a` and `b` are equal, `a == b`, whenever they have equal contents, regardless of the physical representation, or identity, of those values in memory.

同等性 (`==`) とさらに非同等性 (`!=`) は **構造的** です. どういうことかと言うと, 二つの値 `a` と `b` が同等である (`a == b`) のは, メモリ中で値が持っている物理的な表現に拘わらず等しい内容 (アイデンティティ) を持つとき, そのときに限る, と言うことです.

For example, the strings `"hello world"` and `"hello " #  "world"` are equal, even though they are most likely represented by different objects in memory.

例えば文字列 `"hello world"` と `"hello " #  "world"` は同等なのですが, たとえメモリ中ではまったく別なオブジェクトによって表現されていたとしてもその同等性は変わりません.

Equality is defined only on `shared` types or on types that don’t contain mutable fields, mutable arrays, non-shared functions, or components of generic type.

同等性は, `共有` 型や可変フィールドを持たない型, 可変配列, 非共有関数, 汎用型のコンポネントに対してのみ定義されています.

For example, we can compare arrays of objects.

例えばオブジェクトの配列を比較することができます.

``` motoko
let a = [ { x = 10 }, { x = 20 } ];
let b = [ { x = 10 }, { x = 20 } ];
a == b;
```

Importantly, this does *not* compare by reference, but by value.

重要なのは, これは参照による比較では _なく_, 値による比較だと言うことです.

## Subtyping

Equality respects subtyping so `{ x = 10 } == { x = 10; y = 20 }` returns `true`.

同等性は副型付けも考慮しています. したがって`{ x = 10 } == { x = 10; y = 20 }` は `true` を返します.

To accommodate subtyping, two values of different types are equal if they are equal at their most specific, common supertype, meaning they agree on their common structure. The compiler will warn in cases where this might lead to subtle unwanted behaviour. For example: `{ x = 10 } == { y = 20 }` will return `true` because the two values get compared at the empty record type. That’s unlikely the intention, so the compiler will emit a warning here.

副型付けに対応するために, 異なる型の二つの値は, もっとも具体的な共通のスーパー型が同等であれば同等になります. つまり, この二つは共通の構造を持っていることになります. 例えば `{ x = 10 } == { y = 20 }` は `true` を返しますが, それは二つの値が空のレコード型として比較されているからです. 多分意図したものではないでしょうから, コンパイラはここで警告を出します.

``` motoko
{ x = 10 } == { y = 20 };
```

## Generic types

It is not possible to declare that a generic type variable is `shared`, so equality can only be used on non-generic types. For example, the following expression generates a warning like this:

総称型の値を `共有` と宣言することはできません. そのため同等性は非総称型に対してのみ用いられます. 例えば次の式は警告を出します.

``` motoko
func eq<A>(a : A, b : A) : Bool = a == b;
```

Comparing these two at the `Any` type means this comparison will return `true` no matter its arguments, so this doesn’t work as one might hope.

この二つを `Any` の型で比較すると言うことは, 引数が何であれこの比較では `true` を返すので, それは期待したものではないでしょう.

If you run into this limitation in your code, you should accept a comparison function of type `(A, A) -> Bool` as an argument, and use that to compare the values instead.

あなたのコードでこの制限に嵌まったときには, 型 `(A, A) -> Bool` の比較関数を引数として受け付け, それを使って値を比較するようにします.

Let’s look at a list membership test for example. This first implementation *does not* work:

リストの所属テストを例に取って見てみましょう. この最初の実装はうごき _ません_.

``` motoko
import List "mo:base/List";

func contains<A>(element : A, list : List.List<A>) : Bool {
  switch list {
    case (?(head, tail))
      element == head or contains(element, tail);
    case null false;
  }
};

assert(not contains(1, ?(0, null)));
```

This assertion will trap because the compiler compares the type `A` at `Any` which is always `true`. So as long as the list has at least one element, this version of `contains` will always return true.

コンピアラは型 `A` と`Any` を比較しようとしますが, それは常に `true` になってしまうので, この表明はトラップを起こします. このリストが少なくとも一つの要素を持つ限り, このヴァージョンの `contains` は常に true を返します.

This second implementation shows how to accept the comparison function explicitly instead:

次の実装は, 代わりに比較関数を明示的に与えるものです:

``` motoko
import List "mo:base/List";
import Nat "mo:base/Nat";

func contains<A>(eqA : (A, A) -> Bool, element : A, list : List.List<A>) : Bool {
  switch list {
    case (?(head, tail))
      eqA(element, head) or contains(eqA, element, tail);
    case null false;
  }
};

assert(not contains(Nat.equal, 1, ?(0, null)));
```
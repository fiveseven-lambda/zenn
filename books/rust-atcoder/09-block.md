---
title: "ブロックとスコープ"
---
# ブロック
if 式について説明したとき，条件式の後や `else` の後に波括弧 `{ }` で囲まれた部分が登場しました．このように `{ }` で囲まれた部分を**ブロック**といいます．

```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    if x > 0 {　// ブロックの始まり
        println!("正");
    } // ブロックの終わり
}
```

`main` 関数の中身も `{ }` で囲まれています．これもブロックです．
```rust
use proconio::input;

fn main() {　// ブロックの始まり
    input! {
        x: i32,
    }
    if x > 0 {
        println!("正");
    }
} // ブロックの終わり
```

if 式のブロックは条件を満たすときのみ実行されましたが，単にブロックだけを書けば常に実行されます．
```rust
fn main() {
    println!("Good morning");
    {
        println!("Good afternoon");
        println!("Good evening");
    }
    println!("Good night");
}
```
`main` 関数の中身は，
1. `println!("Good morning");` という文
1. `{ println!("Good afternoon"); println!("Good evening"); }` というブロック
1. `println!("Good night");` という文

の 3 つです．  2. のブロックの中身は，
1. `println!("Good afternoon");` という文
1. `println!("Good evening");` という文

の 2 つです．

`main` 関数全体を表すブロックも含めて，ブロックの中身は*上から順に*実行されるので，出力は
```:標準出力
Good morning
Good afternoon
Good evening
Good night
```
となります．
# スコープ
あるブロックの中で変数を宣言するとします．すると，ブロックの外側ではその変数を使うことができません．
```rust:コンパイルエラー
fn main() {
    {
        let hoge = 10;
    }
    println!("{}", hoge);
}
```
次のようなエラーになります．
```
error[E0425]: cannot find value `hoge` in this scope
 --> src/main.rs:5:20
  |
5 |     println!("{}", hoge);
  |                    ^^^^ not found in this scope
```
``cannot find value `hoge` in this scope`` は「`hoge` という変数が見つからない」という意味です．ブロックの外側では， `hoge` という変数が存在しないことになっています．

逆に，ブロックの外側で宣言した変数をブロックの内側で使うことはできます．
```rust
fn main() {
    let hoge = 10;
    {
        println!("{}", hoge);
    }
}
```

これを，変数の**スコープ**といいます．あるブロックの内部で宣言された変数のスコープは，*変数が宣言されてから，そのブロックが終了するまで*です．変数をそのスコープの外側で使うことはできません．
```rust
fn main() {
    let hoge = 10; // hoge のスコープの開始
    {
        println!("{}", hoge);
        let fuga = 20; // fuga のスコープの開始
        println!("{}", fuga);
    } // ブロックの終了：fuga のスコープの終了
    println!("{}", hoge);
} // ブロックの終了：hoge のスコープの終了
```
# シャドーイング
同じ名前の変数を，複数回宣言することができます．
```rust
fn main() {
    let hoge = 10;
    println!("{}", hoge);
    let hoge = 20;
    println!("{}", hoge);
}
```
このとき， 1 つめの `hoge` と 2 つめの `hoge` は，*名前は同じでも別の変数*です．

1 回目の `println!` では， 1 つめの `hoge` の値が出力されます．一方， 2 回目の `println!` では， 2 つめの `hoge` の値が出力されます．よって出力は `10` `20` となります．

このように，同じ名前の変数を宣言すると，前の変数が使えなくなります．これを変数の**シャドーイング**といいます．

シャドーイングをブロックの中で行うと，
```rust
fn main() {
    let hoge = 10;
    {
        println!("{}", hoge);
        let hoge = 20;
        println!("{}", hoge);
    }
    println!("{}", hoge);
}
```
ブロックを抜けた後は 2 つめの `hoge` が存在しないので，最後の `println!` では 1 つめの `hoge` の値が出力されます．よって出力は `10` `20` `10` となります．

2 つの `hoge` は別の変数なので，型が違っても構いません．
```rust
fn main() {
    let hoge: i32 = 10;
    println!("{}", hoge);
    let hoge: f64 = 20.;
    println!("{}", hoge);
}
```
# 値を返すブロック
次のコードを見てください．
```rust
fn main() {
    println!("ブロックの前");
    let hoge = {
        println!("ブロックの中");
        10
    };
    println!("ブロックの後， hoge の値は {}", hoge);
}
```
なにやらブロックが不思議な使われ方をしています．`{ }` の中に， `println!("ブロックの中");` という文と， `10` という式が書かれていて， `10` にはセミコロン `;` が付いていません．

このように書くと，ブロックが実行されたとき，ブロック内の最後の式の値がブロック全体の値となります．これを「ブロックが値を返す」と言います．

今回，ブロックの最後に `10` と書いてあるので，このブロックが実行されると，最後に 10 という値を返します．すると， `let hoge = 10;` と書いたのと同じように， `hoge` に 10 が代入されます．

よって，これを実行すると，
1. `ブロックの前` と出力される
1. ブロックが始まる
1. `ブロックの中` と出力される
1. 10 という値を返してブロックが終わる
1. ブロックの返した値 10 が `hoge` に代入される
1. `ブロックの後， hoge の値は 10` と出力される

となります．

また，今回は `}` の後にセミコロンを付けないと
```rust:コンパイルエラー
fn main() {
    println!("ブロックの前");
    let hoge = {
        println!("ブロックの中");
        10
    }
    println!("ブロックの後， hoge の値は {}", hoge);
}
```
コンパイルエラーになります．
```
error: expected `;`, found `println`
 --> src/main.rs:6:6
  |
6 |     }
  |      ^ help: add `;` here
7 |     println!("ブロックの後， hoge の値は {}", hoge);
  |     ------- unexpected token
```
`` expected `;`, found `println` `` は，「`}` の後に `;` が来るはずだったのに，実際には `println` があった」という意味です． ``add `;` here`` は「ここにセミコロン `;` を付けてはどうか」という提案です．
# 値を返す if 式
前の章の最後に，
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    let abs;
    if x >= 0 {
        abs = x;
    } else {
        abs = -x;
    }
    println!("絶対値は{}です", abs);
}
```
というコードを書きました．しかし， if 式のブロックも値を返すことができるため，これは次のように書き換えることができます．
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    let abs;
    abs = if x >= 0 { x } else { -x };
    println!("絶対値は{}です", abs);
}
```
条件 `x >= 0` が正しければ，ブロック `{ x }` が実行され， x の値が返されます．一方，条件 `x >= 0` が正しくなければ，ブロック `{ -x }` が実行され， -x の値が返されます．よって， `abs` には `x` の絶対値が代入されます．

今回も `}` の後にセミコロンを付けないとコンパイルエラーになります．
## 型
今のコードを，型に着目して見てみましょう． `abs` の型が注釈されていないため，周囲から推論されています．

if 式の 1 つめのブロックが返す `x` は `i32` 型です．また， 2 つめのブロックが返す `-x` も `i32` 型です．よって，この if 式が返す値は常に `i32` 型であると分かるので， `abs` は `i32` 型であると推論されます．

もし 1 つめのブロックと 2 つめのブロックの返す値の型が異なると，
```rust:コンパイルエラー
fn main() {
    let x = 0;
    let y = if x >= 0 { 10 } else { 10. };
}
```
エラーになります．
```
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:3:37
  |
3 |     let y = if x >= 0 { 10 } else { 10. };
  |                         --          ^^^ expected integer, found floating-point number
  |                         |
  |                         expected because of this
```
`` `if` and `else` have incompatible types `` は「`if` と `else` の型が相容れない」という意味です．後者の `10.` を指して `expected integer, found floating-point number` （「整数が来るはずだったが，浮動小数点数が来た」）と書かれ，さらに整数が来るはずだった理由として前者の `10` が示されています．

また， `else` の付いていない `if` だけのブロックで値を返すこともできません．
# 練習問題
- [ABC170 A - Five Variables](https://atcoder.jp/contests/abc170/tasks/abc170_a)
  [解答例 1](https://atcoder.jp/contests/abc170/submissions/19109885) と[解答例 2](https://atcoder.jp/contests/abc170/submissions/19109908)．どちらが見やすいと感じるかは人によるでしょう．

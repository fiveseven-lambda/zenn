---
title: "入力の受け取り"
---
:::message
対応する APG4b の箇所：[1.05.実行順序と入力](https://atcoder.jp/contests/apg4b/tasks/APG4b_f)
:::
# 入力
[この問題](https://atcoder.jp/contests/apg4b/tasks/APG4b_cr)を見てみましょう．
> 問題文
> 2 つの整数 $A$, $B$ が与えられます。 $A + B$ の計算結果を出力してください。 

競技プログラミングでは，このように「入力を受け取って，計算をして，結果を出力する」という問題が出題されます（例外としてインタラクティブ形式の問題があります）．このためにはまず入力を受け取る手段を持っていなければなりません．

入力を受け取るには，次のように書きます．
```rust
proconio::input! {
    a: i32,
    b: i32,
}
```
**`proconio::input!` マクロ**を使っています．こう書くことで， 2 つの整数が読み込まれて，それぞれ `a` ， `b` という名前の変数として使えるようになります．

つまり，
```rust
fn main() {
    proconio::input! {
        a: i32,
        b: i32,
    }
    println!("{}", a + b);
}
```
とすれば， `a` と `b` が読み込まれた後， a + b が計算されて出力されます．これでさっきの問題を解くことができたわけです．[ここ](https://atcoder.jp/contests/apg4b/tasks/APG4b_cr#sourceCode)から提出し， AC になることを確認してください．
# use
また， `fn main() { }` の外側に
```rust
use proconio::input;
```
と書いておくと， `proconio::input!` の代わりに `input!` と書くだけで済むようになります．普通
```rust
use proconio::input;

fn main() {
    input! {
        a: i32,
        b: i32,
    }
    println!("{}", a + b);
}
```
のようにファイルの先頭に書きます．
# 型の指定
`proconio::input!` マクロの中の `i32` の部分を変えると，他の型の値も読み込むことができます．たとえば
```rust
use proconio::input;

fn main() {
    input! {
        value: f64,
    }
    println!("{}", value / 2.);
}
```
とすると `f64` 型の値が変数 `value` として読み込まれ，それを 2. で割った値が出力されます．

[コードテスト](https://atcoder.jp/contests/practice/custom_test)で「標準入力」の欄に適当な小数を入れて実行し，結果が正しく出力されることを確認してください．

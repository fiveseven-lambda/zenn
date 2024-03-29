---
title: "入力の受け取り"
---
# 入力
この章からは実際に競技プログラミングの問題を解き始めます．まずは[この問題](https://atcoder.jp/contests/abc180/tasks/abc180_a)を見てみましょう．

> 問題文 (ABC180 A - box)
> $N$ 個のボールが入っていた箱から $A$ 個のボールを取り出し、新たに $B$ 個のボールを入れました。今、箱にはボールが何個入っていますか?

「入力例 1 」と「出力例 1 」を見てください．入力が
```:stdin
100 1 2
```
であれば， 100 個のボールが入っていた箱から 1 個のボールを取り出し，新たに 2 個のボールを入れたので，箱には 101 個のボールが入っています．よって出力は
```:stdout
101
```
となります．このような計算を行うプログラムを書くことができれば， AC （正解）です．

この問題を解くためには，まず標準入力から数を受け取る必要があります．入力を受け取るには， **`proconio::input!` マクロ**を使います．

:::message
AtCoder 上の環境では，何もしなくても `proconio::input!` マクロを使うことができます．一方，手元で環境構築をしている場合は，下で説明するように `Cargo.toml` を編集しなければいけません．
:::
```rust
proconio::input! {
    n: i32,
    a: i32,
    b: i32,
}
```
こう書くことで， 3 つの整数が読み込まれて，それぞれ `n` ， `a` ， `b` という名前の変数として使えるようになります．たとえば上の入力例 1 が与えられたときは， `n` の値が 100 ， `a` の値が 1 ， `b` の値が 2 になるわけです．

つまり，
```rust
fn main() {
    proconio::input! {
        n: i32,
        a: i32,
        b: i32,
    }
    println!("{}", n - a + b);
}
```
とすれば， `n` ， `a` ， `b` を読み込んだ後， $n - a + b$ を計算して出力するようになります．これでさっきの問題を解くことができました．[ここ](https://atcoder.jp/contests/abc180/submit?taskScreenName=abc180_a)から提出し， AC になることを確認してください．

:::message
`proconio::input!` マクロを書くだけで変数 `n` ， `a` ， `b` が使えるようになるため，今回 `let n;` のような宣言を書く必要はありません．
:::
# `Cargo.toml` （手元で環境構築をしている場合）
`proconio::input!` マクロは， [crates.io](https://crates.io) にある [`proconio`](https://crates.io/crates/proconio) というクレートの中のマクロです．よって，このマクロを使うプログラムをビルドするときは `proconio` クレートのダウンロードとコンパイルも必要になります．

`proconio` にもバージョンがあります．これは， Rust 自体のバージョンとは別のものです．[ここ](https://github.com/rust-lang-ja/atcoder-rust-resources/wiki/2020-Update#proconio)を見ると， AtCoder の環境における `proconio` のバージョンは 0.3.6 であるということが分かります．そこで，手元の環境でもバージョン 0.3.6 の `proconio` を使えるようにします．

`cargo` に対して `proconio` クレートのバージョン 0.3.6 を使うよう命令するには，[第 2 章](https://zenn.dev/toga/books/rust-atcoder/viewer/02-environment-setup)で述べた `Cargo.toml` を編集して， `[dependencies]` と書かれた行の後に `proconio = "0.3.6"` という行を追加します．
```
[dependencies]
proconio = "0.3.6"
```
これで， `proconio::input!` マクロが使えるようになります．

# use
`fn main() { }` の外側に
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

このとき，たとえ標準入力が整数であっても `value` は浮動小数点数として読み込まれます．よって，標準入力が `10` であれば `value` は 10.0 となります．

標準入力として適当な整数や小数を与えて実行してみて，結果が正しく出力されることを確認してください．
# 練習問題
ここまでの内容を使って解ける問題です．
- [ABC161 A - ABC Swap](https://atcoder.jp/contests/abc161/tasks/abc161_a)
- [ABC184 A - Determinant](https://atcoder.jp/contests/abc184/tasks/abc184_a)
- [ABC172 A - Calc](https://atcoder.jp/contests/abc172/tasks/abc172_a)
- [ABC182 A - twiblr](https://atcoder.jp/contests/abc182/tasks/abc182_a)
- [ABC186 A - Brick](https://atcoder.jp/contests/abc186/tasks/abc186_a)
- [ABC176 A - Takoyaki](https://atcoder.jp/contests/abc176/tasks/abc176_a)
  切り上げ割り算に注意します．[解答例](https://atcoder.jp/contests/abc176/submissions/19108814)．
- [ABC163 A - Circle Pond](https://atcoder.jp/contests/abc163/tasks/abc163_a)
  入力が整数として与えられるとはいえ，途中で小数の計算が登場するので，かまわず `f64` 型として受け取るのが良いでしょう．また，円周率は， 3.14……と書く代わりに `std::f64::consts::PI` と書くことができます．[解答例](https://atcoder.jp/contests/abc163/submissions/19108930)．
- [ABC183 B - Billiards](https://atcoder.jp/contests/abc183/tasks/abc183_b) / [解答例](https://atcoder.jp/contests/abc183/submissions/19519502)
- [ABC168 C - : (Colon)](https://atcoder.jp/contests/abc168/tasks/abc168_c) （余弦定理を知っていることが前提になります）
  `f64` 型変数 $x$ があったとき， `x.sqrt()` は $\sqrt x$， `x.cos()` は $\cos x$ になります．
  複雑な計算は適宜変数をおくのが大事です．[解答例](https://atcoder.jp/contests/abc168/submissions/25462405)．

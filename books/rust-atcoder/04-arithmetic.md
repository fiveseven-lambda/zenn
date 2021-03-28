---
title: "四則演算"
---
# マクロ
前章で登場した `println!` `print!` `eprintln!` `eprint!` `panic!` は，全て「マクロ」だといいました．ここでマクロについて少しだけ説明しておきます．

様々なマクロがあり，各々のマクロには何らかの仕事が割り当てられています．たとえば， `println!` マクロを使うと標準出力に何かを出力して改行することができます．何を出力するかは，その後の丸括弧 `( )` の中に書きます．

マクロを使うときには，必ず後に丸括弧 `( )` か波括弧 `{ }` か角括弧 `[ ]` を付け，マクロにさせる仕事の内容をその中に書きます．これをマクロの**呼び出し**といいます．

`println!` `print!` `eprintln!` `eprint!` `panic!` を呼び出すときは，[第 25 章](https://zenn.dev/toga/books/rust-atcoder/viewer/25-function)で説明する「関数」と見た目を似せるために丸括弧 `( )` を使うのが普通です．また，括弧内に書く中身のことも，関数に合わせて引数（ひきすう）と呼びます．たとえば，
```rust
println!("Hello, world!");
```
というマクロ呼び出しにおける引数は，文字列リテラル `"Hello, world!"` です．

:::message
名前の最後に `!` が付くのは，マクロであるというサインです．
:::

# 値の出力
今までは， `println!` `print!` `eprintln!` `eprint!` マクロで単に文字列のみを出力していました．しかし，これらのマクロはこんなこともできます．
```rust
fn main() {
    println!("最小の素数は{}です．", 2);
}
```
括弧の中に，コンマ `,` で区切って 2 つの引数を書いています． 1 つめの引数は文字列リテラル `"最小の素数は{}です"` で， 2 つめの引数は整数 `2` です．このようにプログラム中に直接書かれた整数を**整数リテラル**といいます．

文字列リテラルを見ると，その中に `{}` という部分が存在します．これは**プレースホルダー**と呼ばれます． `println!` マクロは，プレースホルダーが存在すると，その部分に 2 つめの引数の値を埋め込んで出力します．つまり，今回は `{}` の部分が `2` で置き換えられて，`最小の素数は2です．` と出力されることになります．

今回 `{}` の括弧の中身は空ですが，この中で詳細なフォーマットを指定することもできます．とりあえず今は `{}` が値で置き換えられるということだけを覚えておいてください．

単に値だけを出力して改行したいときは
```rust
fn main() {
    println!("{}", 2);
}
```
とすれば良いです．

次のコードはエラーになります．
```rust
fn main() {
    println!("{}");
}
```
プレースホルダーだけがあって，そこに置き換わる値が存在しないからです．エラーメッセージは
```
error: 1 positional argument in format string, but no arguments were given
 --> src/main.rs:2:15
  |
2 |     println!("{}");
  |               ^^
```
のようになります．

また，プレースホルダーは複数個あってもかまいません．
```rust
fn main() {
    println!("{}の次の完全数は{}です．", 6, 28);
}
```
今回は「文字列リテラル `"{}の次の完全数は{}です．"`」「整数リテラル `6`」「整数リテラル `28`」の 3 つの引数があります．1 つ目のプレースホルダーは 6 ， 2 つ目のプレースホルダーは 28 で置き換えられ，`6の次の完全数は28です．`と出力されます．
# 整数の四則演算
整数同士で演算を行うことができます．ここでは次の 5 つを紹介します．
| 演算子 | 意味 |
| -- | -- |
| `+` | 足し算 |
| `-` | 引き算 |
| `*` | かけ算 |
| `/` | 割り算の商 |
| `%` | 割り算のあまり |

たとえばこんなコードが書けます．
```rust
fn main() {
    println!("2 + 3 = {}", 2 + 3);
}
```
このコードでは， `println!` マクロの引数は「文字列リテラル `"2 + 3 = {}"` 」と「式 `2 + 3` 」の 2 つです．式 `2 + 3` が計算されて 5 という値になり，文字列リテラルのプレースホルダー `{}` に埋め込まれて，出力される内容は `2 + 3 = 5` となります．

文字列リテラル中の `2 + 3` は，計算されません．

:::message
このコードでは，コンマ `,` の直後や演算子 `+` の前後に空白文字が入っています．この空白文字は書かなくても動きますが，あった方が見やすいため入れるのが良いとされています．
:::

同様に，引き算やかけ算は
```rust
fn main() {
    println!("5 - 2 = {}", 5 - 2);
    println!("3 * 4 = {}", 3 * 4);
}
```
のようにできます．

割り算については，次のようになります．
```rust
fn main() {
    println!("14を3で割ると，商が{}であまりが{}です．", 14 / 3, 14 % 3);
}
```
プレースホルダーが 2 つあるので， 1 つ目のプレースホルダーは `14 / 3` の値， 2 つ目のプレースホルダーは `14 % 3` の値で置き換えられます．

`14 / 3` は割り算の商， `14 % 3` は割り算のあまりです．よって今回は `14を3で割ると，商が4であまりが2です．` と出力されることになります．

`14 / 3` が $\frac{14}{3} = 4.666\ldots$ にならないことに注意してください．
# 優先順位
たとえば，
```rust
fn main() {
    println!("{}", 5 + 3 * 2);
}
```
のように，足し算とかけ算が混ざった式を書くと，かけ算の方が先に計算され，結果は 5 + 6 = 11 になります．5 + 3 を先に計算したければ，括弧でくくります．
```rust
fn main() {
    println!("{}", (5 + 3) * 2);
}
```

これを演算子の**優先順位**といいます． `*` は `+` より優先順位が高いということになります．`+` と `-` の優先順位は同じです．`*` と `/` と `%` の 3 つの優先順位は同じです．

今後他の演算子も登場しますが，全て優先順位が定まっています．

また，優先順位が同じ演算子が並んでいた場合，**結合順序**に従って順序が決まります． `+` `-` `*` `/` `%` の 5 つは**左結合**といって，優先順位が同じなら左から順に計算されます．すなわち
```rust
fn main() {
    println!("{}", 7 / 2 * 2);
}
```
は，まず 7 / 2 が計算されて 3 となり，その後に 3 * 2 が計算されて結果は 6 となります．
# 小数の四則演算
整数の代わりに `1.2` のような小数を書くと，小数になります．小数同士でも四則演算を行うことができます．
| 演算子 | 意味 |
| -- | -- |
| `+` | 足し算 |
| `-` | 引き算 |
| `*` | かけ算 |
| `/` | 割り算 |
| `%` | 割った余り |

注意するのは `/` と `%` です．
```rust
fn main() {
    println!("{}", 6.5 / 2.5);
}
```
は，普通の割り算が行われて `2.6` と出力されます．一方，
```rust
fn main() {
    println!("{}", 6.5 % 2.5);
}
```
は余りが計算されて `1.5` と出力されます．
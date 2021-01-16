---
title: "パターンマッチと条件分岐"
---
# 論駁可能なパターン
今までに出てきたパターンは，全て「必ず成功する」パターンでした．
```rust
let (x, y) = (1, 2);
// イコールの右側に要素数 2 のタプルがあれば
// パターンマッチは必ずうまくいく
```

一方，「必ず成功するとは限らない」パターンがあります．その一つがスライスパターンです．

たとえば， `[x, y, z]` がスライスパターンです．長さ 3 のスライスが与えられれば，その各要素が `x` `y` `z` に代入されます．一方，長さが 3 でないスライスが与えられると，パターンマッチは失敗します． `[T]` という型自体には長さの情報が含まれないため，型からスライスの長さが 3 であるか判断することはできません．

このように，失敗する可能性のあるパターンを，**論駁可能なパターン**といいます．これに対し，今までの「必ず成功する」パターンを**論駁不可能なパターン**といいます．

この章では，論駁可能なパターンの使い方を紹介します．
# `if let` 式
論駁可能なパターンを使う状況の一つに， **`if let` 式**があります．

```rust
fn main() {
    let ref_slice: &[i32] = &[10, 15, 20];
    if let [x, y, z] = *ref_slice {
        println!("{} {} {}", x, y, z);
    } else {
        println!("マッチに失敗しました");
    }
}
```
`if` の直後に， `let` 文と同じ形式で `let [x, y, z] = *ref_slice` と書かれています． `=` の右がスライスになっています（スライス型の変数を作ることはできないので，一旦スライス参照型の変数 `ref_slice` を作ってから参照外ししています）．

こう書くと，スライス `*ref_slice` が `[x, y, z]` にマッチするとき，すなわち `*ref_slice` の長さがちょうど 3 であるとき*のみ*，パターンマッチが行われてブロックの中身が実行されます．

今回は `*ref_slice` の長さが 3 なので， `x` ， `y` ， `z` の値がそれぞれ 10, 15, 20 になって `println!` が実行されます．よって出力は `10 15 20` となります．

一方， `*ref_slice` の長さが 3 以外だと
```rust
fn main() {
    let ref_slice: &[i32] = &[10, 15]; // 長さ 2
    if let [x, y, z] = *ref_slice {
        println!("{} {} {}", x, y, z);
    } else {
        println!("マッチに失敗しました");
    }
}
```
`if` ではなく `else` の後のブロックが実行されて， `マッチに失敗しました` と出力されます．
# 様々なパターン
## リテラルパターン
論駁可能なパターンの中では，リテラルを書くことができます．
```rust
use proconio::input;

fn main() {
    input! {
        vector: [(i32, i32); 5],
    }
    for &tuple in &vector {
        if let (1, value) = tuple {
            println!("{}", value);
        } else if let (2, value) = tuple {
            println!("{}", value * value);
        } else if let (0, 0) = tuple {
            break;
        } else {
            println!("?");
        }
    }
}
```
ループの中で，入力としてタプルを受け取っています．

パターン `(1, value)` は，タプルの 1 番目の要素が 1 であったとき，マッチに成功して 2 番目の要素が `value` に代入されます．パターン `(0, 0)` は，タプルの要素がどちらも 0 だったときのみマッチに成功し，代入は起こりません．

このプログラムに入力として
```-:標準入力
1 5
2 4
5 2
0 0
1 2
```
を与えると，
1. タプル $(1, 5)$ は `(1, value)` にマッチするので， `value` の値がそのまま出力される．
1. タプル $(2, 4)$ は `(2, value)` にマッチするので， `value` の値の 2 乗が出力される．
1. タプル $(5, 2)$ はいずれにもマッチしないので， `?` と出力される．
1. タプル $(0, 0)$ は `(0, 0)` にマッチするので，ループが終了する．

となって全体では
```-:標準出力
5
16
?
```
と出力されます．
:::message
このコードは後述する `match` 式を使うと綺麗に書き直せます．
:::
## 複数のパターン
パターンは， `|` でつないで複数個書くことができます．
```rust
fn main() {
    let array = [(1, 92), (3, 91), (2, 95)];
    let mut vector = Vec::new();
    for tuple in &array {
        if let (1, value) | (2, value) = *tuple {
            vector.push(value);
        }
    }
    assert_eq!(vector, vec![92, 95]);
}
```
`if let` の後に， `(1, value)` と `(2, value)` という 2 つのパターンを書いています．
- `(1, value)` にマッチすれば， `value` にタプルの 2 つめの要素が代入されてブロックの中身が実行されます．
- `(2, value)` にマッチすれば， `value` にタプルの 2 つめの要素が代入されてブロックの中身が実行されます．
- どちらにもマッチしなければ，ブロックの中身は実行されません．

複数のパターンの間で使われている変数が一致していなかった場合エラーになります．
```rust
fn main() {
    let tuple = (3, 2, 1);
    if let (x, 0, 0) | (x, y, 1) | (x, y, 2) = tuple {
        println!("{} {}", x, y); // y は使える？使えない？
    }
}
```
`(x, y, 1)` と `(x, y, 2)` の中には変数 `y` が現れているのに， `(x, 0, 0)` の中には現れていないので，エラーになります．

パターンの間で変数の型が一致していないときもエラーになります．
```rust
fn main() {
    let tuple: (i32, f64) = (1, 2.0);
    if let (1, x) | (x, 2.0) = tuple {
        println!("{}", x); // x は i32 ？ f64 ？
    }
}
```
## 範囲パターン
`..=` を用いて，ある範囲にマッチするパターンを書くことができます．
```rust
fn main() {
    let tuple = (1, 2);
    if let (0..=5, x) = tuple {
        assert_eq!(x, 2);
    } else {
        panic!();
    }
}
```
`(0..=5, x)` は，一つ目の要素が 0 以上 5 以下であるようなタプルにマッチします．
## ワイルドカードパターン
```rust
fn main() {
    let tuple = (3, 1, 2);
    if let (1, x, y) | (x, 1, y) | (x, y, 1) = tuple {
        println!("少なくとも 1 つが 1");
    }
}
```
`if` ブロックの中身は， `(1, x, y)` `(x, 1, y)` `(x, y, 1)` のいずれかにマッチするとき，すなわち 3 つの要素のうち少なくとも 1 つが 1 であるようなときに実行されます．

しかし，ブロックの中で `x` `y` を使う必要が無ければ，パターンの中の `x` `y` は無駄になってしまいます．このような場合は， `x` `y` を `_` で置き換えます．

```rust
fn main() {
    let tuple = (3, 1, 2);
    if let (1, _, _) | (_, 1, _) | (_, _, 1) = tuple {
        println!("少なくとも 1 つが 1");
    }
}
```

`_` は任意の値にマッチしますが， `_` に対して代入された値は捨てられ，ブロックの中で使うことができません． `_` をワイルドカードパターンといいます．

ワイルドカードパターンは， `for` 式で繰り返しの回数だけ指定するときにも使えます．
```rust
fn main() {
    for _ in 0..4 {
        println!("Knock, knock, knockin' on heaven's door");
    }
}
```
`Knock, knock, knockin' on heaven's door` と 4 回出力します．
## レストパターン
次のコードを見てください．
```rust
fn main() {
    let ref_slice: &[i32] = &[10, 20, 30];
    if let [first, second, ..] = *ref_slice {
        println!("最初の要素： {}", first);
        println!("2 番目の要素： {}", second);
    } else {
        println!("長さ 1 以下のスライスです");
    }
}
```
`[first, second, ..]` というパターンは，スライスの長さが 2 以上であればマッチに成功し， `first` `second` にそれぞれ最初の要素と次の要素が代入されます．一方，スライスの長さが 1 以下であればマッチに失敗します．

このように，「残りの要素」を表す `..` は**レストパターン**といいます．

レストパターンは，スライスパターンの途中や
```rust
fn main() {
    let ref_slice: &[i32] = &[10, 20, 30];
    if let &[first, .., last] = ref_slice {
        assert_eq!(first, 10);
        assert_eq!(last, 30);
    } else {
        panic!();
    }
}
```
最初でも使えます．
```rust
fn main() {
    let ref_slice: &[i32] = &[10, 20, 30];
    if let &[.., second_to_last, last] = ref_slice {
        assert_eq!(second_to_last, 20);
        assert_eq!(last, 30);
    } else {
        panic!();
    }
}
```

スライスパターンの中でレストパターンを二度以上使おうとすると，
```rust
fn main() {
    let ref_slice: &[i32] = &[10, 20, 30];
    if let &[a, .., b, ..] = ref_slice {
        println!("{} {}", a, b);
    }
}
```
マッチの仕方が曖昧になってしまうのでエラーになります．
```
error: `..` can only be used once per slice pattern
 --> src/main.rs:3:24
  |
3 |     if let &[a, .., b, ..] = ref_slice {
  |                 --     ^^ can only be used once per slice pattern
  |                 |
  |                 previously used here
```
`` `..` can only be used once per slice pattern `` は，「`..` はスライスパターン一つにつき一回しか使えない」という意味です．
# `while let` 式
`if let` 式は「マッチしたらブロックの中身が実行される」ものでした．一方， `while let` 式は「マッチしている間ブロックの中身が実行され続ける」ものです．
```rust
fn main() {
    let array = [0, 0, 0, 1, 2];
    let mut ref_slice = &array[..];
    while let [0, ..] = *ref_slice {
        ref_slice = &ref_slice[1..];
    }
    assert_eq!(ref_slice, [1, 2]);
}
```
1. はじめ `ref_slice` は `array` の全体 `[0, 0, 0, 1, 2]` のスライスへの参照です．
1. `[0, 0, 0, 1, 2]` は `[0, ..]` にマッチするので，ブロックが実行されます．
   1. `ref_slice[1..]` は `array` の `[0, 0, 1, 2]` の範囲を指すスライスです．これが `ref_slice` に代入されます．
1. `[0, 0, 1, 2]` は `[0, ..]` にマッチするので，ブロックが実行されます．
   1. `ref_slice[1..]` は `array` の `[0, 1, 2]` の範囲を指すスライスです．これが `ref_slice` に代入されます．
1. `[0, 1, 2]` は `[0, ..]` にマッチするので，ブロックが実行されます．
   1. `ref_slice[1..]` は `array` の `[1, 2]` の範囲を指すスライスです．これが `ref_slice` に代入されます．
1. `[1, 2]` は `[0, ..]` にマッチしないので，ブロックが実行されずにループが終了します．

# `match` 式
これは，上で登場したコードです．
```rust
use proconio::input;

fn main() {
    input! {
        vector: [(i32, i32); 5],
    }
    for &tuple in &vector {
        if let (1, value) = tuple {
            println!("{}", value);
        } else if let (2, value) = tuple {
            println!("{}", value * value);
        } else if let (0, 0) = tuple {
            break;
        } else {
            println!("?");
        }
    }
}
```
3 つのパターン `(1, value)` `(2, value)` `(0, 0)` について， `tuple` がマッチするかどうか順に調べています．

このように，一つの値といくつかのパターンについてマッチするか順に調べていくような場合は， **`match` 式**を使うことで次のように書けます．
```rust
use proconio::input;

fn main() {
    input! {
        vector: [(i32, i32); 5],
    }
    for &tuple in &vector {
        match tuple {
            (1, value) => println!("{}", value),
            (2, value) => println!("{}", value * value),
            (0, 0) => break,
            _ => println!("?"),
        }
    }
}
```
`match` の直後に式 `tuple` が置かれ，その後の `{ }` で囲まれた部分に
```rust
パターン => 式,
```
が並んでいます． `match` 式内の各パターンを**アーム**といいます．上から順にパターンマッチが成功するか確かめられ，マッチしたアームについて式の内容が実行されます．

式の終わりがセミコロン `;` ではなくカンマ `,` であることに注意してください．

## カンマ `,` の省略
`=>` の後がブロックであるような場合は，カンマ `,` を省略できます．
```rust
use proconio::input;

fn main() {
    loop {
        input! {
            x: i32,
        }
        match x {
            0 => {
                break;
            }
            n => {
                for _ in 0..n {
                    print!("!");
                }
                println!();
            }
        }
    }
}
```
## 値を返す `match` 式
`match` 式も値を返すことができます．
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    let y = match x {
        0 => 1,
        1 => 0,
        _ => {
            println!("neither 0 nor 1");
            0
        }
    };
    println!("{}", y);
}
```

## `unreachable!()`
`match` 式のアームは，全ての場合をカバーしなければなりません．例えば，上の例で最後のアームを消すと
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    let y = match x {
        0 => 1,
        1 => 0,
    };
    // x が 0 でも 1 でもないとき，
    // y の値は何になるのか？
    println!("{}", y);
}
```
エラーになります．
```
error[E0004]: non-exhaustive patterns: `i32::MIN..=-1_i32` and `2_i32..=i32::MAX` not covered
 --> src/main.rs:7:19
  |
7 |     let y = match x {
  |                   ^ patterns `i32::MIN..=-1_i32` and `2_i32..=i32::MAX` not covered
  |
  = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
  = note: the matched value is of type `i32`
```
`non-exhaustive patterns` は「網羅的でないパターン」という意味です．

これは，理論上他の場合が存在しないという場合であっても起こることがあります．
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    match x % 3 {
        0 => println!("3 の倍数"),
        1 | -2 => println!("1 を引くと 3 の倍数"),
        2 | -1 => println!("1 を足すと 3 の倍数"),
    }
}
```
`x % 3` という式が -3 以下の値や 3 以上の値になることはありません．しかし，自動的にそのような判定がなされることは無く，エラーになってしまいます．
```
error[E0004]: non-exhaustive patterns: `i32::MIN..=-3_i32` and `3_i32..=i32::MAX` not covered
 --> src/main.rs:7:11
  |
7 |     match x % 3 {
  |           ^^^^^ patterns `i32::MIN..=-3_i32` and `3_i32..=i32::MAX` not covered
  |
  = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
  = note: the matched value is of type `i32`
```
このような場合は，最後にワイルドカードパターンを追加し， `unreachable!()` という式を書きます．
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    match x % 3 {
        0 => println!("3 の倍数"),
        1 | -2 => println!("1 を引くと 3 の倍数"),
        2 | -1 => println!("1 を足すと 3 の倍数"),
        _ => unreachable!(),
    }
}
```
`unreachable!()` には到達しえないものとみなされ，エラーが無くなります．

もし `unreachable!()` と書いた部分に到達してしまったら，実行時エラーになります．
## `match` 式の返す値
`match` 式の返す値は，全て同じ型でなければなりません．次のコードはエラーになります．
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    let y = match x {
        0 => 2i32, // i32
        _ => 2.5f64, // f64
    }; // y の型は？
}
```
```
error[E0308]: `match` arms have incompatible types
  --> src/main.rs:9:14
   |
7  |       let y = match x {
   |  _____________-
8  | |         0 => 2i32,
   | |              ---- this is found to be of type `i32`
9  | |         _ => 2.5f64,
   | |              ^^^^^^ expected `i32`, found `f64`
10 | |     };
   | |_____- `match` arms have incompatible types
```
ただし， `=>` の右が `break` 式や `continue` 式， `unreachable!()` などの場合は値を返す必要がありません．
```rust
use proconio::input;

fn main() {
    loop {
        input! {
            x: i32,
        }
        let y = match x {
            0 => break,
            x => x - 1,
        };
        println!("{}", y);
    }
}
```
## マッチガード
パターンと `=>` の間に， `if` を使って条件を書くことができます．
```rust
fn main() {
    let tuple = (1, 3);
    match tuple {
        (1, x) if x % 2 == 0 => println!("{}", x),
        _ => {}
    }
}
```
タプル `(1, 3)` はパターン `(1, x)` にマッチしますが，その後の条件 `x % 2 == 0` を満たさないので，何も出力されません．
## `@` 演算子
次のプログラムは，タプル `tuple` の最初の要素が 1 のとき，次の要素の桁数を出力します．
```rust
fn main() {
    let tuple = (1, 50);
    match tuple {
        (1, 0..=9) => println!("1 桁"),
        (1, 10..=99) => println!(" 2 桁"),
        (1, 100..=std::i32::MAX) => println!("3 桁以上"),
        (1, _) => println!("負"),
        _ => {}
    }
}
```
ここで， `1 桁` や `2 桁` ではなく `7 は 1 桁` ， `50 は 2 桁` のように出力するには，どうすればよいでしょうか．

パターンの直前に，変数名と `@` を付けると，そのアームの中で変数名が使えるようになります．
```rust
fn main() {
    let tuple = (1, 50);
    match tuple {
        (1, value @ 0..=9) => println!("{} は 1 桁", value),
        (1, value @ 10..=99) => println!("{} は 2 桁", value),
        (1, value @ 100..=std::i32::MAX) => println!("{} は 3 桁以上", value),
        (1, _) => println!("負"),
        _ => {}
    }
}
```
`0..=9` の代わりに `value @ 0..=9` と書くと， `tuple` の 2 つめの要素がパターン `0..=9` にマッチしたとき，その値が `value` として使えるようになります．

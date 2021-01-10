---
title: "可変変数"
---

# 配列の総和
配列の値を全て足し合わせたものを総和といいます．たとえば `[30, 20, 30]` という配列の総和は $30 + 20 + 30 = 80$ になります．この総和を， `for` 式を使って計算することを考えます．これを実現するには，**可変変数**というものが必要になります．

# 可変変数
今までの変数には，値を一度しか代入することができませんでした．しかし，可変変数には値を複数回代入することができます．

変数の宣言において，変数名の直前にキーワード `mut` を付けると，可変変数になります．
```rust
let mut mutable: i32;
```
可変変数に対する代入は何度でも行えます．
```rust
fn main() {
    let mut mutable: i32;
    mutable = 30;
    assert_eq!(mutable, 30);
    mutable = 20;
    assert_eq!(mutable, 20);
}
```
しかし，型を変更することはできません．
```rust
fn main() {
    let mut mutable: i32;
    mutable = 30;
    mutable = 20.0f64; // エラー
}
```
`i32` 型の可変変数に `f64` 型の値を代入しようとしたのでエラーになっています．

また，アドレスも変化しません．
```rust
fn main() {
    let mut mutable: i32;
    mutable = 10;
    println!("{:p}", &mutable);
    mutable = 20;
    println!("{:p}", &mutable);
}
```
`mutable = 20;` の前と後で同じアドレスが出力されます．

## パターン
パターンの中でも，変数名の直前に `mut` を置くと可変変数になります．
```rust
let (mut hoge, fuga) = (10, 20);
```
`hoge` は可変変数ですが， `fuga` は `mut` が付いていないので可変変数ではありません．
## 型推論
可変変数についても，代入や使用をもとに型推論が行われます．
```rust
fn main() {
    let mut mutable;
    mutable = 30;
    assert_eq!(mutable, 30);
    mutable = 20;
    assert_eq!(mutable, 20_i64);
}
```
最後に `20_i64` と比較しているので，変数 `mutable` と整数リテラル `30` ， `20` は全て `i64` 型に推論されます．

## 未初期化
可変変数も，未初期化の可能性があるまま使用することはできません．
```rust
use proconio::input;

fn main() {
    input! {
        input: i32,
    }
    let mut normalized;
    if input >= 0 {
        normalized = 1;
    }
    println!("{}", normalized); // エラー
}
```
`input` が 0 以上なら `normalized` に 1 が代入されますが，そうでないとき `normalized` は未初期化のままなので，中身を出力しようとするとエラーになります．必ず少なくとも一度は代入が行われていなければなりません．
```rust
use proconio::input;

fn main() {
    input! {
        input: i32,
    }
    let mut normalized = -1; // ここで必ず値が代入される
    if input >= 0 {
        normalized = 1;
    }
    println!("{}", normalized);
}
```
ただ，これならそもそも可変変数を使う必要がありません．
```rust
use proconio::input;

fn main() {
    input! {
        input: i32,
    }
    let normalized = if input >= 0 { 1 } else { -1 };
    println!("{}", normalized);
}
```
わざわざ可変変数を使う必要が本当にあるかどうか吟味するのは重要です．

## 評価順序
可変変数に値を代入するとき，代入式の右辺でその変数自体を使うことができます．
```rust
fn main() {
    let mut mutable = 10;
    assert_eq!(mutable, 10);
    mutable = mutable + 20;
    assert_eq!(mutable, 30);
}
```
`mutable = mutable + 20` という式では，まず `mutable + 20` の部分が計算されて 30 となり，その結果が `mutable` に代入されます．

実は， `+=` 演算子というものを使うと同じことが次のように書けます．
```rust
fn main() {
    let mut mutable = 10;
    assert_eq!(mutable, 10);
    mutable += 20;
    assert_eq!(mutable, 30);
}
```
`mutable += 20;` と書くと，可変変数 `mutable` の値に 20 が加算されます．同様に， `-=` ， `*=` ， `/=` ， `%=` 演算子があります．
```rust
fn main() {
    let mut mutable = 100;
    assert_eq!(mutable, 100);
    mutable -= 10;
    assert_eq!(mutable, 90);
    mutable *= 2;
    assert_eq!(mutable, 180);
    mutable /= 3;
    assert_eq!(mutable, 60);
    mutable %= 7;
    assert_eq!(mutable, 4);
}
```

# for 式と可変変数
最初の問題に戻って，配列 `[30, 20, 30]` の総和を `for` 式を使って計算してみましょう．
```rust
fn main() {
    let array = [30, 20, 30];
    let mut sum = 0;
    for num in &array {
        sum += num;
    }
    assert_eq!(sum, 80);
}
```
`sum` の型は `i32` ， `num` の型は `&i32` です． `sum += num;` の部分では `+=` 演算子が `num` を参照外ししています．

`sum` に初め `0` が代入され，その後ループごとに `num` の値が加算されるので，最終的に `sum` の値は 80 になります．こうして配列の総和を計算することができました．

:::message
実は， `sum()` 関数というものを使っても総和を計算することができます．
```rust
fn main() {
    let array = [30, 20, 30];
    let sum: i32 = array.iter().sum();
    assert_eq!(sum, 80);
}
```
後々説明します．
:::

# `proconio::input!` マクロと可変性
`proconio::input!` マクロの中でも，変数名の前に `mut` を付けることで可変にすることができます．
```rust
use proconio::input;

fn main() {
    input! {
        mut a: i32,
    }
    a += 20;
    println!("{}", a);
}
```

---
title: "パターンマッチ"
---
# 複雑なパターンマッチ
今までの内容を使って，次のコードを読んでみましょう．
```rust
fn main() {
    let elements: [(i32, f64); 5] = [(6, 12.0), (7, 14.0), (8, 16.0), (15, 31.0), (16, 32.1)];
    for &(number, weight) in &elements {
        println!("{}: {:.1}", number, weight);
    }
}
```
`elements` の型は `[(i32, f64); 5]` （タプル `(i32, f64)` が 5 個並んだ配列）です．よって， `elements[0]` `elements[1]` …… `elements[4]` の型は全て `(i32, f64)` です．

`for` 文では， `elements` の各要素への参照を，パターン `&(number, weight)` で受け取っています．このうち 1 回目のループに着目すると，
```rust
{
    let &(number, weight) = &elements[0];
    println!("{}: {:.1}", number, weight);
}
```
となります．これは，タプルと参照が組み合わさったパターンです．

![](https://storage.googleapis.com/zenn-user-upload/8vgf3wta6i35lup25s4gh70eyi6h)

`number` に 6 ， `weight` に 12.0 がそれぞれ代入され，出力は `6: 12.0` となります．

これが `elements` の各要素について繰り返されるので，プログラム全体の出力は
```-:標準出力
6: 12.0
7: 14.0
8: 16.0
15: 31.0
16: 32.1
```
となります．
# `ref` パターン
ここで，新しいパターンを紹介します．それは **`ref` パターン**というものです．
```rust
fn main() {
    let hoge = 10;
    let ref reference = hoge;
    assert_eq!(*reference, 10);
}
```
`ref reference` の部分が `ref` パターンです．こう書くと，変数 `reference` には `hoge` の値の代わりに *`hoge` への参照が*代入されます．よって `reference` の型は `&i32` ，値は `&hoge` です．

![](https://storage.googleapis.com/zenn-user-upload/w1elkphpqshc85mexlx0e48tqsnp)

つまり，今回の場合は
```rust
fn main() {
    let hoge = 10;
    let reference = &hoge;
    assert_eq!(*reference, 10);
}
```

と書いているのと変わりません．

では，こんな場合はどうでしょう．
```rust
fn main() {
    let carbon = (6, 12.0);
    let (ref number, ref weight) = carbon;
    assert_eq!(*number, 6);
    assert_eq!(*weight, 12.0);
}
```
`number` と `weight` にそれぞれ `ref` が付いています．すると， `number` には `carbon.0` への参照， `weight` には `carbon.1` への参照が代入されます．

![](https://storage.googleapis.com/zenn-user-upload/vsxeukdya2a5gdeltrnye3kolc4p)

`number` の型は `&i32` ，値は `&carbon.0` です． `weight` の型は `&f64` ，値は `&carbon.1` です．

これを使って先ほどの `for` 式を書き換えてみます．
```rust
fn main() {
    let elements: [(i32, f64); 5] = [(6, 12.0), (7, 14.0), (8, 16.0), (15, 31.0), (16, 32.1)];
    for &(ref number, ref weight) in &elements {
        println!("{}: {:.1}", number, weight);
    }
}
```
今度は， `number` と `weight` の型がそれぞれ `&i32` と `&f64` になりました． `println!` マクロはこの参照を外して使うので，最終的には同じく
```-:標準出力
6: 12.0
7: 14.0
8: 16.0
15: 31.0
16: 32.1
```
と出力されます．

ここで使ったパターンは以下のようなものです．
```rust
&(ref number, ref weight)
```
実は，このように「各要素に `ref` が付いたタプルへの参照」の形式をしたパターンにおいては，*`&` を省略することができます*．
```rust
(ref number, ref weight)
```
さらに，このようにして先頭の `&` が省略されている状況下では，*各要素に付いている `ref` も省略することができます*．
```rust
(number, weight)
```
よって，全体では次のように書くことができます．
```rust
fn main() {
    let elements: [(i32, f64); 5] = [(6, 12.0), (7, 14.0), (8, 16.0), (15, 31.0), (16, 32.1)];
    for (number, weight) in &elements {
        println!("{}: {:.1}", number, weight);
    }
}
```
:::message
このような `&` と `ref` の省略は，しばしば初心者にとって混乱をきたします．分からないと思ったら，省略せず全て書きましょう．
:::

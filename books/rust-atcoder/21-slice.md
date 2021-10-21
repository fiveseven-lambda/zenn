---
title: "スライス"
---
# スライス
## スライス
配列とベクタは，別のものです．しかし，これらは次の点で共通しています．

- 同じ型の値が連続して並んでいる．
- 要素の個数を知ることができる．
- 後に `[i]` を付けると `i` 番目の要素にアクセスできる．
-  `for` 式で走査できる．

このような共通点をもつ 2 つを，まとめて扱うための型があります．**スライス**です．

要素の型を `T` とすると，スライスの型は `[T]` と表されます．配列 `[T; N]` の `; N` を取り去った形です．

しかし， *`[T]` 型の変数を宣言することはできません*．
```rust:コンパイルエラー
let slice: [i32];
```

一方， `&[T]` 型や `&mut [T]` 型の変数であれば宣言できます．
```rust
let ref_slice: &[i32];
```

スライスは，常にこうして参照などの形で使用しなければいけない，特別な型です．
## 型強制
配列への参照 `&[T; N]` とベクタへの参照 `&Vec<T>` は，どちらも型強制によって `&[T]` 型にすることができます．
```rust
fn main() {
    let mut ref_slice: &[i32];

    let array = [1, 2, 3];
    ref_slice = &array;
    println!("{:?}", ref_slice);

    let vector = vec![4, 5, 6];
    ref_slice = &vector;
    println!("{:?}", ref_slice);
}
```
配列への参照も，ベクタへの参照も，ともに同じ「スライスへの参照」として扱えています．

スライスへの参照についても，ライフタイムがもとの変数のスコープをはみ出してはなりません．
```rust:コンパイルエラー
fn main() {
    let ref_slice: &[i32] = {
        let array = [1, 2, 3];
        &array
    }; // エラー： ref_slice への代入も，使用に含まれる
}
```
# `..` 演算子
配列やベクタの要素は，メモリ上で連続して並んでいます．たとえば，次の `array` を構成する 6 個の値は連続して置かれています．
```rust
fn main() {
    let array = [0, 10, 20, 30, 40, 50];
}
```
ここで，`array[1]` `array[2]` `array[3]` の 3 要素のみに着目してみます．もとの配列の要素がメモリ上で連続しているので，この 3 要素だけ取り出しても，やはりメモリ上で連続しています．すると，この部分を新たにスライスとみなすことができると分かります．これを**部分スライス**といいます．

今まで，スライスに `[i]` を付けることで要素にアクセスしていました．一方，部分スライスを得るには `[a..b]` を付けます．
```rust
fn main() {
    let array = [0, 10, 20, 30, 40, 50];
    let ref_slice = &array[1..4];
    assert_eq!(ref_slice[0], 10);
    assert_eq!(ref_slice[1], 20);
    assert_eq!(ref_slice[2], 30);
}
```
`for` 式のときと同様に， `1..4` は「1 以上 4 未満」の範囲を表します． `array[1..4]` と書くと，`array` の中の `array[1]` から `array[3]` までの部分スライスが得られます．これはもとの配列 `array` の一部に着目して新しいスライスとみなしただけであり，3 つの要素を格納する領域が新しくメモリ上に確保されたわけではありません．

こうして作られた部分スライスの `[0]` `[1]` `[2]` 番目は，それぞれもとの配列 `array` の `[1]` `[2]` `[3]` 番目となります．

部分スライスの型は `[T; N]` や `Vec<T>` ではなく必ず `[T]` になります．よって部分スライスそのものを変数に代入することはできず，参照 `ref_slice` としてもつことになります．ここで型注釈を省略していますが，`array[1..4]` の型が `[i32]` なので，`ref_slice` の型は `&[i32]` になります．

`a..b` を `a..=b` に変えると「a 以上 b 以下」になります．
```rust
fn main() {
    let array = [0, 10, 20, 30, 40, 50];
    let ref_slice = &array[1..=4];
    assert_eq!(ref_slice, [10, 20, 30, 40]);
}
```

`..` の後ろを省略して `a..` とすると， `array[a]` から最後までの範囲になります．
```rust
fn main() {
    let array = [0, 10, 20, 30, 40, 50];
    let ref_slice = &array[1..];
    assert_eq!(ref_slice, [10, 20, 30, 40, 50]);
}
```

また， `..` `..=` の前を省略して `..b` `..=b` と書くこともできます．これは，最初からの範囲になります．
```rust
fn main() {
    let array = [0, 10, 20, 30, 40, 50];
    let ref_slice = &array[..4];
    assert_eq!(ref_slice, [0, 10, 20, 30]);
}
```

前も後も省略すると，全体の範囲になります．
```rust
fn main() {
    let array = [0, 10, 20, 30, 40, 50];
    let ref_slice = &array[..];
    assert_eq!(ref_slice, [0, 10, 20, 30, 40, 50]);
}
```
`&array` との違いは，型注釈をしなくても `ref_slice` がスライスへの参照型になっていることです．
## 空のスライス
`a..b` において $a = b$ のとき，あるいは `a..=b` において $a = b + 1$ のとき，スライスは空になります．
```rust
fn main() {
    let empty = &[1, 2, 3, 4, 5][2..2];
    println!("{:?}", empty);
}
```
```-:標準出力
[]
```
空のスライスは，何番目の要素にアクセスしようとしても範囲外アクセスとなりますが，スライスであることに変わりはありません．

:::message
`a..b` において $a > b$ のとき，あるいは `a..=b` において $a > b + 1$ のときは，実行時エラーになります．これは [`for` 式のとき](https://zenn.dev/toga/books/rust-atcoder/viewer/20-loop#..-%E6%BC%94%E7%AE%97%E5%AD%90)の挙動と異なるので注意してください．
:::

# 使用
スライスの使い方をいくつか紹介します．完全な情報は[ドキュメント](https://doc.rust-lang.org/std/primitive.slice.html)に載っています．
## `for` 式
スライスは， `for` 式で走査することができます．
```rust
fn main() {
    let array = [1, 2, 3];
    for i in &array[..] {
        println!("{}", *i);
    }
}
```
## `swap` 関数
`x` が可変なスライスのとき， `x.swap(a, b)` と書くと `x[a]` と `x[b]` の中身が入れ替わります．
```rust
fn main() {
    let mut array = [0, 10, 20, 30, 40];
    let ref_mut_slice = &mut array[..];
    ref_mut_slice.swap(1, 3);
    assert_eq!(array, [0, 30, 20, 10, 40]);
}
```

実は， `x.swap(a, b)` と書いたとき `x` が配列やベクタであればスライスへと型強制されるので，
```rust
fn main() {
    let mut array = [0, 10, 20, 30, 40];
    array.swap(1, 3);
    assert_eq!(array, [0, 30, 20, 10, 40]);
}
```
と書けます．以下で紹介する他の関数についても同じです．

## `reverse` 関数
`x.reverse()` は，可変なスライス `x` の中身の順番をひっくり返します．
```rust
fn main() {
    let mut array = [7, 2, -3, 9, -2, 5];
    array.reverse();
    assert_eq!(array, [5, -2, 9, -3, 2, 7]);
}
```

これを一部分に適用すれば
```rust
fn main() {
    let mut array = [7, 2, -3, 9, -2, 5];
    array[1..4].reverse();
    assert_eq!(array, [7, 9, -3, 2, -2, 5]);
}
```
となります．

## `sort` 関数
`x.sort()` は，可変なスライスの中身を小さい順に並べ替えます．この操作を**ソート**といいます．
```rust
fn main() {
    let mut array = [7, 2, 5, -3, 9, -2, 5];
    array.sort();
    assert_eq!(array, [-3, -2, 2, 5, 5, 7, 9]);
}
```
# 練習問題
- [ABC171 B - Mix Juice](https://atcoder.jp/contests/abc171/tasks/abc171_b) / [解答例](https://atcoder.jp/contests/abc171/submissions/19519770)
- [ABC175 B - Making Triangle](https://atcoder.jp/contests/abc175/tasks/abc175_b) / [解答例](https://atcoder.jp/contests/abc175/submissions/19443184)

---
title: "ベクタ"
---
# ベクタ
配列は，型レベルで大きさが決まっていました．たとえば `[i32; 5]` と `[i32; 6]` は異なる型であり，たとえ可変だろうと要素の個数を変化させることはできませんでした．
```rust
fn main() {
    let mut array = [1, 2, 3, 4, 5];
    array[0] = 10; // 要素を書き換えることは可能

    // 一方，要素の個数を変えることは不可能
}
```

この章で紹介する**ベクタ**は，配列と似ていますが，要素の個数を変えることができるという点が異なります．

各要素の型が `T` であるようなベクタの型は， `Vec<T>` です．たとえば各要素の型が `i32` なら `Vec<i32>` になります．
```rust
let vector: Vec<i32>;
```

# 初期化

配列は， `[1, 2, 3]` のように各要素をカンマ `,` で区切って書くやり方と， `[1; 10]` のように値と要素数をセミコロン `;` で区切って書くやり方の 2 つがありました．

同じことをベクタで行うには， **`vec!` マクロ**を使って
```rust
vec![1, 2, 3]
```
や
```rust
vec![1; 10]
```
と書きます． `vec!` マクロの呼び出しにおいて角括弧 `[ ]` を使うのは，配列と見た目を似せるためです．

また，
```rust
Vec::<i32>::new()
```
のように書くと，空（要素数が 0 ）のベクタになります． `Vec` と `<i32>` ，そして `<i32>` と `new()` の間に両方 `::` を入れなければいけないことに注意してください．

# 取得
## 要素数の取得
ベクタ `vector` に対し， `vector.len()` は要素の個数となります．
```rust
fn main() {
    let vector: Vec<i32> = vec![1, 2, 3];
    assert_eq!(vector.len(), 3_usize);
}
```
`vector.len()` の型は `usize` です．
## 要素の取得
配列と同じように， `vector[0]` で最初の要素， `vector[1]` で次の要素，……が得られます．
```rust
fn main() {
    let vector: Vec<i32> = vec![1, 2, 3];
    assert_eq!(vector[0], 1_i32);
    assert_eq!(vector[1], 2_i32);
    assert_eq!(vector[2], 3_i32);
}
```
ベクタが可変であれば，要素に値を代入することもできます．
```rust
fn main() {
    let mut vector: Vec<i32> = vec![1, 2, 3]; // mut を付ける
    vector[1] = 10; // 代入
    assert_eq!(vector, vec![1, 10, 3]);
}
```
# 型推論
ベクタについても型推論が働きます．
```rust
fn main() {
    let vector = vec![1, 2, 3];
    assert_eq!(vector[2], 3_i64);
}
```
要素 `vector[2]` が `3_i64` と比較されているので，各要素の型は `i64` でなくてはなりません．よって `vector` 自体の型は `Vec<i64>` と推論されます．

また，要素の型が他の箇所から推論できる場合は， `Vec::<T>::new()` を単に `Vec::new()` と書くことができます．
```rust
fn main() {
    let vector: Vec<u32>;
    vector = Vec::new();
}
```
`vector` が `Vec<u32>` 型なので，そこに代入する `Vec::new()` も `Vec<u32>` 型でなければなりません．よって， `Vec::new()` は自動的に `Vec::<u32>::new()` となります．
# 値の追加
ベクタを可変として宣言しておくと， **`push` 関数**を使って末尾に要素を追加することができます．
```rust
fn main() {
    let mut vector = Vec::new(); // mut を付ける
    assert_eq!(vector.len(), 0);

    vector.push(10);
    vector.push(20);
    vector.push(30);
    assert_eq!(vector, vec![10, 20, 30]);

    vector.push(40_i64);
    vector.push(50);
    assert_eq!(vector, vec![10, 20, 30, 40, 50]);
}
```
`vector` は初め空ですが， `push` 関数で 10, 20, 30 を追加すると `vec![10, 20, 30]` と等しくなり，さらに 40, 50 を追加すると `vec![10, 20, 30, 40, 50]` と等しくなります．また，途中で `40_i64` を追加しているので `vector` の型は `Vec<i64>` であると推論され，他の整数リテラル `10` `20` `30` `40` `50` も全て `i64` 型になります．

他にも，末尾の要素を削除することなどが可能ですが，そのためにはもう少し説明が必要なので後に回します．

# `proconio::input!` マクロ
`proconio::input!` マクロでベクタを読み込むことができます．
```rust
input! {
    vector: [i32; 10usize],
}
```
`T` 型の値を `n` 個読みこんでベクタにするには， `[T; n]` と書きます．書き方の形式は配列ですが， `vector` はベクタになります．

要素数の部分に書けるのは `usize` 型の式です．
```rust
input! {
    n: usize,
    vector: [i32; n],
}
```
まず一つの `usize` 型整数 `n` を読み込んで，その後 `n` 個の `i32` 型整数を読み込んでいます．よって，たとえば入力が
```-:標準入力
5
1 6 8 2 3
```
なら， `n` の値は `5` となって， `vector` の値は `vec![1, 6, 8, 2, 3]` となります．

# for 式
`for` 式で， `in` の後に書けるものは配列への参照以外にもあると言いました．ベクタへの参照もその一つです．
```rust
fn main() {
    let vector = vec![30, 20, 30];
    let mut sum = 0;
    for num in &vector {
        sum += num;
    }
    assert_eq!(sum, 80);
}
```
配列のときと同じように， `num` の値が `&vector[0]` ， `&vector[1]` ， `&vector[2]` のときそれぞれについて `sum += num;` が実行されます．

# デバッグ出力
ベクタも `{:?}` によるデバッグ出力が可能です．
```rust
fn main() {
    let vector = vec![1, 2, 3];
    println!("{:?}", vector);
}
```
```-:標準出力
[1, 2, 3]
```

# 練習問題
ここまでの内容を使って解ける問題をいくつか挙げるので，解いてみましょう．

また， `for` 式を使って解ける問題の中には，イテレータというものを使うことで `for` 式を使わずに書けるようなものも多く含まれます．イテレータに関しては後の章で説明します．今はコードだけ添えておくので，雰囲気だけでも見てみてください．
## ABC185 A - ABC Preparation
ベクタと `for` 式を使って， [ABC185 A - ABC Preparation](https://atcoder.jp/contests/abc185/tasks/abc185_a) を解いてみましょう．

まず， `proconio::input!` マクロで
```rust
input! {
    a: [i32; 4],
}
```
と書くことで， $a_1, a_2, a_3, a_4$ がそれぞれ `a[0]` `a[1]` `a[2]` `a[3]` として読み込まれます．

答えるべきものは，この 4 つのうち最も小さい値です．これは， `ans` という可変変数を宣言しておき，ベクタ `a` の各要素 `i` について「`i` が `ans` より小さかったら `ans` を `i` で書き換える」ということを繰り返せばよさそうです．

`a` の各要素は `100` 以下なので， `ans` にははじめ `100` 以上の値を代入しておけばよいです．

全体ではたとえば次のようになります．
```rust
use proconio::input;
fn main() {
    input! {
        a: [i32; 4],
    }
    let mut ans = 100;
    for &i in &a {
        if ans > i {
            ans = i;
        }
    }
    println!("{}", ans);
}
```
実は， `a` と `b` が比較可能（ともに `i32` であるような場合やともに `f64` であるような場合）なら， `a.min(b)` と書くと `a` と `b` のうちの小さい方が得られます．これを使うとコードが次のように書き換えられます．
```rust
use proconio::input;
fn main() {
    input! {
        a: [i32; 4],
    }
    let mut ans = 100;
    for &i in &a {
        ans = ans.min(i);
    }
    println!("{}", ans);
}
```
また，まだ説明していないイテレータを使って書くと次のようになります．
```rust
use proconio::input;
fn main() {
    input! {
        a: [i32; 4],
    }
    println!("{}", a.iter().min().unwrap());
}
```
## ABC181 B - Trapezoid Sum
次は [ABC181 B - Trapezoid Sum](https://atcoder.jp/contests/abc181/tasks/abc181_b) を解いてみましょう．

このように， $A_1$ $B_1$ $A_2$ $B_2$ …… $A_N$ $B_N$ の形で入力が与えられるときは，
```rust
input! {
    v: [(i64, i64); n],
}
```
のように，タプルの並び $(A_1, B_1)$ $(A_2, B_2)$ …… $(A_N, B_N)$ として読み込みます．今回は前の `n` と併せて
```rust
input! {
    n: usize,
    v: [(i64, i64); n],
}
```
が入力読み込み部分になります．

ここで問題です． `i32` ではなく `i64` とした理由は何でしょうか？
:::details クリックで理由を表示
この問題の答えが最も大きくなるのはどのようなときか考えてみましょう．$N = 10^5$ ， $(A_1, B_1) = (A_2, B_2) = \cdots = (A_N, B_N) = (1, 10^6)$ のときに答えは $5.000005 \times 10^{16}$ となって，これが最大値です．

一方， `i32` で表せる値は $2^{31} - 1 \approx 2.15\times10^9$ までです．よって `i32` を使うと計算の途中で表現できる範囲を超えてしまいます．これを**オーバーフロー**といいます．

一方， `i64` で表せる値は $2^{63} - 1 \approx 9.22\times10^{18}$ までです．よって `i64` を使えばオーバーフローすることはありません．
:::

残りは自力で書いてみてください．

[解答例 1](https://atcoder.jp/contests/abc181/submissions/19395812) です．

実は，「`usize` 型の整数 $n$ を受け取り，直後に $n$ 個の入力を受け取る」という場合に限っては，次のように書くことも可能です．
```rust
input! {
    v: [(i64, i64)],
}
```
`n: usize` と `v: [T; n]` が，あわせて `v: [T]` と書けます．[解答例 2](https://atcoder.jp/contests/abc181/submissions/19396536) ．

イテレータを使った[解答例 3](https://atcoder.jp/contests/abc181/submissions/19396582) です．

## ABC176 C - Step
次は [ABC176 C - Step](https://atcoder.jp/contests/abc176/tasks/abc176_c) を解いてみましょう．

[解答例 1](https://atcoder.jp/contests/abc176/submissions/19520294) です．

イテレータを使った[解答例 2](https://atcoder.jp/contests/abc176/submissions/19520517) です．

## ABC180 B - Various distances
次は [ABC180 B - Various distances](https://atcoder.jp/contests/abc180/tasks/abc180_b) を解いてみましょう．

`x.abs()` は `x` の絶対値， `x.sqrt()` は `x` の正の平方根です．

[解答例 1](https://atcoder.jp/contests/abc180/submissions/19396952) です．

イテレータを使った[解答例 2](https://atcoder.jp/contests/abc180/submissions/19397106) です．

## ABC174 B - Distance
次は [ABC174 B - Distance](https://atcoder.jp/contests/abc174/tasks/abc174_b) を解いてみましょう．

`x.hypot(y)` は $\sqrt{x^2 + y^2}$ です．

[解答例 1](https://atcoder.jp/contests/abc174/submissions/19397466) です．

イテレータを使った[解答例 2](https://atcoder.jp/contests/abc174/submissions/19397521) です．

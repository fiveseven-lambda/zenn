---
title: "for 式"
---
# 配列の要素への参照
次のコードを実行してみてください．
```rust
fn main() {
    let primes = [2, 3, 5, 7];
    println!("{:p}", &primes[0]);
    println!("{:p}", &primes[1]);
    println!("{:p}", &primes[2]);
    println!("{:p}", &primes[3]);
    println!("{:p}", &primes);
}
```
:::message
`&primes[0]` は， `&primes` の後に `[0]` が付いたものではなく， `primes[0]` の前に `&` が付いたものです．これは `[ ]` 演算子の優先順位が `&` 演算子より高いためです．
:::

配列の各要素 `primes[0]` `primes[1]` `primes[2]` `primes[3]` はメモリ上の*連続した*領域に配置されます．これらの型は全て `i32` なので，一つの要素が 4 バイトを占めています．よって，出力を見るとアドレスは 4 ずつ増えていっているはずです．

また， `&primes` はこの領域全体の先頭のアドレスなので， `&primes[0]` と同じ値になっているでしょう．

![](https://storage.googleapis.com/zenn-user-upload/06mhncj0v2i510yl1170n1jz8x03)

コード上でそのまま書けるわけではありませんが， `&primes` の値が分かっていれば，そのアドレスに 4 を順次加えていくことによって， `&primes[0]` `&primes[1]` ……の値が得られるわけです．
# `for` 式
**`for` 式**というものを使うと，一つのブロックを繰り返し実行することができます．

次のコードを見てください．
```rust
fn main() {
    let primes = [2, 3, 5, 7];
    for p in &primes {
        println!("{}", p);
    }
}
```
まず， `primes` に `[2, 3, 5, 7]` が代入されています．続いて 3 行にわたって書かれているのが `for` 式です．

`for` 式は， 2 つのキーワード `for` と `in` を使って
```rust
for パターン in 式 {
    ブロックの中身
}
```
と書きます．今回の
```rust
for p in &primes {
    println!("{}", p);
}
```
では，パターンとして単なる変数名 `p`，式として `&primes`，ブロックの中身として `println!("{}", p);` が書かれています．これは英語の "for (each) p in primes" （primes の中の各 p について）という表現に即した構文になっています．

`in` の直後に置かれている式は，今回の場合「配列への参照」（ `&[i32; 4]` 型）です．今はまだ登場していませんが，ここに置ける式は「配列への参照」以外にも存在します．

このプログラムを実行すると， `p` の値が `&primes[0]` ， `&primes[1]` ， `&primes[2]` ， `&primes[3]` のときそれぞれについて順にブロックが実行されます．すなわち，
```rust
fn main() {
    let primes = [2, 3, 5, 7];
    {
        let p = &primes[0];
        println!("{}", p);
    }
    {
        let p = &primes[1];
        println!("{}", p);
    }
    {
        let p = &primes[2];
        println!("{}", p);
    }
    {
        let p = &primes[3];
        println!("{}", p);
    }
}
```
を実行したのと同じようになって，プログラム全体の出力は
```
2
3
5
7
```
となります．

上で述べた通り，内部ではアドレス `&primes` に 4 を順次足していくことによって `&primes[0]` `&primes[1]` ……の値が得られています．

`p` のスコープは，繰り返されるブロックの開始から終了までです．
```rust
fn main() {
    let primes = [2, 3, 5, 7];
    for p in &primes { // ブロックの開始： p のスコープの開始
        println!("{}", p);
    } // ブロックの終了： p のスコープの終了
}
```

また， `for` のブロックから値を返すことはできません．
```rust
fn main() {
    let primes: [i32; 4] = [2, 3, 5, 7];
    let ans = for p in &primes {
        *p
    };
}
```
次のようなエラーになります．
```
error[E0308]: mismatched types
 --> src/main.rs:4:9
  |
4 |         *p
  |         ^^ expected `()`, found `i32`
```
「値を返すことはできない」は，「`()` を返さなければならない」と言い換えられます．よって，エラーメッセージの文面は `` expected `()`, found `i32` `` （「`()` が来るはずだったが `i32` が来た」）となっています．

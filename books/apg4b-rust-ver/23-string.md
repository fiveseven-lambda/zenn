---
title: "文字列"
---

文字列も，変数に代入できます．
```rust
let string = "Hello, world!";
```
この章では，文字列の扱い方について説明します．

# 文字列の表現

`u8` は， 8 ビット（ 1 バイト）の符号なし整数型です．コンピュータは 1 バイトをデータの基本単位とするため，様々なデータが `u8` の配列として表現されます．文字列もその一つです．

:::message
このページでは， `u8` 型の値を 16 進数で表記します． 0 〜 9 までの数字に a 〜 f までのアルファベットを加えた計 16 種類の文字を使い， 1 バイト分を 2 桁で表します．
:::

たとえば， `Hello` という文字列は， 5 バイトのデータ [48, 65, 6c, 6c, 6f] で表されます．次の表のように， 1 文字目の `H` は 1 バイト目の 48， 2 文字目の `e` は 2 バイト目の 65 ……のように対応しています．
| 文字 | 対応する部分 | バイト数 |
| -- | -- | -- |
| `H` | 48 | 1 |
| `e` | 65 | 1 |
| `l` | 6c | 1 |
| `l` | 6c | 1 |
| `o` | 6f | 1 |

一方， `𠮷野家で𩸽` という文字列は， 17 バイトのデータ [f0, a0, ae, b7, e9, 87, 8e, e5, ae, b6, e3, 81, a7, f0, a9, b8, bd] で表されます． `Hello` の場合と異なり， 1 文字が 1 バイトに対応していません．
| 文字 | 対応する部分 | バイト数 |
| -- | -- | -- |
| `𠮷` | f0, a0, ae, b7 | 4 | 
| `野` | e9, 87, 8e | 3 |
| `家` | e5, ae, b6 | 3 |
| `で` | e3, 81, a7 | 3 |
| `𩸽` | f0, a9, b8, bd | 4 |

`Hello` の 3 文字目は `l` ， `𠮷野家で𩸽` の 3 文字目は `家` です．しかし， `Hello` を表すデータの中で 3 文字目に対応するのは 3 バイト目なのに対し， `𠮷野家で𩸽` を表すデータの中で 3 文字目に対応するのは 8 バイト目から 10 バイト目までの 3 バイトです．文字列の 3 文字目がデータのどの部分と対応しているのか知るためには，データを最初から順番に見ていくしかありません．このことにより，次の問題が生じます．

- 配列やベクタは `[i]` を付けて i + 1 番目の要素にアクセスできる．一方，文字列はインデックス i を指定してアクセスすることができない．

配列やベクタは，要素 1 つあたりのバイト数が決まっています．たとえば `[i32; 10]` 型の配列であれば，要素の型が `i32` なので 1 要素あたり 4 バイトを占めています．これにより，先頭のアドレスが分かっていれば，そのアドレスに $i \times 4$ を加算することで， i 個先の要素のアドレスが計算できます．

一方，文字列についてはこのような計算ができないため， i 文字目をすぐに知ることができないわけです．

ただし， 1 文字分の占める長さは最大で 4 バイトです．

:::message
「文字」と「文字列」は別物です．「文字」は文字列を構成する 1 文字 1 文字を指すのに対し，「文字列」は文字が並んだ `u8` 列を指します．
:::

# 文字列を扱う型
## `String`
文字列処理において，ベクタ `Vec<T>` に相当する型は `String` です．

```rust
let s: String;
```

ベクタと同じように， `new()` 関数を使って空の文字列を作ることができます．
```rust
let s = String::new();
```
## `str`
文字列処理において，スライス `[T]` に相当する型は `str` です．

スライスと同じく， `str` 型の変数を宣言することはできず，常に `&str` や `&mut str` の形で使います．
```rust
let s = String::new();
let slice: &str = &s; // &String から &str への型強制
```

文字列リテラルそのものの型も， `&str` です．

```rust
let slice = "Hello"; // slice は &str 型
```

文字列リテラルは，他のリテラルへの参照と同様に静的なライフライムをもちます．

## `&str` から `String` への変換
`s` の型が `&str` であるとき， `s.to_string()` あるいは `String::from(s)` と書くと， `s` をもとに `String` 型の値を作ることができます．
```rust
let string = "Hello".to_string();
```
```rust
let string = String::from("Hello");
```
これら 2 つの書き方は同じであり，どちらも変数 `string` は `String` 型となります．

# 出力
`String` 型の文字列や `&str` 型の文字列スライスは， `println!` マクロで `{}` を用いて出力できます．
```rust
fn main() {
    let greeting = "Hello";
    let world = "world".to_string();
    println!("{}, {}!", greeting, world);
}
```
```-:標準出力
Hello, world!
```
一方，次のように書くことはできません．
```rust
fn main() {
    let s = "Hello";
    println!(s); // エラー
}
```
`println!` マクロの中では，フォーマット文字列として必ず文字列*リテラル*を使う必要があります．

# `for` 式
## `chars` 関数
文字列の i 文字目を直接得ることはできませんが，データを先頭から順番に見ていけば文字の区切りが分かります．よって， `for` 式を用いて文字列を 1 文字ずつ見ることができます．

文字列スライス `s` を 1 文字ずつ走査するには， `for` 式で `in` の直後に `s.chars()` を置きます．
```rust
fn main() {
    let s = "Hello";
    for c in s.chars() {
        // 処理
    }
}
```
`s.chars()` は， `s` の型が `String` のときも使えます．
```rust
fn main() {
    let s = "Hello".to_string();
    for c in s.chars() {
        // 処理
    }
}
```

`c` の型は， **`char`** になります． `char` は文字を表す 4 バイトの型です．

:::message
文字列の中で 1 文字が占めるバイト数は最大で 4 バイトなので， 4 バイトの `char` で表すことができます．
:::

`char` 型のリテラルは， 1 つの文字をダブルクォーテーションマーク `"` の代わりにシングルクォーテーションマーク `'` で囲って書きます．
```rust
fn main() {
    let s = "打打打打打打打打打打";
    for c in s.chars() {
        assert_eq!(c, '打');
    }
}
```

`char` 型の値も， `println!` マクロで `{}` を使って出力できます．

## `bytes` 関数
初めに述べた通り，文字列の中身は `u8` 型の値が並んだものです． `s.chars()` の代わりに `s.bytes()` を使うと， `u8` 型の値をそのまま一つずつ見ることができます．
```rust
fn main() {
    let s = "𠮷野家で𩸽";
    for c in s.bytes() {
        println!("{:x}", c);
    }
}
```
文字列 `𠮷野家で𩸽` を表す 17 バイトが， 1 バイトずつ出力されます．

# 文字列以外から `String` への変換
## `to_string` 関数
`x` が `i32`， `f64` のような整数型や `char` のとき， `x.to_string()` はその値を `String` 値に変換したものになります．

```rust
fn main() {
    let x: i32 = 10;
    assert_eq!(x.to_string(), "10".to_string());

    let x: f64 = 120.0;
    assert_eq!(x.to_string(), "120".to_string());

    let x: char = 'A';
    assert_eq!(x.to_string(), "A".to_string());
}
```
3 つとも， `x.to_string()` の型は `String` です．

## `format!` マクロ
`println!` マクロを使うと，フォーマット文字列に従って値をフォーマットした結果が出力されます．
```rust
fn main() {
    println!("{} {}", 10, 2.5);
    // 2 つの値 10, 2.5 を，
    // "{} {}" に従ってフォーマット
}
```

一方， `format!` マクロを使うと，フォーマットの結果を `String` 型の値として得ることができます．
```rust
fn main() {
    let s = format!("{} {}", 10, 2.5);
    // 10 2.5 と出力される代わりに，
    // s の中身が "10 2.5" になる
    assert_eq!(s, "10 2.5".to_string());
}
```

# バイトリテラル，バイト文字列リテラル
アルファベットや数字などの一部の文字は ASCII 文字と呼ばれ，文字列の中で占める長さが 1 バイトです．

## バイトリテラル
ASCII 文字を `' '` の代わりに `b' '` で囲むと，対応する `u8` 型の値になります．
```rust
fn main() {
    let c = b'A';
    println!("{:x}", c);
}
```
`c` の型は `u8` です． ASCII 文字 `A` に対応する値は 16 進数で 41 なので， `41` と出力されます．

## バイト文字列リテラル
ASCII 文字だけで構成される文字列を `" "` の代わりに `b" "` で囲むと， `u8` 型の配列への参照になります．
```rust
fn main() {
    let array = b"Hello";
    assert_eq!(*array, [b'H', b'e', b'l', b'l', b'o']);
}
```
`array` の型は `&[u8; 5]` です．

# `proconio::input!` マクロ
`proconio::input!` マクロを使って文字列や文字を読み込む方法は， 4 種類あります．
## `String`
型として `String` を指定すると，文字列が*ホワイトスペース区切りで*読み込まれます．ホワイトスペースとは，空白，改行，タブなどの見えない文字のことです．
```rust
use proconio::input;

fn main() {
    input! {
        s1: String,
        s2: String,
    }
    println!(r#""{}""#, s1);
    println!(r#""{}""#, s2);
}
```
たとえば，入力が
```-:標準入力
Hello, world!
```
であれば，最初の空白までの `Hello,` が `s1` ，次の改行までの `world!` が `s2` に代入されて，出力は
```-:標準出力
"Hello,"
"world!"
```
となります．ホワイトスペースは全て読み飛ばされるので，たとえば入力が
```
   	Hello,

     world!
```
だったりしても同じです．
## `proconio::marker::Chars`
`String` の代わりに， `proconio::marker::Chars` と書くと，読み込んだ文字列が `Vec<char>` に変換されてから代入されます．
```rust
use proconio::input;

fn main() {
    input! {
        s: proconio::marker::Chars,
    }
    for c in &s {
        println!("{}", c);
    }
}
```
`s` の型は `Vec<char>` ， `c` の型は `&char` です．

コードの先頭に `use proconio::marker::Chars;` と書いておくと， `proconio::marker::Chars` の代わりに `Chars` と書くだけで済みます．
```rust
use proconio::input;
use proconio::marker::Chars;

fn main() {
    input! {
        s: Chars,
    }
    for c in &s {
        println!("{}", c);
    }
}
```
## `proconio::marker::Bytes`
`proconio::marker::Bytes` と書くと，読み込んだ文字列が `Vec<u8>` に変換されてから代入されます．これも先頭に `use proconio::marker::Bytes;` と書いておくと `Bytes` で済みます．
```rust
use proconio::input;
use proconio::marker::Bytes;

fn main() {
    input! {
        s: Bytes,
    }
    for c in &s {
        println!("{}", c);
    }
}
```
`s` の型は `Vec<u8>` ， `c` の型は `&u8` です．

`use proconio::marker::Bytes;` と `use proconio::marker::Chars;` を両方書きたい場合は，
```rust
use proconio::marker::{Bytes, Chars};
```
とまとめて書くことができます．さらに `use proconio::input;` もまとめるのであれば
```rust
use proconio::{
    input,
    marker::{Bytes, Chars},
};
```
となります．
## `char`
入力が一文字であれば，型 `char` を指定して読み込むこともできます．
```rust
use proconio::input;

fn main() {
    input! {
        c1: char,
        c2: char,
        c3: char,
    }
    println!("{} {} {}", c1, c2, c3);
}
```
ただし，これは入力が
```
a
b
c
```
のように一文字ずつホワイトスペースで区切られているときにのみ使用できます．
```
abc
```
のような状況で `a` `b` `c` を一文字ずつ読み込むことはできません．

# 練習問題
## ABC175 A - Rainy Season
[ABC175 A - Rainy Season](https://atcoder.jp/contests/abc175/tasks/abc175_a) を解いてみましょう．

入力が文字列として与えられるので， `String` として読み込み， `chars` 関数を使って一文字ずつ見ていきます．
```rust
use proconio::input;

fn main() {
    input! {
        s: String,
    }
    for c in s.chars() {
        // 処理
    }
}
```
`R` が連続している回数を調べようとしています．これは，次のようにして実現できます．
1. 可変変数 `count` を用意し，はじめ 0 にしておく．
1. 文字列を一文字ずつ見て，
   - `R` だったら `count` に 1 を加える．
   - `S` だったら `count` を 0 にリセットする．

ここまでをいったん書いてみると次のようになります．
```rust
use proconio::input;

fn main() {
    input! {
        s: String,
    }
    let mut count = 0;
    for c in s.chars() {
        match c {
            'R' => count += 1,
            'S' => count = 0,
            _ => panic!(),
        }
    }
}
```
これに入力として文字列を与えると，一文字ずつ読みながら `count` の値が変化していくはずです．

もし入力が `RRS` なら，
1. 1 文字目は `R` なので `count` に 1 が足されて 1 になる．
1. 2 文字目は `R` なので `count` に 1 が足されて 2 になる．
1. 3 文字目は `S` なので `count` に 0 が代入される．

となるでしょう．答えるのは，この過程で `count` がとった最大の値です．よって，もう一つの可変変数 `ans` を追加して
```rust
use proconio::input;

fn main() {
    input! {
        s: String,
    }
    let mut count = 0;
    let mut ans = 0;
    for c in s.chars() {
        match c {
            'R' => count += 1,
            'S' => count = 0,
            _ => panic!(),
        }
        ans = ans.max(count);
    }
    println!("{}", ans);
}
```
とすれば， AC コードになります．

## ABC166 A - A?C
[ABC166 A - A?C](https://atcoder.jp/contests/abc166/tasks/abc166_a) を解いてみましょう．

実は， `String` 型の文字列と `str` 型のスライスは `==` を用いて比較することができます．よって，
```rust
use proconio::input;

fn main() {
    input! {
        s: String,
    }
    let ans = if s == "ABC" { "ARC" } else { "ABC" };
    println!("{}", ans);
}
```
のように， `String` 型の `s` と `&str` 型の `"ABC"` について `s == "ABC"` と書けます．

## ABC171 A - αlphabet
[ABC171 A - αlphabet](https://atcoder.jp/contests/abc171/tasks/abc171_a) を解いてみましょう．

入力が 1 文字なので， `char` として受け取ることができます．

パターンマッチについての説明の中で， `..=` を用いた範囲パターンを紹介しました．実は，範囲パターンは `char` 型の文字についても使うことができます．たとえば
- パターン `'b'..='e'` は， `'b'` `'c'` `'d'` `'e'`
- パターン `'X'..='Z'` は， `'X'` `'Y'` `'Z'`
- パターン `'0'..='3'` は， `'0'` `'1'` `'2'` `'3'`

にマッチします．よってこの問題は `match` 式で解くことができます．[解答例](https://atcoder.jp/contests/abc171/submissions/19554239)です．

## ABC160 A - Coffee
[ABC160 A - Coffee](https://atcoder.jp/contests/abc160/tasks/abc160_a) を解いてみましょう．

このように，文字列の途中にある文字を簡単に知りたい場合は， `String` ではなく `proconio::marker::Chars` で受け取ってしまうのが良いでしょう．[解答例 1](https://atcoder.jp/contests/abc160/submissions/19554826) です．

また，制約に「$S$ は長さ $6$ の英小文字からなる文字列である」と書いてあります．英小文字は全て ASCII 文字であるため， `proconio::marker::Bytes` で受け取っても問題ありません．[解答例 2](https://atcoder.jp/contests/abc160/submissions/19554838) です．
:::message
競技プログラミングにおいて，入力は全て ASCII 文字であることがほとんどです．
:::

## ABC179 A - Plural Form
[ABC179 A - Plural Form](https://atcoder.jp/contests/abc179/tasks/abc179_a) を解いてみましょう．

`proconio::marker::Chars` と `proconio::marker::Bytes` は入力をそれぞれ `Vec<char>` と `Vec<u8>` に変換するため，ベクタの章で紹介した `len()` を使って文字数を知ることができます． [解答例](https://atcoder.jp/contests/abc179/submissions/19555011)です．

## ABC184 B - Quizzes
[ABC184 B - Quizzes](https://atcoder.jp/contests/abc184/tasks/abc184_b) を解いてみましょう．

最初に与えられる整数 $N$ は，使う必要がありません．このようなときはワイルドカードパターン `_` を使いましょう．[解答例](https://atcoder.jp/contests/abc184/submissions/19925482)です．

## ABC176 B - Multiple of 9
[B - Multiple of 9](https://atcoder.jp/contests/abc176/tasks/abc176_b) を解いてみましょう．

入力が整数と書かれていますが，制約が $0 \leq N < 10^{200000}$ なので，最も大きい値を表せる `u128` 型を使っても受け取ることができません．そこで，代わりに文字列として受け取ります．

数字は全て ASCII 文字なので， `proconio::marker::Bytes` で受け取ることができます．また， `0` から `9` までの数字は 16 進法で 30 から 39 までの値にそれぞれ対応しているため，たとえば `b'5'` から `b'0'` を引いた結果は `5_u8` になります．これを利用すると，[解答例](https://atcoder.jp/contests/abc176/submissions/19555218)のように書くことができます．

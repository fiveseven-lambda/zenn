---
title: "演算子と bool 型"
---

# 演算子
`+` のような演算子も，関数と同じように捉えることができます． `2 + 3` という式は， `+` という演算子が 2 つの引数 `2` と `3` を受け取り，値 `5` を返しています．

実際，標準ライブラリの関数を用いると，四則演算を関数で書き換えることができます．
```rust
fn main() {
    assert_eq!(std::ops::Add::add(2, 3), 5); // 2 + 3 と同じ
    assert_eq!(std::ops::Sub::sub(5, 2), 3); // 5 - 2 と同じ
    assert_eq!(std::ops::Mul::mul(3, 4), 12); // 3 * 4 と同じ
    assert_eq!(std::ops::Div::div(14, 3), 4); // 14 / 3 と同じ
    assert_eq!(std::ops::Rem::rem(14, 3), 2); // 14 % 3 と同じ
}
```

一方， `==` も演算子です．そして，これに対応する標準ライブラリの関数は， [`PartialEq::eq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html#tymethod.eq) 関数です．これを用いると，
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    if x == 5 {
        println!("x is equal to 5.");
    }
}
```
というコードは
```rust
use proconio::input;

fn main() {
    input! {
        x: i32,
    }
    if PartialEq::eq(&x, &5) {
        println!("x is equal to 5.");
    }
}
```
と書き換えることができます．

# `bool` 
上のコードの中で， `PartialEq::eq` 関数は引数として `&i32` 型の値を 2 つ取っています．では， `PartialEq::eq` 関数の返り値の型は何でしょう？

ここで，今まで登場しなかった新しい型が登場します．「真」と「偽」を表す **`bool`** 型です．

`bool` 型の変数がとる値は 2 通りしかありません．「真」を表す `true` と，「偽」を表す `false` です．
```rust
let x: bool = true; // 真
```
```rust
let x: bool = false; // 偽
```

`PartialEq::eq` 関数は，「`x == y` が成り立つ」「成り立たない」という 2 通りを `bool` で表現し，成り立つならば `true`，成り立たなければ `false` を返します．このように，真／偽の 2 通りを表現したいときに `bool` を使います．
## `if` 式
今まで， `if` の後には `x == 5` のような比較式を置いていました．しかし実際には，型が `bool` ならどんな式も置くことができます．

`if` 式のブロックは，条件式が `true` のとき実行され， `false` のとき実行されません．
```rust
if true {
    // 必ず実行される
}
if false {
    // 必ず実行されない
}
```
## 比較演算子
2 つの値を比較する演算子には， `==` `!=` `>` `<` `>=` `<=` の 6 種類があります．これらの演算子の返り値の型は，全て `bool` です．
```rust
fn main() {
    let x = 3 != 5; // 「 3 と 5 は等しくない」は真なので
    assert_eq!(x, true); // x は true になる
    let x = 6 < 4; // 「 6 は 4 より小さい」は偽なので
    assert_eq!(x, false); // x は false になる
}
```
今まで `if x == 5` などと書いていたとき， `x == 5` の結果が `true` か `false` となり，その値に応じてブロックが実行されるかされないか決まっていたわけです．
## `bool` 値を返す関数
整数を受け取って，素数かどうか判定する関数 `is_prime` を考えます．

`is_prime` 関数の返り値の型は `bool` とし，受け取った整数が素数なら `true` ，そうでないなら `false` を返すようにします．

実装は次のようになります．
```rust
fn is_prime(x: i32) -> bool {
    if x < 2 {
        // 0 や 1 は素数ではない
        return false;
    }
    for i in 2.. {
        if i * i > x {
            // √x 以下の約数がないので素数
            return true;
        }
        if x % i == 0 {
            // 約数があるので合成数
            return false;
        }
    }
    // ここには達しない
    unreachable!();
}
```

この関数を使って 100 以下の素数を小さい順に全て出力したければ，次のようになります．
```rust
fn main() {
    for i in 0..=100 {
        if is_prime(i) {
            println!("{}", i);
        }
    }
}
```
# 論理演算子
「かつ」や「または」といった論理演算子を関数として考えると，引数は `bool` 型の値 2 つで，返り値の型は `bool` となります．

## `&` 演算子
「かつ」を表す演算子には `&` と `&&` の 2 種類があります．

`&` は， 2 つの `bool` 値を受け取り，ともに `true` だったときのみ返り値として `true` を返します．少なくともどちらか一方が `false` であれば `&` の返り値は `false` になります．
```rust
fn main() {
    assert_eq!(true & true, true);
    assert_eq!(true & false, false);
    assert_eq!(false & true, false);
    assert_eq!(false & false, false);
}
```
この演算を， AND 演算といいます．

たとえば，次のコードを見てください．
```rust
fn main() {
    let result = fnc1() & fnc2();
    assert_eq!(result, false);
}

fn fnc1() -> bool {
    println!("fnc1: false");
    false
}

fn fnc2() -> bool {
    println!("fnc2: true");
    true
}
```
`fnc1()` は `false` を返し， `fnc2()` は `true` を返しています．片方が `false` なので， `result` は `false` になります．

また， `fnc1` 関数の中で `fnc1: false` と， `fnc2` 関数の中で `fnc2: true` と出力されるので，全体の出力は
```:stdout
fnc1: false
fnc2: true
```
となります．
## `&&` 演算子
一方， `&` の代わりに `&&` を使っても同様のコードを書くことができます．
```rust
fn main() {
    let result = fnc1() && fnc2();
    assert_eq!(result, false);
}

fn fnc1() -> bool {
    println!("fnc1: false");
    false
}

fn fnc2() -> bool {
    println!("fnc2: true");
    true
}
```
今回も `result` は `false` になります．しかし，出力の内容は変化します．
```:stdout
fnc1: false
```

`fnc1: false` は出力されますが， `fnc2: true` は出力されません． `fnc2: true` は `fnc2` 関数の中で必ず出力されるので，今回は `fnc2` 関数が呼び出されていないことが分かります．

`fnc1() && fnc2()` は， `fnc1()` と `fnc2()` が両方`true` だったときにのみ `true` を返します．しかし，今回 `fnc1()` は `false` だったので， `fnc2()` の返り値にかかわらず `fnc1() && fnc2()` は `false` になるということが分かります．このとき， `&&` 演算子は， *`fnc2()` を呼び出すことなく* `false` を返します．これを**短絡評価**といいます．

## `|` 演算子
「または」を表す演算子には `|` と `||` の 2 種類があります．

`|` は， 2 つの `bool` 値を受け取り，少なくともどちらか一方が `true` であれば返り値として `true` を返します．ともに `false` であれば `|` の返り値は `false` になります．
```rust
fn main() {
    assert_eq!(true | true, true);
    assert_eq!(true | false, true);
    assert_eq!(false | true, true);
    assert_eq!(false | false, false);
}
```

この演算を， OR 演算といいます．

たとえば，次のコードを見てください．
```rust
fn main() {
    let result = fnc1() | fnc2();
    assert_eq!(result, true);
}

fn fnc1() -> bool {
    println!("fnc1: true");
    true
}

fn fnc2() -> bool {
    println!("fnc2: false");
    false
}
```
`fnc1()` は `true` を返し， `fnc2()` は `false` を返しています．片方が `true` なので， `result` は `true` になります．

また， `fnc1` 関数の中で `fnc1: true` と， `fnc2` 関数の中で `fnc2: false` と出力されるので，全体の出力は
```:stdout
fnc1: true
fnc2: false
```
となります．
## `||` 演算子
一方， `|` の代わりに `||` を使っても同様のコードを書くことができます．
```rust
fn main() {
    let result = fnc1() || fnc2();
    assert_eq!(result, true);
}

fn fnc1() -> bool {
    println!("fnc1: true");
    true
}

fn fnc2() -> bool {
    println!("fnc2: false");
    false
}
```
`&&` 演算子と同様に `||` 演算子も短絡評価を行います． `fnc1()` が `true` だった時点で， `fnc1() || fnc2()` は `true` ということが確定するので， `fnc2` 関数が呼ばれることなく `result` は `true` となります．
```:stdout
fnc1: true
```
## `^` 演算子
`^` 演算子は， 2 つの `bool` 値を受け取り，どちらか一方のみが `true` のとき `true` を返します．ともに `true` ，あるいはともに `false` であれば `^` の返り値は `false` になります．
```rust
fn main() {
    assert_eq!(true ^ true, false);
    assert_eq!(true ^ false, true);
    assert_eq!(false ^ true, true);
    assert_eq!(false ^ false, false);
}
```
この演算を， XOR 演算といいます． XOR 演算の短絡評価をする演算子はありません．
## `!` 演算子
`!` 演算子は， 1 つの `bool` 値を受け取り， `true` なら `false` ， `false` なら `true` を返します．
```rust
fn main() {
    assert_eq!(!true, false);
    assert_eq!(!false, true);
}
```
この演算を， NOT 演算といいます．
## 複合代入演算子
`+` や `*` に対する `+=` や `*=` と同じように， `&` `|` `^` に対しても `&=` `|=` `^=` があります． `&&=` と `||=` はありません．
```rust
fn main() {
    let mut flag = true;
    flag &= false;
    assert_eq!(flag, false);
    flag |= false;
    assert_eq!(flag, false);
    flag |= true;
    assert_eq!(flag, true);
    flag ^= true;
    assert_eq!(flag, false);
}
```
# 練習問題
[ABC192 B - uNrEaDaBlE sTrInG](https://atcoder.jp/contests/abc192/tasks/abc192_b) を解いてみます．

この問題を解くためには，文字列 $S$ に含まれる各文字 $c$ について「大文字アルファベットか」「小文字アルファベットか」を知る必要があります．範囲パターン `'A'..='Z'` や `'a'..='z'` を使うこともできますが，標準ライブラリにある関数を使うこともできます．

`char` 型あるいは `u8` 型の変数 `c` について， [`c.is_ascii()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii) は `c` が ASCII 文字のとき `true` に，そうでないとき `false` になります．競技プログラミングで与えられる入力は大抵の場合 ASCII 文字です．

`c` が ASCII 文字であると分かっているとき， `c` がどんな文字であるか調べるためには次の関数が使えます．

| 関数名 | 返り値が `true` になる条件 |
|--|--|
| [`c.is_ascii_alphabetic()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_alphabetic) | `c` が英語のアルファベットのとき |
| [`c.is_ascii_uppercase()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_uppercase) | `c` が英語の大文字アルファベットのとき | 
| [`c.is_ascii_lowercase()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_lowercase) | `c` が英語の小文字アルファベットのとき | 
| [`c.is_ascii_digit()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_digit) | `c` が数字のとき |
| [`c.is_ascii_hexdigit()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_hexdigit) | `c` が 16 進数の表記に使われる文字のとき |
| [`c.is_ascii_alphanumeric()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_alphanumeric) | `c` が英語のアルファベットか数字のとき |
| [`c.is_ascii_whitespace()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_whitespace) | `c` がスペース，タブ，改行，改ページのとき | 
| [`c.is_ascii_punctuation()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_punctuation) | `c` が記号のとき | 
| [`c.is_ascii_graphic()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_graphic) | `c` が表示文字のとき |
| [`c.is_ascii_control()`](https://doc.rust-lang.org/std/primitive.char.html#method.is_ascii_control) | `c` が制御文字のとき |

さて，これを使って今回の問題を解いてみましょう．与えられた文字列 $S$ が読みにくい文字列であるためには， $S$ を構成する*全ての*文字 $c$ について，奇数番目なら英小文字，偶数番目なら英大文字という条件が成り立っていなければなりません．

「全ての〜について〜が成り立つ」というような命題を**全称命題**といいます．これをプログラム上で調べる方法は 2 つあります．

まずは，条件が成り立っていないものが 1 つでも見つかったら `return` 式で `main` 関数を終了させるというものです．これを用いると次のように書けます．
```rust
use proconio::input;
use proconio::marker::Chars;

fn main() {
    input! {
        s: Chars,
    }
    for i in 0..s.len() {
        if (i % 2 == 0) ^ s[i].is_ascii_lowercase() {
            // s[i] が条件を満たさないので，
            // S は読みにくい文字列ではない
            println!("No");
            return;
        }
    }
    // 制御がここに達したということは，
    // 全ての文字を調べても一度も return されなかったということなので，
    // S は読みにくい文字列
    println!("Yes");
}
```

もう 1 つの書き方は， `bool` 型の可変変数 `ans` を用意しておき，最初は `true` で初期化しておいて，条件を満たさない文字が見つかったら `false` を代入するというものです．
```rust
use proconio::input;
use proconio::marker::Chars;

fn main() {
    input! {
        s: Chars,
    }
    let mut ans = true;
    for i in 0..s.len() {
        if (i % 2 == 0) ^ s[i].is_ascii_lowercase() {
            ans = false;
        }
    }
    // ここで ans が true ならば，
    // ans に false が一度も代入されなかったということなので，
    // S は読みにくい文字列
    println!("{}", if ans { "Yes" } else { "No" });
}
```
このやり方では， `&=` 演算子を用いて次のように書くこともできます．
```rust
use proconio::input;
use proconio::marker::Chars;

fn main() {
    input! {
        s: Chars,
    }
    let mut ans = true;
    for i in 0..s.len() {
        ans &= !((i % 2 == 0) ^ s[i].is_ascii_lowercase());
    }
    println!("{}", if ans { "Yes" } else { "No" });
}
```
どちらのやり方でも正しいコードを書くことができます．

全称命題に対し，「ある〜が少なくとも 1 つ存在して，〜が成り立つ」という命題を**存在命題**といいます．こちらは，「条件を満たすものが 1 つでも見つかったら `return` 式で `main` 関数を終了させる」「 `bool` 型の可変変数 `ans` をはじめ `false` で初期化しておいて，条件を満たすものが見つかったら `true` を代入する」という 2 通りのいずれかで書くことができます．

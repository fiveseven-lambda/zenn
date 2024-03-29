---
title: "関数"
---

# 関数
今までに見てきたコードでは， `main` 関数の中にプログラムの内容を書くと，それが上から順に実行されるだけでした．しかし，プログラムの規模が大きくなってくると，ただ上から順に処理が進むだけではプログラム全体を把握しにくくなってしまいます．

**関数**を使うと，まとまった処理を周囲から切り離し，そこに名前を付けることができます．こうすることで，コードを見たときにどの部分が何をしているのか分かりやすくなります．

# 引数をとらない関数

次のコードを見てください．
```rust
fn main() {
    let vec = {
        let mut v = Vec::new();
        for i in 0..10 {
            v.push(i);
        }
        v
    };
    assert_eq!(vec, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
}
```

ブロックの中でベクタ `v` に 0 から 9 までの整数が追加された後，ブロックから `v` が返されて `vec` に代入されています．

このように，何らかの処理をブロックの中で行っているとき，このブロックに名前を付け，関数として切り出すことができます．

## 関数の定義

ブロックに名前を付けて関数にするには，次のように書きます．
```rust
fn 関数名() -> 返り値の型 {
    関数の中身
}
```

たとえば，今回の例で関数の名前を `digits` とすると，次のようになります．

```rust
fn digits() -> Vec<i32> {
    let mut v = Vec::new();
    for i in 0..10 {
        v.push(i);
    }
    v
}
```

返り値の型 `Vec<i32>` というのは，ブロックから返される `v` の型のことです．

このように，ブロックに名前を付けて関数を作ることを，「関数を**定義**する」といいます．
:::message
`fn` は， function （関数）の略です．
:::

:::message
関数に対する名前の付け方は，[第 5 章](https://zenn.dev/toga/books/rust-atcoder/viewer/05-variable#変数名)で説明した変数に対する名前の付け方と同じです．
:::
## 関数の呼び出し
こうして `digits` 関数を定義しておくと， `digits()` と書くことでブロックを実行することができます．すなわち，
```rust
let vec = {
    let mut v = Vec::new();
    for i in 0..10 {
        v.push(i);
    }
    v
};
```
と書いていたものが，
```rust
let vec = digits();
```
と書けるようになります．このように，関数名に `()` を付けてブロックを実行することを，「関数を**呼び出す**」といいます．

:::message
関数名の後に `()` を付けるのを忘れないでください． `()` を付けないと呼び出しになりません．
:::

## 全体
コード全体では，次のようになります．
```rust
fn main() {
    fn digits() -> Vec<i32> {
        let mut v = Vec::new();
        for i in 0..10 {
            v.push(i);
        }
        v
    }
    let vec = digits();
    assert_eq!(vec, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
}
```

まず，`fn digits()` で始まる部分で `digits` 関数を定義しています．この段階でブロックはまだ実行されません．次に `let vec = digits();` の行で関数を呼び出し，ここでブロックが実行されます．返り値は変数 `vec` に代入しています．

関数は，何度でも呼び出すことができます．一度 `digits` 関数を定義しておけば，プログラム内で `digits()` と書くたびにブロックが実行され，ベクタが返されます．

`digits` 関数の定義を， `main` 関数の外側に書くこともできます．

```rust
fn main() {
    let vec = digits();
    assert_eq!(vec, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
}

fn digits() -> Vec<i32> {
    let mut v = Vec::new();
    for i in 0..10 {
        v.push(i);
    }
    v
}
```

これで，「 0 から 9 までの整数を要素とするベクタを作る」という処理に `digits` という名前を付け，周囲から切り離すことができました．
:::message
ブロックが値を返すときと同じく， `digits` 関数の `v` から `main` 関数の `vec` へ，値のムーブが起こっています． `v` のスコープは，ムーブが起こった後に終了すると考えられます．
:::
## `()` を返す関数
たとえば，
```rust
{
    println!("Hello");
}
```
というブロックはユニット `()` を返すのでした．このブロックに `greet` という名前を付けて関数にすると，
```rust
fn greet() -> () {
    println!("Hello");
}
```
となります．

実は，関数の返り値が `()` であるとき， `-> ()` の部分を省略することができます．よって， `greet` 関数の定義は単に
```rust
fn greet() {
    println!("Hello");
}
```
と書くことができます．

よって，たとえば
```rust
fn main() {
    greet();
    greet();
    greet();
}

fn greet() {
    println!("Hello");
}
```
を実行すると `Hello` と 3 回出力されます．

:::message
今までずっと書いてきた `main` 関数の定義においても， `-> ()` が省略されています．
:::

逆に，関数が `()` 以外を返すとき， `-> 型` を省略することは*できません*．型推論も起こらないため，必ずブロックが何を返しているか確かめる必要があります．

# 関数のスコープ
変数と同じように，関数にもスコープがあります．ブロックの中で関数を定義したとき，その関数をブロックの外側で使うことはできません．
```rust
fn main() {
    {
        fn greet() {
            println!("Hello");
        }
        greet();
    }
    greet(); // エラー：使えない．
}
```

しかし，関数のスコープの開始は，変数のスコープの開始と少し違います．

変数のスコープは，宣言からブロックの終了まででした．
```rust
fn main() {
    {
        println!("{}", x); // エラー：使えない
        let x = 10; // x のスコープの開始
        println!("{}", x); // 使える
    } // x のスコープの終了
    println!("{}", x); // エラー：使えない
}
```

一方，関数のスコープは，ブロックの開始からブロックの終了までです．
```rust
fn main() {
    { // greet のスコープの開始
        greet(); // 使える
        fn greet() {
            println!("Hello");
        }
        greet(); // 使える
    } // greet のスコープの終了
    greet(); // エラー：使えない
}
```
`main` 関数の外側で定義した関数のスコープは，ソースコードの最初から最後までです．
```rust
// greet のスコープの開始

fn main() {
    greet(); // 使える
}

fn greet() {
    println!("Hello");
}

// greet のスコープの終了
```
## 関数名の重複
同じスコープで同じ名前の関数を定義することはできません．
```rust
fn fnc() {
    println!("first definition");
}

// エラー
fn fnc() {
    println!("second definition");
}
```
# 環境
プログラム中のある時点において，「今存在している変数」を考えます．たとえば，

```rust
fn main() {
    let a = 10;
    let b = 20;
    println!("{}", a);
}
```
というコードの中で， `println!` の行では変数 `a` と `b` が「今存在している変数」です．これを環境といいます．

今まで，スコープの開始と終了を見て変数が使えるか判断していました．しかし，関数を定義するようになるとここに別の話が加わります．それは，環境が関数ごとに独立しているという点です．次のコードを見てください．
```rust
fn main() {
    let a = 10;
    let b = 20;
    fn fnc() {
        let c = 40;
        println!("{}", c);
    }
}
```
`main` 関数と `fnc` 関数は別の関数なので，これらは独立の環境をもちます．どういうことかというと， `fnc` 関数の `println!` の行における環境は変数 `c` だけで，ここに `main` 関数の環境は含まれないということです．

すなわち，この `fnc` 関数の中で変数 `a` `b` を使うことはできません．
```rust:コンパイルエラー
fn main() {
    let a = 10;
    let b = 20;
    fn fnc() {
        let c = 40;

        // エラー：別の関数 main の環境にある変数を使おうとしている
        println!("{}", a);
    }
}
```

スコープの概念はあくまでも同じ関数の中だけの話で，関数が異なるとスコープ以前にそもそも環境が異なるということに注意してください．
# `todo!` マクロ
上の `digit` 関数の例では，

1. まず関数の定義を完成させ，
1. その後に関数を呼び出す部分を書く

という順番でコードを書きました．しかし，
1. まずどんな関数が必要か決め，
1. 次に関数を呼び出す部分を書き，
1. 最後に関数のブロックの中身を書く

という順番で書きたいこともあります．このようなとき，ブロックの中身を書くまでの間，代わりに `todo!();` と書いておきます．

```rust
fn fnc() -> i32 {
    todo!();
}
```

todo は，「これからやる」という意味です．関数の中身を `todo!()` にしておくと，「この関数の中身は今後書く」という意味になります．

関数の中身を書かずに
```rust
fn fnc() -> i32 {
}
```
としたままだと，コンパイルエラーになってしまいます．なぜなら，空のブロック `{}` はユニット `()` を返すため， `-> i32` の部分と合致しないからです． `todo!();` であればこのようなエラーは発生しません．

# 引数をとる関数
上で， 0 以上 10 未満の整数をベクタに追加して返す関数を定義して使いました．これを，「0 以上 10 未満」から「a 以上 b 未満」に変えることを考えます．

関数を定義するとき，括弧 `( )` の中に変数名と型を書くことができます．
```rust
fn iota(a: i32, b: i32) -> Vec<i32> {
    todo!();
}
```
ここでは，
- `a: i32` ……変数名 `a` とその型 `i32`
- `b: i32` ……変数名 `b` とその型 `i32`

をカンマ `,` で区切って書いています．こうすると，関数の中で変数 `a` と `b` が使えるようになります．
```rust
fn iota(a: i32, b: i32) -> Vec<i32> {
    let mut v = Vec::new();
    for i in a..b {
        v.push(i);
    }
    v
}
```

この関数を呼び出すときは，括弧の中に
```rust
iota(3, 6)
```
と書きます．すると， 1 つ目の `3` が `a` に， 2 つ目の `6` が `b` に代入された状態で関数の中身が実行されます．またこのとき整数リテラル `3` ， `6` の型は `i32` に推論されます．

コード全体では，たとえば
```rust
fn main() {
    let vec = iota(3, 6);
    assert_eq!(vec, [3, 4, 5]);
    let vec = iota(0, 10);
    assert_eq!(vec, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
}

fn iota(a: i32, b: i32) -> Vec<i32> {
    let mut v = Vec::new();
    for i in a..b {
        v.push(i);
    }
    v
}
```
のようになります．

`a` ， `b` のことを，**引数**（ひきすう）といいます．引数のスコープは，関数の開始から関数の終了までです．

引数をとる関数の定義は，次のようになります．
```rust
fn 関数名(パターン: 型, パターン: 型, ……) -> 返り値の型 {
    関数の中身
}
```
返り値の `-> 型` が省略できないのと同じように，引数の `: 型` も省略できません．

引数をとる関数の呼び出しは，次のようになります．
```rust
関数名(第1引数, 第2引数, ……)
```

呼び出しの際に引数の型が定義と一致していないと，エラーになります．
```rust:コンパイルエラー
let vec = iota(3_i32, 5.5_f64);
```
```
error[E0308]: mismatched types
 --> src/main.rs:2:27
  |
2 |     let vec = iota(3_i32, 5.5_f64);
  |                           ^^^^^^^ expected `i32`, found `f64`
  |
```

また，関数定義の `( )` の中でもパターンが使えます．
```rust
fn main() {
    let tuple = (5, 10);
    assert_eq!(swap(tuple), (10, 5));
}

fn swap((a, b): (i32, i32)) -> (i32, i32) {
    (b, a)
}
```

## 可変の場合
次のコードを，各変数の属する環境に着目して見てみます．
```rust
fn main() {
    let var = 5;
    assert_eq!(double(var), 25);
}

fn double(x: i32) -> i32 {
    x * x
}
```
変数 `var` は `main` 関数の環境に属します．一方で，変数 `x` は `double` 関数の環境に属します．

ここで， `x` に `mut` を付けて可変にしたとします．
```rust
fn main() {
    let var = 5;
    assert_eq!(double(var), 25);
}

fn double(mut x: i32) -> i32 {
    x *= x;
    x
}
```
`double` 関数が呼び出されるときに， `x` に変数 `var` の値 5 が代入されます．その後 `x` の値は 25 に変わり，それが関数の返り値となります．

ここで書き換わったのは，あくまでも `double` 関数の環境にある変数 `x` です．よって， `main` 関数の環境にある変数 `var` は呼び出しの前後で変化しません．
```rust
fn main() {
    let var = 5;
    assert_eq!(double(var), 25);
    assert_eq!(var, 5);
}

fn double(mut x: i32) -> i32 {
    x *= x;
    x
}
```
# `return` 式
`break` 式を使うと，ループを途中で終了させることができました．一方， `return` 式を使うと，関数を途中で終了させることができます．

値を返す `break` 式と同じように，関数を抜けたい箇所で
```rust
return 式;
```
と書きます．また， `()` を返す関数では，単に
```rust
return;
```
と書けます．たとえば，引数として受け取った整数の約数のうち， 2 以上で最小のものを返す `minimum_factor` 関数は次のように書けます．
```rust
fn main() {
    assert_eq!(minimum_factor(2021), 43);
}

fn minimum_factor(n: i32) -> i32 {
    for i in 2.. {
        if i * i > n {
            break;
        } else if n % i == 0 {
            return i;
        }
    }
    n
}
```
- 途中で $i \mid n$ なる $i$ が見つかれば， `return i;` で $i$ の値を返します．
- 途中で $i^2 > n$ になると， $n$ は素数なので `break;` でループを抜け， $n$ の値を返します．
# 標準ライブラリ
よく使われる便利な関数は，自分で定義しなくても最初から用意されている場合があります．

たとえば， 2 つの数を受け取って大きい方を返す `std::cmp::max` や小さい方を返す `std::cmp::min` がその例です．
```rust
fn main() {
    assert_eq!(std::cmp::max(2, 5), 5);
    assert_eq!(std::cmp::min(2, 5), 2);
}
```

このような関数を自分で書くなら，たとえば次のようになります．
```rust
fn max(x: i32, y: i32) -> i32 {
    if x > y {
        x
    } else {
        y
    }
}

fn min(x: i32, y: i32) -> i32 {
    if x < y {
        x
    } else {
        y
    }
}
```

よく使う関数などをまとめたものをライブラリといいます． `std::cmp::max` や `std::cmp::min` は**標準ライブラリ**というライブラリに属しており，何もしなくても最初から使えるようになっています．

標準ライブラリの関数は，名前の先頭に `std::` が付いています（「標準」を意味する standard の略です）．続く `cmp::` は関数の分類を表します（ `max` や `min` は「比較 (comparison) 」に分類されています）． `std::` で始まる関数が出てきたら，「標準ライブラリに含まれている関数で，最初から定義されているんだな」と思ってください．

:::message
ここに書いた `max` `min` の定義だと，型が `i32` の場合にしか使えません．一方， `std::cmp::max` `std::cmp::min` は，比較さえできれば `usize` や `f64` など他の型についても使えます．このように，複数の型に対して使えるような関数の作り方は，[第 31 章](https://zenn.dev/toga/books/rust-atcoder/viewer/31-generic-function)で説明します．
:::

また，標準ライブラリにある関数の一部は，今まで `x.max(y)` や `slice.sort()` のように `変数名.関数名(引数)` という形式で登場していました．実は， `2.max(5)` と書くと， `std::cmp::Ord::max(2, 5)` と同じ意味になります．このような関数については，[第 35 章](https://zenn.dev/toga/books/rust-atcoder/viewer/35-method)で説明します．

## 公式ドキュメント
[公式ドキュメント](https://doc.rust-lang.org/std)には，標準ライブラリに含まれる*全ての*型，関数，マクロ（そして今後登場する構造体，列挙型，トレイト）が載っています．英語で書かれているため，英語を読むのが難しい場合は機械翻訳などを使って読むと良いでしょう．

公式ドキュメントで `std::cmp::max` の説明を見てみましょう．この関数は標準ライブラリの中で `cmp` という分類に属していると言いましたが，この分類を**モジュール**と呼びます．公式ドキュメントのページをスクロールすると[モジュールの一覧](https://doc.rust-lang.org/std/#modules)が出てくるので，ここから `cmp` を探してクリックすると [`cmp` モジュールの中身の一覧](https://doc.rust-lang.org/std/cmp/index.html)に飛ぶことができます．このページを下の方にスクロールすると[関数の一覧](https://doc.rust-lang.org/std/cmp/index.html#functions)があり，その中に先ほどの [`std::cmp::max`](https://doc.rust-lang.org/std/cmp/fn.max.html) があります．ここにある説明を読んでみましょう．

> Compares and returns the maximum of two values.

2 つの値を比較し，最大値を返すと書いています．

> Returns the second argument if the comparison determines them to be equal.

もし値を比較した結果等しかった場合， 2 つめの引数を返すと書いています．これは以下を実行することで確認できます．

```rust
fn main() {
    let x = 10;
    let y = 10;
    let z = std::cmp::max(&x, &y);
    // 2 つの値が等しいので，返り値は &y になる
    println!("&x: {:p}", &x);
    println!("&y: {:p}", &y);
    println!("z:  {:p}", z);
}
```

Argument （引数）のようにプログラミング特有の英単語が登場することも多いです．検索したり翻訳したりしつつ，このような英単語を少しずつ知ってゆくと良いでしょう．

:::message
ここで紹介した公式ドキュメントのリンクは，最新の stable バージョンについて書かれたものです．一方 AtCoder の環境は Rust 1.42.0 なので，差異があるかもしれません．その場合は[バージョン 1.42.0 のドキュメント](https://doc.rust-lang.org/1.42.0/std)を参照しましょう．
:::

# 練習問題
- [ABC189 B - Alcoholic](https://atcoder.jp/contests/abc189/tasks/abc189_b) / `main` 関数の中でも `return` 式が使えます．[解答例](https://atcoder.jp/contests/abc189/submissions/19690590)

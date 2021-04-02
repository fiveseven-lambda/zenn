---
title: "ジェネリクス"
---

# ジェネリクス
`(f64, i32)` 型のタプルを受け取って， 2 つめの要素を返す， `second_f64_i32` 関数を考えます．
```rust
fn main() {
    assert_eq!(second_f64_i32((5., 3)), 3);
}

fn second_f64_i32((_, x): (f64, i32)) -> i32 { x }
```
引数をパターン `(_, x)` で受け取り， `x` を返しています．

同じように， `(f32, i32)` 型のタプルを受け取って 2 つめの要素を返す関数や， `(bool, i32)` 型のタプルを受け取って 2 つめの要素を返す関数なども作れます．
```rust
fn second_f64_i32((_, x): (f64, i32)) -> i32 { x }
fn second_f32_i32((_, x): (f32, i32)) -> i32 { x }
fn second_bool_i32((_, x): (bool, i32)) -> i32 { x }
```
このように同じ書き方ができるたくさんの関数は，**ジェネリクス**を使うと一度に書くことができます．

ジェネリクスを使った関数の定義は，次のようになります：
```rust
fn second_i32<T>((_, x): (T, i32)) -> i32 { x }
```
関数名 `second_i32` の後に `<T>` が付き， `f64` や `f32` や `bool` だった部分が `T` に置き換わっています．こうすると， `T` を `f64` とした関数 `second_i32::<f64>` ， `T` を `f32` とした関数 `second_i32::<f32>` ， `T` を `bool` とした関数 `second_i32::<bool>` などが*全ての型 `T` について*一斉に使えるようになります．
```rust
fn main() {
    assert_eq!(second_i32::<f64>((3_f64, 5)), 5);
    assert_eq!(second_i32::<f32>((3_f32, 5)), 5);
    assert_eq!(second_i32::<bool>((true, 5)), 5);
}
```

この `T` を**型パラメータ**といいます．ジェネリクスを使って定義された関数 `second_i32` を呼び出すときは，引数だけでなく型パラメータも渡す必要があるわけです．

また，型パラメータの個数を増やすこともできます． `<T>` の代わりに `<T, U>` を付ければ，今まで `i32` としていた 2 つめの要素の型も任意の型に変えることができます．
```rust
fn second<T, U>((_, x): (T, U)) -> U { x }
```
たとえば `second::<f64, char>` は「 `(f64, char)` 型のタプルを受け取って 2 つめの要素を返す関数」となります．

:::message
`T` や `U` といった型パラメータの名前のルールは，[変数名のルール](https://zenn.dev/toga/books/rust-atcoder/viewer/05-variable#%E5%A4%89%E6%95%B0%E5%90%8D%E3%81%AE%E3%83%AB%E3%83%BC%E3%83%AB)と同じです．しかし，名付け方の慣例は変数や関数と違い，ふつう大文字 1 文字とします． 2 文字以上の場合は[キャメルケース](https://ja.wikipedia.org/wiki/%E3%82%AD%E3%83%A3%E3%83%A1%E3%83%AB%E3%82%B1%E3%83%BC%E3%82%B9)（ `ElementType` のように大文字と小文字で単語の区切りを表現するやり方）を用います．
:::
## 型推論
上で定義した `second` 関数を使うとき，型パラメータを与える代わりに `_` と書くと，型を推論させることができます．
```rust
let result = second::<bool, _>((true, 65));
assert_eq!(result, b'A');
```
今回， 1 つめの型パラメータ `T` には `bool` を与えましたが， 2 つめの型パラメータ `U` は `_` として型推論に任せています．ここで， `second` 関数の返り値の型が `U` であることから，変数 `result` の型が `U` と同じになるということが分かります． `result` はそのあと `u8` 型のリテラル `b'A'` と比較されているので， `U` は `u8` となります．そして整数リテラル `65` も `u8` 型に推論されます．

また， `true` の型が `bool` なので `T` も型推論に任せることができます．
```rust
let result = second::<_, _>((true, 65));
assert_eq!(result, b'A');
```
このように全ての型パラメータが推論できる状況では， `::<_, _>` を付ける必要が無くなり，普通の関数を呼び出すときと全く同じ書き方ができるようになります．
```rust
let result = second((true, 65));
assert_eq!(result, b'A');
```
:::message
文字 `A` の ASCII コードは 65 です．
:::
# トレイト
`i32` 型の引数を 1 つ受け取って値を出力する `print_i32` 関数を考えます．
```rust
fn print_i32(x: i32) {
    println!("{}", x);
}
```
`f64` や `&str` など他の型についても，同じような関数を考えることができます．では，これをジェネリクスを使って次のように定義しようとするとどうなるでしょうか．
```rust
fn print<T>(x: T) {
    println!("{}", x);
}
```
残念ながら，これはコンパイルエラーになります．型パラメータ `<T>` が付いた関数は全ての型 `T` について使えるようにならなければいけないのに対して，たとえば `T` がタプル `(i32, i32)` のときは， `println!("{}", x);` で値を出力することができないからです．

そこで， `print::<T>` 関数を「全ての型 `T` について」使えるようにするのではなく，「 `{}` で出力できるような全ての型 `T` について」使えるように定義したいです．このような，型 `T` についての条件は，**トレイト**として扱われます．

今回の「 `{}` で出力できる」という条件に対応するトレイトは， `std::fmt::Display` です． `std::fmt::Display` の条件を満たす型 `T` についてのみ `print::<T>` 関数を定義するときは， `<T>` の代わりに `<T: std::fmt::Display>` と書きます．
```rust
fn print<T: std::fmt::Display>(x: T) {
    println!("{}", x);
}
```
「 `T` が `std::fmt::Display` の条件を満たす」という制限を， `print` 関数の**トレイト境界**といいます．

`i32` や `&str` は `std::fmt::Display` の条件を満たすので， `print::<i32>` 関数や `print::<&str>` 関数は使うことができます．
```rust
print(10);
print("Hello");
```
```-:標準出力
10
Hello
```
一方， `print::<(i32, i32)>` 関数を使おうとするとコンパイルエラーになります．
```rust
print((10_i32, 20_i32));
```
```
error[E0277]: `(i32, i32)` doesn't implement `std::fmt::Display`
 --> src/main.rs:2:11
  |
2 |     print((10_i32, 20_i32));
  |           ^^^^^^^^^^^^^^^^ `(i32, i32)` cannot be formatted with the default formatter
...
5 | fn print<T: std::fmt::Display>(x: T) {
  |    -----    ----------------- required by this bound in `print`
  |
  = help: the trait `std::fmt::Display` is not implemented for `(i32, i32)`
  = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
```
## `impl`
また，このようにトレイト境界をもつジェネリクスは `impl` キーワードを使うと次のように簡単に書くことができます．
```rust
fn print(x: impl std::fmt::Display) {
    println!("{}", x);
}
```
関数名の後の `<T: std::fmt::Display>` が無くなり，代わりに `T` と書いていたところに `impl std::fmt::Display` と書いています．書き方が違うだけで，意味は同じです．
## `where`
トレイト境界は， `where` キーワードを使って `{` の直前に書くこともできます．
```rust
fn print<T>(x: T)
where
    T: std::fmt::Display,
{
    println!("{}", x);
}
```
関数が値を返すときは， `-> (型)` の後に `where (トレイト境界)` を書きます．
```rust
fn print<T>(x: T) -> T
where
    T: std::fmt::Display,
{
    println!("{}", x);
    x
}
```
## 複数のトレイト境界
複数のトレイト境界は， `+` でつないで書きます．

`{}` での出力ができるという条件には `std::fmt::Display` トレイトが対応しました．一方， `{:?}` でのデバッグ出力ができるという条件には `std::fmt::Debug` トレイトが対応します．

関数内で `{}` での出力と `{:?}` でのデバッグ出力を両方行いたいときは，型パラメータ `T` が `std::fmt::Display` の条件と `std::fmt::Debug` の条件を両方満たす必要があります．このように `T` に複数の条件を課す場合は， `+` 演算子を使って `T: std::fmt::Display + std::fmt::Debug` と書きます．

```rust
fn print_display_and_debug<T: std::fmt::Display + std::fmt::Debug>(x: T){
    println!("{}", x);
    println!("{:?}", x);
}
```

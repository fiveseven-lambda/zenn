---
title: "Hello, world!"
---

# `main` 関数
プログラムを書くとき，最低でもこれだけは書かなければなりません：
```rust
fn main() {
    
}
```
これは実行しても何もせずそのまま正常終了するプログラムです．

`fn main()` は，**`main` 関数**というものを表します．一つのプログラムには必ず一つの `main` 関数が無くてはなりません．

プログラムの中身は， `fn main()` の後に続く括弧 `{ }` の中に書きます．今回はここが空なので，何もしないプログラムということになります．
# 出力
## `println!` マクロ
次は，標準出力に `Hello, world!` と出力して，最後に改行し，終了するプログラムです．
```rust
fn main() {
    println!("Hello, world!");
}
```
括弧の中に， `println!("Hello, world!");` と書かれた一行が存在します．これが「`Hello, world!` と出力し，最後に改行する」という意味になります．

`"Hello, world!"` のように， 2 つのダブルクォーテーションマーク `"` で囲まれた部分は，**文字列リテラル**と呼ばれます． `"` で囲むのを忘れてはいけません．

**`println!` マクロ**を使うと，標準出力に文字列を出力することができます．`println!( );` の括弧の中に文字列リテラルを書くと，その中身が出力され，最後に改行が行われます．一方，括弧の中に何も書かずただ `println!();` と書くと，改行だけが出力されます．

最後にセミコロン `;` を付けるのを忘れないようにしましょう．

プログラムの中身は上から順に実行されます．
```rust
fn main() {
    println!("今すぐダウンロー");
    println!();
    println!("ド");
}
```
を実行すると，
```
今すぐダウンロー

ド
```
となります．
## `print!` マクロ
`println!` マクロと違い， **`print!` マクロ**は最後に改行を出力しません．
```rust
fn main() {
    print!("こ");
    print!("ん");
    print!("に");
    print!("ち");
    print!("は");
}
```
と書くと，出力は
```
こんにちは
```
となります．
## `eprintln!` マクロ， `eprint!` マクロ
**`eprintln!` マクロ**は，標準出力ではなく標準エラー出力に出力します． **`eprint!` マクロ**も標準エラー出力に出力しますが，最後に改行を出力しません．
```rust
fn main() {
    eprintln!("An error occurred");
}
```
# エラー
## コンパイルエラー
次のコードは， `println!` の最後のエクスクラメーションマーク `!` を書き忘れています．
```rust
fn main() {
    println("Hello");
}
```
このように，間違ったコードを書くとコンパイルエラーになります．

# コンパイルエラーの読み方
コンパイルエラーが起こったとき，エラーメッセージにそのエラーについての情報が書かれています．エラーメッセージを読むのは非常に大事です．

## ケース 1
たとえば，さっきの間違ったコード
```rust
fn main() {
    println("Hello");
}
```
をコンパイルすると，
```
error[E0423]: expected function, found macro `println`
 --> src/main.rs:2:5
  |
2 |     println("Hello");
  |     ^^^^^^^ help: use `!` to invoke the macro: `println!`
```
というエラーが出力されます．

最初の `` expected function, found macro `println` `` というのは，「`println(...);` という構文中で `println` の部分には関数が来るはずだが，実際には `println` は関数ではなくマクロだった」という意味です．

それに続き `src/main.rs:2:5` と書いているのは，これが `src/main.rs` というファイル中の 2 行目 5 文字目で発生したエラーであるという意味です（AtCoder ではファイル名は常に `src/main.rs` です）．よってソースコードの 2 行目周辺に着目することでエラーの原因が判明する可能性が高いです．

さらにその後に，
```
  |
2 |     println("Hello");
  |     ^^^^^^^ help: use `!` to invoke the macro: `println!`
```
と書かれています． 2 行目の `println` の部分について， `` help: use `!` to invoke the macro: `println!` ``，つまり「`println!` マクロを呼び出したいのであれば， `!` を使うのはどうか」と提案されています．この help の指示に従うことでエラーが解決することは多いです．

## ケース 2
今度は次のコードをそっくりそのままコピーペーストしてコードテストを実行してみましょう．
```rust
fn main() {
　println!("Hello");
}
```

見た目では非常に分かりづらいのですが， 2 行目のインデントが半角空白「` `」ではなく全角空白「`　`」になっています．空白文字として全角空白を用いることは許されていません．これをコンパイルすると
```
error: unknown start of token: \u{3000}
 --> src/main.rs:2:1
  |
2 | 　println!("Hello");
  | ^^
  |
help: Unicode character '　' (Ideographic Space) looks like ' ' (Space), but it is not
  |
2 |  println!("Hello");
  | --
```
というエラーが出力されます．`Unicode character '　' (Ideographic Space) looks like ' ' (Space), but it is not` というのは，「Unicode において Ideographic Space と名付けられている文字 '　' は Space という文字 ' ' に似ているが，これらは異なる文字である」という意味です．よって，全角空白を半角空白に直せば良いのだと分かります．

このように，エラーが起こったらまずはエラーメッセージを読むのが大事です．

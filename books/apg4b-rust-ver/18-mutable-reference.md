---
title: "可変参照"
---

# 可変変数への参照
可変変数も借用することができます．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &hoge;
    println!("{}", reference);
}
```
可変変数 `hoge` への参照を `reference` に代入し， `println!` で出力しています．

## 参照と可変性
元の変数が可変であったとしても，参照を介してこれを書き換えることはできません．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &hoge;
    assert_eq!(*reference, 10); // 使用は OK
    *reference = 20; // 代入はエラー
}
```
`*reference` を使用することはできますが， `*reference` に値を代入することはできません．

## ライフタイムと可変性
次のコードはエラーになります．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &hoge;
    println!("{}", reference);
    hoge = 20;
    println!("{}", reference);
}
```
`hoge = 20;` で `hoge` の中身を書き換えています．すると， `reference` に対しては何も手を加えていないのに， `reference` の参照先が勝手に書き換わってしまいます．このように何もしていないのに参照の中身が変化することは起こらないようになっています．

エラーメッセージは次のようになります．
```
error[E0506]: cannot assign to `hoge` because it is borrowed
 --> src/main.rs:5:5
  |
3 |     let reference = &hoge;
  |                     ----- borrow of `hoge` occurs here
4 |     println!("{}", reference);
5 |     hoge = 20;
  |     ^^^^^^^^^ assignment to borrowed `hoge` occurs here
6 |     println!("{}", reference);
  |                    --------- borrow later used here
```
``cannot assign to `hoge` because it is borrowed`` は，「`hoge` が借用されているので， `hoge` に対する代入はできない」という意味です．

借用された可変変数が書き換えられないのは，参照のライフタイムが終了するまでです．よって，次のコードは正常に動きます．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &hoge;
    println!("{}", reference);
    hoge = 20;
}
```
`&hoge` のライフタイムが，代入より先に終了しているからです．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &hoge;
    println!("{}", reference); // &hoge のライフタイムの終了
    hoge = 20; // hoge への代入が可能
}
```
## 複数回の借用
可変変数も，複数回借用することができます．
```rust
fn main() {
    let mut hoge = 10;
    let reference1 = &hoge;
    let reference2 = &hoge;
    assert_eq!(*reference1, 10);
    assert_eq!(*reference2, 10);
    hoge = 20; // 代入が可能
}
```
`hoge` に対する代入が可能になるのは，*全ての*参照のライフタイムが終了してからです．
# 可変参照
可変変数は，普通の参照の他に**可変参照**をとることができます．可変参照は，参照を外して中身を書き換えることが可能です．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &mut hoge;
    assert_eq!(*reference, 10);
    *reference = 20; // 代入が可能
    assert_eq!(*reference, 20);
}
```
`&` の後に `mut` を付けて `&mut` とすると，可変として借用することができます．

可変参照の型は `&T` ではなく `&mut T` です．よって `reference` の型は `&mut i32` です．
```rust
fn main() {
    let mut hoge = 10;
    let reference: &mut i32 = &mut hoge;
    assert_eq!(*reference, 10);
    *reference = 20;
    assert_eq!(*reference, 20);
}
```

可変参照に対し，可変でない参照を不変参照といいます．
## 可変性
可変参照も，普通の参照と同じように，ライフタイムが終了するまでもとの変数に値を代入することができません．よって次のコードはエラーになります．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &mut hoge;
    println!("{}", reference);
    hoge = 20; // エラー： hoge への代入
    println!("{}", reference); // &mut hoge のライフタイムの終了
}
```
## 変数が可変でない場合
もとの変数が可変として宣言されていない場合，可変として借用することができません．
```rust
fn main() {
    let hoge = 10;
    let reference = &mut hoge; // エラー
}
```
```
error[E0596]: cannot borrow `hoge` as mutable, as it is not declared as mutable
 --> src/main.rs:3:21
  |
2 |     let hoge = 10;
  |         ---- help: consider changing this to be mutable: `mut hoge`
3 |     let reference = &mut hoge; // エラー
  |                     ^^^^^^^^^ cannot borrow as mutable
```
``cannot borrow `hoge` as mutable, as it is not declared as mutable`` は，「`hoge` は可変として宣言されていないので，可変として借用することができない」という意味です．

また，借用によって可変でなくなっている変数についても，可変として借用することができません．
```rust
fn main() {
    let mut hoge = 10;
    let immutable_reference = &hoge; // &hoge のライフタイムの開始
    let mutable_reference = &mut hoge; // エラー
    println!("{}", immutable_reference); // &hoge のライフタイムの終了
}
```
参照 `&hoge` が `immutable_reference` に代入されているので，このライフタイムが終了するまでの間もとの変数 `hoge` を書き換えることができません．このような状況下では， `hoge` を可変として借用することもできません．
```
error[E0502]: cannot borrow `hoge` as mutable because it is also borrowed as immutable
 --> src/main.rs:4:29
  |
3 |     let immutable_reference = &hoge; // hoge のライフタイムの開始
  |                               ----- immutable borrow occurs here
4 |     let mutable_reference = &mut hoge;
  |                             ^^^^^^^^^ mutable borrow occurs here
5 |     println!("{}", immutable_reference); // hoge のライフタイムの終了
  |                    ------------------- immutable borrow later used here
```
``cannot borrow `hoge` as mutable because it is also borrowed as immutable`` は，「`hoge` は不変として借用されているので，可変として借用することができない」という意味です．

もちろん，最初の借用が可変だとしても同じです．
```rust
fn main() {
    let mut hoge = 10;
    let reference1 = &mut hoge; // &mut hoge のライフタイムの開始
    let reference2 = &mut hoge; // エラー
    println!("{}", reference1); // &mut hoge のライフタイムの終了
}
```
エラーメッセージの文面だけが少し変わります．
```
error[E0499]: cannot borrow `hoge` as mutable more than once at a time
 --> src/main.rs:4:22
  |
3 |     let reference1 = &mut hoge; // &mut hoge のライフタイムの開始
  |                      --------- first mutable borrow occurs here
4 |     let reference2 = &mut hoge; // エラー
  |                      ^^^^^^^^^ second mutable borrow occurs here
5 |     println!("{}", reference1); // &mut hoge のライフタイムの終了
  |                    ---------- first borrow later used here
```
``cannot borrow `hoge` as mutable more than once at a time`` は，「`hoge` を一度に 2 回以上可変として借用することはできない」という意味です．
## 可変として借用されている変数の使用
次のコードはエラーになります．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &mut hoge;
    let fuga = hoge + 20;
    *reference += 30;
}
```
可変として借用された変数は，いつ書き換えられるか分かりません．このような状態で，もとの変数を使用したり借用したりすることはできません．
```
error[E0503]: cannot use `hoge` because it was mutably borrowed
 --> src/main.rs:4:16
  |
3 |     let reference = &mut hoge;
  |                     --------- borrow of `hoge` occurs here
4 |     let fuga = hoge + 20;
  |                ^^^^ use of borrowed `hoge`
5 |     *reference += 30;
  |     ---------------- borrow later used here
```
``cannot use `hoge` because it was mutably borrowed`` は，「`hoge` は可変として借用されているので，使用することができない」という意味です．

可変として借用された変数が使用/借用できないのは，可変参照のライフタイムが終了するまでです．よって，次のコードは正常に動きます．
```rust
fn main() {
    let mut hoge = 10;
    let reference = &mut hoge;
    *reference += 30; // &mut hoge のライフタイムの終了
    let fuga = hoge + 20;
    assert_eq!(fuga, 60);
}
```
# パターンマッチ
## `&mut` パターン
可変参照のパターンマッチには `&mut` を使います．
```rust
fn main() {
    let mut hoge = 10;
    let &mut copied = &mut hoge;
}
```
左辺が `&mut` なのに右辺が `&` だったり，
```rust
fn main() {
    let mut hoge = 10;
    let &mut copied = &hoge;
}
```
左辺が `&` なのに右辺が `&mut` だったりすると，
```rust
fn main() {
    let mut hoge = 10;
    let &copied = &mut hoge;
}
```
エラーになります．
```
error[E0308]: mismatched types
 --> src/main.rs:3:9
  |
3 |     let &copied = &mut hoge;
  |         ^^^^^^^   --------- this expression has type `&mut {integer}`
  |         |
  |         types differ in mutability
  |
  = note: expected mutable reference `&mut {integer}`
                     found reference `&_`
```
`types differ in mutability` は，「型の可変性が異なる」という意味です．
## `ref` パターン
`ref` パターンは変数を不変として借用します．
```rust
fn main() {
    let mut hoge = 10;
    let ref reference = hoge;
    assert_eq!(*reference, 10);
}
```
`reference` には不変参照 `&hoge` が代入されています．

一方，変数名に `ref` と `mut` を両方付けると可変参照になります．
```rust
fn main() {
    let mut hoge = 10;
    let ref mut reference = hoge;
    *reference = 20;
    assert_eq!(hoge, 20);
}
```
`reference` には可変参照 `&mut hoge` が代入されています．

`ref` と `mut` の順番を逆にすると，
```rust
fn main() {
    let mut hoge = 10;
    let mut ref reference = hoge;
    *reference = 20;
    assert_eq!(hoge, 20);
}
```
エラーになります．
```
error: the order of `mut` and `ref` is incorrect
 --> src/main.rs:3:9
  |
3 |     let mut ref reference = hoge;
  |         ^^^^^^^ help: try switching the order: `ref mut`
```
``the order of `mut` and `ref` is incorrect`` は，「`mut` と `ref` の順番が間違っている」という意味です． `` try switching the order: `ref mut` `` は，「順番を入れ替えて `ref mut` としたらどうか」という提案です．
## `&mut` と `ref mut` の省略
`&(ref x, ref y)` が `(x, y)` と省略できたのと同様に， `&mut (ref mut x, ref mut y)` も `(x, y)` と省略することができます．
```rust
fn main() {
    let mut tuple = (3, 10);
    let (x, y) = &mut tuple;
    *x *= *y;
    assert_eq!(tuple, (30, 10));
}
```
![](https://storage.googleapis.com/zenn-user-upload/j62ak80e37fmxnj2e92f04cc13y0)

`x` と `y` の型はともに `&mut i32` となり， `x` には `&mut tuple.0` ， `y` には `&mut tuple.1` が代入されます．

# まとめ
可変性と借用に関するルールは，次の 3 つに集約されます．
- 不変参照は参照先を書き換えられない．一方，可変参照は参照先を書き換えられる．
- 変数が借用されている間，変数を書き換えたり可変として借用したりすることはできない．
- 変数が可変として借用されている間，変数を使用したり借用したりすることはできない．
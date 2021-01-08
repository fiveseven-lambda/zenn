---
title: "参照とライフタイム"
---

# 参照
変数は，メモのようなものだと言いました．

たとえば
```rust
fn main() {
    let hoge: i8 = 100;
    println!("{}", hoge);
}
```
というコードを見てみましょう．変数 `hoge` の型は `i8`，そこに代入された値は 100 です． `i8` のバイト長は 1 なので， `hoge` のために 1 バイト分のメモリが使われています．

プログラムが使用するメモリは， 1 バイト分のメモがたくさん並んだようなものです． `let hoge: i8 = 100;` と書くと，たくさんのメモの中のどれか一つに `hoge` という名前が付けられて， 100 という値が書き込まれます．

![](https://storage.googleapis.com/zenn-user-upload/4opigavf5d2ubxk088x1ivkg268x)

実は，これらのメモには全て番号が振られています．これを変数の**アドレス**といいます．

![](https://storage.googleapis.com/zenn-user-upload/j0il35v3qw88qwpu4h4jv1xwldya)

どのメモが使われるかは，プログラム実行時に決められます．プログラマーが変数のアドレスを決定する普通の手段は存在しません．

しかし，変数のアドレスを*知る*手段は存在します．
```rust
fn main() {
    let hoge: i8 = 100;
    let reference = &hoge;
    println!("{:p}", reference);
}
```
`hoge` の前に `&` を付けたものは，変数 `hoge` のアドレスを表します．これを変数 `reference` に代入したので， `reference` の値は `hoge` のアドレスになります．

`println!` マクロのフォーマット文字列で `{:p}` とすると，アドレスが 16 進数で出力されます．よって，出力は（たとえば） `7ffc0ec6c8df` （=140720556394719）のようになります．この値は実行するたびに変わり得ます．

`&` を付けることで得られる変数 `hoge` のアドレスを，`hoge` への**参照**といいます．`&` を付けて `hoge` への参照をとることを， `hoge` を**借用**するといいます．

参照 `&hoge` は `&i8` という型をもちます．一般に，`T` 型の変数への参照は `&T` 型になります．同じアドレスでも， `&i8` と `&u8` のように `T` が異なるものは別の型です．
```rust
fn main() {
    let hoge: i8 = 100;
    let reference: &i8 = &hoge;
    println!("{:p}", reference);
}
```

また，未初期化の変数を借用することはできません．
```rust
fn main() {
    let hoge: i8;
    let reference = &hoge;
}
```
```
error[E0381]: borrow of possibly-uninitialized variable: `hoge`
 --> src/main.rs:3:21
  |
3 |     let reference = &hoge;
  |                     ^^^^^ use of possibly-uninitialized `hoge`
```
`` borrow of possibly-uninitialized variable: `hoge` `` は「未初期化の可能性のある変数 `hoge` を借用しようとしている」という意味です．

ここまでの例で，変数 `hoge` は `i8` という 1 バイトの型でした．もし `hoge` が `i32` （4 バイト）や `[i32; 10]` （40 バイト）のように 1 バイトより大きい型だったら，図のようにメモリ上の*連続した領域*が使われ， `&` を付けて得られるのはその*先頭の*アドレスになります．

![](https://storage.googleapis.com/zenn-user-upload/lc3qzrsa355v0mwnpsez6hlyboov)

# 参照外し
`reference` には，変数 `hoge` のアドレスが入っています．であれば， `reference` に入っている値さえ分かっていれば，メモリ上でそのアドレスを見にゆくことで変数 `hoge` の値を間接的に知ることもできるでしょう．これを**参照外し**といい， `reference` の前に `*` 演算子を付けることでできます．

```rust
fn main() {
    let hoge: i8 = 100;
    let reference = &hoge;
    assert_eq!(*reference, 100_i8);
}
```

`*reference` は，メモリ上でアドレスが `reference` の場所に入っている値です．今回は `reference` が変数 `hoge` のアドレスなので， `*reference` は `hoge` の値，すなわち 100 になります．

## 型推論
型推論は，参照/参照外しを越えて行われます．上のコードで， `hoge` の型注釈を外して
```rust
fn main() {
    let hoge = 100;
    let reference = &hoge;
    assert_eq!(*reference, 100_i8);
}
```
とすると，「`*reference` と `100_i8` が比較されているので `reference` の型は `&i8` でなくてはならない」「`&hoge` を `reference` に代入しているので `hoge` の型は `i8` でなくてはならない」という推論が働いて，変数 `hoge` と整数リテラル `100` は `i8` 型に推論されます．

## 自動的な参照外し
参照は，自動的に外されることがあります．次のコードを見てください．
```rust
fn main() {
    let hoge: i8 = 100;
    let reference = &hoge;
    assert_eq!(reference + 1_i8, 101_i8);
}
```
`reference + 1_i8` という式の中で， `reference` の型は `&i8` である一方， `1_i8` の型は `i8` です． `&i8` と `i8` の足し算は不可能ですが， `i8` と `i8` の足し算であれば可能です．このような状況では， `reference` が自動的に `*reference` に置き換えられて， 100 + 1 が計算されます．

:::message
初心者にとって，自動的な参照外しが起こる条件を正確に把握するのは難しいと思います．少なくとも `*` を付けて手動で参照外しを行っていれば間違いは起こらないので，慣れるまでは使わないでも良いでしょう．自動的な参照外しのメリットは， `*` の数が減ってコードが見やすくなることです．慣れてきたら少しずつ使ってみましょう．
:::

## `println!` による出力
`println!` マクロのフォーマット文字列では， `{ }` の中身を変えることで様々な出力形式を選択できました．しかし，参照型でない `i32` や `f64` を `{:p}` で出力することはできません．
```rust
fn main() {
    println!("{:p}", 100);
}
```

一方，参照型を `{:p}` 以外で出力することは可能です．
```rust
fn main() {
    let hoge = 100;
    let reference = &hoge;
    println!("{}", reference);
}
```
`reference` の参照が外されて， `*reference` の値が出力されます．よって出力は `100` となります．
:::message
これは，自動的な参照外しではありません． `{:p}` が `i32` や `f64` の出力に対応していない一方， `{}` が `&i32` や `&f64` の出力にも対応しているということです．
:::
# パターンマッチ
参照もパターンマッチが可能です．
```rust
fn main() {
    let hoge = 10;
    let reference = &hoge;
    let &copied = reference;
    assert_eq!(copied, 10);
    println!("hoge:   {:p}", &hoge);
    println!("copied: {:p}", &copied);
}
```
`let &copied = reference;` の行で，参照形式のパターンを使っています．`=` の右辺が `&i32` 型なので， `copied` は `i32` 型になり，そこには `hoge` の値が代入されます．

![](https://storage.googleapis.com/zenn-user-upload/xcg5yxri0d7rzu9hjfthrd5a87rd)

`copied` の値は `hoge` と同じ 10 ですが， `hoge` と `copied` のアドレスをそれぞれ出力してみると異なる値になっていることが分かるはずです．
# ライフタイム
あるブロックの中で宣言された変数のスコープは，そのブロックが終わるまででした．
```rust
fn main() {
    {
        let hoge = 100;
    }
    // ここで hoge は存在しない
}
```
では，この変数への参照を，ブロックから返したらどうなるでしょうか．
```rust
fn main() {
    let reference = {
        let hoge = 100;
        &hoge
    };
    println!("{}", reference);
    // 存在しない変数 hoge の値が出力されるのか？
}
```
これは，次のようなエラーとなります．
```
error[E0597]: `hoge` does not live long enough
 --> src/main.rs:4:9
  |
2 |     let reference = {
  |         --------- borrow later stored here
3 |         let hoge = 100;
4 |         &hoge
  |         ^^^^^ borrowed value does not live long enough
5 |     };
  |     - `hoge` dropped here while still borrowed
```
このエラーは，以下で説明する重要なルールの存在によって生じています．

参照には，**ライフタイム**というものがあります．ある参照のライフタイムは，*変数が借用されてから，最後にその参照が使用されるまで*をいいます．
```rust
fn main() {
    let hoge = 100;
    let reference = &hoge; // 借用：ライフタイムの開始
    println!("{}", reference); // 最後の使用：ライフタイムの終了
}
```
`&hoge` と書いて変数 `hoge` が借用されたときに，この参照 `&hoge` のライフタイムが開始します．そして `reference` に代入された `&hoge` の値が最後に使用されるのは `println!("{}", reference);` の箇所なので，ここで `&hoge` のライフタイムが終了します．

ここで，次のようなルールが存在します：*参照のライフタイムは，元の変数のスコープをはみ出してはなりません*．

この例では，ライフタイムの開始がスコープの開始より遅く，かつライフタイムの終了がスコープの終了より早いので，問題なく動きます．
```rust
fn main() {
    let hoge = 100; // hoge のスコープの開始
    let reference = &hoge; // &hoge のライフタイムの開始
    println!("{}", reference); // &hoge のライフタイムの終了
} // hoge のスコープの終了
```

一方，先ほどの例では，
```rust
fn main() {
    let reference = {
        let hoge = 100;
        &hoge
    }; // hoge のスコープの終了
    println!("{}", reference); // &hoge のライフタイムをここまで続けようとしている
}
```
`reference` に代入された `&hoge` のライフタイムの終了点を，ブロックの外側まで引き延ばそうとしています．よってこのコードはエラーとなります．

ここで問題です．次のコードはエラーになるでしょうか？
```rust
fn main() {
    let reference;
    {
        let hoge = 100;
        reference = &hoge;
        println!("{}", reference);
    }
}
```
:::details クリックで答えを表示
エラーにはなりません．
```rust
fn main() {
    let reference;
    {
        let hoge = 100; // hoge のスコープの開始
        reference = &hoge; // &hoge のライフタイムの開始
        println!("{}", reference); // &hoge のライフタイムの終了
    } // hoge のスコープの終了
}
```
`hoge` のスコープは，宣言されてからブロックが終了するまでです．一方， `&hoge` のライフタイムは，借用されてから最後に使用されるまでです．最後に使用されるのがブロックの内側なので， `&hoge` のライフタイムは `hoge` のスコープをはみ出していません．
:::

また，同じ `hoge` を複数回借用したら，
```rust
fn main() {
    let hoge = 100;
    let reference1 = &hoge;
    let reference2 = &hoge;
    println!("{}", reference1);
    assert_eq!(*reference2, 100);
}
```
それぞれに対してライフタイムが存在します．
```rust
fn main() {
    let hoge = 100;
    let reference1 = &hoge; // 1 つめの &hoge のライフタイムの開始
    let reference2 = &hoge; // 2 つめの &hoge のライフタイムの開始
    println!("{}", reference1); // 1 つめの &hoge のライフタイムの終了
    assert_eq!(*reference2, 100); // 2 つめの &hoge のライフタイムの終了
}
```
## 静的なライフタイム
整数リテラルのようなリテラルも，借用することができます．
```rust
fn main() {
    let reference = &100;
    println!("{:p}", reference);
    assert_eq!(*reference, 100);
}
```
このとき， `&100` は「プログラム開始からプログラム終了までずっと続く変数」への参照だと思うことができます．このような参照は，**静的な**ライフタイムをもつといいます．

静的なライフタイムをもつ参照は，プログラム終了までずっと使うことができるので，たとえば次のコードは正常に動きます．
```rust
fn main() {
    let reference;
    {
        reference = &100;
    }
    assert_eq!(*reference, 100);
}
```

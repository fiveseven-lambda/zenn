---
title: "構造体"
---

# 構造体
今まで様々な型が登場してきましたが，実は新しい型を定義することも可能です．この章で説明する**構造体**や，次の章で説明する列挙体を用いると，自分で作った型に名前を付け，意味をもたせることができます．その動機は，[第 25 章](https://zenn.dev/toga/books/rust-atcoder/viewer/25-function)でブロックに名前を付けて関数としたことと似ています．型に名前を付けることで，それが何を表す型なのか分かりやすくなるのです．

構造体には，タプル構造体と，名前付きフィールドをもつ構造体の 2 種類があります．

# タプル構造体
[第 11 章](https://zenn.dev/toga/books/rust-atcoder/viewer/11-tuple)で登場したタプルは，複数の型 `T1` `T2` `T3` ……を組み合わせて型 `(T1, T2, T3, ……)` とするものでした．これに名前を付けて新しい型とするのが，**タプル構造体**です．

## 定義
タプル `(i32, i32)` に `Point` という名前を付けてタプル構造体とするには，次のように書きます．
```rust
struct Point(i32, i32);
```

構造体に名前を付けるときのルールは，[第 5 章](https://zenn.dev/toga/books/rust-atcoder/viewer/05-variable#%E5%A4%89%E6%95%B0%E5%90%8D)で説明した変数名のルールと同じです．一方，慣例は変数名と異なります．
- 先頭は大文字にします．
- 複数の単語をつなげるときはアンダースコア `_` を挟まず，代わりに `StudentData` のように各単語の頭文字を大文字にします．これを，スネークケースに対し**キャメルケース**といいます．

構造体の定義を書ける場所は，関数と同じです．ブロックの中で行うこともできれば，全ての関数の外側で行うこともできます．スコープについても関数と同じです．
```rust
fn main() {
    // ここでは Point も Vector も使える
    struct Vector(i32, i32);
}
fn fnc() {
    // ここでは Point しか使えない
}
struct Point(i32, i32);
```
同じスコープで同じ名前の構造体を定義してもいけません．
```rust
struct Point(i32, i32);
struct Point(i32, i32); // エラー
```

## インスタンス
定義した構造体 `Point` は，新しい型として使えます．よって，型が `Point` であるような変数を宣言することができます．
```rust
let point: Point;
```

`Point` は， `(i32, i32)` とは別の型として扱われます．よって，次の代入はエラーになります．
```rust
let point: Point = (2_i32, 3_i32); // エラー
```
`(i32, i32)` 型の値を `Point` 型の変数に代入しようとしているからです．

`Point` 型の値を作るときは， `(2, 3)` の代わりに `Point(2, 3)` と書きます．
```rust
let point: Point = Point(2, 3);
```
こうして作られた `Point` 型の値を，構造体 `Point` の**インスタンス**といいます．

## フィールド
構造体 `Point(2, 3)` を構成する各々の値 `2` `3` を，構造体の**フィールド**といいます．タプルと同様に， `.0` `.1` `.2` ……を用いてフィールドにアクセスすることができます．
```rust
let point = Point(2, 3);
assert_eq!(point.0, 2);
assert_eq!(point.1, 3);
```
```rust
let mut point = Point(2, 3); // 可変として宣言
std::mem::swap(&mut point.0, &mut point.1); // 2 つのフィールドの値を入れ替える
assert_eq!(point.0, 3);
assert_eq!(point.1, 2);
```
## 要素数 1 のタプル
要素数 1 のタプルを `(i32)` や `(5)` と書こうとすると， `i32` や `5` にただ括弧が付いただけのものと扱われてしまいます．よって，要素数 1 のタプルは `(i32,)` や `(5,)` と書きます．
```rust
let tuple: (i32,) = (5,);
```
一方，要素数 1 のタプル構造体は，ただの括弧と紛らわしくなることが無いので，最後にカンマ `,` を付ける必要がありません．
```rust
struct Length(i32);
let length = Length(10);
assert_eq!(length.0, 10);
```

## 型による区別
2 つの `i32` 値で平面上の点を表す `Point` 構造体と，身長と体重を表す `Physical` 構造体があるとします．
```rust
struct Point(i32, i32);
struct Physical(i32, i32);
```
`Point` と `Physical` は別の型なので，三角形の頂点の座標と David の身体測定の結果を取り違えるようなミスは防ぐことができます．
```rust
let apex: Point = Physical(170, 50); // エラー
let david: Physical = Point(7, 12); // エラー
```
また， `Point(7, 12)` を見た人は「これは点の座標なんだな」， `Physical(170, 50)` を見た人は「これは身長と体重のデータなんだな」と思うことができます．

これらは，ただのタプル `(i32, i32)` にはできなかったことです．

# 名前付きフィールド

上で，身長と体重を表す `Physical` という構造体を定義しました．
```rust
struct Physical(i32, i32);
```
しかし，これだと `.0` と `.1` のどちらが身長でどちらが体重か分かりません．**名前付きフィールド**をもつ構造体を使うと，各フィールドにも名前を付けることができます．
## 定義
名前付きフィールドをもつ構造体は，次のように定義します．
```rust
struct Physical {
    height: i32,
    weight: i32,
}
```
1 つめのフィールドに `height`， 2 つめのフィールドに `weight` という名前が付きます． `height` が身長， `weight` が体重を表すということがはっきりします．
:::message
最後のフィールド `weight: i32` の後のカンマ `,` は無くてもかまいません．
:::
## インスタンス
こうして定義した構造体 `Physical` のインスタンスは，次のように生成します．
```rust
let david = Physical {
    height: 170,
    weight: 50,
};
```
`Physical` 型の変数 `david` に，身長が 170，体重が 50 であることを表す `Physical` 型の値が代入されます．

このとき，フィールドの順番を変えてもかまいません．次のように書いても，同じインスタンスが生成されます．
```rust
let david = Physical {
    weight: 50,
    height: 170,
};
```
:::message
`height = 170` ではなく `height: 170` であることに注意してください．また，最後の `}` の後にセミコロン `;` が必要なことにも注意してください．
:::
## フィールド
タプル構造体のフィールドにアクセスするときは， `.0` `.1` `.2` ……のように番号を指定しました．一方，名前付きフィールドにアクセスするときは， `.height` `.weight` のように，ピリオド `.` に続けてフィールド名を指定します．
```rust
let david = Physical {
    height: 170,
    weight: 50,
};
assert_eq!(david.height, 170); // 身長を表すフィールド
assert_eq!(david.weight, 50); // 体重を表すフィールド
```
フィールド名は，現れる場所によって変数名と区別されます． `david.height` `david.weight` ではなく `height` `weight` と書けば，それは `height` `weight` という名前の変数を指します．
```rust
let david = Physical {
    height: 170,
    weight: 50,
};
let height = 175;
let weight = 60;

assert_eq!(david.height, 170); // 変数 david の，フィールド height
assert_eq!(height, 175); // 変数 height
```
# パターンマッチ
構造体もパターンマッチが可能です．
## タプル構造体
タプル構造体のパターンマッチは次のようになります：
```rust
let point = Point(1, 2);
let Point(x, y) = point;
assert_eq!(x, 1);
assert_eq!(y, 2);
```
`Point(x, y)` の部分が**タプル構造体パターン**です．これにより， `i32` 型の変数 `x` と `i32` 型の変数 `y` が作られ，それぞれ 1， 2 が代入されます．
![](https://storage.googleapis.com/zenn-user-upload/20f808a7d2385ed2cbe6ec51.png)
## 名前付きフィールドをもつ構造体
名前付きフィールドをもつ構造体のパターンマッチは次のようになります：
```rust
let david = Physical {
    height: 170,
    weight: 50,
};
let Physical {
    height: h,
    weight: w,
} = david;
assert_eq!(h, 170);
assert_eq!(w, 50);
```
`Physical { height: h, weight: w, }` の部分が**構造体パターン**です．これにより， `i32` 型の変数 `h` と `i32` 型の変数 `w` が作られ，それぞれ 170， 50 が代入されます．
![](https://storage.googleapis.com/zenn-user-upload/e851ee423410a5e3d8cf6058.png)
コロン `:` の前の `height` `weight` はフィールド名，コロンの後の `h` `w` は変数名です．
# フィールドの省略記法
名前付きフィールドにおいて，フィールド名と変数名が同じであれば， `height: height` を単に `height` と書くことができます．

インスタンスの生成時における省略の例：
```rust
let height = 170;
let david = Physical {
    height,
    weight: 50,
};
/*
let david = Physical {
    height: height,
    weight: 50
};
と同じ
*/
assert_eq!(david.height, 170);
```
パターンマッチにおける省略の例：
```rust
let Physical {
    height,
    weight: w
} = david;
/*
let Physical {
    height: height,
    weight: w
} = david;
と同じ
*/
assert_eq!(height, 170);
```
# 演算，出力，コピー
今まで， `+` で足し算できる型（ `i32` `f64` など）とそうでない型（ `bool` `char` `Vec<i32>` など）がありました．上で定義した構造体は，今のところ足し算できません．
```rust
Point(1, 2) + Point(3, 4); // エラー
```
`-` `*` `/` `%` といった他の演算もできません．また， `==` `!=` `<` `<=` `>` `>=` による比較もできません．
```rust
Point(1, 2) == Point(1, 2); // エラー
```

今まで， `{}` で出力できる型（ `i32` `char` など）とそうでない型（ `Vec<i32>` や `(i32, f64)` など）がありました．上で定義した構造体は，今のところ `{}` で出力できません．
```rust
println!("{}", Point(1, 2)); // エラー
```
`{:?}` や `{:#?}` でも出力できません．
```rust
println!("{:?}", Point(1, 2)); // エラー
```

今まで，コピー可能な型（ `i32` `(i32, f64)` など）とそうでない型（ `Vec<i32>` など）がありました．上で定義した構造体は，今のところコピー不可能です．
```rust
let point = Point(1, 2);
let moved = point; // ムーブ
println!("{}", point.0); // エラー
```

このように，自分で定義した構造体は，そのままではできないことが多くあります．これらを可能にする方法は，後の章で説明します．
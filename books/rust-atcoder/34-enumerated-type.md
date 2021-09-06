---
title: "列挙型"
---

# 列挙型
**列挙型**は，新しい型を作るもう 1 つの方法です．フィールドのない列挙型と，フィールドのある列挙型があります．

## フィールドのない列挙型
フィールドのない列挙型は，取りうる値を列挙することで作られます．

たとえば，図形の種類を表す `Shape` という型は次のように定義されます．
```rust
enum Shape {
    Triangle, // 三角形
    Rectangle, // 長方形
    Circle, // 円
}
```

こう書くと， `Shape` 型の変数は `Shape::Triangle` `Shape::Rectangle` `Shape::Circle` という 3 通りの値だけを取ることができるようになります．
```rust
enum Shape {
    Triangle,
    Rectangle,
    Circle,
}

fn main() {
    let shape1: Shape = Shape::Triangle;
    let shape2: Shape = Shape::Rectangle;
    let shape3: Shape = Shape::Circle;
}
```

`Shape` 型の変数が取りうる 3 通りの値を表す `Triangle` `Rectangle` `Circle` という名前を**列挙子**または**バリアント**といいます．

列挙型の名前 `Shape` と各列挙子の名前 `Triangle` `Rectangle` `Circle` は，構造体と同じくキャメルケースにします．

## フィールドのある列挙型
それぞれの列挙子は，前の章で登場した構造体のように，フィールドをもつことができます．

たとえば上の `Shape` の例で，列挙子 `Rectangle` にタプル構造体のようなフィールドをもたせるには次のようにします．
```rust
enum Shape {
    Triangle,
    Rectangle(f64, f64),
    Circle,
}
```
こう書くと， `Shape` 型の変数は `Shape::Rectangle` のとき追加で `f64` 型の値を 2 つ持てるようになります．

```rust
enum Shape {
    Triangle,
    Rectangle(f64, f64),
    Circle,
}

fn main() {
    // 2 つのフィールドを指定して様々な Shape::Rectangle が作れる
    let shape1 = Shape::Rectangle(1., 2.5);
    let shape2 = Shape::Rectangle(3.5, 2.);

    // Shape::Triangle と Shape::Circle はフィールドを持てない
    let shape3 = Shape::Triangle;
}
```

問題です． `f64` の取りうる値を $n$ 通りとすると，この `Shape` 型の変数が取りうる値は何通りありますか？
:::details 答え
`Shape::Triangle` が $1$ 通り， `Shape::Rectangle` が $n^2$ 通り， `Shape::Circle` が $1$ 通りなので，全て足して $n^2 + 2$ 通りです．
:::

構造体と同様，フィールドに名前を付けることもできます．

```rust
enum Shape {
    Triangle(f64, f64, f64),
    Rectangle { height: f64, width: f64 },
    Circle { radius: f64 },
}

fn main() {
    // 3 辺の長さがそれぞれ 1.5, 2, 2.5 の三角形
    let shape1 = Shape::Triangle(1.5, 2., 2.5);

    // 縦 1 横 2.5 の長方形
    let shape2 = Shape::Rectangle {
        height: 1.,
        width: 2.5,
    };

    // 半径 2 の円
    let shape3 = Shape::Circle { radius: 2. };
}
```
`Shape::Triangle` のもつ `f64` 型の 3 つのフィールドには名前が付いていません．一方で， `Shape::Rectangle` `Shape::Circle` のフィールドには名前が付いていて， `height` と `width` が長方形の高さと幅， `radius` が円の半径を表すことが分かります．

問題です． `f64` の取りうる値を $n$ 通りとすると，この `Shape` 型の変数が取りうる値は何通りありますか？
:::details 答え
`Shape::Triangle` が $n^3$ 通り， `Shape::Rectangle` が $n^2$ 通り， `Shape::Circle` が $n$ 通りなので，全て足して $n^3 + n^2 + n$ 通りです．
:::

# パターンマッチ

前章の構造体と同様，列挙型はそのままだと `==` 演算子による比較ができません．
```rust:コンパイルエラー
enum Shape {
    Triangle,
    Rectangle,
    Circle,
}

fn fnc(shape: Shape) {
    if shape == Shape::Triangle {
        println!("It's triangle.");
    }
}
```

また，フィールドにそのままアクセスすることもできません．

```rust:コンパイルエラー
enum Shape {
    Triangle(f64, f64, f64),
    Rectangle { height: f64, width: f64 },
    Circle { radius: f64 },
}

fn fnc(shape: Shape) {
    let value = shape.0; // .0 は Shape::Triangle にしかない
    let value = shape.height; // .height は Shape::Rectangle にしかない
}
```

列挙型の変数を使用するときは，代わりにパターンマッチを使います．

```rust
enum Shape {
    Triangle,
    Rectangle,
    Circle,
}

// shape が Shape::Triangle のときに true，
// そうでないときに false を返す関数
fn is_triangle(shape: Shape) -> bool {
    if let Shape::Triangle = shape {
        true
    } else {
        false
    }
}

fn main() {
    assert_eq!(is_triangle(Shape::Triangle), true);
    assert_eq!(is_triangle(Shape::Rectangle), false);
}
```

`Shape::Triangle` は論駁可能なパターンであり，右辺が `Shape::Triangle` のとき成功，そうでないとき失敗します．これを `if let` ではなく `match` を使って書けば

```rust
fn is_triangle(shape: Shape) -> bool {
    match shape {
        Shape::Triangle => true,
        _ => false,
    }
}
```

となります．また， [**`matches!` マクロ**](https://doc.rust-lang.org/std/macro.matches.html)を使うと単に

```rust
fn is_triangle(shape: Shape) -> bool {
    matches!(shape, Shape::Triangle)
}
```

とも書けます． `matches!` マクロは 2 つの引数をとりますが，そのうち 2 つめには論駁可能なパターンを渡します．1 つめの引数（式）を 2 つめの引数（パターン）で受け，マッチが成功すれば `true`，失敗すれば `false` を返します．

パターンを使うと，列挙子のもつフィールドの値を取り出すこともできます．
```rust
enum Shape {
    Triangle(f64, f64, f64),
    Rectangle { height: f64, width: f64 },
    Circle { radius: f64 },
}

// 図形の面積を求める関数
fn area(shape: Shape) -> f64 {
    match shape {
        Shape::Triangle(a, b, c) => {
            let s = (a + b + c) / 2.;
            let squared = s * (s - a) * (s - b) * (s - c);
            squared.sqrt()
        }
        Shape::Rectangle {
            height: h,
            width: w,
        } => h * w,
        Shape::Circle { radius } => radius * radius * std::f64::consts::PI,
    }
}

fn main() {
    // 3 辺の長さがそれぞれ 1.5, 2, 2.5 の三角形は直角三角形なので，
    // 面積は 底辺 × 高さ ÷ 2 に等しい
    let shape1 = Shape::Triangle(1.5, 2., 2.5);
    assert!((area(shape1) - 1.5).abs() < 1e-6);

    // 長方形の面積は 縦 × 横 に等しい
    let shape2 = Shape::Rectangle {
        height: 1.,
        width: 2.5,
    };
    assert!((area(shape2) - 2.5).abs() < 1e-6);

    // 円の面積は 半径の 2 乗 × 円周率 に等しい
    let shape3 = Shape::Circle { radius: 2. };
    assert!((area(shape3) - 12.566371).abs() < 1e-6);
}
```

`area` 関数内の `Shape::Triangle(a, b, c)`，`Shape::Rectangle { height: h, width: w }`，`Shape::Circle { radius }` は全て論駁可能なパターンであり，それぞれ `Shape::Triangle`，`Shape::Rectangle`，`Shape::Circle` のときマッチに成功して，あとは構造体パターンと同じようにフィールドを変数名で受けます．特に最後の `Shape::Circle { radius }` は `Shape::Circle { radius: radius }` と同じ意味です．

また， `match` 式のアームは全てのケースをカバーしている必要がありますが， `Shape` 型の値は必ずこれら 3 つのアームのいずれかに該当するので問題ありません．

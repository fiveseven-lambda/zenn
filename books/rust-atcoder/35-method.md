---
title: "メソッド"
---

# 型に関連付けられた関数
$xy$-平面上のベクトルは，$x$ 成分と $y$ 成分の組で表されます．2 成分をそれぞれ `f64` で表現すると，平面ベクトルを表す構造体は次のようになります：
```rust
struct Vector(f64, f64);
```

平面ベクトル $(x, y)$ が与えられると，その長さ $\sqrt{x^2 + y^2}$ を求めることができます．これを計算する関数は，たとえば次のように定義できます：
```rust
fn length(vector: &Vector) -> f64 {
    let Vector(x, y) = *vector;
    (x * x + y * y).sqrt()
}
```
まず構造体パターン `Vector(x, y)` を用いて `vector.0`，`vector.1` をそれぞれ `x`，`y` に代入した後， $\sqrt{x^2 + y^2}$ を計算して返しています．

引数として `Vector` ではなく参照 `&Vector` を受け取っているのは，`Vector` がコピー不可能な型であり，そのまま渡すとムーブが起こるからです．
:::message
引数でもパターンが使えることと，`x.hypot(y)` で $\sqrt{x^2 + y^2}$ が計算できることを使うと，
```rust
fn length(&Vector(x, y): &Vector) -> f64 {
    x.hypot(y)
}
```
とも書けます．
:::

この `length` は，`Vector` という型に特有の関数です．`Vector` 型の値なくしてその長さを考えることはできません．このようなとき，`length` を単なる関数ではなく `Vector` に関連付けられた関数として定義することができます．これを行うには，構造体の定義とは別に `impl Vector { }` と書いて，その中で `length` 関数を定義します．
```rust
struct Vector(f64, f64);

impl Vector {
    fn length(vector: &Vector) -> f64 {
        let Vector(x, y) = *vector;
        (x * x + y * y).sqrt()
    }
}
```
こうして定義した `length` 関数を呼び出すときは， `length` ではなく `Vector::length` と書きます．
```rust
fn main() {
    let v = Vector(1., 1.);
    let r = Vector::length(&v);
    assert!((r - 1.41421356).abs() < 1e-6);
}
```
こうすることで， `Vector::length` は `Vector` 型の値に対して使われる関数であるということがはっきりします．

自分で定義した構造体・列挙型には，こうして関数を関連付けることができます．一方，`i32` や `(f64, char)` のような型に対して `impl i32 { }` や `impl (f64, char) { }` と書くことはできません．

`impl Vector { }` の中で複数個の関数を定義することもできます．
```rust
struct Vector(f64, f64);

impl Vector {
    fn length(vector: &Vector) -> f64 {
        let Vector(x, y) = *vector;
        (x * x + y * y).sqrt()
    }
    // 逆向きのベクトルを返す関数
    fn inverse(vector: &Vector) -> Vector {
        Vector(-vector.0, -vector.1)
    }
}
```
また，構造体 `Vector` の定義は 1 つしか書けませんが，`impl Vector { }` は離れた場所に複数書くこともできます．
```rust
struct Vector(f64, f64);

impl Vector {
    fn length(vector: &Vector) -> f64 {
        let Vector(x, y) = *vector;
        (x * x + y * y).sqrt()
    }
}

impl Vector {
    fn inverse(vector: &Vector) -> Vector {
        Vector(-vector.0, -vector.1)
    }
}
```
# メソッド
上の `Vector::length` 関数の定義において，第 1 引数の変数名を `vector` から `self` に書き変えます．
```rust
impl Vector {
    fn length(self: &Vector) -> f64 {
        let Vector(x, y) = *self;
        (x * x + y * y).sqrt()
    }
}
```
第 1 引数の名前として `self` を使った関数は `Vector` 上の**メソッド**と呼ばれ，呼び出すとき `Vector::length(&v)` の代わりに `v.length()` と書けるようになります．
```rust
fn main() {
    let v = Vector(1., 1.);
    let r = v.length(); // Vector::length(&v) と同じ意味
    assert!((r - 1.41421356).abs() < 1e-6);
}
```

`Vector::length` 関数において第 1 引数 `self` の型は `&Vector` でした．一方，次の `Vector::into_tuple` 関数では第 1 引数 `self` の型が `Vector` です．このとき `v.into_tuple()` は `Vector::into_tuple(&v)` でなく `Vector::into_tuple(v)` と同じ意味になります．
```rust
impl Vector {
    fn into_tuple(self: Vector) -> (f64, f64) {
        (self.0, self.1)
    }
}

fn main() {
    let v = Vector(1., 2.);
    let tuple = v.into_tuple(); // Vector::into_tuple(v) と同じ意味
    assert!((tuple.0 - 1.).abs() < 1e-6);
    assert!((tuple.1 - 2.).abs() < 1e-6);
    // println!("{}", v.0); // エラー： v は所有権を失っている
}
```
また，次の `Vector::inverse` 関数では第 1 引数 `self` の型が `&mut Vector` です．このとき `v.inverse()` は `Vector::inverse(&mut v)` と同じ意味になります．
```rust
impl Vector {
    fn inverse(self: &mut Vector) {
        self.0 = -self.0;
        self.1 = -self.1;
    }
}

fn main() {
    let mut v = Vector(2., 3.);
    v.inverse(); // Vector::inverse(&mut v) と同じ意味
    // v が書き換わっている
    assert!((v.0 + 2.).abs() < 1e-6);
    assert!((v.1 + 3.).abs() < 1e-6);
}
```

次の `Vector::scale` 関数は第 1 引数 `self` にくわえて第 2 引数 `factor` をとっています．このように 2 つ以上の引数をとるメソッドを `v.scale` の形式で呼び出すとき，丸括弧の中には第 2 引数以降を書きます．すなわち，`v.scale(5.)` と `Vector::scale(&v, 5.)` が同じ意味になります．
```rust
impl Vector {
    fn scale(self: &Vector, factor: f64) -> Vector {
        let Vector(x, y) = *self;
        Vector(factor * x, factor * y)
    }
}

fn main() {
    let v = Vector(2., 3.);
    let scaled = v.scale(5.);
    assert!((scaled.0 - 10.).abs() < 1e-6);
    assert!((scaled.1 - 15.).abs() < 1e-6);
}
```

`self` の型を `Vector` と関係ないもの（たとえば `i32` や `(f64, f64)`）にすることはできません．

## 省略記法
`Vector` 上のメソッドを定義するとき，第 1 引数の `self: Vector` を単に `self` と書くことができます．

```rust
impl Vector {
    fn into_tuple(self) -> (f64, f64) {
        (self.0, self.1)
    }
}
```
同様に， `self: &Vector` を `&self`，`self: &mut Vector` を `&mut self` と書くことができます．
```rust
impl Vector {
    fn length(&self) -> f64 {
        let Vector(x, y) = *self;
        (x * x + y * y).sqrt()
    }
    fn inverse(&mut self) {
        self.0 = -self.0;
        self.1 = -self.1;
    }
}
```

すでに登場したメソッドの例として， [`f64::sqrt(self)`](https://doc.rust-lang.org/stable/std/primitive.f64.html#method.sqrt) や [`f64::sin(self)`](https://doc.rust-lang.org/stable/std/primitive.f64.html#method.sin) などがあります．
# 関連関数
`Vector` 型のゼロベクトルを返す `zero` 関数を考えます．
```rust
impl Vector {
    fn zero() -> Vector {
        Vector(0., 0.)
    }
}
```
これも `Vector` に関連付けて定義されている関数ですが，引数 `self` を取りません．このような関数を**関連関数**といい，呼び出すときは毎回 `Vector::` をつけて `Vector::zero()` と書きます．

すでに登場した関連関数の例として， [`String::new()`](https://doc.rust-lang.org/stable/std/string/struct.String.html#method.new) があります．
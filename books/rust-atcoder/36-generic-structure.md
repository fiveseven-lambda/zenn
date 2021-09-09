---
title: "構造体と列挙型のジェネリクス"
---

# ジェネリック構造体の定義
[第 31 章](https://zenn.dev/toga/books/rust-atcoder/viewer/31-generic-function)で，ジェネリクスを使った関数の定義を説明しました．構造体や列挙型も，同様にジェネリクスを使って定義することができます．

まず，`i32` 型の 2 つの値 $x$，$y$ を用いて平面上の点の座標 $(x, y)$ を表現する構造体を考えます．
```rust
struct PointI32(i32, i32);
```
`i32` 以外にも， `i64` や `f64` など様々な $x$，$y$ の型について同様の構造体が作れます．
```rust
struct PointI64(i64, i64);
struct PointF64(f64, f64);
```

ジェネリクスを用いてこれらをまとめて定義するには，次のように書きます．
```rust
struct Point<T>(T, T);
```
こうすると， `T` を `i32` とした構造体 `Point<i32>`，`T` を `i64` とした構造体 `Point<i64>`，`T` を `f64` とした構造体 `Point<f64>` などが全ての型 `T` について一斉に使えるようになります．

こうして定義した `Point` を使うときは，次のようになります．
```rust
let p1: Point<i32> = Point::<i32>(1, 5);
// let p1: PointI32 = PointI32(1, 5);

let p2: Point<i64> = Point::<i64>(1, 5);
// let p2: PointI64 = PointI64(1, 5);
```
コメントアウトした行は，ジェネリクスで書き換える前の `PointI32` `PointI64` を使った場合の書き方です．このように，単に `Point<i32>` と書いて良い場合と，`Point::<i32>` と書かなければいけない場合があります．後者の `::<>` をターボフィッシュといいます．

ターボフィッシュを用いて `Point::<i32>` と書かなければいけない理由は，`Point<i32>` だと `1 < 2` のような比較に用いる演算子 `<`，`>` と区別できない状況が存在するためです．しかし，型の名前に比較演算子 `<`，`>` が現れることはありません．型注釈の `:` や関数定義における `->` の後には型が来ると分かっているので，ターボフィッシュは必要ありません．

ジェネリクスを使って同時に定義されていても，`Point<T>` と `Point<U>` は（`T` と `U` が違えば）別の型です．よって次はコンパイルエラーになります．
```rust:コンパイルエラー
let p: Point<i32> = Point::<i64>(1, 5);
```

## 型推論
`Point<T>` を使うとき，型パラメータを与える代わりに `_` と書くと，その部分の型を推論させることができます．

```rust
let p: Point<_> = Point::<f64>(1., 5.);
```

`p` に `Point<f64>` 型の値を代入しているので，`p` の型も `Point<f64>` でなければなりません．よって `_` の部分は `f64` に推論されます．

```rust
let p: Point<i64> = Point::<_>(1, 5);
```

今度は `p` の型が `Point<i64>` なので，`=` の右辺も `Point<i64>` でなければなりません．よって `_` の部分は `i64` に推論されます．

関数のジェネリクスと同様，`Point::<_>(1, 5)` は `::<_>` を省略して単に `Point(1, 5)` と書けます．
```rust
let p: Point<i64> = Point(1, 5);
```

問題．型注釈 `: Point<i64>` を取り去って，次のように書いたら `p` の型は何になりますか？
```rust
let p = Point(1, 5);
```
:::details 答え
型を決める情報が他に無いとき整数リテラルは自動で `i32` になるので， `Point<i32>` です．
:::

:::message
型注釈 `: Point<_>` の `<_>` は省略できません．
```rust:コンパイルエラー
let p: Point = Point::<i32>(1, 5);
```
:::

# `impl`
ジェネリクスを使って `Point<i32>` `Point<i64>` `Point<f64>` ……などを一斉に定義したとき，これらに対し個別にメソッドや関連関数を実装することができます．たとえば，
```rust
struct Point<T>(T, T);

impl Point<i32> {
    fn abscissa(&self) -> &i32 {
        &self.0
    }
}
```
と書くと，`Point<i32>` 型の変数 `p` に対し `p.abscissa()` と書いて $x$ 座標を取り出すことができるようになります．
```rust
let p = Point(5, 2);
assert_eq!(*p.abscissa(), 5);
```

この場合 `p.abscissa()` は `Point::<i32>::abscissa(&p)` と同じ意味です（これも `<i32>` ではなく `::<i32>` と書く必要があります．ただしこの `i32` は推論できるので `Point::abscissa(&p)` でも構いません）．

`abscissa()` は `Point<i32>` にしか実装されていないため，たとえば `Point<i64>` 型の値に対して `abscissa()` を呼び出すことはできません．
```rust:コンパイルエラー
Point::<i64>(5, 2).abscissa();
```
`impl Point<i32> { }` だけでなく `impl Point<i64> { }` の中でも `abscissa()` を定義すれば，`Point<i64>` 型の値に対しても `abscissa()` を呼び出すことができるようになります．

```rust
struct Point<T>(T, T);

impl Point<i32> {
    fn abscissa(&self) -> &i32 {
        &self.0
    }
}
impl Point<i64> {
    fn abscissa(&self) -> &i64 {
        &self.0
    }
}

fn main() {
    assert_eq!(*Point::<i32>(5, 2).abscissa(), 5);
    assert_eq!(*Point::<i64>(5, 2).abscissa(), 5);
}
```

同様に `impl Point<f32> { }` や `impl Point<f64> { }` の中でも `abscissa()` を定義すれば，`Point<f32>` 型や `Point<f64>` 型の値に対しても `abscissa()` が呼べるようになります．

ここで `impl` 自体に `<T>` を付けると，これらの `impl` を一度に行うことができます．
```rust
struct Point<T>(T, T);

impl<T> Point<T> {
    fn abscissa(&self) -> &T {
        &self.0
    }
}

fn main() {
    assert_eq!(*Point::<i32>(5, 2).abscissa(), 5);
    assert_eq!(*Point::<i64>(5, 2).abscissa(), 5);
}
```
`impl` が `impl<T>` に変わり，`impl Point<i32> { }` や `impl Point<i64> { }` において `i32` や `i64` だった部分が全て `T` に置き換わっています．こうすることで， `T` が `i32` になったときの `impl`， `T` が `i64` になったときの `impl`， `T` が `f64` になったときの `impl` などが全ての型 `T` について一斉に実装されます．そのため，`Point<i32>` 型の値に対しても `Point<i64>` 型の値に対しても同じように `abscissa()` が呼べています．

:::message
標準ライブラリの [Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html) 構造体も，ジェネリクスを用いて定義されています．`Vec<i32>` は型パラメータとして `i32` を与えたものであり，`Vec::<i32>::new()` はその関連関数です． `vec.push(10)` や `vec.pop()` はメソッドの呼び出し `Vec::<i32>::push(&mut v, 10)`，`Vec::<i32>::pop(&mut v)` に他なりません．
:::
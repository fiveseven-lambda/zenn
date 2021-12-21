---
title: "Rust で，行と列を転置する"
emoji: "🔄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "競技プログラミング"]
published: true
---

:::message alert
この記事中のコードより[こっち](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&code=fn%20main()%20%7B%0A%20%20%20%20let%20matrix%20%3D%20vec!%5Bvec!%5B1%2C%202%2C%203%5D%2C%20vec!%5B4%2C%205%2C%206%5D%2C%20vec!%5B7%2C%208%2C%209%5D%5D%3B%0A%20%20%20%20let%20result%3A%20Vec%3C_%3E%20%3D%20matrix.into_iter().transpose().collect()%3B%0A%20%20%20%20assert_eq!(result%2C%20vec!%5Bvec!%5B1%2C%204%2C%207%5D%2C%20vec!%5B2%2C%205%2C%208%5D%2C%20vec!%5B3%2C%206%2C%209%5D%5D)%3B%0A%7D%0A%0Atrait%20Transpose%3CIter%3A%20IntoIterator%3E%20%7B%0A%20%20%20%20fn%20transpose(self)%20-%3E%20Transposed%3CIter%3E%3B%0A%7D%0A%0Aimpl%3CT%3E%20Transpose%3CT%3A%3AItem%3E%20for%20T%0Awhere%0A%20%20%20%20T%3A%20IntoIterator%2C%0A%20%20%20%20T%3A%3AItem%3A%20IntoIterator%2C%0A%7B%0A%20%20%20%20fn%20transpose(self)%20-%3E%20Transposed%3CT%3A%3AItem%3E%20%7B%0A%20%20%20%20%20%20%20%20Transposed(self.into_iter().map(IntoIterator%3A%3Ainto_iter).collect())%0A%20%20%20%20%7D%0A%7D%0A%0Astruct%20Transposed%3CIter%3A%20IntoIterator%3E(Vec%3CIter%3A%3AIntoIter%3E)%3B%0A%0Aimpl%3CIter%3A%20IntoIterator%3E%20Iterator%20for%20Transposed%3CIter%3E%20%7B%0A%20%20%20%20type%20Item%20%3D%20Vec%3CIter%3A%3AItem%3E%3B%0A%20%20%20%20fn%20next(%26mut%20self)%20-%3E%20Option%3CSelf%3A%3AItem%3E%20%7B%0A%20%20%20%20%20%20%20%20self.0.iter_mut().map(Iterator%3A%3Anext).collect()%0A%20%20%20%20%7D%0A%7D%0A)の方がすっきりしていて普通です．いずれ直します．
:::

# やりたいこと

```
[[1, 2, 3],
 [4, 5, 6],
 [7, 8, 9]]
```
という 2 次元ベクタがあったときに，これを転置して
```
[[1, 4, 7],
 [2, 5, 8],
 [3, 6, 9]]
```
にする `transpose` 関数を作りたいと思います． `Rust` はイテレータが扱いやすいので，転置したもののイテレータが得られるようにします．使い方としては，次のようになります．

```rust
fn main() {
    let matrix = vec![vec![1, 2, 3], vec![4, 5, 6], vec![7, 8, 9]];

    let result: Vec<_> = matrix.iter().transpose().collect();

    assert_eq!(
        result,
        vec![vec![&1, &4, &7], vec![&2, &5, &8], vec![&3, &6, &9]]
    );
}
```

`transpose` 関数はベクタへのイテレータを返すようにします．たとえば今回なら，返り値は `Iterator<Item = Vec<&i32>>` を impl しており，それを `.collect()` した `result` の型は `Vec<Vec<&i32>>` となっています．

# 発想
返り値を構造体 `Transposed` として， `impl Iterator for Transposed` することにします．

`Transposed` は `matrix[0].iter()`, `matrix[1].iter()`, `matrix[2].iter()`, ……をベクタとして持っておき， `.next()` が呼び出されるたびに各イテレータを一つずつ進めます．

よって `Transposed` のもつメンバはイテレータを要素にもつベクタ 1 つです．これの名前を `iters` とします．

# 境界
各 `matrix[i]` についてイテレータが呼び出せればよいです． `&Vec<T>` は `IntoIterator<Item = &T>` を impl しているので，これを利用します．

よって， `&matrix[0]` `&matrix[1]` `&matrix[2]` ……が得られれば，各々に `IntoIterator::into_iter` を適用することでベクタ `iters` を作ることができます．

また， `&Vec<Vec<T>>` は `IntoIterator<Item = &Vec<T>>` を impl しているので， `&matrix` に対し `IntoIterator::into_iter` を呼び出せば `&matrix[0]` `&matrix[1]` `&matrix[2]` ……が得られます．

よって， `Iter: IntoIterator<Item = &Elem>` ， `T: IntoIterator<Item = Iter>` として `T` に対して `transpose` 関数を呼び出せるように設計します．

# 実装
## トレイト
`x.transpose()` の形式で呼び出したいので， `trait Transpose` を定義します．
```rust
trait Transpose<'a, Elem, Iter, T>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
    T: IntoIterator<Item = Iter>,
{
    fn transpose(self) /* -> Transposed 構造体 */;
}

impl<'a, Elem, Iter, T> Transpose<'a, Elem, Iter, T> for T
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
    T: IntoIterator<Item = Iter>,
{
    fn transpose(self) /* -> Transposed 構造体 */ {
        todo!();
    }
}
```

`IntoIterator::into_iter` 関数の返り値は `IntoIterator::IntoIter` です． `Transposed` 構造体は， `IntoIterator::IntoIter` を要素にもつベクタ `iters` をメンバとしてもつので， `Transposed` は型パラメータとして `Elem` と `Iter: IntoIterator<Item = &Elem>` をとります．
```rust
struct Transposed<'a, Elem, Iter>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
{
    iters: Vec<Iter::IntoIter>,
}
```

よって，先ほど書かなかった `transpose` 関数の返り値の型は， `Transposed<'a, Elem, Iter>` となります．

そして，この `Transposed<'a, Elem, Iter>` に `Iterator<Item = Vec<&'a Elem>>` を impl します．
```rust
impl<'a, Elem, Iter> Iterator for Transposed<'a, Elem, Iter>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
{
    type Item = Vec<&'a Elem>;
    fn next(&mut self) -> Option<Self::Item> {
	todo!();
    }
}
```
## 関数の中身
実装する関数は， `T` 上の `Transpose::transpose` と， `Transposed` 上の `Iterator::next` の 2 つです．

`transpose` 関数は， `T` に対し `into_iter()` を呼び出し， `IntoIterator::into_iter` を `map` した上で `Vec<Iter::IntoIter>` に `collect` します．返り値は，これを `iters` メンバにもつ `Transposed` です．よって次のようになります．
```rust
impl<'a, Elem, Iter, T> Transpose<'a, Elem, Iter, T> for T
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
    T: IntoIterator<Item = Iter>,
{
    fn transpose(self) -> Transposed<'a, Elem, Iter> {
        Transposed {
            iters: self.into_iter().map(IntoIterator::into_iter).collect(),
        }
    }
}
```

`next` 関数は，まず `iters` の各要素に対し `Iterator::next` を `map` します．すると， `Iterator::next` が `Option<&Elem>` を返すので，これを `Option<Vec<&Elem>>` に `collect` します（ `V: FromIterator<A>` なる `V` と `A` について， `FromIterator<Option<A>> for Option<V>` が impl されています．今回は `A` を `&Elem` ， `V` を `Vec<&Elem>` とした形です）．よって次のようになります．
```rust
impl<'a, Elem, Iter> Iterator for Transposed<'a, Elem, Iter>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
{
    type Item = Vec<&'a Elem>;
    fn next(&mut self) -> Option<Self::Item> {
        self.iters.iter_mut().map(Iterator::next).collect()
    }
}
```
# 全体
全体では，次のようになります．
```rust
fn main() {
    let matrix = vec![vec![1, 2, 3], vec![4, 5, 6], vec![7, 8, 9]];

    let result: Vec<_> = matrix.iter().transpose().collect();

    assert_eq!(
        result,
        vec![vec![&1, &4, &7], vec![&2, &5, &8], vec![&3, &6, &9]]
    );
}

trait Transpose<'a, Elem, Iter, T>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
    T: IntoIterator<Item = Iter>,
{
    fn transpose(self) -> Transposed<'a, Elem, Iter>;
}

impl<'a, Elem, Iter, T> Transpose<'a, Elem, Iter, T> for T
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
    T: IntoIterator<Item = Iter>,
{
    fn transpose(self) -> Transposed<'a, Elem, Iter> {
        Transposed {
            iters: self.into_iter().map(IntoIterator::into_iter).collect(),
        }
    }
}

struct Transposed<'a, Elem, Iter>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
{
    iters: Vec<Iter::IntoIter>,
}

impl<'a, Elem, Iter> Iterator for Transposed<'a, Elem, Iter>
where
    Elem: 'a,
    Iter: IntoIterator<Item = &'a Elem>,
{
    type Item = Vec<&'a Elem>;
    fn next(&mut self) -> Option<Self::Item> {
        self.iters.iter_mut().map(Iterator::next).collect()
    }
}
```
[Playground のリンク](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=45a599e006505df85317fd43bfd97388)．

また，これを使うと [ABC182 E - Akari](https://atcoder.jp/contests/abc182/tasks/abc182_e) が次のように解けます： [AC コード](https://atcoder.jp/contests/abc182/submissions/20202805)．

以上，行と列の転置でした．この記事が誰かの役に立てば幸いです．

---
title: "Rust で抽象化したグラフを扱う"
emoji: "🪩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: false
---

メモ程度の軽いやつです．

# 抽象化
有向グラフの隣接リストの「与えられた頂点から直接辺が伸びている頂点を列挙できる」という機能をトレイトとして抜き出し，辺を陽に持たないグリッドグラフなどにも impl したいねという話です．

まずコード全部出したあと下で説明します
```rust
mod graph {
    use std::{iter, slice};

    pub trait Graph<'a> {
        type Node;
        type Iter: Iterator<Item = Self::Node>;
        fn next(&'a self, _: Self::Node) -> Self::Iter;
    }

    pub struct AdjList(Vec<Vec<usize>>);
    impl AdjList {
        pub fn new(n: usize) -> AdjList {
            AdjList(vec![Vec::new(); n])
        }
        pub fn add_directed(&mut self, from: usize, to: usize) {
            self.0[from].push(to);
        }
        pub fn add_undirected(&mut self, u: usize, v: usize) {
            self.add_directed(u, v);
            self.add_directed(v, u);
        }
    }
    impl<'a> Graph<'a> for AdjList {
        type Node = usize;
        type Iter = iter::Cloned<slice::Iter<'a, usize>>;
        fn next(&'a self, node: usize) -> Self::Iter {
            self.0[node].iter().cloned()
        }
    }

    pub struct Grid4 {
        height: usize,
        width: usize,
    }
    impl Grid4 {
        pub fn new(height: usize, width: usize) -> Grid4 {
            Grid4 { height, width }
        }
    }
    pub const DIR4: [(usize, usize); 4] = [(1, 0), (0, 1), (!0, 0), (0, !0)];
    pub struct Grid4Iter<'a> {
        r: usize,
        c: usize,
        grid: &'a Grid4,
        iter: std::slice::Iter<'static, (usize, usize)>,
    }
    impl<'a> Iterator for Grid4Iter<'a> {
        type Item = (usize, usize);
        fn next(&mut self) -> Option<Self::Item> {
            for &(i, j) in &mut self.iter {
                let r = self.r.wrapping_add(i);
                let c = self.c.wrapping_add(j);
                if r < self.grid.height && c < self.grid.width {
                    return Some((r, c));
                }
            }
            None
        }
    }
    impl<'a> Graph<'a> for Grid4 {
        type Node = (usize, usize);
        type Iter = Grid4Iter<'a>;
        fn next(&'a self, (r, c): (usize, usize)) -> Grid4Iter<'a> {
            Grid4Iter {
                r,
                c,
                grid: self,
                iter: DIR4.iter(),
            }
        }
    }
}
```

# トレイト
重みなしの場合
```rust
pub trait Graph<'a> {
    type Node;
    type Iter: Iterator<Item = Self::Node>;
    fn next(&'a self, _: Self::Node) -> Self::Iter;
}
```
`Node` は頂点を表す型です．隣接リストなら `usize`，グリッドなら `(usize, usize)` みたいな． `next` が次の頂点を列挙する関数です．イテレータを返すので，その型も `Iter` として与えます．

# 隣接リスト
これを隣接リスト
```rust
pub struct AdjList(Vec<Vec<usize>>);
```
に impl すると，
```rust
use std::{iter, slice};

impl<'a> Graph<'a> for AdjList {
    type Node = usize;
    type Iter = iter::Cloned<slice::Iter<'a, usize>>;
    fn next(&'a self, node: usize) -> Self::Iter {
        self.0[node].iter().cloned()
    }
}
```
となります．

# グリッド
4 方向のグリッドグラフ
```rust
pub struct Grid4 {
    height: usize,
    width: usize,
}
```
に impl する場合，
```rust
pub const DIR4: [(usize, usize); 4] = [(1, 0), (0, 1), (!0, 0), (0, !0)];
pub struct Grid4Iter<'a> {
    r: usize,
    c: usize,
    grid: &'a Grid4,
    iter: std::slice::Iter<'static, (usize, usize)>,
}
```
を使って
```rust
impl<'a> Graph<'a> for Grid4 {
    type Node = (usize, usize);
    type Iter = Grid4Iter<'a>;
    fn next(&'a self, (r, c): (usize, usize)) -> Grid4Iter<'a> {
        Grid4Iter {
            r,
            c,
            grid: self,
            iter: DIR4.iter(),
        }
    }
}

impl<'a> Iterator for Grid4Iter<'a> {
    type Item = (usize, usize);
    fn next(&mut self) -> Option<Self::Item> {
        for &(i, j) in &mut self.iter {
            let r = self.r.wrapping_add(i);
            let c = self.c.wrapping_add(j);
            if r < self.grid.height && c < self.grid.width {
                return Some((r, c));
            }
        }
        None
    }
}
```
といった感じかな……

`DIR4` を `pub` にしているのは，何かに使うかも〜という気持ちです．

わざわざ `Grid4Iter` に `<'a>` を付けて中で `&'a Grid4` を持ってるのは， `Grid4Iter` のライフタイムが元の `Grid4` を超えるようなコードはまずそうな感じがするからです．
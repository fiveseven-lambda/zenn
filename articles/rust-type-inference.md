---
title: "Rust で型推論を高速かつ安全に実装"
emoji: "🦀"
type: "tech"
topics: ["rust", "型推論"]
published: true
---

簡単、高速、安全。Rc / RefCell の威力を知ってほしい。

# データ型の定義

## 型

まずは普通に型 $T$ を定義します。たとえば、ブーリアン型 $\rm Bool$ と関数型 $T \to T$ だけの型システムなら、以下のようになります。

```rust
enum Ty {
    Bool,
    Func(Rc<Ty>, Rc<Ty>),
}
```

Box ではなく Rc を使っています：これから型推論を実装しますが、型 $T$ を型変数 $\alpha$ に代入するとき、参照カウントをインクリメントするだけで $T$ を clone できます。

## 型変数

次に、型変数 $\alpha, \beta, \ldots$ をこれにくわえます。型変数は「まだ何も代入されていない」or「型 $T$ が代入されている」のどちらかなので、これをそのまま enum で表します。

```rust
enum Var {
    Assigned(Rc<Ty>),
    Unassigned,
}
```

最後に、型変数 $\alpha, \beta, \ldots$ を元の型 $T$ と同列に扱うため、`Ty` の列挙子に `Var` を追加します。

```rust
enum Ty {
    Bool,
    Func(Rc<Ty>, Rc<Ty>),
    Var(RefCell<Var>), // 型変数Varを追加
}
```

解 $\alpha = T$ が得られたら、$\alpha$ を `Var::Unassigned` から `Var::Assigned(`$T$`)` に書き換える必要があるので、可変性のために `RefCell` を使っています。

定義は以上です。以下、実装です。

# 型推論

## Contains の実装

たとえば $T = \alpha \to \alpha$ のとき、${\rm unify}(\alpha, T)$ の解を $\alpha = T$ としてしまうと、$\alpha$ が無限再帰する型になってしまいます。これを防ぐために、$T$ が内部に $\alpha$ を含むことを判定する関数が必要です。

```rust
impl Ty {
    // self が内部に var を含むなら、true を返す
    fn contains(self: &Rc<Ty>, var: &RefCell<Var>) -> bool {
        todo!()
    }
}
```

`self` ($T$) の内部を再帰的に見て、`var` ($\alpha$) を探します。まず $T = \rm Bool$ と $T = T_{\rm param} \to T_{\rm ret}$ の場合について書くと以下のようになります。

```rust
    fn contains(self: &Rc<Ty>, var: &RefCell<Var>) -> bool {
        match self.as_ref() {
            Ty::Bool => false,
            Ty::Func(param, ret) => param.contains(var) || ret.contains(var),
            Ty::Var(self_var) => todo!(),
        }
    }
```

[AsRef](https://doc.rust-lang.org/stable/std/convert/trait.AsRef.html) を使って `self.as_ref()` と書いていますが、記号が増えるのが嫌じゃなければ別に `&**self` でいいし、use が増えるのが嫌じゃなければ [Deref](https://doc.rust-lang.org/stable/std/ops/trait.Deref.html) でもいいです^[[ドキュメント](https://doc.rust-lang.org/stable/std/convert/trait.AsRef.html#generic-implementations)にも、Deref coercion で済むのに AsRef を使うなと書いてあります。でもなんか `&**` って見栄えがなあ……]。

残るは $T$ が型変数 `self_var`、つまり $T = \beta$ の場合です。もし `self_var` が `Var::Assigned(`$T'$`)`、つまり $\beta = T'$ なら、$T'$ の内部をさらに探索します。一方、`self_var` が `Var::Unassigned` なら、$\alpha$ と $\beta$ を比較し、同一の型変数か確かめます。型は `&RefCell<Var>` なので、参照のまま `ptr::eq` に突っ込んでもいいし、`RefCell::as_ptr` で `*mut Var` に変換してもよいです。

```rust
        match self.as_ref() {
            // ...
            Ty::Var(self_var) => match *self_var.borrow() {
                Var::Assigned(ref ty) => ty.contains(var),
                Var::Unassigned => self_var.as_ptr() == var.as_ptr(),
            },
        }
```           

従って、contains 全体では以下のようになります。

```rust
impl Ty {
    fn contains(self: &Rc<Ty>, var: &RefCell<Var>) -> bool {
        match self.as_ref() {
            Ty::Bool => false,
            Ty::Func(param, ret) => param.contains(var) || ret.contains(var),
            Ty::Var(self_var) => match *self_var.borrow() {
                Var::Assigned(ref ty) => ty.contains(var),
                Var::Unassigned => self_var.as_ptr() == var.as_ptr(),
            },
        }
    }
}
```

## Unify の実装

いよいよ unify です。2 つの型 $T_{\rm left}$ と $T_{\rm right}$ を受け取って、$T_{\rm left} = T_{\rm right}$ を解きます。成功すれば true、途中で失敗すれば false を返します^[途中で失敗しても、それまでの代入の結果は残ります。たとえば $\alpha \to {\rm Bool}$ と $\beta \to ({\rm Bool} \to {\rm Bool})$ を unify したら、${\rm Bool} \ne {\rm Bool} \to {\rm Bool}$ なので失敗しますが、代入 $\alpha = \beta$ は実行されます。これを防ぎたかったら、計算量 $O(\alpha(n))$ を諦めて [rollback](https://nyaannyaan.github.io/library/data-structure/rollback-union-find.hpp.html) を実装します。それでも $O(\log n)$ なので困ることはありません。]。

```rust
fn unify(left: &Rc<Ty>, right: &Rc<Ty>) -> bool {
    todo!()
}
```

まず、Bool 同士の場合、Func 同士の場合について書くと以下のようになります。

```rust
fn unify(left: &Rc<Ty>, right: &Rc<Ty>) -> bool {
    match (left.as_ref(), right.as_ref()) {
        (Ty::Bool, Ty::Bool) => true,
        (Ty::Func(left_param, left_ret), Ty::Func(right_param, right_ret)) => {
            unify(left_param, right_param) && unify(left_ret, right_ret)
        }
        (Ty::Var(left_var), Ty::Var(right_var)) => todo!(),
        (Ty::Var(left_var), _) => todo!(),
        (_, Ty::Var(right_var)) => todo!(),
        _ => false,
    }
}
```

次に Var 同士、つまり $T_{\rm left} = \alpha$, $T_{\rm right} = \beta$ の場合です。まず、$\alpha = T_{\rm left}'$ なら ${\rm unify}(T_{\rm left}', T_{\rm right})$ をします。同様に、$\beta = T_{\rm right}'$ なら ${\rm unify}(T_{\rm left}, T_{\rm right}')$ をします。

```rust
        (Ty::Var(left_var), Ty::Var(right_var)) => {
            if let Var::Assigned(ref left) = *left_var.borrow() {
                return unify(left, right)
            }
            if let Var::Assigned(ref right) = *right_var.borrow() {
                return unify(left, right)
            }
            todo!()
        }
```

あとは、無限再帰する型が生じないように気を付けながら、代入 $\alpha = \beta$ を実行します。

```rust
        (Ty::Var(left_var), Ty::Var(right_var)) => {
            // ...
            if left_var.as_ptr() != right_var.as_ptr() {
                *left_var.borrow_mut() = Var::Assigned(right.clone());
            }
            true
        }
```

$T_{\rm left}$ と $T_{\rm right}$ の片方だけが型変数の場合も、同じように処理します。上で実装した contains を使って、無限再帰する型が生じないようにします。

```rust
        (Ty::Var(left_var), _) => {
            if let Var::Assigned(ref left) = *left_var.borrow() {
                unify(left, right)
            } else if right.contains(left_var) {
                false
            } else {
                *left_var.borrow_mut() = Var::Assigned(right.clone());
                true
            }
        }
        (_, Ty::Var(right_var)) => {
            // 同様
        }
```

if let の右辺で `left_var.borrow()` を呼んで [Ref](https://doc.rust-lang.org/stable/std/cell/struct.Ref.html) を得ていますが、パターンマッチが失敗すると、else 節が始まる前に捨てられるようです。でないと、`left_var.borrow_mut()` でエラーになります。実際、Edition 2024 では動きますが、Edition 2021 に変えると `RefCell already borrowed` と言われて panic します。

何はともあれ、unify は実装できました。

# 計算量の改善

上の unify は、どんなケースで遅くなるでしょうか。問題は、$T_{\rm left} = \alpha$ と $T_{\rm right} = \beta$ が両方 `Var::Unassigned` のときに

```rust
*left_var.borrow_mut() = Var::Assigned(right.clone());
```

としている点です。つまり、$n$ 個の型変数 $\alpha_1, \alpha_2, \ldots, \alpha_n$ を用意して、${\rm unify}(\alpha_1, \alpha_2), {\rm unify}(\alpha_2, \alpha_3), \ldots, {\rm unify}(\alpha_{n - 1}, \alpha_n)$ をこの順に実行すると、それ以降 $\alpha_i$ を参照するだけで $\alpha_n$ に辿り着くまでに $n - i$ 回の deref が必要になります。

これを防ぐには、まだ何も代入されていない各変数 $\alpha$ が「$\alpha$ に辿り着くまでに最大何回の deref が必要か」知っていればよいです。たとえば今の例で $\alpha_n$ なら、$\alpha_n$ に辿り着くまでに必要な deref の回数の最大値は、$\alpha_1$ から始めた場合の $n - 1$ 回、といった具合です。

これを `Var::Unassigned` に持たせます。

```rust
enum Var {
    Assigned(Rc<Ty>),
    Unassigned(u32),
}
```

そして、$T_{\rm left} = \alpha$ と $T_{\rm right} = \beta$ が両方 `Var::Unassigned` のときにこの値を比べます。もし $\beta$ の方が大きければ、$\alpha = \beta$ つまり

```rust
*left_var.borrow_mut() = Var::Assigned(right.clone());
```

という代入を行っても $\beta$ の値は大きくなりません。しかし、もし $\alpha$ の方が大きければ、これだと $\beta$ の値が 1 増えてしまうので、左右を入れ替えて $\beta = \alpha$ つまり

```rust
*right_var.borrow_mut() = Var::Assigned(left.clone());
```

にします。もし $\alpha$ も $\beta$ も同じ値なら、どちらの順番だろうと 1 増えるので、好きな方を選びます。

```rust
*left_var.borrow_mut() = Var::Assigned(right.clone());
*right_var.borrow_mut() = Var::Unassigned(/* 1 増やした値 */);
```

こうすれば、$n$ 個の型変数をどのように unify した後でも、参照に必要な deref の回数は $O(\log n)$ になります。

変更後の unify はこうなります：

```rust
fn unify(left: &Rc<Ty>, right: &Rc<Ty>) -> bool {
    match (left.as_ref(), right.as_ref()) {
        // ...
        (Ty::Var(left_var), Ty::Var(right_var)) => {
            let left_rank = match *left_var.borrow() {
                Var::Assigned(ref left) => return unify(left, right),
                Var::Unassigned(left_rank) => left_rank,
            };
            let right_rank = match *right_var.borrow() {
                Var::Assigned(ref right) => return unify(left, right),
                Var::Unassigned(right_rank) => right_rank,
            };
            if left_var.as_ptr() != right_var.as_ptr() {
                match left_rank.cmp(&right_rank) {
                    Ordering::Greater => *right_var.borrow_mut() = Var::Assigned(left.clone()),
                    Ordering::Less => *left_var.borrow_mut() = Var::Assigned(right.clone()),
                    Ordering::Equal => {
                        *left_var.borrow_mut() = Var::Assigned(right.clone());
                        *right_var.borrow_mut() = Var::Unassigned(right_rank + 1);
                    }
                }
            }
            true
        }
        // ...
    }
}
```

ここまで難しそうなことをしていましたが、改めて見ると何だかすっきりしています。Rc と RefCell が分かれているおかげで、`.borrow()` `.borrow_mut()` を繰り返すこんなコードも綺麗に書けるわけですね。

以上、Rust で型推論を実装する話でした。実用的にはまだ足りない部分も多いですが、面白いな〜と思ってくれたら幸いです。

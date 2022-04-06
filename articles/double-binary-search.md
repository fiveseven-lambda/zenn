---
title: "double の二分探索はループ 64 回で十分すぎる"
emoji: "🔪"
type: "tech"
topics: ['二分探索', '競プロ']
published: false
---

この記事では，「非 NaN 値」という言葉を「実数，正の無限大，負の無限大のうち，倍精度浮動小数点数で表せる数」という意味で使います．

# 序
倍精度浮動小数点数（C/C++ だと `double`，Rust だと `f64`）に対する以下のような二分探索を考えます．

- `double` を受け取って `bool` を返す関数 `f` が与えられる．
- ある非 NaN 値 `x` が存在して，任意の非 NaN 値 `y` について
  - `y <= x` なら `f(y)` は `true`
  - `y > x` なら `f(y)` は `false`

  であることが分かっている．
- 上の `x`（`f(x)` が `true` となる最大の `x`）を求めたい．

競プロではこれをよく相対誤差 $10^{-6}$ 以下とかで求めますね．誤差が○○以下になるには何回くらいループを回せばいいんだろう？みたいな話にもなります．たとえば[こことか](https://rsk0315.hatenablog.com/entry/2020/04/29/155009)．

でもよく考えると，`double` って 64 ビットです．ということは表せる数は多くても $2^{64}$ 通り（実際は NaN とか非正規化数のぶんもっと少ないです）で，二分探索だとこれが毎回のループで半分ずつ減っていくので，64 回あれば `double` そのものの限界まで誤差が減らせるはずですね．

# じゃあなんで普通にやるとできないの
普通は，下限 $l$ と上限 $r$ があったら $m = (l + r) / 2$ で分割しますね．でも，たとえば $1$ 以上 $32$ 未満の範囲に `double` 値は $5 \times 2^{52}$ 通りありますが，そのうち $80\%$ の $4 \times 2^{52}$ 個が $1$ 以上 $16$ 未満の範囲にあります．つまり，半分ずつに分けていくのが二分探索だったのに，$[1, 32)$ を $[1, 16)$ と $[16, 32)$ に分けても，その中にある `double` 値の数で見ると半分になっていなかったわけですね．

# どうしよう
符号なし 64 ビット整数（C/C++ だと `std::uint64_t`，Rust だと `u64`．競プロ C/C++ では `unsigned long long` も）を使います．

`double` と `std::uint64_t` の間で，以下のような変換を行います．
```cpp:C++
std::uint64_t f2u(double x){
    std::uint64_t b;
    std::memcpy(&b, &x, 8);
    return b >> 63 ? ~b : b ^ 1ull << 63;
}

double u2f(std::uint64_t b){
    b = b >> 63 ? b ^ 1ull << 63 : ~b;
    double ret;
    std::memcpy(&ret, &b, 8);
    return ret;
}
```
```rust:Rust
fn f2u(f: f64) -> u64 {
    let b = f.to_bits();
    if b >> 63 == 1 { !b } else { b ^ 1 << 63 }
}
 
fn u2f(b: u64) -> f64 {
    f64::from_bits(if b >> 63 == 1 { b ^ 1 << 63 } else { !b })
}
```
この変換 `f2u`，`u2f` は以下のような性質を満たします．

- 逆変換になっている：任意の非 NaN 値 `x` について，`u2f(f2u(x))` は `x` に等しい．
- 順序を保つ：任意の非 NaN 値 `x`，`y` について，`x <= y` ならば `f2u(x) <= f2u(y)`．
- 間に非 NaN 値以外を挟まない：`x`，`y` が非 NaN 値のとき，`f2u(x)` 以上 `f2u(y)` 以下の任意の整数 `b` に対し `u2f(b)` は非 NaN 値．

この性質のおかげで，`double` の二分探索は 64 ビット整数の二分探索へと成り下がります．あとは普通の二分探索に突っ込めば，多くても 64 回で `double` の限界までいける二分探索の完成です．
```cpp
// 普通の（整数に対する）二分探索
template <class F>
std::uint64_t binary_search(F f, std::uint64_t l, std::uint64_t r){
    while(l + 1 < r){
        std::uint64_t mid = l + (r - l) / 2;
        (f(mid) ? l : r) = mid;
    }
    return l;
}

// double 二分探索
template <class F>
double binary_search_double(F f, double l = -INFINITY, double r = INFINITY){
    return i2f(
        binary_search(
            [&](std::uint64_t b){ return f(i2f(b)); },
            f2i(l), f2i(r)
        )
    );
}
```
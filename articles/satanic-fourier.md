---
title: "ビット演算を用いた非再帰 FFT"
emoji: "🔀"
type: "tech"
topics: ["競プロ", "FFT"]
published: true
---

# 離散フーリエ変換（ DFT ）の定義
$n$ 個の複素数列 $\bm{x} = (x_0, x_1, \ldots, x_{n - 1})$ に対し，これらの**離散フーリエ変換**（ DFT ） $\bm{X} = (X_0, X_1, \ldots, X_{n - 1})$ は次で定義されます．

$$
X_k = \sum_{j = 0}^{n - 1} \zeta_n^{-jk} x_j
\tag{1}
$$

ただし $\zeta_n$ は

$$
\zeta_n = e^{\frac{2\pi i}{n}} = \cos\frac{2\pi}{n} + i\sin\frac{2\pi}{n}
$$

で定義され， $n$ 乗すると $1$ になる虚数の 1 つです．

$\zeta_{2n}$ の $2a$ 乗を計算すると， $\zeta_{2n}^{2a} = e^{\frac{2\pi i}{2n}\cdot 2a} = e^{\frac{2\pi i}{n} \cdot a}$ となって， $\zeta_n$ の $a$ 乗 $\zeta_n^a$ と同じになります．

また， DFT の定義式 $(1)$ は，整数 $k$ が $0 \leq k < n$ の範囲になくても計算することができます． $\zeta_n^{a + n} = \zeta_n^a \cdot \zeta_n^n = \zeta_n^a$ に注意すると， $X_{k + n} = X_k$ が成り立つことが分かります．よって， $k$ が $0 \leq k < n$ の範囲を超えたときは， $k$ の代わりに， $k$ を $n$ で割った余りで考えても問題ありません．

# 時間間引き（ DIT ）
$n$ が偶数であると仮定し， $n = 2m$ とおきます．このとき，長さ $m$ の数列の DFT を用いて長さ $n$ の数列の DFT を表すことを考えます．

まず，定義 $(1)$ より長さ $n$ の DFT は

$$
\begin{aligned}
X_k &= \sum_{j = 0}^{n - 1} \zeta_n^{-jk} x_j \\
&= x_0 + \zeta_n^{-k} x_1 + \zeta_n^{-2k} x_2 + \cdots + \zeta_n^{-(n - 1)k} x_{n - 1}
\end{aligned}
$$

ですが，ここで右辺を $j$ （ $x$ の添字）の偶奇で 2 つに分けます．すると，

$$
\begin{aligned}
X_k &= x_0 + \zeta_n^{-2k} x_2 + \zeta_n^{-4k} x_4 + \cdots + \zeta_n^{-(n - 2)k} x_{n - 2} \\
&+ \zeta_n^{-k} x_1 + \zeta_n^{-3k} x_3 + \zeta_n^{-5k} x_5 + \cdots + \zeta_n^{-(n - 1)k} x_{n - 1}
\end{aligned}
$$

となります（上が偶数，下が奇数）．奇数の方は $\zeta_n^{-k}$ でくくることができます．

$$
\begin{aligned}
X_k &= x_0 + \zeta_n^{-2k} x_2 + \zeta_n^{-4k} x_4 + \cdots + \zeta_n^{-(n - 2)k} x_{n - 2} \\
&+ \zeta_n^{-k} (x_1 + \zeta_n^{-2k} x_3 + \zeta_n^{-4k} x_5 + \cdots + \zeta_n^{-(n - 2)k} x_{n - 1})
\end{aligned}
$$

さらに， $\zeta$ の添字と肩にある $n$ に $n = 2m$ を代入すると

$$
\begin{aligned}
X_k &= x_0 + \zeta_{2m}^{-2k} x_2 + \zeta_{2m}^{-4k} x_4 + \cdots + \zeta_{2m}^{-2(m - 1)k} x_{n - 2} \\
&+ \zeta_n^{-k} (x_1 + \zeta_{2m}^{-2k} x_3 + \zeta_{2m}^{-4k} x_5 + \cdots + \zeta_{2m}^{-2(m - 1)k} x_{n - 1})
\end{aligned}
$$

となりますが， $\zeta_{2m}^{2a} = \zeta_m^a$ が成り立つため

$$
\begin{aligned}
X_k &= x_0 + \zeta_m^{-k} x_2 + \zeta_m^{-2k} x_4 + \cdots + \zeta_m^{-(m - 1)k} x_{n - 2} \\
&+ \zeta_n^{-k} (x_1 + \zeta_m^{-k} x_3 + \zeta_m^{-2k} x_5 + \cdots + \zeta_m^{-(m - 1)k} x_{n - 1})
\end{aligned}
$$

となります．このとき，上は長さ $m$ の数列 $(x_0, x_2, x_4, \ldots, x_{n - 2})$ の DFT になっていて，下は長さ $m$ の数列 $(x_1, x_3, x_5, \ldots, x_{n - 1})$ の DFT になっています．

このように， $(x_0, x_2, x_4, \ldots, x_{n - 2})$ と $(x_1, x_3, x_5, \ldots, x_{n - 1})$ の DFT を用いて $(x_0, x_1, x_2, \ldots, x_{n - 1})$ の DFT を表すことを，**時間間引き**（ DIT ）といいます．

# 記法の導入
$\bm{x} = (x_0, x_1, x_2, \ldots, x_{n - 1})$ の添字を 2 進法で表したとき，最下位（ 1 の位）が $0$ であるようなものだけを取ってきて $\bm{x}_{:0} = (x_0, x_2, x_4, \ldots, x_{n - 2})$ とします．同様に，最下位が $1$ であるようなものだけを取ってきて $\bm{x}_{:1} = (x_1, x_3, x_5, \ldots, x_{n - 1})$ とします．

次に， $\bm{x}_{:0}$ の DFT を $(X_{0:0}, X_{1:0}, X_{2:0}, \ldots, X_{m - 1:0})$， $\bm{x}_{:1}$ の DFT を $(X_{0:1}, X_{1:1}, X_{2:1}, \ldots, X_{m - 1:1})$ とします．

すると，上の DIT は次のように書くことができます．

$$
X_k = X_{k:0} + \zeta_n^{-k} X_{k:1}
$$

# 高速フーリエ変換（ FFT ）
$m$ も偶数と仮定して $m = 2l$ とおき， $\bm{x}_{:0}$，$\bm{x}_{:1}$ に対してさらに DIT をかけることを考えます．

$\bm{x}$ の添字を 2 進法で表し，最下位が $0$，$1$ であるようなものを取ってきて，それぞれ $\bm{x}_{:0}$，$\bm{x}_{:1}$ としたのでした．同じ操作を $\bm{x}_{:0}$ に対してもう一度やると，今度は「$\bm{x}$ の添字を 2 進法で表したとき，下 2 桁が $00$，$10$ であるようなもの」が出てきます．これをそれぞれ $\bm{x}_{:00}$，$\bm{x}_{:10}$ とします．同様に，同じ操作を $\bm{x}_{:1}$ に対してやったものを $\bm{x}_{:01}$，$\bm{x}_{:11}$ とします．そして，各 $\iota = 00, 01, 10, 11$ について， $\bm{x}_{:\iota}$ の DFT を $(X_{0:\iota}, X_{1:\iota}, X_{2:\iota}, \ldots, X_{l - 1:\iota})$ とします（変数 $\iota$ は整数ではなく $0$ と $1$ の並びにつけられた名前だと思ってください）．

すると， $\bm{x}_{:0}$，$\bm{x}_{:1}$ の DFT に対する DIT はそれぞれ

$$
\begin{aligned}
X_{k:0} &= X_{k:00} + \zeta_m^{-k} X_{k:10} \\
X_{k:1} &= X_{k:01} + \zeta_m^{-k} X_{k:11}
\end{aligned}
$$

となります．

もし $n$ が $2$ のべき乗ならば，これを数列の長さが $1$ になるまで繰り返すことができます．

$t$ 回目の DIT を行った後，数列の長さは $\frac{n}{2^t}$ となり，次の DIT は

$$
\begin{aligned}
X_{k:\iota}
&= X_{k:0\iota} + \zeta_{\frac{n}{2^t}}^{-k} X_{k:1\iota} \\
&= X_{k:0\iota} + \zeta_{n}^{-k 2^t} X_{k:1\iota}
\end{aligned}
$$

となります．

数列 $\bm{x}_{:0\ldots00}$，$\bm{x}_{:0\ldots01}$，$\bm{x}_{:0\ldots10}$，$\ldots$，$\bm{x}_{:1\ldots11}$ の長さが $1$ になったとき，これらは $(x_0)$，$(x_1)$，$(x_2)$，$\ldots$，$(x_{n - 1})$ に他なりません．長さ $1$ の数列は DFT しても変わらないので，

$$
\begin{aligned}
(X_{0:0\ldots00}) &= (x_0) \\
(X_{0:0\ldots01}) &= (x_1) \\
(X_{0:0\ldots10}) &= (x_2) \\
\vdots \\
(X_{0:1\ldots11}) &= (x_{n - 1})
\end{aligned}
$$

となります．

$X_k$ を求めるには $X_{k:0}$，$X_{k:1}$ が分かればよく， $X_{k:0}$，$X_{k:1}$ を求めるには $X_{k:00}$，$X_{k:01}$，$X_{k:10}$，$X_{k:11}$ が分かればよいので， $X_{0:0\ldots00}$，$X_{0:0\ldots01}$，$X_{0:0\ldots10}$，$\ldots$，$X_{0:1\ldots11}$ が分かれば，逆にたどることで $X_k$ が得られます．

こうして， DIT を繰り返すことによって DFT を行うのが，**高速フーリエ変換**（ FFT ）です．

# 実装
実装の際は， $X$ の添字 $k:\iota$ を「 $k$ の $2$ 進法表記と $\iota$ を並べて書いて 1 つの数とみなしたもの」として扱います．たとえば， $k = 5$， $\iota = 10$ ならば， $k = 101_{(2)}$ と $10$ を並べて書くと $10110_{(2)} = 22$ なので， $X[22]$ に $X_{5:10}$ の値が入ります．

以下に， C++， python， Rust での実装を示します．
```cpp:FFT の C++ による実装
#include <vector>
#include <complex>
#include <cassert>

constexpr double TAU = 6.28318530717958647692528676655900577;

void fft(std::vector<std::complex<double>> &x, const unsigned bit){
	std::size_t n = x.size();
	assert(n == 1 << bit);
	std::size_t mask1 = n - 1; // k が数列の長さを超えたときに割った余りをとるマスク
	std::vector<std::complex<double>> zeta(n);
	for(std::size_t i = 0; i < n; ++i){
		zeta[i] = std::polar(1., -TAU * i / n); // ゼータの -i 乗
	}
	for(unsigned i = 0; i < bit; ++i){
		std::size_t mask2 = mask1 >> i + 1; // イオタの部分を得るマスク
		std::vector<std::complex<double>> tmp(n);
		for(std::size_t j = 0; j < n; ++j){
			std::size_t lower = j & mask2; // イオタの部分
			std::size_t upper = j ^ lower; // k の部分
			std::size_t shifted = upper << 1 & mask1;
			tmp[j] = x[shifted | lower] + zeta[upper] * x[shifted | mask2 + 1 | lower];
		}
		x = std::move(tmp);
	}
}
```

```python:FFT の python による実装
import math
import cmath

def fft(x, bit):
    n = len(x)
    assert n == 1 << bit
    mask1 = n - 1 # k が数列の長さを超えたときに割った余りをとるマスク
    zeta = [cmath.rect(1, -math.tau * i / n) for i in range(n)] # ゼータの 0 乗から -(n - 1) 乗
    for i in range(bit):
        mask2 = mask1 >> i + 1 # イオタの部分を得るマスク
        tmp = []
        for j in range(n):
            lower = j & mask2 # イオタの部分
            upper = j ^ lower # k の部分
            shifted = upper << 1 & mask1
            tmp.append(x[shifted | lower] + zeta[upper] * x[shifted | mask2 + 1 | lower])
        x = tmp
    return x
```

```rust:FFT の Rust による実装
use num::complex::Complex64; // 複素数
use std::f64::consts::TAU; // 2π

fn fft(x: &mut Vec<Complex64>, bit: u32) {
    let n = x.len();
    assert_eq!(n, 1 << bit);
    let mask1 = n - 1; // k が数列の長さを超えたときに割った余りをとるマスク
    let zeta: Vec<_> = (0..n)
        .map(|i| Complex64::from_polar(1., -TAU * i as f64 / n as f64))
        .collect(); // ゼータの 0 乗から -(n - 1) 乗
    for i in 0..bit {
        let mask2 = mask1 >> i + 1; // イオタの部分を得るマスク
        *x = (0..n)
            .map(|j| {
                let lower = j & mask2; // イオタの部分
                let upper = j ^ lower; // k の部分
                let shifted = upper << 1 & mask1;
                x[shifted | lower] + zeta[upper] * x[shifted | mask2 + 1 | lower]
            })
            .collect();
    }
}
```

# 終わりに
この記事で扱った実装方法は [satanic さんの github.io](https://satanic0258.github.io/snippets/math/FFT.html) に載っていたもので，[るまさんのブログ](https://tomorinao.blogspot.com/2018/10/various-fft.html)でも解説されています．私は勝手に **satanic FFT** などと呼んでいます．

# 追記 (2021/10/12)
[同様の手法で実装した NTT](https://judge.yosupo.jp/submission/63348) です．
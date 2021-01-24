---
title: "ACL の inv gcd の正体"
emoji: "⛳"
type: "tech"
topics: [競技プログラミング]
published: true
---
# inv gcd
AtCoder Library の中の `internal_math.hpp` というファイルを開くと， `inv_gcd` という名前の関数があります． 2 つの整数 $a$ ， $b$ を受け取って， 2 数の最大公約数 $g = \mathrm{gcd}(a, b)$ と， $xa \equiv g \mod b$ なる $x$ （ただし $0 \leq x < \frac bg$）を返します．

この記事では，この関数がどのように実装されているか考えます．

# 手計算
まず， $a = 100$ ， $b = 529$ とした場合を手計算してみます．一般的に知られているやり方は，

$$
\begin{alignedat}{5}
529 &\div &100 = 5 &\cdots &29 \\
100 &\div &29 = 3 &\cdots &13 \\
29 &\div &13 = 2 &\cdots &3 \\
13 &\div &3 = 4 &\cdots &1 \\
\end{alignedat}
$$

を変形して

$$
\begin{alignedat}{3}
29 &= &529 &- 5 \times &100 \\
13 &= &100 &- 3 \times &29 \\
3 &= &29 &- 2 \times &13 \\
1 &= &13 &- 4 \times &3 \\
\end{alignedat}
$$

とした上で，

$$
\begin{aligned}
1 &= 1 \times 13 &&+ (-4) \times 3 \\
&= 1 \times 13 &&+ (-4) \times (29 - 13 \times 2) \\
&= (-4) \times 29 &&+ (1 - (-4) \times 2) \times 13 \\
&= (-4) \times 29 &&+ 9 \times 13 \\
&= (-4) \times 29 &&+ 9 \times (100 - 3 \times 29) \\
&= 9 \times 100 &&+ (-4 - 9 \times 3) \times 29 \\
&= 9 \times 100 &&+ (-31) \times 29 \\
&= 9 \times 100 &&+ (-31) \times (529 - 5 \times 100) \\
&= (-31) \times 529 &&+ (9 - (-31) \times 5) \times 100 \\
&= (-31) \times 529 &&+ 164 \times 100
\end{aligned}
$$

から $164 \times 100 \equiv 1 \mod 529$ を得るものでしょう．これは，

$$
\begin{aligned}
1 &= x_1 \times 13 + x_2 \times 3 \\
&= x_2 \times 29 + x_3 \times 13 \\
&= x_3 \times 100 + x_4 \times 29 \\
&= x_4 \times 529 + x_5 \times 100
\end{aligned}
$$

としたときに

$$
\begin{aligned}
x_1 &= 1 \\
x_2 &= -4 \\
x_3 &= x_1 - x_2 \times 2 \\
x_4 &= x_2 - x_3 \times 3 \\
x_5 &= x_3 - x_4 \times 5 \\
\end{aligned}
$$

という計算で $x_5$ が求められるということを意味します．さらに $x_0 = 0$ とすると

$$
\begin{aligned}
x_0 &= 0 \\
x_1 &= 1 \\
x_2 &= x_0 - x_1 \times 4 \\
x_3 &= x_1 - x_2 \times 2 \\
x_4 &= x_2 - x_3 \times 3 \\
x_5 &= x_3 - x_4 \times 5 \\
\end{aligned}
$$

となります．

# 一般化
以上の議論を全て文字に置き換えると， $a_0 = b$ ， $a_1 = a$ として互除法を

$$
\begin{aligned}
a_0 \div a_1 &= q_0 \cdots a_2 \\
a_1 \div a_2 &= q_1 \cdots a_3 \\
a_2 \div a_3 &= q_2 \cdots a_4 \\
&\vdots \\
a_{n - 1} \div a_n &= q_{n - 1} \cdots 0 \\
\end{aligned}
$$

とした上で，

$$
\begin{aligned}
x_0 &= 0 \\
x_1 &= 1 \\
x_2 &= x_0 - x_1 \times q_{n - 2} \\
x_3 &= x_1 - x_2 \times q_{n - 3} \\
&\vdots \\
x_n &= x_{n - 2} - x_{n - 1} \times q_0 \\
\end{aligned}
$$

としたときの $x_n$ が求めたかったものでした．

しかし，互除法では $q_0, q_1, q_2, \ldots$ の順番に求まるのに対し， $x_0, x_1, x_2, \ldots$ を求めるときは $q_{n - 2}, q_{n - 3}, q_{n - 4}, \ldots$ の順番で使うので，このままだと各 $q_i$ の値を保存しておかなければなりません．これを避けるため，計算順序を変えることを考えます．

# 行列表記
$$
x_i = x_{i - 2} - x_{i - 1} \times q_{n - i}
$$

は，行列を用いると

$$
\begin{pmatrix}
x_{i - 1} \\
x_{i}
\end{pmatrix}
=
\begin{pmatrix}
0 & 1 \\
1 & -q_{n - i}
\end{pmatrix}
\begin{pmatrix}
x_{i - 2} \\
x_{i - 1}
\end{pmatrix}
$$

と書くことができます．よって， $x_n$ を求める段階は

$$
\begin{aligned}
	\begin{pmatrix}
		x_{n - 1} \\
		x_n
	\end{pmatrix}
	&=
	\begin{pmatrix}
		0 & 1 \\
		1 & -q_0
	\end{pmatrix}
	\begin{pmatrix}
		x_{n - 2} \\
		x_{n - 1}
	\end{pmatrix} \\
	\begin{pmatrix}
		x_{n - 2} \\
		x_{n - 1}
	\end{pmatrix}
	&=
	\begin{pmatrix}
		0 & 1 \\
		1 & -q_1
	\end{pmatrix}
	\begin{pmatrix}
		x_{n - 3} \\
		x_{n - 2}
	\end{pmatrix} \\
	\vdots \\
	\begin{pmatrix}
		x_{2} \\
		x_{3}
	\end{pmatrix}
	&=
	\begin{pmatrix}
		0 & 1 \\
		1 & -q_{n - 3}
	\end{pmatrix}
	\begin{pmatrix}
		x_1 \\
		x_2
	\end{pmatrix} \\
	\begin{pmatrix}
		x_{1} \\
		x_{2}
	\end{pmatrix}
	&=
	\begin{pmatrix}
		0 & 1 \\
		1 & -q_{n - 2}
	\end{pmatrix}
	\begin{pmatrix}
		x_0 \\
		x_1
	\end{pmatrix} \\
	&=
	\begin{pmatrix}
		0 & 1 \\
		1 & -q_{n - 2}
	\end{pmatrix}
	\begin{pmatrix}
		0 \\
		1
	\end{pmatrix}
\end{aligned}
$$
と書き換えることができます．これを一本にすると

$$
\begin{pmatrix}
	x_{n - 1} \\
	x_n \\
\end{pmatrix}
=
\begin{pmatrix}
0 & 1 \\
1 & -q_0
\end{pmatrix}
\begin{pmatrix}
0 & 1 \\
1 & -q_1
\end{pmatrix}
\cdots
\begin{pmatrix}
0 & 1 \\
1 & -q_{n - 2}
\end{pmatrix}
\begin{pmatrix}
0 \\
1 \\
\end{pmatrix}
$$

となり， $q_0, q_1, q_2, \ldots$ の順に使って計算することができるようになりました．

# 変形
行列を左から順に計算すると，

$$
\begin{aligned}
A_0 &= \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix} \\
A_1 &= A_0 \begin{pmatrix} 0 & 1 \\ 1 & -q_0 \end{pmatrix} \\
A_2 &= A_1 \begin{pmatrix} 0 & 1 \\ 1 & -q_1 \end{pmatrix} \\
\vdots \\
A_{n - 1} &= A_{n - 2} \begin{pmatrix} 0 & 1 \\ 1 & -q_{n - 2} \end{pmatrix} \\
\begin{pmatrix} x_{n - 1} \\ x_n \end{pmatrix} &= A_{n - 1} \begin{pmatrix} 0 \\ 1 \end{pmatrix} \\
\end{aligned}
$$

となります．

行列 $\begin{pmatrix} x & y \\ z & w \end{pmatrix}$ に行列 $\begin{pmatrix} 0 & 1 \\ 1 & -q_i \end{pmatrix}$ を右からかけると $\begin{pmatrix}y & x - q_iy \\ w & z - q_iw\end{pmatrix}$ となり，上の行（ $x$ $y$ ）と下の行（ $z$ $w$ ）は独立しています．よって，下の行のみに着目することで次のように書き換えることができます．

$$
\begin{aligned}
(z_0,\; w_0) &= (0,\; 1) \\
(z_1,\; w_1) &= (w_0,\; z_0 - q_0w_0) \\
(z_2,\; w_2) &= (w_1,\; z_1 - q_1w_1) \\
\vdots \\
(z_{n - 1},\; w_{n - 1}) &= (w_{n - 2},\; z_{n - 2} - q_{n - 2}w_{n - 2}) \\
x_n &= w_{n - 1}
\end{aligned}
$$

こうして，互除法を行いながら $(z,\;w)$ を更新することで $x_n$ が得られました．

# 確認
最初の $a = 100$ ， $b = 529$ の場合について計算して確かめてみます．

1. $(z_0,\; w_0) = (0,\; 1)$
1. $529 \div 100$ の商は $5$ なので $(z_1,\; w_1) = (w_0,\; z_0 - 5w_0) = (1,\; -5)$
1. $100 \div 29$ の商は $3$ なので $(z_2,\; w_2) = (w_1,\; z_1 - 3w_1) = (-5,\; 16)$
1. $29 \div 13$ の商は $2$ なので $(z_3,\; w_3) = (w_2,\; z_2 - 2w_2) = (16,\; -37)$
1. $13 \div 3$ の商は $4$ なので $(z_4,\; w_4) = (w_3,\; z_3 - 4w_3) = (-37,\; 164)$

ちゃんと，最後に $w_4$ の値が $164$ になっています．

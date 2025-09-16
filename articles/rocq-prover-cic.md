---
title: "Rocq（旧 Coq）の型付け規則を読む"
emoji: "📌"
type: "tech"
topics: ["Rocq", "Coq"]
published: true
---

プログラミング言語 Rocq（旧 Coq）の型システムについての勉強メモ。

# 項の定義

Rocq の項 $t$ は、以下の BNF で定義される。

$$\begin{align*}
t &&\Coloneqq\quad& \text{SProp} \mid \text{Prop} \mid \text{Set} \mid \text{Type}(i) \\
&&\mid\quad& x \quad \text{(変数)} \\
&&\mid\quad& c \quad \text{(定数)} \\
&&\mid\quad& \forall x : t, t \\
&&\mid\quad& \lambda x : t\mathpunct{.} t \quad\text{(ラムダ抽象)} \\
&&\mid\quad& t~t \quad\text{(関数適用)} \\
&&\mid\quad& \text{let}~ x \coloneqq t: t ~\text{in}~ t
\end{align*}$$

最初の 4 つを合わせて sorts と呼ぶ。

- $\text{SProp} : \text{Type}(1)$ はマジで知らないので今回は省く
  (nLab: https://ncatlab.org/nlab/show/strict+proposition)
- $\text{Prop} : \text{Type}(1)$ は命題に付く型。項 $t_1: \text{Prop}$ は命題で、$t: t_1$ なる項 $t$ がその証明（Curry-Howard 同型）
- $\text{Set}: \text{Type}(1)$ は集合に付く型
- $\text{Type}(1)$ の型は $\text{Type}(2)$、$\text{Type}(2)$ の型は $\text{Type}(3)$ ……

$x$ は変数、$c$ は定数。このあと型付け規則を 4 項関係 $E[\Gamma] \vdash t_1 : t_2$ で考えていくが、このとき定数 $c$ はグローバル環境 $E$ で、変数 $x$ はローカル文脈 $\Gamma$ で管理する。
- グローバル環境 $E$ は、仮定 $c: t$ と定義 $c \coloneqq t_1 : t_2$ と [inductive definition](https://rocq-prover.org/doc/V9.0.0/refman/language/core/inductive.html) の列
- ローカル文脈 $\Gamma$ は、仮定 $x: t$ と定義 $x \coloneqq t_1 : t_2$ の列
- $E$, $\Gamma$ が以下を満たすとき well-formed といって $\mathcal{WF}(E)[\Gamma]$ と書く
  - 定数 $c$ や変数 $x$ の名前が被らないこと
  - 仮定 $c: t$ の $t$ が型であること
  - 定義 $x \coloneqq t_1 : t_2$ の $t_1$ が型 $t_2$ をもつこと

$\lambda x : t_1\mathpunct{.} t_2$ はラムダ抽象 (abstraction) で、これにつく型は $\forall x : t_1, t_3$ の形になる。

$t_1 ~ t_2$ は関数適用 (application)。関数 $t_1$ に引数として $t_2$ を渡したもの。

$\text{let}~ x \coloneqq t_1 : t_2 ~\text{in}~ t_3$ は $(\lambda x : t_2\mathpunct{.} t_3) ~t_1$ と同じかな？

# 型付け規則
ここからは、資料を読まずにできるだけ自力で型付け規則を考えていく。

ラムダ抽象 $\lambda x : t_1\mathpunct{.} t_2$ には型 $\forall x : t_1, t_3$ を付けたい。つまり、$E[\Gamma] \vdash \lambda x : t_1\mathpunct{.} t_2 : \forall x : t_1, t_3$ を導くような規則 Lam が欲しい。前提として、$E[\Gamma]$ に変数 $x : t_1$ を追加した well-formed な $E[\Gamma \dblcolon x : t_1]$ の下で項 $t_2$ に型 $t_3$ がつく必要がありそう。

関数適用 $t_1 ~ t_2$ に型がつくには、項 $t_1$ の型が $\forall x: t_{1, 2}, t_{1, 1}$ の形をしている必要がありそうなので、$t_1 ~ t_2$ の型付け規則 App の前提の 1 つは

$$\begin{equation}
E[\Gamma] \vdash t_1 : \forall x : t_{1, 2}, t_{1, 1}
\end{equation}$$

だろうと予想できる。項 $t_2$ は変数 $x$ に代入されるので、前提

$$\begin{equation}
E[\Gamma] \vdash t_2 : t_{1, 2}
\end{equation}$$

も必要になる。このとき項 $t_1 ~ t_2$ の型は何になるだろうか？これは $t_1$ がラムダ抽象の場合を考えればいい。すると型付け $(1)$ は規則 Lam で導出されるはずなので、ラムダ抽象の変数は $x$、型は $t_{1, 2}$ と確定し、$t_1 = \lambda x : t_{1, 2} \mathpunct{.} t_{1, 3}$ とおけば

$$\begin{equation}
E[\Gamma \dblcolon x : t_{1, 2}] \vdash t_{1, 3} : t_{1, 1}
\end{equation}$$

が導出できるはずと分かる。また、$t_1$ がラムダ抽象なら $t_1 ~ t_2$ は $\beta$ 簡約できて、その結果は $t_{1, 3}$ 内の自由変数 $x$ を $t_2$ に書き換えて得られる項 $t_{1, 3} \{x / t_2\}$ だ。$\beta$ 簡約によって型は変わらないので、$t_1 ~ t_2$ の型と $t_{1, 3} \{x / t_2\}$ の型は同じはず。ここで $(2)$ と $(3)$ をよく眺める。$(3)$ の導出に現れる $E[\Gamma \dblcolon x : t_{1, 2}] \vdash x : t_{1, 2}$ を全て $(2)$ で置き換えると、

$$\begin{equation}
E[\Gamma] \vdash t_{1, 3} \{x / t_2\} : t_{1, 1} \{x / t_2\}
\end{equation}$$

も導出できそうに思える。よって、規則 App は前提 $(1)$, $(2)$ から結論 $E[\Gamma] \vdash t_1 ~ t_2 : t_{1, 1} \{x / t_2\}$ を導くのだろうと予想できる。

あとは well-formed な $E[\Gamma]$ に対して

- $E$ が $c : t$ か $c \coloneqq t' : t$ を含んでいれば $E[\Gamma] \vdash c : t$
- $\Gamma$ が $x : t$ か $x \coloneqq t' : t$ を含んでいれば $E[\Gamma] \vdash x : t$

という規則 Var, Const も必要だ。

さて、ここまでで残る懸念点は：

- 任意の項 $t$ に対して $c : t$ や $x : t$ を仮定できてしまう。
- 項 $\forall x : t_1 \mathpunct{.} t_2$ を好きに作れてしまう。

あたりだろうか。具体的に何がまずいのかまだよく分からないけれど、Rocq の型規則を眺めるとこの辺を避けていそうなので、おそらく証明を試みるとどこかで破綻するんだろう。というのを念頭に置きながら、手を動かしてみる。

（分かったら続きを書きます）

# 参考文献

もっぱらこれを読んでいます：

https://rocq-prover.org/doc/V9.0.0/refman/language/cic.html


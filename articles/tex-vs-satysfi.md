---
title: "TeX と SATySFi 主要記法比較"
emoji: "🐥"
type: "tech"
topics: ["tex", "satysfi"]
published: true
---

# 全体の構成
タイトルと著者の入れ方，プリアンブル

TeX：
```
\documentclass[uplatex]{jsarticle}
% プリアンブル
\title{タイトル}
\author{著者}
\begin{document}
\maketitle
段落1

段落2
\end{document}
```
SATySFi：
```
@require: stdjareport
% プリアンブル
document(| title = {タイトル}; author = {著者} |)'<
    +p{ 段落1 }
    +p{ 段落2 }
>
```
# 数式
TeX：
```
正の整数$n$と実数$a_1, \ldots, a_n$，$b_1, \ldots, b_n$に対し
\[
    \left(\sum_{k=1}^n a_k^2\right)
    \left(\sum_{k=1}^n b_k^2\right)
    = \left(\sum_{k = 1}^n a_k b_k\right)^2.
\]
```
SATySFi：
```
正の整数${n}と実数${a_1, \ldots, a_n}，${b_1, \ldots, b_n}に対し
\eqn(${
    \paren{\sum_{k=1}^n a_k^2}
    \paren{\sum_{k=1}^n b_k^2}
    = \paren{\sum_{k=1}^n a_k b_k}^2.
});
```
TeX：
```
\begin{align}
    \Gamma(z) &= \int_0^\infty t^{z-1} e^{-t} dt \\
    &= \left[-t^{z-1} e^{-t}\right]_0^\infty
     + (z-1) \int_0^\infty t^{z-2} e^{-t} dt \\
    &= (z-1) \Gamma(z-1)
\end{align}
```
SATySFi：
```
\align(${
    | \app{\Gamma}{z} |= \int_0^\infty t^{z-1} e^{-t} \ordd t
    ||= \sqbracket{-t^{z-1} e^{-t}}_0^\infty
      + \p{z-1} \int_0^\infty t^{z-2} e^{-t} \ordd t
    ||= \p{z-1} \app{\Gamma}{z-1}
|});
```

# コマンドの定義
TeX：
```
\documentclass[uplatex]{jsarticle}

\usepackage{amssymb}
\newcommand{\bR}{\mathbb{R}}
\usepackage{color}
\newcommand{\red}[1]{\textcolor{red}{#1}}

\begin{document}
    $\bR$を\red{実数}全体の集合とする．
\end{document}
```
SATySFi：
```
@require: stdjareport

let-math \bR = ${\mathbb{R}}
let-inline ctx \red it =
    let red = RGB(1., 0., 0.) in
    read-inline (set-text-color red ctx) it
in

document(| title = {}; author = {} |)'<
    +p{
        ${\bR}を\red{実数}全体の集合とする．
    }
>
```
# 表
TeX：
```
\documentclass[uplatex]{jsarticle}
\begin{document}
    \begin{tabular}{lcc}
        \hline
        & \TeX & SATySFi \\
        \hline
        作者 & D. Knuth & gfngfn \\
        開発年 & 1978 & 2017 \\
        \hline
    \end{tabular}
\end{document}
```
SATySFi：
```
@require: stdjareport
@require: easytable/easytable
open EasyTableAlias
in

document(| title = {}; author = {} |)'<
    +p{
        \easytable[l;c;c]{
        | | \TeX; | \SATySFi;
        | 作者 | D. Knuth | gfngfn
        | 開発年 | 1978 | 2017
        |}
    }
>
```
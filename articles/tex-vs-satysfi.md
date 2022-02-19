---
title: "TeX と SATySFi 主要記法比較"
emoji: "🐥"
type: "tech"
topics: ["tex", "satysfi"]
published: true
---

# 全体の構成
タイトルと著者の入れ方，プリアンブルなど．

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
このあたりはだいぶ違いますね．

TeX は空行で段落を分けますが，SATySFi は `+p{`〜`}` で段落を表します．
# 数式
> 正の整数$n$と実数$a_1, \ldots, a_n$，$b_1, \ldots, b_n$に対し
>
> $$
    \left(\sum_{k=1}^n a_k^2\right)
    \left(\sum_{k=1}^n b_k^2\right)
    \geq \left(\sum_{k = 1}^n a_k b_k\right)^2.
$$

TeX：
```
正の整数$n$と実数$a_1, \ldots, a_n$，$b_1, \ldots, b_n$に対し
\[
    \left(\sum_{k=1}^n a_k^2\right)
    \left(\sum_{k=1}^n b_k^2\right)
    \geq \left(\sum_{k = 1}^n a_k b_k\right)^2.
\]
```
SATySFi：
```
正の整数${n}と実数${a_1, \ldots, a_n}，${b_1, \ldots, b_n}に対し
\eqn(${
    \paren{\sum_{k=1}^n a_k^2}
    \paren{\sum_{k=1}^n b_k^2}
    \geq \paren{\sum_{k=1}^n a_k b_k}^2.
});
```

上付き `^2` 下付き `_1` とか `\sum` あたりはおおよそ同じです．

文中の数式は TeX だと `$`〜`$`，SATySFi だと `${`〜`}`．

別行立て数式は TeX だと `\[`〜`\]` や `equation` 環境を使いますが，SATySFi だと `\eqn(`〜`);` や `+math(`〜`);` を使います．

TeX で `(`〜`)` の大きさを調節するには `\left` `\right` などを使いますが，SATySFi だと `\paren{`〜`}` だけで大きさが自動調節される括弧になります（プリアンブルで azmath を require しておくと `\p{`〜`}`）．

> $$
\begin{align}
    \Gamma(z) &= \int_0^\infty t^{z-1} e^{-t} dt \\
    &= \left[-t^{z-1} e^{-t}\right]_0^\infty
     + (z-1) \int_0^\infty t^{z-2} e^{-t} dt \\
    &= (z-1) \Gamma(z-1)
\end{align}
$$

TeX：
```
\begin{align}
    \Gamma(z) &= \int_0^\infty t^{z-1} e^{-t} dt \\
    &= \left[-t^{z-1} e^{-t}\right]_0^\infty
     + (z-1) \int_0^\infty t^{z-2} e^{-t} dt \\
    &= (z-1) \Gamma(z-1)
\end{align}
```
SATySFi（azmath なし）：
```
\align[
    ${| \app{\Gamma}{z} |= \int_0^\infty t^{z-1} e^{-t} \ordd t |};
    ${||= \sqbracket{-t^{z-1} e^{-t}}_0^\infty
        + \paren{z-1} \int_0^\infty t^{z-2} e^{-t} \ordd t |};
    ${||= \paren{z-1} \app{\Gamma}{z-1} |}
];
```
SATySFi（azmath あり）：
```
\align(${
    | \app{\Gamma}{z} |= \int_0^\infty t^{z-1} e^{-t} \ordd t
    ||= \sqbracket{-t^{z-1} e^{-t}}_0^\infty
      + \p{z-1} \int_0^\infty t^{z-2} e^{-t} \ordd t
    ||= \p{z-1} \app{\Gamma}{z-1}
|});
```

複数行にわたる数式は，TeX では `align` 環境を使って `&` と `\\` で位置を揃えますが，SATySFi だと `\align(`〜`);` を使って `|` で区切ります．

# 章立て
TeX（jsarticle）：
```
\section{夏休みの宿題}
\subsection{概要}
多くの学校では夏休みに宿題が課される．
\subsection{主な宿題}
問題集，日記，読書感想文，自由研究などがある．
\subsection{期限までに出来なかった時にはどうするべきか}
居残りや親の呼び出しを避けるためになんとか言い訳を考える必要がある．
```
SATySFi（stdjabook）：
```
+section{夏休みの宿題}<
    +subsection{概要}<
        +p{
            多くの学校では夏休みに宿題が課される．
        }
    >
    +subsection{主な宿題}<
        +p{
            問題集，日記，読書感想文，自由研究などがある．
        }
    >
    +subsection{期限までに出来なかった時にはどうするべきか}<
        +p{
            なんとか言い訳を考える．
        }
    >
>
```
大きく違いますね．TeX は章や節の先頭に `\section{}` や `subsection{}` を入れるだけですが，SATySFi は `<`〜`>` を使って章や節の始まりと終わりを明記します．

# 箇条書き
TeX：
```
\begin{itemize}
    \item ごはん
    \item 麺
        \begin{itemize}
            \item うどん
            \item そば
            \item ラーメン
        \end{itemize}
    \item パン
\end{itemize}
```
SATySFi：
```
\listing{
    * ごはん
    * 麺
        ** うどん
        ** そば
        ** ラーメン
    * パン
}
```
TeX では `itemize` 環境を使って各項目の先頭に `\item` を付けます．一方 SATySFi では `\listing{`〜`}` を使い，各項目の先頭に `*` を付けます．入れ子にするときは，TeX だと新しく `itemize` 環境を始め，SATySFi だと `**` の数を増やします．

# コマンドの定義
毎回 `\mathbb{R}` と書く代わりに `\bR` を定義しておく．文字を赤くする `\red` を定義しておく．どちらもよくあるコマンド定義です．

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
これもけっこう雰囲気が違いますね．

SATySFi は数式コマンドを `let-math` で，インラインコマンド（文中で使うコマンド）を `let-inline` で定義します．TeX はどちらも `\newcommand` です．

SATySFi は OCaml や F# の影響を大きく受けているだけあって，関数型言語の記法ですね．
# 表
> | | TeX | SATySFi |
> |--|:--:|:--:|
> | 作者 | D. Knuth | gfngfn |
> | 開発年 | 1978 | 2017 |
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
こちらも，TeX は `&` と `\\` で区切るのに対し，SATySFi は `|` で区切ります．

他に何か思いついたら書き足します．
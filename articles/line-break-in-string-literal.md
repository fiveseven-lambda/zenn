---
title: "文字列リテラル中で改行できる言語，できない言語"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [プログラミング言語]
published: true
---

表にしました．

| 言語 | 文字列リテラル中での改行 |
| -- | -- |
| Perl | 可 |
| PHP | 可 |
| Ruby | 可 |
| D | 可 |
| Rust | 可 |
| Julia | 可 |
| Bash, Zsh 等 | 可 |
| Java | `"""` 〜 `"""` で可 |
| Python | `"""` 〜 `"""` 等で可 |
| Elm | `"""` 〜 `"""` で可 |
| C | 不可 |
| C++ | 不可（生文字列リテラルで可） |
| C# | 不可（生文字列リテラル等で可） |
| Go | 不可（生文字列リテラルで可） |
| Lua | 不可（長括弧で可） |
| JavaScript | 不可（テンプレートリテラルで可） |

他に，改行周りで面白い仕様の言語があったら教えてほしいです．
# 補足
## Java
Java15 以降では，`"` 〜 `"` の代わりに `"""` 〜 `"""` を使うと改行を含められます．

## Python
Python では，`'` 〜 `'` や `"` 〜 `"` の代わりに `'''` 〜 `'''` や `"""` 〜 `"""` を使うと改行を含められます．

ちなみに，前に `r` か `R` を付けると生文字列リテラル，`f` か `F` を付けるとフォーマット済み文字列リテラルです．
## Elm
フォロワーに教えてもらいました．
## C++
C++11 から raw string literal `R"` 〜 `"` が導入され，`(` 〜 `)` の中でエスケープが無効になります．このとき，改行も含められるようになります．
## C#
C# の `@` は「逐語的に解釈しろ」という意味で，文字列リテラル `"` 〜 `"` の前に付けて `@"` 〜 `"` とするとエスケープが無効になります．このとき，改行も含められるようになります．

また，C# 11 で導入された生文字列リテラル `"""` 〜 `"""` を使うと，エスケープが無効になることと改行が含められること以外に，インデントまでよしなにされます．これは嬉しい．
## Go
Go の生文字列リテラルはバッククォート \` 〜 \` で，エスケープが無効になるとともに改行が含められます．
## Lua
これもフォロワーに教えてもらいました．Lua の生文字列リテラル相当（エスケープ無効 & 改行可）は `[[` 〜 `]]` だそうです．
## JavaScript
JavaScript はバッククォート \` 〜 \` でくくってテンプレートリテラルを書けます．これは，`${` 〜 `}` 中に式を埋め込めるというものです（Python のフォーマット済み文字列に相当）．このとき，改行も含められるようになります．
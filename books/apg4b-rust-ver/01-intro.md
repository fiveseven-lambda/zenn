---
title: "はじめに"
---
:::message
対応する APG4b の箇所：[1.00.はじめに](https://atcoder.jp/contests/apg4b/tasks/APG4b_a)
:::
# この本は
Rust は 2010 年に登場した言語ですが， C++ や python など他の言語をよく知っている人に向けた説明ばかり出回っていて，プログラミング初心者向けの解説がほとんど存在しないのが現状です．この本は， AtCoder を利用しながら，プログラミング初心者に向けて Rust を分かりやすく解説することを狙いとしています．

この記事では AtCoder のジャッジシステムを利用するため，まず AtCoder にログインすることが必要になります．アカウントを持っていない人は，[ここ](https://atcoder.jp/register)からアカウント作成を行ってください．
# AtCoder を利用したコードの実行
APG4b に倣い， AtCoder のジャッジシステムの挙動とコードテストを使用した Rust の実行について説明します．
## 提出
AtCoder へのコードの提出は以下のように行います．
1. 次のコードをコピーして，[ここ](https://atcoder.jp/contests/apg4b/tasks/APG4b_a)のページ下部のコード入力欄に貼り付けます．
```rust
fn main() {
    println!("Hello, world!");
}
```
1. 言語の選択欄から， Rust を選択します．
1. 「提出」をクリックします．
1. 結果が「WJ」から「AC」になれば成功です．

## コードテスト
また，コードテストは[ここ](https://atcoder.jp/contests/practice/custom_test)や[ここ](https://atcoder.jp/contests/apg4b/custom_test)で行えます．提出と同様にコードを貼り付け， Rust を選択した上で「実行」をクリックします．終了コードが 0 であれば正常終了です．

---
title: "環境構築"
---

# 環境構築
前章で説明した通り，コードテストを用いるとプログラムを実行することができます．しかし，自分のパソコンの上でソースコードを書いたり実行したりしたい人もいるでしょう．この章では，そのような人のために環境構築の仕方を説明します．これからもコードテストを使い続ける人はこの章を読み飛ばしてもかまいません．

また，この章ではターミナルでのコマンドの実行が登場します． MacOS や Linux の場合は各自のターミナルエミュレータを， Windows の場合はコマンドプロンプトや PowerShell を開いてください．もし VSCode を開いているのであれば， VSCode 上のターミナル画面でもかまいません．

この章では実行するコマンドの先頭に `$` を付けています．これは，シェルが入力を受け付けていることを表すもので，実際に入力するのはその続きからになります．環境によっては，これが `>` だったり `%` だったりするかもしれません．
# インストールするもの
## `rustup`
後述する `rustc` （コンパイラ）や `cargo` （ビルドツール）等のインストールとバージョン管理を行うツールです．これをインストールすると， `rustc` と `cargo` も自動的にインストールされます．
## `rustc` （ `rustup` が自動でインストール）
Rust のコンパイラです． Rust で書かれたソースコードをコンパイルし，実行可能ファイルを生成します．
## `cargo` （ `rustup` が自動でインストール）
Rust のビルドを自動化するツールです．この章では， `rustc` を直接実行する代わりに， `cargo` を通して `rustc` を間接的に実行する方法を説明します．

この本の中で `cargo` が役に立つのは， `proconio` や `num` のようなライブラリを使う場面です．様々なライブラリが [crates.io](https://crates.io) で公開されており，使いたいものを `Cargo.toml` というファイルに書き込むと， `cargo` がダウンロードとビルドを自動的に行ってくれます．
## `cargo-edit`
`Cargo.toml` を編集するツールです．この章では， `Cargo.toml` を直接編集する代わりに， `cargo-edit` を通して `Cargo.toml` を間接的に編集する方法を説明します．
## Rust Language Server (RLS)
エディタに，エラーの表示やコードの補完などの機能を提供するツールです．この章では， VSCode を使う場合について説明します．
# 手順
## `rustup` のインストール
[公式](https://www.rust-lang.org/ja/tools/install)の手順に従ってください．

MacOS や Linux の場合は，ターミナルでコマンドを実行するだけです．万が一 `ld` のようなリンカがインストールされていない場合，これもインストールする必要があります．

一方 Windows の場合は， `rustup` 自体のインストーラの実行にくわえて，[ここ](https://visualstudio.microsoft.com/ja/downloads)から Visual C++ Build Tools をインストールすることが必要になります．
## `rustup` `rustc` `cargo` がインストールされていることの確認
バージョンは全て `--version` あるいは `-V` （大文字）で調べることができます．
```
$ rustup --version
$ rustc --version
$ cargo --version
```
## `cargo` によるビルド
`cargo` は， `new` や `build` や `run` などのサブコマンドをとります．

まず， Hello, world! の出力から始めましょう．新たなパッケージを作成するにはサブコマンド `new` を使います．ターミナルで
```
$ cargo new hello_world
```
を実行してください．すると， `hello_world` という名前のディレクトリが新たに作られ，そこに Rust のコードを書く環境が整います． `cd` で今できたディレクトリに移動してください．
```
$ cd hello_world
```
ソースファイルは `src/main.rs` にあります．このとき， `Hello, world!` を出力する次のようなプログラムが初めから書かれています．
```rust:src/main.rs
fn main() {
    println!("Hello, world!");
}
```
サブコマンド `run` で，ビルドと実行を行うことができます．
```
$ cargo run
```
`Hello, world!` と出力されれば成功です．

実行せずにビルドだけ行いときは，サブコマンド `build` を使います．
```
$ cargo build
```
## `cargo-edit` のインストール
ターミナルで次を実行します．
```
$ cargo install cargo-edit
```
これによって，サブコマンド `add` `rm` `list` が追加されます．

今後の章で最初に登場するライブラリは `proconio` です． `proconio` を使うときは，次のコマンドで `Cargo.toml` に `proconio` を追加します．
```
$ cargo add proconio
```
##  RLS のインストール
エディタとして VSCode を使う場合を想定します．

[Rust 用の拡張機能](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust)をインストールしてください．すると， `cargo new` で作ったディレクトリを VSCode で開いたときに RLS が自動的にインストールされてエラーの表示やコードの補完が有効になります．

---
title: "そもそもプログラミング経験自体無い人がRustを学ぶとき，どんな順序が良いのか"
emoji: "🦀"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Rust"]
published: true
---

遠きに行くには必ず邇きよりす．高きに登るには必ず卑きよりす．何事にも順序というものがあります．Rust の学習もそうです．

そこで，前提知識がほぼ無い状態から Rust を学ぶときに，どんな順序が良いのか，考えてみました．

1. コンパイル時と実行時の区別．
   Rust を学ぶとき，**何がコンパイル時に起こって何が実行時に起こるか**分からないと困ります．特に，型検査と借用検査がコンパイル時に行われることは，それらの基本的な規則を知る際に大切です．そこで，最初に Hello world を書く時点で，コンパイル→実行という流れを押さえておくべきでしょう．
1. コンパイルエラーの読み方．
   **まずコンパイルエラーを読む**という基本的な姿勢を身に付けるのは大切です．
1. 公式ドキュメントの場所．
   **まず公式ドキュメントを読む**という基本的な姿勢を身に付けるのも大切です．
1. Hello world 周辺の基本文法．
   例えば以下の項目です．
   - `fn main() { }` が要ること．
   - `println!` マクロ．
   - 文字列リテラル．
   - 整数と小数の，算術演算 (`+` `-` `*` `/` `%`) と出力．
   - コメント．
1. 変数と型．
   **`mut` 変数はまだ難しいので，今の段階では immutable な変数のみ扱いましょう．**`for` 式が出てくるまで，`mut` 変数の存在は知らなくて構いません．
   
   ここは，コンパイル時と実行時の区別が重要となる最初の場面です．例えば，全ての型推論がコンパイル時にできなければいけないことを理解しましょう．

   後で参照や所有権の話が出てくると，メモリに関する最低限のイメージが要求されるので，その基本をここで押さえると良いでしょう．ただし，スタックは再帰関数が登場してからで良いでしょう．
1. `if` 式．
   自分は競技プログラミング向けの記事を書いていたので，`proconio` で読んだ標準入力を用いて，条件分岐が実行時に起こることを説明しました．
1. ブロックとスコープ．
   `if` 式でブロックを目にしたので，変数のスコープや，値を返すブロックについても知ることができます．
1. アサートとパニック．
   配列を学ぶとき，範囲外アクセスの挙動を理解するのに必要です．副産物として，`assert!` や `assert_eq!` を含むサンプルコードが読めるようになります．
1. タプルと配列．
   `let` におけるパターンマッチもここで知っておくと，後で楽でしょう．
1. フォーマット出力．
   `println!` マクロの `{:x}` や `{2:3.1}` などです．タプルと配列が上で登場したので，`{:?}` についても知ることができます．
1. 参照とライフタイム．
   まだ `mut` 変数 / `mut` 参照を扱わないため，そこまで難しくないはずです．**参照のライフタイムが元の変数のスコープを越えられない**ことだけ，しっかりと押さえましょう．上でフォーマット出力を扱ったのは，ここで `{:p}` を使いアドレスを見て，イメージを掴むためです．
1. `for` 式． 
   配列と参照を扱ったので，配列への参照を `for` 式に渡して走査することを学べます．
1. `mut` 変数．
   `for` 式の中で変数を上書きしたいという需要が発生するので，ついに `mut` 変数が必要になります．`+=` のような複合代入演算子もここで知ることができます．
1. `mut` 参照．
   **shared XOR mutable** を理解します．ただし，この段階ではその有難みまで感じ取るのは難しいでしょう．ひとまず，そういうルールとして受け入れるしかないのかもしれないと考えています．
1. 関数．
   既に値を返すブロックを扱っているため，値を返す関数も導入しやすいでしょう．
1. 再帰関数．
   **実は我々はまだヒープを必要としない**ので，余計なことを考えずにスタックの仕組みを理解できます．
1. ベクタ．
   書けるプログラムの幅が一気に広がります．
1. ヒープと所有権．
   スタックの仕組みを理解していれば，それだけでベクタを実現しようとするとだいぶ困ってしまうということが分かると思います．そこで動的なアロケートとデアロケートを知り，**所有権**に基づいた適切なドロップの仕組みを知ることができます．ムーブによる所有権の移動も分かります．既に学んだ借用検査のルールと合わせることで，自然にコンパイルが通るプログラムを書く力が身に付きます．
1. 参照渡し．
   既に学んだことの組合せでしかないので，何も難しくありません．頻繁に使う手法として押さえておけば十分です．

この順序なら，誰でも Rust の基礎をしっかり身に付けることができるでしょう．

Rust 学習の道のりは，今どんどん整備されています．誰もが「初めてのプログラミング言語」として当たり前に Rust を選択できる時代も，そう遠くなさそうです．

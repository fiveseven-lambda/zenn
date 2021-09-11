---
title: "Option"
---

# Option
[`Option`](https://doc.rust-lang.org/std/option/enum.Option.html) は，標準ライブラリで定義されている列挙型です．中身は非常に単純です：
```rust
enum Option<T> {
    None,
    Some(T),
}
```
たとえば，`Option<i32>` は，`Some(-2147483648)` から `Some(2147483647)` までの $2^{31}$ 通りに `None` をくわえた $2^{31} + 1$ 通りの値をとることができます．

ただし，上の定義にくわえて以下のことが可能です．
- `Option::Some`，`Option::None` を単に `Some`，`None` と書くことができます．
  ```rust
  let x = Some(10); // Option::Some(10) と書かなくてよい
  let y = None; // Option::None と書かなくてよい
  let z: i32 = match y {
      Some(i) => i, // Option::Some(i) と書かなくてよい
      None => -1,   // Option::None と書かなくてよい
  };
  ```
  ただし，型パラメータ `T` が推論できなければエラーとなります．
  ```rust:コンパイルエラー
  let x = None;
  ```
- `T` が `==` `!=` で比較可能なら，`Option<T>` も `==` `!=` で比較できます．
  ```rust
  assert!(Some(10) == Some(10));
  assert!(Some(10) != Some(15));
  assert!(Some(10) != None);
  assert!(Option::<i32>::None == None); // None 同士も等しい
  ```
- `T` が `<` `<=` `>` `>=` で比較可能なら，`Option<T>` も `<` `<=` `>` `>=` で比較できます．ただし，`None` はどの `Some(T)` よりも小さい値として扱われます．
  ```rust
  assert!(Some(10) < Some(15));
  assert!(Some(10) > None); // None は最も小さい
  ```
- `T` がコピー可能なら，`Option<T>` もコピー可能です．
  ```rust
  let x = Some(10);
  let copied = x;
  assert_eq!(x, Some(10)); // x が使える
  ```

[公式ドキュメントの解説](https://doc.rust-lang.org/std/option/)では，`Option` がどんな場面で役に立つのか詳しく書かれています．本章では，例題を 1 問解きながら `Option` の使いどころを見てみます．
# 例題
以下の問題を考えます．
## 問題文
縦が $H$ マス，横が $W$ マスある長方形のマス目があります．上から $i$ 行目，左から $j$ 列目（$1 \leq i \leq H$，$1 \leq j \leq W$）のマスは「障害物が置かれている」「$A_{i, j}$ 枚のコインが置かれている」のいずれかです．ただし最も左上のマス（1 行目 1 列目）に障害物は置かれていません．

あなたは今，最も左上のマスにいます．あなたはこれから，今いるマスの右または下のマスに移動することを繰り返して，最も右下のマス（$H$ 行目 $W$ 列目）にたどり着かなければなりません．ただし，障害物が置かれているマスには行けません．

あなたは，自分が通ったマスに置かれていたコインを全て手に入れることができます．手に入れることができるコインの枚数の合計は，最大で何枚でしょうか？ただし，右下のマスにたどり着くことが不可能な場合は，代わりに `Impossible` と出力してください．

図にすると，たとえば下のようになります．❌の書かれたマス（2 行目 3 列目）に障害物があり，左上から右下まで 4 通りの行き方があります．図に示した 2 通りの行き方ではそれぞれ 11 枚, 12 枚のコインを手に入れることができ，今回は 12 枚が最大となります．
![](/images/option-path-dp.png)

入力は既に受け取ったものとします．`h: usize`，`w: usize` をぞれぞれ問題文中の $H$，$W$ とします．各マスの障害物の有無は `obstacle: Vec<Vec<bool>>` で表され，`obstacle[i][j]` が `true` なら `i` 行目 `j` 列目のマスには障害物があり，`false` なら無いものとします．各マスに置かれたコインの枚数は `coin: Vec<Vec<u64>>` で表され，`i` 行目 `j` 列目のマスには `coin[i][j]` 枚のコインが置かれているものとします．
## 解法
もとの問題では「最も右下のマスにたどり着くまでに手に入れることができるコインの枚数の最大値」を訊かれています．これを突然求める代わりに，「$i$ 行目 $j$ 列目のマスにたどり着くまでに手に入れることができるコインの枚数の最大値」を全ての $i$，$j$ について求めることを考えます．

たとえば，上の例で全ての $i$，$j$ についてこれを求めて表にすると以下のようになります．
![](/images/option-path-dp2.png)
たとえば左上から 3 行目 2 列目のマスまでは 3 通りの行き方があります．その過程で手に入れることのできるコインの枚数はそれぞれ 8, 8, 9 枚であり，その最大値 9 が表に書き込まれています．

この表のあるマスに着目したとき，その値は上のマスと左のマスの値に基づいて計算できることが分かります．
![](/images/option-path-dp3.png)
たとえば，3 行目 2 列目のマスまで行く方法は，2 行目 2 列目のマスを通って行くか，3 行目 1 列目のマスを通って行くかのどちらかです．2 行目 2 列目までは 8 枚のコインを集めながら行くことができますが，3 行目 1 列目までだと最大で 7 枚しか集められません．よって 3 行目 2 列目のマスまで行くには 2 行目 2 列目のマスを通るのがよく，3 行目 2 列目のマスに置かれている 1 枚と合わせて計 9 枚のコインが得られることになります．

よってこの表は，左上から順番に埋めていくことで楽に計算できることが分かります．表を全て埋めたら，最も右下のマスに書かれている数が答えとなります．

障害物がもう少し多い場合も考えてみましょう．
![](/images/option-path-dp4.png)
上と同じように表を作り，左上から順番に埋めてみてください．

2 行目 5 列目には障害物が置かれていませんが，このマスにたどり着くことはできません．このように，一部のマスは到達不可能なことがあり，その場合は表に到達不可能だということを書き込む必要があると分かります．

最も上の行と最も左の列を除くと，表の $i$ 行目 $j$ 列目は次のように埋められることが分かります：
- 上のマスと左のマスがどちらも到達可能ならば，書かれた数のうち大きい方を選び，そこに $i$ 行目 $j$ 列目のコインの枚数を足した値を，表の $i$ 行目 $j$ 列目に書き込む．
- 上のマスと左のマスのうち片方のみが到達可能ならば，そこに書かれた数に $i$ 行目 $j$ 列目のコインの枚数を足した値を，表の $i$ 行目 $j$ 列目に書き込む．
- 上のマスと左のマスがどちらも到達不可能ならば，$i$ 行目 $j$ 列目も到達不可能である．

また，最も上の行や最も左の列（ただし最も左上の 1 マスは除く）のように，上のマスや左のマスがそもそも存在しない場合も，到達不可能な場合と同じように扱って良いことが分かります．

最も左上のマスだけは，表の他の部分から計算することができません．代わりに，最も左上のマスに置いてあるコインの枚数が，そのまま書き込む値となります．

## 実装
今回作ろうとしている表は，各マスが「到達可能かどうか」「到達可能ならば，たどり着くまでに最大で何枚のコインを集められるか」という情報を持っています．これはそのまま，`Option` を使って次のように表現することができます．
- `Some(c)`……到達可能であり，たどり着くまでに最大で `c` 枚のコインを集めることができる．
- `None`……到達不可能である．

まずは表となる変数 `table` を用意します．初めは全てのマスを `None` で初期化しておきます．
```rust
let mut table: Vec<Vec<Option<u64>>> = vec![vec![None; w]; h];
```
次に，出発点となる最も左上のマスを書き込みます．
```rust
table[0][0] = Some(coin[0][0]);
```

2 重の `for` ループを使って，表を左上から順に見ていきます．ただし，障害物のあるマスは `continue;` を使って飛ばします．
```rust
for i in 0..h {
    for j in 0..w {
        if obstacle[i][j] {
            continue;
        }
        // ここで table[i][j] を計算する
    }
}
```

あとは，このループの中身を埋めるだけです．まずは，上のマスについて調べます．
```rust
        let above = if i > 0 { table[i - 1][j] } else { None };
```
上のマスが存在すれば `table[i - 1][j]`，存在しなければ `None` を代入しています．すると，`above` は「上のマスが存在しない場合」「上のマスが到達不可能な場合」のどちらにおいても `None` となり，`above` が `Some(c)` であれば上のマスにたどり着くまでに最大で `c` 枚のコインを集めることができるという意味になります．

同様に左のマスについても調べます．
```rust
        let left = if j > 0 { table[i][j - 1] } else { None };
```

そして，前の節で調べたやり方に従って `table[i][j]` を埋めます．
```rust
        let c = match (above, left) {
            (Some(above_c), Some(left_c)) => above_c.max(left_c), // どちらも到達可能
            (Some(above_c), None) => above_c,                     // 上のマスだけ到達可能
            (None, Some(left_c)) => left_c,                       // 左のマスだけ到達可能
            (None, None) => continue,                             // どちらも到達不可能
        };
        table[i][j] = Some(c + coin[i][j]); // i 行目 j 列目のコインの枚数を足す
```
どちらも到達不可能なときは `continue` することで `table[i][j]` の更新を防いでいます．

先ほど，`T` が `<` `<=` `>` `>=` で比較可能なら，`Option<T>` も `<` `<=` `>` `>=` で比較できるといいました．`Option<T>` 同士で大小の比較ができるということは，`Option<T>` に対して直接 `.max()` を呼んで大きい方を得られるということでもあります．よって，今の部分は次のように書き換えることができます．
```rust
        if let Some(c) = above.max(left) {
            table[i][j] = Some(c + coin[i][j]);
        }
```
片方，または両方が `None` だった場合どうなるか考えてみてください．

さて，こうして 2 重ループにより表を埋めることができたら，最後に表の右下を見て答えを出力すれば終わりです．
```rust
match table[h - 1][w - 1] {
    Some(ans) => println!("{}", ans),
    None => println!("Impossible"),
}
```
全体では，次のようになります．
```rust
let mut table = vec![vec![None; w]; h];
table[0][0] = Some(coin[0][0]);
for i in 0..h {
    for j in 0..w {
        if obstacle[i][j] {
            continue;
        }
        let above = if i > 0 { table[i - 1][j] } else { None };
        let left = if j > 0 { table[i][j - 1] } else { None };
        if let Some(c) = above.max(left) {
            table[i][j] = Some(c + coin[i][j]);
        }
    }
}
match table[h - 1][w - 1] {
    Some(ans) => println!("{}", ans),
    None => println!("Impossible"),
}
```
このように，変数が値をもつ場合ともたない場合があるようなときに `Option` が役立ちます．
# メソッド
Option は[様々なメソッド](https://doc.rust-lang.org/std/option/enum.Option.html#implementations)を実装しています．ここではそのうちいくつかを紹介します．
## `is_some()`，`is_none()`
[`is_some()`](https://doc.rust-lang.org/std/option/enum.Option.html#method.is_some) と [`is_none()`](https://doc.rust-lang.org/std/option/enum.Option.html#method.is_none) はそれぞれ次のような関数です．
```rust
impl<T> Option<T> {
    fn is_some(&self) -> bool { /* Some なら true， None なら false */ }
    fn is_none(&self) -> bool { /* Some なら false， None なら true */ }
}
```
`is_some()` は `Some` のとき，`is_none()` は `None` のときにそれぞれ `true` を返します．
## `unwrap()`，`expect()`
[`unwrap()`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap)，[`expect()`](https://doc.rust-lang.org/std/option/enum.Option.html#method.expect) はそれぞれ次のような関数です．
```rust
impl<T> Option<T> {
    fn unwrap(self) -> T { /* 略 */ }
    fn expect(self, msg: &str) -> T { /* 略 */ }
}
```
どちらも `Option<T>` を受け取り，`Some` であるという仮定のもとに，その中身である `T` を返します．もし `None` であれば panic します．

2 つの違いは，panic するときの標準エラー出力の内容を引数 `msg` として指定できるかどうかです．
## `unwrap_or()`
[`unwrap_or()`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or) は `unwrap()` と違い，たとえ `None` でも panic せず，代わりに引数 `default` として与えた値を返します．
```rust
impl<T> Option<T> {
    fn unwrap_or(self, default: T) -> T { /* 略 */ }
}
```
上の例題では「不可能なとき `Impossible` と出力せよ」となっていましたが，これがもし「不可能なとき `0` と出力せよ」であれば
```rust
println!("{}", table[h - 1][w - 1].unwrap_or(0));
```
で済んだわけです．
# `Option` を返すメソッド
返り値として `Option` を返すメソッドもあります．
## `Vec` の `pop()`
`Vec` の末尾の要素を削除する [`pop()`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.pop) 関数は，削除した末尾の要素を `Some` に包んで返します．もし `Vec` が空でこれ以上削除できなければ，`None` を返します．
```rust
let mut v = vec![10];
assert_eq!(v.pop(), Some(10));
assert_eq!(v.pop(), None);
```
## スライスの `get()`
配列，ベクタ，スライスに `[i]` を付けると要素にアクセスできますが，範囲外アクセスのとき panic します．一方，[`get()`](https://doc.rust-lang.org/std/primitive.slice.html#method.get) メソッドを使うと範囲外アクセスのとき `None` を返します．
```rust
let a = [3, 1, 4, 1, 5];
assert_eq!(a.get(2), Some(&4));
assert_eq!(a.get(5), None);
```
## 整数型の `checked_add()` `checked_sub()` 等
`i32` 型では $-2^{31}$ 以上 $2^{31} - 1$ 以下の整数しか表せないため，これを超える範囲で足し算や引き算などを行おうとしても正確に計算できません．計算の結果がその型で表せる範囲を超えてしまうことを，オーバーフローといいます．

[`checked_add()`](https://doc.rust-lang.org/std/primitive.i32.html#method.checked_add) は 2 つの整数の和を `Some` に包んで返します．ただし，オーバーフローが起こると `None` を返します．
```rust
let x: i32 = 1000000000;
let y = 2000000000;
// let z = x + y; // NG: オーバーフローする
assert!(x.checked_add(y).is_none());
```
同様に，引き算 [`checked_sub()`](https://doc.rust-lang.org/std/primitive.i32.html#method.checked_sub)，かけ算 [`checked_mul()`](https://doc.rust-lang.org/std/primitive.i32.html#method.checked_mul)，割り算の商 [`checked_div()`](https://doc.rust-lang.org/std/primitive.i32.html#method.checked_div)，割り算の余り [`checked_rem()`](https://doc.rust-lang.org/std/primitive.i32.html#method.checked_rem) などがあります．
---
title: "再帰関数"
---
関数の中で，その関数自身を呼び出すことができます．
```rust
fn fnc() {
    // ここで fnc() を呼び出す
}
```
内部で自分自身を呼び出すような関数を，**再帰関数**といいます．この章では，再帰関数によって何ができるか考えます．

# 例 1: 階乗

$0$ 以上の整数 $n$ について， $1$ 以上 $n$ 以下の整数をすべてかけ合わせた数を $n$ の階乗といい， $n!$ と書きます．ただし $n = 0$ のときについては $0! = 1$ と定義されます．

`i32` 型の整数 `n` を受け取って， `n` の階乗を返す関数 `fact` を，再帰関数として書くと，次のようになります．
```rust
fn fact(n: i32) -> i32 {
    if n == 0 {
        1
    } else {
        fact(n - 1) * n
    }
}
```
まず，引数 `n` を 3 として `fact(3)` を呼び出すとどうなるか追ってみましょう．

1. 呼び出された `fact(3)` の中では $n = 3$ である．
1. `n == 0` が `false` なので， `else` ブロックが実行される．
1. `fact(n - 1)` すなわち `fact(2)` が呼び出される．
   1. 呼び出された `fact(2)` の中では $n = 2$ である．
   1. `n == 0` が `false` なので， `else` ブロックが実行される．
   1. `fact(n - 1)` すなわち `fact(1)` が呼び出される．
      1. 呼び出された `fact(1)` の中では $n = 1$ である．
      1. `n == 0` が `false` なので， `else` ブロックが実行される．
      1. `fact(n - 1)` すなわち `fact(0)` が呼び出される．
         1. 呼び出された `fact(0)` の中では $n = 0$ である．
         1. `n == 0` が `true` なので， `if` ブロックが実行される．
         1. `fact(0)` の返り値は 1 となる．
      1. `fact(1)` の返り値は， `fact(0) * 1` すなわち 1 となる．
   1. `fact(2)` の返り値は， `fact(1) * 2` すなわち 2 となる．
1. `fact(3)` の返り値は， `fact(2) * 3` すなわち 6 となる．

ちゃんと， $3! = 6$ が返ってきています．

## 再帰関数の書き方

再帰関数を書くときには，まず `fact(a)` と `fact(b)` の間に成り立つ何らかの関係式を考えます．今回使った関係式は，

- $n! = (n - 1)! \times n$ （ $n = 1, 2, 3, \ldots$ ）

です． $(n - 1)!$ の値が分かれば，そこに $n$ をかけることで $n!$ の値も分かります．つまり， `fnc(n)` の中で `fnc(n - 1)` を呼び出すと， `fnc(n - 1) * n` を `fnc(n)` の返り値とすることができます．しかし，

```rust
fn fact(n: i32) -> i32 {
    fact(n - 1) * n
}
```

とは書けません．こう書くと， `fact(3)` が `fact(2)` を呼び出し， `fact(2)` が `fact(1)` を呼び出し， `fact(1)` が `fact(0)` を呼び出し， `fact(0)` が `fact(-1)` を呼び出し， `fact(-1)` が `fact(-2)` を呼び出し……というように，呼び出しが無限に続いてしまいます．

関係式 $n! = (n - 1)! \times n$ が成り立つのは， $n \geq 1$ のときでした．よって $n = 0$ のときは `fact(n - 1) * n` とすることができないので，代わりに $0! = 1$ を返すようにします．こうすると， `fact(3)` から始まった呼び出しのループは， `fact(1)` が `fact(0)` を呼び出したところで終了することになります．このように，再帰を終了させるために作られる場合分けをベースケースといいます．

```rust
fn fact(n: i32) -> i32 {
    if n == 0 {
        1 // ベースケース
    } else {
        fact(n - 1) * n
    }
}
```


# 例 2: スライスの総和

次は， `[i32]` 型のスライスを受け取って，要素の総和を返す関数 `sum` を考えます．たとえば `sum([5, -1, 3, 4, -2])` の返り値は $5 + (-1) + 3 + 4 + (-2) = 9$ となります．

今回は，次の事実に着目します．

- 最初の要素を除いた残りの部分 `[-1, 3, 4, -2]` の総和 $4$ が分かっていれば，そこに先頭の要素 `5` を足すことで， `[5, -1, 3, 4, -2]` の総和が $4 + 5 = 9$ であると分かる．

よって， `sum(slice)` の中で `sum(&slice[1..])` を呼び出すと，総和を計算することができそうです．

次にベースケースを考えます． `sum([5, -1, 3, 4, -2])` は `sum([-1, 3, 4, -2])` を呼び出し， `sum([-1, 3, 4, -2])` は `sum([3, 4, -2])` を呼び出し， `sum([3, 4, -2])` は `sum([4, -2])` を呼び出し， `sum([4, -2])` は `sum([-2])` を呼び出し， `sum([-2])` は `sum([])` を呼び出します．スライスが空スライス `[]` になると，「最初の要素を除いた残りの部分」を考えることができなくなるので，ここをベースケースとすることになります． `sum([])` 自体の返り値は，足すものが何も無いので 0 となります．

全体では次のようになります． `x` がスライスのとき， [`x.is_empty()`](https://doc.rust-lang.org/std/primitive.slice.html#method.is_empty) は `x` が空スライスなら `true` に，そうでないなら `false` になります．
```rust
fn main() {
    assert_eq!(sum(&[5, -1, 3, 4, -2]), 9);
}

fn sum(slice: &[i32]) -> i32 {
    // is_empty 関数は空スライスのとき true を返す
    if slice.is_empty() {
        0 // ベースケース
    } else {
        // 最初の要素を除いた残りの部分の総和
        sum(&slice[1..])
        // に，最初の要素を足す
            + slice[0]
    }
}
```
# 例 3: クイックソート
`x` が可変なスライスのとき， [`x.sort()`](https://doc.rust-lang.org/std/primitive.slice.html#method.sort) を呼び出すと `x` の要素が昇順にソートされる（小さい順に並べ替えられる）のでした．これと同じことをする関数を作ってみます．ソートのやり方は数多くありますが，今回は再帰関数を使ってクイックソートと呼ばれるものを書いてみます．
```rust
fn sort(slice: &mut [i32]) {
    todo!();
}
```

まず，空でないスライスの中から要素を一つ選び，これをピボットと呼びます．今回は，話を簡単にするためスライスの最初の要素をピボットとします．
```rust
let pivot = slice[0];
```

また，スライスの長さを `len` という変数でおいておきます．
```rust
let len = slice.len();
```

## 分割
ここからスライスに対して [`swap`](https://doc.rust-lang.org/std/primitive.slice.html#method.swap) 関数を使った操作を繰り返し，次のような状態を目標とします．

- ある `n` が存在して，
  - 範囲 `slice[..n]` の要素は全て `pivot` 以下であり，
  - `slice[n]` は `pivot` に等しく，
  - 範囲 `slice[n + 1..]` の要素は全て `pivot` より大きい．

例えば次のような状態です．

![](https://storage.googleapis.com/zenn-user-upload/049dnu81wm6qxfyedcalikyvfsdn)

これを，ここでは「スライスが `pivot` で分割されている」と呼ぶことにします．

 `left` ， `right` という 2 つの `usize` 型可変変数を用意し，それぞれ `1` と `len - 1` を代入しておきます．
```rust
let mut left = 1;
let mut right = len - 1;
```
`slice[left]` は，今ピボットの直後を指しています．これを， `left += 1` を繰り返すことで右に動かしていき， `left == len` あるいは `slice[left] > pivot` となった時点で終了します．
```rust
while left < len && slice[left] <= pivot {
    left += 1;
}
```
このループが終了したとき，範囲 `slice[1..left]` の要素は全て `pivot` 以下です．

一方 `slice[right]` は，今スライスの最後の要素を指しています．これを， `right -= 1` を繰り返すことで左に動かしていき， `right == 0` あるいは `slice[right] <= pivot` となった時点で終了します．
```rust
while right > 0 && slice[right] > pivot {
    right -= 1;
}
```
このループが終了したとき，範囲 `slice[right + 1..]` の要素は全て `pivot` より大きいです．

両方のループが終了した段階で，次の 3 通りが考えられます．

1. `left == len` である．
   このとき `slice[1..len]` の要素は全て `pivot` 以下なので， `slice[len - 1] <= pivot` が成り立つ．よって `right` は `right == len - 1` で止まっている．
1. `right == 0` である．
   このとき `slice[1..]` の要素は全て `pivot` より大きいので， `slice[1] > pivot` が成り立つ．よって `left` は `left == 1` で止まっている．
1. それ以外．
   このとき， `left` のループの終了条件より `left < len` かつ `slice[left] > pivot` が成り立っていて， `right` のループの終了条件より `right > 0` かつ `slice[right] <= pivot` が成り立っている．
   また， `slice[1..left]` の要素は全て `pivot` 以下だが， `slice[right + 1]` は `pivot` より大きいので， `left <= right + 1` が成り立つ．

2 のとき， `slice[0] == pivot` であり，範囲 `slice[1..]` の要素が全て `pivot` より大きいので，スライスは `pivot` で分割されています． 1 のときは， `slice[0]` と `slice[len - 1]` を入れ替えると，範囲 `slice[0..len - 1]` の要素が全て `pivot` 以下で， `slice[len - 1] == pivot` となるので，スライスが `pivot` で分割された状態になります．

3 のとき， `left == right + 1` の場合と `left < right` の場合の 2 通りが考えられます． `left == right + 1` の場合， `slice[0]` と `slice[right]` を入れ替えると，スライスは `pivot` で分割された状態になります．一方 `left < right` の場合は， `slice[left]` と `slice[right]` を入れ替えた後，再度 `left` を右に， `right` を左に動かして分割を試みます．このとき，範囲 `slice[1..left + 1]` の要素は全て `pivot` 以下であり，範囲 `slice[right..]` の要素は全て `pivot` より大きいことが分かっているので， `left` のループを `left = 1` から始めたり， `right` のループを `right = len - 1` から始めたりする必要はありません．

`left == len` のときと `right == 0` のときも `left == right + 1` は成り立っているので，場合分けは `left == right + 1` と `left < right` の 2 通りになります．また， `left == right + 1` のときにはスワップ `slice.swap(0, right)` を行いますが， `left == len` のときのスワップ `slice.swap(0, len - 1)` はこれに含めることができ， `right == 0` のときも `slice.swap(0, 0)` という（何も起こらない）スワップを行うことで `slice.swap(0, right)` に含めることができます．よって，ここまでをコードに書くと次のようになります．
```rust
let pivot = slice[0];
let len = slice.len();
let mut left = 1;
let mut right = len - 1;
loop {
    while left < len && slice[left] <= pivot {
        left += 1;
    }
    while right > 0 && slice[right] > pivot {
        right -= 1;
    }
    if left == right + 1 {
        slice.swap(0, right);
        break;
    } else if left < right {
        slice.swap(left, right);
        left += 1;
        right -= 1;
    } else {
        panic!();
    }
}
```

:::message
`left < len && slice[left] <= pivot` で，短絡評価を行う `&&` 演算子を用いています． `left < len` が先にあるので， `left < len` が真だったときにのみ `slice[left] <= pivot` かどうか確かめられます．ここでもし，短絡評価を行わない `&` 演算子を使って `(left < len) & (slice[left] <= pivot)` と書いていたらどうなるでしょうか？仮に `slice[1..]` の要素が全て `pivot` より小さく，ループ中で `left == len` になったとします．このとき， `left < len` は偽ですが，それでも `slice[left] <= pivot` かどうか確かめられます．しかし今 `left` は `0` 以上 `len` 未満の範囲にないので， `slice[left]` は範囲外アクセスになってしまいます．よって， `&` ではなく `&&` を使わなければいけません．
:::
## 再帰呼出し
ここまでの処理が終了したとき， `slice[..right]` の要素は全て `pivot` 以下であり， `slice[right + 1..]` の要素は全て `pivot` より大きいので， `slice[..right]` の部分と `slice[right + 1..]` の部分をそれぞれソートすると， `slice` 全体がソートされたことになります．ここで， `sort` 関数を再帰呼び出しします．
```rust
sort(&mut slice[..right]);
sort(&mut slice[right + 1..]);
```
`slice[..right]` と `slice[right + 1..]` は `slice` よりも小さいので，呼び出しが繰り返されるごとにスライスは小さくなっていきます．最終的にスライスの長さが 0 か 1 となったときをベースケースとします．

これで， `sort` 関数全体は次のようになります．
```rust
fn sort(slice: &mut [i32]) {
    let len = slice.len();
    if len > 1 {
        let pivot = slice[0];
        let mut left = 1;
        let mut right = len - 1;
        loop {
            while left < len && slice[left] <= pivot {
                left += 1;
            }
            while right > 0 && slice[right] > pivot {
                right -= 1;
            }
            if left == right + 1 {
                slice.swap(0, right);
                break;
            } else if left < right {
                slice.swap(left, right);
                left += 1;
                right -= 1;
            } else {
                panic!();
            }
        }
        sort(&mut slice[..right]);
        sort(&mut slice[left..]);
    }
}
```
実際に実行し，ソートが正しく行われているか確かめてみてください．

# 再帰的な考え方
再帰関数を書くのは慣れるまで難しいことが多いです．しかし，一度慣れると高度な処理を書くことができるようになります．

後の章で，深さ優先探索というものについて説明します．このときまた再帰関数が活躍します．

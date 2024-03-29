---
title: "所有権"
---
# コピー可能性
今まで， `i32` や， `&i32` や `Vec<i32>` など，様々な型が登場してきました．

実はこれらの型は，コピー可能なものとそうでないものの 2 つに分けられます．

| コピー可能なもの | そうでないもの |
|--|--|
| `i32` `usize` `f64` などの数値型<br>不変参照`&T` | ベクタ `Vec<T>` <br> 文字列 `String` <br> 可変参照 `&mut T` |

この章では文字列とベクタを例にとって，「コピー可能でない」とはどういうことか説明します．

:::message
注意：この章では，話を簡略化するためにスタック領域とヒープ領域を*区別せず*説明しており，不十分・不正確な部分があります．
:::
# 所有権
## コピー可能な場合
まず， `hoge` という名前の `i32` 型変数を宣言し，値 10 を代入します．
```rust
let hoge: i32 = 10;
```

メモリ上の 4 バイトの領域に値 10 が書き込まれ，これ以降様々な計算に使用できるようになります．

このことを，「変数 `hoge` が値 10 に対して**所有権**をもっている」といいます．所有権を持っている変数を所有者といいます．所有者と値の対応を表にするとこのようになります．

| 所有者 | 値 |
| -- | -- |
| `hoge` | 10 |

次に， `hoge` の値を別の変数 `copied` に代入します．
```rust
let copied = hoge;
```

今度は，メモリ上に*別の* 4 バイトの領域が確保され，そこに `hoge` の値 10 が*コピー*されます．

すると `copied` も値 10 に対して所有権を持ちますが，この 10 は `hoge` の所有する 10 とは別の場所にある 10 です．
| 所有者 | 値 |
| -- | -- |
| `hoge` | 10 |
| `copied` | 10 |

メモリ上に値 10 が 2 つあり， `hoge` と `copied` が 1 つずつ所有している状態です．

## コピー可能でない場合
今度は， `vector` という名前の `Vec<i32>` 型変数を宣言し，ベクタ `vec![1, 2, 3]` を代入します．
```rust
let vector: Vec<i32> = vec![1, 2, 3];
```

変数 `vector` の中身は， `[i]` で要素にアクセスしたり `for` 式で走査したりして使用することができます．これも，変数 `vector` が値 `vec![1, 2, 3]` に対して所有権を持っているからです．

| 所有者 | 値 |
| -- | -- |
| `vector` | `vec![1, 2, 3]` |

次に， `vector` を別の変数 `moved` に代入します．
```rust
let moved = vector;
```

`Vec<i32>` という型は，コピー可能ではありません．このとき， `vector` を `moved` に代入しても，メモリ上に値 `vec![1, 2, 3]` を表す領域が新しく作られる*わけではありません*．代わりに，値 `vec![1, 2, 3]` の*所有権だけ*が `vector` から `moved` に移動します．
| 所有者 | 値 |
| -- | -- |
| `moved` | `vec![1, 2, 3]` |

このことを，「`vector` から `moved` へ値が**ムーブ**された」といいます．

`moved` は所有権を得たので， `moved` を通して値 `vec![1, 2, 3]` を使用することができます．
```rust
fn main() {
    let vector: Vec<i32> = vec![1, 2, 3];
    let moved = vector;
    println!("{:?}", moved); // [1, 2, 3] と出力される
}
```

一方，*所有権を失った変数 `vector` を使用することはできません*．

```rust:コンパイルエラー
fn main() {
    let vector: Vec<i32> = vec![1, 2, 3];
    let moved = vector;
    println!("{:?}", vector); // エラー
}
```

ある値の所有者は，*常に 1 つ*です．メモリ上で同じ場所にある値を複数の変数が所有することはありません．

## スコープとの違い
所有権を失っても，変数 `vector` のスコープが終了するわけではありません．

もし `vector` が可変変数なら，新しい値を代入することで再び使用できるようになります．
```rust
fn main() {
    let mut vector: Vec<i32> = vec![1, 2, 3];
    let moved = vector;
    vector = vec![4, 5, 6]; // 新しいベクタを代入
    println!("{:?}", vector); // [4, 5, 6] と出力される
}
```
値 `vec![1, 2, 3]` が `vector` から `moved` にムーブされた後， `vector` に新しい値 `vec![4, 5, 6]` が代入されています．所有者と値の関係は次のようになります．

| 所有者 | 値 |
| -- | -- |
| `moved` | `vec![1, 2, 3]` |
| `vector` | `vec![4, 5, 6]` |

## 借用
借用はムーブではありません．
```rust
fn main() {
    let vector: Vec<i32> = vec![1, 2, 3];
    let reference = &vector;
    println!("{:?}", vector); // ムーブされていないので使用できる
    println!("{:?}", reference);
}
```
`reference` には， `vector` 自体ではなく `vector` への参照を代入しています． `reference` は，変数 `vector` のアドレスこそ所有していますが，ベクタの中身自体を所有しているわけではありません．よって `vector` は値 `vec![1, 2, 3]` への所有権を持ち続けます．
| 所有者 | 値 |
| -- | -- |
| `vector` | `vec![1, 2, 3]` |
| `reference` | `&vector` |

# ドロップ
`i32` 型の変数 `hoge` を宣言すると，メモリ上に 4 バイトの領域が確保されます．

`hoge` が宣言されていたブロックが終了すると，変数 `hoge` は存在しなくなります．今までこれを「変数 `hoge` のスコープが終了した」という見方でのみ捉えてきました．一方，同じことを変数ではなく値に着目して捉える見方もあります．

`hoge` のスコープが終了するとき， `hoge` の値を保存するために確保されていた 4 バイトの領域は解放され，未使用の状態に戻ります．これを値の**ドロップ**といいます．

```rust
fn main() {
    {
        let hoge = 10;
    } // ドロップ
}
```

ドロップは，変数ではなく値に着目した言い方です．値がコピーされたときは，それぞれに対して別々にドロップが発生します．
```rust
fn main() {
    let hoge = 10; // 1 つめの 10
    {
        let fuga = hoge; // 2 つめの 10
    } // 2 つめの 10 のドロップ
} // 1 つめの 10 のドロップ
```
ブロック内で `hoge` の値が `fuga` にコピーされ，メモリ上には 10 という値が 2 つ存在する状態になります．ブロックが終了するとき， `fuga` が所有している方の 10 がドロップされます．一方， `hoge` が所有している方の 10 は， `main` 関数の終了時にドロップされます．

## ドロップと所有権
変数ではなく値に着目して，ドロップという概念を持ち出したのは，所有権との関わりを考えるためです．

変数のスコープが終了するときにドロップするのは，その変数が所有していた値です．変数が何の値も所有していなければ，ドロップは起こりません．次のコードを見てください．
```rust
fn main() {
    let vector = vec![10, 20, 30];
    {
        let moved = vector;
    } // ドロップ
}
```
ブロック内でベクタの所有権が `vector` から `moved` に移動しています．このブロックが終了するとき， `moved` のスコープが終了します．ここで `moved` は所有権を持っているので，値がドロップされ，中身を格納するために使用されていた領域が解放されて未使用の状態になります．

一方， `main` 関数が終了するときに変数 `vector` のスコープが終了しますが，このとき `vector` は何も所有していないため，ドロップは起こりません．

「値の所有者は常に 1 つ」という規則により，ドロップも必ず 1 つの値に対して 1 度だけ発生します．
# コピー可能な型とそうでない型
今までに登場した型のコピー可能性は，次のようになっています．
- `i8` `i16` `i32` `i64` `i128` `isize` `u8` `u16` `u32` `u64` `u128` `usize` `f32` `f64` `char` は全てコピー可能です（同様に今後登場する `bool` もコピー可能です）．
- `T` に関わらず，不変参照 `&T` はコピー可能です．
- `T` に関わらず，可変参照 `&mut T` はコピー*不可能*です．
- `T` に関わらず，ベクタ `Vec<T>` はコピー*不可能*です．
- 文字列 `String` はコピー*不可能*です．
- `T` がコピー可能ならば，配列 `[T; N]` もコピー可能です．
- `T` `U` ……が*全て*コピー可能ならば，タプル `(T, U, ...)` もコピー可能です．特にユニット `()` はコピー可能です．
- スライスについては，そもそもスライス型の変数を作ることができません．

# ムーブできない場合
## ベクタ
`Vec<Vec<i32>>` という型を考えます．これはベクタのベクタです．

`vec!` マクロを使って `Vec<Vec<i32>>` 型の値を作ると，たとえば次のようになります．
```rust
let vector: Vec<Vec<i32>> = vec![vec![2, 3, 4], vec![1], vec![0; 5]];
```

`vector` の型が `Vec<Vec<i32>>` なので， `vector[0]` `vector[1]` `vector[2]` の型は全て `Vec<i32>` になります． `vector[0]` は `vec![2, 3, 4]` に， `vector[1]` は `vec![1]` に， `vector[2]` は `vec![0; 5]` （`vec![0, 0, 0, 0, 0]`）にそれぞれ等しくなります．よって
```rust
fn main() {
    let vector: Vec<Vec<i32>> = vec![vec![2, 3, 4], vec![1], vec![0; 5]];
    for i in &vector[0] {
        println!("{}", i);
    }
}
```
と書けば
```:stdout
2
3
4
```
と出力されますし，
```rust
fn main() {
    let vector: Vec<Vec<i32>> = vec![vec![2, 3, 4], vec![1], vec![0; 5]];
    for i in &vector {
        for j in i {
            print!("{} ", j);
        }
        println!();
    }
}
```
と書けば
```:stdout
2 3 4 
1 
0 0 0 0 0 
```
と出力されます．

`Vec<T>` の `T` がコピー可能でない場合，*ベクタから要素だけをムーブすることはできません*．よって次のコードはエラーになります．
```rust:コンパイルエラー
fn main() {
    let vector: Vec<Vec<i32>> = vec![vec![2, 3, 4], vec![1], vec![0; 5]];
    let moved = vector[0]; // エラー：ムーブできない
}
```
`vector` というベクタ全体ではなく，その要素である `vector[0]` のみを `moved` にムーブしようとしています． `vector` は，常に「ベクタ全体への所有権を持っている」か「一切所有権を持っていない」のいずれかであり，ベクタの一部を所有していることはありません．

ここで，先ほどのコードの
```rust
for i in &vector {
    for j in i {
        print!("{} ", j);
    }
    println!();
}
```
の部分を，型に着目して見てみましょう． `for` 式では， `in` の後に `&[T; N]` や `&Vec<T>` や `&[T]` を置くと，各要素への参照 `&T` が走査されます．今回， `&vector` の型は `&Vec<Vec<i32>>` なので， `i` の型は `&Vec<i32>` になり， `j` の型は `&i32` になります．

これを，次のように書いてしまうとエラーになります．
```rust:コンパイルエラー
for &i in &vector { // エラー：ムーブ
    for j in &i {
        print!("{} ", j);
    }
    println!();
}
```
`Vec<i32>` 型の変数 `i` に，ベクタ `vector` の要素をムーブすることになるからです．

## 参照
変数が借用されている間は，ムーブを行うことができません．
```rust:コンパイルエラー
fn main() {
    let vector = vec![1, 2, 3];
    let reference = &vector; // 借用
    let moved = vector; // エラー：ムーブできない
    println!("{:?}", reference); // ライフタイムの終了
}
```
ムーブ前は
| 所有者 | 値 |
| -- | -- |
| `vector` | `vec![1, 2, 3]` |
| `reference` | `&vector` |

だったのに対し，ムーブ後は
| 所有者 | 値 |
| -- | -- |
| `moved` | `vec![1, 2, 3]` |
| `reference` | `&vector` |

となって，参照 `reference` の指す先が失われてしまうからです．

同様に，参照の中身をムーブすることもできません．
```rust:コンパイルエラー
fn main() {
    let vector = vec![1, 2, 3];
    let reference = &vector;
    let moved = *reference; // エラー：ムーブできない
}
```
# ムーブが起こるその他の状況
## タプル
タプルの要素にコピー不可能なものが含まれていると，タプル自体もコピー不可能になります．
```rust
fn main() {
    let tuple: (String, f64) = ("hello".to_string(), 0.1); // String はコピー不可能
    let moved = tuple; // ムーブ
}
```
## タプルの生成
タプルを作る際にも，ムーブが起こります．
```rust:コンパイルエラー
fn main() {
    let hello = "hello".to_string();
    let tuple: (String, f64) = (hello, 0.1); // hello がムーブされる
    println!("{}", hello); // エラー
}
```
## 部分的なムーブ
タプルの一部だけをムーブすることが可能です．
```rust
fn main() {
    let tuple: (String, f64) = ("hello".to_string(), 0.1);
    let hello = tuple.0; // 一部をムーブ
}
```
一部の要素がムーブされていても，残りの要素は使用することができます．
```rust
fn main() {
    let tuple: (String, f64) = ("hello".to_string(), 0.1);
    let hello = tuple.0; // 一部をムーブ
    assert_eq!(tuple.1, 0.1); // 使用できる
}
```
しかし，タプル全体を使用することはできなくなります．
```rust:コンパイルエラー
fn main() {
    let tuple: (String, f64) = ("hello".to_string(), 0.1);
    let hello = tuple.0; // 一部をムーブ
    let reference = &tuple; // エラー：借用できない
}
```
## ワイルドカードパターン
ワイルドカードパターン `_` への代入は使用に含まれず，ムーブも起こりません．よって次のコードは正常に動きます．
```rust
fn main() {
    let hello = "hello".to_string();
    let _ = hello;
    println!("{}", hello);
}
```
## match
ここまで `let` による代入のみを例として挙げてきましたが， `match` を用いたパターンマッチでも同様にムーブが起こります．
```rust
fn main() {
    let tuple = (0, String::from("hello"));
    match tuple {
        (0, _) => {
            // tuple.1 はムーブされない
            println!("zero");
        }
        (_, s) => {
            // tuple.1 から s にムーブされる
            println!("{}", s);
        }
    }
    // println!("{}", tuple.1);
    // tuple.1 はムーブされている可能性があるため，
    // 使用できない
}
```
`ref` パターンを使って借用すると，ムーブが起こるのを防げます．
```rust
fn main() {
    let tuple = (10, String::from("hello"));
    match tuple {
        (0, _) => println!("zero"),
        (_, ref s) => {
            // s には &tuple.1 が代入され，
            // ムーブは起こらない
            println!("{}", s);
        }
    }
    println!("{}", tuple.1);
}
```
## ブロックが返す値とムーブ
ブロックが値を返す際にも，ムーブが発生します．次のコードを見てください．
```rust
fn main() {
    let power_of_2 = {
        let mut v = vec![1];
        for i in 0..4 {
            v.push(v[i] * 2);
        }
        v
    };
    assert_eq!(power_of_2, vec![1, 2, 4, 8, 16]);
}
```
ブロックの中で，可変変数 `v` の型は `Vec<i32>` です． `for` 式を使って `v` に 2, 4, 8, 16 を追加した後，ブロックの最後で `v` を返しています．このとき，ベクタは `v` から `power_of_2` へとムーブされます．

# 確認問題
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let vector = vec![1, 2, 3];
       let moved = vector;
       let moved_2 = vector;
   }
   ```
   :::details 答え
   `let moved_2 = vector;` で**エラー**になります．ムーブされた後の `vector` を使用しようとしているためです．
   :::
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let vector = vec![1, 2, 3];
       let moved = vector;
       let reference = &vector;
   }
   ```
   :::details 答え
   `&vector` で**エラー**になります．ムーブ後は，使用も借用もできません．
   :::
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let vector = vec![1, 2, 3];
       let moved = vector;
       let _ = vector;
   }
   ```
   :::details 答え
   **動きます**． `vector` がムーブされた後でも，ワイルドカードパターン `_` への代入は可能です．
   :::
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let tuple = (vec![1, 2, 3], vec![4, 5, 6]);
       let (former, _) = tuple;
       let (_, latter) = tuple;
   }
   ```
   :::details 答え
   **動きます**．
   
   `let (former, _) = tuple;` では， `tuple.0` が `former` にムーブされます．このときタプルパターンの中の 2 つめはワイルドカードパターンなので， `tuple.1` はムーブされずに残ります．
   
   `let (_, latter) = tuple;` では， `tuple.1` が `latter` にムーブされます． `tuple.0` は既にムーブされていますが，対応する左辺がワイルドカードパターンなので大丈夫です．
   :::
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let reference;
       let vector = {
           let v = vec![10, 20, 30];
           reference = &v;
           v
       };
   }
   ```
   :::details 答え
   **動きます**．
   
   `v` から `vector` へムーブが起こりますが，*このとき既に `reference` のライフタイムは終了しています*．
   
   ブロックの後に `println!("{:?}", reference);` などを付け加えるとエラーになります．
   :::
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let reference;
       let vector = {
           let v = vec![10, 20, 30];
           reference = &v;
           *reference
       };
   }
   ```
   :::details 答え
   **エラー**になります． `reference` のライフタイムが， `v` のスコープをはみ出そうとしています．
   :::
1. 次のコードは正常に動くでしょうか．
   ```rust
   fn main() {
       let vector = vec![
           (vec![5, 2], 3.5),
           (vec![1, 4, 3, 3], 2.75),
           (vec![4, 6], 5.),
       ];
       for &(v, mean) in &vector {
           println!("{:?}: {}", v, mean);
       }
   }
   ```
   :::details 答え
   **エラー**になります．
   
   `vector[0].0` / `vector[1].0` / `vector[2].0` から `vec` へベクタをムーブしようとしているためです．パターンを `&(ref v, mean)` あるいは `(v, mean)` に変えると動きます．
   :::
# 練習問題
- [ABC186 B - Blocks on Grid](https://atcoder.jp/contests/abc186/tasks/abc186_b) / `as` を使うと，型を変換できます．[解答例](https://atcoder.jp/contests/abc186/submissions/19443273)

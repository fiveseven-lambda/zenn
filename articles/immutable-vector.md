---
title: "誰でも書けるスクリプト言語は、immutable な配列を提供すればよかった"
emoji: "😺"
type: "idea"
topics: ["Python"]
published: true
---

# 突然ですが、

Python あるある〜。

```python
a = [1, 2, 3]
b = a # コピー？
c = a # コピー？

b.append(4) # 追加
c.append(5) # 追加

print(b)
# [1, 2, 3, 4, 5]
# c に追加したはずの 5 が
# b にも追加されていて、ビビる

print(c)
# [1, 2, 3, 4, 5]
# b に追加したはずの 4 が
# c にも追加されていて、ビビる
```

:::details JavaScript 版もあります
```javascript
const a = [1, 2, 3]
const b = a // コピー？
const c = a // コピー？

b.push(4) // 追加
c.push(5) // 追加

console.log(b)
/* [ 1, 2, 3, 4, 5 ]
c に追加したはずの 5 が
b にも追加されていて、ビビる */

console.log(c)
/* [ 1, 2, 3, 4, 5 ]
b に追加したはずの 4 が
c にも追加されていて、ビビる */
```
:::

その日人類は困りました。

# C++ はどうか
ビビりません。
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector a { 1, 2, 3 };
    auto b = a, c = a; // コピー
    
    b.push_back(4); // 追加
    c.push_back(5); // 追加
    
    for (int &i : b) std::cout << i << std::endl;
    // 1
    // 2
    // 3
    // 4
    std::cout << std::endl;

    for (int &i : c) std::cout << i << std::endl;
    // 1
    // 2
    // 3
    // 5
}
```

これは `std::vector` が deep copy だからです。

## ならば軽率にコピーしまくろう

いいえ。Deep copy にかかる時間は**要素数に比例**するため、予期せぬ計算量の悪化を招くことがあります。たとえば、

```cpp
int binary_search(std::vector<int> vec, int target) {
    // 対数時間で高速！
    // ……ではない
}
```

この関数は、長さ $n$ の `std::vector` を受け取った時点で時間計算量 $\Theta(n)$ です。
二分探索の計算量 $\Theta(\log n)$ にしたければ、こう：

```cpp
int binary_search(const std::vector<int> &vec, int target)
```

Python や JavaScript でビビるのを防ごうとしたら、今度は `&` の有無を意識する必要が生じてしまいました！

C++ を書く人なら大体パフォーマンスのことも考えられるでしょう。でも「誰でも書けるスクリプト言語」にこの手間があると嫌に思う人もいそうです。

# Rust はいいぞおじさん、現る

上の話をすると必ず現れるおじさんがいて、こんなコードを見せてくれます。

```rust
let a = vec![1, 2, 3];
let b = a;
let c = a; // エラー
```

3 行目から早くもエラー！？ただしヘルプが出てきます：

```
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let b = a.clone();
  |              ++++++++
```

> パフォーマンスの悪化が許容できるなら、`clone` はいかがですか？

なるほど、Python や JavaScript のような苦しみは避けつつ、計算量の悪化に気付けるんですね。

でも、Rust はこれが至るところで起こります。
```rust
let b = a; // 代入すれば a を奪われる……
foo(a); // 関数に渡せば a を奪われる……
dbg!(a); // デバッグすれば a を奪われる……
a; // 何なら書くだけで奪われる……
```
（念のため付け加えておきますが、私は好きですよ。Rust）
（以前、RustCoder というのを書いていました）
（Rust のやさしい解説です）
（Rust 好きです。信じて下さい）

どうやら、Rust を書くには、コンパイラとゆっくり話し合う時間が必要そうです。でも私は今すぐ動くコードが必要なんです！

もし 1970 年代の AT&T ベル研究所で Rust が覇権を取っていれば、「苦しんで覚える C 言語」は「苦しんで覚える Rust」になり、ムーブを理解することが全プログラマーにとっての第一歩になっていたかもしれません。しかし残念ながら今はそうじゃありません。Rust の挙動は、他の多くのプログラミング言語と異なります。

OS が作りたい？ Rust は良い選択肢でしょう。
言語処理系が書きたい？ Rust は良い選択肢でしょう。
私は応用数学が専門で、プログラミングはからっきしなのだが、さっきシャワーと一緒に頭に降ってきたアイディアを今すぐ試したい？うーん……

「誰でも書けるスクリプト言語」にしては考えることが多そうです。

# ここで Scala を見てみましょう

```scala
val a = Vector(1, 2, 3)

val b = a :+ 4
val c = a :+ 5

println(b)
// Vector(1, 2, 3, 4)
// ビビらない！

println(c)
// Vector(1, 2, 3, 5)
// ビビらない！
```

Scala の Vector は immutable です。つまり、`a` に直接要素を追加はできません。

代わりに、`:+` 演算子で**要素を追加した新しい Vector** を返します。

これを標準で備えていれば、Python や JavaScript のような混乱は起きません！

「誰でも書けるスクリプト言語」は、この immutable な配列を用意すればよかったんですね。

# 頂いたコメントをもとに追記

色々な方から他の言語についても教えていただきました。

## Swift

Swift は copy on write なのだそうです。

```swift
let a = [1, 2, 3]

var b = a
var c = a

b.append(4) // copy on write
c.append(5) // copy on write

print(b) // [1, 2, 3, 4]
print(c) // [1, 2, 3, 5]
```
書き換えない限りコピーは起こりません。
```swift
// O(log n)
func binarySearch(array: [Int], target: Int) -> Int {
    // array を書き換えなければ、コピーされない
}
```

## R

存じませんでした……。

```r
a <- c(1, 2, 3)

b <- c(a, 4)
c <- c(a, 5)

print(b) # [1] 1 2 3 4
print(c) # [1] 1 2 3 5
```

## PHP

これも copy on write らしいです

```php
$a = [1, 2, 3];

$b = $a;
$c = $a;

$b[] = 4;
$c[] = 5;

foreach ($b as $i) echo "$i<br>";
// 1<br>2<br>3<br>4<br>
foreach ($c as $i) echo "$i<br>";
// 1<br>2<br>3<br>5<br>
```

# 結論

Immutable な配列を標準装備すればよかった。


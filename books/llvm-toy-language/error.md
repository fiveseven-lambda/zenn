---
title: "エラー報告"
---
# Pos
エラーが起こったとき，どこでなぜエラーが起こったのか教えてもらえないととても嫌な気持ちになる．

じゃあその「場所」はどうやって表そうか，となる．大抵の言語は，何行目の何バイト目，のように教えてくれると思う．じゃあそれを表現するクラスを作ろう．
```cpp:pos.hpp
#include <cstddef>

namespace pos {
    class Pos {
        std::size_t line, byte;
    public:
        Pos();
        Pos(std::size_t, std::size_t);
    };
}
```
```cpp:pos.cpp
#include "pos.hpp"

namespace pos {
    Pos::Pos(): line(0), byte(0) {}
    Pos::Pos(std::size_t line, std::size_t byte): line(line), byte(byte) {}
}
```
あっ，インクルードガードはここでは省略する．実際のコードではちゃんと書いてる．

`line`，`byte` は 0-indexed で持つことにする．ソースコードの先頭は 0 行目 0 文字目だ．でも，エラーメッセージまで 0-indexed だと嫌そう（エディタの行数表示は普通 1-indexed だよね）なので，出力のときに 1-indexed に直す．いずれ `operator<<` を定義しよう．

コンストラクタはなんとなく要りそうなものを書いている．もしかしたら後で変えるかもしれない．
# Range
`Pos` を使うとある 1 文字の位置を表現できる．一方で，式みたいな複数文字にわたるものの場所は「ここからここまで」みたく範囲で表される．

じゃあそれは `Pos` 2 つの組で表そう．
```cpp:pos.hpp
namespace pos {
    class Range {
        Pos start, end;
    public:
        Range();
        Range(Pos, Pos);
    };
}
```
```cpp:pos.cpp
namespace pos {
    Range::Range() = default;
    Range::Range(Pos start, Pos end): start(start), end(end) {}
}
```
区間を持つとき，開区間，閉区間，半開区間のどれが一番いいだろうか．これは後で lexer を書くときに分かって，半開区間がいい．`start` は含み，`end` は含まない．これも，エラーメッセージまで半開区間だとたぶん嫌なので，出力のときに閉区間に直す．

`Pos` と同様，コンストラクタは何が必要になるかまだよく分からない．

# eprint
正直，コンパイラに何行目何文字目って数字だけ言われてもパッと分からないこともあると思う（行数の表示されないエディタを使っていたり？）．エラーメッセージ自体にソースコードの一部があった方がなんかいい．

`Pos` や `Range` は，何行目何文字目っていう数字を持っている．じゃあ，ソースコードを読みながら全体をどこかに保存しておいて，`Pos` や `Range` に渡すことでその中から当該の部分文字列を切り取ってくれるようにしたらいいんじゃないか？入力は対話っぽく stdin で与えられるかもしれないので，あとで見たいなら変数に保存しておくしかない．

今回はソースコードを 1 行ずつに分けて `std::vector<std::string>` に入れておこうと思う．以下を追記だ．

```cpp:pos.hpp
#include <iostream>
#include <vector>
#include <string>

namespace pos {
    class Pos {
        /*略 */
        void eprint(const std::vector<std::string> &) const;
    };

    class Range {
        /*略 */
        void eprint(const std::vector<std::string> &) const;
    };
}
```
中身は省略．好みで出力する．

`Range::eprint()` の中で `Pos` の private メンバ `line` `byte` を使いたいかもしれない．その場合たとえば `std::pair<std::size_t, std::size_t> Pos::into_inner() const` を用意するなりなんなりすればいい．

# Error
エラーはいろんなエラーがある．持ちたいメンバも，エラーの種類ごとに違いそうだ．

そういうときは抽象クラスにする．
```cpp:error.hpp
namespace error {
    class Error {
    public:
        virtual ~Error();
        void virtual eprint(const std::vector<std::string> &) const = 0;
    };
}
```
```cpp:error.cpp
#include "error.hpp"

namespace error {
    Error::~Error() = default;
}
```
`void eprint(const std::vector<std::string> &)` がエラーメッセージを stderr に出力する関数．メンバとして `Pos` や `Range` を持っているエラーであれば，引数の `std::vector<std::string>` をそいつに渡して場所を出力させるだろう．
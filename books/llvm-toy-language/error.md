---
title: "エラー報告"
---
# Pos
エラーが起こったとき，どこでなぜエラーが起こったのか教えてもらえないととても嫌な気持ちになる．

じゃあその「場所」はどうやって表そうか，となる．大抵の言語は，何行目の何バイト目，のように教えてくれると思う．じゃあそれを表現するクラスを作ろう．
```cpp:pos.hpp
#include <cstddef>
#include <iostream>
#include <utility>

namespace pos {
    class Pos {
        std::size_t line, byte;
    public:
        Pos();
        Pos(std::size_t, std::size_t);
        std::pair<std::size_t, std::size_t> into_inner() const;
        friend std::ostream &operator<<(std::ostream &os, const Pos &);
    };
}
```
```cpp:pos.cpp
#include "pos.hpp"

namespace pos {
    Pos::Pos(): line(0), byte(0) {}
    Pos::Pos(std::size_t line, std::size_t byte): line(line), byte(byte) {}

    std::pair<std::size_t, std::size_t> Pos::into_inner() const {
        return {line, byte};
    }

    std::ostream &operator<<(std::ostream &os, const Pos &pos){
        return os << pos.line + 1 << ":" << pos.byte + 1;
    }
}
```
あっ，インクルードガードはここでは省略する．実際のコードではちゃんと書いてる．

`line`，`byte` は 0-indexed で持つことにする．ソースコードの先頭は 0 行目 0 バイト目だ．でも，エラーメッセージまで 0-indexed だと嫌そう（エディタの行数表示は普通 1-indexed だよね）なので，出力のときに 1-indexed に直す．

コンストラクタはなんとなく要りそうなものを書いている．もしかしたら後で変えるかもしれない．

`into_inner` は中身を `std::pair<std::size_t, std::size_t>` にして返す．何かと使える．
# Range
`Pos` を使うとある 1 文字の位置を表現できる．一方で，式みたいな複数文字にわたるものの場所は「ここからここまで」みたく範囲で表される．

じゃあそれは `Pos` 2 つの組で表そう．
```cpp:pos.hpp
namespace pos {
    class Range {
        Pos start, end;
    public:
        Range();
        Range(std::size_t, std::size_t, std::size_t);
        friend std::ostream &operator<<(std::ostream &os, const Range &);
    };
}
```
```cpp:pos.cpp
namespace pos {
    Range::Range() = default;
    Range::Range(std::size_t line, std::size_t start, std::size_t end):
        start(line, start),
        end(line, end) {}

    std::ostream &operator<<(std::ostream &os, const Range &range){
        auto [sline, sbyte] = range.start.into_inner();
        auto [eline, ebyte] = range.end.into_inner();
        return os
            << sline + 1 << ":" << sbyte + 1
            << "-" << eline + 1 << ":" << ebyte;
    }
}
```
区間を持つとき，開区間，閉区間，半開区間のどれが一番いいだろうか．これは後で lexer を書くときに分かって，半開区間がいい．`start` は含み，`end` は含まない．これも，エラーメッセージまで半開区間だとたぶん嫌なので，出力のときに閉区間に直す．

# eprint
正直，コンパイラに何行目何文字目って数字だけ言われてもパッと分からないこともあると思う（行数の表示されないエディタを使っていたり？）．エラーメッセージ自体にソースコードの一部があった方がなんかいい．

`Pos` や `Range` は，何行目何文字目っていう数字を持っている．じゃあ，ソースコードを読みながら全体をどこかに保存しておいて，`Pos` や `Range` に渡すことでその中から当該の部分文字列を切り取ってくれるようにしたらいいんじゃないか？入力は対話っぽく stdin で与えられるかもしれないので，あとで見たいなら変数に保存しておくしかない．

今回はソースコードを 1 行ずつに分けて `std::vector<std::string>` に入れておこうと思う．以下を追記だ．

```cpp:pos.hpp
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
#include <vector>
#include <string>
#include <memory>

namespace error {
    class Error {
    public:
        virtual ~Error();
        void virtual eprint(const std::vector<std::string> &) const = 0;
    };

    template<class Err, class... Args>
    std::unique_ptr<Error> make(Args&&... args){
        return std::make_unique<Err>(std::forward<Args>(args)...);
    }
}
```
```cpp:error.cpp
#include "error.hpp"

namespace error {
    Error::~Error() = default;
}
```
`eprint()` がエラーメッセージを stderr に出力する関数となる．メンバとして `Pos` や `Range` を持っているエラーであれば，`Pos::eprint()` や `Range::eprint()` に引数の `std::vector<std::string>` を渡して場所を出力させるだろう．

`make()` はヘルパ関数だ．これを使えば，`static_cast<error::Error>(std::make_unique<error::Foo>(bar))` が単に `error::make<error::Foo>(bar)` と書けて楽になる．
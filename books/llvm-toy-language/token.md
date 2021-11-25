---
title: "トークン"
---

字句解析のために，まずはトークンを定義する．

# Token
まずは基底クラスから作る．

```cpp:token.hpp
namespace token {
    class Token {
    public:
        virtual ~Token();
    };
}
```
```cpp:token.cpp
#include "token.hpp"

namespace token {
    Token::~Token() = default;
}
```

# Identifier
識別子は，正規表現 `[A-Za-z_][A-Za-z_0-9]*` に合致するものとする．

```cpp:token.hpp
namespace token {
    class Identifier : public Token {
        std::string name;
    public:
        Identifier(std::string &&);
    };
}
```
```cpp:token.cpp
namespace token {
    Identifier::Identifier(std::string &&name): name(std::move(name)) {}
}
```
# Integer
整数リテラルは `[0-9]+`．

整数なのにメンバ `value` の型は `std::string`．あとでこれを 32 ビット整数値に直すとき `2147483648` 以上はエラーとするけれど，前に `-` が付いた `-2147483648` は ok とする．
```cpp:token.hpp
namespace token {
    class Integer : public Token {
        std::string value;
    public:
        Integer(std::string &&);
    };
}
```
```cpp:token.cpp
namespace token {
    Integer::Integer(std::string &&value): value(std::move(value)) {}
}
```
# 記号類
何があるといい？
- 四則演算 `+` `-` `*` `/` `%`
- ビット演算 `&` `|` `^` `~` `<<` `>>`
- 代入 `=` `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `<<=` `>>=`
- 論理演算 `&&` `||` `!`
- 比較 `==` `!=` `<` `>` `<=` `>=`
- 括弧 `(` `)` `{` `}` `[` `]`
- 区切り文字 `.` `:` `;` `,`
```cpp:token.hpp
namespace token {
    class Plus : public Token {};
    class Hyphen : public Token {};
    class Asterisk : public Token {};
    class Slash : public Token {};
    /* 略 */
}
```
いずれここにたくさん純粋仮想関数を追加する．

使える文字の中ではあと `#` と `?` が残ってる．
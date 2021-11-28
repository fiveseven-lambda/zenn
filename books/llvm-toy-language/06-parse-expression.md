---
title: "構文解析 (式)"
---

式はこんな感じの BNF で表される．
```
<factor> ::= <identifier>
           | <integer>
           | <unary operator> <factor>
           | `(` <expression> `)`
           | <factor> `(` <arguments list> `)`
<expression> ::= <factor>
               | <expression> <binary operator> <factor>
<arguments list> ::= ε
                   | <expression>
                   | <expression> `,` <arguments list>
```
これをパースする関数を作っていく．
```cpp:parser.hpp
#include <memory>
#include <vector>

#include "syntax.hpp"
#include "lexer.hpp"

namespace parser {
    std::unique_ptr<syntax::Expression>
        parse_factor(lexer::Lexer &),
        parse_binary_operator(lexer::Lexer &),
        parse_expression(lexer::Lexer &);
    std::vector<std::unique_ptr<syntax::Expression>>
        parse_arguments(lexer::Lexer &);
}
```

# factor
まず，factor をパースする `parse_factor()` 関数から書いていく．

`lexer` からトークンを 1 個 peek し，そいつに基づいて `|` で繋がれた 5 通りのうちのどれかを選択する．
```cpp:parser.cpp
#include "parser.hpp"

namespace parser {
    std::unique_ptr<syntax::Expression> parse_factor(lexer::Lexer &lexer){
        auto &token_ref = lexer.peek();

        // EOF に達した場合
        if(!token_ref) return nullptr;

        /* ここで factor を返す */
    }
}
```

## Identifier
1 個目のトークンが Identifier なら，1 つめの規則
```
<factor> ::= <identifier>
```
が選ばれる．

`token::Token` に，識別子ならその名前を返すような関数 `identifier()` を追加する．
```cpp:token.hpp
#include <optional>

namespace token {
    class Token {
        /* 略 */
    public:
        virtual std::optional<std::string> identifier();
    };
    
    class Identifier : public Token {
        /* 略 */
    public:
        std::optional<std::string> identifier() override;
    };
}
```
```cpp:token.cpp
namespace token {
    std::optional<std::string> Token::identifier(){
        return std::nullopt;
    }
    std::optional<std::string> Identifier::identifier(){
        return std::move(name);
    }
}
```
これを用いて `parse_factor()` に 1 つめの規則を追加する．
```cpp:parser.cpp
std::unique_ptr<syntax::Expression> parse_factor(lexer::Lexer &lexer){
    auto &token_ref = lexer.peek();
    if(!token_ref) return nullptr;

    std::unique_ptr<syntax::Expression> ret;
    pos::Range pos;
    if(auto name = token_ref->identifier()){
        // peek した 1 個目のトークンは識別子だった．

        // peek していたものを読むと同時に，位置情報 pos をもらう
        pos = lexer.next()->pos;

        // syntax::Identifier を作り，返り値に代入
        ret = std::make_unique<syntax::Identifier>(std::move(name).value());
    }else{
        // 残り 4 つの規則
    }

    // 位置情報を付与
    ret->pos = pos;

    return ret;
}
```

## Positive Integer
1 個目のトークンが Integer なら，2 つめの規則
```
<factor> ::= <integer>
```
が選ばれる．

`token::Token` に，整数リテラルならその値を `std::int32_t` で返すような関数 `positive_integer()` を追加する．
```cpp:token.hpp
#include <cstddef>

namespace token {
    class Token {
        /* 略 */
    public:
        virtual std::optional<std::int32_t> positive_integer();
    };
    
    class Identifier : public Token {
        /* 略 */
    public:
        std::optional<std::int32_t> positive_integer() override;
    };
}
```
オーバーフローの検出には Boost の Safe Numerics を使う．
```cpp:token.cpp
#include <boost/safe_numerics/safe_integer.hpp>

namespace token {
    using safe_i32 = boost::safe_numerics::safe<std::int32_t>;
    std::optional<std::int32_t> Token::positive_integer(){
        return std::nullopt;
    }
    std::optional<std::int32_t> Integer::positive_integer(){
        safe_i32 ret(0);
        constexpr safe_i32 base(10);
        try{
            for(char c : value) ret = ret * base + safe_i32(c - '0');
        }catch(std::exception &e){
            throw error::make<error::InvalidIntegerLiteral>(e, pos);
        }
        return ret;
    }
}
```
ただし `error::InvalidIntegerLiteral` は次のように定義．
```cpp:error.hpp
namespace error {
    class InvalidIntegerLiteral : public Error {
        std::exception &error;
        pos::Range pos;
    public:
        InvalidIntegerLiteral(std::exception &, pos::Range);
        void eprint(const std::vector<std::string> &) const override;
    };
```

これを用いて，上と同様に `parse_factor()` に 2 個目の規則を追加．
```cpp:parser.cpp
std::unique_ptr<syntax::Expression> parse_factor(lexer::Lexer &lexer){
    auto &token_ref = lexer.peek();
    if(!token_ref) return nullptr;

    std::unique_ptr<syntax::Expression> ret;
    pos::Range pos;
    if(auto name = token_ref->identifier()){
        // 略
    }else if(auto value = token_ref->positive_integer()){
        // 1 個目のトークンは整数リテラルだった
        pos = lexer.next()->pos;
        ret = std::make_unique<syntax::Integer>(value.value());
    }
    ret->pos = pos;
    return ret;
}
```
## prefix
1 個目のトークンが Integer なら，3 つめの規則
```
<factor> ::= <unary operator> <factor>
```
が選ばれる．

`token::Token` に，前置演算子なら対応する `syntax::UnaryOperator` を返すような関数 `prefix()` を追加する．

```cpp:token.hpp
#include "syntax.hpp"

namespace token {
    class Token {
        // 略
    public:
        virtual std::optional<syntax::UnaryOperator> prefix();
    }
    class Plus : public Token {
        // 略
        std::optional<syntax::UnaryOperator> prefix() override;
    };
    class Hyphen : public Token {
        // 略
        std::optional<syntax::UnaryOperator> prefix() override;
    };
    class Tilde : public Token {
        // 略
        std::optional<syntax::UnaryOperator> prefix() override;
    };
    class Exclamation : public Token {
        // 略
        std::optional<syntax::UnaryOperator> prefix() override;
    };
}
```
```cpp:token.cpp
namespace token {
    std::optional<syntax::UnaryOperator> Token::prefix(){
        return std::nullopt;
    }
    std::optional<syntax::UnaryOperator> Plus::prefix(){
        return syntax::UnaryOperator::Plus;
    }
    std::optional<syntax::UnaryOperator> Hyphen::prefix(){
        return syntax::UnaryOperator::Minus;
    }
    std::optional<syntax::UnaryOperator> Tilde::prefix(){
        return syntax::UnaryOperator::BitNot;
    }
    std::optional<syntax::UnaryOperator> Exclamation::prefix(){
        return syntax::UnaryOperator::LogicalNot;
    }
}
```

ここで，演算子の位置情報とオペランドの位置情報を合わせて式全体の位置情報とする必要がある．そこで，`pos::Range` に次のような演算子をオーバーロードする．

```cpp:pos.hpp
namespace pos {
    class Range {
        // 略
    public:
        Range &operator+=(const Range &);
    };
}
```
```cpp:pos.cpp
namespace pos {
    Range &Range::operator+=(const Range &other){
        end = other.end;
        return *this;
    }
}
```
すると，`parse_factor` の 3 個目の規則は次のように書ける．
```cpp:parse.cpp
std::unique_ptr<syntax::Expression> parse_factor(lexer::Lexer &lexer){
    auto &token_ref = lexer.peek();
    if(!token_ref) return nullptr;
    std::unique_ptr<syntax::Expression> ret;
    pos::Range pos;
    if(auto name = token_ref->identifier()){
        // 略
    }else if(auto value = token_ref->positive_integer()){
        // 略
    }else if(auto prefix = token_ref->prefix()){
        // 演算子の位置
        pos = lexer.next()->pos;

        // parse_factor() を再帰呼出ししてオペランドを得る
        auto operand = parse_factor(lexer);

        // オペランドの位置情報を合わせて，全体の位置情報を得る
        pos += operand->pos;

        ret = std::make_unique<syntax::Unary>(prefix.value(), std::move(operand));
    }
    ret->pos = pos;
    return ret;
}
```
ただし，これだと -2147483648 がエラーになる．
```
> -2147483648
invalid integer literal (addition result too large: positive overflow error) at 1:2-1:11
- !-> 2147483648 <-!
```
よって，`-` の直後に整数リテラルが来る場合のみ別にする．`token::Token::positive_integer()` と同様に今度は `token::Token::negative_integer()` を用意して，
```cpp:token.hpp
namespace token {
    class Token {
        /* 略 */
    public:
        virtual std::optional<std::int32_t> negative_integer();
    };
    
    class Identifier : public Token {
        /* 略 */
    public:
        std::optional<std::int32_t> negative_integer() override;
    };
}
```
```cpp:token.cpp
namespace token {
    std::optional<std::int32_t> Token::negative_integer(){
        return std::nullopt;
    }
    std::optional<std::int32_t> Integer::negative_integer(){
        safe_i32 ret(0);
        constexpr safe_i32 base(10);
        try{
            for(char c : value) ret = ret * base - safe_i32(c - '0');
        }catch(std::exception &e){
            throw error::make<error::InvalidIntegerLiteral>(e, pos);
        }
        return ret;
    }
}
```
`-` の直後に整数リテラルが来るときのみこれを使うようにする．
```cpp:parse.cpp
std::unique_ptr<syntax::Expression> parse_factor(lexer::Lexer &lexer){
    auto &token_ref = lexer.peek();
    if(!token_ref) return nullptr;
    std::unique_ptr<syntax::Expression> ret;
    pos::Range pos;
    if(auto name = token_ref->identifier()){
        // 略
    }else if(auto value = token_ref->positive_integer()){
        // 略
    }else if(auto prefix = token_ref->prefix()){
        pos = lexer.next()->pos;
        if(
            prefix.value() == syntax::UnaryOperator::Minus
            && (value = lexer.peek()->negative_integer())
        ){
            // 整数リテラルを peek 中

            // 整数リテラル（オペランド）の位置情報を追加
            pos += lexer.next()->pos;

            ret = std::make_unique<syntax::Integer>(value.value());
        }else{
            auto operand = parse_factor(lexer);
            pos += operand->pos;
            ret = std::make_unique<syntax::Unary>(prefix.value(), std::move(operand));
        }
    }
    ret->pos = pos;
    return ret;
}
```

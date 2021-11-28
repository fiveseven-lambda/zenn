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
    if(auto name = token_ref->identifier()){
        // peek した 1 個目のトークンは識別子だった．

        // peek していたものを読むと同時に，位置情報 pos をもらう
        pos::Range pos = lexer.next()->pos;

        // syntax::Identifier を作り，返り値に代入
        ret = std::make_unique<syntax::Identifier>(std::move(name).value());

        // 位置情報を付与
        ret->pos = pos;
    }else{
        // 残り 4 つの規則
    }
    return ret;
}
```

# Positive Integer
1 個目のトークンが Integer なら，1 つめの規則
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
    std::optional<std::int32_t> Token::positive_integer(){ return std::nullopt; }
    std::optional<std::int32_t> Integer::positive_integer(){
        safe_i32 ret(0);
        constexpr safe_i32 base(10);
        try{
            for(char c : value){
                ret = ret * base + safe_i32(c - '0');
            }
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
    if(auto name = token_ref->identifier()){
        // 略
    }else if(auto value = token_ref->positive_integer()){
        // 1 個目のトークンは整数リテラルだった

        pos::Range pos = lexer.next()->pos;
        ret = std::make_unique<syntax::Integer>(value.value());
        ret->pos = pos;
    }
    return ret;
}
```
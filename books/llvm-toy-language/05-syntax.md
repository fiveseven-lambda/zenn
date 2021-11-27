---
title: "構文木"
---

構文解析のために，まずは AST を定義する．

# 式 (expression)
## Expression
まずは基底クラスから作る．
```cpp:syntax.hpp
#include <memory>
#include <utility>

#include "pos.hpp"

namespace syntax {
    class Expression {
    public:
        virtual ~Expression();
        virtual void debug_print(int depth = 0) const = 0;
    };
    using ExpressionWithPos = std::pair<pos::Range, std::unique_ptr<Expression>>;
}
```
```cpp:syntax.cpp
#include "syntax.hpp"

namespace syntax {
    Expression::~Expression() = default;
}
```
`debug_print()` はデバッグ出力用．式はネストするので，デバッグ出力時は `debug_print()` が再帰的に呼ばれる感じになる．引数 `depth` はその再帰の深さで，出力をインデントするのに使う．イメージとしては，`(1 + 2) * (3 - 4)` が
```
Mul
    Add
        1
        2
    Sub
        3
        4
```
とか
```
        1
    Add
        2
Mul
        3
    Sub
        4
```
みたくなってほしい．

## Identifier
単一の識別子は，式．
```cpp:syntax.hpp
#include <string>

namespace syntax {
    class Identifier : public Expression {
        std::string name;
    public:
        Identifier(std::string);
        void debug_print(int) const override;
    };
}
```
```cpp:syntax.cpp
#include <iostream>

namespace syntax {
    Identifier::Identifier(std::string name): name(std::move(name)) {}

    void Identifier::debug_print(int) const {
        std::cout << "Identifier(" << name << ")" << std::endl;
    }
}
```
## Integer
単一の整数リテラルも，式．
```cpp:syntax.hpp
#include <cstddef>

namespace syntax {
    class Integer : public Expression {
        std::int32_t value;
    public:
        Integer(std::int32_t);
        void debug_print(int) const override;
    };
}
```
```cpp:syntax.cpp
namespace syntax {
    Integer::Integer(std::int32_t value): value(value) {}

    void Integer::debug_print(int) const {
        std::cout << "Integer(" << value << ")" << std::endl;
    }
}
```
## Unary
式の前に単項演算子が付いたものも式．

まずは演算子を表す `enum class` を定義して，
```cpp:syntax.hpp
namespace syntax {
    enum class UnaryOperator {
        Plus, Minus,
        Not, BitNot
    };
}
```
演算子と式の組として定義する．
```cpp:syntax.hpp
namespace syntax {
    class Unary : public Expression {
        UnaryOperator unary_operator;
        std::unique_ptr<Expression> operand;
    public:
        Unary(UnaryOperator, std::unique_ptr<Expression>);
        void debug_print(int) const override;
    };
}
```
```cpp:syntax.cpp
namespace syntax {
    Unary::Unary(
        UnaryOperator unary_operator,
        std::unique_ptr<Expression> operand
    ):
        unary_operator(unary_operator),
        operand(std::move(operand)) {}

    static constexpr char INDENT[] = "    ";
    void Unary::debug_print(int depth) const {
        std::string_view name;
        switch(unary_operator){
            case UnaryOperator::Plus: name = "plus"; break;
            case UnaryOperator::Minus: name = "minus"; break;
            case UnaryOperator::Not: name = "not"; break;
            case UnaryOperator::BitNot: name = "bitwise not";
        }
        for(int i = 0; i < depth; ++i) std::cout << INDENT;
        std::cout << "Unary(" << name << ")" << std::endl;
        operand->debug_print(depth + 1);
    }
}
```
`debug_print()` の実装は以後省略．
## Binary
二項演算子も同様に定義する．
```cpp:syntax.hpp
namespace syntax {
    enum class BinaryOperator {
        Mul, Div, Rem,
        Add, Sub,
        LeftShift, RightShift,
        BitAnd, BitOr, BitXor,
        Equal, NotEqual,
        Less, Greater,
        LessEqual, GreaterEqual,
        LogicalAnd, LogicalOr,
        Assign,
        MulAssign, DivAssign, RemAssign, AddAssign, SubAssign,
        BitAndAssign, BitOrAssign, BitXorAssign
    };
    class Binary : public Expression {
        BinaryOperator binary_operator;
        std::unique_ptr<Expression> left, right;
    public:
        Binary(BinaryOperator, std::unique_ptr<Expression>, std::unique_ptr<Expression>);
        void debug_print(int) const override;
    };
}
```
```cpp:syntax.cpp
namespace syntax {
    Binary::Binary(
        BinaryOperator binary_operator,
        std::unique_ptr<Expression> left,
        std::unique_ptr<Expression> right
    ):
        binary_operator(binary_operator),
        left(std::move(left)),
        right(std::move(right)) {}
}
```
## Invocation
関数の呼び出しは，呼び出される関数と，引数の `std::vector` の組として表す．
```cpp:syntax.hpp
namespace syntax {
    class Invocation : public Expression {
        std::unique_ptr<Expression> function;
        std::vector<std::unique_ptr<Expression>> arguments;
    public:
        Invocation(std::unique_ptr<Expression>, std::vector<std::unique_ptr<Expression>>);
        void debug_print(int) const override;
    };
}
```
```cpp:syntax.cpp
namespace syntax {
    Invocation::Invocation(
        std::unique_ptr<Expression> function,
        std::vector<std::unique_ptr<Expression>> arguments
    ):
        function(std::move(function)),
        arguments(std::move(arguments)) {}
}
```
# 文
# 型
## Type
型の基底クラス．
```cpp:syntax.hpp
namespace syntax {
    class Type {
    public:
        virtual ~Type();
        virtual void debug_print() const = 0;
    };
}
```
## Primitive
プリミティブ型．
```cpp:syntax.hpp
namespace syntax {
    enum class PrimitiveType {
        Integer,
        Boolean
    };
    class Primitive : Type {
        PrimitiveType type;
    public:
        Primitive(PrimitiveType);
        void debug_print() const override;
    };
}
```
```cpp:syntax.cpp
namespace syntax {
    Primitive::Primitive(PrimitiveType type): type(type) {}
}
```
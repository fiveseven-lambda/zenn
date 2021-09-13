---
title: "軽実装！左結合中置演算子の優先順位付パース"
emoji: "➗"
type: "tech"
topics: ["構文解析", "cpp"]
published: true
---

中置演算子の構文解析が面倒みたいに言っている人を見かけたので，そんなことないよ〜という記事です．使う言語は C++17 です．
# パーサの仕様
字句解析やカッコの解析等の要素を全て省いて，左結合の中置演算子の構文解析にのみ着目するために，今回作るパーサは非常に単純化された以下のような式を読みます．

- 変数は英大文字 `A`-`Z` あるいは英小文字 `a`-`z` の 1 文字のみ．
- 2 項演算子は `*` `/` `+` `-` `<` `>` `&` `|` の 8 種類．
  - 優先順位は高い方から `*` `/` → `+` `-` → `<` `>` → `&` → `|` の順．
  - 全て左結合．
- 余計な空白を含まない．

BNF で書くと次のようになります．
```
<Id> ::= [A-Za-z]
<Term> ::= <Id> | <Term> '*' <Id> | <Term> '/' <Id>
<Side> ::= <Term> | <Side> '+' <Term> | <Side> '-' <Term>
<Ineq> ::= <Side> | <Ineq> '<' <Side> | <Ineq> '>' <Side>
<Conj> ::= <Ineq> | <Conj> '&' <Ineq>
<Disj> ::= <Conj> | <Disj> '|' <Conj>
```
# 構文木
パーサは文字列を構文木に変換します．今回作る構文木は，全ての頂点が「子をもたない」「ちょうど 2 つの子をもつ」のどちらかであるような木構造であり，以下のように基底クラス `Expr` と 2 つの派生クラス `Id`，`Binary` として定義できます．
```cpp
#include <memory>
#include <utility>

// 基底クラス
class Expr {
public:
	virtual ~Expr() = default;
};

// 1 文字の変数／子をもたない頂点
class Id : public Expr {
	char name; // 変数名
public:
	Id(char name): name(name) {}
	~Id() = default;
};

// 2 項演算子
enum class Op {
	Mul,     // '*'
	Div,     // '/'
	Add,     // '+'
	Sub,     // '-'
	Less,    // '<'
	Greater, // '>'
	And,     // '&'
	Or,      // '|'
};

// 2 項演算／子を 2 つもつ頂点
class Binary : public Expr {
	std::unique_ptr<Expr> left, right;
	Op op;
public:
	Binary(std::unique_ptr<Expr> left, std::unique_ptr<Expr> right, Op op):
		left(std::move(left)),
		right(std::move(right)),
		op(op) {}
	~Binary() = default;
};
```
# 確認用の出力
ちゃんとパースされたか確認するときのために出力用のメンバ関数 `print(int indent)` も用意しておきます．引数 `indent` は見やすくするためのインデントです．
```cpp
class Expr {
	/* 略 */
public:
	virtual void print(int = 0) = 0; // これを追加
};

class Id : public Expr {
	/* 略 */
public:
	void print(int) override; // これを追加
};

class Binary : public Expr {
	/* 略 */
public:
	void print(int) override; // これを追加
};

constexpr char TAB[] = "    ";

void Id::print(int indent) {
	for(int i = 0; i < indent; ++i) std::cout << TAB;
	std::cout << name << std::endl;
}

void Binary::print(int indent) {
	for(int i = 0; i < indent; ++i) std::cout << TAB;
	std::string name;
	switch(op){
		case Op::Mul: name = "mul"; break;
		case Op::Div: name = "div"; break;
		case Op::Add: name = "add"; break;
		case Op::Sub: name = "sub"; break;
		case Op::Less: name = "less"; break;
		case Op::Greater: name = "greater"; break;
		case Op::And: name = "and"; break;
		case Op::Or: name = "or";
	}
	std::cout << name << std::endl;
	left->print(indent + 1);
	right->print(indent + 1);
}
```
これで，たとえば `a+b-c*d` のパースが正しくできていれば
```
sub
    add
        a
        b
    mul
        c
        d
```
と出力されるようになります．
# `*` `/` のみの場合
まず，含まれる 2 項演算子が `*` と `/` の 2 種類のみの場合について考えます．BNF で言うと `<Term>` のみをパースすることになります．`*` と `/` の優先順位は同じなので，たとえば `a*b/c*d` であれば
```
mul
    div
        mul
            a
            b
        c
    d
```
となります．これを `parse_term` 関数として実装します．簡単のため，文字列を引数として受け取るような形ではなく，関数内で直接 `std::cin` から 1 文字ずつ読む形にします．
```cpp
std::unique_ptr<Expr> parse_term();
```

優先順位を気にしなくてよいため，まず返り値をもつ `ret` という変数を用意しておいて，次に式を前から順に読みながら `ret` を順次書き換えてゆくだけでできます．
```cpp
std::unique_ptr<Expr> parse_term() {
	// 式の先頭は単一の変数
	std::unique_ptr<Expr> ret = std::make_unique<Id>(std::cin.get());
	for(;;){
		Op op;
		switch(std::cin.peek()){
			case '*': op = Op::Mul; break;
			case '/': op = Op::Div; break;
			default: return ret;
		}
		std::cin.get(); // '*' か '/' を読む
		ret = std::make_unique<Binary>(
			// 今までの ret は左の被演算子になる
			std::move(ret),
			// 右の被演算子は常に単一の変数
			std::make_unique<Id>(std::cin.get()),
			op
		);
	}
}
```
# 優先順位のある場合
次に，8 種類すべての 2 項演算子を入れてパースします．すると今度は，`<Side>` のパースの中で `<Term>` のパースを使い，`<Ineq>` のパースの中で `<Side>` のパースを使い，……というように，`<Disj>`→`<Conj>`→`<Ineq>`→`<Side>`→`<Term>` の順でパース関数が別のパース関数を呼び出すことになります．この 5 つを全て別の関数として書くと大変なので，1 つの再帰関数として実装します．

といっても基本的には変わらなくて，
- 5 段階のうち何段階目のパースをしているか表す引数 `precedence` がくわわる
- `switch` 文の中身が「今の段階で解析の対象となる 2 項演算子」の判別に変わる．
- `parse_term` 関数において `std::make_unique<Id>(std::cin.get())` だった部分が再帰呼出しになる．

結局全体では次のようになります．
```cpp
#include <optional>

std::unique_ptr<Expr> parse(int precedence = 0) {
	if(precedence == 5){
		// 再帰のベースケース：単一の変数
		return std::make_unique<Id>(std::cin.get());
	}
	std::unique_ptr<Expr> ret = parse(precedence + 1); // 再帰呼出し
	for(;;){
		std::optional<Op> op;
		switch(std::cin.peek()){
			case '*':
				if(precedence == 4) op = Op::Mul;
				break;
			case '/':
				if(precedence == 4) op = Op::Div;
				break;
			case '+':
				if(precedence == 3) op = Op::Add;
				break;
			case '-':
				if(precedence == 3) op = Op::Sub;
				break;
			case '<':
				if(precedence == 2) op = Op::Less;
				break;
			case '>':
				if(precedence == 2) op = Op::Greater;
				break;
			case '&':
				if(precedence == 1) op = Op::And;
				break;
			case '|':
				if(precedence == 0) op = Op::Or;
				break;
		}
		if(!op){
			// 今の段階で解析の対象となる演算子ではない
			return ret;
		}
		std::cin.get();
		ret = std::make_unique<Binary>(
			std::move(ret),
			parse(precedence + 1), // 再帰呼出し
			op.value()
		);
	}
}
```
# コード全体
今回のパーサの全体像です．
```cpp
#include <iostream>
#include <memory>
#include <optional>

class Expr {
public:
	virtual ~Expr() = default;
	virtual void print(int = 0) = 0;
};

class Id : public Expr {
	char name;
public:
	Id(char name): name(name) {}
	~Id() = default;
	void print(int) override;
};

enum class Op {
	Mul, Div,
	Add, Sub,
	Less, Greater,
	And,
	Or,
};

class Binary : public Expr {
	std::unique_ptr<Expr> left, right;
	Op op;
public:
	Binary(std::unique_ptr<Expr> left, std::unique_ptr<Expr> right, Op op):
		left(std::move(left)),
		right(std::move(right)),
		op(op) {}
	~Binary() = default;
	void print(int) override;
};

std::unique_ptr<Expr> parse(int precedence = 0) {
	if(precedence == 5) return std::make_unique<Id>(std::cin.get());
	std::unique_ptr<Expr> ret = parse(precedence + 1);
	for(;;){
		std::optional<Op> op;
		switch(std::cin.peek()){
			case '*':
				if(precedence == 4) op = Op::Mul;
				break;
			case '/':
				if(precedence == 4) op = Op::Div;
				break;
			case '+':
				if(precedence == 3) op = Op::Add;
				break;
			case '-':
				if(precedence == 3) op = Op::Sub;
				break;
			case '<':
				if(precedence == 2) op = Op::Less;
				break;
			case '>':
				if(precedence == 2) op = Op::Greater;
				break;
			case '&':
				if(precedence == 1) op = Op::And;
				break;
			case '|':
				if(precedence == 0) op = Op::Or;
				break;
		}
		if(!op) return ret;
		std::cin.get();
		ret = std::make_unique<Binary>(
			std::move(ret),
			parse(precedence + 1),
			op.value()
		);
	}
}

int main(){
	parse()->print();
}

constexpr char TAB[] = "    ";

void Id::print(int indent) {
	for(int i = 0; i < indent; ++i) std::cout << TAB;
	std::cout << name << std::endl;
}

void Binary::print(int indent) {
	for(int i = 0; i < indent; ++i) std::cout << TAB;
	std::string name;
	switch(op){
		case Op::Mul: name = "mul"; break;
		case Op::Div: name = "div"; break;
		case Op::Add: name = "add"; break;
		case Op::Sub: name = "sub"; break;
		case Op::Less: name = "less"; break;
		case Op::Greater: name = "greater"; break;
		case Op::And: name = "and"; break;
		case Op::Or: name = "or";
	}
	std::cout << name << std::endl;
	left->print(indent + 1);
	right->print(indent + 1);
}
```
ね，決して実装は重くないでしょう？
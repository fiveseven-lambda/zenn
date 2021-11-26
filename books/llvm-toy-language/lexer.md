---
title: "字句解析"
---

いよいよ字句解析を始めよう．

対話環境を実現したいので，入出力と字句解析は完全に切り離せない．これがちょっと面倒を生む．

# Inner（ヘルパ）
字句解析は，`Lexer` 構造体が担う．
```cpp:lexer.hpp
namespace lexer {
    class Lexer;
}
```
……わけなのだが，`Lexer` は入出力とか peek だけを扱い，実際に文字列からトークンへの分解を行う部分は `Inner` 構造体に任せる．

```cpp:lexer.hpp
#include <cstddef>
#include <string>
#include <queue>

#include "token.hpp"

namespace lexer {
    class Inner {
    public:
        void run(
            std::size_t,
            const std::string &,
            std::queue<token::TokenWithPos> &
        );
    };
}
```
```cpp:lexer.cpp
#include "lexer.hpp"

namespace lexer {
    void Inner::run(
        std::size_t line_num,
        const std::string &str,
        std::queue<token::TokenWithPos> &queue
    ){
        /* queue にトークンを追加 */
    }
}
```

`Inner::run()` は，引数として
- `line_num`（`std::size_t`）：今見ている行番号
- `str`（`const std::string &`）：今見ている行
- `queue`（`std::queue<token::TokenWithPos> &`）：分解したトークンの保存先

を受け取る．`Lexer` が入力を 1 行読み，`Inner` に「この queue に追加しておいてね〜」というと，`Inner` はその行をトークンに分解して queue に格納するわけだ．

`Inner::run()` の具体的な実装は後回しにして，その前に `Lexer` の方を完成させる．
# Lexer
`Lexer` の仕事は，入力を読んで `Inner` に分解させ，トークンを返すイテレータとなることだ．
```cpp:lexer.hpp
#include <iostream>
#include <fstream>
#include <vector>

namespace lexer {
    class Lexer {
        std::istream &source;
        bool prompt;
        Inner inner;
        std::vector<std::string> log;
        std::queue<token::TokenWithPos> tokens;
    public:
        Lexer();
        Lexer(std::ifstream &);
        const std::vector<std::string> &get_log() const;
        token::TokenWithPos next(), &peek();
    };
}
```
対話環境では `> ` みたいなプロンプトが出るとうれしい．それを出すかどうかはメンバ変数 `prompt` で切り替える．

コンストラクタは，対話環境用とファイルからの入力用の 2 つを用意する．

```cpp:lexer.cpp
namespace lexer {
    Lexer::Lexer():
        source(std::cin),
        prompt(true) {}
    Lexer::Lexer(std::ifstream &source):
        source(source),
        prompt(false) {}
}
```

受け取った入力は `log` に保存しておいて，`get_log()` で手に入るようにする．

```cpp:lexer.cpp
namespace lexer {
    const std::vector<std::string> &Lexer::get_log() const {
        return log;
    }
}
```
`next()`，`peek()` は `Lexer` がイテレータとして働くための関数．EOF に達したら，返り値の `.second` が `nullptr` になる．
```cpp:lexer.cpp
namespace lexer {
    token::TokenWithPos &Lexer::peek(){
        while(tokens.empty()){
            if(source){
                // 次に読むのは何行目か (0-indexed)
                auto line_num = log.size();
                // 空の行を追加
                log.emplace_back();
                // 対話環境ならプロンプトを出す
                if(prompt) std::cout << "> ";
                // 1 行読む
                std::getline(source, log.back());
                // トークンに分解
                inner.run(line_num, log.back(), tokens);
            }else{
                // EOF に達したので nullptr を追加
                tokens.emplace();
            }
        }
        return tokens.front();
    }
    token::TokenWithPos Lexer::next(){
        auto ret = std::move(peek());
        tokens.pop();
        return ret;
    }
}
```
# `Inner::run()`
さて，`Inner::run()` の中身を埋めていく．
```cpp:lexer.cpp
void Inner::run(
    std::size_t line_num,
    const std::string &str,
    std::queue<token::TokenWithPos> &queue
){
    /* 中身 */
}
```
まずは，文字列を走査するためのカーソル `cursor` を作る．「何バイト目か」という数字が `pos::Range` を作るときに必要なので，`cursor` の型は `std::size_t` にしておいて，各文字を見るには `str[cursor]` と書こう．これをイテレータにしておいて，`std::distance` を使ってもいいと思う．
```cpp:lexer.cpp
    std::size_t cursor = 0;
```

次に `while` 文を回す．1 回のループでは，1 個のトークンを読み，読んだ文字数だけ `cursor` を進める．

ループ内では，まず空白を読み飛ばす．この間に行末に達したら，その行にトークンはもう残っていないので関数を抜ける．
```cpp:lexer.cpp
    while(true){
        // この中でトークンを 1 個読む

        // 空白を読み飛ばす
        while(true){
            if(cursor == str.size()) return;
            if(!std::isspace(str[cursor])) break;
            ++cursor;
        }
        std::size_t start = cursor;
        /* str[start] はトークンの先頭の文字 */

    }
```
あとは，1 文字目が数字なら整数リテラル，アルファベットなら識別子……という具合にトークンを読んで，queue に格納する．

ここで，こんなのを用意しておくと都合がいい．
```cpp:lexer.cpp
    auto advance_if = [&](char c){
        bool ret = str[cursor] == c;
        if(ret) ++cursor;
        return ret;
    };
```

使ってる様子を見たら便利なのが分かると思う．
```cpp:lexer.cpp
        std::unique_ptr<token::Token> token;
        if(std::isdigit(str[start])){
            // 整数リテラル
            while(std::isdigit(str[cursor])) ++cursor;
            token = std::make_unique<token::Integer>(str.substr(start, cursor - start));
        }else if(std::isalpha(str[start]) || str[start] == '_'){
            // 識別子
            while(std::isalnum(str[cursor]) || str[cursor] == '_') ++cursor;
            token = std::make_unique<token::Identifier>(str.substr(start, cursor - start));
        }else if(advance_if('+')){
            if(advance_if('=')) token = std::make_unique<token::PlusEqual>();
            else token = std::make_unique<token::Plus>();
        }else if(advance_if('<')){
            if(advance_if('<'))
                if(advance_if('=')) token = std::make_unique<token::DoubleLessEqual>();
                else token = std::make_unique<token::DoubleLess>();
            else if(advance_if('=')) token = std::make_unique<token::LessEqual>();
            else token = std::make_unique<token::Less>();
        }else if /* 中略．他の記号についても同様 */
        }else{
            // 知らない文字が現れたらエラー
            throw error::make<error::UnexpectedCharacter>(pos::Pos(line_num, cursor));
        }
        // pos::Range と一緒に queue に追加
        queue.emplace(pos::Range(line_num, start, cursor), std::move(token));
```

最後の `error::UnexpectedCharacter` は次のように定義しておく．
```cpp:error.hpp
#include "pos.hpp"

namespace error {
    class UnexpectedCharacter : public Error {
        pos::Pos pos;
    public:
        UnexpectedCharacter(pos::Pos &&);
        void eprint(const std::vector<std::string> &) const override;
    };
}
```
```cpp:error.cpp
namespace error {
    UnexpectedCharacter::UnexpectedCharacter(pos::Pos &&pos):
        pos(std::move(pos)) {}
    void UnexpectedCharacter::eprint(const std::vector<std::string> &log) const {
        std::cerr << "unexpected character at " << pos << std::endl;
        pos.eprint(log);
    }
}
```

これで字句解析ができた．

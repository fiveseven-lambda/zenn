---
title: "言語処理系のツボ：制御フロー (WIP)"
emoji: "🐈"
type: "tech"
topics: ["rust"]
published: true
---
:::message
未完成記事です　残しておかないと存在を忘れてしまう……
:::

『コンパイラ 原理・技法・ツール（第 2 版）』の、制御フローに関する部分の基本をおさえた以下の短いコードについて、説明する記事を書きます（予定）
```rust
fn main() {
    let input = "
      a b c
      if x
        d e
      else
        f g
      end
      h i j
      if y
        k l
        if z
          m
        else
          n o
        end
        p q
      end
      r s t
      while x
        u v
      end
      w
    end";
    let mut words = input.split_whitespace();
    let mut stmts = Vec::new();
    parse(&mut words, &mut stmts, None);

    for stmt in &stmts {
        stmt.translate();
    }
    println!("End.");
}

enum Stmt {
    Expr(String),
    If(String, Vec<Stmt>, Vec<Stmt>),
    While(String, Vec<Stmt>),
    Break,
    Continue,
}

fn parse<'a>(
    words: &mut impl Iterator<Item = &'a str>,
    stmts: &mut Vec<Stmt>,
    stmts_else: Option<&mut Vec<Stmt>>,
) {
    loop {
        match words.next().unwrap() {
            "if" => stmts.push({
                let cond = words.next().unwrap();
                let mut stmts_then = Vec::new();
                let mut stmts_else = Vec::new();
                parse(words, &mut stmts_then, Some(&mut stmts_else));
                Stmt::If(String::from(cond), stmts_then, stmts_else)
            }),
            "else" => return parse(words, stmts_else.unwrap(), None),
            "while" => stmts.push({
                let cond = words.next().unwrap();
                let mut stmts = Vec::new();
                parse(words, &mut stmts, None);
                Stmt::While(String::from(cond), stmts)
            }),
            "break" => stmts.push(Stmt::Break),
            "continue" => stmts.push(Stmt::Continue),
            "end" => return,
            expr => stmts.push(Stmt::Expr(String::from(expr))),
        }
    }
}

impl Stmt {
    fn translate(&self) {
        match self {
            Stmt::Expr(expr) => print!("{expr}, "),
            Stmt::If(cond, stmts_then, stmts_else) => {
                println!("Br {cond}.");
                for stmt in stmts_then {
                    stmt.translate();
                }
                println!("Jmp.");
                for stmt in stmts_else {
                    stmt.translate();
                }
                println!("Jmp.");
            }
            Stmt::While(cond, stmts) => {
                println!("Jmp.");
                println!("Br {cond}.");
                for stmt in stmts {
                    stmt.translate();
                }
                println!("Jmp.");
            }
            Stmt::Break | Stmt::Continue => println!("Jmp."),
        }
    }
}
```

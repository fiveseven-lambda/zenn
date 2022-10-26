---
title: "GitHub ActionsでLaTeX自動コンパイル"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github", "latex"]
published: false
---

# 目標

GitHub に LaTeX ソースファイルを push すると自動で PDF へとコンパイルされる環境を作ります．

# 準備
Git リポジトリを用意します．編集作業を行うブランチ名は，main とします．
```sh
$ git init -b main
$ git remote add origin (URL)
```

# main ブランチの編集
とりあえず今回は以下の `sample.tex` を作っておきます．
```tex:sample.tex
\documentclass{article}
\begin{document}
Hello, world!
\end{document}
```
`$ latexmk -pdf` でコンパイルできることを確認します．

LaTeX はコンパイル毎にたくさんのファイルを生成しますが，多分全部ソースファイルの名前にならって `sample.*` になると思います．よって `sample.tex` だけを管理するために以下のようにします．
```:.gitignore
sample.*
!sample.tex
```

ここまでの変更を GitHub 上に push します．
```sh
$ git add .
$ git commit -m "Initial commit"
$ git push -u origin main
```

# pdf ブランチの作成
出来上がった PDF ファイルの push 先となる pdf ブランチを作ります．
```sh
$ git checkout -b pdf
```

こちらは `sample.*` を ignore したくないので，`.gitignore` を書き換えます．ただし，LaTeX が自動的に生成する `sample.aux` は，前のものが残っているとコンパイルが失敗することがあるので，これだけ ignore します．
```:.gitignore
sample.aux
```
この状態で GitHub 上に push します．
```sh
$ git add .gitignore
$ git commit -m "Created branch pdf"
$ git push origin pdf
```

# GitHub Actions の利用
main ブランチに戻り，`.github/workflows`下の yml ファイルにワークフローを記述します．
```sh
$ git checkout main
$ mkdir -p .github/workflows
```
```yml:.github/workflows/latex.yml
name: LaTeX compilation
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps: [ ここに記述 ]
```

Action 内で Git リポジトリの中身に対する操作を行うときは，actions/checkout を利用します．
```yml
      - name: Set up Git repository
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
```
`fetch-depth: 0` は，main ブランチだけでなく pdf ブランチも fetch するよう指定しています．

次に，Git 環境を整え，main ブランチを pdf ブランチに merge します．
```yml
      - name: Merge main branch
        run: |
          git config user.name fiveseven-lambda
          git config user.email fiveseven.lambda@gmail.com
          git checkout pdf
          git merge main
```

そして TeX ファイルをコンパイルします．[GitHub Marketplace](https://github.com/marketplace)で検索すると，xu-cheng/latex-action というものが出てくるので，これを使います．
```yml
      - name: Compile LaTeX document
        uses: xu-cheng/latex-action@v2
        with:
          root_file:
            sample.tex
```
デフォルトで，`latexmk` の `-pdf` オプションで PDF へコンパイルされます．

最後に，できあがった PDF ファイルを pdf ブランチに push します．
```yml
      - name: Push PDF file
        run: |
          git add .
          git commit -m "LaTeX compilation"
          git push origin pdf
```

全体では以下のようになります．
```yml:.github/workflows/latex.yml
name: LaTeX compilation
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Git repository
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Merge main branch
        run: |
          git config user.name fiveseven-lambda
          git config user.email fiveseven.lambda@gmail.com
          git checkout pdf
          git merge main
      - name: Compile LaTeX document
        uses: xu-cheng/latex-action@v2
        with:
          root_file:
            sample.tex
      - name: Push PDF file
        run: |
          git add .
          git commit -m "LaTeX compilation"
          git push origin pdf
```

これを push すれば終わりです．
```sh
$ git add .
$ git commit -m "Added workflow for LaTeX compilation"
$ git push
```

これ以降は，`sample.tex` を編集して main に commit，push するだけで OK です！
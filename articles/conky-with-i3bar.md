---
title: "i3status の代替としての conky"
emoji: "📝"
type: "tech"
topics: ["i3wm"]
published: true
---

i3 window manager のデフォルトの設定では，画面下に表示されるバー (i3bar) の内容は i3status によって生成されています．ためしにコンソールで `$ i3status` を実行すると，バーと同じ内容が出てくるでしょう．

i3status の生成する文字列は `~/.config/i3status/config` で設定できますが，i3status 自体を別のもので置き換えてさらに自由度の高い設定をすることも可能です．ここで代替として使えるのが conky です．

conky はもともと GUI 用のシステム監視ソフトで，CPU や RAM の使用状況などをウィンドウに表示してくれるのですが，GUI ではなく代わりに stdout に出力させるようにも設定できます．これを i3bar に渡してやれば，conky が i3status の代替となるのです．

[英語の解説ページ](https://i3wm.org/docs/user-contributed/conky-i3bar.html)．

# conky のインストール
[conky](https://www.archlinux.jp/packages/extra/x86_64/conky/) パッケージがあります．
```
$ pacman -S conky
```
設定ファイルは Lua の文法で書き，`~/.config/conky/conky.conf` に置きます．`$ conky -C` で雛形が生成できます．
```
$ mkdir -p ~/.config/conky
$ conky -C > ~/.config/conky/conky.conf
```
`conky.conf` を開いてみると，`conky.config =` の後にたくさん設定項目があります．`out_to_X` と`own_window` を `false` にして，`out_to_console` を `true` にすれば，`conky` を実行してもウィンドウが表示されず，代わりにシステム情報が stdout へ出力されるようになります．

`conky.text =` の後の 2 重四角括弧 `[[` 〜 `]]` の中で出力の内容を設定します（Lua の改行可能な文字列リテラルらしい）．

# conky - i3bar 間の橋渡し
`i3bar` は，バーに表示する内容を以下のような json で受け取ります．

- 全体は 1 つの配列 A．
- A の各要素 A[t] は，時刻 t におけるバー全体の内容．
- A[t] も配列．
- A[t] の各要素 A[t][i] は，バーを構成するブロック（CPU 使用率のブロックとか，RAM 使用率のブロックとか）．
- A[t][i] はオブジェクト．
- A[t][i]['full_text'] でテキスト，A[t][i]['color'] で文字色を指定する．

`conky` が A の各要素 A[t] を一定の時間間隔（たとえば 1 秒おき）で `i3bar` に入力すると，そのたびにバーの表示内容が更新されます．

`conky` は毎回同じ内容を出力します．一方 json の配列は `[` で始まってその後カンマ区切りで要素が続くので，毎回同じ文字列だと最初の `[` だけ出力できません．そこで，軽くスクリプトをかませる必要があります．

以下のスクリプトをどこかに置いておき，`conky` を直接呼び出す代わりにこのスクリプトを呼び出すことにします．
```bash
#!/bin/bash
echo '{"version":1}'
echo '['
exec conky
```
`{"version":1}` もなんかヘッダとして要るっぽいです．

そうしたら，`~/.config/i3/config` も編集します．
```
bar {
    status_command (スクリプトの場所)
}
```
# 設定
さて，あとは `~/.config/conky/conky.conf` 内の `conky.text =` 以降を編集するだけです！

執筆途中　まってね

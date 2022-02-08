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
`i3bar` は，バーに表示する内容を以下のような JSON で受け取ります．

- 全体は 1 つの配列 A．
- A の各要素 A[t] は，時刻 t におけるバー全体の内容．
- A[t] も配列．
- A[t] の各要素 A[t][i] は，バーを構成するブロック（CPU 使用率のブロックとか，RAM 使用率のブロックとか）．
- A[t][i] はオブジェクト．
- A[t][i]['full_text'] でテキスト，A[t][i]['color'] で文字色を指定する．

`conky` が A の各要素 A[t] を一定の時間間隔（たとえば 1 秒おき）で `i3bar` に入力すると，そのたびにバーの表示内容が更新されます．

`conky` は毎回同じ内容を出力します．一方 JSON の配列は `[` で始まってその後カンマ区切りで要素が続くので，毎回同じ文字列だと最初の `[` だけ出力できません．そこで，軽くスクリプトをかませる必要があります．

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

conky(1) を見ながらがいいです．あと，こういうとき JSON が trailing comma ダメなの腹たつ．

手始めに，RAM 使用率，CPU 使用率，時刻だけを表示する次のような例を見てみます．
```lua
conky.text = [[
    [
        {"full_text": "RAM $mem / $memmax ($memperc%)"},
        {"full_text": "CPU $cpu%"},
        {"full_text": "$time"}
    ],
]]
```
まず，`[[`〜`]]` で囲まれた部分が Lua の文字列です（このとき `"` は関係ありません）．

よって，`conky.text` には以下のような文字列が代入されることになります．
```
    [
        {"full_text": "RAM $mem / $memmax ($memperc%)"},
        {"full_text": "CPU $cpu%"},
        {"full_text": "$time"}
    ],
```
次に，conky がこの文字列中の `$` で始まる部分を値で置き換えて出力します（ここでも `"` は関係ありません）．`$mem` はメモリ使用量，`$cpu` は CPU 使用率，`$time` は現在時刻といった具合です．これにより，conky の出力はたとえば
```json
    [
        {"full_text": "RAM 4.00 GiB / 16.0 GiB (25%)"},
        {"full_text": "CPU 10%"},
        {"full_text": "2022-02-08 20:00:00"}
    ],
```
のようになります．

i3bar はこれを JSON として読み取って
```
RAM 4.00 GiB / 16.0 GiB (25%)|CPU 10%|2022-02-08 20:00:00
```
のように表示します．

`${cpu cpu1}` のように `{`〜`}` でくくると引数を与えられます．この例だと `$cpu` に `cpu1` という引数を渡していて，これは 1 つめのコアの使用率を表します（1 から数える．`cpu0` は総使用率）．

Lua，conky，JSON という 3 段階を経ているのでややこしいですね．

# 文字色を変える
i3bar に渡す JSON では，
```json
{"full_text": "CPU 10%", "color": "#ff0000"}
```
のように `color` プロパティを追加することで文字色が指定できます．

`$if_match` を使うと，CPU 使用率が 90% 未満なら白，90% 以上なら赤というように，条件分岐を挟むことができます．
```lua
conky.text = [[ [
    {
        "full_text": "CPU $cpu%",
        "color": "${if_match $cpu < 90}\#ffffff${else}\#ff0000${endif}"
    }
], ]]
```
これにより，CPU 使用率が 10% のときと 95% のときの conky の出力はそれぞれ
```json
[
    {
        "full_text": "CPU 10%",
        "color": "#ffffff"
    }
],
```
```json
[
    {
        "full_text": "CPU 95%",
        "color": "#ff0000"
    }
],
```
となるわけです．

# Lua の関数を呼び出す
文字色を 2 つの間で切り替えるだけでなく，CPU 使用率が増えるにつれて段々と赤に近づくようにするには，どうすれば良いでしょうか．こんなときは `$lua` を使って Lua の関数を呼び出します．

まず適当な `.lua` ファイルを用意し，呼び出したい関数を書きます．このとき関数名は `conky_` で始まっている必要があります．そして `conky.config = {`〜`}` の中に `lua_load = '(ファイル名).lua'` を追加します．

この状態で `conky.text` の中に `${lua (関数名)}`（ただし関数名の先頭の `conky_` は取り除く）という文字列を含めておくと，conky は関数を呼び出し，その部分を返り値で置き換えます．

また，`${lua (関数名) (引数)}` と書くと引数も渡せます．ただしこのとき，`${lua fnc $cpu}` と書いても，`conky_fnc('10')` のように置換後の `$cpu` が渡されることはなく，呼び出しは `conky_fnc('$cpu')` となります．関数内で実際の CPU 使用率を得るには，`conky_parse` 関数を使って `conky_parse('$cpu')` と書きます．

さて，今回は文字色を変えるために以下のような関数を書きました．
```lua
function conky_color(arg)
    tmp = math.floor(conky_parse(arg) * 255 / 100)
    return string.format("#%02x%02x%02x", tmp, 255 - tmp, 255 - tmp)
end
```
これにより，`conky.text` 中にたとえば `${lua color $cpu}` と書くと，置換時に `conky_color('$cpu')` という関数呼び出しが起こり，その中で `conky_parse('$cpu')` が CPU 使用率を返すので，その値に応じて文字色を表す `#XXXXXX` 形式の文字列が返されます．よって，`conky.conf` 内には
```lua
conky.text = [[ [
    {
        "full_text": "RAM $mem / $memmax ($memperc%)",
        "color": "${lua color $memperc}"
    },
    {
        "full_text": "CPU $cpu%",
        "color": "${lua color $cpu}"
    },
    {
        "full_text": "$time"
    }
], ]]
```
のように書いておけば良いわけです．

# Lua の機能を使う
たとえば CPU が 8 コア 16 スレッドあって，それぞれの使用率を `${cpu cpu1}` から `${cpu cpu16}` まで並べて全部表示したいとします．こんなときは，Lua の機能が役立ちます．
```lua
cpu = {}
for i = 1, 16 do
    cpu[i] =
        '{"full_text": "${cpu cpu' .. i .. '}",' ..
        '"color": "${lua color ${cpu cpu' .. i .. '}}"}'
end
conky.text = '[' .. table.concat(cpu, ',') .. ']'
```
もはや何がなんだかって感じですね．

`'${cpu cpu' .. i .. '}'` は，`'${cpu cpu'` と `i` と `'}'` を文字列として連結したものなので，たとえば `i` の値が 5 なら `${cpu cpu5}` となります．

`table.concat(cpu, ',')` は，`cpu[1]` から `cpu[16]` までを `,` 区切りで連結します．

結果的に `conky.text` には cpu1 の表示から cpu16 の表示までが JSON で突っ込まれることになります．

最後 `conky.text` に代入している部分では，前後に他のブロックも追加できます．

これでだいぶ自由度の高い設定ができるようになるんじゃないかと思います．
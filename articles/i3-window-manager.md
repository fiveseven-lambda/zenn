---
title: "i3wm，config しないなんて勿体ない！"
emoji: "🪟"
type: "tech"
topics: ["i3wm"]
published: true
---

# みなさん i3 window manager 使ってますか？
使ってますよね，**使い心地いい**ですもんね．

[User's Guide](https://i3wm.org/docs/userguide.html) で説明されている**色々な機能**を使って config を書くと**本当に快適に**……

え？ **config はデフォルトのまま**？？

**勿体ない！！！** i3 は**好きに config を書き換えて自分の手に一番馴染むものを追求する**のが良いんですよ！！！（主張）

# さぁ， i3 の真の魅力を引き出しましょう．
要約：この記事は [i3 User's Guide](https://i3wm.org/docs/userguide.html) の内容のうち，キーボードショートカット等に関する部分を日本語で説明したものです．

i3 を設定するには， `/etc/i3/config` に用意されているデフォルト設定ファイルを `~/.i3/config` あるいは `~/.config/i3/config` にコピーし，編集します．

i3 の設定ファイルの書き方はバージョンごとに違い，今の最新は v4 です．コメント（ `#` で始まる行はコメントです）を用いて
```
# i3 config file (v4)
```
と書き， v4 であることを明記してください（でないと，古いバージョンで書かれているとみなされる可能性があります）．

設定ファイル内では変数を使うことができます． `$` で始まるのが変数です．
```
set $hoge fuga
```
と書いておくと，ファイル内の `$hoge` が `fuga` で置き換わります．

# キーバインド（ショートカットキーの設定）
キーバインドは **`bindsym`** あるいは **`bindcode`** を使って設定します．前者はキーを keysym，後者は keycode で指定します．なので設定したいキーの keysym あるいは keycode を事前に知る必要があります．

`xorg-docs` パッケージに入っている `X(7)` の `KEYBOARDS` セクションによれば， keycode は物理的な各キーの番号で，一方 keysym はそのキーが表す文字です． `$ xmodmap -pke` で keycode と keysym の対応一覧を見ることができます．たとえばエンターキーの keysym は `Return` です．

普通ショートカットキーは修飾キー（Ctrl や Alt）と一緒に使いますが，これはたとえば前に `Mod1+` を付けて `Mod1+Return` とすることで Alt+エンターになります．前に付けられる修飾キーは `Control` `Shift` `Mod1` `Mod2` `Mod3` `Mod4` `Mod5` です（`Mod1` が Alt， `Mod4` が Super）．

```
bindsym Mod1+Return exec i3-sensible-terminal
```
これは Alt+エンターでターミナルが起動するように設定しています．たとえばエンターの keycode が 36 なら
```
bindcode Mod1+36 exec i3-sensible-terminal
```
としても同じです．

keysym / keycode の前に `--release` を付けると，キーを押したタイミングではなく，離したタイミングでコマンドが実行されます．
```
bindsym --release Mod1+Return exec i3-sensible-terminal
```

# マウスバインド
マウスバインドは，ウィンドウのどこかをクリックしたときに実行するショートカットです．キーバインドと同じ **`bindsym`** を使います．

自分の環境では，左クリックが `button1`，中央（ホイール）クリックが `button2`，右クリックが `button3`，ホイールのスクロールが `button4` / `button5` なんですがこれって共通なのかな？一応 keycode / keysym と一緒に `xev` （キーボード／ポインタイベントの情報をリアルタイムで逐一出力してくれるツール）で調べられます．

デフォルトでは，タイトルバーがクリックされたときに反応します．
```
bindsym button3 floating toggle
```
こうすると，タイトルバーで右クリックしたときに，ウィンドウがタイル状態と自由に動かせるフロート状態の間で切り替わります．マウスバインドと同様に修飾キーを付けることもできます．

```
bindsym Mod1+button3 floating toggle
```

`--release` を付けると離したときに反応します．
```
bindsym --release button2 kill
```
タイトルバーでホイールボタンを離したときに，画面が閉じます．

タイトルバーだけでなくウィンドウ全体が反応するようにしたければ， `--whole-window` を指定します．
```
bindsym --whole-window Mod1+button2 kill
```
ウィンドウ内のどこかで `Mod1+button2` を押すことで画面が閉じます．ただしこのときウィンドウのボーダー（境界線）だけは含まれないので，それも含めるには `--border` を追加します．また，逆にタイトルバーを除外するには `--exclude-titlebar` を指定します．

# モード
**`mode`** を使うとモードを定義することができます．モードとは次のようなものです．

- キーバインド／マウスバインドを用いてモード間を移動することができる．
- 各キーバインド／マウスバインドは，特定のモードでのみ有効になる

`/etc/i3/config` を見ると， resize というモードが定義されています．

```-:/etc/i3/config
mode "resize" {
        (略)
        bindsym Left        resize shrink width 10 px or 10 ppt
        bindsym Down        resize grow height 10 px or 10 ppt
        bindsym Up          resize shrink height 10 px or 10 ppt
        bindsym Right       resize grow width 10 px or 10 ppt
        (略)
        bindsym Return mode "default"
        bindsym Escape mode "default"
        bindsym Mod1+r mode "default"
}

bindsym Mod1+r mode "resize"
```

`bindsym Mod1+r mode "resize"` により， `Mod1+r` を押して resize モードに入ることができます． resize モードでは `mode "resize"` のブロック内に書かれているキーバインド／マウスバインドのみが有効になり，上下左右キーを用いてウィンドウの大きさを変えることができます．`bindsym Return mode "default"` により， `Return` で resize モードを抜けることができます． default モードは初めから用意されているモードで，モードを指定せずに書いたキーバインド／マウスバインドは全て default モードに属しています．

あるモードに入ると，そのモード内で定義されたキーバインド／マウスバインド以外全部無効になってしまうため注意が必要です．特に， default モードに戻る手段を書き忘れると大変なことになります． `Return` でも `Escape` でも `Mod1+r` （モードに入るときのキーバインドと同じにしておく）でもいいので必ず書きましょう．

さて，モードを使うと使えるキーバインドを一気に増やすことができます！キーバインド同士の被りを避けるために `Ctrl` `Alt` `Shift` 等の組み合わせに苦心しなくても，たとえば
```
bindsym Mod4+l mode "layout"
mode "layout" {
	bindsym Return mode "default"
	bindsym h layout splith; mode "default"
	bindsym v layout splitv; mode "default"
	bindsym s layout stacking; mode "default"
	bindsym t layout tabbed; mode "default"
	bindsym f floating toggle; mode "default"
}
```
のように書いておくだけで，ウィンドウを横に並べる `Mod4+l` `h`，縦に並べる `Mod4+l` `v`，スタック状に重ねる `Mod4+l` `s`，タブ状に重ねる `Mod4+l` `t`，タイル／フロート間で切り替える `Mod4+l` `f` と一度に 5 つのコマンドが定義できるわけです（※セミコロン `;` で区切るのはシェルと同じで，複数のコマンドを順に実行できます）．押すキーの個数は増えますが，単純計算で使えるキーの組み合わせが 26 倍に増えるため被りなんて気にしないでよくなります！

しかしこうやってキーバインドを増やしすぎると，自分でどう設定したか分からなくなってしまいますね．そういうときは，モードに入ったときモードの名前がステータスバーに表示されることを利用して，名前を `"layout"` から `"layout [h: splith] [v: splitv] [s: stacking] [t: tabbed] [f: floating toggle]"` のようなものに変えておくと便利です．長いので変数でおいておくといいでしょう．

```
set $layout "layout [h: splith] [v: splitv] [s: stacking] [t: tabbed] [f: floating toggle]"
bindsym Mod4+l mode $layout
mode $layout {
	bindsym Return mode "default"
	bindsym h layout splith; mode "default"
	bindsym v layout splitv; mode "default"
	bindsym s layout stacking; mode "default"
	bindsym t layout tabbed; mode "default"
	bindsym f floating toggle; mode "default"
}
```

# コマンド
`bindsym` / `bindcode` でキーやボタンに対応させることのできるコマンドとして，ここまで `exec` や `kill` や `floating toggle` などが登場しましたが，他にも多くのコマンドが存在します．

まずいくつか用語を説明します．複数のウィンドウはある**コンテナ**に属することでタイル状に並びます．コンテナは**水平**か**垂直**いずれかの状態をもち，水平なコンテナに属するウィンドウは横に，垂直なコンテナに属するウィンドウは縦に並びます．コンテナがさらに他のコンテナに属していれば，画面が左右に分かれ，うち右半分がさらに上下に分かれるということもできます（水平なコンテナの中に垂直なコンテナがある）．ウィンドウ自体もコンテナです．コンテナ A がコンテナ B を含むとき， A を B の**親**， B を A の**子**といいます．常にいずれか 1 つのコンテナが**フォーカス**されており， `kill` や `resize` などのコマンドはフォーカスされたコンテナに作用します．

以下， `$mod` は `Mod1` や `Mod4` などの修飾キーを指す変数とします．
## `exec <command>`
`exec` はコマンドをシェルに渡して実行します．
```
bindsym $mod+g exec gimp
```
`Mod1+g` を押すと，シェル上で `$ gimp` を実行したのと同じことになります．

## `split vertical` `split horizontal` `split toggle`
`split` は，フォーカスされたコンテナ A を， A を子としてもつコンテナ X に変えます．その後 X に属する新しいウィンドウ B を作れば， A と B は縦か横に並ぶことになります．コンテナの状態は， `split vertical` であれば垂直， `split horizontal` であれば水平になります． `split toggle` は垂直／水平を入れ替えます．
```
bindsym $mod+v split vertical
bindsym $mod+h split horizontal
bindsym $mod+t split toggle
```
## `layout splitv` `layout splith` `layout stacking` `layout tabbed`
`layout` は，フォーカスされたコンテナ（またはフォーカスされたウィンドウが属するコンテナ）のレイアウトを変更します．`splitv` は垂直で，ウィンドウが縦に並びます．`splith` は水平で，ウィンドウが横に並びます．`stacking` は垂直ですがウィンドウ本体は重なり，タイトルバーだけが縦に並びます． `tabbed` も水平ですがウィンドウ本体は重なり，タイトルバーだけが横（タブ状）に並びます．
```
bindsym $mod+s layout stacking
bindsym $mod+w layout tabbed
```

`layout toggle` でこれらのレイアウトを順に切り替えることができます．
```
bindsym $mod+x layout toggle stacking tabbed splith
```
これで `$mod+x` を押すたびに `stacking`→`tabbed`→`splith`→`stacking`→……の順に切り替わります． `toggle all` なら `stacking`→`tabbed`→`splith`→`splitv`→`stacking`， `toggle` だけなら `stacking`→`tabbed`→`split`（`splith`または`splitv`のうち最後に使われていた方）になります．
## `fullscreen enable` `fullscreen disable` `fullscreen toggle`
`fullscreen enable` でフルスクリーンになり， `fullscreen disable` で解除されます． `fullscreen toggle` は 2 つを切り替えます．
## `floating enable` `floating disable` `floating toggle`
`floating enable` でフロート状になり， `floating disable` でタイル状になります． `floating toggle` は 2 つを切り替えます．
## `focus (対象)`
フォーカスの対象を変えるには `focus` を使います．

`focus left` で左，`focus right` で右，`focus up` で上，`focus down` で下，`focus parent` で親，`focus child` で子のコンテナをフォーカスします．`focus next` は水平なら右，垂直なら下になり， `focus prev` は水平なら左，垂直なら上になります．

`focus floating` は最後にフォーカスしていたフロート状のコンテナ，`focus tiling` は最後にフォーカスしていたタイル状のコンテナをフォーカスし，`mode_toggle` はこれらの間で切り替わります．
```
bindsym $mod+j focus left
bindsym $mod+k focus down
bindsym $mod+l focus up
bindsym $mod+semicolon focus right
bindsym $mod+u focus parent
bindsym $mod+g focus mode_toggle
```
また，ウィンドウのクラス（`xprop` コマンドの `WM_CLASS`）などを用いて特定のウィンドウを指定してフォーカスすることもできます．
```
bindsym $mod+F1 [class="Firefox"] focus
```
これはブラウザ firefox のウィンドウをフォーカスします．
## `move (方向)` `move position (位置)`
画面上でウィンドウを移動させるには `move` を使います．

`move left` で左，`move right` で右，`move down` で下，`move up` で上に移動します．デフォルトの移動量は 10 px ですが， `move left 20 px` のようにして指定もできます．

`move position 200 px 100 px` で座標 (200 px, 100 px) の点に移動します． `move position center` で画面の中心， `move position mouse` でマウスのある位置に移動します．

```
bindsym $mod+j move left
bindsym $mod+k move down
bindsym $mod+l move up
bindsym $mod+semicolon move right
```
## `workspace <name>` `workspace prev` `workspace next`
ワークスペースは， Mac の操作スペースとか Windows の仮想デスクトップと同じようなものです．

`workspace 1` はワークスペース 1 を開きます．`workspace browser` や `workspace slack` など好きな名前が使えます． `workspace next` `workspace prev` を使ってワークスペース間を順次切り替えることができます．直前に開いていたワークスペースに戻るには `workspace back_and_forth` とします．

## `move container to workspace <name>`
コンテナを特定のワークスペースに移動します．これも名前の代わりに `prev` `next` が使えます．
## `rename workspace <old_name> to <new_name>`
ワークスペースの名前を変更します．古い名前を省略すると，今いるワークスペースの名前が変更されます．
## `resize grow (方向)` `resize shrink (方向)` `resize set (幅) (高さ)`
`resize` は，フォーカスされたウィンドウの大きさを変更します．

`resize grow up` は上に，`resize grow down` は下に，`resize grow left` は左に，`resize grow right` は右に大きくなります．`shrink` は逆に小さくなります．デフォルトの変化量は 10 px ですが， `resize grow up 20 px` のようにして指定もできます．

`resize set 640 px 480 px` で幅 640 px 高さ 480 px になります．
## `border normal` `border pixel` `border none`
`border` は，フォーカスされたウィンドウの境界線のスタイルを変更します．

`border normal` は普通の境界線で，タイトルバーも表示されます．`border pixel` だと，ウィンドウを囲む枠はありますがタイトルバーはありません．`border none` は境界線なしです．`border pixel 1` のようにして枠線の太さを指定できます．`border normal 0` ならタイトルバーは表示されますが枠はありません．
```
bindsym $mod+t border normal 0
bindsym $mod+y border pixel 3
```
## `reload` `restart` `exit`
`reload` は config を再読込みします．`restart` は i3 を再起動し，`exit` は i3 を終了します．
```
bindsym $mod+Shift+r restart
bindsym $mod+Shift+w reload
bindsym $mod+Shift+e exit
```
## `mark <identifier>`
`mark` を使うと，フォーカスされたコンテナにマークを付け， `focus` コマンド等で利用することができます．

`mark X` でコンテナに `X` というマークが付きます．その後， `focus` コマンドで `[con_mark="X"] focus` とするといつでもそのコンテナに戻ってくることができます．
```
# 今のコンテナに mark-A というマークを付ける
bindsym Mod4+a mark mark-A
# mark-A というマークが付いたコンテナをフォーカスする
bindsym Mod4+Shift+a [con_mark="mark-A"] focus
```

# i3 は多機能！
i3 にはこんなに**多くの機能**があるんだから使わなきゃ勿体ない！今すぐ**あなただけの config** を書いて**幸せ**になりましょう！！！

では！！！！！
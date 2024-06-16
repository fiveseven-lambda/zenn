---
title: "Re: C言語でひらがなの部分文字列"
emoji: "😃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["C"]
published: true
---

[この記事](https://zenn.dev/masakielastic/articles/fc63a2951b32c5)を見て，おお〜こりゃ便利だとなりました😂
それで早速使ってみたら，変なことになってしまい……🥲

```cpp
#include <stdio.h>
#include <stdint.h>

int main(void){
    uint8_t *str = "ひらがな";
    printf("%.*s\n", 9, str);
}
```

出力が「ひらか」になってしまったんですね😯

> ひらがなの場合、1文字は3バイトなので、

どうやら濁点が含まれるとそうとは限らないらしく，なるほどと🤔

同じ記事に書かれていた，`strndup` の方でも，うまく行かず……😭
```cpp
#include <stdio.h>
#include <stdint.h>
#include <string.h>

int main(void){
    uint8_t *src = "ひらがな";
    uint8_t *dest = strndup(src, 9);
    printf("%s\n", dest);
}
```

調べていると，どうやらこういう場合は ICU (International Components for Unicode) というものを使うのが良いのだそうです✋😃

コード例です👇
```cpp
#include <stdio.h>
#include <unicode/ubrk.h>

int main(void){
    uint8_t *str = "ひらがな";
    UErrorCode err = U_ZERO_ERROR;
    UText *text = utext_openUTF8(NULL, str, -1, &err);
    UBreakIterator *iter = ubrk_open(UBRK_CHARACTER, "ja_JP", NULL, 0, &err);
    ubrk_setUText(iter, text, &err);
    int32_t len = 0;
    for(int i = 0; i < 3; i++){
        printf("%d文字目は%dバイト目から", i + 1, len + 1);
        len = ubrk_next(iter);
        printf("%dバイト目まで\n", len);
    }
    printf("%.*s\n", len, str);
    utext_close(text);
    ubrk_close(iter);
}
```
出力は以下のようになりました！
```
1文字目は1バイト目から3バイト目まで
2文字目は4バイト目から6バイト目まで
3文字目は7バイト目から12バイト目まで
ひらが
```
ちゃんと「ひらが」になっています😆

「が」は7バイト目から12バイト目までの6バイトだったんですね🤣

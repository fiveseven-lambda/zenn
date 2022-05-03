---
title: "Result"
---

# Result
文字列スライス型 `str` には，文字列から数値等の変換を行う `parse::<F>()` というメソッドがあります．型パラメータ `F` には変換先の型が入ります．

たとえば `"120"` の指す中身はあくまで `[0x31, 0x32, 0x30]` という `u8` 値の並びに過ぎませんが，`parse::<i32>()` を使うとここから 120 という `i32` 型の値を得ることができるわけです．

しかし，どんな文字列も `i32` に変換できるわけではありません．`parse::<i32>()` が受け取る文字列は `"xxx"` かもしれませんし，空文字列 `""` かもしれませんし，`i32` で表せない `"2147483648"` かもしれません．

このようなとき，前の章で登場した `Option` を使えば，成功したとき `Some()` を返し，失敗したとき `None` を返すことができます．しかし，これだと `None` のとき「失敗した」ことは分かっても「なぜ失敗したか」は分かりません．余計な文字が含まれていたから失敗したのか，空文字列だったから失敗したのか，その整数型で表現できる範囲を超えていたから失敗したのか．失敗したときはその理由も返り値に含めたいです．

このようなとき，`Option` と同じく標準ライブラリで定義された列挙型である [`Result`](https://doc.rust-lang.org/std/result/enum.Result.html) を使います．
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
`Result` の列挙子は，成功したことを表す `Ok(T)` と，失敗したことを表す `Err(E)` の 2 つです．`Result` を使うと，`Err(E)` の `E` にエラーの詳細を含められるので，`Option` と違って失敗したときの返り値にも情報をもたせることができます．[公式ドキュメントの解説](https://doc.rust-lang.org/std/result/)では例を交えてその役割を説明しています．

`Option` の列挙子と同様，`Result::Ok`，`Result::Err` はそれぞれ単に `Ok`，`Err` と書くことができます．また，`==` `!=` による比較も同様に行うことができます．さらに[色々なメソッド](https://doc.rust-lang.org/std/result/enum.Result.html#implementations)を実装しており，その中には以下のものもあります．

- `Option` の `is_some()` `is_none()` に相当する `is_ok()` `is_err()`
- `Option` の `unwrap()` `expect()` `unwrap_or()` に相当する `unwrap()` `expect()` `unwrap_or()`

## 使用例
`Result` を返す `parse()` を例にとって，`Result` がどう使えるか見てみましょう．
```rust
let result = "120".parse();
assert!(matches!(result, Ok(120)));
assert!(result.is_ok());
assert_eq!(result.unwrap(), 120); 
```
1 行目で `parse()` を呼んでいます．`"120"` は 120 という整数に変換できるので，2 行目で `result` が `Ok(120)` となっていることが確認できます．3 行目では `is_ok()` を呼び出しており，これは `Ok` のとき `true` になります．4 行目では `unwrap()` を使って中身を取り出し，120 に等しいことを確認しています．

一方，数値への変換に失敗する場合は次のようになります．
```rust
let result = "xxx".parse();
assert!(matches!(result, Err(_)));
assert!(result.is_err());
if let Err(ref err) = result {
    eprintln!("{}", err);
}
assert_eq!(result.unwrap_or(-1), -1);
```
1 行目で `parse()` を呼んでいます．`"xxx"` は整数に変換できないので，2 行目で `result` が `Err(_)` となっていることが確認できます．3 行目では `is_err()` を呼び出しており，これは `Err` のとき `true` になります．その後，`if let` を使ってエラーの中身を取り出し，`eprintln!` マクロで出力しています．今回は余計な文字が含まれた文字列を変換しようとしたので，`invalid digit found in string` といった出力になります．最後の行で `unwrap_or()` メソッドを呼び出しています．`result` が `Err` なので，`unwrap_or()` は引数 `-1` の値をそのまま返します．
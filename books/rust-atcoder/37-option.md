---
title: "Option"
---

[`Option`](https://doc.rust-lang.org/stable/std/option/enum.Option.html) は，標準ライブラリで定義されている列挙型です．中身は非常に単純です：
```rust
enum Option<T> {
    None,
    Some(T),
}
```
たとえば，`Option<i32>` は，`Some(-2147483648)` から `Some(2147483647)` までの $2^{31}$ 通りに `None` をくわえた $2^{31} + 1$ 通りの値をとることができます．


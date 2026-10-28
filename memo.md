# 技術メモ

- pub(super): 親モジュール内
- pub(crate): クレート内
- pub(in ): 指定したモジュール内

```rust
// newtype: コンポジションの形 要素が一つのタプルを持つ構造体のような感じ
pub struct ImageAttachment(Attachment);

self.0.get_filename() // みたいな感じでアクセスする
```

// cfg: 条件付きコンパイル
#[cfg(feature = "discord")]

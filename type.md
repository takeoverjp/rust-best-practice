# 型

- [`as`によるキャストではなく、`from`/`into`による変換を使用する](#asによるキャストではなくfromintoによる変換を使用する)
- [セマンティクスを強制すべきときは、プリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う](#セマンティクスを強制すべきときはプリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う)
- [`struct`のフィールドでは、参照より所有を優先する](#structのフィールドでは参照より所有を優先する)
- [`Ord`, `PartialOrd`を実装するときは、本当に対象の型が表す情報に順序関係があるのかを考慮し、順序関係がないべきものには実装しないように気をつける](#ord-partialordを実装するときは本当に対象の型が表す情報に順序関係があるのかを考慮し順序関係がないべきものには実装しないように気をつける)

## `as`によるキャストではなく、`from`/`into`による変換を使用する

- 検出方法：[clippyの`as_conversions`](https://rust-lang.github.io/rust-clippy/master/#as_conversions)

- `as`によるキャストは、データが失われる変換も許されるため、一貫性と安全性のために避けるべき
- キャストのセマンティクスを完全に理解し、かつ、Cとの互換性などのために必要なときのみ`as`を使用する

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目５
- 参考：[Let’s deprecate `as` for lossy numeric casts](https://internals.rust-lang.org/t/lets-deprecate-as-for-lossy-numeric-casts/16283)

## セマンティクスを強制すべきときは、プリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う

- newtypeイディオムで専用型を定義することで、異なるセマンティクスを持った値をコンパイラ・IDEが区別できるようにする
- プリミティブ型の場合、異なるセマンティクスを持った値を区別する責務が呼び出し側に委ねられてしまう

- 例

```rust
/// Units for force. 力の単位
pub struct PoundForceSeconds(pub f64);

/// Units for force. 力の単位
pub struct NewtonSeconds(pub f64);
```

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目６

## `struct`のフィールドでは、参照より所有を優先する

- フィールドに参照をもつ`struct`は、その`struct`自体を所有するデータ構造も含めて、参照の生存期間しか有効でないため、使い勝手が悪い
- そのため、可能な限り内容を所有するデータ構造を使う
- 所有が難しい場合は、`Rc`等のスマートポインタにより、生存期間の制限を緩和することが望ましい

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１４


## `Ord`, `PartialOrd`を実装するときは、本当に対象の型が表す情報に順序関係があるのかを考慮し、順序関係がないべきものには実装しないように気をつける

- 大小関係がないenumなどに`Ord`, `PartialOrd`を実装すると、API利用者が誤って大小関係を前提としたコードを書いてしまう可能性がある

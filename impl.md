# 実装

- [有用なlintはデフォルトで有効にする](#有用なlintはデフォルトで有効にする)
- [Don't panic](#dont-panic)
- [明示的なループの代わりにイテレータ変換を使う](#明示的なループの代わりにイテレータ変換を使う)
- [状態遷移を表現するときは、Typestateパターンを使う](#状態遷移を表現するときはtypestateパターンを使う)
- [ジェネリクスとトレイトオブジェクトを適切に使い分ける](#ジェネリクスとトレイトオブジェクトを適切に使い分ける)
- [暗黙的な整数のラッピングをつかわない](#暗黙的な整数のラッピングをつかわない)
- [生ポインタへのキャストで、型推論を使わない](#生ポインタへのキャストで型推論を使わない)
- [文字列を結合するときはformat!を使う](#文字列を結合するときはformatを使う)
- [newtypeパターンで定義した型に`Deref`トレイト・`DerefMut`トレイトを実装するかは慎重に判断する](#newtypeパターンで定義した型にderefトレイトderefmutトレイトを実装するかは慎重に判断する)
- [値は使わないが、RAIIで副作用を起こすために変数に束縛したい場合は、`_`ではなく`_xxx`に束縛する](#値は使わないがraiiで副作用を起こすために変数に束縛したい場合は_ではなく_xxxに束縛する)
- [warningを撲滅するときは、`#![deny(warnings)]`ではなく、`-D warnings`を使う](#warningを撲滅するときはdenywarningsではなく-d-warningsを使う)
- [`Rc`を`clone()`するときは、`rc.clone()`ではなく`Rc::clone(&rc)`と書く](#rcをcloneするときはrccloneではなくrcclonercと書く)

## 有用なlintはデフォルトで有効にする

- [`#![deny(missing_debug_implementations)]`](https://doc.rust-lang.org/stable/nightly-rustc/rustc_lint/builtin/static.MISSING_DEBUG_IMPLEMENTATIONS.html)
  - 説明：`Debug`トレイトを実装していない型がある場合にコンパイルエラーにする
  - 理由：`Debug`トレイトを実装していない方はデバッグ効率が悪いため
  - 背反：コンパイル時間とコードサイズが増えること、`Debug`トレイトを実装するboilerplateコードが増えること

```
#![deny(missing_docs)]
#![deny(broken_intra_doc_links)]
#![deny(rustdoc::missing_crate_level_docs)]
```

- [`#![deny(unsafe_op_in_unsafe_fn)]`](https://doc.rust-lang.org/stable/nightly-rustc/rustc_lint/builtin/static.UNSAFE_OP_IN_UNSAFE_FN.html)
  - 説明：`unsafe`関数の中で、`unsafe`ブロックを使わずに`unsafe`コードを実行することを禁止する
  - 理由：`unsafe`関数の中であっても、`unsafe`ブロックを使うことで、どこが`unsafe`コードなのかを明示的にすることができ、レビュー効率が向上するため
  - 背反：特になし。互換性のために現時点では`warn`だが、将来的にはデフォルトで`deny`になる可能性も十分ある。

- 参考：[Crust of Rust: Lifetime Annotations](https://youtu.be/rAl-9HwD858?list=PLqbS7AVVErFiWDOAVrPt7aYmnuuOLYvOa&t=343)
- 参考：[Conprehensive Rust - Unsafe関数の呼び出し](https://google.github.io/comprehensive-rust/ja/unsafe-rust/unsafe-functions.html)
- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目27

## Don't panic

- 理由
  - panic時の挙動はプロジェクト全体の設定次第
  - 例外安全性が必要になる
  - ffi境界で未定義動作になる

- 検出方法
  - [no_panic crate](https://crates.io/crates/no-panic)

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目４

## 明示的なループの代わりにイテレータ変換を使う

- `for`ループや`while`ループの代わりに、イテレータ変換を使うことで、コードが簡潔になり、パフォーマンスも向上することがある
- イテレータ変換メソッドが多数用意されているのは、これらを使うことでコードがよりRustらしく、よりコンパクトになり、意図が明確になるため

- イテレータ変換
  - `take(n)`, `skip(n)`, `step_by(n)`, `chain(other)`, `zip(other)`, `cycle()`, `rev()`, `map(f)`, `cloned()`, `copied()`, `enumerate()`, `zip(it)`, `filter(f)`, `take_while()`, `skip_while()`, `flatten()`
  - `flatten()`は、`Result`や`Option`のイテレータに対して、`Some(v)`もしくは`Ok(v)`の場合のみ`v`を出力するイテレータにする
- イテレータ消費
  - `for_each(f)`, `sum()`, `product()`, `min()`, `max()`, `min_by(f)`, `max_by(f)`, `reduce(f)`, `fold(f)`, `scan(init, f)`
  - 1つの値を選択するメソッド
    - `find(p)`, `position(p)`, `nth(n)`
  - テストを行うメソッド
    - `all(p)`, `any((p)`
  - クロージャの失敗を許容するメソッド
    - `try_fold(f)`, `try_find(f)`, `try_for_each(f)`
  - 新しいコレクションを作るメソッド
    - `collect()`, `unzip()`, `partition()`

## 状態遷移を表現するときは、Typestateパターンを使う

- 参考：[Rust タイプステートパターンによるAPI設計 | Yabutan 技術ブログ ](https://share.google/GqhRWK2ya2hmuKv0f)

## ジェネリクスとトレイトオブジェクトを適切に使い分ける

- ジェネリクス
  - 実行時のオーバーヘッドがない
  - コードサイズが大きくなる
  - コンパイル時間が長くなる
  - 複数のトレイト制約を課すことが容易
  - サブトレイトを実装したアイテムを、スーパートレイト制約を持つメソッドに渡せない（vtableが融合されていてアップキャストできない）
  - 型消去により、異なる具象型のアイテムを１つのコレクションに収めることができる
  - コンパイル時に単相化対象の型がわからない場合でも使用できる（`dlopen(3)`など）
- トレイトオブジェクト
  - ２段階参照に伴う実行時のオーバーヘッドがある
  - コードサイズが小さくなる
  - コンパイル時間が短くなる
  - 複数のトレイト制約を課すことが難しい
  - サブトレイトを実装したアイテムを、スーパートレイト制約を持つメソッドに渡せる（コンパイル時に具象型のスーパートレイトメソッドを利用するように単相化される）
  - 異なる具象型のアイテムを１つのコレクションに収めることができない
  - コンパイル時に単相化対象の型がわからない場合は使用できない（`dlopen(3)`など）

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１２

## 暗黙的な整数のラッピングをつかわない

- 参考: [Avoid Implicit Integer Wrapping](https://coding-guidelines.arewesafetycriticalyet.org/coding-guidelines/types-and-traits.html)

## 生ポインタへのキャストで、型推論を使わない

- 参考：[Avoid as underscore pointer casts](https://coding-guidelines.arewesafetycriticalyet.org/coding-guidelines/expressions.html#gui_HDnAZ7EZ4z6G)

## 文字列を結合するときはformat!を使う

- 参考：[Concatenating strings with format!](https://rust-unofficial.github.io/patterns/idioms/concat-format.html)

## newtypeパターンで定義した型に`Deref`トレイト・`DerefMut`トレイトを実装するかは慎重に判断する

- 参考：https://stackoverflow.com/questions/45086595/is-it-considered-a-bad-practice-to-implement-deref-for-newtypes

- 反対派
  - Derefはスマートポインタ用に設計されている。
  - newtypeに適用すると、has-a関係をis-a関係にすることになる。中の型のAPIを不必要に公開してしまい、継承と同じ強い依存が発生する。
  - 必要なAPIのみ、明示的に処理の委譲を書くべき
  - 言語仕様で処理の委譲をサポートしてくれることが望ましいが、それまではdelegateクレートなどでサボることができる
- 賛成派
  - 多くのnewtypeは、その型が満たすべきある不変性を静的に確認するために使われる。オブジェクト構築後はnewtype自体に意味はなく、中身をそのまま露出したいので、都度 .0を書くのは冗長。
    - （この意見に対する反対意見）DerefMutしたら、不変性も壊される

## 値は使わないが、RAIIで副作用を起こすために変数に束縛したい場合は、`_`ではなく`_xxx`に束縛する

- `_`は値を束縛しないので、`_`を定義したスコープが生存期間と扱われない

## warningを撲滅するときは、`#![deny(warnings)]`ではなく、`-D warnings`を使う

- 参考：https://rust-unofficial.github.io/patterns/anti_patterns/deny-warnings.html

## `Rc`を`clone()`するときは、`rc.clone()`ではなく`Rc::clone(&rc)`と書く

- 理由：他の`clone()`呼出と区別を付きやすくすることで、パフォーマンスの問題が発生したときにディープコピーの箇所を見つけやすくするため

- 参考：https://doc.rust-jp.rs/book-ja/ch15-04-rc.html

## 要素をもつenumのバリアントを変換するときは、`std::mem::take`で不要な`clone`を避ける

- https://rust-unofficial.github.io/patterns/idioms/mem-replace.html

## stackに確保した変数でdynamic dispatchする場合、deferred conditional initializationを使う

- 参考:https://rust-unofficial.github.io/patterns/idioms/on-stack-dyn-dispatch.html

## `Option`型をイテレータとして処理するときは、`IntoIterator`トレイトを活用する

- 参考:https://rust-unofficial.github.io/patterns/idioms/option-iter.html

## Closureに変数をキャプチャさせる場合、専用のブロックに専用の変数を定義する

- 参考:https://rust-unofficial.github.io/patterns/idioms/pass-var-to-closure.html

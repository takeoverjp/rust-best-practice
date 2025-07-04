# Rust Best Practices

- [本リポジトリの目的](#本リポジトリの目的)
- [参考文献](#参考文献)
- [API設計](#api設計)
  - [APIを定義する場合は、Rust API Guidelinesに従う](#apiを定義する場合はrust-api-guidelinesに従う)
  - [関数の引数では、所有型の借用より、借用型を優先する (`&str` \> `&String`, `&[T]` \> `&Vec<T>`, `&T` \> `&Box<T>`)](#関数の引数では所有型の借用より借用型を優先する-str--string-t--vect-t--boxt)
  - [複雑な型にはビルダパターンを使う](#複雑な型にはビルダパターンを使う)
  - [Trait objectとして使用することを想定するTraitは、object-safetyを満たすように設計する](#trait-objectとして使用することを想定するtraitはobject-safetyを満たすように設計する)
- [標準ライブラリ内のトレイト](#標準ライブラリ内のトレイト)
  - [機密情報が含まれない型であれば、`Debug`トレイトを実装する](#機密情報が含まれない型であればdebugトレイトを実装する)
  - [解放しなければならない何らかの資源を保持する型には`Drop`トレイトを実装し、RAIIにする](#解放しなければならない何らかの資源を保持する型にはdropトレイトを実装しraiiにする)
  - [自分が実装するクロージャは`Fn` \> `FnMut` \> `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` \> `FnMut` \> `Fn`の順に優先する](#自分が実装するクロージャはfn--fnmut--fnonceの順に優先しトレイト境界への指定はfnonce--fnmut--fnの順に優先する)
- [型](#型)
  - [`as`によるキャストではなく、`from`/`into`による変換を使用する](#asによるキャストではなくfromintoによる変換を使用する)
  - [セマンティクスを強制すべきときは、プリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う](#セマンティクスを強制すべきときはプリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う)
- [エラーハンドリング](#エラーハンドリング)
  - [`Option`と`Result`は`match`を用いずに変換する](#optionとresultはmatchを用いずに変換する)
  - [ライブラリでは、`thiserror`クレートを使って具体的で詳細なエラー情報を呼び出し側に伝える](#ライブラリではthiserrorクレートを使って具体的で詳細なエラー情報を呼び出し側に伝える)
  - [アプリケーションでは、`anyhow`クレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する](#アプリケーションではanyhowクレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する)
- [実装](#実装)
  - [明示的なループの代わりにイテレータ変換を使う](#明示的なループの代わりにイテレータ変換を使う)
- [CI](#ci)
  - [`cargo udeps`で不必要な依存クレートをチェックする](#cargo-udepsで不必要な依存クレートをチェックする)
  - [`cargo deny`で依存クレートをチェックする](#cargo-denyで依存クレートをチェックする)


## 本リポジトリの目的

本リポジトリには、Rust開発におけるベストプラクティスを集約する。  

- 記載ルール
  - 関連する資料を可能な限り参照する
  - clippy等を使って機械的にチェックできるものは、その手段も記載する

## 参考文献

- [Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)
- [Rust for Rustaceans](https://rust-for-rustaceans.com/)
- [Comprehensive Rust](https://google.github.io/comprehensive-rust/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- Rustにおける破壊的変更
  - [Rust RFC 1105](https://rust-lang.github.io/rfcs/1105-api-evolution.html)
  - [The Cargo Book on SemVer compatibility](https://doc.rust-lang.org/cargo/reference/semver.html)
- [Rust Design Patterns](https://rust-unofficial.github.io/patterns/)
- [Clippy Lints](https://rust-lang.github.io/rust-clippy/master/)
- [Rust Cook Book](https://rust-lang-nursery.github.io/rust-cookbook/)

## API設計

### APIを定義する場合は、[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)に従う

- まずは一通り目を通して理解をする
- 実際の設計時には、各APIに対して[チェックリスト](https://rust-lang.github.io/api-guidelines/checklist.html)でチェックする

- 参考：[Rust for Rustaceans](https://rust-for-rustaceans.com/)のChapter 3 "Designing Interfaces"

### 関数の引数では、所有型の借用より、借用型を優先する (`&str` > `&String`, `&[T]` > `&Vec<T>`, `&T` > `&Box<T>`)

- `&str`であれば文字列リテラル(`&'static str`)やslice(`&str`)も受け取ることができるが、`&String`では受け取れない

- 参考：[Use borrowed types for arguments](https://rust-unofficial.github.io/patterns/idioms/coercion-arguments.html)

### 複雑な型にはビルダパターンを使う

- `struct`を初期化するためには、`Default`トレイトを実装している場合を除いて、すべての要素を書き出さなければならないが、それは煩雑なだけではなくメンバの追加が難しくなる
- ビルダパターンを使うことで、初期化時の煩雑さを軽減し、メンバの追加も容易になる
- ビルダパターンには、以下のパターンがある
  - パターン１：`self`を消費し、`self`を返す
    - ○：ビルダの構築とセッタの呼出をワンライナーで書ける
    - ×：ビルダ一つに付き、一回しかアイテムをビルドできない（`Clone`トレイトを実装できるなら、大した問題ではない）
  - パターン２：排他参照を受け取り、排他参照を返す
    - ○：ビルダ一つに付き、複数回アイテムをビルドできる
    - ×：ビルダの構築とセッタの呼出を分けて記述する必要がある

```rust
// ビルダパターンを使わない例
```rust
let me = Details {
  given_name: "Takeo".to_owned(),
  preferred_name: Some("Take".to_owned()),
  middle_name: Some("".to_owned()),
  family_name: "Kondo".to_owned(),
  mobile_phone: None,
  date_of_birth: time::Date::from_calendar_date(
    1984,
    time::Month::September,
    23,
  )
  .unwrap(),
  last_seen: None,
};
```

```rust
// パターン１：`self`を消費し、`self`を返すの例
let mut builder = DetailsBuilder::new(
  "Takeo",
  "Kondo",
  time::Date::from_calendar_date(1984, time::Month::September, 23).unwrap(),
);
if informal {
  builder = builder.preferred_name("Take");
}
let me = builder.build();
```

```rust
// パターン２：排他参照を受け取り、排他参照を返す例
let mut builder = DetailsBuilder::new(
  "Takeo",
  "Kondo",
  time::Date::from_calendar_date(1984, time::Month::September, 23).unwrap(),
);
builder.middle_name("the").just_seen();
if informal {
  builder.preferred_name("Take");
}
let me = builder.build();
```

### Trait objectとして使用することを想定するTraitは、[object-safety](https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md)を満たすように設計する

- 同じTraitを実装する異なる型のインスタンスをコレクションに入れるなど、Trait objectとして使うことが想定されるTraitの場合、[object-safety](https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md)を満たすように設計しなければ、利用者がコンパイルエラーに遭遇してしまう
- object-safetyを満たさないTraitを満たすように修正する場合、破壊的変更を避けるのは難しいので、予めAPI設計時に考慮することが望ましい
- object-safetyを満たすためには、以下の２つのルールに従わなければならない
  - トレイとメソッドはジェネリックであってはならない
  - トレイとメソッドでは、レシーバ（メソッドの第一引数）以外に、`Self`を含む方を使ってはならない

- 参考：[Let's Get Rusty - Using Trait Objects in Rust](https://youtu.be/ReBmm0eJg6g?list=PLai5B987bZ9CoVR-QEIN9foz4QCJ0H2Y8&t=743)
- 参考：[object-safetyの定義]([object-safety](https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md))
- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１２

## 標準ライブラリ内のトレイト

### 機密情報が含まれない型であれば、`Debug`トレイトを実装する

- 基本はderiveマクロによる自動導出をする
- 自動導出だと煩雑すぎる場合は、独自で実装する

- 検出方法：[`rustc` lintの`missing_debug_implementations`](https://doc.rust-lang.org/stable/rustc/lints/listing/allowed-by-default.html#missing-debug-implementations)

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１０

### 解放しなければならない何らかの資源を保持する型には`Drop`トレイトを実装し、RAIIにする

- `Drop`トレイトを実装することで、型のインスタンスがスコープを抜けるときに、自動的に資源を解放することができる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１２

### 自分が実装するクロージャは`Fn` > `FnMut` > `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` > `FnMut` > `Fn`の順に優先する

- 自分がクロージャを実装する場合、呼び出し元の制約が少ない順に優先する
- トレイト境界を指定する場合、受け入れられるクロージャを広くするために、制約が多い順に優先する

- `Fn` : 共有参照しかキャプチャしていないので、何回でも呼出でき、再帰呼出しもできる
- `FnMut` : 排他参照をキャプチャしているので、何回でも呼出できるが、再帰呼出しはできない
- `FnOnce` : 所有権をmoveするので、１回しか呼び出せない（⇒再帰呼出しもできない）

- 参考：[Comprehensive Rust - Closures](https://google.github.io/comprehensive-rust/ja/std-traits/closures.html)

## 型

### `as`によるキャストではなく、`from`/`into`による変換を使用する

- 検出方法：[clippyの`as_conversions`](https://rust-lang.github.io/rust-clippy/master/#as_conversions)

- `as`によるキャストは、データが失われる変換も許されるため、一貫性と安全性のために避けるべき
- キャストのセマンティクスを完全に理解し、かつ、Cとの互換性などのために必要なときのみ`as`を使用する

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目５
- 参考：[Let’s deprecate `as` for lossy numeric casts](https://internals.rust-lang.org/t/lets-deprecate-as-for-lossy-numeric-casts/16283)

### セマンティクスを強制すべきときは、プリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う

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

## エラーハンドリング

### `Option`と`Result`は`match`を用いずに変換する

- `match`を使わない値の取り出し方
  - エラーを無視する場合 : `if let`
  - エラー時に`panic!`する場合 : `unwrap()`, `expect()`
  - エラー処理を上位に委ねる場合 : `?`演算子

- [OptionとResultの変換](https://docs.google.com/drawings/d/1EOPs0YTONo_FygWbuJGPfikO9Myt5HwtiFUHRuE1JVM/preview)
  - `as_ref()`を使うことで、`Option`への参照を参照への`Option`に変換できる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目３

### ライブラリでは、`thiserror`クレートを使って具体的で詳細なエラー情報を呼び出し側に伝える

- `thiserror`クレートは、元になるエラーが複数あり、それらの型を保持する必要があるときに使う
  - `thiserror`クレートは、生成されたAPIに`thiserror`が現れないように設計されているので、ライブラリでも使用できる
- `anyhow`クレートは、エラーをトレイトオブジェクトとして扱うため、利用者側で詳細なエラー情報に応じた処理をすることが難しいので、ライブラリでは使用しない

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目４

### アプリケーションでは、`anyhow`クレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する

- `anyhow`クレートは、エラーをトレイトオブジェクトとして扱うため、依存するすべてのライブラリのエラーを単純に一貫して扱うことができる
- `thiserror`クレートは、元になるエラーが複数あり、それらの型を保持する必要があるときに使う

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目４

## 実装

### 明示的なループの代わりにイテレータ変換を使う

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

## CI

### `cargo udeps`で不必要な依存クレートをチェックする

- `cargo udeps`により、リファクタリング等で不要となった依存に気づくことができる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目４

### `cargo deny`で依存クレートをチェックする

- `cargo deny`で依存クレートに対して下記のようなチェックができる
  - 使用されているバージョンに既知のセキュリティ問題がある依存ライブラリ
  - 受入不能なライセンスで提供されている依存ライブラリ
  - ただ受け入れられない依存ライブラリ
  - 依存グラフ中で、複数の異なるバージョンが使われている依存ライブラリ

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目４

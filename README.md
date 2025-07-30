# Rust Best Practices

- [本リポジトリの目的](#本リポジトリの目的)
- [参考文献](#参考文献)
- [API設計](#api設計)
  - [APIを定義する場合は、Rust API Guidelinesに従う](#apiを定義する場合はrust-api-guidelinesに従う)
  - [公開範囲を最小化する](#公開範囲を最小化する)
  - [値の所有権が必要な場合、参照を受け取って内部で`clone`するのではなく、所有権をとる](#値の所有権が必要な場合参照を受け取って内部でcloneするのではなく所有権をとる)
  - [関数の引数では、所有型の借用より、借用型を優先する (`&str` \> `&String`, `&[T]` \> `&Vec<T>`, `&T` \> `&Box<T>`)](#関数の引数では所有型の借用より借用型を優先する-str--string-t--vect-t--boxt)
  - [複雑な型にはビルダパターンを使う](#複雑な型にはビルダパターンを使う)
  - [Trait objectとして使用することを想定するTraitは、object-safetyを満たすように設計する](#trait-objectとして使用することを想定するtraitはobject-safetyを満たすように設計する)
  - [デフォルト実装を用意することで、実装しなければならないトレイトメソッドを最小限にする](#デフォルト実装を用意することで実装しなければならないトレイトメソッドを最小限にする)
  - [SemVerを理解する](#semverを理解する)
- [標準ライブラリ内のトレイト](#標準ライブラリ内のトレイト)
  - [機密情報が含まれない型であれば、`Debug`トレイトを実装する](#機密情報が含まれない型であればdebugトレイトを実装する)
  - [解放しなければならない何らかの資源を保持する型には`Drop`トレイトを実装し、RAIIにする](#解放しなければならない何らかの資源を保持する型にはdropトレイトを実装しraiiにする)
  - [自分が実装するクロージャは`Fn` \> `FnMut` \> `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` \> `FnMut` \> `Fn`の順に優先する](#自分が実装するクロージャはfn--fnmut--fnonceの順に優先しトレイト境界への指定はfnonce--fnmut--fnの順に優先する)
- [型](#型)
  - [`as`によるキャストではなく、`from`/`into`による変換を使用する](#asによるキャストではなくfromintoによる変換を使用する)
  - [セマンティクスを強制すべきときは、プリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う](#セマンティクスを強制すべきときはプリミティブ型ではなくnewtypeイディオムで定義した専用の型を使う)
  - [`struct`のフィールドでは、参照より所有を優先する](#structのフィールドでは参照より所有を優先する)
- [エラーハンドリング](#エラーハンドリング)
  - [`Option`と`Result`は`match`を用いずに変換する](#optionとresultはmatchを用いずに変換する)
  - [ライブラリでは、`thiserror`クレートを使って具体的で詳細なエラー情報を呼び出し側に伝える](#ライブラリではthiserrorクレートを使って具体的で詳細なエラー情報を呼び出し側に伝える)
  - [アプリケーションでは、`anyhow`クレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する](#アプリケーションではanyhowクレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する)
- [実装](#実装)
  - [有用なlintはデフォルトで有効にする](#有用なlintはデフォルトで有効にする)
  - [Don't panic](#dont-panic)
  - [明示的なループの代わりにイテレータ変換を使う](#明示的なループの代わりにイテレータ変換を使う)
  - [状態遷移を表現するときは、Typestateパターンを使う](#状態遷移を表現するときはtypestateパターンを使う)
  - [ジェネリクスとトレイトオブジェクトを適切に使い分ける](#ジェネリクスとトレイトオブジェクトを適切に使い分ける)
- [コメント](#コメント)
  - [APIに対するドキュメンテーションコメントは、rust-lang/rfcs#1574に従う](#apiに対するドキュメンテーションコメントはrust-langrfcs1574に従う)
  - [ドキュメンテーションコメントに記載するサンプルコードにおいて、利用者の理解の助けにならない部分は、`/// #`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて利用者の理解の助けにならない部分は-を使う)
  - [ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっていない場合は`/// ```no_run`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて実行可能なテストになっていない場合は-no_runを使う)
- [並列実行](#並列実行)
  - [メモリを共有して通信するのではなく、通信してメモリを共有する](#メモリを共有して通信するのではなく通信してメモリを共有する)
  - [状態共有並列実行は気を付ける](#状態共有並列実行は気を付ける)
- [unsafe](#unsafe)
  - [unsafeコードは書かない](#unsafeコードは書かない)
  - [unsafeコードを書かざるを得ない場合場合、ラッパレイヤに集約する](#unsafeコードを書かざるを得ない場合場合ラッパレイヤに集約する)
  - [unsafe関数の中であっても、明示的にunsafeブロックを使う](#unsafe関数の中であっても明示的にunsafeブロックを使う)
  - [unsafeコードが依存する前提条件と普遍条件を明記する](#unsafeコードが依存する前提条件と普遍条件を明記する)
  - [unsafeコードに対してMiriを実行する](#unsafeコードに対してmiriを実行する)
  - [unsafeコードに対してマルチスレッドで使用される場合を慎重に考える](#unsafeコードに対してマルチスレッドで使用される場合を慎重に考える)
- [依存ライブラリ](#依存ライブラリ)
  - [ワイルドカードインポートはしない](#ワイルドカードインポートはしない)
  - [`cargo udeps`で不必要な依存クレートをチェックする](#cargo-udepsで不必要な依存クレートをチェックする)
  - [`cargo deny`で依存クレートをチェックする](#cargo-denyで依存クレートをチェックする)
  - [`cargo install`するときは、`--locked`オプションをつける](#cargo-installするときは--lockedオプションをつける)
  - [後方互換性を満たすための必要条件を理解し、適切にセマンティックバージョンを管理する](#後方互換性を満たすための必要条件を理解し適切にセマンティックバージョンを管理する)
- [テスト](#テスト)
  - [private関数のテストはソースコードに記載する](#private関数のテストはソースコードに記載する)
  - [public関数のテストはtestsディレクトリに集約する](#public関数のテストはtestsディレクトリに集約する)
  - [cargo mutantsの導入を検討する](#cargo-mutantsの導入を検討する)
  - [コンパイルに失敗することを確認するテストは、docテストの`compile_fail`属性を使う](#コンパイルに失敗することを確認するテストはdocテストのcompile_fail属性を使う)


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
- 参考：[Rust API Guidelines - Structs have private fields (C-STRUCT-PRIVATE)](https://rust-lang.github.io/api-guidelines/future-proofing.html#structs-have-private-fields-c-struct-private)

### 公開範囲を最小化する

- `pub(crate)`, `pub(super)`, `pub(in <path>)`, `pub(self)`などを適切に活用し、クレート外部に公開するシンボルを最低限にする
- そうすることで、APIの破壊的変更やメンテナンスコストを最小限に抑えることができる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２２

### 値の所有権が必要な場合、参照を受け取って内部で`clone`するのではなく、所有権をとる

- 参照を受け取って内部で`clone`する場合、呼び出し側がもう対象の変数が不要であってもメモリコピーを避けることができない
- 所有権を取るAPIとすることで、呼び出し側が所有権を必要なときだけ`clone`を呼び出すことができるようにある

- 参考：[Let's Get Rusty - 5 deadly Rust anti-patterns to avoid](https://youtu.be/SWwTD2neodE?si=1OQG2Vc5jH0yLdnt)

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
- 参考：[Rust API Guidelines - C-OBJECT](https://rust-lang.github.io/api-guidelines/flexibility.html#traits-are-object-safe-if-they-may-be-useful-as-a-trait-object-c-object)

### デフォルト実装を用意することで、実装しなければならないトレイトメソッドを最小限にする

- トレイトには2種類の利用者がいる
  - トレイトを実装するstructの開発者
  - トレイトに依存した処理を書く開発者
- 前者に向けてはトレイトメソッドを少なく、後者に向けてはトレイトメソッドを充実させることが望ましい
- 実装必須のトレイトメソッドを極少数、それらに依存するデフォルト実装つきのトレイトメソッドを充実させることで、両方の要件を満たすことができる
こ
- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１3

### SemVerを理解する

- 何かを変更したらパッチバージョンを変更する
- APIに機能を追加したらマイナーバージョンを更新する
- APIを削除、もしくは変更した場合にはメジャーバージョンを更新する

- 一番左の数字が0のときは、最初の0でない数字をメジャーバージョン相当と考える
  - 例えば、`0.2.3`から`0.3.0`や、`0.0.1`から`0.0.2`は互換性のないのない変更を含む可能性があると考える

- 参考：参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２１

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

### `struct`のフィールドでは、参照より所有を優先する

- フィールドに参照をもつ`struct`は、その`struct`自体を所有するデータ構造も含めて、参照の生存期間しか有効でないため、使い勝手が悪い
- そのため、可能な限り内容を所有するデータ構造を使う
- 所有が難しい場合は、`Rc`等のスマートポインタにより、生存期間の制限を緩和することが望ましい

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１４

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

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目18

## 実装

### 有用なlintはデフォルトで有効にする

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

### Don't panic

- 理由
  - panic時の挙動はプロジェクト全体の設定次第
  - 例外安全性が必要になる
  - ffi境界で未定義動作になる

- 検出方法
  - [no_panic crate](https://crates.io/crates/no-panic)

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目４

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

### 状態遷移を表現するときは、Typestateパターンを使う

- 参考：[Rust タイプステートパターンによるAPI設計 | Yabutan 技術ブログ ](https://share.google/GqhRWK2ya2hmuKv0f)

### ジェネリクスとトレイトオブジェクトを適切に使い分ける

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

## コメント

### APIに対するドキュメンテーションコメントは、[rust-lang/rfcs#1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md)に従う

- 参考：[rust-lang/rfcs#1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md)

### ドキュメンテーションコメントに記載するサンプルコードにおいて、利用者の理解の助けにならない部分は、`/// #`を使う

- 参考：[The rustdoc book - Hiding portions of the example](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html#hiding-portions-of-the-example)

### ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっていない場合は`/// ```no_run`を使う

- 参考：[The rustdoc book - Documentation tests](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html#attributes)

## 並列実行

### メモリを共有して通信するのではなく、通信してメモリを共有する

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目17

### 状態共有並列実行は気を付ける

- Rustはデータ競合を防いでくれるが、デッドロックを防ぐのはプログラマの責務

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目17

## unsafe

### unsafeコードは書かない

### unsafeコードを書かざるを得ない場合場合、ラッパレイヤに集約する

### unsafe関数の中であっても、明示的にunsafeブロックを使う

### unsafeコードが依存する前提条件と普遍条件を明記する

### unsafeコードに対してMiriを実行する

### unsafeコードに対してマルチスレッドで使用される場合を慎重に考える

## 依存ライブラリ

### ワイルドカードインポートはしない

- ワイルドカードインポートは、依存クレートの更新に伴い、シンボルの衝突などにつながるリスクがあるため、避けるべき

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２３

### `cargo udeps`で不必要な依存クレートをチェックする

- `cargo udeps`により、リファクタリング等で不要となった依存に気づくことができる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２５

### `cargo deny`で依存クレートをチェックする

- `cargo deny`で依存クレートに対して下記のようなチェックができる
  - 使用されているバージョンに既知のセキュリティ問題がある依存ライブラリ
  - 受入不能なライセンスで提供されている依存ライブラリ
  - ただ受け入れられない依存ライブラリ
  - 依存グラフ中で、複数の異なるバージョンが使われている依存ライブラリ

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２５

### `cargo install`するときは、`--locked`オプションをつける

- `cargo install`は、デフォルトで`Cargo.lock`を参照せずに毎回`Cargo.toml`で指定された範囲で最新のバージョンの依存ライブラリを使ってビルドするため、再現性がなくなってしまう
- `--locked`オプションをつけることで、`Cargo.lock`に記載されているバージョンの依存ライブラリを使ってビルドすることができ、再現性を確保できる

- 参考：[cargo-install(1) - Dealing with the Lockfile](https://doc.rust-lang.org/cargo/commands/cargo-install.html#dealing-with-the-lockfile)

### 後方互換性を満たすための必要条件を理解し、適切にセマンティックバージョンを管理する

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目21

## テスト

### private関数のテストはソースコードに記載する

### public関数のテストはtestsディレクトリに集約する

### cargo mutantsの導入を検討する

### コンパイルに失敗することを確認するテストは、docテストの`compile_fail`属性を使う

```
/// ```compile_fail
/// let x = 1 + "a";
/// ```
#[allow(dead_code)]
struct CompileFailTest;
```

- 参考：[Crust of Rust - Declarative Macros](https://youtu.be/q6paRBbLgNw?list=PLqbS7AVVErFiWDOAVrPt7aYmnuuOLYvOa&t=3394)

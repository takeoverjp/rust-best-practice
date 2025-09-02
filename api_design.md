# API設計

- [APIを定義する場合は、Rust API Guidelinesに従う](#apiを定義する場合はrust-api-guidelinesに従う)
- [公開範囲を最小化する](#公開範囲を最小化する)
- [値の所有権が必要な場合、参照を受け取って内部で`clone`するのではなく、所有権をとる](#値の所有権が必要な場合参照を受け取って内部でcloneするのではなく所有権をとる)
- [関数の引数では、所有型の借用より、借用型を優先する (`&str` \> `&String`, `&[T]` \> `&Vec<T>`, `&T` \> `&Box<T>`)](#関数の引数では所有型の借用より借用型を優先する-str--string-t--vect-t--boxt)
- [複雑な型にはビルダパターンを使う](#複雑な型にはビルダパターンを使う)
- [Trait objectとして使用することを想定するTraitは、object-safetyを満たすように設計する](#trait-objectとして使用することを想定するtraitはobject-safetyを満たすように設計する)
- [デフォルト実装を用意することで、実装しなければならないトレイトメソッドを最小限にする](#デフォルト実装を用意することで実装しなければならないトレイトメソッドを最小限にする)
- [SemVerを理解する](#semverを理解する)
- [`cargo-semver-check`を使って、バージョン更新のアセスメントを行う](#cargo-semver-checkを使ってバージョン更新のアセスメントを行う)
- [略語は１語として扱う](#略語は１語として扱う)
- [クレート名に`-rs`, `-rust`をつけない](#クレート名に-rs--rustをつけない)
- [ビルダーパターンのAPIを提供するときは、validateを追加する可能性をふまえて、`Result`型を返す](#ビルダーパターンのapiを提供するときはvalidateを追加する可能性をふまえてresult型を返す)

## APIを定義する場合は、[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)に従う

- まずは一通り目を通して理解をする
- 実際の設計時には、各APIに対して[チェックリスト](https://rust-lang.github.io/api-guidelines/checklist.html)でチェックする

- 参考：[Rust for Rustaceans](https://rust-for-rustaceans.com/)のChapter 3 "Designing Interfaces"
- 参考：[Rust API Guidelines - Structs have private fields (C-STRUCT-PRIVATE)](https://rust-lang.github.io/api-guidelines/future-proofing.html#structs-have-private-fields-c-struct-private)

## 公開範囲を最小化する

- `pub(crate)`, `pub(super)`, `pub(in <path>)`, `pub(self)`などを適切に活用し、クレート外部に公開するシンボルを最低限にする
- そうすることで、APIの破壊的変更やメンテナンスコストを最小限に抑えることができる

- 検出方法：[`#![deny(clippy::redundant_pub_crate)]`](https://rust-lang.github.io/rust-clippy/stable/index.html#redundant_pub_crate)

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２２

## 値の所有権が必要な場合、参照を受け取って内部で`clone`するのではなく、所有権をとる

- 参照を受け取って内部で`clone`する場合、呼び出し側がもう対象の変数が不要であってもメモリコピーを避けることができない
- 所有権を取るAPIとすることで、呼び出し側が所有権を必要なときだけ`clone`を呼び出すことができるようにある

- 参考：[Let's Get Rusty - 5 deadly Rust anti-patterns to avoid](https://youtu.be/SWwTD2neodE?si=1OQG2Vc5jH0yLdnt)

## 関数の引数では、所有型の借用より、借用型を優先する (`&str` > `&String`, `&[T]` > `&Vec<T>`, `&T` > `&Box<T>`)

- `&str`であれば文字列リテラル(`&'static str`)やslice(`&str`)も受け取ることができるが、`&String`では受け取れない

- 参考：[Use borrowed types for arguments](https://rust-unofficial.github.io/patterns/idioms/coercion-arguments.html)

## 複雑な型にはビルダパターンを使う

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

## Trait objectとして使用することを想定するTraitは、[object-safety](https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md)を満たすように設計する

- 同じTraitを実装する異なる型のインスタンスをコレクションに入れるなど、Trait objectとして使うことが想定されるTraitの場合、[object-safety](https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md)を満たすように設計しなければ、利用者がコンパイルエラーに遭遇してしまう
- object-safetyを満たさないTraitを満たすように修正する場合、破壊的変更を避けるのは難しいので、予めAPI設計時に考慮することが望ましい
- object-safetyを満たすためには、以下の２つのルールに従わなければならない
  - トレイとメソッドはジェネリックであってはならない
  - トレイとメソッドでは、レシーバ（メソッドの第一引数）以外に、`Self`を含む方を使ってはならない

- 参考：[Let's Get Rusty - Using Trait Objects in Rust](https://youtu.be/ReBmm0eJg6g?list=PLai5B987bZ9CoVR-QEIN9foz4QCJ0H2Y8&t=743)
- 参考：[object-safetyの定義]([object-safety](https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md))
- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１２
- 参考：[Rust API Guidelines - C-OBJECT](https://rust-lang.github.io/api-guidelines/flexibility.html#traits-are-object-safe-if-they-may-be-useful-as-a-trait-object-c-object)

## デフォルト実装を用意することで、実装しなければならないトレイトメソッドを最小限にする

- トレイトには2種類の利用者がいる
  - トレイトを実装するstructの開発者
  - トレイトに依存した処理を書く開発者
- 前者に向けてはトレイトメソッドを少なく、後者に向けてはトレイトメソッドを充実させることが望ましい
- 実装必須のトレイトメソッドを極少数、それらに依存するデフォルト実装つきのトレイトメソッドを充実させることで、両方の要件を満たすことができる
こ
- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１3

## SemVerを理解する

- 何かを変更したらパッチバージョンを変更する
- APIに機能を追加したらマイナーバージョンを更新する
- APIを削除、もしくは変更した場合にはメジャーバージョンを更新する

- 一番左の数字が0のときは、最初の0でない数字をメジャーバージョン相当と考える
  - 例えば、`0.2.3`から`0.3.0`や、`0.0.1`から`0.0.2`は互換性のないのない変更を含む可能性があると考える

- 参考：参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２１

## `cargo-semver-check`を使って、バージョン更新のアセスメントを行う

## 略語は１語として扱う

- CamelCaseにおいて、`UUID`ではなく、`Uuid`とする

- 参考：[Rust API Guidelines - C-CASE](https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case)

## クレート名に`-rs`, `-rust`をつけない

- 参考：[Rust API Guidelines - C-CASE](https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case)

## ビルダーパターンのAPIを提供するときは、validateを追加する可能性をふまえて、`Result`型を返す

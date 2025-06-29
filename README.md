# Rust Best Practices

- [本リポジトリの目的](#本リポジトリの目的)
- [参考文献](#参考文献)
- [API設計](#api設計)
  - [APIを定義する場合は、Rust API Guidelinesに従う](#apiを定義する場合はrust-api-guidelinesに従う)
  - [関数の引数では、所有型の借用より、借用型を優先する (`&str` \> `&String`, `&[T]` \> `&Vec<T>`, `&T` \> `&Box<T>`)](#関数の引数では所有型の借用より借用型を優先する-str--string-t--vect-t--boxt)
- [標準ライブラリ内のトレイト](#標準ライブラリ内のトレイト)
  - [自分が実装するクロージャは`Fn` \> `FnMut` \> `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` \> `FnMut` \> `Fn`の順に優先する](#自分が実装するクロージャはfn--fnmut--fnonceの順に優先しトレイト境界への指定はfnonce--fnmut--fnの順に優先する)
- [エラーハンドリング](#エラーハンドリング)
  - [`Option`と`Result`は`match`を用いずに変換する](#optionとresultはmatchを用いずに変換する)
  - [ライブラリでは、`thiserror`クレートを使って具体的で詳細なエラー情報を呼び出し側に伝える](#ライブラリではthiserrorクレートを使って具体的で詳細なエラー情報を呼び出し側に伝える)
  - [アプリケーションでは、`anyhow`クレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する](#アプリケーションではanyhowクレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する)
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

## API設計

### APIを定義する場合は、[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)に従う

- まずは一通り目を通して理解をする
- 実際の設計時には、各APIに対して[チェックリスト](https://rust-lang.github.io/api-guidelines/checklist.html)でチェックする

- 参考：[Rust for Rustaceans](https://rust-for-rustaceans.com/)のChapter 3 "Designing Interfaces"

### 関数の引数では、所有型の借用より、借用型を優先する (`&str` > `&String`, `&[T]` > `&Vec<T>`, `&T` > `&Box<T>`)

- `&str`であれば文字列リテラル(`&'static str`)やslice(`&str`)も受け取ることができるが、`&String`では受け取れない

- 参考：[Use borrowed types for arguments](https://rust-unofficial.github.io/patterns/idioms/coercion-arguments.html)

## 標準ライブラリ内のトレイト

### 自分が実装するクロージャは`Fn` > `FnMut` > `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` > `FnMut` > `Fn`の順に優先する

- 自分がクロージャを実装する場合、呼び出し元の制約が少ない順に優先する
- トレイト境界を指定する場合、受け入れられるクロージャを広くするために、制約が多い順に優先する

- `Fn` : 共有参照しかキャプチャしていないので、何回でも呼出でき、再帰呼出しもできる
- `FnMut` : 排他参照をキャプチャしているので、何回でも呼出できるが、再帰呼出しはできない
- `FnOnce` : 所有権をmoveするので、１回しか呼び出せない（⇒再帰呼出しもできない）

- 参考：[Comprehensive Rust - Closures](https://google.github.io/comprehensive-rust/ja/std-traits/closures.html)

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

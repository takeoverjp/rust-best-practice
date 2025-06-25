# Rust Best Practices

- [本リポジトリの目的](#本リポジトリの目的)
- [参考文献](#参考文献)
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

## 参考文献

- [Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)
- [Rust for Rustaceans](https://rust-for-rustaceans.com/)
- [Comprehensive Rust](https://google.github.io/comprehensive-rust/)

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

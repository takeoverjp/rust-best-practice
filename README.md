# Rust Best Practices

- [本リポジトリの目的](#本リポジトリの目的)
- [参考文献](#参考文献)
- [エラーハンドリング](#エラーハンドリング)
  - [`Option`と`Result`は`match`を用いずに変換する](#optionとresultはmatchを用いずに変換する)
  - [ライブラリでは、`thiserror`クレートを使って具体的で詳細なエラー情報を呼び出し側に伝える](#ライブラリではthiserrorクレートを使って具体的で詳細なエラー情報を呼び出し側に伝える)
  - [アプリケーションでは、`anyhow`クレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する](#アプリケーションではanyhowクレートを使ってすべての依存ライブラリのエラーを一貫した方法で処理する)


## 本リポジトリの目的

本リポジトリには、Rust開発におけるベストプラクティスを集約する。  

- 記載ルール
  - 関連する資料を可能な限り参照する

## 参考文献

- [Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)
- [Rust for Rustaceans](https://rust-for-rustaceans.com/)

## エラーハンドリング

### `Option`と`Result`は`match`を用いずに変換する

- `match`を使わない値の取り出し方
  - エラーを無視する場合 : `if let`
  - エラー時に`panic!`する場合 : `unwrap()`, `expect()`
  - エラー処理を上位に委ねる場合 : `?`演算子

- `Option`と`Result`の変換
  - 以下のダイアグラムを参照
  - `as_ref()`を使うことで、`Option`への参照を参照への`Option`に変換できる

![OptionとResultの変換](https://docs.google.com/drawings/d/1EOPs0YTONo_FygWbuJGPfikO9Myt5HwtiFUHRuE1JVM/preview)

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

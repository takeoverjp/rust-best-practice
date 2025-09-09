# Rust Best Practices

- [本リポジトリの目的](#本リポジトリの目的)
- [参考文献](#参考文献)
- [API設計](#api設計)
- [標準ライブラリ内のトレイト](#標準ライブラリ内のトレイト)
- [型](#型)
- [エラーハンドリング](#エラーハンドリング)
- [実装](#実装)
- [モジュール](#モジュール)
- [ログ](#ログ)
- [依存ライブラリ](#依存ライブラリ)
- [テスト](#テスト)
- [コメント](#コメント)
- [並列実行](#並列実行)
- [async](#async)
- [マクロ](#マクロ)
- [unsafe](#unsafe)
- [低レイヤー](#低レイヤー)
- [Rustバージョン](#rustバージョン)


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
- [Safety-Critical Rust Coding Guidelines](https://coding-guidelines.arewesafetycriticalyet.org/)
- [Edition Guide](https://doc.rust-jp.rs/edition-guide/)

- 要チェックセッション
  - RustConf 2025
    - [Cancelling Async Rust](https://rustconf.com/schedule-tag/rust-internals/)
    - [Fine-Grained C++ Interop](https://rustconf.com/schedule-tag/interop/)
    - [Auto-Instrumenting Rust Applications Using eBPF and OpenTelemetry](https://rustconf.com/schedule-tag/ebpf/)

## [API設計](api_design.md)
## [標準ライブラリ内のトレイト](standard_trait.md)
## [型](type.md)
## [エラーハンドリング](error_handling.md)
## [実装](impl.md)
## [モジュール](module.md)
## [ログ](log.md)
## [依存ライブラリ](dependency.md)
## [テスト](test.md)
## [コメント](comment.md)
## [並列実行](parallelism.md)
## [async](async.md)
## [マクロ](macro.md)
## [unsafe](unsafe.md)
## [低レイヤー](low_layer.md)
## [Rustバージョン](rust_version.md)

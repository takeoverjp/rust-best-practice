# [依存ライブラリ](dependency.md)

- [ワイルドカードインポートはしない](#ワイルドカードインポートはしない)
- [`cargo udeps`で不必要な依存クレートをチェックする](#cargo-udepsで不必要な依存クレートをチェックする)
- [`cargo deny`で依存クレートをチェックする](#cargo-denyで依存クレートをチェックする)
- [`cargo install`するときは、`--locked`オプションをつける](#cargo-installするときは--lockedオプションをつける)
- [`cargo build`するときに、依存クレートを更新する必要のないときは、`--locked`オプションをつける](#cargo-buildするときに依存クレートを更新する必要のないときは--lockedオプションをつける)
- [後方互換性を満たすための必要条件を理解し、適切にセマンティックバージョンを管理する](#後方互換性を満たすための必要条件を理解し適切にセマンティックバージョンを管理する)
- [普段の開発ではレジストリのpackageではなく、ローカルのpackageを参照する場合、`Cargo.toml`の`[patch]`セクションを使う](#普段の開発ではレジストリのpackageではなくローカルのpackageを参照する場合cargotomlのpatchセクションを使う)

## ワイルドカードインポートはしない

- ワイルドカードインポートは、依存クレートの更新に伴い、シンボルの衝突などにつながるリスクがあるため、避けるべき

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２３

## `cargo udeps`で不必要な依存クレートをチェックする

- `cargo udeps`により、リファクタリング等で不要となった依存に気づくことができる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２５

## `cargo deny`で依存クレートをチェックする

- `cargo deny`で依存クレートに対して下記のようなチェックができる
  - 使用されているバージョンに既知のセキュリティ問題がある依存ライブラリ
  - 受入不能なライセンスで提供されている依存ライブラリ
  - ただ受け入れられない依存ライブラリ
  - 依存グラフ中で、複数の異なるバージョンが使われている依存ライブラリ

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目２５

## `cargo install`するときは、`--locked`オプションをつける

- `cargo install`は、デフォルトで`Cargo.lock`を参照せずに毎回`Cargo.toml`で指定された範囲で最新のバージョンの依存ライブラリを使ってビルドするため、再現性がなくなってしまう
- `--locked`オプションをつけることで、`Cargo.lock`に記載されているバージョンの依存ライブラリを使ってビルドすることができ、再現性を確保できる

- 参考：[cargo-install(1) - Dealing with the Lockfile](https://doc.rust-lang.org/cargo/commands/cargo-install.html#dealing-with-the-lockfile)

## `cargo build`するときに、依存クレートを更新する必要のないときは、`--locked`オプションをつける

- `cargo build`は、デフォルトで`Cargo.lock`を参照せずに毎回`Cargo.toml`で指定された範囲で最新のバージョンの依存ライブラリを使ってビルドするため、再現性がなくなってしまう
- `--locked`オプションをつけることで、`Cargo.lock`に記載されているバージョンの依存ライブラリを使ってビルドすることができ、再現性を確保できる
- 逆に、依存クレートの更新だけをするときには、`cargo update`を使う

- 参考：[cargo-build(1) - Manifest Options](https://doc.rust-lang.org/cargo/commands/cargo-build.html#manifest-options)

## 後方互換性を満たすための必要条件を理解し、適切にセマンティックバージョンを管理する

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目21

## 普段の開発ではレジストリのpackageではなく、ローカルのpackageを参照する場合、`Cargo.toml`の`[patch]`セクションを使う

- 以下のように記載することで、普段の開発(patchが適用される)ではローカルのpackageを参照し、リリース時(patchが適用されない)にはレジストリのpackageを参照するようにできる

```toml
[patch.crates-io]
your-library-package = { path = "../your-library-package" }
```

- `patch`というキーの後ろには、`crates-io`のようなレジストリの名前や、URLを指定することができる

- 参考：[The [patch] section](https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html#the-patch-section)

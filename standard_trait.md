# 標準ライブラリ内のトレイト

- [機密情報が含まれない型であれば、`Debug`トレイトを実装する](#機密情報が含まれない型であればdebugトレイトを実装する)
- [解放しなければならない何らかの資源を保持する型には`Drop`トレイトを実装し、RAIIにする](#解放しなければならない何らかの資源を保持する型にはdropトレイトを実装しraiiにする)
- [自分が実装するクロージャは`Fn` \> `FnMut` \> `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` \> `FnMut` \> `Fn`の順に優先する](#自分が実装するクロージャはfn--fnmut--fnonceの順に優先しトレイト境界への指定はfnonce--fnmut--fnの順に優先する)

## 機密情報が含まれない型であれば、`Debug`トレイトを実装する

- 基本はderiveマクロによる自動導出をする
- 自動導出だと煩雑すぎる場合は、独自で実装する

- 検出方法：[`rustc` lintの`missing_debug_implementations`](https://doc.rust-lang.org/stable/rustc/lints/listing/allowed-by-default.html#missing-debug-implementations)

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１０

## 解放しなければならない何らかの資源を保持する型には`Drop`トレイトを実装し、RAIIにする

- `Drop`トレイトを実装することで、型のインスタンスがスコープを抜けるときに、自動的に資源を解放することができる

- 参考：[Effective Rust](https://www.oreilly.co.jp/books/9784814400942/)の項目１２

## 自分が実装するクロージャは`Fn` > `FnMut` > `FnOnce`の順に優先し、トレイト境界への指定は、`FnOnce` > `FnMut` > `Fn`の順に優先する

- 自分がクロージャを実装する場合、呼び出し元の制約が少ない順に優先する
- トレイト境界を指定する場合、受け入れられるクロージャを広くするために、制約が多い順に優先する

- `Fn` : 共有参照しかキャプチャしていないので、何回でも呼出でき、再帰呼出しもできる
- `FnMut` : 排他参照をキャプチャしているので、何回でも呼出できるが、再帰呼出しはできない
- `FnOnce` : 所有権をmoveするので、１回しか呼び出せない（⇒再帰呼出しもできない）

- 参考：[Comprehensive Rust - Closures](https://google.github.io/comprehensive-rust/ja/std-traits/closures.html)


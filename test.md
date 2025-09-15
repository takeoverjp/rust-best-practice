# [テスト](test.md)

- [private関数のテストはソースコードに記載する](#private関数のテストはソースコードに記載する)
- [public関数のテストはtestsディレクトリに集約する](#public関数のテストはtestsディレクトリに集約する)
- [cargo mutantsの導入を検討する](#cargo-mutantsの導入を検討する)
- [コンパイルに失敗することを確認するテストは、docテストの`compile_fail`属性を使う](#コンパイルに失敗することを確認するテストはdocテストのcompile_fail属性を使う)
- [tokio::timeを使うコードのテストでは、`tokio::time::pause`と`tokio::time::advance`を使って時間の進行を制御する](#tokiotimeを使うコードのテストではtokiotimepauseとtokiotimeadvanceを使って時間の進行を制御する)

## private関数のテストはソースコードに記載する

## public関数のテストはtestsディレクトリに集約する

## cargo mutantsの導入を検討する

## コンパイルに失敗することを確認するテストは、docテストの`compile_fail`属性を使う

```
/// ```compile_fail
/// let x = 1 + "a";
/// ```
#[allow(dead_code)]
struct CompileFailTest;
```

- 参考：[Crust of Rust - Declarative Macros](https://youtu.be/q6paRBbLgNw?list=PLqbS7AVVErFiWDOAVrPt7aYmnuuOLYvOa&t=3394)

## tokio::timeを使うコードのテストでは、`tokio::time::pause`と`tokio::time::advance`を使って時間の進行を制御する

- 参考：https://docs.rs/tokio/latest/tokio/time/fn.pause.html
- 参考：https://docs.rs/tokio/latest/tokio/time/fn.advance.html

## 複数スレッドの同時実行をサポートしていないテストは、cargo-nextestの使用を検討する

- 参考:https://nexte.st/docs/design/why-process-per-test/

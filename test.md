# [テスト](test.md)

- [private関数のテストはソースコードに記載する](#private関数のテストはソースコードに記載する)
- [public関数のテストはtestsディレクトリに集約する](#public関数のテストはtestsディレクトリに集約する)
- [cargo mutantsの導入を検討する](#cargo-mutantsの導入を検討する)
- [コンパイルに失敗することを確認するテストは、docテストの`compile_fail`属性を使う](#コンパイルに失敗することを確認するテストはdocテストのcompile_fail属性を使う)

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


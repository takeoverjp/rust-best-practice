# コメント

- [APIに対するドキュメンテーションコメントは、rust-lang/rfcs#1574に従う](#apiに対するドキュメンテーションコメントはrust-langrfcs1574に従う)
- [ドキュメンテーションコメントに記載するサンプルコードにおいて、利用者の理解の助けにならない部分は、`/// #`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて利用者の理解の助けにならない部分は-を使う)
- [ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっていない場合は`/// ```no_run`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて実行可能なテストになっていない場合は-no_runを使う)
- [ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっている場合は`/// ```ignore`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて実行可能なテストになっている場合は-ignoreを使う)

## APIに対するドキュメンテーションコメントは、[rust-lang/rfcs#1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md)に従う

- 参考：[rust-lang/rfcs#1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md)

## ドキュメンテーションコメントに記載するサンプルコードにおいて、利用者の理解の助けにならない部分は、`/// #`を使う

- 参考：[The rustdoc book - Hiding portions of the example](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html#hiding-portions-of-the-example)

## ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっていない場合は`/// ```no_run`を使う

- 参考：[The rustdoc book - Documentation tests](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html#attributes)
## ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっている場合は`/// ```ignore`を使う

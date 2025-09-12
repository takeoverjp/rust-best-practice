# コメント

- [APIに対するドキュメンテーションコメントは、rust-lang/rfcs#1574に従う](#apiに対するドキュメンテーションコメントはrust-langrfcs1574に従う)
- [ドキュメンテーションコメントは、可能な限りリンクにする](#ドキュメンテーションコメントは可能な限りリンクにする)
- [ドキュメンテーションコメントに記載するサンプルコードにおいて、利用者の理解の助けにならない部分は、`/// #`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて利用者の理解の助けにならない部分は-を使う)
- [ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっていない場合は`/// ```no_run`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて実行可能なテストになっていない場合は-no_runを使う)
- [ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっている場合は`/// ```ignore`を使う](#ドキュメンテーションコメントに記載するサンプルコードにおいて実行可能なテストになっている場合は-ignoreを使う)
- [feature flagsを用意する場合は、クレートのトップレベルのドキュメントに`# Feature flags`セクションを設ける](#feature-flagsを用意する場合はクレートのトップレベルのドキュメントに-feature-flagsセクションを設ける)

## APIに対するドキュメンテーションコメントは、[rust-lang/rfcs#1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md)に従う

- 参考：[rust-lang/rfcs#1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md)

## ドキュメンテーションコメントは、可能な限りリンクにする

## ドキュメンテーションコメントに記載するサンプルコードにおいて、利用者の理解の助けにならない部分は、`/// #`を使う

- 参考：[The rustdoc book - Hiding portions of the example](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html#hiding-portions-of-the-example)

## ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっていない場合は`/// ```no_run`を使う

- 参考：[The rustdoc book - Documentation tests](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html#attributes)
## ドキュメンテーションコメントに記載するサンプルコードにおいて、実行可能なテストになっている場合は`/// ```ignore`を使う

## feature flagsを用意する場合は、クレートのトップレベルのドキュメントに`# Feature flags`セクションを設ける

- 参考：[tokio - Feature flags](https://github.com/tokio-rs/tokio/blob/925c614c89d0a26777a334612e2ed6ad0e7935c3/tokio/src/lib.rs#L305-L342)

## 複雑な初期化が必要で、`no_run`を指定するドキュメントコードは、呼ばれない関数を定義する形でかく

- 参考:https://rust-unofficial.github.io/patterns/idioms/rustdoc-init.html

## 利用者が意識しなくていい関数は、`doc(hidden)`で見せないようにする

- 参考:https://rust-lang.github.io/api-guidelines/documentation.html#rustdoc-does-not-show-unhelpful-implementation-details-c-hidden

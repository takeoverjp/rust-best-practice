# FFI

## FFIで多言語から受けとる文字列は、以下の原則にしたがう

- 受け取った文字列はコピーするのではなく借用のまま扱う
- cの文字列とRustの文字列を変換するための複雑性と `unsafe` ブロックは最小にする
- 参考:https://rust-unofficial.github.io/patterns/idioms/ffi/accepting-strings.html

## エラーコードは以下のようにマッピングする

- https://rust-unofficial.github.io/patterns/idioms/ffi/errors.html

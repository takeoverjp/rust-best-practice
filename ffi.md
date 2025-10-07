# FFI

## FFIで他言語から受けとる文字列は、以下の原則にしたがう

- 受け取った文字列はコピーするのではなく借用のまま扱う
- cの文字列とRustの文字列を変換するための複雑性と `unsafe` ブロックは最小にする
- 参考:https://rust-unofficial.github.io/patterns/idioms/ffi/accepting-strings.html

## FFIで他言語に渡す文字列は、以下の原則にしたがう

- 所有する文字列のライフタイムは可能な限り長くする
- 変換するための`unsafe` ブロックは最小にする
- cコードが文字列を変更する可能性がある場合、`Vec`を使う
- API要件でない限り、所有権は渡さない
- 参考:https://rust-unofficial.github.io/patterns/idioms/ffi/passing-strings.html

## エラーコードは以下のようにマッピングする

- https://rust-unofficial.github.io/patterns/idioms/ffi/errors.html

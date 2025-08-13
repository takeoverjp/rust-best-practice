# [unsafe](unsafe.md)

- [unsafeコードは書かない](#unsafeコードは書かない)
- [unsafeコードを書かざるを得ない場合場合、ラッパレイヤに集約する](#unsafeコードを書かざるを得ない場合場合ラッパレイヤに集約する)
- [unsafe関数の中であっても、明示的にunsafeブロックを使う](#unsafe関数の中であっても明示的にunsafeブロックを使う)
- [unsafeコードが依存する前提条件と普遍条件を明記する](#unsafeコードが依存する前提条件と普遍条件を明記する)
- [unsafeコードに対してMiriを実行する](#unsafeコードに対してmiriを実行する)
- [unsafeコードに対してマルチスレッドで使用される場合を慎重に考える](#unsafeコードに対してマルチスレッドで使用される場合を慎重に考える)
- [intをポインタに変換しない](#intをポインタに変換しない)

## unsafeコードは書かない

## unsafeコードを書かざるを得ない場合場合、ラッパレイヤに集約する

## unsafe関数の中であっても、明示的にunsafeブロックを使う

## unsafeコードが依存する前提条件と普遍条件を明記する

## unsafeコードに対してMiriを実行する

## unsafeコードに対してマルチスレッドで使用される場合を慎重に考える

## intをポインタに変換しない

- 参考：[An integer shall not be converted to a pointer](https://coding-guidelines.arewesafetycriticalyet.org/coding-guidelines/expressions.html#gui_PM8Vpf7lZ51U)

# module

## 関数名はmoduleパスを省略せず書く、構造体・enum・トレイト名は省略して書く

- 理由
  - 関数名はmoduleパスを省略せず書くことで、どのmoduleの関数かが明確になるため
  - 構造体・enum・トレイト名はmoduleパスを省略して書くことで、コードが読みやすくなるため

```rust
// NG
use ::std::io::write;

write(&mut buffer, b"Hello, world!");

// OK

::std::io::write(&mut buffer, b"Hello, world!");

```


## use宣言は、まとめない

- 理由
  - 検索しやすくなるため
  - 差分が明確になるため

```rust
// NG
use ::std::io::{Read, Write};

// OK
use ::std::io::Read;
use ::std::io::Write;
```

## moduleパスは、`::`もしくは`crate::`で始める

- 理由
  - モジュールを一意に特定できるようにするため

```rust
// NG
use std::io::{self, Read, Write};

// OK
use ::std::io::{self, Read, Write};
```

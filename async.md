# async

- [ログにはtracing](#ログにはtracing)
- [backtrace](#backtrace)
- [taskには名前をつける](#taskには名前をつける)
- [tokio-consoleが簡単に使えるようにする](#tokio-consoleが簡単に使えるようにする)
- [ライブラリはtokioに依存しないようにする](#ライブラリはtokioに依存しないようにする)
- [lockを取りながらawaitするFutureは返さない](#lockを取りながらawaitするfutureは返さない)
- [Pinで受け取った構造体の中のフィールドをPinで参照したい場合は、`pin_project!`を使う](#pinで受け取った構造体の中のフィールドをpinで参照したい場合はpin_projectを使う)
- [`tokio::select!`のbranchに書く`<async expression>`は、cancel safeであることを確認する](#tokioselectのbranchに書くasync-expressionはcancel-safeであることを確認する)
- [`tokio::select!`のbranchで`, if <precondition>`を使う場合、非同期処理自体は評価されることを理解しておく](#tokioselectのbranchで-if-preconditionを使う場合非同期処理自体は評価されることを理解しておく)

## ログにはtracing

## backtrace

- 参考: [debug tool](https://gihyo.jp/article/2023/02/tfen007-rust-debug-tool)

## taskには名前をつける

- 参考: https://docs.rs/tokio/latest/tokio/task/struct.Builder.html#method.name

## tokio-consoleが簡単に使えるようにする

## ライブラリはtokioに依存しないようにする

- rustは標準でasyncをサポートしており、それだけで機能実現することで、ライブラリ利用者がtokio以外のランタイムを選択する余地を残しておく

## lockを取りながらawaitするFutureは返さない

-  await中に他のタスクが同じロックを取得しようとしたら、デッドロックしてしまうため

```
// NG

async fn lock_and_await() {
    let resource = resource.lock().unwrap();
    loop {
        if resource.is_ready() {
            break;
        }

        yield_now().await; // ここでlockを保持したままawaitしている
    }
}

// OK
async fn await_and_lock() {
    loop {
        // 即座にlockを開放する
        if resource.lock().unwrap().is_ready() {
            break;
        }

        yield_now().await; // ここでlockを保持していないので、awaitしても問題ない
    }
}
```

- 参考: [await_holding_lock - clippy](https://rust-lang.github.io/rust-clippy/master/index.html#await_holding_lock)

## Pinで受け取った構造体の中のフィールドをPinで参照したい場合は、`pin_project!`を使う

- 参考：[Projections and Structural Pinning](https://doc.rust-lang.org/std/pin/index.html#projections-and-structural-pinning)
- 参考：[pin_project_lite](https://docs.rs/pin-project-lite/latest/pin_project_lite/);

## `tokio::select!`のbranchに書く`<async expression>`は、cancel safeであることを確認する

- 参考：https://docs.rs/tokio/1.47.1/tokio/macro.select.html#cancellation-safety

## `tokio::select!`のbranchで`, if <precondition>`を使う場合、非同期処理自体は評価されることを理解しておく

> 2. Aggregate the <async expression>s from each branch, including the disabled ones. If the branch is disabled, <async expression> is still evaluated, but the resulting future is not polled.

- 参考：https://docs.rs/tokio/1.47.1/tokio/macro.select.html

# async

- [ログにはtracing](#ログにはtracing)
- [backtrace](#backtrace)
- [lockを取りながらawaitするFutureは返さない](#lockを取りながらawaitするfutureは返さない)

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

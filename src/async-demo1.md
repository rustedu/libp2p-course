# Demo 1
## `Future` 、`async` 、`await`

```rust
// main.c

use futures::executor::block_on;
// use async_std::task::{sleep};
// use std::time::Duration;

struct Buff {
    data: Vec<u8>,
}

async fn rx_socket() -> Buff {
    // sleep(Duration::from_secs(2)).await;
    Buff {
        data: vec![1, 2, 3, 4],
    }
}

async fn rx_log(buff: Buff) {
    println!("socket 接收：{:?}", buff.data);
}

async fn socket_log() {
    let buff = rx_socket().await;
    rx_log(buff).await;
}

async fn tx_serial() {
    let buf = Buff {
        data: vec![11, 22, 33, 44],
    };
    println!("serial 发送：{:?}", buf.data);
}

async fn async_main() {
    let f1 = socket_log();
    let f2 = tx_serial();

    futures::join!(f1, f2);
}

fn main() {
    block_on(async_main());
}

```

## Future
可以理解为 **将来的结果**，在不确定的某个时刻会产生一个期望的结果。每一个 async 都会返回 `Future`，原型为：
```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

核心方法 `poll` 可以获取期望结果。如果当前没有结果，也不会阻塞在这里，而是通过后续的轮询，当有结果了，再唤醒当前任务。传递给 `poll` 方法的 `context` 可以提供 `Waker`，它是唤醒当前任务的句柄。

当使用 future 时，通常不会直接调用 poll，而是 .await 该值。

## Waker
可以理解为一个信号量，实际是一个特定的唤醒行为，当 `future` 有结果了，通知 `poll` 唤醒当前任务。原型为：
```rust
#[repr(transparent)]
pub struct Waker { /* fields omitted */ }
```

`Waker` 是通过通知执行者准备运行来唤醒任务的句柄。

该句柄封装了 `RawWaker` 实例，该实例定义了特定于执行者的唤醒行为。

实现 `Clone`，`Send` 和 `Sync`。

## Executor
Rust 的 `Future` 是惰性的，创建后并不会自动执行，而是需要显式的驱动它执行。`async` 函数内可以使用 `.await` 来驱动其中的 `Future`，而最外面的 `async` 函数，就需要一个执行器来驱动。

在这里 `futures::executor::block_on` 就是执行器。

执行器会管理一批 `Future` (最外层的 `async` 函数)，然后通过不停地 `poll` 推动它们直到完成。最开始，执行器会先 `poll` 一次 Future ，后面就不会主动去 `poll` 了，而是等待 `Future` 通过调用 wake 函数来通知它可以继续，它才会继续去 `poll` 。这种wake 通知然后 `poll`的方式会不断重复，直到 `Future` 完成。

## join!
表示并发地处理多个 `Future`，若其中一个阻塞，则执行其它的，当全部阻塞时，当前函数会阻塞，让将线程的所有权让给执行器所在的函数（线程）。

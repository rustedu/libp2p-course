# Demo 2
## `join!` 、`select!`

### `join!`
Demo1 中提到要多个 `Future` 并发执行，可以使用 `join!`，感受一下这两段程序的区别：
```rust
async fn enjoy_book() {}
async fn enjoy_music() {}

async fn enjoy_book_then_music() -> (Book, Music) {
    let book = enjoy_book().await;
    let music = enjoy_music().await;
    (book, music)
}
```
```rust
async fn enjoy_book() {}
async fn enjoy_music() {}

async fn enjoy_book_and_music() -> (Book, Music) {
    let book = enjoy_book();
    let music = enjoy_music();
    join!(book, music)
}
```
前者使用 `.await` 驱动 `Future`， `Future`时惰性的，驱动时才会执行，且需要全程驱动，所以后续 `Future` 只有在前一个完成的情况才会的到机会执行。*这种写法适合多个目标任务需要顺序运行，但不能阻塞其它任务的场景。如根据当前请求结果，发起下一次请求。*

后者使用 `join!` 同时驱动多个 `Future`，本质上还是在同一个线程内，所以更准确的描述时分时驱动，即并行。当所有 `Future` 都完成后，`join!` 退出。*这种写法适合执行多个互不相关目标任务的场景。如一次发起多个请求以获取多个文件。*

以上都是基于多个 `Future` 都成功完成的假设。那如果其中一个发生错误了呢？
```rust
#use futures::{
#    future::TryFutureExt,
#    try_join,
#    executor::block_on
#};
#
#struct Book {
#    book: String,
#}
#
#struct Music {
#    music: String,
#}
#
async fn get_book() -> Result<Book, String> {
    Ok(Book{book: "book".into()})
}
async fn get_music()-> Result<Music, String> {
    // Ok(Music{music: "music".into()})
    Err("music err".to_string())
}

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book = get_book().map_err(|e| e);
    let music = get_music().map_err(|e| e);
    try_join!(book, music)
}

#fn main() {
#    match block_on(get_book_and_music()) {
#        Ok((b, m)) => println!("{} {}", b.book, m.music),
#        Err(e) => println!("{}", e),
#    }
#}
```
可以使用 `try_join!` 来在发生错误时终止该段程序（并获取错误）。

HTTP 客户端向服务器请求资源就涵盖了上述三种场景。

### `select!`
同样是同时执行多个 `Future`，感受一下下面的程序：
```rust
#use futures::{
#    future::FutureExt, // for `.fuse()`
#    pin_mut,
#    select,
#    executor::block_on
#};
#
async fn task_one() -> String {
    "task one completed first".into()
}

async fn task_two() -> String {
    "task two completed first".into()
}

async fn race_tasks() {
    let t1 = task_one().fuse();
    let t2 = task_two().fuse();

    pin_mut!(t1, t2);

    select! {
        msg = t1 => println!("{}", msg),
        msg = t2 => println!("{}", msg),
    }
}
#
#fn main() {
#    block_on(race_tasks());
#}
```

`select!` 在任何一个 `Future` 完成时就退出，这与 C 语言中的 `select()` 函数意思相近。*这种写法适合在同时执行多个目标任务，有完成的就立即处理的场景。*

通过 `loop{}` 可以达到 `join!` 相同的作用，完成所有 `Future`。

`fuse` 是保险丝的意思，可以理解为 **熔断**，即 `select!` 中已经完成的 `Future` 在后续的 `poll` 中都不会执行了。

`select!` 获取的是 `Future` 的可变引用，以保证未完成的 `Future` 在 `select!` 外仍有被驱动的能力，完成后仍可以得到响应。所以必须添加 `pin_mut!`。





# 占位符
在异步编程中曾介绍了 `Future`，其原型是：
```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
``` 
这里的 `Output` 就是一个类型占位符，在设计 `trait` 之时，没有明确规定类型，具体类型可以交给实现者定义。

```rust
trait Opts {
    type Output;
    fn pair(&self) -> Self::Output;
}

impl Opts for i32 {
    type Output = i32;
    fn pair(&self) -> Self::Output {
        self * 2
    }
}

impl Opts for String {
    type Output = String;
    fn pair(&self) -> Self::Output {
        format!("{}{}", self, self)
    }
}

fn main() {
    let a: i32 = 3;
    let msg: String = "msg".into();

    println!("pair of a: {}", a.pair());
    println!("pair of a: {}", msg.pair());
}
```

这种方式类似于泛型编程，个人理解就是占位符是一种强约束性的泛型，在内部逻辑依赖具体类型时，从书写和阅读等方面，占位符会有更优雅的体验。





# trait 定义
```rust
trait SomeOpts {
    fn pair(&self) -> i32;
    fn falf(&self) -> i32;
    fn zero(&self) -> i32 {
        0
    }
}

impl SomeOpts for i32 {
    fn pair(&self) -> i32 {
        self * 2
    }
    fn falf(&self) -> i32 {
        self / 2
    }
}

fn main() {
    let a: i32 = 3;
    println!("pair of a: {}", a.pair());
    println!("falf of a: {}", a.falf());
}
```

这里没有定义 `struct`，就是为了体现 `trait` 服务于类型，而不是简单的类（对象）。

`SomeOpts` 内定义了一系列的方法，其中由一个含有默认实现。Rust 规定为某个类型添加 `trait` 时，需要将其内的所有方法都实现，未实现的方法使用其默认方法。



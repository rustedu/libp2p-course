# 作为类型
Rust 中 `trait` 本身可以作为类型，用来描述变量的类型，这里的变量就包含的函数、方法的入口参数和返回值的类型。

```rust
trait Opts1 {
    fn pair(&self) -> i32;
    fn falf(&self) -> i32;
}

trait Opts2 {
    fn triple(&self) -> i32;
}

impl Opts1 for i32 {
    fn pair(&self) -> i32 {
        self * 2
    }

    fn falf(&self) -> i32 {
        self / 2
    }
}

impl Opts2 for i32 {
    fn triple(&self) -> i32 {
        self * 3
    }
}

fn do_pair(a: impl Opts1) -> impl Opts1{
    let a = a.pair();
    a
}

fn do_falf(a: impl Opts1) -> impl Opts1{
    let a = a.falf();
    a
}

fn do_triple(a: impl Opts1 + Opts2) -> i32 {
    let a = a.pair().falf();
    a.triple()
}

fn main() {
    let a: i32 = 3;

    do_pair(do_falf(a));
    println!("triple of a: {}", do_triple(a));
}
```

这里不仅体现了 `trait` 可以作为类型来描述入口参数和返回值，还体现了 Rust 一个强大的特性，类型可以用运算符的形式来更加精确的描述。

> 用实际行动来体现 **组合大于继承** 的设计原则


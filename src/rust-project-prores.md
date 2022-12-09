# 属性
Rust 程序中的任何特性项都可以用 `属性(attribute)` 来修饰。属性是 Rust 中写给编译器看的各种指令和建议的普适语法。

出现的形式是在文件的最开始文件，在变量、函数、方法、结构体等各种代码的上方：
```
#[...]
```

可以简单的理解就是**开启功能**。

## 常见属性举例
* #[allow]

    允许某些行为通过编译器的检查，如：

    `#[allow(non_camel_case_types)]`，允许非驼峰形式的命名方式；<br>
    `#allow[unused_xxx]`，允许未使用的某些东西；<br>
    `#allow[dead_code]`，允许未使用的代码；<br>

* #[cfg]

    添加条件编译的内容，例举一些最常用的部分：

| #[cfg(...)] 选项 | 何时启用 |
| --- | --- |
| test | 启用测试，方式可以是 cargo test 或 rustc --test |
| debug_assertions | 启用调试断言 |
| unix | 为 Unix 编译（包括 macOS） |
| windows | 为 Windows 编译 |
| target_pointer_width = "64" | 为 64 位平台编译，还可以是：32 |
| target_arch = "x86_64" | 为 x86_64 架构编译。<br> 其它值还有：x86、arm、aarch64 …… |
| target_os = "macos" | 为 macos 编译。<br> 其它值还有：windows、ios、android …… |
| feature = "robots" | 开启了 robots 特性，方式可以是cargo build --feature robots <br> 或 cargo build --cfg feature='"robots"' <br> 或在 Cargo.toml 的 [features] 下申明 |
| (A) not(A) | 成对出现，提供一个函数的两种实现，分别对应满足 A 和不满足 A 的情况 |
| all(A, B) | A 和 B 都满足时（&&） |
| any(A, B) | A 或 B 满足时（||） |

<br>

* #[inline]

    #[inline(always)]、#[inline(never)]

* #[feature(xxx)]
    
    开启某些特性。

* #[derive]
    使用 `derive` 属性的属性，会有编译器生成对应 trait 的默认实现代码。

    需要先写过程宏。
    
    标准库中的可派生 trait：`Debug`、`PartialEq`、`Eq`、`PartialOrd`、`Ord`、`Clone`、`Copy`、`Hash`、`Default`

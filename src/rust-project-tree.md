# Rust 工程结构
对于一个

常见的三种工程目录结构：
1. `cargo new` 原始工程
```
.
├── .gitignore
├── .git
│   └── ...
├── Cargo.toml
└── src
    └── lib.rs/main.rs
```

适用于简单工程，通常是一些临时工程，用来快速验证某种想法，如临时写个demo、tool。

常用命令：`cargo run`、`cargo build`

2. 单个工程的完整目录
```
.
├── .gitignore
├── .git
│   └── ...
├── Cargo.toml
├── src
│   ├── lib.rs/main.rs
│   └── bin
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable
│           ├── main.rs
│           └── some_module.rs
├── benches
│   ├── large-input.rs
│   └── multi-file-bench
│       ├── main.rs
│       └── bench_module.rs
├── examples
│   ├── simple.rs
│   └── multi-file-example
│       ├── main.rs
│       └── ex_module.rs
└── tests
    ├── some-integration-tests.rs
    └── multi-file-test
        ├── main.rs
        └── test_module.rs
```

这是一个典型的 crate 结构，一个完整的项目应具备上述所有内容，对性能不敏感的项目可能会没有 `benches`。

3. 工作空间
即 `workspace`，将多个工程组成一个工作空间
```
.
├── .gitignore
├── .git
│   └── ...
├── Cargo.toml
│── src
│   └── lib.rs/main.rs
├── proj1
│   ├── Cargo.toml
│   └── src
│       └── lib.rs/main.rs
├── proj2
│   ├── Cargo.toml
│   └── src
│       └── lib.rs/main.rs
└── proj3
    ├── Cargo.toml
    └── src
        └── lib.rs/main.rs
```

根目录下的 `Cargo.toml` 一定会有 `[workspace]` 选项，用来显式的描述该工作空间内的成员工程：
```toml
[workspace]
members = [
    "core",
    "misc/metrics",
    "misc/multistream-select",
    "misc/rw-stream-sink",
]
```

当遇到更大项目，大到其中的某个功能都可以拿出来单独作为一个 crate 进行开发、维护时，就可以使用这种结构。

libp2p 就是这样产生的，本身时 `ipfs` 的功能组件，后来被提出来的单独开发，rust 实现版本 `rust-libp2p` 就是这种目录结构。而因为 libp2p 又有功能值的单独开发，`rust-libp2p` 中就将这类功能作为 `member`,如 `swarm` 等。

还有一种情况适用与这种结构：组件工程不在一个目录中，用 workspace 的形式讲他们组合起来，可以指定绝对路径。


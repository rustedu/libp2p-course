# 测例
rust 的 cargo 系统集成了测试系统，使得开发者不必再去编写或者挑选测试工具，在不使用 IDE 的情况下，依旧能有一个很棒的测试体验。

## 基本命令格式
`cargo test` [*options*] [*testname*] [`--` *test-options*]

## 单元测试
是与源码写在一个文件中的测例。通常同于测试文件中的某些功能函数。用 `#[test]` 对测例函数进行注解。

```rust
#[test]
fn test_peer_id() {
    let local_key = identity::Keypair::generate_ed25519();
    let local_peer_id = PeerId::from(local_key.public());
    println!("Local peer id: {:?}", local_peer_id);
}
```

命令：
```bash
cargo test --example chat
```

## 集成测试
单独创建一个区域用于编写测例，这里测例通常都是功能丰富的，用于测试多个函数、模块是否可以正常合作，功能块的运行是否符合预期。

cargo 默认将集成测试目录至与根目录中：
```
.
├── Cargo.lock
├── Cargo.toml
├── examples
│   └── ...
├── README.md
├── src
│   └── ...
└── tests
    └── chat.rs
```

测例函数同样需要用 `#[test]` 对测例函数进行注解，且此时测试文件是一个完整的源码文件，依赖关系需要正确：
```rust
// ./tests/chat.rs
use libp2p::{
    floodsub::{self, Floodsub, FloodsubEvent},
    identity,
    mdns::{Mdns, MdnsConfig, MdnsEvent},
    swarm::SwarmEvent,
    Multiaddr, NetworkBehaviour, PeerId, Swarm,
};

///
/// ```
/// let local_key = identity::Keypair::generate_ed25519();
/// let local_peer_id = PeerId::from(local_key.public());
/// assert!(true)
/// ```
/// 
#[test]
fn new_peer_id() {
    let local_key = identity::Keypair::generate_ed25519();
    let local_peer_id = PeerId::from(local_key.public());
    assert!(true)
}
```

命令：
```bash
cargo test --test chat
```

## 文档测例
cargo 有一套功能完善的文档系统，完善到可以在代码中写注释、在注释中写文档、在文档中写测例。

> 注意：
> 
> 文档测例在库（`lib`）工程中有用。

```rust
///
/// ```
/// assert!(true);
/// ```
/// 
mod handler;
mod protocol;
// ...
```

命令：
```bash
cargo test --doc
```
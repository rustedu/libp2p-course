# Floodsub 代码导读

普遍的中文名称是：洪泛订阅。基于 发布-订阅 模式的一种通讯协议。物联网人熟知的 `MQTT` 也是这种模式。早期的报纸订阅、现在的各类 APP 中关注作者、账号、频道，等等，都是这种模式的真实写照。

这种模式可以有效隔离发送者和接收的联系，让各方只包含自己该处理的内容，以达到从设计上的代码解耦效果。

```
protocols/floodsub/
├── build.rs            // 主要用来编译 Protocol Buffers
├── Cargo.toml
├── CHANGELOG.md
└── src
    ├── layer.rs        // behaviour layer 行为（描述）层
    ├── lib.rs          // 对外接口描述
    ├── protocol.rs     // 协议的数据类型
    ├── rpc.proto       // Protocol Buffers 格式描述文件
    └── topic.rs        // 主题模块

1 directory, 8 files
```

## 其它文件浏览
`Floodsub` 基本逻辑就是**订阅消息**、**发送消息**与**传递消息**。每一个节点在收到消息后，都会对连接的全部节点进行发送消息，“一传十，十传百”，真的就是字面上的意思，所以会造成大量的重复发送，是一种以带宽换取传播速度的方式，适合小型网络。

* `topic.rs`：定义了主题的数据结构，为其添加 `From<Topic>` trait，以此实现主题的创建；
* `protocol.rs`: 协议传输相关内容，定义了协议和消息的数据结构，实现了协议的升级 trait，消息的解析、打包 trait；
* `rpc.proto`:消息内容的 protobuf 描述文件，会经过 `build.rs` 里定义的行为，在编译初期转换为 rust 代码；
* `lib.rs`: `Floodsub` 包对外提供的数据结构、方法。

## layer.rs

### `Floodsub` 提供的方法
* `pub fn new(local_peer_id: PeerId) -> Floodsub`

    使用默认配置创建一个 `Floodsub`

* `pub fn from_config(config: FloodsubConfig) -> Floodsub`

    使用指定配置创建一个 `Floodsub`

* `pub fn add_node_to_partial_view(&mut self, peer_id: PeerId)`

    将节点添加到消息传播节点链表中

* `pub fn remove_node_from_partial_view(&mut self, peer_id: &PeerId)`

    将节点从消息传播节点链表中移除

* `pub fn subscribe(&mut self, topic: Topic) -> bool` ✨

    订阅该主题
    如果订阅有效返回 `true`，如果已经订阅了就返回 `false`

* `pub fn unsubscribe(&mut self, topic: Topic) -> bool`

    取消订阅该主题 <br>
    **注意：** 只需要主题名称 <br>
    如果之前订阅了该主题就返回 `true`

* `pub fn publish( &mut self, topic: impl Into<Topic>, data: impl Into<Vec<u8, Global>>)` ✨

    只在订阅了该主题的情况下，将消息发布出去

* `pub fn publish_any( &mut self, topic: impl Into<Topic>, data: impl Into<Vec<u8, Global>>)`

    发布消息，即使没有订阅该主题

* `pub fn publish_many( &mut self, topic: impl IntoIterator<Item = impl Into<Topic>>, data: impl Into<Vec<u8, Global>>)`

    以多主题的方式发布消息 <br>
    **注意：** 如果没有订阅其中的任何主题，就不发布该消息

* `pub fn publish_many_any( &mut self, topic: impl IntoIterator<Item = impl Into<Topic>>, data: impl Into<Vec<u8, Global>>)`

    以多主题的方式发布消息，即使没有订阅其中的任何主题


### 为 `Floodsub` 实现 `NetworkBehaviour`
重点是在 `inject_event` 实现了事件处理和最后的消息的分发，即 `Flood`。





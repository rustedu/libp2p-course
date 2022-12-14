# 私聊
作业内容：<br>
修改 `examples/chat.rs` 代码，添加类似**私聊**的功能，即通过某种既定的方式，使得发送的消息只对指定的接收者显示。

私聊格式：
```
@name msg
```

## 方案一
从接收方思考，即除了指定格式的消息能显示外，其它格式不显示：
```rust
SwarmEvent::Behaviour(OutEvent::Floodsub(
    FloodsubEvent::Message(message)
)) => {
    if let Some((token, msg)) = String::from_utf8_lossy(&message.data).split_once(' ') {
        if token == "@name" {
            println!(
                "Received: '{:?}' from {:?}",
                msg,
                message.source
            );
        }
    }
    // println!(
    //     "Received: '{:?}' from {:?}",
    //     String::from_utf8_lossy(&message.data),
    //     message.source
    // );
}
```

## 方案二
从发送方思考，发送的时候除了消息内容外，其实还有主题内容，那么可以在主题内容上做文章，从而实现私聊功能。

1. 增加一个主题订阅：
```rust
// Create a Floodsub topic
let floodsub_topic = floodsub::Topic::new("chat");

behaviour.floodsub.subscribe(floodsub_topic.clone());
behaviour.floodsub.subscribe(floodsub::Topic::new("@name").clone());
```

2. 发送时修改主题：
```rust
let topic;
let mut msg = line.expect("Stdin not to close");
// '@' 开头 且 带有空格 ‘ ’
if msg.starts_with("@") && msg.contains(" ") {
    // 用 ‘ ’ 分割消息主题，提取主题
    let v: Vec<&str> = msg.split(' ').collect();
    println!("topic: {}", v[0]);
    // 创建主题
    topic = floodsub::Topic::new(v[0]);
    msg = v[1].to_string();
} else {
    println!("topic: {}", "chat");
    topic = floodsub_topic.clone();
}
```

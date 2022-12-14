# 自由讨论

## 超时问题

在运行 `chat` 例子的时候发现，当程序空闲（无数据交换）10秒左右，就再也无法发送、接收消息了。

查找发现是因为：

`chat` 例子使用了 `floodsub` 协议，而 `floodsub` 使用了 `OneShotHandler` 网络行为（`NetworkBehaviourAction`）

### 解决方案1
修改默认超时时间：
```rust
// swarm/src/handler/one_shot.rs
impl Default for OneShotHandlerConfig {
    fn default() -> Self {
        let timeout = 10;
        OneShotHandlerConfig {
            keep_alive_timeout: Duration::from_secs(timeout),
            outbound_substream_timeout: Duration::from_secs(timeout),
            max_dial_negotiated: 8,
        }
    }
}
```

### 解决方案1
去除超时事件抛出动作：
```rust
// protocols/mdns/src/behaviour.rs

// Emit expired event.
// let now = Instant::now();
// let mut closest_expiration = None;
// let mut expired = SmallVec::<[(PeerId, Multiaddr); 4]>::new();
// self.discovered_nodes.retain(|(peer, addr, expiration)| {
//     if *expiration <= now {
//         log::info!("expired: {} {}", peer, addr);
//         expired.push((*peer, addr.clone()));
//         return false;
//     }
//     closest_expiration = Some(closest_expiration.unwrap_or(*expiration).min(*expiration));
//     true
// });
// if !expired.is_empty() {
//     let event = MdnsEvent::Expired(ExpiredAddrsIter {
//         inner: expired.into_iter(),
//     });
//     return Poll::Ready(NetworkBehaviourAction::GenerateEvent(event));
// }
// if let Some(closest_expiration) = closest_expiration {
//     let mut timer = T::at(closest_expiration);
//     let _ = Pin::new(&mut timer).poll_next(cx);

//     self.closest_expiration = Some(timer);
// }
Poll::Pending

```






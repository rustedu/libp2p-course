# Relay 代码导读

## clap
默认参数

## 协议规范

## 与 floodsub 的对比
| floodsub | relay |
| --- | --- |
| layer.rs | client relay.rs relay |
| topic.rs protocol.rs | protocol.rs protocol |
| rpc.proto | message.proto |

## hop 协议
预留请求：accept()、deny()

回路请求：accept()、deny()

## stop 协议
回路建立：accept()、deny()

## 客户端
* handler.rs

Reserve, EstablishCircuit, 事件类型；

swarm 连接处理（监听、拨号……）

预留 Reservation

* transport.rs

需要中继的地址上的监听和拨号处理

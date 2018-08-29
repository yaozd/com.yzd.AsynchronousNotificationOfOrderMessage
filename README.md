# com.yzd.AsynchronousNotificationOfOrderMessage
抢购时,用异步队列处理下单,那怎么实时把下单结果通知用户呢？
## [抢购时,用异步队列处理下单,那怎么实时把下单结果通知用户呢？](https://segmentfault.com/q/1010000007163890/a-1020000007164448)
```
抢购时,用异步队列处理下单,那怎么实时把下单结果通知用户呢？
==
抢购最重要的是要保证库存数据的强一致性，抢购的瞬时流量非常大，如果使用MySql等一些关系型数据库可能会扛不住这方面的压力。一般会结合缓存中间件进行处理，例如redis。抢购开始前，将商品和库存数据同步到redis中，所有的抢购操作都在redis中进行处理，后台开启一个异步任务，定时的将库存数据刷到数据库中。
跟着开始对订单进行付款，由于流量较大，第三方支付系统本身也对调用端的应用限制流量，所以你这边所说的应该是我接下来需要描述的。
这里必然要使用消息队列（也就是你所说的异步队列），可以参考淘宝双11的限流措施，为了保护系统不受高流量的冲击而导致系统崩溃的问题，消息队列做了一层缓冲保护，系统需要设计一个窗口模型，窗口模型会实时的刷新用户办理手续的状态。
例如，用户下单之后准备去付款，这个时候会跳到办事大厅的服务窗口，如果此时窗口都满了，也就是消费者的数量达到上线了，那么需要用户开始排队，系统可以通过弹出等待窗口，让用户等待一下，一旦有空闲的线程释放出来，用户就可以开始支付下单。
上面的是以拍下减库存的模型进行说明，如果你们设计的系统是付款减库存，稍微会有些出入，但是同样也需要这样的窗口需要告知用户状态，及时用户付款成功，虽然没及时把状态返回给用户，用户能够通过一个页面及时查看到他的窗口状态就可以了。
==
client端用js轮询一个接口，用来获取处理状态
```
### client端用js轮询一个接口，用来获取处理状态-实现思路
```
client端用js轮询一个接口,产生的问题，轮询会放大请求。会影响到Redis或mysql等服务的连接池太多占用问题。
1.
构建高性能的web服务，Vert.x+spring boot--->客户端--->轮训请求
轮训请求的数据结构=orderId+时间戳的加密数据。订单的处理时间最太只有1个小时，不可能有昨天出的单子今天再来轮训查找，避免恶意请求。
参考：
Spring Boot同步架构与Vert.x异步架构高并发性能对比
https://blog.csdn.net/u013615903/article/details/79599446
2.
Hazelcast--->分布式内存数据网格进行数据存取
读取内存是为了解决【连接池太多占用问题】
3.请求读取Hazelcast内存中的map
4.消息队列把支付成功数据写入到Hazelcast内存中的map中
5.解决消息队列连接池满的问题，如果不在意连接池满的问题可以使用Redis来代替Hazelcast
```
### client端用js轮询一个接口，用来获取处理状态-使用技术
```
1.
hazelcast
hazelcast 分布式内存数据网格，提供多种数据结构的分布式实现
https://www.chkui.com/category
2.
vertx 
vertx 基于netty实现的Java非阻塞事件驱动框架
https://www.chkui.com/category

```
### client端用js轮询一个接口，用来获取处理状态-参考内容
```
微信->收藏->分布式内存数据网格与vertx非阻塞事件驱动框架
```
### client端用js轮询一个接口，用来获取处理状态-参考代码
1.[chkui/hazelcast-demo](https://github.com/yaozd/hazelcast-demo)
```
https://github.com/chkui/hazelcast-demo
https://github.com/yaozd/hazelcast-demo(备份)
```
2.[ vertx-springboot-demo](https://gitee.com/yaozd/vertx-springboot-demo)
```
Spring Boot同步架构与Vert.x异步架构高并发性能对比
https://blog.csdn.net/u013615903/article/details/79599446
springboot和vert.x整合 并使用vert.x服务代理和JPA
https://gitee.com/jaster/vertx-springboot-demo
https://gitee.com/yaozd/vertx-springboot-demo(备份)
```
3.[Hazelcast 分布式Map数据结构](https://www.chkui.com/article/hazelcast/hazelcast_distributed_map_structure)
```

```

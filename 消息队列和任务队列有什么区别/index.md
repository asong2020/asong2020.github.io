# 消息队列和任务队列有什么区别？


## 前言

> 昨天发了一篇文章是关于`machinery`的入门教程，有一位读者在留言中问我 这个和kafka有什么区别？一时我也有点懵，这两个的概念很近，到底有什么不同呢？根据我自己的理解，简单分析了一下，有不足之处欢迎指出。



## 消息队列

消息队列这个概念其实在我之前的文章：[手把手教姐姐写消息队列](https://mp.weixin.qq.com/s/0MykGst1e2pgnXXUjojvhQ)，自己动手用go写一个简易版的消息队列，有兴趣的小伙伴们可以看一下这篇文章。回归正题，我们再来介绍一下什么是消息队列。

消息队列，一般我们会简称它为MQ(Message Queue)。他是由两个单词组成，我们应该对队列(Queue)很熟悉吧。队列是一种先进先出的数据结构。再配合上消息，消息队列可以简单理解为：把要传输的数据放在队列中。使用较多的消息队列有ActiveMQ，RabbitMQ，ZeroMQ，Kafka，MetaMQ，RocketMQ。这里我们就不具体讲解消息队列实现细节，这不是本文的主题，只知道概念就可以了。了解了什么是消息队列，我们一起来看看他在什么场景使用。

![](https://song-oss.oss-cn-beijing.aliyuncs.com/golang_dream/article/static/queue.png)



### 场景

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。这里举一个消息队列的使用场景：日志处理。

日志处理是指将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题。架构简化如下：

![](https://song-oss.oss-cn-beijing.aliyuncs.com/golang_dream/article/static/log.png)

- 日志采集客户端，负责日志数据采集，定时写入`Kfaka`队列。
- `Kfaka`消息队列，负责日志数据的接收，存储和转发。
- 日志处理应用，订阅并消费kafka队列中日志数据。



## 任务队列

既然消息队列称为MQ，那么任务队列我们就可以叫其TQ(Task Message)。任务队列可以简单理解为：把要执行的任务放在队列中。使用较多的任务队列有`machiney`、`Celery`、`goWorker`、`YTask`。每一个任务队列都有自己的特点，这里就不细讲了。我写了一篇[`machinery`入门教程](https://mp.weixin.qq.com/s/4QG69Qh1q7_i0lJdxKXWyg)，并且翻译了[一篇`machinery`中文文档](https://mp.weixin.qq.com/s/lFhhytWbJE5g7R8KcA7Tfw)，有需要的公众号自取。具体任务队列的细节就不讲了。这不是本文的主题，下面我们看一看任务队列的使用场景。



### 场景

任务队列是用来执行一个耗时任务。大家可能都使用过马爸爸的花呗，每当我们还款时，就会增加自己的芝麻信用分。这就可以用到任务队列来计算用户的积分和等级了。架构简化如下：

![](https://song-oss.oss-cn-beijing.aliyuncs.com/golang_dream/article/static/task.png)

- 用户还款，当用户还款成功时，发送一个计算用户积分计算的任务到任务队列。
- 任务队列，可以是`mq`，也可是`redis`，用来存储任务。

- 任务执行者，任务的执行者，监听任务队列，当任务队列中有任务时，便会执行。



## 区别

消息队列和任务队列，我觉得最大的不同就是理念的不同：任务队列传递的是"任务"，消息队列传递的是"消息"。任务队列可以说是消息队列的二次开发。

![](https://song-oss.oss-cn-beijing.aliyuncs.com/golang_dream/article/static/uion.png)

通过上面两个场景例子，我们可以总结一下两者区别：

- 消息队列更侧重于消息的吞吐、处理，具有有处理海量信息的能力。另外利用消息队列的生产者和消费者的概念，也可以实现任务队列的功能，但是还需要进行额外的开发处理。
- 任务队列则提供了执行任务所需的功能，比如任务的重试，结果的返回，任务状态记录等。虽然也有并发的处理能力，但一般不适用于高吞吐量快速消费的场景。其实任务队列和远程函数调用很像，不过和rpc调用不同，他的调用不是网络请求的方式，而是通过利用消息队列传递任务信息。

综上所述，个人认为任务队列就是消息队列在异步场景下的深度二次开发，根据实际项目开发根据实际场景做相应选择即可。



## 后言

以上全是个人理解，有什么不对的，欢迎指出。

好啦，这一篇文章到这就结束了，我们下期见～～。希望对你们有用。可添加我的golang交流群，我们一起学习交流。

**结尾给大家发一个小福利吧，最近我在看[微服务架构设计模式]这一本书，讲的很好，自己也收集了一本PDF，有需要的小伙可以到自行下载。获取方式：关注公众号：[Golang梦工厂]，后台回复：[微服务]，即可获取。**

**我翻译了一份GIN中文文档，会定期进行维护，有需要的小伙伴后台回复[gin]即可下载。**

**自己翻译了一份`machinery`官方中文文档，会定期维护，有需要的小伙伴后台回复[machinery]即可下载。**

**我是asong，一名普普通通的程序猿，让gi我一起慢慢变强吧。我自己建了一个`golang`交流群，有需要的小伙伴加我`vx`,我拉你入群。欢迎各位的关注，我们下期见~~~**

![](https://song-oss.oss-cn-beijing.aliyuncs.com/wx/qrcode_for_gh_169729c17b9c_344.jpg)

推荐往期文章：

- [machinery入门看这一篇（异步任务队列）](https://mp.weixin.qq.com/s/4QG69Qh1q7_i0lJdxKXWyg)
- [这个缓存更新的套路你都知道吗？](https://mp.weixin.qq.com/s/9GeYSVvKMtKGpTyBWTs_Zw)

- [手把手教姐姐写消息队列](https://mp.weixin.qq.com/s/0MykGst1e2pgnXXUjojvhQ)
- [常见面试题之缓存雪崩、缓存穿透、缓存击穿](https://mp.weixin.qq.com/s?__biz=MzIzMDU0MTA3Nw==&mid=2247483988&idx=1&sn=3bd52650907867d65f1c4d5c3cff8f13&chksm=e8b0902edfc71938f7d7a29246d7278ac48e6c104ba27c684e12e840892252b0823de94b94c1&token=1558933779&lang=zh_CN#rd)
- [详解Context包，看这一篇就够了！！！](https://mp.weixin.qq.com/s/JKMHUpwXzLoSzWt_ElptFg)
- [go-ElasticSearch入门看这一篇就够了(一)](https://mp.weixin.qq.com/s/mV2hnfctQuRLRKpPPT9XRw)
- [面试官：go中for-range使用过吗？这几个问题你能解释一下原因吗](https://mp.weixin.qq.com/s/G7z80u83LTgLyfHgzgrd9g)
- [学会wire依赖注入、cron定时任务其实就这么简单！](https://mp.weixin.qq.com/s/qmbCmwZGmqKIZDlNs_a3Vw)
- [听说你还不会jwt和swagger-饭我都不吃了带着实践项目我就来了](https://mp.weixin.qq.com/s/z-PGZE84STccvfkf8ehTgA)

# Distributed Transaction Solution（分布式事务解决方案）

## 1. 2PC（Two Phase Commitment Protocol两阶段提交）
![2PC](https://github.com/baayso/distributed-transaction-solution/blob/master/images/2PC.png)
* 两种角色
  * 一个事务协调者(Coordinator)：负责协调多个事务参与者进行事务投票及提交、回滚
  * 多个事务参与者(Participants)：本地事务执行者
* 两个阶段
  * 投票阶段(Voting Phase)：事务协调者通知事务参与者准备提交或者取消事务，然后进入表决过程。事务参与者将告知事务协调者自己的决策：同意（事务参与者本地事务执行成功，但未提交）、取消（本地事务执行故障）；
  * 提交阶段（Commit Phase）：收到事务参与者的通知后，协调者再向参与者发出通知，根据反馈情况决定各参与者是否要提交还是回滚。
* 优缺点
  * 优点：尽量保证了数据的强一致，适合对数据强一致要求很高的关键领域。但是如果事务协调者在通知所有事务参与者进行提交或者回滚时，其中某一个事务参与者在接收到通知的一刻突然停电则会造成数据的不一致。
  * 缺点：实现复杂，牺牲了可用性，对性能影响较大，不适合高并发高性能场景。事务协调者必须接收到所有事务参与者的投票通知之后才会通知所有事务参与者进行提交或者回滚，如果某一事务参与者执行速度较慢则会影响整个事务的执行速度。也就是所有的事务参与者都必须等待最慢的事务参与者。
* 2PC的问题
  * 性能问题，在阶段1，锁定资源之后，要等所有节点返回，然后才能一起进入阶段2，不能很好地应对高并发场景。
  * 阶段1完成之后，如果在阶段2事务协调者宕机，则所有的事务参与者接收不到Commit或Rollback指令，将处于“悬而不决”状态。
  * 阶段1完成之后，在阶段2，事务协调者向所有的参与者发送了Commit或Rollback指令，但其中一个事务参与者超时或出错了（没有正确返回ACK），则其他事务参与者提交还是回滚呢？也不能确定。

## 2. 3PC（三阶段提交）


## 3. TCC（Try-Confirm-Cancel，又称事务补偿或代码补偿）
> 和两阶段提交相类似，Try为第一阶段，Confirm-Cancel为第二阶段，是一种应用层面侵入业务的两阶段提交，需要写代码实现Try-Confirm-Cancel三个方法。

![TCC](https://github.com/baayso/distributed-transaction-solution/blob/master/images/TCC.png)

操作方法 | 含义
-|-
Try | 预留业务资源/数据校验，尝试检查当前操作是否可执行 |
Confirm | 确认执行业务操作，实际提交数据，不做任何业务检查，Try成功，Confirm必定成功，需要保证幂等 |
Cancel | 取消执行业务操作，实际回滚数据，需要保证幂等 |

* 示例：假设从A系统账户中扣款并给B账户加款，应用程序首先需要开启事务，然后调用Try方法，检查A系统账户中是否有足够的余额以及检查B系统账户是否存在并给数据加锁。如果所有Try方法都返回成功，则提交事务，事务协调器调用所有Confirm方法对A系统账户扣款以及对B系统账户加款。如果某一个Try方法返回失败，则回滚事务，事务协调器调用所有Cancel方法对数据进行释放锁。
* 注意：事务协调器在调用所有Confirm方法或者Cancel方法时可能某一个Confirm方法或Cancel方法会出现执行超时的情况。出现执行超时后事务协调器会重新调用超时的方法（重试机制），需要保证所有Confirm方法和Cancel方法的幂等性。
* 优缺点
  * 优点：与2PC相比，实现以及流程相对简单，但数据的一致性比2PC要差。
  * 缺点：需要手动编写Try-Confirm-Cancel三个方法的实现，如果项目中使用TCC事务的业务比较多时需要编写大量的代码，以及后续的代码维护工作量。

## 4. 可靠消息最终一致性
![可靠消息最终一致性方案2-独立消息服务](https://github.com/baayso/distributed-transaction-solution/blob/master/images/%E5%8F%AF%E9%9D%A0%E6%B6%88%E6%81%AF%E6%9C%80%E7%BB%88%E4%B8%80%E8%87%B4%E6%80%A7%E6%96%B9%E6%A1%882-%E7%8B%AC%E7%AB%8B%E6%B6%88%E6%81%AF%E6%9C%8D%E5%8A%A1.png)

* 【消息状态确认子系统】定时查询“状态确认”超时的消息，然后查询【主动方应用系统】业务操作结果，如果业务操作成功则确认并发送消息，如果业务操作失败则删除消息。产生“状态确认”超时消息的异常情况：
  * 【主动方应用系统】预发送消息，【消息服务子系统】成功存储消息，但是返回结果时由于网络超时导致【主动方应用系统】未接收到返回结果。
  * 【主动方应用系统】预发送消息成功，业务操作失败。
  * 【主动方应用系统】预发送消息成功，【主动方应用系统】执行本地业务成功，但是发送业务操作结果由于网络超时未能成功。
* 【消息恢复子系统】定时查询“消息确认”超时的消息并进行重新投递。
* 【消息管理子系统】可以对消息进行集中管理，如：查询超时消息、查询多次投递失败的消息、批量重投失败消息。
* 优化：
  * 消息数据的存储在数据量大时可以考虑使用非关系型数据库。
  * 


## 5. 柔性事务-最大努力通知（定期校对）
![柔性事务-最大努力通知（定期校对）](https://github.com/baayso/distributed-transaction-solution/blob/master/images/%E6%9F%94%E6%80%A7%E4%BA%8B%E5%8A%A1-%E6%9C%80%E5%A4%A7%E5%8A%AA%E5%8A%9B%E9%80%9A%E7%9F%A5%EF%BC%88%E5%AE%9A%E6%9C%9F%E6%A0%A1%E5%AF%B9%EF%BC%89.png)

* 特点：
  * 【主动方应用系统】在完成业务处理后，向业务【被动方应用系统】发送通知消息（允许消息丢失）
  * **【主动方应用系统】可以设置时间阶梯型的通知规则，在通知失败后按规则重复通知，直至通知【设定的最大通知次数】后不再通知**
  * 【主动方应用系统】提供校对查询接口给【被动方应用系统】按需调用，用于恢复丢失的业务消息
* 适用场景：
  * 对最终一致性时间敏感度低的跨企业或垮系统的业务，如：充值、银行支付（支付成功后通知）
* 优化：
  * 通知记录、通知日志可视化管理，手工触发等功能（消息管理）
  * 区分通知队列、不同队列可配置不同的通知规则
* 注意：
  * 【通知服务】在启动时需要加载数据库中的通知记录
  * 【被动方应用系统】接收通用的接口需要实现幂等性

## 6. Seata（ http://seata.io ）

* Seata 是什么?
  * https://seata.io/zh-cn/docs/overview/what-is-seata.html

* Seata术语
  * TC - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
  * TM - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
  * RM - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

* AT模式（Automatic (Branch) Transaction Mode）
  > AT 模式基于 支持本地 ACID 事务 的 关系型数据库。
  > 提供无侵入自动补偿的事务模式，目前已支持 MySQL、 Oracle 的AT模式、PostgreSQL、H2 开发中。
  > http://seata.io/zh-cn/docs/dev/mode/at-mode.html
  * 一阶段 prepare 行为：在本地事务中，一并提交业务数据更新和相应回滚日志记录。
  * 二阶段 commit 行为：马上成功结束，自动 异步批量清理回滚日志。
  * 二阶段 rollback 行为：通过回滚日志，自动 生成补偿操作，完成数据回滚。

* TCC模式（TCC (Branch) Transaction Mode）
  > 不依赖于底层数据资源的事务支持。
  > http://seata.io/zh-cn/docs/dev/mode/tcc-mode.html
  * 一阶段 prepare 行为：调用 自定义 的 prepare 逻辑。
  * 二阶段 commit 行为：调用 自定义 的 commit 逻辑。
  * 二阶段 rollback 行为：调用 自定义 的 rollback 逻辑。
  * 所谓 TCC 模式，是指支持把 自定义 的分支事务纳入到全局事务的管理中。
  * **TCC 适用模型与适用场景分析**（ https://seata.io/zh-cn/blog/tcc-mode-applicable-scenario-analysis.html ）

* Saga模式
  > Saga模式是SEATA提供的长事务解决方案，在Saga模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。
  > http://seata.io/zh-cn/docs/dev/mode/saga-mode.html

* **分布式事务 Seata 及其三种模式详解**
  * https://seata.io/zh-cn/blog/seata-at-tcc-saga.html
  * https://tech.antfin.com/community/activities/779/review/901

* Spring Cloud 快速集成 Seata
  * https://github.com/seata/seata-samples/blob/master/doc/quick-integration-with-spring-cloud.md

## 参考文献：
* 分布式事务解决方案汇总：2PC、3PC、消息中间件、TCC、状态机+重试+幂等（ https://www.cnblogs.com/myseries/p/10939355.html 、 https://blog.csdn.net/uxiAD7442KMy1X86DtM3/article/details/88968532 ）
* Seata官方文档（ http://seata.io ）

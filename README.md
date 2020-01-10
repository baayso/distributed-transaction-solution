# Distributed Transaction Solution（分布式事务解决方案）

## 1. 2PC（Two Phase Commitment Protocol两阶段提交）与 XA协议
* 两种角色
  * 一个事务协调者(Coordinator)：负责协调多个事务参与者进行事务投票及提交、回滚
  * 多个事务参与者(Participants)：本地事务执行者
* 两个阶段
  * 投票阶段(Voting Phase)：事务协调者通知事务参与者准备提交或者取消事务，然后进入表决过程。事务参与者将告知事务协调者自己的决策：同意（事务参与者本地事务执行成功，但未提交）、取消（本地事务执行故障）；
  * 提交阶段（Commit Phase）：收到事务参与者的通知后，协调者再向参与者发出通知，根据反馈情况决定各参与者是否要提交还是回滚。
* 优缺点
  * 优点：尽量保证了数据的强一致，适合对数据强一致要求很高的关键领域。但是如果事务协调者在通知所有事务参与者进行提交或者回滚时，其中某一个事务参与者在接收到通知的一刻突然停电则会造成数据的不一致。
  * 缺点：实现复杂，牺牲了可用性，对性能影响较大，不适合高并发高性能场景。事务协调者必须接收到所有事务参与者的投票通知之后才会通知所有事务参与者进行提交或者回滚，如果某一事务参与者执行速度较慢则会影响整个事务的执行速度。也就是所有的事务参与者都必须等待最慢的事务参与者。

## 2. 3PC（三阶段提交）


## 3. TCC（Try-Confirm-Cancel，又称补偿事务）


## 4. 可靠消息最终一致性


## 5. 柔性事务-最大努力通知（定期校对）


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

* Saga模式
  > Saga模式是SEATA提供的长事务解决方案，在Saga模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。
  > http://seata.io/zh-cn/docs/dev/mode/saga-mode.html

* Spring Cloud 快速集成 Seata
  * https://github.com/seata/seata-samples/blob/master/doc/quick-integration-with-spring-cloud.md

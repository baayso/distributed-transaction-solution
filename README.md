# Distributed Transaction Solution（分布式事务解决方案）

## 1. 2PC（两阶段提交）与 XA协议


## 2. 3PC（三阶段提交）


## 3. TCC（Try-Confirm-Cancel，又称补偿事务）


## 4. 最终一致性（消息队列）


## 5. 弱一致性（消息队列）


## 6. Seata（ http://seata.io ）

* Seata 是什么?
  * https://seata.io/zh-cn/docs/overview/what-is-seata.html

* Seata术语
  * TC - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
  * TM - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
  * RM - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

* Spring Cloud 快速集成 Seata
  * https://github.com/seata/seata-samples/blob/master/doc/quick-integration-with-spring-cloud.md

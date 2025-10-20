---
source: https://www.cnblogs.com/ciel717/p/19101222
create: 2025-10-10 13:34
read: true
knowledge: true
knowledge-date: 2025-10-10
tags:
  - 数据库
  - 系统架构
summary: "[[数据库扩容方案]]"
---
随着业务量的快速增长，经常会遇到由于关系型数据库（如:`MySql`）单表数据量增长过大而引发的线上事故；虽然这些事故多数时候是由于不合理的慢 SQL 而引起的系统雪崩，但有时也会出现由于数据库热点块 IO 争用而引发的系统性性能下降。总之，单表数据量的无限增长总是会在这样或那样的情况下增加系统的不稳定性因素。所以在大规模实时系统的设计中，除了重点考虑应用结构的分布式化外，往往也不应该忽略数据库实时存储、计算能力扩展性方面的考虑。

目前解决实时数据增长一般有两种思路：一种是直接采用分布式数据库（例如:`Tidb`、`OceanBase`之类）；另一种是对关系型数据库进行分库分表来最大化利用现有数据库的实时计算能力。绝大部分情况下，后一种方案往往会更现实一些。

本文的主要内容就是通过模拟一个交易系统的订单库，来具体演示如何通过`ShardingJdbc`实现交易订单数据的分库分表存储。在这个过程中会到涉及分库分表实践的三种主要场景：

1.  新系统在设计之初直接使用分库分表方案；
2.  历史系统运行一段时间后如何平滑地实施分库分表；
3.  对现有分库分表逻辑的`Scaling`操作 (包括减少分表、增加分表) 涉及的数据迁移问题。

交易系统的订单数据是分库分表的一个非常典型场景，由于交易系统对单条数据的实时处理性能要求很高，所以一旦单个订单表数据量规模达到 10 亿 +，就很容易出现由于数据库热点块 IO 争用而导致的性能下降，也很容易出现个别不谨慎的 SQL 操作而引起的系统性雪崩。

但一旦决定实施分库分表就要提前做好存储规划，并对未来数据增长的规模进行一定的评估，同时做好未来增加分库、增加分表的系统`Scaling`方案。此外，分库分表的实施还要考虑应用的接入难度，分库分表的细节逻辑应该对应用透明；所以一般来说我们需要一个中间代理层来屏蔽分库分表对应用程序本身带来的侵入。

目前在 Java 社区中比较知名的分库分表代理组件就是`ShardingJdbc`（目前已被集成在`Apache`开源项目`ShardingSphere`之中），`ShardingJdbc`本质上是一个轻量级的 JDBC 驱动代理，在使用的过程中只需要依赖相关 Jar 包即可，并不需要额外部署任何服务。通过系统配置文件就可以实现分库分表逻辑的定义，并实现应用透明代理访问。

接下来，我们以`SpringBoot`为例演示如何集成`ShardingJdbc`实现对交易订单的分库分表操作，具体步骤如下：

## 2.1 分库分表规划

在系统设计之初，如果能够预见到未来数据量的增长规模，那么提前做好分库分表规划是非常有远见的。从分库分表的形式上来说，一般可以有两类规划方式：

1.  单库水平分表，如果单一数据库计算能力比较强，可以在同一个库中进行数据表的水平拆分；
2.  分库 + 分表，如果数据规模爆炸式增长，单库的计算资源有限，为了提升数据库的整体计算处理性能，也可以同时实现多个库的分库分表存储。

在本文的实例中，我们将订单数据分库分表规划为：

1.  数据库节点 2 个（ds0、ds1）；
2.  每个库的分表数为 32 张表（0~31）。

订单表的整体数据分库分表逻辑是根据订单表中的 “user_id 字段 %2” 实现分库；然后在分库逻辑的基础上根据订单表中的 “order_id 字段 %32” 实现水平分表。

例如，有条`user_id`为`1001`、订单编号为`20200713001`的订单数据，根据上述分库分表规则`1001%2=1`，`20200713001%32=9`，那么该数据将存储在`ds1`库中的第`9`个分表。

具体的订单逻辑表结构如下：

```sql
create table t_order (
    `id` bigint not null primary key auto_increment,
    `order_id` bigint comment '业务方订单号（业务方系统唯一）',
    `trade_type` varchar(30) comment '业务交易类型，例如topup-表示钱包充值',
    `amount` bigint comment '交易金额，以分为单位',
    `currency` varchar(10) comment '币种，cny-人民币',
    `status` varchar(2) comment '支付状态，0-待支付；1-支付中；2-支付成功；3-支付失败',
    `channel` varchar(10) comment '支付方式，0-微信支付，1-支付宝支付',
    `trade_no` varchar(32) comment '流水号',
    `user_id` bigint(60) comment '业务方用户id',
    `remark` varchar(128)  comment '备注',
    `create_time` timestamp null default current_timestamp comment '创建时间',
    `update_time` timestamp null default current_timestamp on update current_timestamp comment '更新时间',
    key unique_idx_pay_id ( order_id ),
    key idx_user_id (user_id ),
    key idx_create_time (create_time)
) comment = '交易订单表';


```

以上逻辑表的具体分表形式为`t_order_{0~31}`，分别分布在`ds0`、`ds1`两个数据库节点中。

## 2.2 工程结构

首先创建一个基于`Maven`构建的`SpringBoot`项目，并集成`MyBatis`数据库访问框架，代码结构如下：

![](https://s2.51cto.com/images/blog/202108/01/8418cd307d2e27f0284b118af3ad74c1.png)

如上图所示，我们创建了一个基于`Spring Boot`的基本工程，并在集成了基于`Mybatis`的数据库访问功能。

## 2.3 分库分表规则配置

接下来我们来看下在`Spring Boot`项目中如何集成`ShardingJdbc`，并按照规划的分库分表规则进行具体的配置。

首先引入`ShardingJdbc`针对`SpringBoot`项目的`starter`依赖包，具体如下：

```xml
<!-- 引入Sharding-JDBC Spring Boot依赖组件 -->
<!-- Sharding-JDBC For Spring Boot Start -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>
<!-- for spring namespace -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>
<!-- Sharding-JDBC For Spring Boot End -->


```

引入`Spring Boot Starter`依赖后，`ShardingJdbc`会使用自己的数据源配置逻辑，为避免冲突需要在主类中排除掉默认的数据源自动配置类，具体如下：

```java
//排除掉默认的数据源自动配置类
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class OrderServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServerApplication.class, args);
    }
}


```

完成上述操作后，从工程逻辑上看就已经完成了`ShardingJdbc`与`Spring Boot`应用的集成。接下来我们要做的就是根据规划的分库分表规则，通过配置文件进行分库分表规则的配置，具体如下：

```yml
spring:
  shardingsphere:
    #SQL控制台打印（开发时配置）
    props:
      sql:
        show: true
    datasource:
      # 配置真实数据源
      names: ds0,ds1
      # 配置第1个数据源
      ds0:
        driver-class-name: com.mysql.jdbc.Driver
        password: 123456
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/order_0?characterEncoding=utf-8
        username: root
      # 配置第1个数据源
      ds1:
        driver-class-name: com.mysql.jdbc.Driver
        password: 123456
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/order_1?characterEncoding=utf-8
        username: root
    sharding:
      tables:
        t_order:
          # 配置t_order表规则
          actual-data-nodes: ds$->{0..1}.t_order_$->{0..31}
          # 配置t_order表分库策略（inline-基于行表达式的分片算法）
          database-strategy:
            inline:
              algorithm-expression: ds${user_id % 2}
              sharding-column: user_id
          # 配置t_order表分表策略
          table-strategy:
            inline:
              algorithm-expression: t_order_$->{order_id % 32}
              sharding-column: order_id
      #如其他表有分库分表需求，配置同上述t_order表
      # ...


```

上述配置文件中，我们配置了两个数据源，对应的数据库分别为`order_0`、`order_1`，这两个数据库中分别存储了`t_order`这张逻辑表的分表信息`{0~31}`，总共 64 张数据库来分散存储订单信息。分库分表的维度主要有 2 个分别是：用户 ID 作为分库键、订单 ID 作为分表键。

## 2.4 测试

通过上述步骤，到这里我们已经从功能上完成了针对订单表的分库分表逻辑。具体针对订单表的操作逻辑，还是和正常使用`Mybatis`操作数据库表一样，并不需要针对分库分表进行额外的代码操作，因为`ShardingJdbc`会在数据库驱动层拦截`SQL`并进行分库分表规则的匹配及路由操作。

和正常基于`Spring Mvc`的开发一样，我们编写一个基于`Mvc`分层的订单创建接口, 启动应用程序，效果如下：

![](https://s2.51cto.com/images/blog/202108/01/e7e960fb2fc62d9c784acff8aa15ad6b.png)

可以看到从编程方式上看，与我们平时写`Java`代码的分层结构完全一致，此时模拟调用该订单创建接口，具体请求参数如下：

```json
{
    "orderId":123458,
    "tradeType":"topup",
    "amount":1000,
    "currency":"cny",
    "userId":63631725
}


```

按照该请求参数的数据规则，此条订单应该存在编号为 1 数据库中编号 2 的分表中，具体计算（“userId->63631725%2=1;orderId->123458%32=2”），完成接口调用后可以查询数据库表数据进行验证！

前面我们演示了在预先规划好分库分表结构的情况下，使用`ShardingJdbc`实现了应用透明的分库分表操作。但大部分情况下，能够有先见之明提前规划好长远数据表分散存储方案的系统是很少的，只有当数据规模达到一定的级别，系统性能遇到相应瓶颈时分库分表方案才会被顺理成章的放到桌面选项中。

而这一般又会涉及两个场景的场景：

1.  尚未进行分库分表的单库单表系统如何平稳的实施分库分表方案；
2.  已经实时过分库分表方案的系统，由于数据量的持续增长导致原有分库分表不够用了，需要二次扩容的情况。

上述两个场景，无论哪种情况都会面临由于存储规则改变而需要大量进行数据迁移的情况，这对于正在线上运行的系统来说，无异于是要给正在高速行驶中的汽车换轮子，稍有不慎就会造成系统崩溃的严重后果。所以敢于对这类系统进行结构性重塑的程序员都是真的猛士！

具体将以如下两个常见的场景进行演示：

1)、尚未进行分库分表的单库单表系统如何平稳的实施分库分表方案；  
2)、已经实施过分库分表方案的系统，由于数据量的持续增长导致原有分库分表不够用了，需要二次扩容的情况。

## 3.1 实施方案概述

在具体演示之前，我们先简单聊一聊关于分库分表数据迁移中，几种常见的思路，具体如下：

### 3.1.1 停服迁移

停服迁移方案，是数据迁移方案中最常见、也是相对安全，但是也是最不能让人接受的方案。之所以说它安全，是因为停服之后就不会有新的数据写入，这样能保证数据迁移工作能够在一个相对稳定的环境中进行，因而能较大程度上避免迁移过程中产生数据不一致的问题。

但停服务方案会比较严重伤害用户体验、降低服务的可用性，而如果数据量较大迁移需要大量的时间的话，长时间的停服会严重影响业务，对于强调 "9999" 服务体验的互联网公司来说，停服方案绝对是让人不能接受的。

一般来说停服方案更适合于哪些非 7X24 小时，且对自身数据一致性要求非常高的系统，例如社保基金、银行核心系统等。当然，这也并不是说非停服方案，做不到数据迁移的绝对准确，仅仅在于这些系统出于管理、规则等非技术因素上的考虑是否能够容忍小概率的风险事件罢了。

停服迁移的流程，一般如下：

1.  提前进行演练、预估停服时间，发布停服公告；
2.  停服，通过事先准备好的数据迁移工具（一般为迁移脚本），按照新的数据分片规则，进行数据迁移；
3.  核对迁移数据的准确性；
4.  修改应用程序代码，切换数据分片读写规则，并完成测试验证；
5.  启动服务，接入外部流量；

### 3.1.2 升级从库方案

关于升级从库的方案，一般是针对线上数据库配置了主从同步结构的系统来说的。其具体思路是，当需要重新进行分库分表扩容时，可将现有从库直接升级成主库。例如原先分库分表结构是 A、B 两个分库为主库、A0、B0 分别为 A、B 对应的从库，具体如下图所示：

![](https://s2.51cto.com/images/blog/202102/08/2e45fd1abea25c1b4442f39f5a63a012.png)

假设此时如果需要扩容，新增两个分库的话，那么可以将 A0、B0 升级为主库，如此原先的两个分库将变为 4 个分库。与此同时，将上层服务的分片规则进行更改，将原先 uid%2=0(原先存在 A 库) 的数据分裂为 uid%4=0 和 uid%=2 的数据分别存储在 A 和 A0 上；同时将 uid%2=1(原先存在 B 库) 的数据分裂为 uid%4=1 和 uid%=3 的数据分别存储在 B 和 B0 上。而因为 A0 库和 A 库，B0 库与 B1 库数据相同，所以这样就不需要进行数据迁移了，只需要变更服务的分片规则配置即可。之后结构如下：

![](https://s2.51cto.com/images/blog/202102/08/a89a982f365ac887dc93a5ae04d76e09.png)

而之前 uid%2 的数据分配在 2 个库中，此时分散到 4 个库，由于老数据还在，所以 uid%4=2 的数据还有一半存储在 uid%4=0 的分库中。因此还需要对冗余数据进行清理，但这类清理并不影响线上数据的一致性，可以随时随地进行。

处理完成后，为保证高可用，以及下一步扩容的需求，可以为现有主库再次分配一个从库，具体如下图所示：

![](https://s2.51cto.com/images/blog/202102/08/d7012275d80e991ff79834df5437b43e.png)

升级从库方案的流程，一般如下：

1.  修改服务数据分片配置，做好新库和老库的数据映射；
2.  DBA 协助完成从库升级为主库的数据库切换；
3.  DBA 解除现有数据库主从配置关系；
4.  异步完成冗余数据的清理；
5.  重新为新的数据节点搭建新的从库；

这种方案避免了由于数据迁移导致的不确定性，但也有其局限性。首先，现有数据库主从结构要能满足新的分库分表的规划；其次，这种方案的主要技术风险点被转移至 DBA，实施起来可能会有较大阻力，毕竟 DBA 也不见得愿意背这个锅；最后，由于需要在线更改数据库的存储结构，可能也会出现意想不到的情况，而如果还存在多应用共享数据库实例情况的话，情况也会变得比较复杂。

### 3.1.3 双写方案

双写方案是针对线上数据库迁移时使用的一种常见手段，而对于分库分表的扩容来说，也涉及到数据迁移，所以也可以通过双写来协助分库分表扩容的问题。

双写方案实际上同升级从库的原理类似，都是做 "分裂扩容"，从而减少直接数据迁移的规模降低数据不一致的风险，只是数据同步的方式不同。双写方案的核心步骤是：

1.  是直接增加新的数据库，并在原有分片逻辑中增加写链接，同时写两份数据；
2.  与此同时，通过工具 (例如脚本、程序等) 将原先老库中的历史数据逐步同步至新库，此时由于新库只有新增写入，应用上层其他逻辑还在老库之中，所以数据的迁移对其并无影响；
3.  对迁移数据进行校验，由于是业务直接双写，所以新增数据的一致性是非常高的（但需要注意 insert、update、delete 操作都需要双更新操作）；
4.  完成数据迁移同步，并校验一致后就可以在应用上层根据老库的分裂方式重新修改分片配置规则了。

以前面的例子为例，双写方案如下图所示：

![](https://s2.51cto.com/images/blog/202102/08/c7abe2cd09b780bab5747932e090dcf2.png)

如上图所示原先的 A、B 两个分库，其中 uid%2=0 的存放在 A 库，uid%2=1 的存放在 B 库；增加新的数据库，其中写入 A 库是双写 A0 库，写入 B 库时双写 B0 库。

之后分别将 A 库的历史数据迁移至 A0 库；B 库的历史数据迁移至 B0 库。最终确保 A 库与 A0 库的数据一致；B 库与 B0 库的数据一致。具体如下图所示：

![](https://s2.51cto.com/images/blog/202102/08/680d106c36766a8e7b897f6365887386.png)

之后修改分片规则，确保原先 uid%2=0 存放在 A 库的数据，在分裂为 uid%4=0 和 uid%4=2 的情况下能分别存储在 A 库和 A0 库中；原先 uid%2=1 存放在 B 库的数据，在分裂为 uid%4=1 和 uid%4=3 的情况下分别存在到 B 库和 B0 库之中。具体如下图所示：

![](https://s2.51cto.com/images/blog/202102/08/6799fd05ac1ad00f411b541cd4e0abfd.png)

双写方案避免了像升级从库那样改变数据库结构的风险，更容易由开发人员自己控制，但双写方案需要侵入应用代码，并且最终需要完成数据迁移和冗余数据删除两个步骤，实施起来也不轻松。

## 3.2 在线扩容

那么到底有没有一个绝对完美的方案呢？

答案其实是没有！因为无论哪种方案都无法避免要迁移数据，即便像升级从库那样避免了数据迁移，也无法避免对冗余数据进行删除的额外操作。但我们可以在数据迁移手段和重新处理数据分片的方式上进行优化，目前在分库分表领域著名的开源项目`ShardingSphere`本质上其实就是在这两方面进行了优化，从而提供了一组解决方案工具集。

### 3.2.1 迁移方案

具体来说`ShardingSphere`是由 "`Sharding-JDBC`+`Sharding-Proxy`+`Sharding-Scaling`"三个核心组件组成的分库分表开源解决方案，而"`Sharding-Proxy`+`Sharding-Scaling`" 则是专门用于设计处理分库分表扩容数据迁移问题的组件，其运行原理如下：

![](https://s2.51cto.com/images/blog/202102/08/7f311ca10fbf7d0640e201f6cc79ea58.png)

如上图所示，在`ShardingSphere`的解决方案中，当需要对`Service`(旧) 服务进行分库分表扩容时，我们可以先部署`Sharding-Scaling`+`Sharding-Proxy`组件进行数据迁移 + 数据分片预处理。具体来说步骤如下：

1.  在`Sharding-Proxy`中按照扩容方案配置好分片规则，启动服务并提供`JDBC`协议连接机制，此时通过`Sharding-Proxy`连接写入的数据会按照新的分片规则进行数据存储；
2.  部署`Sharding-Scaling`服务，并通过`HTTP`接口的形式，向其发送数据迁移任务配置（配置数据中有需要迁移的数据库连接串，也有`Sharding-Proxy`的数据连接串）；
3.  启动`Sharding-Scaling`迁移任务后，`Sharding-Scaling`将根据目标数据源的`Binlog`日志变化，读取后重新发送至`Sharding-Proxy`进行分片数据的重新处理；
4.  当`Sharding-Scaling`迁移数据任务完成，检查数据迁移结果，如果没有问题，则可以修改`Service`（新）服务的数据分片规则，并完成`Service`（旧）服务的替换；
5.  确认扩容无误后，停止`Sharding-Proxy`+`Sharding-Scaling`服务；
6.  异步完成冗余数据的清理（目前`ShardingSphere`还不支持数据迁移后自动完成冗余数据的清理，所以需要自己根据数据分裂规则，编写清除脚本）；

上述过程完全自动，在完成数据迁移及重新分片前，旧服务保持持续服务，不会对线上造成影响；此外由于`Sharding-Scaling`一直处于监听目标数据源`Binlog`日志状态，所以即便在服务切换过程中，旧`Service`服务仍有数据写入，也会自动被`Sharding-Proxy`重新分片处理，所以不用担心会出现不一致。

此外`Sharding-Scaling`+`Sharding-Proxy`通信方式采用的是`JDBC`连接原生连接方式以及基于`Binlog`日志的同步方案，所以在迁移效率上也是有保证的。接下来将基于`Sharding-Scaling`+`Sharding-Proxy`方案具体演示在不停服的情况下进行分库分表在线扩容。

### 3.2.2 在线扩容

还是以上一篇文章中的订单分库分表存储为例，将其原有的分库分表规划：

1.  数据库节点 2 个 (ds0、ds1)；
2.  每个库的分表数为 32 张表 (031)。

扩容为：

1.  数据库节点 4 个 (ds0、ds1、ds2、ds3)；
2.  每个库的分表数仍为 32 张表（031）。

首先我们部署`Sharding-Scaling`+`Sharding-Proxy`进行在线数据迁移及数据分片处理，具体如下：

1)、部署 Sharding-Proxy

该服务的作用是一个数据库中间件，我们在此服务上编辑好分库分表规则后，`Sharding-Scaling`会把原数据写入`Sharding-Proxy`，然后由`Sharding-Proxy`对数据进行路由后写入对应的库和表。

我们可以在`Github`上下载`ShardingSphere`对应的版本的源码（演示所用版本为`4.1.1`）自行编译，也可以下载已经编译好的版本文件。本次演示所用方式为源码编译，其执行程序目录如下：

```bash
/shardingsphere-4.1.1/sharding-distribution/sharding-proxy-distribution/target/apache-shardingsphere-4.1.1-sharding-proxy-bin.tar.gz


```

找到编译执行程序后进行解压！之后编辑 "conf/server.yaml" 文件，添加连接账号配置，具体如下：

```yml
authentication:
  users:
    root:
      password: 123456


```

该配置主要是供`Sharding-Scaling`连接`Sharding-Proxy`时使用！之后编辑`Sharding-Proxy`的分库分表配置文件 "conf/config-sharding.yaml"，按照新的扩容方案进行配置，具体如下：

```yml
#对外暴露的数据库名称
schemaName: order
dataSources:
  ds_0:
    url: jdbc:mysql://127.0.0.1:3306/order_0?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 300
  ds_1:
    url: jdbc:mysql://127.0.0.1:3306/order_1?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 300

  ds_2:
    url: jdbc:mysql://127.0.0.1:3306/order_2?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 300
  ds_3:
    url: jdbc:mysql://127.0.0.1:3306/order_3?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 300
shardingRule:
  tables:
    t_order:
      actualDataNodes: ds_${0..3}.t_order_$->{0..31}
      databaseStrategy:
        inline:
          shardingColumn: user_id
          algorithmExpression: ds_${user_id % 4}
      tableStrategy:
        inline:
          shardingColumn: order_id
          algorithmExpression: t_order_${order_id % 32}
      keyGenerator:
        type: SNOWFLAKE
        column: id


```

完成分库分表配置后，启动`Sharding-Proxy`服务，命令如下：

```bash
sh bin/start.sh


```

为了验证`ShardingProxy`的部署正确下，可以通过`Mysql`命令进行连接，并插入一条数据，验证其是否按照新的分库分表规则进行存储，具体如下：

```bash
#使用mysql客户端检查是否能链接成功
mysql -h 127.0.0.1 -p 3307 -uroot -p123456
mysql> show databases;
+----------+
| Database |
+----------+
| order    |
+----------+
1 row in set (0.02 sec)


#执行以下脚本，写入成功后， 检查order_0～3的分表数据是否按照规则落库
mysql> insert into t_order values('d8d5e92550ba49d08467597a5263205b',10001,'topup',100,'CNY','2','3','1010010101',63631722,now(),now(),'测试sharding-proxy');
Query OK, 1 row affected (0.20 sec)


```

按照新的分库分表规则 uid->63631722%4=2;orderId->10001%32=17，测试数据应该落在 ds_2 中的 t_order_17 表中！如正确插入，则说明新的分库分表规则配置正确。

2)、部署 Sharding-Scaling

源码编译的`Sharding-Scaling`执行程序包路径为：

```bash
/shardingsphere-4.1.1/sharding-distribution/sharding-scaling-distribution/target/apache-shardingsphere-4.1.1-sharding-scaling-bin.tar.gz


```

解压后，启动`Scaling`服务，命令如下：

```bash
sh bin/start.sh 


```

`Sharding-Scaling`是一个独立的数据迁移服务，其本身不与任何具体的环境关联，在创建迁移任务时，具体信息由接口传入。接下来我们调用`Sharding-Scaling`创建具体的迁移任务。

在此之前，原有分库中的数据有：

```bash
1)、userId=63631725;orderId=123458存储在ds_1中的t_order_2号表中；
2)、userId=63631722;orderId=123457存储在ds_0中的t_order_1号表中；


```

而按照新的规则数据 1 不需要迁移，数据 2 需要迁移至`ds_2`中的`t_order_1`号表中，具体创建`Sharding-Scaling`迁移任务的指令如下：

```bash
#提交order_0的迁移数据命令
curl -X POST --url http://localhost:8888/shardingscaling/job/start \
--header 'content-type: application/json' \
--data '{
    "ruleConfiguration": {
        "sourceDatasource": "ds_0: !!org.apache.shardingsphere.orchestration.core.configuration.YamlDataSourceConfiguration\n  dataSourceClassName: com.zaxxer.hikari.HikariDataSource\n  properties:\n    jdbcUrl: jdbc:mysql://127.0.0.1:3306/order_0?serverTimezone=UTC&useSSL=false&zeroDateTimeBehavior=convertToNull\n    driverClassName: com.mysql.jdbc.Driver\n    username: root\n    password: 123456\n    connectionTimeout: 30000\n    idleTimeout: 60000\n    maxLifetime: 1800000\n    maxPoolSize: 100\n    minPoolSize: 10\n    maintenanceIntervalMilliseconds: 30000\n    readOnly: false\n",
        "sourceRule": "tables:\n  t_order:\n    actualDataNodes: ds_0.t_order_$->{0..31}\n    keyGenerator:\n      column: order_id\n      type: SNOWFLAKE",
        "destinationDataSources": {
            "name": "dt_1",
            "password": "123456",
            "url": "jdbc:mysql://127.0.0.1:3307/order?serverTimezone=UTC&useSSL=false",
            "username": "root"
        }
    },
    "jobConfiguration": {
        "concurrency": 1
    }
}'


```

以上我们提交了针对老库`order_0`的数据迁移任务，如果提交成功`Sharding-Scaling`会返回如下信息：

```json
{"success":true,"errorCode":0,"errorMsg":null,"model":null}


```

此时可通过命令查看任务详情进度，命令及结果显示如下：

```bash
#curl  http://localhost:8888/shardingscaling/job/progress/1;
{
    "success": true,
    "errorCode": 0,
    "errorMsg": null,
    "model": {
        "id": 1,
        "jobName": "Local Sharding Scaling Job",
        "status": "RUNNING",
        "syncTaskProgress": [{
            "id": "127.0.0.1-3306-order_0",
            "status": "SYNCHRONIZE_REALTIME_DATA",
            "historySyncTaskProgress": [{
                "id": "history-order_0-t_order_24#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_25#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_22#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_23#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_20#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_21#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_19#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_17#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_18#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_15#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_16#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_13#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_14#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_8#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_11#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_9#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_12#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_6#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_31#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_7#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_10#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_4#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_5#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_30#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_2#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_3#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_0#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_1#0",
                "estimatedRows": 1,
                "syncedRows": 1
            }, {
                "id": "history-order_0-t_order_28#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_29#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_26#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }, {
                "id": "history-order_0-t_order_27#0",
                "estimatedRows": 0,
                "syncedRows": 0
            }],
            "realTimeSyncTaskProgress": {
                "id": "realtime-order_0",
                "delayMillisecond": 8759,
                "logPosition": {
                    "filename": "mysql-bin.000001",
                    "position": 190285,
                    "serverId": 0
                }
            }
        }]
    }
}


```

同样针对其他老库，如`order_1`数据库的迁移，也可以以类似的方式提交迁移任务！如果此时观察数据 2，就会发现其已经被重新分片到`ds_2`的`t_order_1`号表中；但其之前的历史分片数据仍然会冗余在`ds_0`的`t_order_1`号表中 (需要清理)。

假设此时，有一条通过老服务写入的 "userId=63631723&orderId=123457" 数据，那么其会被存储在`ds_1`的`t_order_1`号表中，而其最新存储规则应该在`ds_3`的`t_order_1`号表中。如果之前也同时开启了`order_1`库的`Scaling`数据迁移任务，那么此时该数据将会被自动重新迁移并分片至`ds_3`的`t_order_1`号表中。

完成数据迁移后，就可以将之前旧服务的分片规则进行调整并重新发布了，具体如下：

```yml
spring:
  shardingsphere:
    #SQL控制台打印（开发时配置）
    props:
      sql:
        show: true
    datasource:
      # 配置真实数据源
      names: ds0,ds1,ds2,ds3
      # 配置第1个数据源
      ds0:
        driver-class-name: com.mysql.jdbc.Driver
        password: 123456
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/order_0?characterEncoding=utf-8
        username: root
      # 配置第2个数据源
      ds1:
        driver-class-name: com.mysql.jdbc.Driver
        password: 123456
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/order_1?characterEncoding=utf-8
        username: root
      # 配置第3个数据源
      ds2:
        driver-class-name: com.mysql.jdbc.Driver
        password: 123456
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/order_2?characterEncoding=utf-8
        username: root
      # 配置第4个数据源
      ds3:
        driver-class-name: com.mysql.jdbc.Driver
        password: 123456
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/order_3?characterEncoding=utf-8
        username: root
    sharding:
      tables:
        t_order:
          # 配置t_order表规则		
          actual-data-nodes: ds$->{0..3}.t_order_$->{0..31}
          # 配置t_order表分库策略（inline-基于行表达式的分片算法）
          database-strategy:
            inline:
              algorithm-expression: ds${user_id % 2}
              sharding-column: user_id
          # 配置t_order表分表策略
          table-strategy:
            inline:
              algorithm-expression: t_order_$->{order_id % 32}
              sharding-column: order_id

          #如其他表有分库分表需求，配置同上述t_order表
          # ...


```

以上内容详细介绍并演示了针对已经分库分表的系统进行二次扩容时使用`Sharding-Scaling`+`Sharding-Proxy`进行数据迁移的过程；而关于尚未进行过分库分表，但需要进行分库分表的系统来说，其过程与二次扩容并无差别，这里就不再赘述！

在具体的分库分表实践中还需要注意表的主键问题，一般可以考虑分布式`ID`生成方案，例如`UUID`等，避免在扩容迁移数据时发生主键冲突。另外关于`Sharding-Scaling`+`Sharding-Proxy`的使用，也需要注意一些异常情况，目前`Sharding-Scaling`的版本还是`Alpha`版本，所以使用过程中不排除会有一些问题，可以多看看源码，增进了解！
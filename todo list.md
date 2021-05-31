### 待办
- 布隆过滤器
- 为何未通过索引去查找数据不阻塞，通过索引去查找数据就阻塞了:[[MySQL · 引擎特性 · InnoDB 事务子系统介绍#InnoDB MVCC 实现#一个有趣的可见性问题]]
- Spring Cloud Gateway 请求没走filter直接走了controller

### 学习计划
#### 四月学习计划
- 数据库知识
	- [ ] sql 语句执行过程
	- [ ] 索引
	- [ ] undo redo log
	- [ ] 基础系统知识
	- [ ] sql优化
	- [ ] 索引条件下推
	- [ ] 数据库伪记录
	- [ ] Mysql Binlog格式 mixed/statement/row
	
### 每日清单

- 2021-04-05
	-  [x] 数据库MVCC与LBCC关系梳理:[参考博客](https://www.codenong.com/cs110441924/)
	-  [x] 聚簇索引,非聚簇索引区别:[参考博客](https://www.cnblogs.com/jiawen010/p/11805241.html)

- 2021-04-06
	- [x] 配合前端优化老资源数据判定
	- [x] foundation单测补充,包结构调整(BaseEntity,consistents等类移到dto/consistent包下)
	- [x] Obsidian采用git作为数据同步工具
	- [x] 资管状态数据拉取接口问题 
		/open/pdm/unit/syncInterface/fileStatus，多个调改材料ID只查出一条数据，但是在资管页面可以看到多条数据:资管是将多条数据一起返回，可以通过调改材料ID区分

-  2021-04-07
	- [x] Mysql的半一致性读
		- [基于源码角度解析MySQL半一致性读原理](https://www.yisu.com/zixun/30489.html)
		- [MySQL介于普通读和加锁读之间的读取方式：semi-consistent read](https://juejin.cn/post/6844904022499917838)

- 2021-04-13
	-  [x] Mysql的[[ReadView]]
	-  [x] [超全面sql语句加锁分析上](https://mp.weixin.qq.com/s/wSlNZcQkax-2KZCNEHOYLA)
	-  [x] 字段类型不一致会导致数据库的索引失效,如当数据库字段类型定义为varchar,where条件为 field = value 而非 'value'时，就会导致索引失效

- 2021-04-14
	- [x] [超全面sql语句加锁分析中](https://mp.weixin.qq.com/s/wSlNZcQkax-2KZCNEHOYLA)
	- [x] 全表查询删除数据时会对[[超全面Mysql语句加锁分析#全表扫描的情况|所有二级索引加锁]]情况复现

- 2021-05-04
	- [x] [超全面sql语句加锁分析下](https://mp.weixin.qq.com/s/9WWBXLNoUcTkS4DJnM5ViA)
	- [x] sql加锁分析区分为Mysql8.0及Mysql5.7实验
- 2021-05-05
	- [x] 历史数据库笔记复习
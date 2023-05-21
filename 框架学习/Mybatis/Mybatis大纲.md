#知识大纲 #Mybatis

# 1. Mybatis 大纲

- Mybatis 分页
	1. Mybatis 基于 RowBounds 分页，是内存分页
	2. 手动写 limit 分页
	3. 基于分页插件，如 PageHelper
- 执行结果封装
	1. 基于标签定义映射关系
	2. 基于 sql 别名映射
- Mybatis 的缓存机制
	1. 一级缓存：基于 Session 的 LocalCache
	2. 二级缓存：多 Session 共享的缓存

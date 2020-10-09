# Redis深度历险 核心原理与应用实践

## 基础和应用篇

### 授人以鱼不如授人以渔

- 由Redis面试想到的

  Redis能用来做什么？
  缓存。
  
  还有呢？
  分布式锁。

- Redis可以做什么

	- 记录帖子的点赞数、评论数和点击数（hash）
	- 记录用户帖子ID列表（排序），便于快速显示用户的帖子列表（zset）
	- 记录帖子的标题、摘要、作者和封面信息，用于列表也展示（hash）
	- 记录帖子的点赞用户ID列表，评论ID列表，用于显示和去重计数（zset）
	- 缓存近期热帖内容（帖子内容的空间占用比较大），减少数据库压力（hash）
	- 记录帖子内容的相关文章ID，根据内容推荐相关帖子（list）
	- 如果帖子ID是整数自增的，可以使用Redis来分配帖子ID（计数器）
	- 收藏集和帖子之间的关系（zset）
	- 记录热榜帖子ID列表、总榜和分类热榜（zset）
	- 缓存用户行为历史，过滤恶意行为（zset、hash）

- Redis作者介绍

	- Redis由意大利人Salvatore Sanfilippo（网名Antirez）开发
	- Antirez出生在非英语系国家，英语能力长期以来是一个短板，他曾经专门为自己蹩脚的英语能力写过一篇博文《英语伤痛15年》，用自己的成长经历来鼓励那些非英语系的技术开发者们努力攻克英语难关。
	- 6379是手机键盘字母 MERZ 的位置
	- MERZ 在Antirez的朋友圈语言中是“愚蠢”的代名词

### 万丈高楼平地起-Redis基础数据结构

- 5种基础数据结构
- 容器型数据结构的通用规则
- 思考&作业

### 千帆竞发-分布式锁

- 分布式锁的奥义
- 超时问题
- 可重入性
- 思考&作业

### 缓兵之计-延时队列

- 异步消息队列
- 队列空了怎么办
- 阻塞读
- 空闲连接自动断开
- 锁冲突处理
- 延时队列的实现
- 进一步优化
- 思考&作业

### 节衣缩食-位图

- 基本用法
- 统计和查找
- 魔术指令bitfield
- 思考&作业

### 四两拨千斤-HyperLogLog

- 使用方法
- pfadd中的pf是什么意思
- pfmerge适合的场合
- 注意事项
- HyperLogLog实现原理
- pf的内存占用为什么是12KB
- 思考&作业

### 层峦叠嶂-布隆过滤器

- 布隆过滤器是什么
- Redis中的布隆过滤器
- 布隆过滤器的基本用法
- 注意事项
- 布隆过滤器的原理
- 空间占用估计
- 实际元素超出时，误判率会怎样变化
- 用不上Redis4.0怎么办
- 布隆过滤器的其他应用

### 断尾求生-简单限流

除了流量控制，限流还由一个应用目的是控制用户行为，避免垃圾请求。

比如在UGC社区，用户的发帖、回复、点赞等行为都要严格控制，一般严格限定某行为在规定时间内被允许的次数，超过了次数就是非法行为。

- 如何使用Redis来实现简单限流策略
- 解决方案

	- zset + 滑动窗口

	  ```python
	  # coding: utf8
	  
	  import time
	  import redis
	  
	  client = redis.StrictRedis()
	  
	  def is_action_allowed(user_id, acction_key, period, max_count):
	  	key = 'hist:%s:%s' % (user_id, action_key)
	  	now_ts = int(time.time() * 1000)
	  	with client.pipeline() as pipe:
	  		# value 和 score 都使用毫秒时间戳
	  		pipe.zadd(key, now_ts, now_ts)
	  		# 移除时间窗口之前的行为记录，剩下的都是时间窗口内的
	  		pipe.zremrangebyscore(key, 0, now_ts - 	period * 1000)
	  		# 获取窗口内的行为数量
	  		pipe.zcard(key)
	  		# 设置过期时间，避免冷用户持续占用内存
	  		pipe.expire(key, period + 1)
	  		# 批量执行
	  		_, _, current_count, _ = pipe.execute()
	  	# 比较数量是否超标
	  	return max_count >= current_count
	  
	  ```

- 小结

	- 优点：简单、方便、易实现
	- 缺点：限定60s内操作不得超过100万次，会占用大量内存，显然不合适

### 一毛不拔-漏斗限流

- Redis-Cell
- 思考&作业
- Redis-Cell作者介绍

### 近水楼台-GeoHash

- 用数据库来算附近的人
- GeoHash算法
- Geo指令的基本用法
- 注意事项

### 大海捞针-scan

- scan的基本用法
- 字典的结构
- scan的遍历顺序
- 字典扩容
- 对比扩容、缩容前后的遍历顺序
- 渐进式rehash
- 更多的scan指令
- 大key扫描

## 原理篇

## 集群篇

## 拓展篇

## 源码篇

*XMind - Trial Version*
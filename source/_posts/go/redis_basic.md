---
title: Redis基础知识 
categories: 
- GolangStudy
---

## Redis简介

* Redis(Remote Dictionary Server)是用C语言开发的一个开源的高性能键值对(key-value)内存数据库
* NoSQL: 即Not-Only SQL(泛指非关系型数据库)，作为关系型数据库的补充
* 特征
	- 数据间没有必然的关联关系
	- 内部采用单线程机制进行工作
	- 高性能
	- 多种数据类型支持
		* 字符串型 string
		* 列表类型 list
		* 散列类型 hash
		* 集合类型 set
		* 有序集合类型 sorted_set
	- 持久化支持
* 应用
	* 为热点数据加速查询
	* 任务队列, 如秒杀、抢购、排队购票
	* 即时信息查询, 如各位排行榜、各类网站访问统计、公交到站信息、在线人数信息
	* 时效性信息控制,如验证码控制、投票控制
	* 分布式数据共享
	* 消息队列
	* 分布式锁

## 启动 

* 服务端启动方式

``` redis
redis-server 
redis-server --port 6380
redis-server conf/redis-6379.conf
```

* 客户端连接

``` redis
redis-cli 
redis-cli -p 6380
redis-cli -h 127.0.0.1 -p 6380
```



## Redis数据类型

* Redis数据都是Map，采用key:value的形式存储,其中key永远是string，value可以是以下的几种类型之一

### string

* 存储单个数据
* 如果字符串以整数的形式展示，可以作为数字操作使用
* 基本操作
	- 添加/修改数据
		``` redis 
		# 添加/修改单条数据
		set key val
		# 添加/修改多条数据
		mset key1 val1 key2 val2
		```
	- 获取数据  
		``` redis
		# 获取单条数据
		get key
		# 获取多条数据
		mget key1 key2
		``` 
	- 删除数据
		``` redis
		del key
		```
	- 获取数据字符个数  
		``` redis
		strlen key
		```
	- 追加信息到原始信息后部  
		``` redis
		append key value
		```
* 扩展操作
	- redis用于控制数据表主键id自增
		``` redis
		incr key
		decr key
		incrby key increment
		decrby key incrment
		incrbyfloat key increment 
		```

	- 设置数据具有指定生命周期, 适用于所有具有时效性限定控制的操作(投票每4小时只能投一票)
		``` redis
		setex key seconds value
		psetex key milliseconds value

		# 示例:
		setex 13879631902 60 1
		```

* 格式化存储
	* 将所有的信息存放在表中，表名:主键名:主键值:字段名 value
	    ``` redis
	    set user:id:00789:fans 1242526
	    set user:id:00789:blogs 789
	    ```
	* 用json格式存储信息
		``` redis
		user:id:2506 '{"name"":春晚, "fans": 6143, "focuns"":83}''
		```

### hash

* 对一系列存储的数据进行编组，方便管理，典型应用存储对象信息
* 需要的存储结构: 一个存储空间保存多个键值对数据
* 基本操作
	- 添加/修改数据  
		``` redis
		# 添加/修改单个数据
		hset key filed1 value1 filed2 value2  
		# 添加/修改多个数据
		hmset key1 filed1 value1 filed2 value2 key2 filed1 value1  
		```
	- 获取数据   
		```  redis
		# 获取key下面的一个字段值
		hget key filed    
		# 获取key下面的所有字段值
		hgetall key
		# 获取key下面的多个字段值
		hmget key filed1 field2
		```
	- 删除数据   
		``` redis
		hdel key field
		```
	- 获取哈希表中字段数量  
		``` redis
		hlen key
		```
	- 获取哈希表中是否存在指定的字段  
		``` redis
		hexists key field
		```
	- 添加之前会检查是否有value值，有的话set不成功  
		``` redis
		hsetnx key fidld value
		```

* 扩展操作
	* 获取哈希表中所有的字段名或者字段值 
		``` redis
		# 获取字段名
		hkeys key    
		# 获取字段值
		hvals key
		```
	* 设置指定字段的数值数据增加指定范围的值 
		``` redis
		hincrby key field increment  
		hincrbyfloat key field incrment
		```

* 注意事项
	* hash类型下的value只能存储字符串，不允许存储其他类型的数据类型
	* hash类型十分贴近对象的数据存储形式，并且可以灵活添加删除对象属性
	* hgetall操作可以获取全部属性，如果内部field过多，遍历整体数据效率会很低，有可能成为数据访问瓶颈

* 应用场景
	- 电商网站购物车设计与实现
		* 客户id作为key，每位客户创建一个hash存储结构存储对应的购物车信息
		* 将商品编号作为field，购买数量作为value进行存储
		* 添加商品: 追加全新的fileld与value
		* 浏览: 遍历hash
		* 更改数量: 自增/自减, 设置value值
		* 删除商品: 删除field
		* 清空: 删除key

		``` redis
		hset user:id:001 g01 300 g02 500
		hset user:id:001 g03 30
		hincrby user:id:001 g03 2
		hlen user:id:001
		hdel user:id:001 g02
		```

	- 改进
		* 可以将购物车中的每条商品信息保存成多条filed
			* eg: 商品id:nums 用于保存购买的数量
				商品id:info 用于保存对应商品的信息
			``` redis
			hset 001 g01:nums 100 g01:info '{"name":"爽歪歪", "price":100}'
			```
			* 商品信息可以存储为独立的hash
	- 抢购
		* 以商家id作为key
		* 将参与抢购的商品id作为field
		* 将参与抢购的商品数量作为对应的value
		``` redis
		hset p01 c30 100 c50 100 c100 100
		hincrby p01 c30 -10
		```

### list

* 存储多个数据，并对数据进入存储空间的顺序进行区分
* 基本操作
	- 添加/修改数据  
		``` redis
		lpush key value1 value2.... (从左边push)
		rpush key value1 value2.... (从右边push)
		```
	- 获取数据
		``` redis
		lrange key start stop  (-1表示最后一个元素,-2表示倒数第二个元素)
		lindex key index
		llen key
		```
	- 获取并移除数据
		``` redis
		lpop key
		rpop key
		```
	- 规定时间内获取并移除数据(lpop和rpop的阻塞版本(b是blocking的缩写))(任务队列)
		``` redis
		blpop key1 key2 ... timeout
		brpop key2 key2 ... timeout
		```
	- 移除指定数据
		``` redis
		lrem key count value
		```

* list扩展操作
	- 朋友圈点赞，要求按照点赞顺序显示点赞好友的信息
	- list可以对数据进行分页操作，通常第一页信息来自于list，第二页及更多的信息通过数据库的形式加载
	- list用于展示最新的消息(最新关注的公众号，最新的日志)
	
### set

* 存储大量的数据，在查询方面提供更高的效率
* 存储结构和hash完全相同，仅存储键，不存储值(nil),并且键是不允许重复的
* 基本操作
	- 添加数据
		``` redis
		sadd key member1 member2 ...
		```
	- 获取全部数据
		``` redis
		smembers key
		```
	- 删除数据
		``` redis
		srem key member1 member2 ....
		```
	- 获取集合数据总量
		``` redis
		scard key
		```
	- 判断集合中是否包含指定数据
		``` redis
		sismember key member
		```
	- 随机获取集合中指定数量的数据
		``` redis
		srandmember  key count
		```
	- 随机获取集合中的某个数据并将该数据移出集合
		``` redis
		spop key
		```
	- 求两个集合的交集
		``` redis
		sinter key1 key2 
		```
	- 求两个集合的并集
		``` redis
		sunion key1 key2
		```
	- 求两个集合的差集
		``` redis
		sdiff key1 key2
		```

	- 求两个集合的交、并、差集并存储到指定集合中
		``` redis
		sinterstore destination key1 key2
		sunionstore destination key1 key2
		sdiffstore destination key1 key2
		```

	- 将指定数据从原始集合移动到目标集合中
		``` redis
		smove source destination member
		```
* 扩展操作
	- 应用于随机推荐类信息检索，例如热点歌单推荐、购买旅游线路
		* 系统分析出各个分类的最新最热点信息条目并组织成set集合
		* 随机挑选出其中部分信息
		* 配合用户关注信息分类中的热点信息组织成展示的全信息集合

	- 应用于同类信息的关联检索，二度关联搜索、深度关联搜索
	
		* 显示共同关注
		* 显示共同好友

	- 权限校验
		* 依赖set集合数据不重复的特征，依赖set集合hash存储结构特征完成数据过滤和快速查询
		* 根据用户id获取用户所有角色
		* 根据用户所有角色获取用户所有权限放入到set集合中
		* 根据用户所有角色获取用户所有数据全选放入set集合中

		sadd rid:001 getall getById
		sadd rid:002 getall getCount insert
		sunionstore uid:007 rid:001 rid:002
		smembers uid:007

	- 统计网站的访问量，PV(访问量) 、UV(独立访客)、IP(独立IP)
		* PV: 网站被访问次数，可通过刷新页面提高访问量
		* UV: 网站被不同用户访问的次数，可通过cookie统计访问量，相同用户切换IP地址，UV不变
		* IP: 网站被不同IP地址访问的总次数，可通过IP地址统计访问量，相同IP不同用户访问IP不变

		* 通过set的特点做重复性过滤
		* 建立set模型，记录不同cookie数量(UV)
		* 建立set模型，记录不同IP数量

	- 黑白名单

* 注意事项
	* set中数据不允许重复
	* set虽然与hash中的存储结构相同，但是无法启用hash中的存储值的空间

### sort_set

* 数据排序有利于数据展示，需要提供有一种可以根据自身特征进行排序的方式
* 在set的存储结构基础上添加可排序字段
* 基本操作
	- 添加数据
		``` redis
		zadd key score1 member1 score2 member2
		```
	- 获取全部数据
		``` redis
		zrange key start stop [withscores]      (从小到大的顺序)
		zrevrange key start stop [withscores]   (从大到小的顺序)
		```
	- 删除数据
		``` redis
		zrem key member 
		```
	- 按条件获取数据
		``` redis
		zrangebyscore key min max [withscores] [limit]
		zrevrangebyscore key max min [withscores]
		```
	- 条件删除数据
		``` redis
		zremrangebyrank key start stop
		zremrangebyscore key min max
		```
	- 获取集合数据总量
		``` redis
		zcard key
		zcount key min max
		```
	- 集合交、并操作
		``` redis
		zinterstore destination numkeys key1 key2 ...
		zunionstore destination numkeys key1 key2 ...
		```
	- 获取数据对应的索引(排名)
		``` redis
		zrank key member
		zrevrank key member
		```
	- score值获取与修改
		``` redis
		zscore key member
		zincrby key increment member
		```

* 注意点
	- min与max用于限定搜索查询的条件
	- start和stop用于限定查询范围，作用于索引，表示开始和结束索引
	- offset与count用于限定查询范围，作用于查询结果，表示开始位置和数据总量

* 扩展操作
	- 排序类问题
		* 十大杰出青年
		* 各类资源类网站TOP10
		* 聊天室活跃度统计
		* 游戏好友亲密度

	- 时效性控制,用于定时任务执行顺序管理或任务过期管理
		* 对于基于时间线限定的任务处理，将处理时间记录为score值，利用排序功能区分处理的先后顺序
		* 记录下一个要处理的时间，当到期后处理对应任务，移除redis中的记录，并记录下一个要处理的时间
		* 当新任务加入时，判断并跟新当前下一个要处理的任务时间
		* 为提升sort_set的性能通常将任务根据特征存储成若干个sorted_set
	``` redis
	zadd tasks 20210613 task1 20130620 task2 20130626 task3
	zrange tasks 0 -1 withscores
	zremrangbyrank tasks 0 0
	```

	* 对于带有权重的任务，优先处理权重高的任务，采用score记录权重即可
* 总结
	* 带有生命周期的计数器(string)

## 通用命令

* key 

	- key是一个字符串，通过key获取redis中保存的数据
	- key的操作
		* 与key自身状态相关的操作
			* 删除指定key 		`del keyi`
			* 获取key是否存在 	`exists key`
			* 获取key的类型 	`type key`
		* 与key有效性控制相关操作 
			* 设置有效期
				``` redis
				expire key seconds
				pexpire key milliseconds
				expireat key timestamp
				pexpireat key milliseconds-timestamp
				```
			* 获取key的有效时间
				``` redis
				ttl key
				pttl key
				```
			* 切换key从时效性到永久性
				``` redis
				persist key
				```
		* key查询操作 
			* 查询key `keys pattern`
		* key的改名操作
			``` redis
			rename key newkey
			renamenx key newkey
			```
		* 对key排序
			``` redis
			sort
			```
		* 其它key通用操作
			``` redis
			help @generic
			```
* 数据库

	- redis为每个服务提供16个数据库, 编号0-15
	- 每个数据库之间相互独立

	- 切换数据库  `select index`
	- 其它操作
		* quit
		* ping
		* echo message
	- 数据移动 	move key db
	- 数据清除操作
		* dbsize
		* flushdb
		* flushall


## 持久化

* 利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复的工作机制称为持久化
* 持久化可以防止数据的丢失，确保数据安全性

* 持久化方案
	- RDB
		save 	保存数据
		bgsave  后台保存数据，可能不会立即执行
		通过更改配置文件自动RDB:  `save second changes`  满足限定时间范围内key变化的数量这一条件就进行持久化
		redis内部涉及RDB的操作都采用bgsave方式，save命令基本可以放弃使用了
		
		* RDB优点
			* RDB是一个紧凑压缩的二进制文件，存储效率高
			* RDB内部存储的是redis在某个时间点的数据快照，非常适合数据备份，全量复制等场景
			* RDB恢复数据的速度要比AOF快很多
			* RDB通常用于灾难恢复
		* RDB缺点
			* RDB方式无法做到实时持久化，具有较大的可能性丢失数据
			* bgsave指令每次运行都要fork操作创建子进程，会牺牲掉一些性能
			* Redis的众多版本中未进行RDB文件格式的版本统一
	- AOF
		* 以独立日志的方式记录每次写的命令，重启时再重新执行AOF文件中命令达到数据恢复的目的
		* 实现实时性的数据持久化

		* AOF写数据的三种策略
			* always 	每次写入操作均同步到AOF文件中，数据零失误，性能较低
			* everysec  每秒将缓冲区内的指令同步到AOF文件中，数据准确性较高，性能较高(默认配置)
			* no 		由操作系统控制每次同步到AOF文件的周期，整体过程不可控

		* AOF配置
			* appendonly yes 			开启AOF持久化功能，默认为不开启状态
			* appendfsync erverysec  	AOF写文件策略(默认everysec)
			* appendfilename filename 	配置AOF持久化文件的名字
		* AOF重写
			* AOF重写就是对同一数据的若干条命令执行结果转换为最终的结果数据对应的指令进行记录

			* 作用
				* 降低磁盘占用量，提高磁盘利用率
				* 提高持久化效率，降低持久化写时间，提高IO性能
				* 降低数据恢复用时，提高数据恢复效率

			* 重写方式
				* 手动重写      bgrewriteaof
				* 自动重写 		
					* auto-aof-rewrite-min-size size  
					* auto-aof-rewrite-percentage percentage

## 事务

* redis事务就是一个命令执行的队列，将一系列预定义命令包装成一个整体(一个队列), 当执行时，一次性按照添加顺序依次执行，中间不会被打断
* 基本操作
	* 开启事务 	multi   (设置指令开启位置，此指令后续的所有指令均加入到事务中)
	* 执行事务  exec 	(设置事务的结束位置，同时执行事务， 与multi成对出现)
	* 取消事务  discard (终止当前事务的定义，发生在multi之后，exec之前)
	* 注意: 加入到事务的命令暂时进入到任务队列中， 并没有立即执行，只有执行了exec命令才开始执行
	* 注意: 在添加事务的过程中如果出现语法错误，整体事务中的所有命令均不会执行
	* 注意: 在添加事务的过程中如果出现无法执行的操作，事务中能正确运行的指令会执行，错误的命令不会被执行
	* 注意: 已经执行完毕的命令对应的数据不会自动回滚，需要在代码中实现回滚

## 锁

* 监视锁
	- 添加监视锁 	watch key1 [key2...]   (在执行exec之前如果watch的key发生了变化，事务会被终止执行)
	- 取消监视锁    unwatch
	``` redis
	# 客户端1
	set name 123
	watch name
	multi
	set aaa bb
	get aaa
	exec

	# 客户端2 (客户端2在客户端1执行exec之前修改了被监视的name值，事务会被取消执行)
	set name 234
	```

* 分布式锁
	- 设置分布式锁		setnx lock-key value  (setnx在key有值时设置失败，无值时设置成功)
	- 取消锁			del lock-key
	- 设置锁的超时时间 expire lock-key second
	```
	set num 123

	# 修改num值之前先获取锁
	setnx lock-num 1
	# 给锁设置超时时间
	expire lock-num 20

	set num 234
	# 取消锁
	del lock-num
	```

## 删除策略

* 过期数据
	* Redis是一种内存级数据库，所有的数据存放在内存中，内存中的数据可以通过TTL指令获取其状态
		* TTL为正数: 具有时效性的数据
		* TTL = -1 : 永久有效的数据
		* TTL = -2 : 已经过期的数据 或者 被删除的数据 或者 未定义的数据
* 数据删除策略(在内存占用和CPU占用达到平衡)
	* 定时删除
		* 创建一个定时器，当key设置有过期时间时，当到达过期时间执行对键的删除操作
		* 优点: 节约内存，快速释放掉不必要的内存占用
		* 缺点: CPU压力很大，可能会影响redis服务器响应时间和指令吞吐量
	* 惰性删除
		* 数据到达过期时间时， 不作处理，等下次访问该数据时
			* 如果未过期，返回数据
			* 如果过期了，删除数据，返回不存在
		* 优点: 节约CPU性能
		* 缺点: 内存压力很大,可能有些无用内存会长期占用
	* 定期删除
		* 周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度
		* 周期性抽查存储空间(随机抽查 重点抽查)
* 逐出算法
	* redis添加数据的时候如果内存不满足要求，redis会临时删除一些数据为当前指令清理存储空间, 清理数据的策略称为逐出算法

	* 数据逐出的相关配置
		* 最大可使用内存			maxmemory  (占用物理内存的比例， 默认为0，表示不限制, 通常设置在50%以上)
		* 每次选取待删除数据个数 	maxmemory-samples
		* 删除策略 					maxmemory-policy (达到最大内存后对挑出的数据进行删除的策略)
			* 检测易失数据(可能会过期的数据集)
				* volatile-lru 		挑选最近没有使用的数据淘汰
				* volatile-lfu  	挑选最近使用次数最少的数据淘汰
				* volatile-ttl 		挑选将要过期的数据淘汰
				* volatile-random 	任意选择数据淘汰
			* 检测全库数据
				* allkeys-lru		挑选最近没有使用的数据淘汰
				* allkeys-lfu		挑选使用次数最少的数据淘汰
				* allkeys-random 	任意选择数据淘汰
			* 不删除
				* noeviction

## 高级数据类型

* Bitmaps
	* 获取指定key对应偏移量上的bit值 	getbit key offset
	* 设置指定key对应偏移量上的bit值    setbit key offset value
	* 对指定key按位进行并、交、非、异或操作并将结果保存到destKey中
		bitop op desKey key1 [key2...]
			* and 	交
			* or 	并
			* not   非
			* xor   异或
	* 统计key中1的数量
		bitcount key [start end]
* HyperLogLog
	* 统计不重复数据的数量
	* 基数: 数据集去重后的元素个数
	* HyperLogLog是用来做基数统计的

	* 基本操作
		* 添加数据 	pfadd key element [element...]
		* 统计数据  pfcount key [key...]
		* 合并数据  pfmerge destkey sourcekey [sourcekey...]
			
* GEO
	* 添加坐标点
		geoadd key longitude latitude member [longitude latitude member ...]
	* 获取坐标点
		geopos key member [member...]
	* 计算坐标点的记录
		geodist key member1 member2 [unit]
	* 根据坐标求范围内的数据
		georadius key longitude latitude radius 
	* 根据点求范围内的数据
		georadiusbymember key member radius
	* 获取指定点对应的坐标hash值
		geohash key member

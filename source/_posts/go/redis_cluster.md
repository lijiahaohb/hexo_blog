---
title: Redis集群 
categories: 
- GolangStudy
---
# Redis集群

## redis主从复制

* 为了避免单点redis服务器故障，准备多台服务器，将数据复制到多个副本保存在不同的服务器上，连接在一起，并且保证数据同步, 实现高可用, 并且实现数据冗余备份

* 多台服务器连接方案
	- master(主服务器) 收集数据，负责写数据，并将数据同步到slave结点
	- slave(从服务器)  从master接收数据并提供给用户(用户读数据从slave结点读)
	- 注意: 需要解决数据同步的问题，及将master的数据复制到slave中

* 主从复制就是将master中的数据即时、有效地复制到slave中(一个master拥有多个slave， 一个slave只能对应一个master)

* 职责分配
	- master
		* 写数据
		* 执行写操作时，将出现变化的数据自动同步到slave中
		* 读数据(可忽略)
	- slave
		* 读数据
		* 禁止写数据

* 主从复制的作用
	- 读写分离: master写、slave读、提高服务器的读写负载能力
	- 负载均衡: 基于主从结构，配合读写分离，由slave分担master负载，并根据需求变化，改变slave的数量
		，通过从多个结点分担数据读取负载，大大提高redis服务器并发量和数据吞吐量
	- 故障恢复: mater出问题时， 由slave提供服务，实现快速故障恢复
	- 数据冗余: 实现数据热备份，是持久化之外的一种数据冗余方式
	- 高可用基石: 基于主从复制，构建哨兵模式和集群，实现redis的高可用方案

* 主从复制工作流程
	- 建立连接阶段(slave连接master)
		* 建立slave到master的连接，使得master能够识别slave，并保存slave的端口号
		* 连接建立工作流程
			1、slave发送指令 : slaveof ip port   (将一台服务器变成另一台服务器的从服务器)
			2、master收到指令后，响应对方
			3、保存master的IP和端口:   masterhost  masterport
			4、根据保存的信息创建连接master的socket
			5、slave周期性的发送ping
			6、master回复pong
			7、slave发送身份校验指令: auth password
			8、master验证授权
			9、slave发送指令: replconflistening-port <port-number>
			10、master保存slave的端口号

		* 建立连接的方式
			1、客户端发送命令
				slaveof <masterip> <masterport>
			2、启动服务器参数
				redis-server --slaveof <masterip> <masterport>
			3、服务器配置(写在配置文件中)
				slaveof <masterip> <masterport>
		* 断开连接的方式
			slaveof no one
	- 数据同步阶段
		1. slave发送指令 `psync2 ? -1` (? 代表runid -1 代表offset)
		2. master执行bgsave生成RDB文件，并记录offset
		3. 发送 +FULLRESYNC runid offset
		   并将RDB文件通过socket发送给slave
		4. slave 保存runid和offset，并接收RDB，清空数据，执行RDB文件恢复过程(全量复制)
		5. 发送命令 psync2 runid offset
		6. 接收命令，判断runid是否匹配，并且offset是否在复制缓冲区中
		7. 如果runid不匹配或者offset不在复制缓冲区中，再次执行全量复制
		8. 如果runid匹配offset也在复制缓冲区中，检查两者保存的offset是否匹配，如果匹配说明是同步的
		9. 如果不匹配则引发增量复制，发送 +CONTINUE offset
		9. 保存offset, 并执行rewriteaof恢复数据

		* 请求同步数据 
		* 创建RDB同步数据
		* 恢复RDB同步数据
		* 请求部分同步数据
		* 恢复部分同步数据

		* 注意: master端复制缓冲区大小设置不合理可能会导致数据溢出，可以更改: repl-backlog-size 

	- 命令传播阶段
		* 命令传播阶段需要部分复制
		* 部分复制的三个核心要素
			- 服务器的运行id(run id)
				* 服务器运行ID是每一台服务器每次运行的身份识别码，一台服务器多次运行可以生成多个运行ID
				* 运行ID是40位16进制字符
				* 运行ID被用于在服务器直接识别身份
				* 运行ID在每台服务器启动时自动生成，master在首次连接slave时，会将自己的运行ID发送给slave，slave保存此ID
			
			- 主服务器的复制积压缓冲区
				* 复制缓冲区用于存储服务器执行过的命令，每次传播命令，master都会将传播的命令个记录下来，并存储在复制缓冲区
				* 复制缓冲区组成
					* 偏移量
					* 字节值
					* 通过offset区分不同的slave当前数据传输的差异
					* master记录已发送的信息对应的offset
					* slave记录已接受的信息对应的offset
			- 主从服务器的复制偏移量

		* 心跳机制
			- 命令传播阶段，master和slave需要进行信息交换，使用心跳机制进行维护，需要双方连接保持在线
			- master心跳:
				* 指令: PING
				* 周期: repl-ping-slave-period(默认10秒)
				* 作用: 判断slave是否在线
				* 查询: INFO replication
			- slave心跳:
				* 指令: REPLICONF ACK offset
				* 周期: 1s
				* 作用:
					* 判断master是否在线
					* 汇报slave自身的复制偏移量，获取最新的数据变更指令

## 哨兵模式

* 哨兵是一个分布式系统，用于对主从结构中的每台服务器进行监控，当出现故障时通过投票机制选择新的master并将所有的slave连接到master
* 哨兵的作用
	- 监控master和slave是否正常运行
		* 用于同步各个节点的状态信息: 包括sentinel、master和slave的状态
		* 几个sentinel内部发布订阅一个sentinel得到的master和slave的状态信息
	- 通知(提醒) : 当被监控的服务器出现问题时，向其他(哨兵、客户端)发送通知
	- 自动故障转移 : 断开master和slave连接，选取一个slave作为master，将其他slave连接到新的master上，并通知客户端新的服务器地址
		* master的状态: S_DOWN(主观下线) O_DOWN(客观下线)

		* 服务器列表中选择备选的master的原则
			* 在线
			* 响应速度快
			* 优先原则(offset runid)
* 哨兵的配置
	* 配置一拖二的主从结构
	* 配置三个哨兵(配置相同,端口号不同) sentinel.conf
	* 启动哨兵:   redis-sentinel sentinel-端口号.conf


## 集群

* redis提供的服务QPS可以达到10万/秒，当处理速度不够用或者内存不够用的时候考虑集群方案

* 集群的作用
	* 分散单机服务器的访问压力，实现负载均衡
	* 分散单台服务器的存储压力，实现可扩展性
	* 降低单台服务器宕机带来的业务灾难

* Redis集群结构设计
	* 数据存储设计
		* 通过算法设计，计算出key应该保存的位置
		* 将所有的存储空间分为16384份，每台主机保存一部分存储空间
		* 槽用于区分数据的存储空间
		* 增强可扩展性
	* 集群内部的通讯设计
		* 各个数据库相互通信，保存每个库中槽的编号数据
		* 一次命中直接返回
		* 一次未命中，告知具体位置

* cluster配置
	* cluster-enabled yes   成为cluster结点
	* cluster-config-file  nodes-6379.conf
	* cluster-node-timeout 10000
	* cluster-migration-barrier <count>  master连接的slave的数量
* cluster结点操作指令
	* cluster nodes     			  查看集群结点信息
	* cluster replicate <master-id>   进入一个从节点redis，切换到其主节点
	* cluster meet ip:port            发现一个新节点，新增主节点
	* cluster forget<id>              忽略一个没有slot的结点
	* cluster failover 				  手动故障转移

	* redis-trib.rb create --replicas 1(连接master的slave个数) 127.0.0.1:6379(master结点的ip:port)
	* redis-cli -c (客户端连接的时候需要加上-c参数)

# 企业级解决方案

* 缓存预热
	- 缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统，避免用户请求的时候，先查询数据库，然后再将数据缓存的情况，用户直接查询事先预热的缓存数据
	- 解决方案
		* 前期准备工作
			* 日常例行统计数据访问记录，统计访问频度较高的热点数据
			* 利用LRU数据删除策略，构建数据留存队列
		* 准备工作
			* 将统计结果中的数据分类，根据级别，redis优先加载级别高的热点数据
			* 利用分布式多服务器同时进行数据读取，加速数据加载过程
		* 实施
			* 使用脚本固定出发数据预热过程
			* 使用CDN(内容分发网络)
* 缓存雪崩
	- 缓存雪崩是由于短时间范围内大量的key集中过期导致的
	- 解决方案(道)
		* 更多的页面静态处理
		* 构建多级缓存架构
			Nginx缓存 + redis缓存 + ehcache缓存
		* 检查Mysql严重耗时业务进行优化
		* 灾难预警机制
			* 监控redis服务器性能指标
				* CPU占用和CPU使用率
				* 内存容量
				* 查询平均响应时间
				* 线程数
		* 限流、降级
	- 解决方案(术)
		* LRU和LFU切换(切换删除策略)
		* 数据有效期策略调整
			* 分类到期
			* 同类别下采用固定时间+随机值的形式，稀释集中到期的key的数量
		* 超热数据使用永久key
		* 定期维护
		* 加锁(慎用)
* 缓存击穿
	- 缓存击穿是由单个高热的key过期导致的
	- 解决方案
		* 预先设定
		* 监控访问量，对自然流量激增的数据延长过期时间或设置为永久key
		* 后台刷新数据
		* 二级缓存, 设置不同的失效时间，保证数据不被同时淘汰
		* 加分布式锁，防止被击穿(慎用)

* 缓存穿透
	- 缓存穿透是访问了不存在的数据，跳过了合法数据的redis缓存阶段，每次访问数据库，导致对数据库服务器造成压力
	- 解决方案
		* 白名单策略(使用布隆过滤器）
	- 实施监控
	- key加密

* 性能指标监控
	- 性能指标
		* latency     				Redis响应一个请求时间
		* instantaneous_os_per_sec  平均每秒处理请求总数
		* hit rate 					缓存命中率
	- 内存指标
		* used_memory 				已使用内存
		* mem_fragmentation_ratio 	内存碎片率
		* evicted_keys 				由于最大内存限制而被移除的key的数量
		* blocked_clients          	
	- 基本活动指标
	- 持久化指标
	- 错误指标
		* rejected_connenctions   			由于达到maxclient限制而被拒绝的连接数 
		* keyspace_misses 					key值查找失败的次数
		* master_link_down_since_seconds  	主从断开的持续时间
* 压测工具(benchmark)
	- redis-benchmark [-h] [-p] [-c] [-n <requests>] [-k]
	eg:
		* redis-benchmark    （默认开50个连接,10000次请求对应的性能）
		* redis-benchmark  -c 100 -n 5000       （-c 请求连接数 -n 请求次数） 
* 性能指标监控工具
	- monitor    打印服务器调试信息
	- slowlog    慢查询日志
	  * 命令
		slowlog [operator]
			* get:      获取慢查询日志
			* len：     获取慢查询日志条目数
			* reset: 	重置慢查询日志
	  * 配置
		* slowlog-log-slower-than 1000    设置慢查询的时间上限(单位: 微妙)
		* slow-max-len  100 			  设置慢查询命令对应的日志显示长度
			










			
			



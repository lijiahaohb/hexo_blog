---
title: Linux 命令行工具
categories: 
- tools
---

## Linux基础命令

### man
* 查询命令的详细信息
	``` bash
	man ${command}
	```
* 查询文件的详细信息
	``` bash
	man 5 ${filename}
	```

### 用户管理和密码管理
* useradd 
	``` bash
	useradd 用户名
	```

* userdel 
	``` bash
	userdel 用户名
	```

* passwd 
	``` bash
	# 修改当前用户密码
	passwd
	# 修改其他用户密码(需要是拥有root权限的用户)
	passwd 用户名
	```

### chmod 

``` bash
# 修改文件权限
chmod 750 filename

# 递归修改文件权限
chmod -R 750 dirname
```

### chown 

``` bash
# 修改归属人以及归属组
chown user:group filename

# 递归修改目录下的归属人以及归属组
chown -R user:group dirname
```

## 网络

### tcpdump

* <font color=red>note: tcpdump抓包时默认只抓每一个数据包前68字节</font>
* 常用选项
	- -i 指定网卡
	- -D 列出所有的网卡列表
	- -C 当使用 -w 写入文件时，限制文件的最大大小，超出时会新开一个文件
	- -s 指定数据报大小(指定为0的话会抓取全部的数据包)
	- -w 把抓到的数据包保存到一个文件中
	- -r 读取保存了抓到数据包的文件
	- -v 抓包时输出包的附加信息
	- -c 指定抓取数据包的个数
	- -A 显示抓取包的全部内容
	- -n 不要把地址转换为主机名(直接显示ip不要解析为域名)
	- -nn 不要转换协议和端口号
	- -t 不显示时间戳

* 过滤筛选
	- 网卡过滤
		``` bash
		# 指定抓取 eth1 网卡上的包
		tcpdump -i eth1

		# 指定抓取所有网卡上的包
		tcpdump -i any
		```

	- 主机过滤
		``` bash
		# 要获取指定 IP 的数据包，不管是作为源地址还是目的地址
		tcpdump host 192.168.3.7

		# 要指定 IP 地址是源地址或是目的地址
		tcpdump src 192.168.1.100
		tcpdump dst 192.168.1.100
		```
	
	- 网段过滤
		``` bash
		tcpdump -i eth1 net 192.168
		tcpdump -i eth1 src net 192.168
		tcpdump -i eth1 dst net 192.168
		```
	- 端口过滤
		``` bash
		tcpdump port 22
		tcpdump src port 22
		tcpdump dst port 22
		```
	- 端口范围过滤
		``` bash
		tcpdump portrange 22-125
		```
	- 协议过滤
		``` bash
		tcpdump -i eth1 arp
		tcpdump -i eth1 ip
		tcpdump -i eth1 tcp
		tcpdump -i eth1 udp
		tcpdump -i eth1 icmp
		```

* 具体案例
	``` bash
	tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

		- tcp:  ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
		- -i eth1 : 只抓经过接口eth1的包
		- -t : 不显示时间戳
		- -s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
		- -c 100 : 只抓取100个数据包
		- dst port ! 22 : 不抓取目标端口是22的数据包
		- src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
		- -w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析
	```

### netstat

``` bash
netstat -anp | grep 端口号
```

### nc

* 控制参数
	- -l  指定nc处于监听模式
	- -p  指定本机端口
	- -s  指定发送数据的源IP地址
	- -u  指定nc使用UDP协议，默认为TCP
	- -v  输出交互或出错信息，新手调试时尤为有用
	- -w  超时秒数，后面跟数字
	- -z  表示zero，表示扫描时不发送任何数据
* 实例
	- 监听入站连接
		``` bash
	    # 监听tcp
		nc -l 127.0.0.1 8080

		# 监听udp
		nc -l -u 1234
		```
	- 连接远程系统
		``` bash
		# 连接tcp
		nc 127.0.0.1 8080

		# 连接udp
		nc -u 127.0.0.1 8080

		# 设置连接超时(该选项只能用于客户端)
		nc -w 10 127.0.0.1 8080
		```
	- 使用nc作为代理
		``` bash
		# 发往到 8080 端口的连接会自动转发到 192.168.1.200 上的 80 端口
		ncat -l 8080 | ncat 192.168.1.200 80
		```
	- 使用nc拷贝文件
		``` bash
		# 接收端
		nc -l  8080 > file.txt

		# 发送端
		nc 192.168.1.100 8080 --send-only < data.txt
		```
	- 端口转发
		``` bash
		# 通过选项 -c  进行端口转发
		# 连接到 80 端口的连接都会转发到 8080 端口
		nc -u -l 80 -c 'nc -u -l 8080'
		```
	

### curl

### telnet

``` bash
# 判断主机是否开放某个端口
telnet 127.0.0.1 3389
```

## 进程

### ps

``` bash
# 查看内存占用较多的进程
ps aux | sort -k4nr | head -N

# head: -N可以指定显示的行数，默认显示10行
# ps: 
	- a 指代所有的进程
	- u 执行该进程的用户id
	- x 指代显示所有程序，不以终端来区分
# sort:
	- k 根据指定关键词排序
	- 4 表示按照第四列排序
	- n 表示按照数值排序
	- r 按照从大到小排序(排序默认从小到大)
# ps aux 的输出
	- %MEM 在第 4 个位置，-k4 按照内存占用排序
	- %CPU 在第三个位置，-k3 表示按照cpu占用率排序。
```

### perf

``` bash
sudo perf record -F 99 -p 13204 -g -- sleep 30
	
	* perf record 	表示记录
	* -F 			表示记录的频率
	* -p 			表示进程号
	* -g 			表示记录调用栈
	* --sleep 		表示持续记录30s
```
``` bash
sudo perf report -n --stdio
```

	
## 文本处理

### wc

* 该命令用来计算文件的字节数，字数或是行数
* -l 该选项用于显示行数
	``` grep
	wc -l test.txt
	```
* -c 该选项用于显示bytes数
* -w 只显示字数

### 正则表达式

#### 基础正则
* ^ 过滤以...开头的行
	``` bash
	grep '^lijiahao' test.txt
	grep '^li' test.txt
	```

* $ 过滤以...结尾的行
	``` bash
	grep 'lijihao$' test.txt
	grep 'm$' test.txt
	```

* ^$ 过滤空行
	``` bash
	grep -n '^$' test.txt
	```

* . 匹配任意一个字符
	- note: . 不会匹配空行
	- \. 转义字符 用于匹配'.'
	``` bash
	grep '\.$' test.txt
	```

* \* 前一个字符连续出现0次或者0次以上
	``` bash
	# note: 本条命令会导致不包含i的行也被过滤出来，因为可以i可以出现0次
	grep 'i*' test.txt
	```

* .* 表示匹配所有
	- * 匹配的贪婪性: 简单来说就是.*会尽可能多的匹配
	``` bash
	grep '.*o' test.txt
	```

* [] 表示匹配中括号中的任意一个字符
	- [abc]  匹配abc中任意一个
	- [a-z]  匹配a-z中的任意一个
	- [a-zA-Z0-9] 匹配大小写字母和数字中的任意一个
	- 中括号里面出现的内容会被去掉特殊含义
	``` bash
	grep 'li[abc]' test.txt
	```

* [^] 匹配括号中内容以外的内容
	- [^abc] 匹配abc以外的所有内容

#### 扩展正则

* <font color=red>note: 扩展正则匹配要使用egrep命令或者给grep命令添加-E选项</font>

* \+ 前一个字符连续出现1次或者1次以上
	``` bash
	egrep 'i+' test.txt

	# 匹配连续出现数字的行
	egrep '[0-9]+' test.txt
	```

* | 表示或者
	``` bash
	egrep 'li|lijiahao' test.txt
	```

* () 表示()中的内容表示一个整体
	- 可以在sed中表示反向引用
	``` bash
	egrep 'li(|jiahao)' test.txt
	```
* {} 表示连续出现
	- o{n,m} 前面的字符o连续出现次数在n-m之间
	- o{n} 前面的字符o连续出现n次
	``` bash
	egrep 'i{3,4}' test.txt
	egrep 'i{3}' test.txt

	# 匹配身份证号 前17位位数字，第18位为数字或者X
	egrep '[0-9]{17}[0-9X]' id.txt
	```
* ? 前一个字符出现了0次或者1次
	``` bash
	egrep 'go?d' test.txt
	```

### find
* -name 根据文件名查找
	``` bash
	find / -name filename
	```

* -type 根据类型进行查找
	``` bash
	# 查找类型为目录的文件
	find / -type d
	```


### grep
* grep主要作用是过滤
* -n 该选项用于显示行号
	``` bash
	grep -n 't[ae]st' test.txt
	grep -n '^$' test.txt
	```
* -rl
	``` bash
	# 查找dirname文件夹下所有包含abc内容的文件
	grep -rl abc dirname
	```
* -v 该选项用于反选
	``` bash
	# 不显示空行
	grep -vn '^$' test.txt

	# 不显示空行和带有井号的行
	egrep -v '^$|#' test.txt
	``` 
* -E 该选项用于支持扩展正则
	``` bash
	grep -E 'i+' test.txt
	```
* -o 该选项用于显示匹配过程
	``` bash
	grep -o 'i{2,3}' test.txt
	```
* -c 统计出现了多少行 类似于wc -l
	```
	ps -ef | grep -c sshd
	```
* -i 该选项用于忽略大小写
	``` bash
	# 过滤包含字母的行
	grep -i '[a-z]' test.txt
	```
* -w 精确匹配
	``` bash
	netstat -anp | grep -w ':80'
	```

### sed
* 主要作用是替换修改文件内容
* 命令格式:  sed 选项 '命令功能 修饰符' 参数
	``` bash
	# 将test.txt 中的一个内容替换为另一个内容
	# s  命令功能——替换  g 修饰符
	sed -r 's#olgboy#oldgirl#g' test.txt
	```
* 命令功能(增删改查)
	- s 	替换
		``` bash
		# 将[0-9] 替换为 空
		sed 's#[0-9]##g' test.txt

		# 如果不加 g 只会匹配每行中第一个复合的元素
		sed 's#[0-9]##' test.txt

		# 反向引用 (先保护起来，再使用)
		# 在 123456 的两边加上 <>
		echo 123456 | sed -r 's#(.*)#<\1>#'

		# 调整两个单词的顺序
		echo lijiahao_zhaozijin | sed -r 's#(^.*)_(.*)#\2_\1#' 

		# 通过反向引用来取出ens32网卡的IP地址 
		ip a s ens32 | sed -n '4p' | sed -r 's#(^.*t )(.*)(/.*$)#\2#g'
		ip a s ens32 | sed -rn '3s#(^.*t )(.*)(/.*$)#\2#gp'
		```
	- p 	显示
		``` bash
		# 查找某一行
		sed '1p' test.txt

		# 查找某个范围的行
		sed '1,5p' test.txt
		sed '1,$p' test.txt
		sed '$p' test.txt

		# 查找固定内容(会正则匹配//之间的内容)
		sed '/lijiahao/p' test.txt

		# ! 表示取反 表示不显示空行和带井号的行
		sed -nr '/^$|#/!p' /etc/ssh/sshd_config

		# 查找固定范围内的内容
		sed '/10:00/,/11:00/p' test.txt
		sed -n '/li/,/lijiahao/p' test.txt
		```
	- d 	删除
		``` bash
		# 删除某一行
		sed '3d' test.txt

		# 删除某个范围的行
		sed '2,3d' test.txt

		# 删除空行和带井号的行 
		sed -r '/^$|#/d' /etc/ssh/sshd_config
		```
	- cai 增加c/a/i
		``` bash
		# c 替换一行内容
		sed '3c lijihaohaobang' test.txt

		# a 在行后面一行添加内容
		sed '3a lijiahaohaohba' test.txt
		sed '$a lijiahao\nzhaozijin\nshengqiqi' test.txt

		# i 在行的前面加入内容
		sed '3i lijihaohaobang' test.txt
		```
* 选项
	- -n 取消默认输出
		``` bash
		# 显示第三行(带有默认输出，效果是第三行打印两遍)
		sed '3p' test.txt
		# 显示第三行(取消默认显示，效果是只会显示第三行)
		sed -n '3p' test.txt
		```
	- -r 支持扩展正则
		``` bash
		sed -nr '/[0-9]{3}/p' test.txt
		```
	- -i 表示写入(如果不添加-i选项，只会对文件内容进行操作，不会将操作后的结果写会文件)

### awk
* 主要作用是取列和统计操作
* 命令格式
	- 命令 选项 '条件{动作}'

* 执行流程分析
	- 读取文件前: BEGIN{print "name"}
	- 读取文件中: {print $2} 打印第二列 note: {}之前还可以加上过滤条件
	- 读取文件后: END{print "end of file"}
	``` bash
	awk -F, 'BEGIN{print "name"}{print $2}END{print "end of file"}' test.txt
	# 取出ens32网卡的IP地址 
	ip a s ens32 | awk -F "[ /]+" 'NR==3{print $3}' 
	```

* 取行
	- NR 	行号
		* NR==1 			取出某一行
		* NR>=1 && NR<=5 	取出1到5行的范围
		* /lijiahao/ 		取包含某个内容的行
		* /li/,/lijiahao/ 	取包含某两个内容之间的行
	``` bash
	# 取出第一行
	awk 'NR==1' test.txt

	# 取出第1到5行
	awk 'NR>=1 && NR<=5' test.txt
	```
* 取列
	- $数字 取出某一列
	- $0 	取出整行内容
	- NF 	每行有多少列
	- $NF 	最后一列
	- FS  	字段分隔符
	- OFS 	输出字段分隔符
	``` bash
	# 打印第二列内容
	awk '{print $2}' filename 

	# 打印最后一列内容
	awk '{print $NF}' filename 

	# -F: 指定分隔符为: 
	# 取出以:为分割的最后一列和第一列
	awk -F: '{print $1,$NF}' /etc/passwd | column -t
	```

* 选项
	- -F 	指定每列的分隔符(默认是' ')
	- -v    修改选项值
		``` bash
		awk -F: -v OFS=: '{print $1,$NF}' /etc/passwd | column -t
		```

## 磁盘管理

### df
``` bash
# 显示磁盘分区上可以使用的磁盘空间
df -h
```
### du
``` bash
# 显示每个目录和文件的磁盘使用空间
du -h
```
	


## Linux性能调优

### 性能问题分析
* 需要性能优化的现象
	- 响应慢
	- 负载高

* 性能优化的方向
	- 系统问题:  例如CPU利用率、SWAP利用率或者IO过高导致的整体性能下降
	- 功能性问题:
	- 新出现问题:  例如系统做了哪些变动
	- 不规律问题:

* 操作系统问题
	```
	操作系统的几个问题之间是相互依赖的:
		- CPU过度使用会造成大量的进程等待CPU资源，系统响应变慢，等待的进程数量会增加，导致内存增加，内存耗尽会使用虚拟内存，虚拟内存使用又会造成磁盘IO增加和CPU开销增加
	```
	- CPU
	- 内存
	- 磁盘
	- 网络

* 性能问题出现的原因
	- 应用程序设计的缺陷和数据库查询的滥用最有可能导致性能问题
	- 可能造成CPU瓶颈的问题: 不合理的数据库查询
	- 可能造成内存瓶颈的问题: 高并发、系统进程多、或者是内存泄漏
	- 可能会造成磁盘瓶颈的原因: 数据库频繁更新，或者查询大表
	- 性能瓶颈如果是内存/磁盘，最终表现出的结果就是CPU耗尽，系统负载极高，响应缓慢，甚至暂时失去响应
	- 物理内存不够时会使用交换内存，会带来磁盘IO和CPU的开销

### 系统性能分析工具
* vmstat
	- vmstat是Virtual Memory Statistics(虚拟内存统计)的缩写，可以对操作系统的内存信息、进程状态、CPU活动进行监视
	- 使用语法
	```
	vmstat [-V] [-n] [delay [count]]
		- -V 表示打印出版本信息
		- -n 表示在周期性循环输出时，输出的头部信息仅显示一次
		- delay 表示两次输出之间的间隔时间
		- count 表示统计的总次数，默认为1

	eg:
		- vmstat 3
		- vmstat 3 5
	```
* iostat
	- 对输入输出进行统计，主要的功能是对系统的磁盘IO进行监视
	- 使用语法:
	```
	iostat 
		- -c 显示CPU使用情况
		- -d 显示磁盘的使用情况
		- -k 每秒以k bytes为单位显示数据
		- -x 指定要统计的磁盘设备的名称，默认为所有的磁盘设备
		- interval 指定两次统计间隔时间
		- count 指定统计的次数
	
	eg:
		iostat -c 5 3
		iostat -d 5 3
	```
* sar
	- sar命令很强大，是分析系统性能的重要工具之一，可以全面的获取系统的CPU、运行队列、磁盘IO、分页(交换区)、内存、CPU中断、网络等性能参数
	- 使用语法
	```
	sar 
		- -a 显示系统所有资源设备(CPU、内存、磁盘)的运行状况
		- -u 显示系统所有CPU在采样时间内的负载状态
		- -p 显示当前系统中指定CPU的使用状况
		- -d 显示系统所有磁盘设备在采样时间内的使用状况
		- -r 显示系统内存在采样时间内的使用状况
		- -b 显示缓冲区在采样时间内的使用情况
		- -v 显示进程、文件、inode和锁表状态
		- -n 显示网络运行状态，参数后面可以跟DEV、EDEV、SOCK和FULL
		- interval 表示采样时间间隔
		- count 采样次数

	eg:
		sar -u 3 5
		sar -d 3 5
		sar -r 3 5
		sar -n 3 5
	```
* top
	- 在top界面按1会出现全部CPU的使用情况
	- 在top界面按大写的M可以按照内存使用情况进行排序
	- 在top界面按k可以将对应pid号的进程终止
```
使用语法: top 
	* -n 设置屏幕刷新的次数
	* -b 将top的输出信息排版以适合输出文件的格式输出到屏幕上
eg:
	top -b -n 1 | grep cpu
```

* iotop

* free
	- free命令显示系统内存使用情况，包括物理内存、交换文件(swap)和内核缓冲区内存
	```
	使用语法: free
		- -h 显示数据人性化
		- -s 指定刷新的频率
	eg:
		free -h -s 3
	```

* ps
```
# 根据CPU使用升序排序
ps aux --sort -pcpu | less

# 按照内存使用情况升序排序
ps aux --sort -pmem | less
``` 
* /proc目录
``` bash
# 查看CPU信息
cat /proc/cpuinfo

# 查看内存使用情况
cat /proc/meminfo

# /proc/stat 提供系统CPU和任务统计信息，如只需要各个CPU的信息
cat /proc/stat | grep ^cpu

# 查看调用信息
watch -d cat /proc/interrupts
```

### 评判标准
* CPU
	- user% + sys% < 70%时状态良好
	- user% 表示CPU在用户模式下的时间百分比
	- sys%  表示CPU在系统模式下的时间百分比
* 内存
	- swap in(si) = 0
	- swap out(so) = 0
* 磁盘
	- iowait% < 20%




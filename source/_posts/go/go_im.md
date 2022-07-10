---
title: im-goim 
categories: 
- GolangStudy
---

## goim架构

![](../img/arch.png)

## 环境搭建

* 启动`kafka`
	- 首先启动`zookeeper`
		``` bash
		sudo bin/zookeeper-server-start.sh  config/zookeeper.properties

		# 后台启动zookeeper
		sudo bin/zookeeper-server-start.sh -daemon  config/zookeeper.properties
		```
	- 再启动`kafka`
		``` bash
		sudo bin/kafka-server-start.sh config/server.properties

		# 后台启动kafka
		sudo bin/kafka-server-start.sh -daemon config/server.properties
		```

* 编译goim
	``` bash
	git clone https://github.com/Terry-Mao/goim
	cd goim 
	make build
	# make build 之后即可在target文件加中找到可执行文件
	```

* 编译`discovery`
	``` bash
	git clone https://github.com/bilibili/discovery.git
	cd discovery
	# make build 之前需要修改Makefile文件 将文件中的cmd/discovery/discovery-example.toml改为cmd/discovery/discovery.toml
	make build
	```

* 启动`discovery`
	``` bash
	cd discovery
	./dist/bin/discovery -conf ./dist/conf/discovery.toml
	```

* 启动`comet`
	``` bash
	# 设置环境变量
	export REGION=sh
    export ZONE=sh001
    export DEPLOY_ENV=dev

	# 前台运行
	target/comet -conf=target/comet.toml -region=sh -zone=sh001 -deploy.env=dev -weight=10 -addrs=127.0.0.1 

	# 后台运行
	nohup target/comet -conf=target/comet.toml -region=sh -zone=sh001 -deploy.env=dev -weight=10 -addrs=127.0.0.1 -debug=true 2>&1 > target/comet.log &
	```

* 启动`logic`
	``` bash
	# 前台运行
	target/logic -conf=target/logic.toml -region=sh -zone=sh001 -deploy.env=dev -weight=10

	# 后台运行
	nohup target/logic -conf=target/logic.toml -region=sh -zone=sh001 -deploy.env=dev -weight=10 2>&1 > target/logic.log &
	```

* 启动`job`
	``` bash
	# 前台运行
	target/job -conf=target/job.toml -region=sh -zone=sh001 -deploy.env=dev

	# 后台运行
	nohup target/job -conf=target/job.toml -region=sh -zone=sh001 -deploy.env=dev 2>&1 > target/job.log &
	```

* 直接启动`logic`，`comet`,`job`
	``` bash
	make run
	```


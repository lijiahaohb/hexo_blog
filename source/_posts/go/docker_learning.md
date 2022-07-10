---
title: docker基础 
categories: 
- GolangStudy
---

# docker

## docker架构

* docker架构
	- docker client
	- docker daemon
		* containers
		* images
	- registry

* docker的守护进程
	- 查看守护进程的状态: `sudo service docker status`
	- 启动、关闭、重启守护进程
	``` bash
	# 停止守护进程
	sudo service docker stop
	# 启动守护进程
	sudo service docker start
	# 重启守护进程
	sudo service docker restart
	```
	- 守护进程的配置


## 容器

* <b>运行容器</b>

	<font color="red"><b>note: </b></font>运行容器的时候需要给容器一个要执行的任务，该任务执行完之后，容器就自动停止了

	``` bash
	sudo docker run --name mysql8019 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.0.19

	sudo docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p

	# 创建一个守护式容器(在后台运行)
	sudo docker run --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
	```

	命令格式:<br/>
	Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...] <br/>
	-i, --interactive=false 是否打开控制台交互 <br/>
	-t, --tty=false 是否分配tty设备支持终端登录 <br/>
	-d,    创建守护式容器，在后台运行，为了防止容器停止，给其一个运行的任务 <br/>
	-e,    指定环境变量，容器中可以使用该环境变量 <br/>
	-P,    指定容器的端口号 <br/>
	-p,    指定本地端口到容器内部端口的映射 <br/>
	--rm,  指定容器停止后自动删除容器 <br/>
	-h,    指定容器的主机名 <br/>
	--name 指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字 <br/>

* <b>查看容器信息</b>
	``` bash
	sudo docker info
	```

* <b>显示容器</b>
	``` bash
	# 显示全部容器
	sudo docker ps -a

	# 显示当前运行的容器
	sudo docker ps
	```

* <b>启动容器</b>
	``` bash
	# 启动一个已经停止的容器
	sudo docker start 容器ID

	# 重启一个容器
	sudo docker restart 容器ID
	```

* <b>容器附着</b>
	``` bash
	# 进入一个运行中的容器
	sudo docker attach 容器ID
	```

* <b>关闭容器</b>
	``` bash
	# 关闭运行中的容器
	sudo docker stop 容器ID
	```

* <b>容器日志查看</b>
	``` bash
	# 通过容器名查看
	sudo docker logs bob_the_container

	# 通过容器ID查看
	sudo docker logs 16ecf568ca88

	# 查看守护容器的日志消息

	# -f：类似于tail -f命令，可以用来监控Docker的日志，会持续动态更新打印日志的内容，按下Ctrl+C可退出查看
	sudo docker logs -f daemon_dave

	# --tail 用来查看日志的某一片段

	# 查看日志的最后10行内容
	sudo docker logs --tail 10 daemon_dave
	# 查看某个容器的最新日志而不必读取整个日志文件
	sudo docker logs --tail 0 -f daemon_dave
	 
	# --t 为每一条日志加上时间戳
	sudo docker logs -ft daemon_dave
	```

* <b>容器的进程信息</b>
	``` bash
	sudo docker top 容器ID/容器名称
	```

* <b>容器的统计信息</b>
	``` bash
	sudo docker stats 容器ID/容器名称
	```

* <b>容器的详细信息</b>
	``` bash
	sudo docker inspect
	```

* <b>在容器的内部执行新程序</b>
	``` bash
	# 先运行容器
	sudo docker start lijiahao

	# 在不进入该容器的情况下，让其执行一条touch命令
	sudo docker exec bob_the_container touch /etc/new_config_file

	# 进入该容器，可以查看到该文件创建成功
	sudo docker attach lijiahao 
	ls /etc/new_config_file
	```

	``` bash
	# 在一个守护容器中启动新的任务 /bin/bash
	sudo docker exec -it 66ba58480ea9 /bin/bash
	```

* <b>自动重启容器</b>

	--restart选项可以让Dcoker在出错的情况下自动重启 <br/>
	--restart会检查容器的退出代码，并依次来判断是否要重启容器 <br/>
	默认的情况下，Docker是不会重启任何容器的 <br/>

	创建一个容器：--restart的取值为always，表示Docker总是会自动重启该容器

	``` bash
	sudo docker run --restart=always -i -t --name container_dong -d ubuntu /bin/sh
	```

	创建一个容器: on-failure标志表示只有当容器的退出代码为非0值的时候才会自动重启 <br/>
	on-failure标志后面还接一个可选的数值，表示最多重启多少次

	``` bash
	sudo docker run --restart=on-failure:5 -i -t --name container_dong -d ubuntu /bin/sh
	```

* <b>删除容器</b>
	``` bash
	# 删除容器
	sudo docker rm 容器ID

	# 删除所有容器(-q 只列出容器ID)
	sudo docker rm $(sudo docker ps -aq)
	```

* <b>导出容器</b>
	``` bash
	# 导出容器
	sudo docker export 容器ID > xxx.tar
	```

* 导入容器
	``` bash
	sudo docker load -i xxx.tar
	```

## 镜像

* <b>查看本地镜像</b>
	``` bash
	# 查看本地镜像
	sudo docker images
	```

* <b>删除本地镜像</b>
	``` bash
	# 删除指定镜像
	sudo docker rmi 镜像名
	# 强制删除镜像(会把镜像生成的容器一并删除)
	sudo docker rmi -f 镜像名
	# 删除所有镜像
	sudo docker rmi $(sudo docker images -q)
	```

* <b>镜像标签</b>
	``` bash
	# 为了区分一个仓库中不同的镜像，Docker提供了标签（tag）的功能
	```

* <b>存储路径</b>
	``` bash
	/var/lib/docker：该目录存放着Docker镜像、容器以及容器的配置
	/var/lib/docker/image：这个目录下存放着镜像
	/var/lib/docker/containers：所有的容器都保存在该目录下
	```

* <b>镜像的拉取</b>
	``` bash
	# 如果本地没有ubuntu镜像，就会去Docker Hub中拉取镜像
	sudo docker run -t -i ubuntu /bin/bash

	# 拉取镜像
	sudo docker pull ubuntu

	# docker run和docker pull在下载镜像的时候，如果没有指定镜像的标签，那么默认下载的是标签为“latest”的镜像
	```

* <b>镜像的推送</b>
	``` bash
	# 登录
	sudo docker login

	# 更改仓库名(更改的仓库名必须前面是 用户名)
	sudo docker tag  lijiahao/static_web:v2 lijahaohb/static_web:v2

	# push
	sudo docker push lijahaohb/static_web:v2
	```

* <b>镜像的搜素</b>
	``` bash
	# 通过docker search命令来查找所有Docker Hub上公共的可用镜像
	sudo docker search puppet
	```

* <b>镜像的构建</b>
	``` bash
	# 使用docker commit命令来提交一个新镜像

	# 演示案例
	# 第一步：运行一个带有ubuntu镜像的容器
	sudo docker run -i -t ubuntu /bin/bash

	# 第二步：然后我们在该ubuntu镜像系统中安装Apache软件包
	# 更新，-yqq忽略所有提示信息
	apt-get -yqq update
	# 安装apache2服务器
	apt-get -y install apache2

	# 提交指定的容器 
	# 查看刚才那个容器的ID
	sudo docker ps -a
	# 提交定制容器，需要指定容器ID、镜像名
	sudo docker commit f3f694f1fc97 jamtur01/apache2

	# 第四步：查看一下新创建的镜像
	sudo docker images jamtur01/apache2

	# 第五步：可以通过dcoker inspect命令查看新创建的镜像的详细信息
	sudo docker inspect jamtur01/apache2
	```

* <b>dockerfile + docker build</b>
	- 项目地址

		[自己制作的项目地址](https://github.com/gzyunke/test-docker)
	- 编写`dockerfile`

		`dockerfile:`
		``` dockerfile
		FROM node:11
		MAINTAINER easydoc.net

		# 复制代码
		ADD . /app

		# 设置容器启动后的默认运行目录
		WORKDIR /app

		# 运行命令，安装依赖
		# RUN 命令可以有多个，但是可以用 && 连接多个命令来减少层级。
		# 例如 RUN npm install && cd /app && mkdir logs
		RUN npm install --registry=https://registry.npm.taobao.org

		# CMD 指令只能一个，是容器启动后执行的命令，算是程序的入口。
		# 如果还需要运行其他命令可以用 && 连接，也可以写成一个shell脚本去执行。
		# 例如 CMD cd /app && ./start.sh
		CMD node app.js
		```
	- 使用`docker build`指令构建镜像
		``` bash
		sudo docker build -t=test:v1 .
		```
	- 查看镜像是否构建成功
		``` bash
		sudo docker images
		```
	- 查看构建历史
		``` bash
		sudo docker history test:v1 
		```
	- 根据上面创建的镜像运行一个容器
		``` bash
		sudo docker run -d -p 8080:8080 --name test-hello test:v1
		```

## 目录挂载

* `bind mount` 直接把宿主机目录映射到容器内，适合挂代码目录和配置文件。可挂到多个容器上
* `volume` 由容器创建和管理，创建在宿主机，所以删除容器不会丢失，官方推荐，更高效，Linux 文件系统，适合存储数据库数据。可挂到多个容器上
* `tmpfs mount` 适合存储临时文件，存宿主机内存中。不可多容器共享。
	``` bash
	# bind mount 方式用绝对路径 -v /home/lijiahao/docker/test-docker:/code:/app

	# volume 方式，只需要一个名字 -v db-data:/app

	sudo docker run -p 8080:8080 --name test-hello -v /home/lijiahao/docker/test-docker:/app -d test:v1
	```

## 容器间通信

* 容器间通信的方式
	- docker内部网路
	- docker networking功能
	- docker链接

* docker networking
	``` bash
	# 1. 创建新的Docker网络
	sudo docker network create test-net

	# 2. 查看新创建的网络
	sudo docker network inspect test-net

	# 3. 列出当前宿主机中所有的网络
	sudo docker network ls

	# 4. 启动容器，并加入到app新网络
	sudo docker run -d --name redis --network test-net --network-alias redis redis:latest

	# 5. 启动另一个容器，并加入到app新网络
	sudo docker run -p 8080:8080 --name test -v /home/lijiahao/docker/test-docker:/app --network test-net -d test:v1
	```

* docker链接

## docker-compose

* docker-compose安装
	``` bash
	# 安装docker-compose稳定版本
	sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

	# 对二进制文件应用可执行权限
	sudo chmod +x /usr/local/bin/docker-compose

	# 验证安装是否成功
	docker-compose --help
	```

* docker-compose 使用
	- 编写`docker-compose.yml`
		
		`docker-compose.yml`
		``` yml
		version: "3.7"

		services:
		  app:
			build: ./
			ports:
			  - 80:8080
			volumes:
			  - ./:/app
			environment:
			  - TZ=Asia/Shanghai
		  redis:
			image: redis:5.0.13
			volumes:
			  - redis:/data
			environment:
			  - TZ=Asia/Shanghai

		volumes:
		  redis:
		```
	- 启动
		``` bash
		sudo docker-compose up -d
		```
	- 查看运行状态
		``` bash
		sudo docker-compose ps
		```
	- 停止运行
		``` bash
		sudo docker-compose stop
		```
	- 重启
		``` bash
		sudo docker-compose restart
		```
	- 重启单个服务
		``` bash
		sudo docker-compose restart service-name
		```
	- 进入容器命令行
		``` bash
		sudo docker-compose exec service-name sh
		```
	- 查看容器运行log
		``` bash
		sudo docker-compose logs [service-name]
		```


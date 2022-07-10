---
title: go程序调试 
categories: 
- GolangStudy
---

# go程序调试

## delve

* `delve`安装
	``` bash
	go install github.com/go-delve/delve/cmd/dlv@latest

	# 如果go版本小于1.16可以使用下面方式安装
	git clone https://github.com/go-delve/delve
	cd delve
	go install github.com/go-delve/delve/cmd/dlv]
	```

* 进入调试的方法
	``` bash
	# 1、编译并开始调试当前目录下的包或者指定的包
	dlv debug xxx.go

	# 2、找到项目运行的进程id,attach上去
	go build -gcflags "-N -l" main.go
	./main
	ps -ef | grep ./main
	dlv attach 进程id
	```

* 报错处理
	``` bash
	# Could not attach to pid 129591: this could be caused by a kernel security setting, try writing "0" to /proc/sys/kernel/yama/ptrace_scope

	sudo sysctl kernel.yama.ptrace_scope=0
	```

* `dlv`指令
	- 断点管理
		``` bash
		break(b)		设置断点	
		breakpoints(bp)	查看当前所有断点	
		clear			删除断点
		clearall		删除多个断点	
		toggle			启用或关闭断点
		```

	- 调试指令
		``` bash
		continue(c)				继续执行到一个断点或者程序结束
		next(n)					执行下一行代码
		step(s)					进入函数执行下一行代码
		step-instruction(si)	执行下一行机器码
		stepout(so)				跳出当前执行函数	
		restart(r)				重新执行程码
		```
	- 参数管理
		``` bash
		locals		打印所有的局部变量
		vars 		打印所有的全局变量
		args		打印当前函数的参数	
		display		打印加入到display的变量的值	
		print(p)	打印表达式的结果
		set			设置某个变量的值	
		vars		查看全局变量
		whatis		查看变量类型	
		```

* debug案例1
	`config.yaml`
	``` yaml 
	TimeStamp: "2018-07-16 10:23:19"
	Author: "WZP"
	PassWd: "Hello"
	Information:
	  Name: "Harry"
	  Age: "37"
	  Alise:
		- "Lion"
		- "NK"
		- "KaQS"
	  Image: "/path/header.rpg"
	  Public: false

	Favorite:
	  Sport:
		- "swimming"
		- "football"
	  Music:
		- "zui xuan min zu feng"
	  LuckyNumber: 99
	```

	`main.go`
	``` go
	package main

	import (
			"fmt"
			"log"

			"github.com/spf13/viper"
	)

	type config struct {
			v *viper.Viper
	}

	func LoadConfigFromYaml(c *config) error {
			c.v = viper.New()
			c.v.SetConfigFile("./config.yaml")

			if err := c.v.ReadInConfig(); err != nil {
					return err
			}
			age := c.v.Get("Information.Age")
			name := c.v.Get("Information.Name")
			log.Printf("age:%s, name:%s\n", age, name)

			m := c.v.Sub("information")
			log.Printf("keys:%s, image:%s", m.AllKeys(), m.Get("image"))
			return nil
	}

	func main() {
			cfg := config{v: viper.New()}
			if err := LoadConfigFromYaml(&cfg); err != nil {
					fmt.Println("Failed read config")
			}
	}
	```


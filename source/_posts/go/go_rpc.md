---
title: Golang rpc框架 
categories: 
- GolangStudy
---

## 基本概念

* 注册中心: 服务注册中心用来实现服务发现和服务的元数据存储。现在主流的做法是通过：`zookeeper`，`eureka`，`consul`，`etcd` 等开源框架实现。 

* 服务注册: 服务端提供者将服务的元数据信息注册到注册中心的过程，这些元数据包含：服务名，监听地址，监听协议，权重v吞吐率等。

* 服务发现: 客户端获取服务元数据的过程，有了这些元数据，客户端就可以发起服务调用了。获取元数据可以有两种实现方式：pull（自己去注册中心取）、push（注册中心主动告诉我）

## rpcx

* [rpcx技术文档](https://doc.rpcx.io/)

* 安装`rpcx`

``` bash
go get -u -v -tags "reuseport quic kcp zookeeper etcd consul ping" github.com/smallnest/rpcx/...
```

`服务端代码:`

`server/server.go`

``` go
package main

import (
	"flag"
	"github.com/smallnest/rpcx/server"
	"github.com/smallnest/rpcx/serverplugin"
	"log"
	"time"
)

var (
	addr      = flag.String("addr", "localhost:8972", "server address")
	etcdServers = []string{"127.0.0.1:2379"}
	basePath  = "/rpcx/test"
)

func main() {
	flag.Parse()

	//1、new一个服务struct
	s := server.NewServer()
	//2、连接注册中心（这里是zookeeper）
	addRegistryPlugin(s)
	//3、服务注册
	s.RegisterName("Arith", new(Arith), "")
	//4、启动服务监听
	s.Serve("tcp", *addr)
}

func addRegistryPlugin(s *server.Server) {
	r := &serverplugin.EtcdV3RegisterPlugin{
		ServiceAddress: "tcp@" + *addr,
		EtcdServers:   etcdServers,
		BasePath:       basePath,
		UpdateInterval: time.Minute,
	}
	err := r.Start()
	if err != nil {
		log.Fatal(err)
	}
	s.Plugins.Add(r)
}
```

`server/rpc.go`

``` go
package main 

// 定义服务
type Arith int

func (t *Arith) Mul(cxt context.Context, args *Args, reply *int) error {
  fmt.Println("Mul on", *addr)
  *reply = args.A * args.B
  return nil
}

func (t *Arith) Div(cxt context.Context, args *Args, quo *Quotient) error {
  fmt.Println("Div on", *addr)
  if args.B == 0 {
    return errors.New("divide by 0")
  }

  quo.Quo = args.A / args.B
  quo.Rem = args.A % args.B
  return nil
}
```

公共代码`proto/proto.go`

``` go
package proto

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

```

* 初始化server
* 连接注册中心
* 注册服务
* 启动监听服务


`客户端代码`

`client.go`

``` go
package main

import (
	"github.com/smallnest/rpcx/client"
	"context"
	"flag"
	"log"
	"time"

	"exercise/rpcx_test/proto"
)
var (
	etcdServers = []string{"localhost:2379"}
	basePath  = "/rpcx/test"
)

func main()  {
	flag.Parse()

	//1、启动一个ZookeeperDiscovery实例
	d := client.NewEtcdV3Discovery(basePath,"Arith", etcdServers, nil)
	//2、启动一个客户端
	xclient := client.NewXClient("Arith", client.Failtry, client.RandomSelect, d, client.DefaultOption)

	defer xclient.Close()

	args := &proto.Args{
		A: 25,
		B: 4,
	}

	quo := &proto.Quotient{}

	for {
		reply := new(int)

		err := xclient.Call(context.Background(), "Mul", args, reply)
		if err != nil {
			log.Printf("failed to call: %v\n", err)
		}

		log.Printf("%d * %d = %d", args.A, args.B, *reply)

		err = xclient.Call(context.Background(), "Div", args, quo)
		if err != nil {
			log.Printf("failed to call: %v\n", err)
		}
		log.Printf("%d / %d = %d, 余数为%d", args.A, args.B, quo.Quo, quo.Rem)
		time.Sleep(3*time.Second)
	}
}

```

* 服务发现

服务发现主要涵盖两个方面：
	- 客户端获取服务元数据
	- 自动剔除失效的服务

* 启动服务发现实例
* 启动客户端


## protobuf环境搭建

* 安装protobuf编译工具
``` bash
# 下载protobuf
git clone https://github.com/protocolbuffers/protobuf.git
# 安装依赖库
sudo apt-get install autoconf  automake  libtool curl make  g++  unzip libffi-dev -y
# 进入目录
cd protobuf/
# 自动生成configure配置文件：
./autogen.sh
# 配置环境：
./configure
# 编译源代码(要有耐心！)：
make
# 安装
sudo make install
# 刷新共享库 （很重要的一步啊）
sudo ldconfig
# 成功后需要使用命令测试
protoc -h
```

* 安装go 插件
``` bash
go get -u github.com/golang/protobuf/protoc-gen-go
go get -u google.golang.org/protobuf/proto
```

## protobuf使用步骤

* 书写.proto文件

	`pb/Person.proto: `

	``` protobuf
	syntax = "proto3";
	package pb;

	import "google/protobuf/timestamp.proto";

	option go_package = "./;pb";

	message Person {
		string name = 1;
		int32 id = 2;  // Unique ID number for this person.
		string email = 3;

		enum PhoneType {
		  MOBILE = 0;
		  HOME = 1;
		  WORK = 2;
		}

		message PhoneNumber {
		  string number = 1;
		  PhoneType type = 2;
		}

		repeated PhoneNumber phones = 4;

		google.protobuf.Timestamp last_updated = 5;
	  }

	  // Our address book file is just one of these.
	  message AddressBook {
		repeated Person people = 1;
	}
	```

* 编译.proto文件
``` bash
protoc *.proto --go_out=.
```

* 使用

	``` go
	package main

	import (
		"fmt"
		"zinx/myDemo/protobufDemo/pb"

		"google.golang.org/protobuf/proto"
	)

	func main() {
		// 定义一个Person结构体对象
		person := &pb.Person{
			Name:  "lijiahao",
			Id:    0,
			Email: "18702748294@163.com",
			Phones: []*pb.Person_PhoneNumber{
				&pb.Person_PhoneNumber{
					Number: "15136588270",
					Type: pb.Person_MOBILE,
				},
				&pb.Person_PhoneNumber{
					Number: "190019393",
					Type: pb.Person_HOME,
				},
				&pb.Person_PhoneNumber{
					Number: "17182920303",
					Type: pb.Person_WORK,
				},
			},
		}

		// 将Person对象进行序列化
		data, err := proto.Marshal(person)
		if err != nil {
			fmt.Println("proto marshal error: ", err)
		}

		// 反序列化
		newData := &pb.Person{}
		err = proto.Unmarshal(data, newData)
		if err != nil {
			fmt.Println("unmarshal error: ", err)
		}
		fmt.Println(newData)
	}
	```

## grpc 使用步骤

* 安装grpc go插件
```
go get -u -v google.golang.org/grpc
```

* 编写.proto文件

	`pb/hello_grpc.proto`

	``` protobuf
	// 指定protobuf版本号
	syntax = "proto3";

	package pb;

	option go_package = "./;hello_grpc";

	// 定义服务
	service Greeter {
	  rpc SayHello (HelloRequest) returns (HelloReply) {}
	}

	message HelloRequest {
	  string name = 1;
	}

	message HelloReply {
	  string message = 1;
	}
	```

* 编译.proto文件
	- 安装go语言插件
		``` bash
		go get -u github.com/golang/protobuf/protoc-gen-go
		```
	- 编译.proto文件
		``` bash
		protoc -I pb/ pb/*.proto --go_out=plugins=grpc:service
		* -I 后面指定proto文件存放目录，和proto文件
		* --go_out=plugins=grpc:后面指定生成go代码存放的目录
		```


* 编写服务端代码
	`server/server.go`
	``` go
	package main

	import (
		"context"
		"fmt"
		"hello_grpc/service"
		"google.golang.org/grpc"
		"google.golang.org/grpc/reflection"
		"net"
	)

	type server struct {}

	// 给对象绑定方法，实现接口
	func (s *server) SayHello(ctx context.Context, in *service.HelloRequest) (*service.HelloReply, error) {
		return &service.HelloReply{Message: "hello " + in.Name}, nil
	}

	func main() {
		// 监听本地端口
		lis, err := net.Listen("tcp", ":8080")
		if err != nil {
			fmt.Printf("监听端口失败: %s", err)
			return
		}

		// 创建gRPC服务器
		s := grpc.NewServer()

		// 注册服务
		service.RegisterGreeterServer(s, &server{})

		reflection.Register(s)

		// 开启服务
		err = s.Serve(lis)
		if err != nil {
			fmt.Printf("开启服务失败: %s", err)
			return
		}
	}
	```

* 编写客户端代码
	`client/client.go`
	``` go
	package main

	import (
		"context"
		"fmt"
		"hello_grpc/service"
		"google.golang.org/grpc"
	)

	func main(){
		// 连接服务器
		conn, err := grpc.Dial(":8080", grpc.WithInsecure())
		if err != nil {
			fmt.Printf("连接服务端失败: %s", err)
			return
		}
		defer conn.Close()

		// 新建一个客户端
		c := service.NewGreeterClient(conn)

		// 调用服务端函数
		r, err := c.SayHello(context.Background(), &service.HelloRequest{Name: "horika"})
		if err != nil {
			fmt.Printf("调用服务端代码失败: %s", err)
			return
		}

		fmt.Printf("调用成功: %s", r.Message)
	}

	```

## RPC封装
* 服务端使用步骤
	- 注册Rpc服务对象，并给对象绑定方法(定义类、绑定方法)
	``` go
	rpc.RegisterName("服务名", 回调对象)
	```

	- 创建监听器
	``` go
	listener, err := net.Listen()
	```

	- 建立连接
	``` go
	conn, err := listener.Accept()
	```

	- 给连接绑定Rpc服务
	``` go
	rpc.ServeConn(conn)
	```

* 客户端使用步骤
	- 使用Rpc连接服务器
	``` go
	conn, err := rpc.Dial()
	```

	- 调用远程函数
	``` go
	conn.Call("服务名.方法名", 传入参数, 传出参数)
	```

* 客户端和服务端封装
	``` go
	package design

	import (
		"net/rpc"
		"net/rpc/jsonrpc"
	)

	// 服务端封装
	/*
		* 封装一个接受接口参数的函数来注册服务，传入参数的对象必须实现了接口的方法
		* 封装一个接口来限定注册服务的时候传入的对象的方法的参数
	*/
	type MyInterface interface {
		HelloWorld(string, *string) error
	}

	func RegisterService(i MyInterface) error {
		return rpc.RegisterName("hello", i)
	}

	// 客户端封装
	/*
		* 封装连接远程服务器和调用远程函数过程
		* 定义类(类中包含了 rpc.Client的指针) 绑定类方法
		* 使用InitClient函数初始化类，其中封装了连接远程服务器的过程，在类方法中封装了调用远程函数的过程
	*/
	type MyClient struct {
		c *rpc.Client
	}

	func InitClient(addr string) *MyClient {
		conn, _ := jsonrpc.Dial("tcp", addr)

		return &MyClient{c:conn}
	}

	func (m *MyClient)HelloWorld(a string, b *string) error {
		return m.c.Call("hello.HelloWorld", a, b)
	}
	```


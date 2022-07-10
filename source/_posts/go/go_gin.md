---
title: Golang后端框架 
categories: 
- GolangStudy
---
# go Web后端框架

## go标准库`net/http`

`服务端示例代码:`
``` go 
package main

import (
	"fmt"
	"net/http"
	"io/ioutil"
)

func sayhello(res http.ResponseWriter, req *http.Request) {
	b, err := ioutil.ReadFile("./hello.txt")
	if err != nil {
		fmt.Printf("ReadFile failed, err: %v", err)
		return
	}
	fmt.Fprintln(res, string(b))
}

func main() {
	http.HandleFunc("/hello", sayhello)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Printf("http server failed, err: %v", err)
		return
	}
}
```

## gin框架基本使用

* gin框架安装

``` bash
# gin框架安装
go get -u "github.com/gin-gonic/gin""
```

`服务端代码`
``` go
package main

import "github.com/gin-gonic/gin"

func sayhello(c *gin.Context) {
	c.JSON(200, gin.H{
		"message": "hello golang",
	})
}

func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()

	//指定用户使用GET请求访问/hello时，执行sayhello
	r.GET("/hello", sayhello)
	//启动服务
	r.Run()
}
```

* <b>gin 返回json数据<b/>

``` go
func main() {
	r := gin.Default()

	// gin.H 是map[string]interface{}的缩写
	r.GET("/someJSON", func(c *gin.Context) {
		// 方式一：自己拼接JSON
		c.JSON(http.StatusOK, gin.H{"message": "Hello world!"})
	})
	r.GET("/moreJSON", func(c *gin.Context) {
		// 方法二：使用结构体
		type msg struct {
			// 结构体tag标签
			Name    string `json:"user"`
			Message string
			Age     int
		}

		me := msg {
			"lijihao",
			"male",
			18,
		}
		c.JSON(http.StatusOK, me)
	})
	r.Run(":8080")
}
```

* <b>gin 获取querystring参数<b/>

`querystring`指的是URL中?后面携带的参数，例如：`/web?username=小王子&address=沙河` 

``` go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	r.GET("/web", func(c *gin.Context) {
		//获取get请求中的querystring
		name := c.Query("query") //通过Query方法获取get请求中的querystring数据
		age := c.Query("age")
		//name := c.DefaultQuery("query", "somebody") //找不到就给一个指定的默认值
		//name, ok := c.GetQuery("query") //如果取不到第二个参数返回布尔值
		//if !ok {
		//	name = "somebody"
		//}
		c.JSON(http.StatusOK, gin.H {
			"name": name,
			S
		})
	})
	r.Run(":9090")
}
```

* <b>gin 获取form表单参数<b/>

``` go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()

	r.LoadHTMLFiles("./login.tmpl", "./index.tmpl")

	r.GET("/login", func(c *gin.Context) {
		c.HTML(http.StatusOK, "login.tmpl", nil)
	})

	// 处理login post请求
	r.POST("/login", func(c *gin.Context) {
		//获取form表单提交的数据
		username := c.PostForm("username")
		password := c.PostForm("password")
		//username := c.DefaultPostForm("username", "***")
		//password := c.DefaultPostForm("password", "***")
		//username, _ := c.GetPostForm("username")
		//password, _ := c.GetPostForm("password")

		c.JSON(http.StatusOK, gin.H{
			"Username": username,
			"Password": password,
		})
	})
	r.Run(":9090")
}
```

`login.tmpl`的内容如下:

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>login</title>
</head>
<body>
{{/*form表单参数*/}}
<form action="/login" method="post" novalidate autocomplete="off">
    <div>
        <label for="username">username:</label>
        <input type="text" name="username" id="username">
    </div>

    <div>
        <label for="password">password:</label>
        <input type="password" name="password" id="password">
    </div>

    <div>
        <input type="submit" value="登录">
    </div>

</form>
</body>
</html>
```

`index.tmpl`的内容如下:

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
<h1>Hello Golang {{ .Username }}!</h1>
<p>你的密码是{{ .Password }}</p>
</body>
</html>
```
* <b>gin 获取path参数</b>

请求的参数通过URL路径传递，例如：`/user/search/小王子/沙河` 
``` go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	//动态路由, note:两个路径应该能够区分开
	r.GET("/user/:name/:age", func(c *gin.Context) {
		name := c.Param("name")
		age := c.Param("age")
		c.JSON(http.StatusOK, gin.H{
			"Name": name,
			"Age": age,
		})
	})

	r.GET("/blogs/:year/:month", func(c *gin.Context) {
		year := c.Param("year")
		month := c.Param("month")
		c.JSON(http.StatusOK, gin.H{
			"year": year,
			"month": month,
		})
	})
	r.Run(":9090")
}
```

* <b>gin 参数绑定<b/>


为了能够更方便的获取请求相关参数，可以使用`ShouldBind()`自动提取JSON、form表单和QueryString类型的数据，并把值绑定到指定的结构体对象。

``` go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

type UserInfo struct {
	//note: `form:"xx" json:"xxx"`指定表单和json中使用的key
	Username string `form:"username" json:"username"`
	Password string `form:"password" json:"password"`
}

func main() {
	r := gin.Default()
	r.LoadHTMLFiles("./login.tmpl")
	r.GET("user", func(c *gin.Context) {
		//var u1 UserInfo
		//u1.Username = c.Query("username")
		//u1.Password = c.Query("password")
		//参数绑定
		var u UserInfo
		//提取querystring类型的参数，绑定到结构体
		err := c.ShouldBind(&u)
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
		} else {
			fmt.Printf("%#v", u)
			c.JSON(http.StatusOK, gin.H {
				"status": "ok",
				"name": u.Username,
				"password": u.Password,
			})
		}
	})

	r.GET("/login", func(c *gin.Context) {
		c.HTML(http.StatusOK, "login.tmpl", nil)
	})

	r.POST("/form", func(c *gin.Context) {
		var u UserInfo
		//绑定form表单参数
		err := c.ShouldBind(&u)
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
		} else {
			c.JSON(http.StatusOK, gin.H{
				"status": "ok",
				"name": u.Username,
				"password": u.Password,
			})
		}
	})

	r.POST("/json", func(c *gin.Context) {
		var u UserInfo
		//绑定Json参数
		err := c.ShouldBind(&u)
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
		} else {
			c.JSON(http.StatusOK, gin.H{
				"status": "ok",
				"name": u.Username,
				"password": u.Password,
			})
		}
	})

	r.Run(":9090")
}
```

`login.tmpl`的内容:

``` html
<!DOCTYPE html>
<html lang="ch-CN">
<head>
    <title>login</title>
</head>
<body>
{{/*from表单 action -->表示要往哪提交数据 method--> 表示强求的方法*/}}
<form action="/form" method="post">
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="submit" value="提交">
</form>
</body>
</html>
```

* <b>gin 单个文件上传</b>

``` go 
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	r.LoadHTMLFiles("./index.tmpl")
	r.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", nil)
	})
	r.POST("/upload", func(c *gin.Context) {
		// 从请求中读取文件
		f, err := c.FormFile("f1")
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H {
				"error": err.Error(),
			})
		} else {
			//将文件保存到本地(服务器)
			dst := fmt.Sprintf("./%s", f.Filename)
			c.SaveUploadedFile(f, dst)
			c.JSON(http.StatusOK, gin.H {
				"status": "Ok",
			})
		}
	})
	r.Run(":9090")
}
```
`index.tmpl`的内容如下:
``` html
<!DOCTYPE html>
<html lang="ch-CN">
<head>
    <title>index</title>
</head>
<body>
<form action="/upload"  method="post" enctype="multipart/form-data">
    <input type="file" name="f1">
    <input type="submit" value="上传">
</form>
</body>
</html>
```

* <b>gin 多个文件上传</b>

``` go
func main() {
	router := gin.Default()
	// 处理multipart forms提交文件时默认的内存限制是32 MiB
	// 可以通过下面的方式修改
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["file"]

		for index, file := range files {
			log.Println(file.Filename)
			dst := fmt.Sprintf("C:/tmp/%s_%d", file.Filename, index)
			// 上传文件到指定的目录
			c.SaveUploadedFile(file, dst)
		}
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("%d files uploaded!", len(files)),
		})
	})
	router.Run()
}
```

* <b>gin 重定向</b>

请求重定向:
``` go
r.GET("/index", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.sogo.com")
})
```
请求转发
``` go
r.GET("/a", func(c *gin.Context) {
	//跳转到b对应的路由函数
	c.Request.URL.Path = "/b" //修改请求的URL地址
	r.HandleContext(c) //继续后续的处理
})
r.GET("/b", func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H {
		"message": "b",
	})
})
```

* <b>gin 重定向</b>

* <b>普通路由</b>:

``` go
r.GET("/index", func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H {
		"method": "GET",
	})
})
r.POST("/index", func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H {
		"method": "POST",
	})
})
r.PUT("/index", func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H {
		"method": "PUT",
	})
})
r.DELETE("/index", func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H {
		"method": "delete",
	})
})
```
此外，还有一个可以匹配所有请求方法的`Any`方法如下：
``` go
//能够使用这一个函数处理/user的所有请求
r.Any("/user", func(c *gin.Context) {
	switch c.Request.Method {
	case "GET":
		c.JSON(http.StatusOK, gin.H{"method": "GET"})
	case "POST":
		c.JSON(http.StatusOK, gin.H{"method": "PSOT"})
	case http.MethodPut:
		c.JSON(http.StatusOK, gin.H{"method": "PUT"})
	}
})
```
为没有配置处理函数的路由添加处理程序，默认情况下它返回404代码，下面的代码为没有匹配到路由的请求都返回`views/404.html`页面。
``` go 
r.NoRoute(func(c *gin.Context) {
		c.HTML(http.StatusNotFound, "views/404.html", nil)
	})
```

<b>路由组</b>:

``` go 
//路由组 多用于区分不同的业务线或者API版本
//把公共前缀提取出来组成一个路由组
videoGroup := r.Group("/video")
{
	videoGroup.GET("/index", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"msg": "/video/index"})
	})
	videoGroup.GET("/xx", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"msg": "/video/xx"})
	})
	videoGroup.GET("/oo", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"msg": "/video/oo"})
	})
}
```
路由组也是支持嵌套的，例如：
``` go
shopGroup := r.Group("/shop")
	{
		shopGroup.GET("/index", func(c *gin.Context) {...})
		shopGroup.GET("/cart", func(c *gin.Context) {...})
		shopGroup.POST("/checkout", func(c *gin.Context) {...})
		// 嵌套路由组
		xx := shopGroup.Group("xx")
		xx.GET("/oo", func(c *gin.Context) {...})
	}
```
* <b>gin 中间件</b>:

Gin框架允许开发者在处理请求的过程中，加入用户自己的钩子（Hook）函数。这个钩子函数就叫中间件，中间件适合处理一些公共的业务逻辑，比如登录认证、权限校验、数据分页、记录日志、耗时统计等。

<b>定义中间件</b>

Gin的中间件必须是`func(c *gin.Context)`类型 
``` go
// StatCost 是一个统计耗时请求耗时的中间件
func StatCost(c *gin.Context) {
		println("StatCost in")
		start := time.Now()
		c.Set("name", "小王子") // 可以通过c.Set在请求上下文中设置值，后续的处理函数能够取到该值
		// 调用该请求的剩余处理程序
		c.Next()
		// 不调用该请求的剩余处理程序
		// c.Abort()
		// 计算耗时
		cost := time.Since(start)
		log.Println(cost)
		println("StatCost out")
}
```
中间件一般使用闭包来做，中间还可以加一些其他逻辑，以权限校验为例:
``` go
func authMiddleware(doCheck bool) gin.HandlerFunc {
	//连接数据库
	//或者一些其它的准备工作
	return func(c *gin.Context) {
		if doCheck { //doCheck相当于一个开关
			//是否登录的判断
			// if 是登录用户
			// c.Next()
			// else
			// c.Abort()
		} else {
			c.Next()
		}
	}
}
```

<b>注册中间件</b>

在gin框架中，我们可以为每个路由添加任意数量的中间件。

* 为全局路由注册中间件
``` go
func main() {
	// 新建一个没有任何默认中间件的路由
	r := gin.New()
	// 注册一个全局中间件
	r.Use(StatCost)

	r.GET("/test", func(c *gin.Context) {
		name := c.MustGet("name").(string) // 从上下文取值
		log.Println(name)
		c.JSON(http.StatusOK, gin.H{
			"message": "Hello world!",
		})
	})
	r.Run()
}
```
* 为某个路由单独注册中间件
``` go
// 给/test2路由单独注册中间件（可注册多个）
r.GET("/test2", StatCost, func(c *gin.Context) {
	name := c.MustGet("name").(string) // 从上下文取值
	log.Println(name)
	c.JSON(http.StatusOK, gin.H{
		"message": "Hello world!",
	})
})
```
* 为路由组注册中间件
为路由组注册中间件有以下两种写法:
1. 写法1:
``` go
shopGroup := r.Group("/shop", StatCost)
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```
2. 写法2:
``` go
shopGroup := r.Group("/shop")
shopGroup.Use(StatCost)
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

`note:`
<b>gin默认中间件</b>
`gin.Default()`默认使用了`Logger`和`Recovery`中间件，其中：

`Logger`中间件将日志写入gin.DefaultWriter，即使配置了`GIN_MODE=release`
`Recovery`中间件会recover任何panic。如果有panic的话，会写入500响应码。
如果不想使用上面两个默认的中间件，可以使用`gin.New()`新建一个没有任何默认中间件的路由。


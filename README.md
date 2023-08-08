# 零、前言：

## 简介： 

Q：Gin和Gorm都是干什么的？有什么区别？

A：Gin 和 Gorm 是 Go 编程语言中流行的开源库。但是，它们服务于不同的目的，通常在 web 开发项目中一起使用。

Gin 是一个用于构建 HTTP 服务器的 web 框架。它提供了一个简单易用的 API，用于处理 HTTP 请求和响应、路由、中间件和其他常见的 web 应用程序所需的功能。它以其高性能和简约为特点，提供了轻量级和灵活的解决方案来构建 web 服务器。

Gorm 是 Go 的一个 ORM（对象关系映射）库。它提供了一个简单易用的 API，用于与数据库交互、处理数据库迁移和执行常见的数据库操作，如查询、插入、更新和删除记录。它支持多种数据库后端，包括 MySQL、PostgreSQL、SQLite 等。

总而言之, Gin 是用于处理 HTTP 请求和响应、路由、中间件和其他与网络相关的东西的 web 框架，而 Gorm 则是用于与数据库交互并执行常见数据库操作的 ORM 库。它们通常一起使用，来处理 HTTP 请求/响应并在 web 开发项目中存储或获取数据。

## 开发环境：

- Windows 10
- VSCode

---

# 一、Gin

## 0. 快速入门：

```Go
package main

import (
	"encoding/json"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/thinkerou/favicon"
)

// 中间件（拦截器），功能：预处理，登录授权、验证、分页、耗时统计...
// func myHandler() gin.HandlerFunc {
// 	return func(ctx *gin.Context) {
// 		// 通过自定义中间件，设置的值，在后续处理只要调用了这个中间件的都可以拿到这里的参数
// 		ctx.Set("usersesion", "userid-1")
// 		ctx.Next()  // 放行
// 		ctx.Abort() // 阻止
// 	}
// }

func main() {
	// 创建一个服务
	ginServer := gin.Default()
	ginServer.Use(favicon.New("./Arctime.ico")) // 这里如果添加了东西然后再运行没有变化，请重启浏览器，浏览器有缓存

	// 加载静态页面
	ginServer.LoadHTMLGlob("templates/*") // 一种是全局加载，一种是加载指定的文件

	// 加载资源文件
	ginServer.Static("/static", "./static")

	// 相应一个页面给前端

	ginServer.GET("/index", func(ctx *gin.Context) {
		ctx.HTML(http.StatusOK, "index.html", gin.H{
			"msg": "This data is come from Go background.",
		})
	})

	// 能加载静态页面也可以加载测试文件

	// 获取请求中的参数

	// 传统方式：usl?userid=xxx&username=conqueror712
	// Rustful方式：/user/info/1/conqueror712

	// 下面是传统方式的例子
	ginServer.GET("/user/info", func(context *gin.Context) { // 这个格式是固定的
		userid := context.Query("userid")
		username := context.Query("username")
		// 拿到之后返回给前端
		context.JSON(http.StatusOK, gin.H{
			"userid":   userid,
			"username": username,
		})
	})
	// 此时执行代码之后，在浏览器中可以输入http://localhost:8081/user/info?userid=111&username=666
	// 就可以看到返回了JSON格式的数据

	// 下面是Rustful方式的例子
	ginServer.GET("/user/info/:userid/:username", func(context *gin.Context) {
		userid := context.Param("userid")
		username := context.Param("username")
		// 还是一样，返回给前端
		context.JSON(http.StatusOK, gin.H{
			"userid":   userid,
			"username": username,
		})
	})
	// 指定代码后，只需要在浏览器中http://localhost:8081/user/info/111/555
	// 就可以看到返回了JSON数据了，非常方便简洁

	// 序列化
	// 前端给后端传递JSON
	ginServer.POST("/json", func(ctx *gin.Context) {
		// request.body
		data, _ := ctx.GetRawData()
		var m map[string]interface{} // Go语言中object一般用空接口来表示，可以接收anything
		// 顺带一提，1.18以上，interface可以直接改成any
		_ = json.Unmarshal(data, &m)
		ctx.JSON(http.StatusOK, m)
	})
	// 用apipost或者postman写一段json传到localhost:8081/json里就可以了
	/*
		json示例：
		{
			"name": "Conqueror712",
			"age": 666,
			"address": "Mars"
		}
	*/
	// 看到后端的实时响应里面接收到数据就可以了

	// 处理表单请求 这些都是支持函数式编程，Go语言特性，可以把函数作为参数传进来
	ginServer.POST("/user/add", func(ctx *gin.Context) {
		username := ctx.PostForm("username")
		password := ctx.PostForm("password")
		ctx.JSON(http.StatusOK, gin.H{
			"msg":      "ok",
			"username": username,
			"password": password,
		})
	})

	// 路由
	ginServer.GET("/test", func(ctx *gin.Context) {
		// 重定向 -> 301
		ctx.Redirect(301, "https://conqueror712.gitee.io/conqueror712.gitee.io/")
	})
	// http://localhost:8081/test

	// 404
	ginServer.NoRoute(func(ctx *gin.Context) {
		ctx.HTML(404, "404.html", nil)
	})

	// 路由组暂略

	// 服务器端口，用服务器端口来访问地址
	ginServer.Run(":8081") // 不写的话默认是8080，也可以更改
}
```



**API用法示例**：`https://gin-gonic.com/zh-cn/docs/examples/`

---

## 1. 基准测试

Q：基准测试是什么？

A：基准测试，也称为性能测试或压力测试，是一种**用于测量系统或组件性能的测试**。基准测试的**目的是了解系统或组件在特定条件下的性能，并将结果与其他类似系统或组件进行比较**。基准测试可用于评估各种类型的系统和组件，包括硬件、软件、网络和整个系统。

Q：什么时候需要基准测试呀？

A：基准测试通常涉及在被测系统或组件上运行特定工作负载或任务，并测量**吞吐量、延迟时间、CPU使用率、内存使用率**等各种性能指标。基准测试的结果可用于识别瓶颈和性能问题，并做出有关如何优化系统或组件以提高性能的明智决策。

有许多不同类型的基准测试，每种类型都有自己的指标和工作负载。常见的基准测试类型包括：

- 人工基准测试：使用人工工作负载来测量系统或组件的性能。
- 真实世界基准测试：使用真实世界的工作负载或场景来测量系统或组件的性能。
- 压力测试：旨在将系统或组件推到极限，以确定在正常使用条件下可能不明显的性能问题

重要的是要知道基准测试不是一次性的活动，而是应该**定期进行**的活动，以评估系统的性能并检测随时间的消耗。

Q：什么样的基准测试结果是我们想要的呀？

A：

- 在一定的时间内实现的总调用数，越高越好
- 单次操作耗时（ns/op），越低越好
- 堆内存分配 （B/op）, 越低越好
- 每次操作的平均内存分配次数（allocs/op），越低越好

---

## 2. Gin的特性与Jsoniter：

Gin v1 稳定的特性:

- 零分配路由。
- 仍然是最快的 http 路由器和框架。
- 完整的单元测试支持。
- 实战考验。
- API 冻结，新版本的发布不会破坏你的代码。
- Gin 项目可以轻松部署在任何云提供商上。



Gin 使用 encoding/json 作为默认的 json 包，但是你可以在编译中使用标签将其修改为 jsoniter。

```
$ go build -tags=jsoniter .
```

**Jsoniter是什么？**

- `json-iterator`是一款快且灵活的`JSON`解析器,同时提供`Java`和`Go`两个版本。
- `json-iterator`是最快的`JSON`解析器。它最多能比普通的解析器快10倍之多
- 独特的`iterator api`能够直接遍历`JSON` ，极致性能、零内存分配
- 从dsljson和jsonparser借鉴了大量代码。

下载依赖：`go get github.com/json-iterator/go`

---



# 二、GORM

## 0. 特性与安装：

- 全功能 ORM
- 关联 (Has One，Has Many，Belongs To，Many To Many，多态，单表继承)
- Create，Save，Update，Delete，Find 中钩子方法
- 支持 `Preload`、`Joins` 的预加载
- 事务，嵌套事务，Save Point，Rollback To Saved Point
- Context、预编译模式、DryRun 模式
- 批量插入，FindInBatches，Find/Create with Map，使用 SQL 表达式、Context Valuer 进行 CRUD
- SQL 构建器，Upsert，数据库锁，Optimizer/Index/Comment Hint，命名参数，子查询
- 复合主键，索引，约束
- Auto Migration
- 自定义 Logger
- 灵活的可扩展插件 API：Database Resolver（多数据库，读写分离）、Prometheus…
- 每个特性都经过了测试的重重考验
- 开发者友好

```
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```



其他的补充内容：

- Gorm是软删除，为了保证数据库的完整性

---

# 三、Navicat

破解教程：

```
https://www.jianshu.com/p/9c4c499429da
```



新建连接 -> MySQL -> 连接名随便 -> 密码随便 -> 双击左侧打开 -> 右键information_schema -> 新建数据库 -> 名称crud-list -> 字符集utf8mb4

这里如果打开的时候报错`navicat 1045 - access denied for user 'root'@'localhost' (using password: 'YES')`，则需要查看自己的数据库本身的问题



---



# 四、Gin+Gorm的CRUD

## 连接数据库

编写测试代码，成功运行即可，但是这个时候还不能查看数据库是否被创建，

如果要看我们需要定义结构体，然后定义表迁移，具体代码如下：

```Go
package main

import (
	"fmt"
	"time"

	// "gorm.io/driver/sqlite"
	"github.com/gin-gonic/gin"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func main() {
	// 如何连接数据库 ? MySQL + Navicat
	// 需要更改的内容：用户名，密码，数据库名称
	dsn := "root:BqV?eGcc_1o+@tcp(127.0.0.1:3306)/crud-list?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

	fmt.Println("db = ", db)
	fmt.Println("err = ", err)

	// 连接池
	sqlDB, err := db.DB()
	// SetMaxIdleConns 设置空闲连接池中连接的最大数量
	sqlDB.SetMaxIdleConns(10)
	// SetMaxOpenConns 设置打开数据库连接的最大数量。
	sqlDB.SetMaxOpenConns(100)
	// SetConnMaxLifetime 设置了连接可复用的最大时间。
	sqlDB.SetConnMaxLifetime(10 * time.Second) // 10秒钟

	// 结构体
	type List struct {
		Name    string
		State   string
		Phone   string
		Email   string
		Address string
	}

	// 迁移
	db.AutoMigrate(&List{})

	// 接口
	r := gin.Default()

	// 端口号
	PORT := "3001"
	r.Run(":" + PORT)
}
```

定义好之后我们运行，没有报错并且在终端显示出来3001就是正确的，这个时候我们可以去Navicat里面查看crud-list下面的"表"，刷新后发现有一个lists产生，那就是对的了。



但是这个时候我们存在两个问题：

1. 没有主键：在struct里添加`gorm.Model`来解决，Ctrl+左键可以查看`model`
2. 表里面的名称变成了复数：详见文档的高级主题-GORM配置里，在`*db*, *err* *:=* gorm.Open(mysql.Open(dsn), *&*gorm.Config{})`里面添加一段话即可

更改完成之后我们要先在Navicat里面把原来的表`lists`删掉才能重新创建，这个时候我们重新运行，就会发现表单里面多了很多东西

---

## 结构体定义与优化

例如：

```
`gorm:"type:varchar(20); not null" json:"name" binding:"required"`
```

需要注意的是：

1. 结构体里面的变量(Name)必须首字母大写，否则创建不出列，会被自动忽略
2. gorm指定类型
3. json表示json接收的时候的名称
4. binding required表示必须传入

----

## CRUD接口

```Go
// 测试
r.GET("/", func(c *gin.Context) {
	c.JSON(200, gin.H{
		"message": "请求成功",
	})
})
```

编写完这一段之后运行代码，然后去postman里面新建一个GET接口`127.0.0.1:3001`然后send一下，出现请求成功就请求成功了。

---

增也是一样，写好之后直接用如下JSON来测试就可以：

```JSOn
{
	"name" : "张三",
    "state" : "在职",
    "phone" : "13900000000",
    "email" : "6666@qq.com",
    "address" : "二仙桥成华大道"
}
```

返回：

```Json
{
    "code": "200",
    "data": {
        "ID": 1,
        "CreatedAt": "2023-01-24T09:27:36.73+08:00",
        "UpdatedAt": "2023-01-24T09:27:36.73+08:00",
        "DeletedAt": null,
        "name": "张三",
        "state": "在职",
        "phone": "13900000000",
        "email": "6666@qq.com",
        "address": "二仙桥成华大道"
    },
    "msg": "添加成功"
}
```

这时候也可以在数据库里看到这条数据

---

删除也是一样，编写完运行之后添加一个DELETE接口，然后输入`127.0.0.1:3001/user/delete/2`之后send就可以看到返回了（前提是有数据）

```JSON
{
    "code": 200,
    "msg": "删除成功"
}
```

如果是删除了不存在的id，就会返回

```JSON
{
    "code": 400,
    "msg": "id没有找到，删除失败"
}
```

顺带一提，事实上这个代码还可以优化，这里的if else太多了，后面优化的时候有错误直接return

---

修改也是一样，示例：

```JSON
{
	"name" : "张三",
    "state" : "离职",
    "phone" : "13900000000",
    "email" : "6666@qq.com",
    "address" : "二仙桥成华大道"
}
```

返回：

```JSON
{
    "code": 200,
    "msg": "修改成功"
}
```

---

查询分为两种：

1. 条件查询
2. 分页查询

条件查询的话，直接写好了请求`127.0.0.1:3001/user/list/王五`

返回：

```Json
{
    "code": "200",
    "data": [
        {
            "ID": 3,
            "CreatedAt": "2023-01-24T10:06:25.305+08:00",
            "UpdatedAt": "2023-01-24T10:06:25.305+08:00",
            "DeletedAt": null,
            "name": "王五",
            "state": "在职",
            "phone": "13100000000",
            "email": "8888@qq.com",
            "address": "八仙桥成华大道"
        }
    ],
    "msg": "查询成功"
}
```



全部 / 分页查询的话

譬如说请求是：`127.0.0.1:3001/user/list?pageNum=1&pageSize=2`

意思就是查询第一页的两个

返回：

```JSON
{
    "code": 200,
    "data": {
        "list": [
            {
                "ID": 3,
                "CreatedAt": "2023-01-24T10:06:25.305+08:00",
                "UpdatedAt": "2023-01-24T10:06:25.305+08:00",
                "DeletedAt": null,
                "name": "王五",
                "state": "在职",
                "phone": "13100000000",
                "email": "8888@qq.com",
                "address": "八仙桥成华大道"
            }
        ],
        "pageNum": 1,
        "pageSize": 2,
        "total": 2
    },
    "msg": "查询成功"
}
```



如果请求是：`127.0.0.1:3001/user/list`

返回：

```JSON
{
    "code": 200,
    "data": {
        "list": [
            {
                "ID": 1,
                "CreatedAt": "2023-01-24T09:27:36.73+08:00",
                "UpdatedAt": "2023-01-24T09:55:20.351+08:00",
                "DeletedAt": null,
                "name": "张三",
                "state": "离职",
                "phone": "13900000000",
                "email": "6666@qq.com",
                "address": "二仙桥成华大道"
            },
            {
                "ID": 3,
                "CreatedAt": "2023-01-24T10:06:25.305+08:00",
                "UpdatedAt": "2023-01-24T10:06:25.305+08:00",
                "DeletedAt": null,
                "name": "王五",
                "state": "在职",
                "phone": "13100000000",
                "email": "8888@qq.com",
                "address": "八仙桥成华大道"
            }
        ],
        "pageNum": 0,
        "pageSize": 0,
        "total": 2
    },
    "msg": "查询成功"
}
```

---




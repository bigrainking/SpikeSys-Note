[toc]

# Iris的基本使用

> 主要包括：
>
> 1. 创建框架对象
>
> 2. 注册中间件
> 3. 注册路由
> 4. 启动服务器等



```go
package main

import (
    "github.com/kataras/iris"

    "github.com/kataras/iris/middleware/logger"
    "github.com/kataras/iris/middleware/recover"
)

func main() {
    //1. 创建对象
    app := iris.New()
    
    //2. 注册中间件
    app.Use(recover.New()) //错误修复
    app.Use(logger.New()) //日志记录
	
    
    //3. 路由注册
    // Method:   GET
    // Resource: http://localhost:8080
    app.Handle("GET", "/", func(ctx iris.Context) {
        ctx.HTML("<h1>Welcome</h1>")
    })

    // same as app.Handle("GET", "/ping", [...])
    // Method:   GET
    // Resource: http://localhost:8080/ping
    app.Get("/ping", func(ctx iris.Context) {
        ctx.WriteString("pong")
    })

    // Method:   GET
    // Resource: http://localhost:8080/hello
    app.Get("/hello", func(ctx iris.Context) {
        ctx.JSON(iris.Map{"message": "Hello Iris!"})
    })
	
    //4. 服务器启动
    // http://localhost:8080
    // http://localhost:8080/ping
    // http://localhost:8080/hello
    app.Run(iris.Addr(":8080"))
}
```


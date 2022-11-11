[toc]



# 一、Iris的MVC特性

## 1. 根目录下的Controller

### 1）默认匹配

仅仅根据**请求的方法**直接Controller默认匹配到我们定义的function

比如 `func Get()` , 所有Get请求方法的请求都会匹配到这个function

- 八种请求方法都可以默认匹配，只是需要function的名字都是请求方法的名字，如：Put() Get() Post()....

### 2）自动匹配

根据 **请求方法+请求路径** 一起匹配到function

比如 `GetInfo()` function的名字决定了 localhost:8080/info的Get请求会自动匹配到这个function。 但名字一定要是方法and路径首字母都大写



GetInfo() 仅能匹配localhost:8080/info 路径小写

```html
Request URL: http://localhost:8080/info
Request Method: GET
Status Code: 200 OK
```



```go
//请求路径大写不行
Request URL: http://localhost:8080/Info
Request Method: GET
Status Code: 404 Not Found
```



### 3）自定义匹配

除了上面的自动 默认匹配，还可以自定义：自定义方法、路径. 在function中可以加入中间件

具体如下：

```go
func (c *CustomController) BeforeActivation(b mvc.BeforeActivation) {
	anyMiddlewareHere := func(ctx iris.Context) {
		ctx.Application().Logger().Warnf("Inside /helloBaby")
		ctx.Next()
	}
	b.Handle(
		"GET",             //httpMethod:全部大写！！这里一定要
		"/helloBaby",      // path
		"BeforeFunc",      // FuncName
		anyMiddlewareHere, // middleWare ：可以添加中间件
	)
}

// 具体的自定义匹配对应的function
func (c *CustomController) BeforeFunc() string {
	return "我是自定义处理器"
}
```





### 示例

```go
package main

// 测试Iris的MVC特性

import (
	"github.com/kataras/iris/v12"
	"github.com/kataras/iris/v12/mvc"
)

func main() {
	app := iris.New() //app相当于Engine，是全局最大的Router

	// 创建服务于 “/” 的Controller:自定义注册控制器
	mvc.New(app).Handle(new(CustomController))

	app.Run(iris.Addr(":8080"))
}

// 声明自定义的控制器
// 服务于："/" "/ping" "/hello"路由
type CustomController struct{}

// 默认匹配："/"路径的Get请求都会走这里（除非有自动匹配or自定义匹配）
// serves Get
// Request URL: http://localhost:8080/ （/hello不走这个func）
// Request Method: GET
func (c *CustomController) Get() string {
	return "there is Get() function"
}

// 自动匹配
// Request URL: http://localhost:8080/info
// Request Method: GET
func (c *CustomController) GetInfo() string {
	return "there is GetInfo() function"
}

// 自定义匹配
func (c *CustomController) BeforeActivation(b mvc.BeforeActivation) {
	anyMiddlewareHere := func(ctx iris.Context) {
		ctx.Application().Logger().Warnf("Inside /helloBaby")
		ctx.Next()
	}
	b.Handle(
		"GET",             //httpMethod:全部大写！！这里一定要
		"/helloBaby",      // path
		"BeforeFunc",      // FuncName
		anyMiddlewareHere, // middleWare ：可以添加中间件
	)
}

// 具体的自定义匹配对应的function
func (c *CustomController) BeforeFunc() string {
	return "我是自定义处理器"
}
```







## 2. 路由组下的Controller

如果是在路由组下， 默认匹配会自动匹配该路由组根下的GET、POST方法

比如 路由组是 "localhost:8080/hello", Controller有方法Get(), 将会自动匹配 localhost:8080/hello method：Get














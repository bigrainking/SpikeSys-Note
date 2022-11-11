# 一、Movie示例

> 一个完整的示例，包含了Dao(数据查询)、Model(数据库)、service(逻辑处理)、Controller-核心(控制器)、View(视图)
>
> - Model
> - Controller
> - View
> - Service
>
> 应用Iris的MVC包：Controller



## 1. 目录结构

```shell
.
├── main.go
├── model #Model
│   └── movie_model.go
├── repository
│   └── movie_repository.go
├── service #逻辑处理
│   └── movie_service.go
└── webs
    ├── controller #Controller
    │   └── movie_controller.go
    └── views    #View
        └── index.html
```



## 2. main.go

```go
package main

import (
	"Iris/IrisForMVCDemo/webs/controller"

	"github.com/kataras/iris/v12"
	"github.com/kataras/iris/v12/mvc"
)

func main() {
	app := iris.New()
	app.Logger().SetLevel("debug")
	// 注册模板
	app.RegisterView(iris.HTML("./webs/views", ".html"))
	// 注册Controller:路由组"/movie"下面的
	mvc.New(app.Party("/movie")).Handle(new(controller.MovieController))
	app.Run(iris.Addr(":8080"))
}
```



## 3. Controller

将Controller的声明、function匹配都放在这个文件

**function功能**：

- 根据请求匹配function
- 获取数据
- 渲染到模板

```go
// 声明一个Controller
type MovieController struct {
}

// GET请求匹配"/"路径下的所有内容
func (m *MovieController) Get() mvc.View {
	// 交给service去获取数据，并获取到封装好的数据，渲染到View
	repo := repository.NewMovieRepositoryManager()
	movieService := services.NewMovieServiceManager(repo)
	movieResult := movieService.ShowMovieName()
	return mvc.View{
		Name: "movie/index.html",
		Data: movieResult,
	}
}
```



## 4. model

**model对接数据库数据**

```go
package model

// Model
// 对应需求结构体
type Movie struct {
	Id   int
	Name string
}
```



**查询数据库，并返回对应结构体**

```go
// 包含所有增删查改的函数
type MovieRepository interface {
	GetMovieName() string
}

type MovieRepositoryManager struct {
}

func NewMovieRepositoryManager() MovieRepository {
	return &MovieRepositoryManager{}
}

// 实现接口
func (m *MovieRepositoryManager) GetMovieName() string {
	movie := model.Movie{
		Name: "《快乐星球》",
	}
	return movie.Name
}
```



## 5. 逻辑处理层

```go
// 获取repository查询到的内容并组装
type MovieService interface {
	ShowMovieName() string
}

type MovieServiceManager struct {
	repo repository.MovieRepository //承载另一个对象
}

func NewMovieServiceManager(repo repository.MovieRepository) MovieService {
	return &MovieServiceManager{
		repo: repo,
	}
}

// 获取到需要的数据，并整合好
func (m *MovieServiceManager) ShowMovieName() string {
	return m.repo.GetMovieName()
}
```



### go.mod

```go
module Iris/IrisForMVCDemo

go 1.18

require github.com/kataras/iris/v12 v12.1.8
```
















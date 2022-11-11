[toc]





# 一、MVC的目录结构

```shell
├── main.go
├── datamodels# 需要的数据构成的model的结构体
│   └── move.go
├── repository# 操作数据库的增删改查的类
│   └── movie_repository.go
├── service# 获取models的业务逻辑 
│   └── movie_service.go
└── webs# Views
    ├── controllers
    │   └── movie_controller.go
    └── views
        └── movie
            └── index.html
```





### 1 datamodels 从数据库中获得的结构

只放models的结构体，根据需求从数据库中获取到的信息组成结构体

`datamodels\move.go`

```go
package datamodels
// 服务器需要的数据对应的结构体：从数据库中获取数据后封装成结构体
type Movie struct {
	Name string
}
```





<img src="./pic/Iris%E6%A1%86%E6%9E%B6%E5%85%A5%E9%97%A8.assets/image-20220805115137712.png" alt="image-20220805115137712" style="zoom:25%;" />

### 2 repositories

从数据库中获取信息并返回：相当于dao层 从数据库中获取信息，并与datamodels对应封装好

```go
package repository

import "Iris-Learn/datamodels"

// 接口
type MovieRepository interface {
	GetMovieName() string
}

// 实例化接口
type MovieRepositoryManager struct {
}

// 创建对象函数
func NewMovieManager() MovieRepository {
	return &MovieRepositoryManager{}
}

// 实现接口函数
func (m *MovieRepositoryManager) GetMovieName() string {
	// 此处模拟已经从数据库中获得了数据
	movie := &datamodels.Movie{Name: "费主张的电影频道"}
	return movie.Name
}
```



<img src="./pic/MVC%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.assets/image-20220807231850764.png" alt="image-20220807231850764" style="zoom: 25%;" />



与models有一一对应关系，在数据库中的查询结果用models来装

操作数据库的类。对model拉取数据需要对数据库增删改查，其中的增删改查有许多重复的代码，将这些**对数据库的操作封装成一个类** => repository(相当于Dao文件夹)

<img src="./pic/Iris%E6%A1%86%E6%9E%B6%E5%85%A5%E9%97%A8.assets/image-20220805151959528.png" alt="image-20220805151959528" style="zoom: 25%;" />

### 3 services



**业务逻辑**，用获取到的结构体内容，并且封装打包成前端所需要的的内容。

```go
package service

import (
	"Iris-Learn/repository"
)

// 用获取到的结构体对象，封装成前端需要的内容一个整体

type MovieService interface {
	ShowMovieName() string
}

// 实现接口
type MovieServiceManager struct {
	repo repository.MovieRepository //嵌套接口暂时不管
}

func NewMoviServiceManager(repo repository.MovieRepository) MovieService {
	return &MovieServiceManager{
		repo: repo,
	}
}

func (m *MovieServiceManager) ShowMovieName() string {
	return ("我们视频的名字是" + m.repo.GetMovieName())
}
```



<img src="./pic/MVC%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.assets/image-20220807232800686.png" alt="image-20220807232800686" style="zoom:33%;" />





models的业务逻辑，一个service对应一个models。 

如下，某个service文件**返回的是一个model类型的结构体实例**，service的作用在于负责如何填充这个model，填充过程的业务逻辑就是他负责的内容，具体要填充的某些内容他可以直接找repository来获取(Dao)



<img src="./pic/Iris%E6%A1%86%E6%9E%B6%E5%85%A5%E9%97%A8.assets/image-20220805152221461.png" alt="image-20220805152221461" style="zoom: 33%;" /> <img src="./pic/Iris%E6%A1%86%E6%9E%B6%E5%85%A5%E9%97%A8.assets/image-20220805152245867.png" alt="image-20220805152245867" style="zoom:25%;" />



### 4 webs

##### Controllers：router控制器

接受来自model层的内容，选择对应的view进行渲染

```go
package controllers

import (
	"Iris-Learn/repository"
	"Iris-Learn/service"

	"github.com/kataras/iris/v12/mvc"
)

// 封装数据并渲染到对应的html页面

type MovieController struct {
}

func (m *MovieController) Get() mvc.View {
	movieRepository := repository.NewMovieManager()
	movieService := service.NewMoviServiceManager(movieRepository)
	// 需要的全部数据
	movieResult := movieService.ShowMovieName()
	// 渲染到前端
	return mvc.View{
		Name: "movie/index.html",
		Data: movieResult,
	}
}
```



![image-20220807233444452](./pic/MVC%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.assets/image-20220807233444452.png)

##### Views

只放页面对应的模板

<img src="./pic/Iris%E6%A1%86%E6%9E%B6%E5%85%A5%E9%97%A8.assets/image-20220805152336537.png" alt="image-20220805152336537" style="zoom:25%;" />

放模板传入到view里面的变量

<img src="./pic/MVC%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.assets/image-20220807233651475.png" alt="image-20220807233651475" style="zoom:25%;" />



# 二、MVC入口main

```go
package main

import (
	"Iris-Learn/webs/controllers" # 注意这里导入要写webs，把Controller的路径写完整

	"github.com/kataras/iris/v12"
	"github.com/kataras/iris/v12/mvc"
)

func main() {
	// 注册Iris
	app := iris.New() 
	// 设置错误等级
	app.Logger().SetLevel("debug")
	// 注册模板目录，views下所有.html结尾的文件
    // 第一个参数是基于根目录的模板文件的位置，第2个参数模板文件的后缀
	app.RegisterView(iris.HTML("./webs/views", ".html"))
	// 启动控制器
	mvc.New(app.Party("/hello")).Handle(new(
		controllers.MovieController),
	)
	// 启动服务器，监听8080
	app.Run(
		iris.Addr("localhost:8080"),
	)
}
```







# 三、入门案例

> 案例的代码都在上面 一、MVC的目录结构中了

### 整体运行流程

1. main.go 监听端口

2. 当有请求进来，会转发到控制器 **Controller** 处理请求

3. Controller 会调用 Repository & Service 

​	Service 从数据库中获取到数据并封装好

4. Controller 将获取到的内容渲染给view

**运行结果**

<img src="pic/2Iris的MVC目录结构.assets/image-20220808144626277.png" alt="image-20220808144626277" style="zoom:33%;" />


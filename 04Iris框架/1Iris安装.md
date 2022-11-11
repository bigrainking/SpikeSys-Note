[toc]





# 安装:

## 方法一

- Iris要求Go语言版本最低是1.8(看清楚不是18)

   > `go version` 查看版本

```shell
go get github.com/kataras/iris/v12@master
```

## 遇到问题

[BUG链接](https://github.com/kataras/iris/issues/1646)

![image-20220805163119067](../pic/1Iris%E5%AE%89%E8%A3%85.assets/image-20220805163119067.png)



**解决**

Create a new project

```shell
$ mkdir myapp
$ cd myapp
$ go mod init myapp
$ go get github.com/kataras/iris/v12@master # or @v12.2.0-beta4
```

[解决问题参考链接](https://github.com/kataras/iris#-learning-iris)

## 方法二

[安装方法参考链接](https://www.topgoer.com/Iris/%E5%AE%89%E8%A3%85.html)

编辑你项目的 go.mod 文件

在项目中的 `go.mod` 文件中

```go
module your_project_name

go 1.13

require (
    github.com/kataras/iris/v12 v12.0.0
)
```

在 `main.go`文件中使用

```go
package main
import "github.com/kataras/iris/v12"

func main() {
	add := iris.New()
}
```

> 我采取了方法二

# 参考

[安装教程](https://learnku.com/docs/iris-go/10/installation/3762)

[安装视频](https://www.bilibili.com/video/BV1UT4y137dS?p=2&vd_source=47272764e1eb400edc65776bfe6a48af)
# Go实现MVC

整体结构和流程如下：

View层： 包含template模板文件

Controller ： 处理请求，根据请求寻找对应的handler，调用对应的model。获取models返回的数据库中的内容， 将返回的结构化数据渲染到模板中。

model：创建需要的结构化数据，从数据库中获取数据并填充。 包含结构体数据、对数据库的增删改查等数据库操作，类似博客项目的dao、model



**Controller**：

- 1. 查询请求URL对应的handler

- 2. 调用对应的models

  - 让models从数据库中获取数据
  - Controller获取到model对应的数据

- 3. 将获取到的数据渲染到前端：

  - 解析模板
  - 将数据渲染到模板

> 注意：
>
> Controller尽量不要包含业务逻辑代码，他的主要功能仅仅是：获取model层返回的数据，再将数据渲染到模板中。
>
> > 比如，在博客项目中，他功能是：
> >
> > 1.获取模板 
> >
> > 2.调用service(service调用models，并返回需要的结构化封装好的数据) 
> >
> > 3.将返回的封装好的数据渲染到模板中
> >
> > 上面就说明，可以调用逻辑层的函数，只要逻辑层返回的内容是封装好的查询到的数据库中

**Controller通用伪代码**

```go
func (c *PostController) Show() {
    // 1. 根据请求,查询对应的路由
    id, ok := c.Ct.Params["post_id"] //省略没有查到的情况
    
    // 2. 调用对应的model，让model从数据库中查询数据
    post := model.RainlabBlogPosts{Id: intId}
	err = o.Read(&post)
    
    // 3. 获取model查询到的数据，调用view视图层的模板，将model的数据渲染到页面中
    s1, _ := template.ParseFiles("view/layout/header.tpl", "view/post/show.tpl", "view/layout/footer.tpl")
    // s1.ExecuteTemplate(os.Stdout, "show", nil)
    s1.ExecuteTemplate(c.Ct.ResponseWriter, "show", template.HTML(post.ContentHtml))
			
}
```



```go
func (c *PostController) Show() {
	// 1. 根据请求,查询对应的路由
	fmt.Println("\nHello PostController Show:%s", c.Ct.Params["post_id"])
	id, ok := c.Ct.Params["post_id"] /*如果确定是真实的,则存在,否则不存在 */
	if !ok {
		fmt.Println("不存在")
		http.NotFound(c.Ct.ResponseWriter, c.Ct.Request)
		// 2. 找到对应的路由
	} else {
		// 3. 调用models对应的数据模型，让model从数据库中查询出数据
		o := orm.NewOrm()
		intId, err := strconv.Atoi(id)
		CheckErr(err)
		post := model.RainlabBlogPosts{Id: intId}
		// 4. 获取models获得的数据
		err = o.Read(&post)
		if err == orm.ErrNoRows {
			fmt.Println("查询不到")
		} else if err == orm.ErrMissPK {
			fmt.Println("找不到主键")
		} else {
			// 5. 将models中的数据渲染到views中
			fmt.Println(post.Id, post.ContentHtml)
			// 控制器调用视图
			s1, _ := template.ParseFiles("view/layout/header.tpl", "view/post/show.tpl", "view/layout/footer.tpl")

			// s1.ExecuteTemplate(os.Stdout, "show", nil)
			s1.ExecuteTemplate(c.Ct.ResponseWriter, "show", template.HTML(post.ContentHtml))
			// s1.Execute(c.Ct.ResponseWriter, post)
		}
		// s1.Execute(c.Ct.ResponseWriter, nil)
	}
}
```





# 参考资料

[Go实现一个MVC的小demo](https://learnku.com/articles/29546)
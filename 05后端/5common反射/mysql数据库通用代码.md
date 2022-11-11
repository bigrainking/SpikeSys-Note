



## 数据库查询通用代码



### 1. 常规查询写法

数据库查询多行结果需要

1. 执行查询语句获得rows 2. 再将rows中的每一行遍历读取到结构体中 rows.Scam(&...) 3. 再将每一行的内容添加到数组中

```go
rows, _ := db.Query("select * from product")
defer rows.Close()
productArry := []*datamodels.Product{}
for rows.Next() {
    product := datamodels.Product{}
    rows.Scan(&product.ID, &product.ProductName,....)
    productArry = append(productArry, &product)
}
```



**缺点**：

这样做代码高度内聚。 对于每次不同的多行查询，需要自己构建多个不同的结构体，同时每个查询的function中rows.Scan(&...)的内容都要重新写。

如果想修改一个查询结果，对应的结构体、Scan中的内容都要修改。

### 2. 通用代码的实现

**改进：**

因此，我们可以有一个通用的查询function：

- rows *sql.Rows : 执行查询后的结果

- 输出[]Map[string]interface{}：一个每行数据对应切片的数组集合：表示多行数据，每一行是一个Map，Key是列名，value是对应的值

  比如:

  ```go
  Products[
      map["ID":3, "name":"feizhuzhang", "badmintom":"尤尼克斯"], //第1行数据
      map["ID":4, "name":"feizhu样", "badmintom":"尤尼克斯++"], //第2行数据
      ......
  ]
  ```





```go
func GetResultRows(rows *sql.Rows) (dataMaps []map[string]string) {
	// 1. 查询到的数据列名、返回值
	columns, _ := rows.Columns() //列名
	count := len(columns)
	values, valuesPoints := make([][]byte, count), make([]interface{}, count)

	// 2. 遍历Rows读取每一行
	for rows.Next() {
		// for i, v := range values { // 读取value地址到valuePoints
		// 	valuesPoints[i] = &v
		// }
		for i := 0; i < count; i++ {
			valuesPoints[i] = &values[i]
		}

		// 2.1 数据库中读取出每一行数据
		rows.Scan(valuesPoints...) //将所有内容读取进values

		// 2.2 准备接收数据的结构体Product
		row := make(map[string]string)

		// 2.3 将读取到的数据填充到product
		for i, val := range values { // val是每个列对应的值
			key := columns[i] //列名

			// 列名与值对应
			row[key] = string(val)
		}

		// 将product归到集合中
		dataMaps = append(dataMaps, row)
	}
	return
}
```



[参考链接:golang操作数据库 - SQL查询结果集输出到Map切片](https://juejin.cn/post/7026932558348681253#heading-0)

### 













# 提出问题：

#### 为什么每个增删改查的函数都要链接一边数据库呢？

因为最开始是没有链接数据库的，比如当只需要add一个表时候，就会调用add函数，此前就没有链接数据库。因此每每有一个repository的关于数据的操作就要判断是否链接了数据库

```go
if err = p.Conn(); err != nil {
		return
	}
```


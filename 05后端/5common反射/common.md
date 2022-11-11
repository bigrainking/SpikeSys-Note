

# 1.反射：数据库中查询到的数据填充到struct

> **涉及知识点**
>
> - 反射
>
> **主要内容**
>
> - 对于从数据库中查询到的一条数据data，将数据中的每个字段填充到结构体中
> - data中字段和struct中字段的如何一一对应？ 结构体中的Tag对应结构体中的字段- 
>
> - 实际就是将Select中必经的一步改为通用的：
>
>   ```go
>   rows.Scan(&user.Id, &user.Username, &user.Sex, &user.Email)
>   ```



### 1. **实现**

- 1. 找到**struct中字段名**字`nameModel := typModel.Field(i).Name //比如Name Age， Field()直接显示这些字段对应的值`

- 2. 通过struct字段对应**Tag找到在data中的值**:Tag是data数据中的Key，Tag对应的值就是应该存到struct中的

  ```go
  dataValue := data[typModel.Field(i).Tag.Get("sql")]
  ```

- 3. 将找到data的值转换成struct字段对应的类型

     转换函数见下面

- 4. 找到对应的字段，并将data填入

  `.Set(x reflect.Value)`

  ```go
  valModel.FieldByName(nameModel).Set(dataValRef)
  ```

  

**实现代码**

```go
func DataToModelByTag(data map[string]string, model interface{}) {
	valModel := reflect.ValueOf(model).Elem()
	typModel := reflect.TypeOf(model).Elem()
	// 依次遍历ModelStruct的每个字段，将对应的data的字段填入
	num := valModel.NumField()
	for i := 0; i < num; i++ {
		// 1. 通过Model字段对应的Tag SQL找到data对应的值
		dataValue := data[typModel.Field(i).Tag.Get("sql")]
		// 2. Model字段对应名字
		nameModel := typModel.Field(i).Name //比如Name Age， Field()直接显示这些字段对应的值
		// 3. Model字段类型 & SQL查询结果类型是否匹配
		typeModel := valModel.Field(i).Type() //比如string、int对应的reflect.Type
		dataValRef := reflect.ValueOf(dataValue)
		if typeModel != dataValRef.Type() {
			// 类型转换：将data转换成Model字段对应的类型的reflect的类型
			TypeConversion(dataValue, typeModel.Name()) //。Name可以将Type转换成传统类型
		}
		// 4. 填入Model中
		valModel.FieldByName(nameModel).Set(dataValRef)
	}
}
```

**转换函数**

```go
//类型转换：将value转换成ntype类型，并返回value转换后对应reflect.Value类型的结果值
func TypeConversion(value string, ntype string) (reflect.Value, error) {
	// switch desType {
	// case "string":
	// 	return reflect.ValueOf(dataValue), nil
	// case "int":
	// 	res, err := strconv.Atoi(dataValue)
	// 	return reflect.ValueOf(res), err
	// case "int8":
	// 	i, err := strconv.ParseInt(dataValue, 10, 64)
	// 	return reflect.ValueOf(int8(i)), err
	// }
	if ntype == "string" {
		return reflect.ValueOf(value), nil
	} else if ntype == "time.Time" {
		t, err := time.ParseInLocation("2006-01-02 15:04:05", value, time.Local)
		return reflect.ValueOf(t), err
	} else if ntype == "Time" {
		t, err := time.ParseInLocation("2006-01-02 15:04:05", value, time.Local)
		return reflect.ValueOf(t), err
	} else if ntype == "int" {
		i, err := strconv.Atoi(value)
		return reflect.ValueOf(i), err
	} else if ntype == "int8" {
		i, err := strconv.ParseInt(value, 10, 64)
		return reflect.ValueOf(int8(i)), err
	} else if ntype == "int32" {
		i, err := strconv.ParseInt(value, 10, 64)
		return reflect.ValueOf(int64(i)), err
	} else if ntype == "int64" {
		i, err := strconv.ParseInt(value, 10, 64)
		return reflect.ValueOf(i), err
	} else if ntype == "float32" {
		i, err := strconv.ParseFloat(value, 64)
		return reflect.ValueOf(float32(i)), err
	} else if ntype == "float64" {
		i, err := strconv.ParseFloat(value, 64)
		return reflect.ValueOf(i), err
	}

	//else if .......增加其他一些类型的转换

	return reflect.ValueOf(value), errors.New("未知的类型：" + ntype)
}
```


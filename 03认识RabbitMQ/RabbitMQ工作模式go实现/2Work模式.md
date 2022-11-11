

# 一、工作模式

### 1. 理论介绍



> 1. Work模式相比 simple模式 消费者变多
> 2. 消息只会被消费一次，一个消息只能被一个消费者消费

<img src="pic/2Work%E6%A8%A1%E5%BC%8F.assets/image-20220803224808905.png" alt="image-20220803224808905" style="zoom:33%;" />

### 2. 使用场景

生产者速度 > 消费者速度 采用工作模式：





### 3. 实现代码

> **相比simple模式**：
>
> 生产者生产多个消息，生产速率＞消费速率；
>
> 消费者与simple模式的消费者相同；
>
> 消费者有2个；

<img src="pic/2Work%E6%A8%A1%E5%BC%8F.assets/image-20220803233025562.png" alt="image-20220803233025562" style="zoom:33%;" />

##### 生产端

```go
package main

import (
	"RabbitMQ/rabbitmq" //注意导入包 是rabbitmq文件夹
	"fmt"
	"strconv"
	"time"
)

// 生产者
func main() {
	// 创建rabbitMQ
	rabbitMQ := rabbitmq.NewRabbitMQsimple("rabbitSimple")
	// 发送消息到队列
	for i := 1; i < 101; i++ {
		rabbitMQ.PublishSimple("Hello I'm Producer" + strconv.Itoa(i) + "!")
		time.Sleep(1 * time.Second) // 每次发送了消息停歇一下
		fmt.Println(i)              //输出到terminal
	}
}
```



##### 消费端1

```go
package main

import (
	"RabbitMQ/rabbitmq"
)

func main() {
	// 创建队列
	rabbitmq := rabbitmq.NewRabbitMQsimple("rabbitSimple")
	// 消费message
	rabbitmq.ConsummerSimple()
}
```



##### 消费端2

```go
package main

import (
	"RabbitMQ/rabbitmq"
)

func main() {
	// 创建队列
	rabbitmq := rabbitmq.NewRabbitMQsimple("rabbitSimple")
	// 消费message
	rabbitmq.ConsummerSimple()
}
```





### 4. 运行结果

> 每个消息只被消费了一次；
>
> 两个消费者分别消费奇数 偶数；

###### 消费者1

```shell
 yang@VM  ~/go/src/RabbitMQ/RabbitMQWork  go run mainWorkConsummer1.go
2022/08/03 23:26:17 [*]Waiting for message， To Exit by ctrl+Enter
2022/08/03 23:26:40 Recive a message:Hello I'm Producer1!
2022/08/03 23:26:42 Recive a message:Hello I'm Producer3!
2022/08/03 23:26:44 Recive a message:Hello I'm Producer5!
2022/08/03 23:26:46 Recive a message:Hello I'm Producer7!
2022/08/03 23:26:48 Recive a message:Hello I'm Producer9!
2022/08/03 23:26:50 Recive a message:Hello I'm Producer11!
2022/08/03 23:26:52 Recive a message:Hello I'm Producer13!
2022/08/03 23:26:54 Recive a message:Hello I'm Producer15!
2022/08/03 23:26:56 Recive a message:Hello I'm Producer17!
.......
```



###### 消费者2

```shell
 yang@VM  ~/go/src/RabbitMQ/RabbitMQWork  go run mainWorkConsummer2.go
2022/08/03 23:26:30 [*]Waiting for message， To Exit by ctrl+Enter
2022/08/03 23:26:41 Recive a message:Hello I'm Producer2!
2022/08/03 23:26:43 Recive a message:Hello I'm Producer4!
2022/08/03 23:26:45 Recive a message:Hello I'm Producer6!
2022/08/03 23:26:47 Recive a message:Hello I'm Producer8!
2022/08/03 23:26:49 Recive a message:Hello I'm Producer10!
2022/08/03 23:26:51 Recive a message:Hello I'm Producer12!
2022/08/03 23:26:53 Recive a message:Hello I'm Producer14!
2022/08/03 23:26:55 Recive a message:Hello I'm Producer16!
2022/08/03 23:26:57 Recive a message:Hello I'm Producer18!
......
```



###### 生产者

```shell
 yang@VM  ~/go/src/RabbitMQ/RabbitMQWork  go run mainWorkPublish.go 
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
```




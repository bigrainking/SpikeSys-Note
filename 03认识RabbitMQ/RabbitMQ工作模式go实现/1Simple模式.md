[toc]





# 一、创建实例通用代码

URL

服务器默认端口是 5672

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803123444156.png" alt="image-20220803123444156" style="zoom:33%;" />



创建RabbitMQ基础实例

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803123650182.png" alt="image-20220803123650182" style="zoom:33%;" />

创建MQ实例

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803124644460.png" alt="image-20220803124644460" style="zoom:33%;" />



函数：断开连接， 如果不断开connection和channel将会一直被占用浪费服务器资源



<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803124738593.png" alt="image-20220803124738593" style="zoom:33%;" />



下面的只是方便处理错误的函数

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803125025753.png" alt="image-20220803125025753" style="zoom:33%;" />



```go
package rabbitmq

import (
	"fmt"
	"log"

	"github.com/streadway/amqp"
)

// MQ的URL
// URL固定的格式: amqp://账号：密码@rabbitmq服务器地址：端口号/vhost
const MQURL = "amqp://guest:guest@127.0.0.1:5672/rabbitVHost"

// 实例类
type RabbitMQ struct {
	// 连接channel
	conn    *amqp.Connection
	channel *amqp.Channel
	// 队列
	QueueName string
	// 交换机
	Exchange string
	// key
	Key string
	// URL
	Mqurl string
}

//RabbitMQ实例创建函数
func NewRabbitMQ(exchange, queueName, key string) *RabbitMQ {
	// 1. 返回一个创建的实例
	rabbitmq := &RabbitMQ{Exchange: exchange, QueueName: queueName, Key: key, Mqurl: MQURL}
	// 2. 建立连接
	var err error
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	rabbitmq.failOnErr(err, "建立连接出错")
	// 3. 获取channel
	rabbitmq.channel, err = rabbitmq.conn.Channel()
	rabbitmq.failOnErr(err, "获取channel失败")
	return rabbitmq
}

// 断开连接：类下面的功能function
func (r *RabbitMQ) Destory() {
	r.channel.Close()
	r.conn.Close()
}

// 错误处理:
// message是给人类看的错误原因
func (r *RabbitMQ) failOnErr(err error, message string) {
	if err != nil {
		log.Fatalf("%s:%s", err, message) //打印到log
		panic(fmt.Sprintf("%s:%s", err, message))
	}
}
```





# 二、 simple 模式

### 1. 理论图

最简单的模式：

1. 一个生产者、一个消费者
2. 每个消息只能被一个消费者消费

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803123205463.png" alt="image-20220803123205463" style="zoom:33%;" />





### 2. 创建 simple模式 代码

#### 1. 创建实例

exchange为空：使用默认的交换机，而不是没有交换机

```go
func NewRabbitMQsimple(queueName string) *RabbitMQ {
	// 使用默认exchange而不是没有交换机
	return NewRabbitMQ("", queueName, "")
}
```



<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803142511252.png" alt="image-20220803142511252" style="zoom:25%;" />

#### 2. 生产端发送消息

生产者and消费者都是首先申请队列



**主要步骤**

> 1. 创建队列
> 2. 发布消息



###### **1)申请队列**

```go
func (*amqp.Channel).QueueDeclare(name string, durable bool, autoDelete bool, exclusive bool, noWait bool, args amqp.Table) (amqp.Queue, error)
```

durable：bool：持久化； false消息进来会在队列里面，如果服务器重启就没有了

autoDeclare：是否自动删除：如果最后一个消费者断开以后，是否自动删除消息，默认否

exclusive：true 排他性，true表示创建一个只有自己可见的队列， 其他用户不能访问

noWait：是否阻塞 发送消息后是否等待服务器响应， false 表示需要等待响应



<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803161313106.png" alt="image-20220803161313106" style="zoom: 25%;" />



###### **2)发布消息**

mandatory：true 根据exchange的类型和routineKey来判断是否能找到符合条件的队列， 如果找不到就会将消息返还给生产者

immediate：true 如果消息发送到队列中，队列没有绑定消费者，那消息还是会返回给生产者

发送信息的类型限制

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803162237341.png" alt="image-20220803162237341" style="zoom: 25%;" />

```go
// Step2. 生产者:生产消息
func (r *RabbitMQ) PublishSimple(message string) {
	// 1. 申请队列
	_, err := r.channel.QueueDeclare(
		// 要申请队列的名称
		r.QueueName,
		// durable：bool 持久化； false消息进来会在队列里面，如果服务器重启就没有了
		false,
		// autoDelete bool,消费者自动断开后是否删除消息
		false,
		// 队列是否有排他性exclusive bool,
		false,
		// noWait bool是否阻塞，发送消息后阻塞，等待服务器回应
		false,
		// 额外参数
		nil,
	)
	if err != nil {
		fmt.Printf("%s:%s", err, "生产者申请队列失败")
	}

	// 2. 生产消息
	r.channel.Publish(
		// exchange string, 交换机
		r.Exchange,
		// key string, routingKey 指定要发送到的queue
		r.QueueName,
		// mandatory bool, true根据交换机androutineKey判断是否有合适的queue，如果没有返回message给生产者
		false,
		// immediate bool, true 队列没有消费者，则返回给生产者
		false,
		// msg amqp.Publishing:要发送的消息
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		},
	)
}
```





#### 3. 消费端消费消息



###### 1) 申请队列

与生产者一样

###### 2) 接受消息



consummer："" : 用来标识不同的消费者

autoAck = true ： 表示自动签收；消费者只要拿到消息，就把MQ中的消息删除

- 自动签收的弊端：如果消费者拿到message之后消费失败，就需要重试，重新向MQ获取消息，而如果设置自动签收，此时消息已经被删除了
- 解决方案：**手动签收**： 消费成功没有报错的情况下，才会签收消息，删除MQ中的消息

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803163058012.png" alt="image-20220803163058012" style="zoom:33%;" />



###### 3) 消费消息

chan forever用来阻塞

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803163639666.png" alt="image-20220803163639666" style="zoom: 33%;" />

```go
// Step3. 消费者：消费消息
func (r *RabbitMQ) ConsummerSimple() {
	// 1. 申请队列:与生产者队列相同，必须是同一个
	_, err := r.channel.QueueDeclare(
		// 要申请队列的名称
		r.QueueName,
		// durable：bool 持久化； false消息进来会在队列里面，如果服务器重启就没有了
		false,
		// autoDelete bool,消费者自动断开后是否删除消息
		false,
		// 队列是否有排他性exclusive bool,
		false,
		// noWait bool是否阻塞，发送消息后阻塞，等待服务器回应
		false,
		// 额外参数
		nil,
	)
	if err != nil {
		fmt.Printf("%s:%s", err, "消费者申请队列失败")
	}
	// 2. 接收消息
	msgs, err := r.channel.Consume(
		// queue string,
		r.QueueName,
		// consumer string, 消费者的名字：区分多个消费者
		"",
		// autoAck bool, 自动回应：true 收到消息立刻回应服务器删除消息，但如果消费失败，无法重新获取消息(消息已经被删除)
		true,
		// exclusive bool,排他性，队列仅自己可见
		false,
		// noLocal bool, true 表示不能将消息传递给本connection中的另一个消费者
		false, //当当前消费者阻塞，可以传递给其他消费者
		// noWait bool, 消费队列是否阻塞
		false,
		// args amqp.Table额外参数
		nil,
	)
	// 3. 消费消息
	// 阻塞，直到人工主动停止接收消息
	forever := make(chan bool)
	// 协程 一直不断接收消息
	go func() {
		for msg := range msgs {
			// 处理消息
			log.Printf("Recive a message:%s", msg.Body)
		}
	}()
	log.Printf("[*]Waiting for message， To Exit by ctrl+Enter")
	<-forever //channel中没有信息，一直阻塞，ctrl+c输入一个bool值到channel中，OK可以从channel中读出消息，执行完毕，程序结束
}
```





### 3. 运行

###### 发送端

```go
package main
import (
	"RabbitMQ/rabbitmq"
	"fmt"
)

// 生产者
func main() {
	// 创建rabbitMQ
	rabbitMQ := rabbitmq.NewRabbitMQsimple("rabbitSimple")
	// 发送消息到队列
	rabbitMQ.PublishSimple("Hello I'm Producer!")
	fmt.Println("成功发送message!")
}
```



###### 消费端

```go
package main

import (
	"RabbitMQ/rabbitmq"
)

func main() {
	// 创建队列
	rabbitmq := rabbitmq.NewRabbitMQsimple("rabbitSimple") //拉取消息的队列名称
	// 消费message
	rabbitmq.ConsummerSimple()
}
```





### 4. 运行结果

###### 发送端结果

```shell
 ~/go/src/RabbitMQ  go run mainSimplePublish.go
成功发送message!
```

###### 消费端结果

```shell
~/go/src/RabbitMQ  go run mainSimpleConsummer.go
2022/08/03 22:22:18 [*]Waiting for message， To Exit by ctrl+Enter
2022/08/03 22:22:58 Recive a message:Hello I'm Producer!
```



###### rabbitMQ管理界面

queue收到消息

<img src="image/工作模式.pic/image-20220803222531977.png" alt="image-20220803222531977" style="zoom: 25%;" />



<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803164849299.png" alt="image-20220803164849299" style="zoom: 33%;" />

两个端的queueName必须是同一个(发送到queue，从queue中获取消息)

发送端

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803163924144.png" alt="image-20220803163924144" style="zoom:33%;" />

消费端

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803164026312.png" alt="image-20220803164026312" style="zoom:33%;" />



先运行消费端，再运行发送端

![image-20220803164638118](pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803164638118.png)



管理界面

下面波峰图标识有消息进来

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803164718486.png" alt="image-20220803164718486" style="zoom:33%;" />



使用的默认交换机

<img src="pic/%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F.assets/image-20220803164821873.png" alt="image-20220803164821873" style="zoom:33%;" />



### 5. 创建Simple模式全部代码

```go
package rabbitmq

import (
	"fmt"
	"log"

	"github.com/streadway/amqp"
)

// MQ的URL
// URL固定的格式: amqp://账号：密码@rabbitmq服务器地址：端口号/vhost
const MQURL = "amqp://guest:guest@127.0.0.1:5672/rabbitVHost"

// 实例类
type RabbitMQ struct {
	// 连接channel
	conn    *amqp.Connection
	channel *amqp.Channel
	// 队列
	QueueName string
	// 交换机
	Exchange string
	// key
	Key string
	// URL
	Mqurl string
}

//RabbitMQ实例创建函数
func NewRabbitMQ(exchange, queueName, key string) *RabbitMQ {
	// 返回一个创建的实例
	rabbitmq := &RabbitMQ{Exchange: exchange, QueueName: queueName, Key: key, Mqurl: MQURL}
	// 建立连接
	var err error
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	rabbitmq.failOnErr(err, "建立连接出错")
	// 获取channel
	rabbitmq.channel, err = rabbitmq.conn.Channel()
	rabbitmq.failOnErr(err, "获取channel失败")
	return rabbitmq
}

// 断开连接：类下面的功能function
func (r *RabbitMQ) Destory() {
	r.channel.Close()
	r.conn.Close()
}

// 错误处理:
// message是给人类看的错误原因
func (r *RabbitMQ) failOnErr(err error, message string) {
	if err != nil {
		log.Fatalf("%s:%s", err, message) //打印到log
		panic(fmt.Sprintf("%s:%s", err, message))
	}
}

// =================simple=============
// Step1. 创建simple实例:简单实例只有队列
func NewRabbitMQsimple(queueName string) *RabbitMQ {
	// 使用默认exchange而不是没有交换机
	return NewRabbitMQ("", queueName, "")
}

// Step2. 生产者:生产消息
func (r *RabbitMQ) PublishSimple(message string) {
	// 1. 申请队列
	_, err := r.channel.QueueDeclare(
		// 要申请队列的名称
		r.QueueName,
		// durable：bool 持久化； false消息进来会在队列里面，如果服务器重启就没有了
		false,
		// autoDelete bool,消费者自动断开后是否删除消息
		false,
		// 队列是否有排他性exclusive bool,
		false,
		// noWait bool是否阻塞，发送消息后阻塞，等待服务器回应
		false,
		// 额外参数
		nil,
	)
	if err != nil {
		fmt.Printf("%s:%s", err, "生产者申请队列失败")
	}

	// 2. 生产消息
	r.channel.Publish(
		// exchange string, 交换机
		r.Exchange,
		// key string, routingKey 指定要发送到的queue
		r.QueueName,
		// mandatory bool, true根据交换机androutineKey判断是否有合适的queue，如果没有返回message给生产者
		false,
		// immediate bool, true 队列没有消费者，则返回给生产者
		false,
		// msg amqp.Publishing:要发送的消息
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		},
	)
}

// Step3. 消费者：消费消息
func (r *RabbitMQ) ConsummerSimple() {
	// 1. 申请队列:与生产者队列相同，必须是同一个
	_, err := r.channel.QueueDeclare(
		// 要申请队列的名称
		r.QueueName,
		// durable：bool 持久化； false消息进来会在队列里面，如果服务器重启就没有了
		false,
		// autoDelete bool,消费者自动断开后是否删除消息
		false,
		// 队列是否有排他性exclusive bool,
		false,
		// noWait bool是否阻塞，发送消息后阻塞，等待服务器回应
		false,
		// 额外参数
		nil,
	)
	if err != nil {
		fmt.Printf("%s:%s", err, "消费者申请队列失败")
	}
	// 2. 接收消息
	msgs, err := r.channel.Consume(
		// queue string,
		r.QueueName,
		// consumer string, 消费者的名字：区分多个消费者
		"",
		// autoAck bool, 自动回应：true 收到消息立刻回应服务器删除消息，但如果消费失败，无法重新获取消息(消息已经被删除)
		true,
		// exclusive bool,排他性，队列仅自己可见
		false,
		// noLocal bool, true 表示不能将消息传递给本connection中的另一个消费者
		false, //当当前消费者阻塞，可以传递给其他消费者
		// noWait bool, 消费队列是否阻塞
		false,
		// args amqp.Table额外参数
		nil,
	)
	// 3. 消费消息
	// 阻塞，直到人工主动停止接收消息
	forever := make(chan bool)
	// 协程 一直不断接收消息
	go func() {
		for msg := range msgs {
			// 处理消息
			log.Printf("Recive a message:%s", msg.Body)
		}
	}()
	log.Printf("[*]Waiting for message， To Exit by ctrl+Enter")
	<-forever //channel中没有信息，一直阻塞，ctrl+c输入一个bool值到channel中，OK可以从channel中读出消息，执行完毕，程序结束
}
```


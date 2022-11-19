[toc]



# 入门





1首先创建virtual host队列

<img src="pic/6RabbitMQ%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.assets/image-20220728161717133.png" alt="image-20220728161717133" style="zoom:25%;" /> <img src="image/6RabbitMQ快速入门.pic/image-20220728164303871.png" alt="image-20220728164303871" style="zoom: 33%;" /> 







<img src="image/6RabbitMQ快速入门.pic/image-20220728163910689.png" alt="image-20220728163910689" style="zoom:25%;" />



### 1 创建队列

1. 创建virtual host

<img src="image/6RabbitMQ快速入门.pic/image-20220728163815546.png" alt="image-20220728163815546" style="zoom:25%;" />

2. 创建队列

   <img src="image/6RabbitMQ快速入门.pic/image-20220728164036489.png" alt="image-20220728164036489" style="zoom:25%;" />

   选择队列所属分类

   <img src="image/6RabbitMQ快速入门.pic/image-20220728164223157.png" alt="image-20220728164223157" style="zoom:25%;" /> 



### 2 编写生产者代码

1. 创建链接

   指定virtual host

   设置账户密码

   MQ链接信息的地址

<img src="image/6RabbitMQ快速入门.pic/image-20220728164815486.png" alt="image-20220728164815486" style="zoom: 50%;" />

2. 生产者

   设置消息时， 包含交换机(这里我们没有交换机所以为null)、指定队列名称、消息内容

   ![image-20220728165643972](image/6RabbitMQ快速入门.pic/image-20220728165643972.png)

   



### 3 消费者

autoAck = true ： 表示自动签收；消费者只要拿到消息，就把MQ中的消息删除

- 自动签收的弊端：如果消费者拿到message之后消费失败，就需要重试，重新向MQ获取消息，而如果设置自动签收，此时消息已经被删除了
- 解决方案：**手动签收**： 消费成功没有报错的情况下，才会签收消息，删除MQ中的消息



![image-20220728172005242](image/6RabbitMQ快速入门.pic/image-20220728172005242.png)












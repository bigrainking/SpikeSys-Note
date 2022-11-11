



# 一、常见名词

virtual house ： 分类； 比如一个rabbitMQ处理的信息有很多，支付信息、个人账户管理信息，virtual house就是对应不同的分类。

queue ：队列； virtual house下面有多个queue处理信息。

Exchange ：交换机； 在每个virtual house有独立的交换机，将message分发到对应的队列中

# 二、后台管理系统介绍

**端口号**

<img src="pic/5MQ%E5%B8%B8%E8%A7%81%E5%90%8D%E8%AF%8D.assets/image-20220728161145682.png" alt="image-20220728161145682" style="zoom:25%;" />

<img src="pic/5MQ%E5%B8%B8%E8%A7%81%E5%90%8D%E8%AF%8D.assets/image-20220728161059819.png" alt="image-20220728161059819" style="zoom:33%;" />

创建用户and分类virtual house


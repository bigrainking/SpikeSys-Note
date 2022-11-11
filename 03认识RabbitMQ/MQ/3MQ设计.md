





<img src="pic/3MQ%E8%AE%BE%E8%AE%A1.assets/image-20220726185721344.png" alt="image-20220726185721344" style="zoom:25%;" />





<img src="pic/3MQ%E8%AE%BE%E8%AE%A1.assets/image-20220726185812238.png" alt="image-20220726185812238" style="zoom:33%;" />



[课程来源](https://www.bilibili.com/video/BV1dt4y1z7vG?p=6&spm_id_from=pageDriver&vd_source=47272764e1eb400edc65776bfe6a48af)



# MQ设计



**队列**

msgs是一个存放消息的队列



**生产者线程**

生产者产生data，并写入到msgs队列中

<img src="pic/3MQ%E8%AE%BE%E8%AE%A1.assets/image-20220726190247602.png" alt="image-20220726190247602" style="zoom: 50%;" />





**消费者线程**

一个不断从msgs中取出消息的死循环，直到msgs中没有消息为止

![image-20220726190633973](pic/3MQ%E8%AE%BE%E8%AE%A1.assets/image-20220726190633973.png)
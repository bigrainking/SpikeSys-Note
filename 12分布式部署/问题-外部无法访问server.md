# 主机无法访问网址



### **问题**

1. 主机与虚拟机可以相互ping通
2. 虚拟机浏览器可以访问网址 serverIP:port/index:  `http://10.10.10.128:8081/user/login`

3. 主机web浏览器无法访问网址



### **原因**

Linux开启了防火墙， server需要开启对应的端口



### **解决方法**

```bash
# 开启防火墙
> sudo ufw enable

# 开启端口
> sudo ufw allow 8081

# 查看已经开启的端口
> sudo ufw status
[sudo] password for yang: 
Status: active

To                         Action      From
--                         ------      ----
...                
8081                       ALLOW       Anywhere                  
...             
8081 (v6)                  ALLOW       Anywhere (v6)
```



### 注意

注意： 在远程Xshell中开启防火墙会出现下面的问题

[解决问题参考链接](https://www.cxyzjd.com/article/qq_36938617/95234909#_Toc13563177)

```bash
baby@baby:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```



表示：命令可能会中断现有的ssh连接。继续操作(y|n)?

因为是在远程的Xshell进行连接开启防火墙的，有的系统是没有将SSH的22端口设置为public的，所以会有这样的提示，这里分为两种情况，如果开启防火墙时在防火墙之中检测到22端口已添加为防火墙的开放端口，那么输入y继续操作以后，当前Xshell会自动断开连接；相反，如果开启防火墙时在防火墙之中没有检测到22端口，那么输入y继续操作以后22端口将会不再支持其他连接，只支持当前已有的这个连接，保持当前连接的原因是可以通过该连接开放22端口。

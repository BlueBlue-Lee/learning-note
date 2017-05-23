# iptables学习文档
---

## 1 概念

iptables是一个运行在**用户空间**的应用软件，通过控制Linux**内核netfilter模块**，来管理网络数据包的流动与转送。

需要用到ROOT权限。OSI模型的**二、三、四层**。


---

## 2 主要功能
- 防火墙
- NAT

---

## 3 基本结构
#### 3.1 三个表
- nat：prerouting、output、postrouting
- filter：input、forward、output
- mangle：具有五条链，并且表中的链在包的处理流程中处于比较优先的位置

#### 3.2 五条链
- prerouting：路由前经过这条链，设置mark以及DNAT
- input：主要是filter表，设置防火墙规则
- forward：主要是filter表，设置防火墙规则
- output：主要是filter表，设置防火墙规则
- postrouting：主要是nat表，设置SNAT

#### 3.3 术语解释
- SNAT--在postrouting链，修改包的源地址，通常是为了多个内网用户共享一个外网端口来访问外网    
- DNAT--在prerouting链，修改包的目的地址，通常是为了让外部访问内网中的服务

#### 3.4 状态跟踪机制

下面我们就说说我一直在上面提到的关于那个ESTABLISHED,RELATED的规则是怎么回事，到底有什么用处。

说这个东西就要简单说一下网络的数据通讯的方式，我们知道，网络的访问是双向的，也就是说一个Client与Server之间完成数据交换需要双方的发包与收包。在netfilter中，有几种状态，也就是new, established,related,invalid。

当一个客户端，在本文例一中，内网的一台机器访问外网，我们设置了规则允许他出去，但是没有设置允许回来的规则阿，怎么完成访问呢？这就是netfilter的 状态机制 ，当一个lan用户通过这个linux访问外网的时候，它发送了一个请求包，这个包的状态是new,当外网回包的时候他的状态就是established,所以，linux知道，哦，这个包是我的内网的一台机器发出去的应答包，他就放行了。

而外网试图对内发起一个新的连接的时候，他的状态是new,所以linux压根不去理会它。这就是我们为什么要加这一句的原因。

还有那个related,他是一个关联状态，什么会用到呢？tftp,ftp都会用到，因为他们的传输机制决定了，它不像http访问那样，Client_IP: port-->server:80然后server:80-->Client_IP:port，ftp使用tcp21建立连接，使用20端口发送数据，其中又有两种方式，一种主动active mode，一种被动passive mode。主动模式下，client使用port命令告诉server我用哪一个端口接受数据，然后server主动发起对这个端口的请求。被动模式下，server使用port命令告诉客户端，它用那个端口监听，然后客户端发起对他的数据传输，所以这对于一个防火墙来说就是比较麻烦的事情，因为有可能会有new状态的数据包，但是它又是合理的请求，这个时候就用到这个related状态了，他就是一种关联，在linux中，有个叫 ftp_conntrack的模块，它能识别port命令，然后对相应的端口进行放行。

另外，可以考虑一下SNAT，修改了源地址和源端口号。那么收到回复时怎么知道应该如何转给哪个地址和哪个端口呢？其实就是用到了连接跟踪机制conntrack。里面存储了很多的映射关系，保存了端口和地址等是如何转换的。


#### 3.4 工作流程图示

![image](https://github.com/shenlanse/study-matetial/blob/master/iptables/iptables_entables.png?raw=true)


![image](https://github.com/shenlanse/study-matetial/blob/master/iptables/iptables_netfilter_chains.png?raw=true)


---

## 4 常见应用场景

#### 4.1 简单的nat路由器
场景描述：    
内网中有一台机器，有内网IP 10.1.1.254/24 eth0和外网IP 60.1.1.1/24 eth1。如何控制内网中的其他机器（只有内网IP）通过这台机器访问外网的行为。

分析：    
- 首先将其他内网机器的网关设置为10.1.1.254
- 打开Linux转发功能，并且设置默认的转发策略为DROP：

```
sysctl net.ipv4.ip_forward=1

iptables -P FORWARD DROP
```
- 设置NAT规则

```
iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -j SNAT --to 60.1.1.1
```

- 假如允许10.1.1.9访问外网

```
iptables -A FORWARD -s 10.1.1.9 -j ACCEPT
```

- 假如只允许访问3.3.3.3这个ip

```
iptables -A FORWARD -d 3.3.3.3 -j ACCEPT
```

- 但是设置了上面的规则后，并不能正常工作，流量能正常的出去，但是回不来，还需要设置

```
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```
这条规则应该放到第一条，以提高效率。


#### 4.2 端口转发
场景描述：    
内网中有一台机器，有内网IP 10.1.1.254/24 eth0和外网IP 60.1.1.1/24 eth1。另一台内网机器上有一个webserver，地址为10.1.1.1:80。如何配置规则让外部可以访问到内网的web服务器。

分析：
- 打开Linux的转发功能

```
sysctl net.ipv4.ip_forward=1
```

- 设置默认转发策略

```
iptables -P FORWARD DROP
```

- 利用状态跟踪机制，允许ESTABLISHED,RELATED连接上的包通过

```
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

- 设置DNAT

```
iptables -t nat -A PREROUTING -d 60.1.1.1 -p tcp --dport 80 -j DNAT --to 10.1.1.1:80
```
- 设置Forward规则，允许某类型的包通过

```
iptables -A FORWARD -d 10.1.1.1 -p tcp --dport 80 -j ACCEPT
```

- 现在仍然不能正常工作，外部流量可以到达webserver，但是webserver并不知道如何将流量转发出去，有两种方式解决：
    - 设置webserver的默认网关为10.1.1.254
    - 在转发机器上设置SNAT
    
```
iptables -t nat -A POSTROUTING -d 10.1.1.1 -p tcp --dport 80 -j SNAT --to 10.1.1.254
```

#### 4.3 如何让外部访问kubernetes的8080端口
场景描述：    
为了安全考虑，非安全端口8080是bind到localhost的。那么如何设置iptables规则，让外部能访问8080端口呢。

分析：
- 设置DNAT

```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 127.0.0.1:8080
```

- 允许外部访问localhost

```
sysctl -w net.ipv4.conf.all.route_localnet=1
```
该设置的解释：
[Redirect to localhost](https://unix.stackexchange.com/questions/111433/iptables-redirect-outside-requests-to-127-0-0-1/112232#112232)


---

## 5 在kubernetes中的应用



---

## 6 存在的缺点
请参考： [缺点](http://www.chinaunix.net/old_jh/4/828490.html)


---


## 参考文献
> 1. http://xstarcd.github.io/wiki/Linux/iptables_forward_internetshare.html
> 2. http://blog.csdn.net/zm_21/article/details/8508653
> 3. http://www.cnblogs.com/bangerlee/archive/2013/02/27/2935422.html
> 4. https://my.oschina.net/u/156249/blog/32238
> 5. http://blog.chinaunix.net/uid-13423994-id-3212414.html
> 6. https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Firewalls-IPTables_and_Connection_Tracking.html
> 7. http://brokestream.com/iptables.html
> 8. http://bbs.chinaunix.net/thread-1926255-1-1.html
> 9. http://andys.org.uk/bits/2010/01/27/iptables-fun-with-mark/
> 10. http://lesca.me/archives/iptables-examples.html
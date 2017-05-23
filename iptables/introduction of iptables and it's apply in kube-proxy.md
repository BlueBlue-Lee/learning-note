# iptablesѧϰ�ĵ�
---

## 1 ����

iptables��һ��������**�û��ռ�**��Ӧ�������ͨ������Linux**�ں�netfilterģ��**���������������ݰ���������ת�͡�

��Ҫ�õ�ROOTȨ�ޡ�OSIģ�͵�**���������Ĳ�**��


---

## 2 ��Ҫ����
- ����ǽ
- NAT

---

## 3 �����ṹ
#### 3.1 ������
- nat��prerouting��output��postrouting
- filter��input��forward��output
- mangle�����������������ұ��е����ڰ��Ĵ��������д��ڱȽ����ȵ�λ��

#### 3.2 ������
- prerouting��·��ǰ����������������mark�Լ�DNAT
- input����Ҫ��filter�����÷���ǽ����
- forward����Ҫ��filter�����÷���ǽ����
- output����Ҫ��filter�����÷���ǽ����
- postrouting����Ҫ��nat������SNAT

#### 3.3 �������
- SNAT--��postrouting�����޸İ���Դ��ַ��ͨ����Ϊ�˶�������û�����һ�������˿�����������    
- DNAT--��prerouting�����޸İ���Ŀ�ĵ�ַ��ͨ����Ϊ�����ⲿ���������еķ���

#### 3.4 ״̬���ٻ���

�������Ǿ�˵˵��һֱ�������ᵽ�Ĺ����Ǹ�ESTABLISHED,RELATED�Ĺ�������ô���£�������ʲô�ô���

˵���������Ҫ��˵һ�����������ͨѶ�ķ�ʽ������֪��������ķ�����˫��ģ�Ҳ����˵һ��Client��Server֮��������ݽ�����Ҫ˫���ķ������հ�����netfilter�У��м���״̬��Ҳ����new, established,related,invalid��

��һ���ͻ��ˣ��ڱ�����һ�У�������һ̨�����������������������˹�����������ȥ������û��������������Ĺ��򰢣���ô��ɷ����أ������netfilter�� ״̬���� ����һ��lan�û�ͨ�����linux����������ʱ����������һ����������������״̬��new,�������ذ���ʱ������״̬����established,���ԣ�linux֪����Ŷ����������ҵ�������һ̨��������ȥ��Ӧ��������ͷ����ˡ�

��������ͼ���ڷ���һ���µ����ӵ�ʱ������״̬��new,����linuxѹ����ȥ����������������ΪʲôҪ����һ���ԭ��

�����Ǹ�related,����һ������״̬��ʲô���õ��أ�tftp,ftp�����õ�����Ϊ���ǵĴ�����ƾ����ˣ�������http����������Client_IP: port-->server:80Ȼ��server:80-->Client_IP:port��ftpʹ��tcp21�������ӣ�ʹ��20�˿ڷ������ݣ������������ַ�ʽ��һ������active mode��һ�ֱ���passive mode������ģʽ�£�clientʹ��port�������server������һ���˿ڽ������ݣ�Ȼ��server�������������˿ڵ����󡣱���ģʽ�£�serverʹ��port������߿ͻ��ˣ������Ǹ��˿ڼ�����Ȼ��ͻ��˷�����������ݴ��䣬���������һ������ǽ��˵���ǱȽ��鷳�����飬��Ϊ�п��ܻ���new״̬�����ݰ������������Ǻ�����������ʱ����õ����related״̬�ˣ�������һ�ֹ�������linux�У��и��� ftp_conntrack��ģ�飬����ʶ��port���Ȼ�����Ӧ�Ķ˿ڽ��з��С�

���⣬���Կ���һ��SNAT���޸���Դ��ַ��Դ�˿ںš���ô�յ��ظ�ʱ��ô֪��Ӧ�����ת���ĸ���ַ���ĸ��˿��أ���ʵ�����õ������Ӹ��ٻ���conntrack������洢�˺ܶ��ӳ���ϵ�������˶˿ں͵�ַ�������ת���ġ�


#### 3.4 ��������ͼʾ

![image](https://github.com/shenlanse/study-matetial/blob/master/iptables/iptables_entables.png?raw=true)


![image](https://github.com/shenlanse/study-matetial/blob/master/iptables/iptables_netfilter_chains.png?raw=true)


---

## 4 ����Ӧ�ó���

#### 4.1 �򵥵�nat·����
����������    
��������һ̨������������IP 10.1.1.254/24 eth0������IP 60.1.1.1/24 eth1����ο��������е�����������ֻ������IP��ͨ����̨����������������Ϊ��

������    
- ���Ƚ�����������������������Ϊ10.1.1.254
- ��Linuxת�����ܣ���������Ĭ�ϵ�ת������ΪDROP��

```
sysctl net.ipv4.ip_forward=1

iptables -P FORWARD DROP
```
- ����NAT����

```
iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -j SNAT --to 60.1.1.1
```

- ��������10.1.1.9��������

```
iptables -A FORWARD -s 10.1.1.9 -j ACCEPT
```

- ����ֻ�������3.3.3.3���ip

```
iptables -A FORWARD -d 3.3.3.3 -j ACCEPT
```

- ��������������Ĺ���󣬲��������������������������ĳ�ȥ�����ǻز���������Ҫ����

```
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```
��������Ӧ�÷ŵ���һ���������Ч�ʡ�


#### 4.2 �˿�ת��
����������    
��������һ̨������������IP 10.1.1.254/24 eth0������IP 60.1.1.1/24 eth1����һ̨������������һ��webserver����ַΪ10.1.1.1:80��������ù������ⲿ���Է��ʵ�������web��������

������
- ��Linux��ת������

```
sysctl net.ipv4.ip_forward=1
```

- ����Ĭ��ת������

```
iptables -P FORWARD DROP
```

- ����״̬���ٻ��ƣ�����ESTABLISHED,RELATED�����ϵİ�ͨ��

```
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

- ����DNAT

```
iptables -t nat -A PREROUTING -d 60.1.1.1 -p tcp --dport 80 -j DNAT --to 10.1.1.1:80
```
- ����Forward��������ĳ���͵İ�ͨ��

```
iptables -A FORWARD -d 10.1.1.1 -p tcp --dport 80 -j ACCEPT
```

- ������Ȼ���������������ⲿ�������Ե���webserver������webserver����֪����ν�����ת����ȥ�������ַ�ʽ�����
    - ����webserver��Ĭ������Ϊ10.1.1.254
    - ��ת������������SNAT
    
```
iptables -t nat -A POSTROUTING -d 10.1.1.1 -p tcp --dport 80 -j SNAT --to 10.1.1.254
```

#### 4.3 ������ⲿ����kubernetes��8080�˿�
����������    
Ϊ�˰�ȫ���ǣ��ǰ�ȫ�˿�8080��bind��localhost�ġ���ô�������iptables�������ⲿ�ܷ���8080�˿��ء�

������
- ����DNAT

```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 127.0.0.1:8080
```

- �����ⲿ����localhost

```
sysctl -w net.ipv4.conf.all.route_localnet=1
```
�����õĽ��ͣ�
[Redirect to localhost](https://unix.stackexchange.com/questions/111433/iptables-redirect-outside-requests-to-127-0-0-1/112232#112232)


---

## 5 ��kubernetes�е�Ӧ��



---

## 6 ���ڵ�ȱ��
��ο��� [ȱ��](http://www.chinaunix.net/old_jh/4/828490.html)


---


## �ο�����
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
# tcpdump监听网络流量

[TOC]

## 1. tcpdump是否能抓到包  -  iptables限制 

    如果添加的入站规则则可以抓得到包
    
    如果是添加出站规则抓不到包
    
    其顺序如下：

```
进来的顺序 Wire -> NIC -> tcpdump -> netfilter/iptables
出去的顺序 iptables -> tcpdump -> NIC -> Wire
```

## 2. 命令详解

 - 选项说明

    - 介绍

    tcpdump命令是一个截获网络数据包的包分析工具。tcpdump可以将网络中传送的数据包的"头"完全截获下来以提供分析。它支持针对网络层、协议层、主机、端口等的过滤，并支持与、或、非逻辑语句协助过滤有效信息。
     tcpdump命令工作时要先把网卡的工作模式切换到混杂模式(promiscuous mode)。因为要修改网络接口的工作模式。所以tcpdump命令需要以root的身份运行

    - 语法格式

    ``` shell
    tcpdump  [option]  [expression]
    tcpdump  [选项]  [表达式]
    ```

    - 说明

    在tcpdump命令及后面的选项和表达式里，每个元素之间都至少要有一个空格。

    | 参数选项        | 解释说明(带*的为重点)                                        |
    | --------------- | ------------------------------------------------------------ |
    | -A              | 以ASCII码的方式显示每一个数据包(不会显示数据包中的链路层的头部嘻嘻)。在抓取包含网页数据的数据包时，可方便查看数据 |
    | -c <数据包数目> | 接收到指定的数据包数目后退出命令                             |
    | -e              | 每行的打印输出中将包括数据包的数据链路层头部信息             |
    | -i <网络接口>   | 指定要监听数据包的网络接口*                                  |
    | -n              | 不进行DNS解析，加快显示速度*                                 |
    | -nn             | 不将协议和端口数字等转换成名字*                              |
    | -q              | 以快速输出的方式运行，此选项仅显示数据包的协议概要信                  息，输出信息较短* |
    | -s <数据包大小> | 设置数据包抓取长度，如果不设置则默认为68字节，设置为0则                  自动选择合适长度来抓取数据包 |
    | -t              | 在每行输出信息中不显示时间戳标记                             |
    | -tt             | 在每行输出信息汇总不显示无格式的时间戳标记                   |
    | -ttt            | 显示当前行与前一行的延迟                                     |
    | -tttt           | 在每行打印的时间戳之前添加日志                               |
    | -ttttt          | 显示当前行与第一行的延迟                                     |
    | -v              | 显示命令执行的详细信息                                       |
    | -vv             | 显示比-v选项更佳详细的信息                                   |
    | -vvv            | 显示                                                         |
    |                 |                                                              |
    |                 |                                                              |
    |                 |                                                              |


### tcpdump 命令安装

    ``` shell
    yum -y install tcpdump
    ```


## 3. 使用范围

###  3.1. 不参加参数运行tcpdump命令监听网络

    ```shell
    [root@localhost /]# tcpdump          #<==默认情况下，直接启动tcpdump将监视第一个网络接口上所以流过的数据包
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    14:35:03.112228 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 2923122276:2923122460, ack 3597148564, win 585, options [nop,nop,TS val 5261893 ecr 1045340817], length 184
    14:35:03.112347 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 184, win 4090, options [nop,nop,TS val 1045340914 ecr 5261893], length 0
    14:35:03.112605 IP 192.168.121.129.50800 > 192.168.121.2.domain: 43519+ PTR? 1.121.168.192.in-addr.arpa. (44)
    14:35:03.125630 ARP, Request who-has 192.168.121.129 tell 192.168.121.2, length 46
    14:35:03.125638 ARP, Reply 192.168.121.129 is-at 00:0c:29:63:29:db (oui Unknown), length 28
    14:35:03.125690 IP 192.168.121.2.domain > 192.168.121.129.50800: 43519 NXDomain*- 0/0/0 (44)
    14:35:03.125817 IP 192.168.121.129.57593 > 192.168.121.2.domain: 48319+ PTR? 129.121.168.192.in-addr.arpa. (46)
    14:35:03.165269 IP 192.168.121.2.domain > 192.168.121.129.57593: 48319 NXDomain*- 0/0/0 (46)
    14:35:03.165625 IP 192.168.121.129.35799 > 192.168.121.2.domain: 26078+ PTR? 2.121.168.192.in-addr.arpa. (44)
    14:35:03.166321 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 184:560, ack 1, win 585, options [nop,nop,TS val 5261947 ecr 1045340914], length 376
    14:35:03.166562 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 560, win 4084, options [nop,nop,TS val 1045340967 ecr 5261947], length 0
    14:35:03.178026 IP 192.168.121.2.domain > 192.168.121.129.35799: 26078 NXDomain*- 0/0/0 (44)
    14:35:03.179413 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 560:1704, ack 1, win 585, options [nop,nop,TS val 5261960 ecr 1045340967], length 1144
    14:35:03.179594 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 1704, win 4060, options [nop,nop,TS val 1045340980 ecr 5261960], length 0
    ^C
    97 packets captured
    97 packets received by filter
    0 packets dropped by kernel
    ```
### 3.2.使用tcpdump命令时，如果不输入过来规则，则输出的数据量将会很大

- 精简输出信息
    ```shell
    [root@localhost /]# tcpdump -q        #<==默认情况下，tcpdump命令的输出信息较多，为了显示精简的信息，可以使用-q选项
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    14:38:26.706220 IP 192.168.121.129.ssh > 192.168.121.1.51078: tcp 184
    14:38:26.706323 IP 192.168.121.1.51078 > 192.168.121.129.ssh: tcp 0
    14:38:26.706655 IP 192.168.121.129.51379 > 192.168.121.2.domain: UDP, length 44
    14:38:26.720091 IP 192.168.121.2.domain > 192.168.121.129.51379: UDP, length 44
    14:38:26.720449 IP 192.168.121.129.43737 > 192.168.121.2.domain: UDP, length 46
    14:38:26.753629 IP 192.168.121.2.domain > 192.168.121.129.43737: UDP, length 46
    14:38:26.753879 IP 192.168.121.129.41780 > 192.168.121.2.domain: UDP, length 44
    14:38:26.754338 IP 192.168.121.129.ssh > 192.168.121.1.51078: tcp 168
    14:38:26.754564 IP 192.168.121.1.51078 > 192.168.121.129.ssh: tcp 0
    14:38:26.766716 IP 192.168.121.2.domain > 192.168.121.129.41780: UDP, length 44
    14:38:26.767268 IP 192.168.121.129.ssh > 192.168.121.1.51078: tcp 664
    14:38:26.767547 IP 192.168.121.1.51078 > 192.168.121.129.ssh: tcp 0
    14:38:26.768272 IP 192.168.121.129.ssh > 192.168.121.1.51078: tcp 168
    14:38:26.768435 IP 192.168.121.1.51078 > 192.168.121.129.ssh: tcp 0
    803 packets captured
    803 packets received by filter
    0 packets dropped by kernel
    ```
    ```shell
    [root@localhost /]# tcpdump -c 5    #<==使用-c选项指定监听的数据包数量，这样就不需要使用Ctrl+C了。
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    14:47:54.835455 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 2924737100:2924737284, ack 3597158404, win 585, options [nop,nop,TS val 6033616 ecr 1046110972], length 184
    14:47:54.835645 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 184, win 4090, options [nop,nop,TS val 1046111182 ecr 6033616], length 0
    14:47:54.836241 IP 192.168.121.129.52725 > 192.168.121.2.domain: 41821+ PTR? 1.121.168.192.in-addr.arpa. (44)
    14:47:54.846751 ARP, Request who-has 192.168.121.129 tell 192.168.121.2, length 46
    14:47:54.846765 ARP, Reply 192.168.121.129 is-at 00:0c:29:63:29:db (oui Unknown), length 28
    5 packets captured
    14 packets received by filter
    0 packets dropped by kernel
    ```

### 3.3.监听制定网卡收到的数据包

``` shell
[root@localhost /]# tcpdump -i eth0        #<==使用-i选项可以指定要监听的网卡
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
14:42:16.383512 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 2924691140:2924691324, ack 3597156644, win 585, options [nop,nop,TS val 5695164 ecr 1045773040], length 184
14:42:16.383746 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 184, win 4090, options [nop,nop,TS val 1045773052 ecr 5695164], length 0
14:42:16.384243 IP 192.168.121.129.47300 > 192.168.121.2.domain: 39880+ PTR? 1.121.168.192.in-addr.arpa. (44)
14:42:16.397463 IP 192.168.121.2.domain > 192.168.121.129.47300: 39880 NXDomain*- 0/0/0 (44)
14:42:16.397628 IP 192.168.121.129.49148 > 192.168.121.2.domain: 8740+ PTR? 129.121.168.192.in-addr.arpa. (46)
14:42:16.429776 IP 192.168.121.2.domain > 192.168.121.129.49148: 8740 NXDomain*- 0/0/0 (46)
14:42:16.430063 IP 192.168.121.129.48011 > 192.168.121.2.domain: 54037+ PTR? 2.121.168.192.in-addr.arpa. (44)
14:42:16.430268 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 184:560, ack 1, win 585, options [nop,nop,TS val 5695211 ecr 1045773052], length 376
14:42:16.430475 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 560, win 4084, options [nop,nop,TS val 1045773098 ecr 5695211], length 0
14:42:16.444533 IP 192.168.121.2.domain > 192.168.121.129.48011: 54037 NXDomain*- 0/0/0 (44)
14:42:16.445293 IP 192.168.121.129.ssh > 192.168.121.1.51078: Flags [P.], seq 560:1528, ack 1, win 585, options [nop,nop,TS val 5695226 ecr 1045773098], length 968
14:42:16.445427 IP 192.168.121.1.51078 > 192.168.121.129.ssh: Flags [.], ack 1528, win 4065, options [nop,nop,TS val 1045773113 ecr 5695226], length 0
^C
240 packets captured
240 packets received by filter
0 packets dropped by kernel
```
- 以下是命令结果。

    **14:42:16.383512:当前时间，精确到微秒。
    IP 192.168.121.129.ssh > 192.168.121.1.51078：从主机192.168.121.129的SSH端口发送数据到192.168.121.1的51078端口，">" 代表数据流向。
    Flags [P.]: TCP包中的标识信息，S是SYN标志的缩写，F(FIN)、P(PUSH)、R(RST)、"."(没有标记)。
    seq：数据包粽的数据的顺序号。
    ack:下次期望的顺序号。
    win:接收缓存的窗口大小。
    length:数据包的长度。**

### 3.4.建通指定主机的数据包

```shell
[root@localhost /]# tcpdump -n host 192.168.121.131        #<==使用-n选项不进行DNS解析，加快显示速度。监听指定主机的关键字为host，后面直接主机名或IP地址即可。本行命令的作用是监听所有192.168.121.131的主机收到的和发出的数据包。
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
14:59:08.905381 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 2947015398:2947015438, ack 4084885214, win 65535, length 40
14:59:08.907692 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 1:41, ack 40, win 25776, length 40
14:59:08.907701 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 41, win 65535, length 0
14:59:09.032865 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 40:80, ack 41, win 65535, length 40
14:59:09.034680 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 41:81, ack 80, win 25776, length 40
14:59:09.034687 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 81, win 65535, length 0
14:59:09.177312 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 80:120, ack 81, win 65535, length 40
14:59:09.179493 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 81:121, ack 120, win 25776, length 40
14:59:09.179500 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 121, win 65535, length 0
14:59:09.181494 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 121:1057, ack 120, win 25776, length 936
14:59:09.181501 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 1057, win 65535, length 0
^C
11 packets captured
11 packets received by filter
0 packets dropped by kernel
```

- 

```shell
[root@localhost /]# tcpdump -n  src host 192.168.121.131       #<==只监听从192.168.121.131发出的数据包，即源地址为192.168.121.131,关键字为src(source,源地址)。
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:01:38.460558 ARP, Reply 192.168.121.131 is-at 00:0c:29:9e:a9:d7, length 46
15:01:38.462523 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 4084886270:4084886390, ack 2947015558, win 25776, length 120
15:01:38.874518 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 120:160, ack 41, win 25776, length 40
15:01:39.022528 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 160:200, ack 81, win 25776, length 40
15:01:39.148409 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 200:240, ack 121, win 25776, length 40
15:01:39.150529 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 240:1176, ack 121, win 25776, length 936
15:01:44.946040 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 1176:1216, ack 161, win 25776, length 40
15:01:45.087128 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 1216:1256, ack 201, win 25776, length 40
15:01:45.340033 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 1256:1296, ack 241, win 25776, length 40
15:01:45.528735 IP 192.168.121.131.ssh > 192.168.121.1.51626: Flags [P.], seq 1296:1432, ack 281, win 25776, length 136
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```



```shell
[root@localhost /]# tcpdump -n  dst host 192.168.121.131     #<==只监听192.168.121.131收到的数据包，即目标地址为192.168.121.131,关键字为dst(destination,目的地)。
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:03:54.869394 ARP, Request who-has 192.168.121.131 (00:0c:29:9e:a9:d7) tell 192.168.121.1, length 46
15:03:54.869408 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 2947015838:2947015878, ack 4084887702, win 65535, length 40
15:03:54.872671 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 121, win 65535, length 0
15:03:55.067330 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 40:80, ack 121, win 65535, length 40
15:03:55.069563 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 241, win 65535, length 0
15:03:55.364657 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 80:120, ack 241, win 65535, length 40
15:03:55.366673 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 281, win 65535, length 0
15:03:55.504578 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 120:160, ack 281, win 65535, length 40
15:03:55.506674 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 321, win 65535, length 0
15:03:55.642867 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [P.], seq 160:200, ack 321, win 65535, length 40
15:03:55.645251 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 361, win 65535, length 0
15:03:55.649562 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 1217, win 65535, length 0
15:03:55.650650 IP 192.168.121.1.51626 > 192.168.121.131.ssh: Flags [.], ack 1337, win 65535, length 0
^C
13 packets captured
13 packets received by filter
0 packets dropped by kernel
```

### 3.5. 监听指定端口的数据包

```shell
[root@localhost /]# tcpdump -nn port 22        #<==使用-n选项不进行DNS解析，但是会将一些协议、端口转换，比如22端口转为ssh，监听指定端口的关键字是port，后面接上端口号即可。
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:06:54.965260 IP 192.168.121.129.22 > 192.168.121.1.51078: Flags [P.], seq 2925446556:2925446740, ack 3597172100, win 585, options [nop,nop,TS val 7173746 ecr 1047249445], length 184
15:06:54.965490 IP 192.168.121.1.51078 > 192.168.121.129.22: Flags [.], ack 184, win 4090, options [nop,nop,TS val 1047249517 ecr 7173746], length 0
15:06:54.966443 IP 192.168.121.129.22 > 192.168.121.1.51078: Flags [P.], seq 184:560, ack 1, win 585, options [nop,nop,TS val 7173747 ecr 1047249517], length 376
15:06:54.966676 IP 192.168.121.1.51078 > 192.168.121.129.22: Flags [.], ack 560, win 4084, options [nop,nop,TS val 1047249518 ecr 7173747], length 0
15:06:54.967317 IP 192.168.121.129.22 > 192.168.121.1.51078: Flags [P.], seq 560:904, ack 1, win 585, options [nop,nop,TS val 7173748 ecr 1047249518], length 344
15:06:54.967500 IP 192.168.121.1.51078 > 192.168.121.129.22: Flags [.], ack 904, win 4085, options [nop,nop,TS val 1047249518 ecr 7173748], length 0
15:06:54.968443 IP 192.168.121.129.22 > 192.168.121.1.51078: Flags [P.], seq 904:1248, ack 1, win 585, options [nop,nop,TS val 7173749 ecr 1047249518], length 344
^C
742 packets captured
742 packets received by filter
0 packets dropped by kernel
```



### 3.6. 监听指定协议的数据包

```


​```shell

[root@localhost /]# tcpdump -n arp        #<==监听ARP数据包，因此表达式直接写arp即可。
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:11:01.898359 ARP, Request who-has 192.168.121.129 tell 192.168.121.2, length 46
15:11:01.898386 ARP, Reply 192.168.121.129 is-at 00:0c:29:63:29:db, length 28
15:11:06.899494 ARP, Request who-has 192.168.121.2 tell 192.168.121.129, length 28
15:11:06.900038 ARP, Reply 192.168.121.2 is-at 00:50:56:fc:3a:9a, length 46
^C
4 packets captured
4 packets received by filter
0 packets dropped by kernel


[root@localhost /]# tcpdump -n icmp        #<==监听icmp数据包(想要查看下面的监控数据，可以使用其他服务器ping本机即可)
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:12:30.533308 IP 192.168.121.131 > 192.168.121.129: ICMP echo request, id 13481, seq 1, length 64
15:12:30.533330 IP 192.168.121.129 > 192.168.121.131: ICMP echo reply, id 13481, seq 1, length 64
15:12:31.535165 IP 192.168.121.131 > 192.168.121.129: ICMP echo request, id 13481, seq 2, length 64
15:12:31.535182 IP 192.168.121.129 > 192.168.121.131: ICMP echo reply, id 13481, seq 2, length 64
15:12:32.537233 IP 192.168.121.131 > 192.168.121.129: ICMP echo request, id 13481, seq 3, length 64
15:12:32.537253 IP 192.168.121.129 > 192.168.121.131: ICMP echo reply, id 13481, seq 3, length 64
15:12:33.537889 IP 192.168.121.131 > 192.168.121.129: ICMP echo request, id 13481, seq 4, length 64
15:12:33.537912 IP 192.168.121.129 > 192.168.121.131: ICMP echo reply, id 13481, seq 4, length 64
15:12:34.540105 IP 192.168.121.131 > 192.168.121.129: ICMP echo request, id 13481, seq 5, length 64
15:12:34.540129 IP 192.168.121.129 > 192.168.121.131: ICMP echo reply, id 13481, seq 5, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

### 3.7.多个过滤条件混合使用

- 前面的几种方法都是使用单个过滤条件过滤数据包，其实过滤条件可以混合使用，因为tcpdump命令支持逻辑运算and(与)、or(或)、!(非)。
```shell

[root@localhost /]# tcpdump -n ip host 192.168.121.129 and ! 192.168.121.1   #<==获取主机192.168.121.139(tcpdump主机)(除了主机192.168.121.1之外)通信的IP数据包。
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
15:23:15.489964 IP 192.168.121.131 > 192.168.121.129: ICMP echo request, id 19369, seq 1, length 64
15:23:15.489991 IP 192.168.121.129 > 192.168.121.131: ICMP echo reply, id 19369, seq 1, length 64
^C
2 packets captured
2 packets received by filter
0 packets dropped by kernel
复制代码  
```
### 3.8. 利用tempdump 抓包详解tcp/ip链接和端口号
一、正常的TCP连接的三个阶段。

TCP三次握手

数据传送

TCP四次断开

二、TCP三次握手与四次挥手

TCP连接的状态机制图见：https://www.cnblogs.com/hwlong/p/9060693.html

三、TCP状态标识

SYN：(同步序列编号，Synchronize Sequence Numbers)该标志仅在三次握手建立TCP连接时有效。表示一个新的TCP连接请求。
ACK：(确认编号，Acknowledement Number)是对TCP请求的确认标志，同时提示对端系统已经成功接受了所有数据。
FIN：(结束标志，FINish)用来结束一个TCP回话。但对应端仍然处于开放状态，准备接受后续数据。
四、使用tcpdump对tcp数据进行抓包
## 


Title: 服务器双线配置
Date: 2014-08-27
Category: other
Tag: linux, 双线配置


  Linux服务器在有电信、联通双线接入的情况下，因为默认网关只有一个，可能会导致一条线路不通的情况，因此需要对路由进行配置。下面简单介绍下配置流程。

### 配置工具
  
  工欲善其事，必先利其器，针对策略路由的配置，使用iproute2，大部分的linux系统已经安装。

### 配置过程
    假如我们网卡的配置情况如下：
     1、电信网卡：
          name:eth0
          IP:$IP0
          网段:$NET0
          网关:$GW0
     2、联通网卡
           name:eth1
           IP:$IP1
           网段:$NET1
           网关:$GW1

1、增加路由表
    我们分别为电信和联通增加两个路由表，后面的具体的路由条目加入到这两个表中，配置方法如下：
    
    vi /etc/iproute2/rt_tables
    #增加如下两行
    252 tel
    351 cnc

2、增加路由条目  

     ip route add $NET0 dev eth0 src $IP0 table tel
     ip route add default via $GW0 table tel
     ip route add $NET1 dev eth1 src $IP1 table cnc
     ip route add default via $GW1 table cnc

3、增加源地址策略路由
    源地址策略路由，就是将从该网卡进入的数据回应从该网卡返回。

    ip rule add from $IP0 table tel
    ip rule add from $IP1 table cnc

  完成以上配置后，系统的双网卡IP应该均可以ping通。但是重启后这些命令会失效，因此可以将这些命令写入到/etc/rc.local中，在每次重启后都能够生效。

**Tips**

  1、如果服务器只启动了一块网卡，而且另一块网卡没有写入配置和启用。如果采用写入网卡的config文件中，需要重启网络服务，这样通过ssh与服务器的连接也会down掉。可以通过ifconfig为我们要启用的网卡设置好IP和mask等等，通过ifconfig ethX up，就不会影响另外一块网卡。

  2、[策略路由说明文档](http://lartc.org/howto/index.html)
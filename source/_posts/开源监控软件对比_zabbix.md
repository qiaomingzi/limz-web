---
title: 开源监控软件对比_zabbix
date: 2017-09-18 15:42:33
tags: 软文收集
thumbnail: /css/images/favicon.gif
banner: /css/images/favicon.gif

---
## 开源监控软件对比

- **Cacti****（英文含义仙人掌）**

是一套基于PHP、MySQL、SNMP和RRDtool开发的网络流量监测图形分析工具，它通过snmpget来获取数据使用RRDtool绘图，简化RRDtool使用。提供了非常强大的数据和用户管理功能，可以指定每一个用户能查看树状结构、主机设备以及任何一张图，还可以与LDAP结合进行用户认证，同时也能自定义模板。在历史数据展示监控方面，其功能相当不错。

Cacti通过添加模板，使不同设备的监控添加具有可复用性，并且具备可自定义绘图的功能，具有强大的运算能力（数据的叠加功能）

-  **nagios**

Nagios是一款开源的免费网络监视工具，能有效监控Windows、Linux和Unix的主机状态，交换机路由器等网络设置，打印机等。

Nagios 可以监控的功能有： 
1、监控网络服务（SMTP、POP3、HTTP、NNTP、PING等）； 
2、监控主机资源（处理器负荷、磁盘利用率等）； 
3、简单地**插件设计**使得用户可以方便地扩展自己服务的检测方法； 
4、并行服务检查机制； 
5、具备定义**网络分层结构**的能力，用"parent"主机定义来表达网络主机间的关系，这种关系可被用来发现和明晰主机宕机或不可达状态； 
6、当服务或主机问题产生与解决时将告警发送给联系人（通过EMail、短信、用户定义方式）； 
7、具备定义事件句柄功能，它可以在主机或服务的事件发生时获取更多问题定位； 
8、自动的日志回滚； 
9、可以支持并实现对主机的冗余监控； 
10、可选的WEB界面用于查看当前的网络状态、通知和故障历史、日志文件等；

- Ganglia

是一个跨平台的、可扩展的、**高性能**的分布式监控系统，如集群和网格。它基于**分层设计**，使用广泛的技术，用RRDtool存储数据。具有可视化界面，适合对集群系统的自动化监控。其精心设计的数据结构和算法使得监控端到被监控端的连接开销非常低。目前已经有成千上万的集群正在使用这个监控系统，可以轻松的处理2000个节点的集群环境。

Ganglia的强大在于：ganglia服务端能够通过一台客户端收集到同一个网段的所有客户端的数据，ganglia集群服务端能够通过一台服务端收集到它下属的所有客户端数据。

- **Zabbix**

是一个基于web界面的分布式监控系统，支持多种采集方式采集客户端，有专用的Agent代理，也支持SNMP、IPMI、JMX、Telnet、SSH等多种协议，它将采集到的数据存放到数据库，然后对其进行分析整理，达到条件触发告警。其灵活的扩展性和丰富的功能是其他监控系统所不能比的。

[![clip_image002](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231229283-64771159.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231227923-1370234025.jpg)

结论：

从以上各种监控系统的对比来看，Zabbix都是具有优势的，其丰富的功能、可扩展的能力、二次开发的能力和简单易用的特点，读者只要稍加学习，即可构建自己的监控系统。

## Zabbix

### 选择zabbix的理由：

\- 安装与配置简单，学习成本低 
\- 支持多语言（包括中文） 
\- 免费开源

\- 数据采集到数据库，可二次分析监控数据的。 
\- 自动发现服务器与网络设备 
\- 分布式监视以及WEB集中管理功能 
\- 可以无agent监视 
\- 用户安全认证和柔软的授权方式 
\- 通过WEB界面设置或查看监视结果 
\- email等通知功能 
等等

### 组成：

zabbix server：可以通过SNMP、zabbixagent、ping、端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，它可以运行在Linux, Solaris, HP-UX, AIX, Free BSD, Open BSD, OS X等平台之上。

zabbix agent（可选组件）：安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU等信息的收集。

### 生命周期

[![clip_image004](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231230501-657878972.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231229751-1485347326.jpg)

目前推荐使用**Zabbix2.2**

### 安装

#### 准备：

epel源、mysql已安装(yum install mysql-server)、zabbix官方仓库（<http://repo.zabbix.com/>）

1. **安装epel源**

\# wget –O  /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo

2. **安装Zabbix官方源码：**

\#rpm -ivh <http://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/zabbix-release-2.2-1.el6.noarch.rpm>

#### 安装zabbix server：

\3. 安装依赖包yum install -y OpenIPMI

```

```

\4. yum install -y zabbix-server zabbix-server-mysql

[![clip_image008](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231233033-482209523.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231232455-351075957.jpg)

5. **创建数据库**

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
mysql> create database zabbix;

mysql> grant all on zabbix.* to zabbix@localhost identified by "zabbix_pass";

# mysql -uzabbix -pzabbix_pass -hlocalhost zabbix # 测试

# rpm -ql zabbix-server-mysql

导入server端数据表（注意顺序）：

# mysql -uzabbix -pzabbix_pass -hlocalhost zabbix </usr/share/doc/zabbix-server-mysql-2.2.11/create/schema.sql  <--导入数据结构

#mysql -uzabbix -pzabbix_pass -hlocalhost zabbix </usr/share/doc/zabbix-server-mysql-2.2.11/create/images.sql  <--导入图片

# mysql -uzabbix -pzabbix_pass -hlocalhost zabbix </usr/share/doc/zabbix-server-mysql-2.2.11/create/data.sql   <--导入数据
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

6. **修改配置文件** # vim /etc/zabbix/zabbix_server.conf   <--增加zabbix使用数据库的密码

[root@zabbix-server ~]# egrep -v "^#|^$" /etc/zabbix/zabbix_server.conf

LogFile=/var/log/zabbix/zabbix_server.log

LogFileSize=0

PidFile=/var/run/zabbix/zabbix_server.pid

**DBHost=localhost**

**DBName=zabbix**

**DBUser=zabbix**

**DBPassword=zabbix_pass **  <--需修改同数据库设置****

DBSocket=/tmp/mysql.sock 

SNMPTrapperFile=/var/log/snmptt/snmptt.log

AlertScriptsPath=/usr/lib/zabbix/alertscripts

ExternalScripts=/usr/lib/zabbix/externalscripts

7. **启动zabbix server**：

\#service zabbix-server start

\# netstat -ntlpu|grep 10051 ß检查启动状态

\# chkconfig zabbix-server on   ß添加开机启动

#### server端安装zabbix web：

\1. # yum install zabbix-web zabbix-web-mysql

[![clip_image012](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231234408-600666281.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231233658-602110986.jpg)

默认安装了httpd服务，启动httpd

\# chkconfig httpd on    ß添加开机启动

\2. 访问<http://ip/zabbix>进入安装

[![clip_image014](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231235564-1308699009.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231234876-404563067.jpg)

==>**解决**：修改时区 # vim /etc/httpd/conf.d/zabbix.conf

[![clip_image016](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231236705-822655594.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231236158-1130877742.jpg)

3. **登录**默认用户admin 密码：zabbix

4. **修改字体**

vim /usr/share/zabbix/include/locales.inc.php

yum -y install wqy-microhei-fonts ß安装中文字体集

rm -f /etc/alternatives/zabbix-web-font #删除原有字体连接文件

ln -s /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /etc/alternatives/zabbix-web-font

#### 安装agent

客户端和服务端都安装

\1. # yum install -y zabbix-agent

[![clip_image018](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231237642-429558313.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231237220-1600455322.jpg)

\# vim /etc/zabbix/zabbix_agentd.conf     //修改被动模式IP为zabbix server ip

[![clip_image020](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231238455-857696247.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231238033-1708506885.jpg)

**注**：（一个Agent是可以同时向多个服务器端发送数据，多个Ip用逗号隔开）

Server：被动模式，允许哪台服务器连接Agent。

ServerActive：主动模式，向哪台服务器传送数据。

\# /etc/init.d/zabbix-agent start //启动agent

\# netstat -lntp |grep 10050 //验证端口

 

如果提示不能正常运行，分别检查zabbix_server.conf中的各项配置文件、Selinux、Iptables等。配置文件请检查以下文件的正确配置参数。

1./etc/zabbix/zabbix_server.conf中的参数。

\# 
DBHost=localhost         ß数据库的IP(域名)地址

DBName=zabbix              ß数据库的名称

DBUser=zabbix                ß数据库的用户

DBPassword=zabbix        ß数据库的密码

2./etc/zabbix/web/zabbix.conf.php中的配置。

[root@linux-node1 ~]# cat 
/etc/zabbix/web/zabbix.conf.php

<?php

// Zabbix GUI configuration file.

global $DB;

$DB[‘TYPE’]    = ‘MYSQL’;          //数据库类型

$DB[‘SERVER’]  = ‘localhost’;              //数据库的IP（域名）地址

$DB[‘PORT’]    = ‘0’;           //数据库的端口

$DB[‘DATABASE’] = ‘zabbix’;           //数据库的名称

$DB[‘USER’]    = ‘zabbix’;            //数据库的用户

$DB[‘PASSWORD’] = ‘zabbix’;          //数据库的密码

// Schema name. Used for IBM DB2 and PostgreSQL.

$DB[‘SCHEMA’] = ”;

$ZBX_SERVER     = ‘localhost’;          ßZabbix-Server的IP（域名）地址

$ZBX_SERVER_PORT = ‘10051’;              ßZabbix-Server的端口

$ZBX_SERVER_NAME = ‘Zabbix-Xuliangwei’;          ßZabbix-Server web界面的标识

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;

?>

### 监控流程

[![image](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231239658-629855012.png)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231238970-1704404905.png)

### Zabbix-Get使用

Zabbix-Get是Zabbix中的一个程序，用于Zabbix-Server到Zabbix-Agent的数据获取，通常可以用来检测验证Agent的配置是否正确。

用法：zabbix_get [-hV] -s <host name or IP> [-p <port>] [-I <IP address>] -k <key>

-s：远程Zabbix-Agent的IP地址或者是主机名。

-p：远程Zabbix-Agent的端口。

-l：本机出去的IP地址，用于一台机器中又多个网卡的情况。

-k：获取远程Zabbix-Agent数据所使用的Key。

#### 实现监控cpu（zabbix server主机上操作）

\# yum install -y zabbix-get //安装zabbix-get工具

\# zabbix_get -s 192.168.3.113 -k system.cpu.util[,user]

[![clip_image024](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231241064-270046254.jpg)](http://images2015.cnblogs.com/blog/856286/201603/856286-20160302231240095-1864992364.jpg)

Agent数据采集方式：passive、active

Other Agent：SNMP、IPMI、Java Gateway
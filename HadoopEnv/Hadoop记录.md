
# --Hadoop--

## Hadoop介绍

### Hadoop历史

创始人：Doug Cutting

解决问题：存储数据困难，检索速度慢

思想起源：Google的3篇论文

GFS--HDFS，分布式文件系统

Map-Reduce--MR，映射和分析计算

BigTable--HBase，大表



### Hadoop的3大发行版

Apache版本，原始基础

Cloudera，CDH和Hortonworks合并，

CDP新版本收费，服务器1W美金/台

CDH的升级版，是CDP

但是现在由于CDP收费，发展前景可能受阻，后面课程会逐渐弱化这一块。

Apache版本，需要更多配置，相当于手动挡

CDH、CDP安装部署很方便，相当于自动档



### Hadoop版本问题

1.X

MapReduce，计算+资源调度

2.X

MapReduce，计算

Yarn，资源调度

区别

MapReduce的任务进行了拆分，解



### Hadoop架构

#### HDFS

NN，namenode元数据

DN，datanode数据

ZNN，NN的备份



#### YARN架构

AM，applicationManager应用管理，是任务的管理者

Container，容器，相当于计算机（虚拟机、云计算为其中一种形式）为AM提供资源

RM，resourceManager资源管理，是资源的总管理

NM，nodeManager节点管理，受到RM的管理



YARN很重要，可与Hadoop、spark、flink使用

Zookeeper，要和所有的软件配合使用，特别是Kafka。



#### MapReduce

Map分，Reduce合



## Hadoop文件结构

### 文件目录介绍

.so文件，动态链接文件，在压缩时会用到

hadoop文件，sbin文件夹，都是启动和停止集群的脚本。

bin、etc、sbin用得多。



### linux中Hadoop的配置

site的优先级比default高，比如core-site和core-default。

#### 1.配置文件

**/etc/profile**系统环境配置

作用：添加环境变量

注意：修改完毕后，需要用`source`命令刷新，才会生效。

```bash
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin


##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

```



**$hadoop/etc/hadoop/，下的配置文件**

**hadoop-env.sh**

**作用：**

- hadoop环境
- 修改JAVA_HOME路径

```bash
export JAVA_HOME=/opt/module/jdk1.8.0_144
```



**yarn-env.sh**

**作用：**

- yarn环境
- 修改JAVA_HOME路径

```bash
export JAVA_HOME=/opt/module/jdk1.8.0_144
```



**mapred-env.sh**

**作用：**

- MapReduce环境
- 修改JAVA_HOME路径

```bash
export JAVA_HOME=/opt/module/jdk1.8.0_144
```





**core-site.xml**

**作用：**

- 指定NameNode地址；
- 指定Hadoop运行时产生文件的存储目录

修改原因：Hadoop运行时产生文件的存储目录，默认是放在/tmp/下，但是tmp是临时文件夹，linux会定期清除。

```bash
<!-- 指定HDFS中NameNode的地址 -->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop101:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/module/hadoop-2.7.2/data/tmp</value>
</property>

```



**hdfs-site.xml**

**作用：**

- 指定HDFS副本数量；
- 指定Hadoop辅助名称节点

```xml
<!-- 指定HDFS副本的数量 -->
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>

<!-- 指定Hadoop辅助名称节点主机配置 -->
<property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop104:50090</value>
</property>

```



**yarn-site.xml**

**作用：**

- Reducer获取数据的方式；
- 指定RM的地址；
- 日志聚集功能、保留时间；

```xml
<!-- Reducer获取数据的方式 -->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop103</value>
</property>

<!-- 日志聚集功能使能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>


```



**mapred-site.xml**

**作用：**

- 指定MR运行方式；
- 历史服务器地址；
- 历史服务器web地址

```xml
<!-- 指定MR运行在YARN上 -->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>

<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop102:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>


```



#### **2.slave**

**作用：**

- 配置slave主机名，不能有空行和空格，比如`hadoop104`，前后都不能有空格。



#### 3.配置优先级

优先级从高到低依次为

- Java代码中，`Confuguration.set`方法。
- resource，Java项目中的`java/resources/`下的配置文件
- 集群，hadoop中的site等配置文件（在hadoop的etc/hadoop/中）
- default，hadoop中的默认配置文件。（存放在hadoop的jar包中）




## 集群

### 集群的使用操作

#### 1.集群部署规划

原则：NamoeNode和SecondaryNameNode会很耗内存资源，不要安装在同一台服务器上。

ResourceManager也很耗内存，不要和NameNode、SecondaryNameNode配置在同一台机器上。



#### 2.群起操作步骤

- 配置slaves
- 集群的第一次群起，需要NN格式化，bin/hdfs namenode -format
- 用NN服务器，启动HDFS，`sbin/start-dfs.sh`
- 用RM服务器，启动YARN，`sbin/start-yarn.sh`
- 用history服务器，启动历史守护程序，`sbin/mr-jobhistory-daemon.sh start historyserver`
- MR任务，都会有input输入路径，和output输出路径。其中output必须是空的，没有这个文件夹，需要自动创建这个文件。

其中，启动命令，可以查看hadoop中的`sbin/`下的文件。

如`start-all.sh`和`stop-all.sh`，会同时启动或关闭HDFS和YARN。但是现在官方不推荐这种方式，所以笔记上被删除了。



#### 3.SSH无密登陆注释

- NN服务器，配置root账户的SSH公钥，给其他服务器
- RM服务器，配置atguigu账户的SSH公钥，给其他服务器







### 使用问题

**1.namenode无法启动**

格式化namenode，如果出现y/N选项，那么肯定格式化失败了。

比如一个错误，用jps查看java进程，发现NameNode和DataNode会挂掉，那么可能是格式化问题。因为集群的clusterID，nn和dn中的不一致了。

解决办法：正确的格式化nn步骤，kill所有进程，rm /data和/logs，再次格式化

hdfs和yarn的env配置，都要修改，之前错过了。凡是碰到env文件，都要配置一下。











集群群起，配置slave文件，注意，不用有空格、空行。比如`hadoop104`，前后都不能有空格。

NameNode服务器，在这个服务器上进行hdfs的群起。2NN服务器会跟着启动。

Yarn服务，在resourceManager服务器上进行yarn的群起。



### 常用端口(待补充)

8088:YARN的浏览器页面查看，/cluster

50070:查看HDFS文件系统，/dfshealth.html#tab-overview

9000：NameNode端口，内部端口。老版本是8020

50090：2NN端口。后期会换成HA高可用，生成2个2NN

10020：历史服务器

19888：历史服务器web端，/jobhistory

8485：journalnode的RPC服务

2181：HQuorumPeer，zookeeper端口





### 真实开发坏境

虚拟机用桥接模式。

关闭防火墙。公司内用内网，服务器是内部自己连的，所以为了效率就关了防火墙。公司内网和外网间会有一个大的防火墙。但是一些不太正规的企业，会开防火墙。



shuffle，混洗、洗牌，大数据中最重要的单词。



快速克隆，克隆后安装JDK和HADOOP，都是用拷贝的方式安装。风险低的方法，在其中一个克隆上装JDK和HADOOP，然后拷贝到其他克隆机上。

跨服务器的拷贝，scp，很常用





<Hadoop入门笔记>4.3.10，步骤中，有必须使用root的部分，用sudo也不行。root和sudo是有些区别的。

目前hadoop用得越来越少了，所以第五章hadoop源码了解即可。现在hadoop只作为存储，HDFS。其他计算，使用Hive，Spark、Flink等。



用户网页点击购买、Tomcat收集访问日志信息，Sqoop、Flume、Kafka、Spark、Flink、返回结果到数据库或形成文件、Tomcat推荐业务、网页信息展示

## 虚拟机配置

### 基本配置

- 服务器名映射，`/etc/hosts`


- 环境路径


java安装好后，要source一下，才能看到version

准备1台最小的虚拟机，改好IP等配置

IP

192.168.1.100

/etc/sysconfig/network-scripts/ifcng-eth0

主机名hadoop100

/etc/hosts

防火墙关闭

chckd iptables off

用户名atguigu，配置成sudo权限，后期不要使用root账户

/etc/sudoers



### SSH无密登陆

#### **常用方法**

- 生成公钥私钥，`ssh-keygen -t rsa`
- 公钥拷贝，`ssh-copy-id 目标服务器地址`
- .ssh文件夹在家目录下



#### SSH原理（待补充）

（积累中，不全面）

**公钥的记录**

- ssh会把你每个你访问过计算机的公钥(public key)都记录在~/.ssh/known_hosts。当下次访问相同计算机时，OpenSSH会核对公钥。如果公钥不同，OpenSSH会发出警告，避免你受到DNS Hijack之类的攻击。


- SSH对主机的public_key的检查等级是根据StrictHostKeyChecking变量来配置的。默认情况下，StrictHostKeyChecking=ask。简单所下它的三种配置值：

  - 1.StrictHostKeyChecking=no，最不安全的级别
    - 没有那么多烦人的提示了，相对安全的内网测试时建议使用。如果连接server的key在本地不存在，那么就自动添加到文件中（默认是known_hosts），并且给出一个警告。
  - 2.StrictHostKeyChecking=ask  #默认的级别
    - 出现提示。如果连接和key不匹配，给出提示，并拒绝登录。
  - 3.StrictHostKeyChecking=yes  #最安全的级别
    - 如果连接与key不匹配，就拒绝连接，不会提示详细信息。

  ​



#### 使用问题（待完善）

- SSH连接问题，`The authenticity of host XX can't be established.`

**问题：**用SSH无密启动hadoop时，出现了`The authenticity of host 'localhost (::1)' can't be established.`提示。但是输入密码或者回车后也可以完成启动。为什么会出现这种情况呢？

**原因：**SSH对主机的public_key的检查等级过高。

**解决方法：**配置public_key检查等级为`StrictHostKeyChecking=no  `



- Warning: Permanently added 'localhost' (RSA) to the list of known hosts.（待完善）





### 集群分发脚本

目标：集群同步脚本

`#3`代码解释：cd -P，是为了防止进入软连接

脚本存放位置：一般放在`家目录/bin/`

```bash
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if ((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for host in hadoop102 hadoop103 hadoop104
do
    echo ------------------- $host --------------
    rsync -av $pdir/$fname $user@$host:$pdir
done

```



### 使用问题

linux问题，reboot一直卡住，就直接强制关闭了虚拟机。再打开后，发现有gnu grub version 0.97页面，无法正常启动。

在管理服务器的过程中因为异常断电造成系统无法启动



克隆服务器后，发现无法连接网络，service network start报错，Device not managed by NetworkManager or unavailable。

原因：Window中的服务，VMnetDHCP，VMnetNAT服务停止了。



> 目前的课程还是用的hadoop2.7X，受疫情影响，升级的hadoop3X课程会在1-2个月后出来，到时候我们再升级。（课程相关）
>
> 案例，wordCount，会伴随全程，包括面试手写代码
>
> 各种端口号，是入门面试的必问项。后面会有总结。
>
> 总结修改过的配置文件，说明修改的作用是什么？
>
> 拿白纸，写这些配置文件，复习的方法。
>
> 面试，会问你写过哪些脚本（面试经验）
>
> 开发过程中，完成4.3.1-4.3.7，就完成了开发的90%。











本次搭建3个集群的失误点

/etc/profile文件没有分发下去，导致hadoop启动失败。

ssh公私钥，自己也要配置的。

对任务的了解不彻底，比如做wordCount案例需要上传文件。用hdfs  dfs

关闭hadoop时，dfs关闭，无法关闭dataNode进程。





java.net.BindException: 无法指定被请求的地址

解决问题：查端口，发现并没有被占用；

IP绑定问题，在/etc/hosts文件中记录了IP地址及其对应的主机名，需要增加`本机IP localhost`这一行字段。







Hadoop入门部分重点

技术

3

4.3

面试

1.6

2.5







# --HDFS--

Hadoop现在还能生存的原因，就是因为HDFS。MR现在被其他的取代了。

笔记重点

4章，HDFS中唯一的面试题

5章，原本是重点，但现在1年内没有面试问过了，所以列为了解。

5.4重点

6章，开发重点，不是面试重点了。

7章，开发中也常用，小文件存储是HDFS的痛点



5.1中，2NN在工作环境下，被HA替代了。变成2个NN，并用第三方工具keep来管理。

大数据场景下，最不值钱的是磁盘，最值钱的是数据

数据可以冗余，但不能少。

笔记5.4是开发重点。笔记5.1-5.3，由于2NN被HA替代，所以降级为了解。

### 缺点

- 小文件存储是最大的问题

- 不支持文件随机修改，仅支持数据append追加

- Fsimage，镜像文件

- Edits，编辑日志

- 热备，服务器高可用应用的另一种说法




## 概念与介绍

### Block Size文件块

文件的实际大小Size，块大小Block Size可能不一致，并且实际占用磁盘的空间是实际大小Size，占用上限是Block Size。

为什么说块设置太大，会影响性能？（疑问）

设置Block Size的依据，寻址时间为传输时间的1%时，最佳，经验值。然后取2的指数。主要取决于磁盘传输速度。

现在企业中，Block Size定义为256M。



## 客户端操作

### HDFS-shell命令

#### `hadoop fs`与`hdfs fs`的区别

相同：这两个命令，可以画等号，操作方式基本相同，用起来一样。

区别：一个是hadoop框架的命令，一个是hdfs的命令。



mkdir，创建文件夹时，如果没有以`/`开头，那么HDFS会自动将路径改为`/user/操作的用户名/...`



存储副本，和集群数量有关。副本只能按着服务器来存，不能1个服务器存同一个文件的多个副本。

设置replication数量，不一定代表有这么多副本，要看有多少台服务器。服务器不够，则不会创建replicatioin数量的副本，会等待新加入服务器后再把副本创建到replication数量。



笔记3.1

hadoop-client，是需要配置的。jdk-tools，是提示错误时再添加的，不提示错误不添加。

maven没有自动import，可以右键project，Maven-Reimport





crc文件，循环校验码。

在传输过程中，从hdfs传到本地磁盘上，保证传输的文件完整无误。比如，传输过程中，数据丢失了一部分，那么crc文件校验会失败。





再搭一遍集群，102-2nn，103-nn，104-rm

HDFS的shell操作，2-3遍

HDFS客户端操作集群，至少2遍，上传、下载、查看、移动、剪切等，大公司要求高的需要



DAY4自习日

xsync脚本，shell编写时的错误<shell>（易错）

注意“ ` ”，这个符号，称为后引号（backquotes）或是斜引号（backticks），执行结果能赋值给一个变量。不要跟单引号混淆！



配置3台集群群起

RM服务器中的RM进程未能正常启动，查看log，发现错误，bindException。但是单独再RM服务器上，用`sbin/start-yarn.sh`启动，则可以开启RM进程。（疑问）（解答）RM进程要在RM服务器上启动，不是在NN服务器上启动。



### Hadoop-Client

#### 常用方法

**org.apache.hadoop.fs.FileSystem;**

**copyToLocalFile**

copyToLocalFile(Boolean delSrc, Path src, Path dec, Boolean useRawLocalFileSystem)推荐使用，避免出现问题。



**delete**

delete(Path des, Boolean recursive)，des是要删除的文件或文件夹。recursive是指示是否递归删除。



#### 使用问题

##### copyToLocalFile的空指针异常

**问题：**使用copyToLocalFile(Path src, Path dec)参数，会出现NullPointerException空指针异常。其中，默认delSrc=false, useRawLocalFileSystem=false。`useRawLocalFileSystem`这是本问题的关键。可能导致 setPermission文件本地文件效验失败。

**原因：**Hadoop LocalFileSystem是客户端校验的类。在使用LocalFileSystem写文件时，会透明的创建一个.filename.crc的文件。 校验文件大小的字节数由io.bytes.per.checksum属性设置，默认是512bytes,即每512字节就生成一个CRC-32校验 和。.filename.crc文件会存 io.bytes.per.checksum的信息。在读取的时候，会根据此文件进行校验。事实上LocalFileSystem是通过继承 ChecksumFileSystem实现校验的工作。

**解决方法：**使用(Boolean delSrc, Path src, Path dec, Boolean useRawLocalFileSystem)的参数，配置(false, path, path, true)则可以正常执行。





```html
<tbody><tr><td>组件</td>
			<td>节点</td>
			<td>默认端口</td>
			<td>配置</td>
			<td>用途说明</td>
		</tr><tr><td>HDFS</td>
			<td>DateNode</td>
			<td>50010</td>
			<td>dfs.datanode.address</td>
			<td>datanode服务端口，用于数据传输</td>
		</tr><tr><td>HDFS</td>
			<td>DateNode</td>
			<td>50075</td>
			<td>dfs.datanode.http.address</td>
			<td>http服务的端口</td>
		</tr><tr><td>HDFS</td>
			<td>DateNode</td>
			<td>50475</td>
			<td>dfs.datanode.https.address</td>
			<td>http服务的端口</td>
		</tr><tr><td>HDFS</td>
			<td>DateNode</td>
			<td>50020</td>
			<td>dfs.datanode.ipc.address</td>
			<td>ipc服务的端口</td>
		</tr><tr><td><span style="color:#f33b45;">HDFS</span></td>
			<td><span style="color:#f33b45;">NameNode</span></td>
			<td><span style="color:#f33b45;">50070</span></td>
			<td><span style="color:#f33b45;">dfs.namenode.http-address</span></td>
			<td><span style="color:#f33b45;">http服务的端口</span></td>
		</tr><tr><td>HDFS</td>
			<td>NameNode</td>
			<td>50470</td>
			<td>dfs.namenode.https-address</td>
			<td>https服务的端口</td>
		</tr><tr><td><span style="color:#f33b45;">HDFS</span></td>
			<td><span style="color:#f33b45;">NameNode</span></td>
			<td><span style="color:#f33b45;">8020</span></td>
			<td><span style="color:#f33b45;">fs.defaultFS</span></td>
			<td><span style="color:#f33b45;">接收Client连接的RPC端口，用于获取文件系统metadata信息。</span></td>
		</tr><tr><td>HDFS</td>
			<td>journalnode</td>
			<td>8485</td>
			<td>dfs.journalnode.rpc-address</td>
			<td>RPC服务</td>
		</tr><tr><td>HDFS</td>
			<td>journalnode</td>
			<td>8480</td>
			<td>dfs.journalnode.http-address</td>
			<td>HTTP服务</td>
		</tr><tr><td>HDFS</td>
			<td>ZKFC</td>
			<td>8019</td>
			<td>dfs.ha.zkfc.port</td>
			<td>ZooKeeper FailoverController，用于NN HA</td>
		</tr><tr><td><span style="color:#f33b45;">YARN</span></td>
			<td><span style="color:#f33b45;">ResourceManage</span></td>
			<td><span style="color:#f33b45;">8032</span></td>
			<td><span style="color:#f33b45;">yarn.resourcemanager.address</span></td>
			<td><span style="color:#f33b45;">RM的applications manager(ASM)端口</span></td>
		</tr><tr><td>YARN</td>
			<td>ResourceManage</td>
			<td>8030</td>
			<td>yarn.resourcemanager.scheduler.address</td>
			<td>scheduler组件的IPC端口</td>
		</tr><tr><td>YARN</td>
			<td>ResourceManage</td>
			<td>8031</td>
			<td>yarn.resourcemanager.resource-tracker.address</td>
			<td>IPC</td>
		</tr><tr><td>YARN</td>
			<td>ResourceManage</td>
			<td>8033</td>
			<td>yarn.resourcemanager.admin.address</td>
			<td>IPC</td>
		</tr><tr><td><span style="color:#f33b45;">YARN</span></td>
			<td><span style="color:#f33b45;">ResourceManage</span></td>
			<td><span style="color:#f33b45;">8088</span></td>
			<td><span style="color:#f33b45;">yarn.resourcemanager.webapp.address</span></td>
			<td><span style="color:#f33b45;">http服务端口</span></td>
		</tr><tr><td>YARN</td>
			<td>NodeManager</td>
			<td>8040</td>
			<td>yarn.nodemanager.localizer.address</td>
			<td>localizer IPC</td>
		</tr><tr><td>YARN</td>
			<td>NodeManager</td>
			<td>8042</td>
			<td>yarn.nodemanager.webapp.address</td>
			<td>http服务端口</td>
		</tr><tr><td>YARN</td>
			<td>NodeManager</td>
			<td>8041</td>
			<td>yarn.nodemanager.address</td>
			<td>NM中container manager的端口</td>
		</tr><tr><td>YARN</td>
			<td>JobHistory Server</td>
			<td>10020</td>
			<td>mapreduce.jobhistory.address</td>
			<td>IPC</td>
		</tr><tr><td><span style="color:#f33b45;">YARN</span></td>
			<td><span style="color:#f33b45;">JobHistory Server</span></td>
			<td><span style="color:#f33b45;">19888</span></td>
			<td><span style="color:#f33b45;">mapreduce.jobhistory.webapp.address</span></td>
			<td><span style="color:#f33b45;">http服务端口</span></td>
		</tr><tr><td>HBase</td>
			<td>Master</td>
			<td>60000</td>
			<td>hbase.master.port</td>
			<td>IPC</td>
		</tr><tr><td>HBase</td>
			<td>Master</td>
			<td>60010</td>
			<td>hbase.master.info.port</td>
			<td>http服务端口</td>
		</tr><tr><td>HBase</td>
			<td>RegionServer</td>
			<td>60020</td>
			<td>hbase.regionserver.port</td>
			<td>IPC</td>
		</tr><tr><td>HBase</td>
			<td>RegionServer</td>
			<td>60030</td>
			<td>hbase.regionserver.info.port</td>
			<td>http服务端口</td>
		</tr><tr><td>HBase</td>
			<td>HQuorumPeer</td>
			<td>2181</td>
			<td>hbase.zookeeper.property.clientPort</td>
			<td>HBase-managed ZK mode，使用独立的ZooKeeper集群则不会启用该端口。</td>
		</tr><tr><td>HBase</td>
			<td>HQuorumPeer</td>
			<td>2888</td>
			<td>hbase.zookeeper.peerport</td>
			<td>HBase-managed ZK mode，使用独立的ZooKeeper集群则不会启用该端口。</td>
		</tr><tr><td>HBase</td>
			<td>HQuorumPeer</td>
			<td>3888</td>
			<td>hbase.zookeeper.leaderport</td>
			<td>HBase-managed ZK mode，使用独立的ZooKeeper集群则不会启用该端口。</td>
		</tr><tr><td>Hive</td>
			<td>Metastore</td>
			<td>9085</td>
			<td>/etc/default/hive-metastore中export PORT=&lt;port&gt;来更新默认端口</td>
			<td>&nbsp;</td>
		</tr><tr><td>Hive</td>
			<td>HiveServer</td>
			<td>10000</td>
			<td>/etc/hive/conf/hive-env.sh中export HIVE_SERVER2_THRIFT_PORT=&lt;port&gt;来更新默认端口</td>
			<td>&nbsp;</td>
		</tr><tr><td>ZooKeeper</td>
			<td>Server</td>
			<td>2181</td>
			<td>/etc/zookeeper/conf/zoo.cfg中clientPort=&lt;port&gt;</td>
			<td>对客户端提供服务的端口</td>
		</tr><tr><td>ZooKeeper</td>
			<td>Server</td>
			<td>2888</td>
			<td>/etc/zookeeper/conf/zoo.cfg中server.x=[hostname]:nnnnn[:nnnnn]，标蓝部分</td>
			<td>follower用来连接到leader，只在leader上监听该端口</td>
		</tr><tr><td>ZooKeeper</td>
			<td>Server</td>
			<td>3888</td>
			<td>/etc/zookeeper/conf/zoo.cfg中server.x=[hostname]:nnnnn[:nnnnn]，标蓝部分</td>
			<td>用于leader选举的。只在electionAlg是1,2或3(默认)时需要</td>
		</tr></tbody>
```



#### 工作实际应用

和kafka的配合。从kafka读数据，写入hdfs。

或者从hdfs读数据，写入到mysql。

这些需要用



## 内部原理

### HDFS写数据流程

![Snipaste_2020-02-23_23-54-01](E:\itHeiMa\资料\课程截图类\Snipaste_2020-02-23_23-54-01.png)

#### **写数据流程解析**

- client传输数据，通过packet，1个packet有64byte
- DN存储数据，先存入内存，经过序列化再存入磁盘。DN中存入内存和存入磁盘是并行的。

- nn阶段返回dn节点给client，选取dn节点的原则是选取dn节点距离最近原则

- 如果传输中某个DN挂了，或者某个DN满了，那么上传的Block会自动备份到其他DN。




#### 节点距离计算

**client和哪个DN节点最近？**

- 以进行上传操作的那个DN节点为起点，计算节点距离。
- 比如从hadoop105这个DN服务器进行上传文件操作，那么hadoop105就是起点，hadoop105与自己的距离就是0，所以Block上传的第一个位置就是hadoop105.

**节点的共同祖先**

- 路由器，也就是网关

#### 机架感知（副本存储节点选择）

不同hadoop版本，其默认的机架感知策略也不同。

- Hadoop1.X
  - 第2个副本是在不同机架上，因为当时机器性能较弱，考虑更偏向安全。
- Hadoop2.X
  - 第2个副本在同一机架上，因为随时时代发展机器性能更好了，考虑更偏向性能。



### NN工作机制

![Snipaste_2020-02-24_00-22-55](E:\itHeiMa\资料\课程截图类\Snipaste_2020-02-24_00-22-55.png)

#### **NN节点需求**

NN的元数据，经常需要进行随机访问，相应客户请求。这些元数据存在内存，担心掉电造成数据丢失。存在磁盘上，会导致效率问题。

#### 解决方法

内存中，NN元数据，进行工作。

磁盘中，NN元数据分成2个部分，用小文件满足效率，大文件满足数据安全。

- 小文件Edits

作用：记录元数据的增删改

过程：小文件Edits滚动更新，最新的Edits为`edits_inprogress`，而`edits_inprogress`会随着时间或操作的进行，滚动更新为`edits` 。

- 大文件FsImage

作用：备份全量的元数据，定期与edits合并更新

过程：设定一定时间或者edits数据满了之后，FsImage和edits的数据合并，生成新的FsImage。

#### 2NN流程解析

- client的增删改请求，先加载到edit编辑日志中，再加载到内存


- NN和2NN的区别

  - edits中，NN有最新的一个edits_inprogress数据，而2NN没有。
  - 如果NN挂了，这个最新的edit数据不能恢复的。

  ​



上传文件到HDFS中，发现传递过程中一直都是`文件名_copying`，这是为了防止在文件未写入完整时，有其他程序读取该文件，所以文件名后加了copying



scp传输文件时，可以使用`scp -r 用户名@主机IP:/文件位置 des`，用Tab打文件位置时，注意，提示的是scp命令的这台主机的文件目录，而不是目标主机的文件目录。



## 集群使用

### NN故障处理（开发重点）

NN丢失了，但是数据data并没有丢失，还是存在磁盘里，成为了一个没有索引的文件。并且这样的文件不会被扫描删除。

很难通过DN来恢复数据，edit_inprogress中的修改丢了就丢了。在一堆数据中找到这些没有索引的文件，就像大海捞针。



笔记5.4中，恢复NN的第二个方法，需要配置hdfs-site.xml，配置出namenode的文件位置。

生产环境中，第二个方法用得多，比较正规的用法。因为直接一行命令，不用复制2NN文件就可以启动NN。但是一旦关闭了这个命令，NN也会随之关掉，所以第二个方法适合应急。



生产环境中，现在用的HA，会有2个NN，所以不会挂得这么彻底。如果都挂了，那就完了。

从大量的数据中，获取那么1、2条有用的数据，这效率是我们需要关心的。

在使用NN恢复第二个方法时，查看NN网页出现提示Upgrade in progress. Not yet finalized.

### 安全模式

安全模式退出判断，最小副本级别，指的是当所有数据块至少有n个（默认为1）副本与NN取得了联系。

进入安全模式的原因，刚启动；集群中的坏道很多



数仓项目，是为公司做决策的

大数据，是不允许错的，要保证可靠和安全



NN和DN的心跳时间interval，和recheck-interval的设置，根据电脑性能来配置。旧电脑配置时间长一些，性能好的电脑可以配置时间短一些。







### DN多目录

在一个机器上可以配置多目录，也就是一个机器上有多块硬盘。相当于扩容，而不是副本数的增加。

比如mnt挂载。阿里云可以动态分配，但是一般企业中，应该是先把集群规划配置好，所以类似这样的多目录用得很少。

NN多目录，多目录中的数据会同步，而不是扩容。



改配置文件，刷新对应的部分。





### 白名单与黑名单

#### 区别

白名单，没有退出过程

黑名单，有退役的过程，对数据的转移保存更好。

所以生产环境中，先配置黑名单，再配置白名单。

在阿里云等云服务器中，必须配置白名单，否则肯定回被当肉鸡。



#### 操作问题

- 退役了，为什么还可以上传文件？


服务器DN退役，超过了10+30s，那么NN会认为这个DN已经挂了，就不会再往这个退役的DN中存储数据了。这个服务器DN就退化成了一个Client，可以上传和下载，但是不能存储数据。

但是让退役的DN变成Client并不安全，所以后续还要增加权限管理。

退役的目的，是为了把服务器腾出来；或者服务器坏道多需要更换。让退役的服务器不存储数据就是最终目的。



总结当中，大公司用到的平台岗（运维）的概率很小，小公司或者从0到1的公司，需要用到平台岗（运维）的概率就大。侧重点。



namenode能存储多少个数据块？

1个数据块，需要150byte的namenode内存

如果namenode内存有128g，那么可以有9亿个文件块

1个文件块128m，



### 处理小文件

HDFS存档文件，har文件

对外是一个整体，对内是一个一个小文件。

解决小文件问题，生产环境中会大量使用，很重要。

har文件的查看，下载需要使用`har://`har协议。

小文件处理有3类4种方法。（待补充）



## HA高可用（待补充）

### 核心思想

#### 如何解决HA问题

**1.必须保证**

NameNode和ResoureManager不能故障，或HDFS和YARN故障后可以快速容灾恢复。



**2.避免单点故障**

为避免单点故障造成集群宕机，可以用启动多个NameNode的方式应对。



**3.多个NameNode的同步，避免单个NameNode压力过大，出现并发问题。**

多个NameNode，需要同步元数据。若只由Active向其他Standby同步数据，则会造成LEADER压力过大。

通过引入JournalNode解决多个NameNode同步的资源问题。



#### 多个NameNode同步原理

**1.保证多个Namenode进程元数据必须同步**

1.1元数据 fsimage

Namenode在格式化时，会生成空白的fsimage文件，此时让Namenode将格式化的Namenode的fsimage文件拷贝即可！

1.2edits

Namenode将edits文件发送给Journalnode，其他的Namnoede从Journalnode同步edits文件！

注意： a)journalnode进程采用paxos协议设计,适合运行在奇数台机器！

 b)如果要实现HA，至少要启动3台Journalnode!

c)如果启动了Journalnode，那么没有必要，也不能再启动 SecondaryNamenode



**2.启动了多个Namenode的状态处理**

只能选择其中的一个作为active状态的NN，其余的Namenode都只能作为standby状态。

采用状态是为了标识哪个namenode是正在工作的

可以为客户端提供服务的NN，只有active状态的NN可以接收客户端请求。



**3.状态处理原则**

不能出现脑裂现象(不能同时出现两个active状态的namenode)











### 实现HA

#### 要求

**1.HDFS进程要求**

- 必须进程
  - NameNode，2个以上
  - DataNode，N个
  - JournalNode，3个以上
- 可选进程
  - SecondaryNameNode，1个（在HA中一般不配置）



**2.YARN-HA配置**

与HDFS-HA配置，区别就是YARN不需要zkfc，用ResourceManager就可以完成zkfc的工作。



#### 操作

**1.格式化NameNode**

为什么要格式化？

格式化是为了：

- ①生成Namenode工作的目录(存放元数据)
- ②格式化是为了生成fsimage文件，每次NN启动都会先读取fsimage中的元数据
- ③为了生成Namenode的id和clusterId，所有的 Datanode在启动是都会根据clusterId向Namenode上报



**2.格式化NN后的操作**

集群格式化NN后，启动的NN，都是Standby状态。需要手动将其中一个NN提升为Active状态。







同步fsimage，edits。

引入journalnode，负责将edits文件同步给其他NameNode。

强制要求，实现HA，至少启动3台Journalnode。

启动多个Namenode，只能有其中一个作为active状态，接收client请求。





### HDFS自动故障转移

#### 要求

①借助zookeeper集群

②一般采用sshfence，需要配置两台Namenode所运行机器的ssh互相联通



#### HDFS-HA故障图解释（疑问）

**1.zkfc的抢占流程**

抢占流程：

1. zkfc负责心跳检测NN，一旦检测到NN心跳，那么zkfc就会在zk服务端的某个指定节点下创建一个临时节点。
2. 其他的zkfc也会一直关注这个临时节点。一旦发现临时节点消失，那么其他zkfc就会抢着在这个指定节点下创建临时节点。

3. 发现NN心跳丢失的zkfc，所在的这个节点就会将NN进程kill，不管这个NN是真死还是假死。

4. 下一个抢占了临时节点的zkfc所在的这个节点，先会对上一个active的NN节点通过SSH进行补刀，强制杀死NN（防止脑裂），然后再将自己服务器上的NN提升为active。


注意：zkfc之间是没有通讯的，这是图中的错误。



**2. 判断该服务器上的NN能否提升为active？**

NameNode提升为Active的条件：zkfc抢占到了Znode节点，则该zkfc服务器上的NameNode会被提升为Active。

通过zkfc是否抢占了临时节点。zkfc主动或者被动断开session，都会被其他的zkfc抢占临时节点。



**3.初始化HA在Zookeeper中状态**

先启动Zookeeper集群，再在任意一台服务器上运行初始化zkfc命令即可。

这是因为Zookeeper与zkfc命令有协作关系？或者zkfc直接自动运行了？（疑问）







# --MapReduce--

**笔记概览**

1.3还是很重要的

2章，平台岗

3章，开发岗，全是重点，3.3 shuffle是重点中的重点。虽然现在MR使用得不多了，但是MR的底层原理是后续Hive和Spark优化的基础，必须弄清楚。

4章，开发岗，必备技能，生产环境下Lzo压缩和Snappy压缩

5章，5.5 新增加的，开放岗必备技能。 5.2和5.4是面试题。5.4绝对的重点。

6章，面试必问

7章，扩展类，7.1和7.2后续可用Spark简单实现。

2年前是着重Hadoop开发，但现在是着重Hadoop理论。Hadoop理论是后续框架的基础。



## 概述

### 优点

- 扩展性好

mysql集群，最多100台。而MR可以实现上千台服务器集群并发工作。

- 适合PB级海量数据的离线计算

MR在分析大量数据时才有优势，小数据量时，比不过SQL。



### 缺点

- MR不擅长流式计算

因为有INPUT和OUTPUT，有一个落盘的操作，INPUT不是动态变化的。

- 不适合DAG计算

DAG有向图，程序之间有前后依赖关系，比如A出对应B的入。MR的OUTPUT有落盘，所以不适合这种DAG计算。



### 其他概念

**实时与准实时**

- 实时，过来1条处理1条

- 准实时，批处理


MR基于磁盘，tez基于内存。所以后续的框架会基于tez



**轻量级与重量级**

- 重量级，指功能比较强大，复杂
- 轻量级，指功能比较简单，只包括核心功能，操作简单



**空格和/t**

- 看txt文件中，数据之间是空格还是/。
  - 空格是很小的一个格子
  - /t是很大的一个格子。
- 这会影响String的切割split。



**路径中`/`和`\`的使用**

- java，路径一般用"/"
- windows，路径一般用"\"，可以识别"/"
- linux、unix，路径一般用"/"


- 建议：为了jajva的跨平台特性，路径全部用"/"



**虚拟机和未来要使用的阿里云**

没有任何区别

云服务器ECS，可以选择按量付费

通用型g6，1小时0.52元

CPU 2核或者4核

内存8G



1年前的面试是需要手写WC的Hadoop代码。现在更多的是Flink和Spark的WC手写。



## MapReduce编程规范

### Mapper阶段

- 定义一个类继承Mapper；
- 定义K-V信息的序列化类型；
- 重写map方法，写map业务逻辑；
- 每一个<K, V>，都调用1次map方法

### Reducer阶段

- 定义一个类继承Reduce；
- 定义K-V信息的序列化类型；
- 重写reduce方法，写reduce业务逻辑；
- 相同的<K, V>，调用1次reduce方法

### Driver阶段

- 定义一个Driver类，写psvm方法；
- 创建Configuration和Job类，配置参数；
- 基本模版：4个输入输出KV值类；3个阶段的类；2个输入输出路径；1个提交结果；System.exit为0；



### 使用问题

main()中的**args[]是什么**？Driver类中输入输出路径，args[0]，args[1]，分别是什么意思？



**输入文件从哪来？**

我们安装的是WIN10的Hadoop，没有做配置，应用的本地模式。

应用本地模式时，输入文件的来源是本地，而不是集群。



**Maven打jar包**

- 生成2种jar包
  - 有依赖dependency的jar包
    - 运行环境中有对应的依赖，用没有依赖的jar包就可以运行，比如我的linux虚拟机中已经装了hadoop。
  - 没有依赖的jar包。
    - 运行环境中没有对应的依赖，需要用dependency的jar包。



**linux中运行hadoop的jar包命令，需要注意输入输出路径的相对路径是`/user/atguigu/`而不是`/`**

![Snipaste_2020-02-24_09-14-26](E:\itHeiMa\资料\课程截图类\Snipaste_2020-02-24_09-14-26.png)

- jar路径需要写全类名，如com.atguigu.mapreduce.wordCountDriver



- input路径和output路径，最好写绝对路径。
  - 因为相对路径是`/user/$用户名/` ，而这个路径在hadoop文件系统中不是默认创建的，需要手动创建。
  - 若input路径因为这个相对路径问题找不到文件，则会在运行hadoop的jar命令时，抛出如`Input path does not exist: hdfs://hadoop103:9000/user/atguigu/l.txt`这样的错误提示。
  - 注意`/user/atguigu/` ，这就是前面所提到的相对路径位置。




**Win10环境的Hadoop启动，出现UnsatisfiedLinkError警告。**

解决方法：环境变量Path中，路径的先后顺序，可能影响了Hadoop的运行。比如其他Path中也有bin文件夹。



**Maven提示错误**

无法找到`apache-maven-3.5.3-bin\repo\org\apache\hadoop\hadoop-yarn\2.7.2`，也就是yarn的jar包。

解决办法：直接在pom中删除hadoop-yarn。



**Driver是易错点**

如在写Mapper和Reducer继承类时，中途修改了K-V的类型，容易忘记对Driver中的输入输出类型做修改。









## Hadoop序列化

### 概述



**序列化与反序列化**

序列化，从内存到磁盘

反序列化，从磁盘到内存

为了在2种进程中高效地传递对象。



**不使用Java序列化器的原因**

Java序列化会包含对象结构，继承属性。内容太多了，不适合大数据处理。



#### 优点

- 紧凑，高效实用存储空间
- 快速，读写数据的额外开销小


- 可扩展，也就是可自定义
- 互操作，是每个序列化的共同特点，多语言之间传输，并不是hadoop独有。


### 作用

- map和reduce之间的通讯，需要hadoop序列化。
- Bean对象需要自定义序列化，因为Hadoop没有给Bean对象定义序列化类。



#### 自定义bean对象实现序列化接口的解析（存疑）

**为什么Hadoop的序列化顺序和反序列化的顺序需要完全一致。**

- hadoop的传输过程，*默认是用`队列`传输的*（此处存疑）。
- 队列的数据结构特点，就是先进先出。



**mapper的输出key需要实现排序**

- 因为MapTask和ReduceTask均会对数据按照Keyj进行排序。
- Mapper`<K, V, K, V>中的第3个类K，必须要能排序，实现Comparable接口



**Bean对象需实现toString方法**

- 因为最后结果的展示格式来源于Reduce<K, V, K, V>的第4个类V，其中的toString()。
- 打`/t`是为了便于阅读。



**查看运行情况的方法**

- 单线程，可以打断点
- 多线程，打日志




## MapReduce数据流

inputformat，可以定义输入文件的读取方式，比如K-V之前K是行距离，V是一行的值。

shuffle，最重要的

outputformat，可以定义输出文件的输出方式，比如输出了part-r-0000文件。



### MapTask并行度

**决定因素：**切片个数决定MapTask的个数。

**控制参数：**FileInputFormat类，计算切片大小，由min,max,blocksize

开启1个MapTask，需要1G内存。（存疑）



### 数据切片

#### **数据切片与数据块的区别**

- 数据切片
  - 是逻辑分块，理论上的；
  - 只是一个标记，比如记录存储索引
  - 例如MapTask的数据切片。
- 数据块
  - 是物理分块，实际上的；
  - 是磁盘中实际物理分开存储的数据块；
  - 例如HDFS的Block。






#### 切片数量与什么有关？

- 切片大小
  - **必要性：**如果数据切片设计不合理，那么1个切片中的数据就可能会来自不同的DN节点，出现跨节点IO传输。
  - **设置方法：**实际开发中，默认将 数据切片大小 = BlockSize数据分块大小。


- 切片机制
  - **默认：**TextInputFormat切片机制，每个文件单独切片。
  - **小文件切片机制：**CombineTextInputFormat，可以将多个小文件逻辑规划为1个切片中。





查看源码

校验信息，先校验输入输出路径等信息，再执行其他代码

兼容旧API

在win10上查看时，注意，hadoop是本地运行，而不是集群模式。看源码时，可以关注本地运行和集群模式的区分地方。

cache缓存，现阶段可以不用看，先往下运行。缓存和优化有关。



创建客户端，Job.submit()

JobResourceUploader，本地模式没有创建

JobSubmitter，writeSplits方法是重点，怎样切片的。进入到下一个方法

FileInputFormat.getSplits()，minSize默认为1，maxSize默认long的最大值。isSplitable，是否可切片，因为压缩文件是不支持切片的，但是Loz等压缩格式支持切片。

获取文件位置，获取文件大小，按splitSize切片，Stag路径写XML文件，Job提交

splitSize




## 分布式缓存

### 目的

解决频繁从HDFS请求执行下载操作带来的过载问题， 如NN压力过大、负载过高、集群阻塞。



### 使用方式

- 存入缓存：在Driver类中，添加文件到分布式缓存。
  - `job.addCacheFile(URI)`。
- 读取缓存：
  - `Job.getCacheFiles()`
  - `FileSystem.get(Configuration)`获取FileSystem，开流。







## 数据压缩

### 目的

节省磁盘IO和网络IO

使用原则

IO密集型job，多用压缩

运算密集型job，少用压缩



### 压缩类型

| 压缩格式    | hadoop自带？ | 算法      | 文件扩展名    | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| ------- | --------- | ------- | -------- | ----- | ------------------- |
| DEFLATE | 是，直接使用    | DEFLATE | .deflate | 否     | 和文本处理一样，不需要修改       |
| Gzip    | 是，直接使用    | DEFLATE | .gz      | 否     | 和文本处理一样，不需要修改       |
| bzip2   | 是，直接使用    | bzip2   | .bz2     | 是     | 和文本处理一样，不需要修改       |
| LZO     | 否，需要安装    | LZO     | .lzo     | 是     | 需要建索引，还需要指定输入格式     |
| Snappy  | 否，需要安装    | Snappy  | .snappy  | 否     | 和文本处理一样，不需要修改       |

hadoop未安装的压缩格式，怎样安装？

①安装： 需要把压缩格式的库文件放入到hadoop的指定目录/opt/module/hadoop-2.7.2/lib/native。

安装的库文件，必须和使用的hadoop版本要适配，通常需要和自己的版本进行编译！

②配置：  在hadoop的core-site.xml中配置要使用的压缩格式在hadoop的全类名！

io.compression.codecs=压缩格式的全类名,xxx



### 压缩格式在MR中的位置

Map的输入：  考虑单个文件的大小

如果单个文件压缩完，超过130M，尽量选择可以切片的压缩格式 Bzip2和LZO

shuffle阶段（Map的输出）：  压缩和解压缩的速度

一般选择snappy(推荐)或LZO

Reduce的输出：  输出的单个文件大小

考虑输出文件是否作为下一个Job输入；

考虑输出文件的使用频率；

考虑输出文件的大小。



### Hadoop参数配置——压缩

| 参数                                       | 默认值                                      | 阶段        | 建议                               |
| ---------------------------------------- | ---------------------------------------- | --------- | -------------------------------- |
| io.compression.codecs     （在core-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec,  org.apache.hadoop.io.compress.GzipCodec,  org.apache.hadoop.io.compress.BZip2Codec,  org.apache.hadoop.io.compress.Lz4Codec | 输入压缩      | Hadoop使用文件扩展名判断是否支持某种编解码器        |
| mapreduce.map.output.compress            | false                                    | mapper输出  | 这个参数设为true启用压缩                   |
| mapreduce.map.output.compress.codec      | org.apache.hadoop.io.compress.DefaultCodec | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据     |
| mapreduce.output.fileoutputformat.compress | false                                    | reducer输出 | 这个参数设为true启用压缩                   |
| mapreduce.output.fileoutputformat.compress.codec | org.apache.hadoop.io.compress. DefaultCodec | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2         |
| mapreduce.output.fileoutputformat.compress.type | RECORD                                   | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK |





JavaAPI压缩

CompressionCodec：代表压缩的编解码器。

创建一对支持压缩和解压缩的输入和输出流。



## 常用API

#### Job串联

如何指定多个Job按照依赖关系顺序执行？

JobControl:  指定一组Job运行，还可以指定其依赖关系！

构造： public JobControl(String groupName)

添加运行的Job:  JobControl.addJob(ControlledJob aJob)


ControlledJob: 可以创建一个Job，指定这个Job的依赖Job

构造： public ControlledJob(Configuration conf)

指定依赖关系：public synchronized boolean addDependingJob(ControlledJob dependingJob)



### 小文件优化

- （1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS。（最为常用）
- （2）在业务处理之前，在HDFS上使用MapReduce程序对小文件进行合并。
  - 用Hadoop Achive打包成har文件；
  - 生成SequenceFile文件，该文件由一系列的二进制key/value组成的文件，key为文件名，value为文件内容。
- （3）在MapReduce处理时，优化切片方式或者JVM重用
  - 可采用CombineTextInputFormat提高效率。
  - JVM重用，一个MapTask运行一个JVM。重用，就是用同一个JVM，来处理多个文件。而不是每次都创建一个JVM。





# --YARN--（待补充）

修改配置文件capacity-scheduler.xml



数据倾斜

map任务，使用能够切片的压缩方式，避免不可切

reduece任务，合理合区。可以对数据进行抽样调查，看哪一部分的数据多，然后进行分区优化。



推测执行

用空间换时间。如果集群资源不足，则不适合开启这个功能。







## 源码分析

InputFormat、RecordReader读取方式

Comparator、Comparable比较方式



## 重要过程

- org.apache.hadoop.mapred.**LocalJobRunner**.Job.MapTaskRunnable#run，运行MapTask的容器
  - 其中的map.run(localConf, Job.this);——进入MapTask


- org.apache.hadoop.mapred.**MapTask**#runNewMapper，是mapper运行的全流程
  - input.initialize(split, mapperContext);——相应的InputFormat运行，根据之前的切片split结果，读取数据
  - MapOutputCollector创建缓冲区对象，if判断是否有reduce阶段
    - output = new NewOutputCollector(taskContext, job, umbilical, reporter);——有reduce阶段
    - output = new NewDirectOutputCollector(taskContext, job, umbilical, reporter);——无reduce阶段，直接输出
  - mapper.run(mapperContext);——继承的Mapper运行
  - setPhase(TaskStatus.Phase.SORT);——运行sort阶段，会根据之前的设置判断是否运行sort
  - org.apache.hadoop.mapred.**TaskStatus**#setPhase——判断是哪一个阶段的sort，map阶段的sort，还是reduce之前的sort，还是reduce之后的sort。TaskStatus是一个抽象类，有MapStatus和ReduceStatus两个子类。
- org.apache.hadoop.mapred.**LocalJobRunner**有两个内部类，MapTask和ReduceTask
- org.apache.hadoop.mapred.**ReduceTask**
  - 最重要的是分组比较器这一行代码，RawComparator comparator = 
  - 没有定义grouping比较器，则调用map的keyout定义的比较器；
  - 定义了grouping比较器，则调用。
- org.apache.hadoop.mapred.**ReduceTask**#runNewReducer，是mapper运行的全流程
  - key迭代器和value迭代器，每次获取key-value，先是获取的字节流，然后再反序列化。
  - 获取的数据，以key值先进行了排序，获取key-value迭代器，循环读取。当读取到key值相同的key-value时，进入一次reduce()。当读取到不同的key值时，则进入下一个Reducer.run()阶段。
  - 此时的key-value迭代器，key和value都是同一个对象，只不过是每次的属性发生变化。
  - 一个ReduceTask创建一个reducer
  - 最后执行reducer.run()
  - 在context.nextKey()，会使用Reduce的RawComparator的compare。




## 比较器

### 分组比较器和Map阶段KeyOut的比较器的区别

**相同点：**

- 比较器必须是RawComparator类型的子类。

**不同点：**

- 使用时机不同
  - 分组比较器，只用在Reduce的分组阶段。也就是Reducer之前的那个阶段。
  - KeyOut的比较器，用在Map的sort阶段，Reduce的sort阶段，也可用在Reduce的分组阶段。
- 使用优先级不同
  - 分组时使用比较器会进行判断
  - 分组比较器优先选用job.setGroupingComparatorClass()中设定的比较器。
  - 如果没有设定分组比较器，则使用Map阶段KeyOut的比较器。



### 自定义Comparator

#### 2种实现自定义比较的方式

- Bean类，继承WritableComparator类

  - 重写compare方法。

- 自定义Comparator，实现RawComparator接口，重写compare方法

  - 实现RawComparator需要重写两个compare方法
    - 一个来自于父辈的RawComaprator
    - 另一个来自于爷爷辈的Comparator

    - 父辈compare方法，需要将字节数组变成字节流，并用相应的Mapper的KeyOut类型来反序列化字节流。
    - 爷爷辈compare方法，将值进行比较，写业务逻辑。


```
public class UpComparator implements RawComparator<Text> {

    //key1、key2是Mapper中的KeyOut类，该类需要实现Writable，以拥有序列化encode和反序列化decode方法。
    //DataInputBuffer用于将字节数组转换成字节输入流
    private Text key1 = new Text ();
    private Text key2 = new Text ();
    private DataInputBuffer buffer = new DataInputBuffer ();


    /*
    * 父实现类RawComparator的compare方法。
    * 将序列化的字节数组变成反序列化后的字节数组，并输出成相应KeyOut类型，如这里的Text key1。
    * */
    @Override
    public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2) {
        try {
            buffer.reset (b1,s1,l1);
            key1.readFields (buffer);
            buffer.reset (b2,s2,l2);
            key2.readFields (buffer);
        } catch (IOException e) {
            e.printStackTrace ();
        }


        return compare (key1,key2);
    }

    /*
    * 爷爷实现类Comparator的compare方法。
    * 将值进行比较，返回比较的业务逻辑。
    * */
    @Override
    public int compare(Text o1, Text o2) {
        long v1 = Long.parseLong (o1.toString ());
        long v2 = Long.parseLong (o2.toString ());

        return v1<v2?1:-1;
    }
}
```


## 实现类的学习方式

通过什么方式来知道，如何实现RawComparator？（实现类的学习方法）

- 可以抄，参照实现了RawComparator接口的实现类。看它们是怎么写的。比如WritableComparator，它和我们自定义的Comparator都是实现了RawComparator。所以可以参照WritableComparator，进行分析，来实现自己的实现类。

例如：查看WritableComparator，发现需要给变量名buffer创建对象。

buffer是org.apache.hadoop.io.DataInputBuffer对象。

```java
static {                                        // register this comparator
  WritableComparator.define(UTF8.class（）, new Comparator());}
```





Java中字节和字符的转换（忘记）

new一个String()，构造方法中有bytes；

Charset；

ByteBuffer；

但是这里的字节数组不能是序列化后的字节数组。

关于序列化，还是有问题。有时候传参是byte[]，但是为什么还不行。（疑问）



## 调度器

### 1.调度器的分类

#### 1.1 FIFO调度器

​	特点：单队列，先到先服务

​	弊端： 如果队首有一个资源需求大的Job，所需的资源没有得到满足！此时后续的Job都需要等待！

​				资源利用率低！

#### 1.2 容量调度器

​	特点： **容量**。集群资源利用率高！满足不同类型Job和用户的需求！默认调度器！

- 每个队列可以设置占用集群资源容量的最低和最高限制

  - 限制每个用户或每个Job使用资源的限制

    - 队列中多余的资源可以匀给其他队列使用

      **多个队列！队列内部FIFO(先到先服务)**！

测试：

```shell
hadoop jar hadoop-mapreduce-examples-2.7.2.jar wordcount /mapjoin /output2
```

默认情况提交时，所有的Job都会提交到default，取决于mapred-default.xml

```
mapreduce.job.queuename=default
```

设置容量调度器为多个队列：

​		修改$HADOOP_HOME/etc/hadoop/capacity-scheduler.xml

​		不需要重启YARN，执行

```
yarn rmadmin -refreshQueues
```

提交Job到不同的队列：

```
hadoop jar hadoop-mapreduce-examples-2.7.2.jar  wordcount -Dmapreduce.job.queuename=a /mapjoin /output3
```

#### 1.3 公平调度器

​	公平调度器是在容量调度器的基础上开发的！

​		区别：调度的策略不同！ 容量调度器在队列内部FIFO进行调度！

​					公平调度器，基于最大最小公平算法，默认基于内存进行计算，为当前队列中所有的待运行的Job,

​	平均地分配资源！

​		特点： 同一个队列中，小的Job占优势，可以快速地完成，大的job不至于饿死！

## 推测执行

### 1.数据倾斜

MapTask端的数据倾斜： 由切片不均匀造成！

ReduceTask端的数据倾斜： 由分区不均匀造成！




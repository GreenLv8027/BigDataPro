## Zookeeper

### 概述和应用

#### 1.概念

**1.1节点**

是指Zookeeper节点目录下的节点

状态有4种

- 持久化目录节点：
  - 客户端与Zookeeper断开，也依然存在的节点。
- 持久化顺序编号目录节点：
  - 客户端与Zookeeper断开，也依然存在的节点。
  - 该节点名称会用顺序编号。
- 临时目录节点：
  - 客户端与Zookeeper断开，会被删除的节点。
- 临时顺序编号目录节点客户端与Zookeeper断开，会被删除的节点。
  - 该节点名称会用顺序编号。



**1.2服务器**

是指运行Zookeeper的服务器

状态有4种

- LOOKING：寻找Leader状态，处于该状态需要进入选举流程
- LEADING：领导者状态，处于该状态的节点说明是角色已经是Leader
- FOLLOWING：跟随者状态，表示Leader已经选举出来，当前节点角色是follower
- OBSERVER：观察者状态，表明当前节点角色是observer

#### 设计模式理解

- 是一个基于观察者模式设计的分布式服务管理框架，它负责**存储和管理大家都关心的数据**，然后接受**观察者**的注册，一旦这些**数据的状态发生变化**，Zookeeper就将负责**通知**已经在Zookeeper上注册的那些**观察者做出相应的反应**。

- “只存储大家都关心的文件，不是存储大数据”的理解。如Hadoop的配置文件，core-site.xml。

  之前的做法：在一台主机上修改配置文件，然后用xsync分发到集群。

  现在的做法：用zookeeper设置监听同一个Znode，可以实现集群服务器的配置文件同步。

  注意：每个Znode默认的数据量上限是1M，超过1M就存储不了。

- **核心思想：**观察者模式，文件系统+通知机制



#### 应用在何处？

- 为分布式应用提供协调服务，分布式应用如HDFS、MR、YARN。

- 多作为集群提供服务的中间件。
- 主要应用于大数据开发中的，统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等场景。
- 该框架相当于大数据框架中的润滑剂。是大数据开发工程师必须会的框架之一。



#### 应用场景理解

所有的应用场景，都离不开ZK的核心思想。

**1.统一集群管理，查看集群中有多少个服务器可用？**

- 每台集群设备Client，与Zookeeper Service连接，有一个临时进程。此时Client将节点信息写入该Zk Service中的一个Znode中，创建为临时节点。当该服务器停止工作，Client下线，临时进程结束，ZK Service中对应的临时节点删除。
- 查看Znode中，临时节点的数量，就可以查看有多少机器在工作。





#### 其他问题（有疑问）

**1.Zookeeper版本问题**

2.5.0以下的版本，会有因为create多层dir创建无效而导致Client崩溃的bug。最好使用2.5.0以上的版本。



**2.XX-bin.tar.gz 和XX.tar.gz区别。**

- 带bin，是官方编译好的文件，可以直接使用。
- 不带bin的，需要编译。





4字命令，需要安装nc（Linux扩展）



create只能创建节点，节点下怎么存储数据？（疑问）



### ZK内部原理

#### paxos协议，半数机制

**1.了解paxos协议**

Paxos 算法解决的问题是一个分布式系统如何就某个值（决议）达成一致。

paxos协议，要求分布式系统中有**半数**以上节点存活，才能判断分布式系统正常。



**2.半数机制在ZK中的使用**

半数机制，在ZK内部原理中使用得很多。

- 集群启动，判断集群启动是否成功
  - 要求半数以上节点启动成功
- 选举机制，判断哪个节点成为leader
  - 要求半数以上节点投票给某个节点
- 写数据流程，判断集群是否写数据成功
  - 半数以上节点（包括leader）写成功



**3.为什么zookeeper启动奇数个节点？**

资源节约的角度，因为半数机制。

集群可用的条件是，集群中半数以上的服务器存活。而奇数个服务器和偶数个服务器，如3和4，集群可用标准都是2台以上服务器存活。那么4台就比3台多一台服务器配置，达成集群可用的情况下，多用了一台服务器。





#### 选举机制

**1.谁有优势？**

myid大的，有优势。赢者通吃。

空白集群启动，在半数服务器启动时，启动成功的服务器中myid大的成为Leader。

**2.选举资格**

读写时，如果Leader挂了，重新选举。此时，在Leader挂之前，写成功的服务器才有资格选举，未写成功的服务器不能参与选举，只能投票。此时还是myid大的，有优势，成为Leader。



#### 脑裂

指一个集群中出现2个leader的情况。

问题产生：这种情况会发生在，leader挂机后，剩下的follower选举出新的leader。此时若改变旧leader中的zoo.cfg配置，新增节点服务器，则启动后的节点服务器与之前的follower服务器无法正常选举通信，导致出现2个leader。

扩容注意事项：ZooKeeper集群扩容时，Leader节点应最后启动，以避免脑裂问题。因为在Leader节点重启前，所有的Follower节点zoo.cfg配置已经是相同的，他们基于同一个集群配置两两互联，做投票选举。



#### 监听器原理（无法理解）

- main()线程
  - 创建Zookeeper客户端时，会创建2个线程，1个负责网络连接通讯connet，1个负责监听listene。
- 监听事件（无法理解）
  - 如getChildren方法中，有2种Watcher方式。
    - 是否使用Watcher的Boolean值，此时使用的Watcher是创建ZK时的Wathcer；
    - 传递一个Watcher对象，此时可以使用创建ZK时的Wathcer，也可以再创建1个Wather。
- Watcher监听者
  - 监听到指定路径数据发生变化，则调用process()方法。
  - Watcher只能进行一次监听事件行动，执行process()方法后，就注销了。若需要重复监听，则应在process中，再调用一次监听事件。

![Snipaste_2020-03-01_22-29-17](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-01_22-29-17.png)



#### Start属性

ephemeralOwner

- 状态中的ephemeralOwner = 0x0，代表当前节点是一个永久节点

dataLength

numChildren

dataVersion，版本号，ZK是乐观锁



### ZK操作

Linux上的常用命令

启动zookeeper

- 服务端：  bin/zkServer.sh start|status|stop|restart
- 客户端：  bin/zkCli.sh -server  zk服务实例所在主机名:端口号
  - localhost:2181可以省略！
  - bin/zkCli.sh 默认连接本机



create [] path data，创建节点

- create -s -e path data： 创建临时带序号的节点
- create -s  path data： 创建带序号的节点
- create  -e path data： 创建临时节点
- create  path data： 创建永久节点


get path，查看节点保存的数据

set path data，修改节点保存的数据

delete path，删除单个非空节点

rmr path，删除当前节点级联删除所以子孙节点



IDEA上的

zookeeper的API介绍：https://segmentfault.com/a/1190000012262940

创建ZK客户端








### ZK的集群部署

#### 集群可用性

Zookeeper集群，基于paxos分布式协议设计。





#### 集群读写

##### 写流程

- 写过程：
  - client连接的server实例，如果不是leader，则follower会将请求发送给leader。
  - leader先写成功，才会进行广播。所以leader是一定会读写成功的。
- 写流程成功判断：
  - 只要有半数server写成功，那么Leader就认为集群写成功。
- 数据一致：
  - ZK集群中，每个follower节点需要与leader节点数据保持一致。

##### 读流程

client请求任一节点，不管是leader还是follower，都可以直接返回数据。



#### 集群配置

- zoo.cfg
  - 在{ZK_HOME}/conf/zoo.cfg中增加集群的配置，结构为`server.id=ip:port1:port2`
    - server.id，是节点id，只能是数字，因为在选举时需要通过id比大小。
    - ip，是节点ip
    - port1，是节点间非选举通讯端口
    - port2，是节点间选举通讯端口


- myid文件
  - 在zoo.cfg中的tmp目录下，创建myid文件，
  - myid需要与zoo.cfg中的server.id一致。选举机制时需要myid来进行大小的排序。




#### 集群数据同步(待补充)

集群只有主从复制，没有读写分离。








### 使用问题

**1. zookeeper启动失败**

Starting zookeeper ... FAILED TO START

可能进程中有zookeeper

集群中只能start一个服务器，第二个服务器的zookeeper会start失败。



**2. IDEA中使用zookeeperAPI的问题**

总是会报出connection loss错误，这与IDEA有关。老师说：“用Eclipse时，没有遇到过这个问题”。

可能的原因：IDEA会访问上一次的zookeeper服务。



分布式锁

一个应用场景？



后续开始讲Hive，重点。会将5天以上。






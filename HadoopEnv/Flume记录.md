Flume

# --概念--

## 作用

Flume是一个针对日志数据进行高效收集、聚合和传输的框架	。

Flume是Java写的，启动一个Java程序，需要启动一个进程，Flume是启动了一个Agent。

Agent包含source,sink,channel三部分。

## 框架理解

### 运行流程

#### 总体流程

![Snipaste_2020-03-11_16-40-31](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-11_16-40-31.png)

source从外界接收数据，channel作为缓冲，sink向外输出数据。



#### Agent内部流程

![Snipaste_2020-03-11_17-45-48](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-11_17-45-48.png)

**可以设置的组件**

- ChannelSelector：2种，Replicating（复制）、Multiplexing（多路复用）
- SinkProcessor：3种，DefaultSinkProcessor、LoadBalancingSinkProcessor、FailoverSinkProcessor
- Interceptor自定义。
- Source、Channel、Sink，配置文件。




### 核心概念

可按着<Agent内部流程>图查看。

**Agent**: 一个Flume的java进程，是使用flume的基本单元

**Source**:  负责对接数据源，将读到的数据封装为event，写入channel。数据源是不同的类型，就使用不同的source。相当于MR中的FileInputFormat。

**Sink**:  负责从channel中读取数据，写出到指定的目的地。写出的目的地不同，就使用不同的sink。

**Channel**：source和sink之间的缓存。由于有channel存储，就可允许sink和source运行在不同的速率上。

**Event**: flume中数据传输的最基本单位。包含一个header(map)和 body(byte[])

**Interceptors**: 拦截器是在source向Channel写入event的途中，对event进行拦截处理。

**Channel Selectors**： 当一个source对接多个Channel，会使用选择器选取合适的channel。

**Sink Processors**:  用于从多个sink组成的sink组中，同一时刻，挑选一个sink去channel读取数据。

**Channel Processor：**使用在source写入event到channel的阶段。在该阶段中，处理event的操作，如调用拦截器，调用channelSelector等。



### 基本常识

1个source可以对接多个channel；

1个sink，只能对接1个channel。

多个sink可以配置成1个组，增强可用性，1个组内只能有1个sink从channel读取数据，其他sink作为备用，在当前活动sink挂掉后，启用备用sink。



## 其他知识

**版本**

og，捐赠前的版本

ng，捐赠给apache之后的版本

目前，我们使用的flume ng 1.7版本





### netcat

**用途：**网络通信传输的小工具，TCP和UDP扫描器。网络工具中的瑞士军刀。



**命令解释**

```shell
-u UDP模式,[netcat-1.15可以:远程nc -ulp port -e cmd.exe，本地nc -u ip port连接，得到一个shell.
-v 详细输出——用两个-v可得到更详细的内容
-w sec 指定超时的时间
-z 将输入输出关掉——用于扫描时
-l 监听模式，用于入站连接
-k 为多个链接保持入站socket打开

```



**监听端口**

```shell
nc -kl IP地址 端口号
nc -kl hadoop103 4444
```



**向其他服务器发送数据**

```shell
nc IP地址 端口号（对方监听的端口）
nc hadoop103 4444
```



**检测UDP端口情况**

```shell
nc -uvzw sec IP地址 端口号段LO-HI
例如：nc -uvzw2 192.168.0.80 1-140
```



**检测TCP端口情况**

```shell
nv -vzw sec IP地址 端口号段LO-HI
例如：nc -vzw2 192.168.0.80 1-140
```





**补充知识**

UDP与TCP

- UDP和TCP都属于，TCP/IP协议4层分层模型中的传输层。
- UDP特点
  - 面向无连接,传输数据快,数据不安全,容易丢失
  - 传输数据是以数据包的形式传输的
  - 数据包的大小限制在64k以内
- TCP特点
  - 面向连接,传输数据慢,数据安全,不容易丢失
  - 传输数据以流的形式,大小没有限制



# --使用与配置--

## Flume命令

启动flume-agent

```shell
$FLUME_HOME/bin/flume-ng agent -c conf/ -n $agent_name -f agent配置文件路径 -Dflume.root.logger=flume日志等级,console控制台输出

--conf,-c 明确flume的全局参数配置文件夹路径；
-n 给本次agent设置名称，需要与agent的配置文件中的agent名字一致
--conf-file,-f 本次agent（局部参数）的配置文件路径
-Dproperty=value Java系统的配置参数值
flume.root.logger，flume的logs日志信息显示等级，INFO\DEBUG\TRACE
console 将flume信息打出在控制台

例子：
$FLUME_HOME/bin/flume-ng agent -c conf/ -n a1 -f flumeagents/netcatSource-loggersink.conf -Dflume.root.logger=DEBUG,console
```









## 可自定义组件的配置

### 组件配置的编写与使用方法

**1. 组件配置可以满足的需求：**Sink、Channel、Source，需要什么就配置什么。基本上都支持了企业中所有常用的日志类型和输出目的地。

**2. 使用方法**：将组件配置文件，在写成一个conf文件。用Flume命令启动，`--conf-file,-f`后面写上该conf文件的路径。

**3. 如何编写组件配置**

以一个agent配置文件为例

```shell
#1. 命名每个组件 a1代表agent的名称 
#a1.sources代表a1中配置的source,多个使用空格间隔
#a1.sinks代表a1中配置的sink,多个使用空格间隔
#a1.channels代表a1中配置的channel,多个使用空格间隔
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#2. 配置需要配置的组件参数，开头是固定的
# 配置source
a1.sources.r1.type = netcat
a1.sources.r1.bind = hadoop103
a1.sources.r1.port = 44444

# 配置sink
a1.sinks.k1.type = logger
a1.sinks.k1.maxBytesToLog=100

# 配置channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000

#3. 绑定和连接组件
#sources连接channel，sink连接channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```









### Sink

#### **1.Logger Sink**

**1.1概念**

作用：采用 logger以info级别将event输出到指定的路径（文件或控制台）。

**1.2必须配置参数**

- type，组件名称
  - 配置为`logger`

**1.3可选配置参数**

- maxBytesToLog（1.7版本，无效）
  - log能读入的最大的event body字节数。
  - 默认值，16





#### **2. HDFS Sink**（有疑问）

**2.1概念**

支持的输出文件格式：text、SequenceFile，这2个文件格式都可使用压缩。

目录滚动条件：基于时间周期性滚筒 OR 基于文件大小滚动 OR 基于events数量滚动

目录滚动命名：上传的路径名可以包含 格式化的转义序列，转义序列会在文件/目录真正上传时被替换

滚动作用：按时间节点归纳文档，可以根据数据产生的时间戳或主机名对数据进行分桶或分区

使用HDFS Sink的前提：已安装Hadoop

**2.2必须配置参数**

- type，组件名称
  - 配置为`hdfs`
- hdfs.path，上传的路径名
  - 可以包含 格式化的转义序列，在真正上传时会按照上传时间修改。



**2.3可选配置参数**

**2.3.1文件滚动策略**，0代表禁用

- hdfs.rolInterval，滚动间隔
  - 每间隔多少秒滚动一次文件
  - 默认值，30
- hdfs.rollSize，滚动文件大小
  - 文件一旦达到多少bytes就触发滚动
  - 默认值，1024
- hdfs.rollCount，滚动计数
  - 文件一旦写入多少个event就触发滚动
  - 默认值，10





**2.3.2文件类型和压缩类型**

- hdfs.codeC，压缩类型
  - 支持的压缩类型：gzip, bzip2, lzo, lzop, snappy
- hdfs.fileType，文件类型
  - 当前支持： `SequenceFile`, `DataStream` or `CompressedStream`。
  - 默认值：SequenceFile
  - DataStream 代表不使用压缩也就是纯文本
  - CompressedStream 代表使用压缩

**2.3.3目录滚动策略**

- hdfs.round，时间元整开关
  - 默认值：false
  - 代表时间戳是否需要向下舍去，如果为true，会影响所有的基于时间的转义序列（escape sequences），除了%t（制表符 \t）
- hdfs.roundValue，时间元整倍数值
  - 默认值：1
  - 将时间戳向下舍到离此值最高倍数的一个时间，小于等于当前时间
  - 比当前时间小的最高倍数。数据倍数，向下取整
- hdfs.roundUnit，时间元整时间单位级别
  - 默认值：second
  - 可选second\minute\hour

**时间滚动举例理解**

hdfs.roundUnit=minute

hdfs.roundValue=10

会把 11时50分-11时59分产生的数据文件，都放在 1150（也就是分针级，10倍数）的文件夹中



**2.3.4最关键的属性**（为什么是最关键的 ？疑问）

- hdfs.useLocalTimeStamp，使用本地时间戳
  - 默认值：fasle
  - 使用flume进程所在的本地时间，替换event header中的timestamp属性，替换后，用来影响转义序列



**2.4其他问题**

**2.4.1和时间相关的转义序列的要求**

event的header中，有timestamp的属性名，值为时间戳。



**2.4.2使用本地timeStamp的方法**

配置hdfs.useLocalTimeStamp=true；

使用TimestampInterceptor生成时间戳的key。



**2.4.3滚动文件的状态**

.tmp，是正在滚动的文件。



#### 3. Avro Sink

**3.1概念**

Avro是大数据中的序列化格式

可与Avro Source配合使用

**3.2必须配置参数**

- type
  - `avro`
- hostname
  - source绑定的主机名
- port
  - source绑定的端口号



#### 4.File Roll Sink

**4.1概念**

将event写入到本地磁盘！

数据在写入到目录后，会自动进行滚动文件！

**4.2必须配置参数**

- type
  - `file_roll`
- sink.directory
  - 数据写入的目录

**4.3可选配置参数**

- sink.rollInterval
  - 每间隔多久时间滚动，单位秒
  - 默认值：30
  - 若设置为0，则禁用滚动，所有数据写入1个文件。



### Channel

#### **1.Memory Channel**

**1.1概念**

原理：将event存储在内存中的队列中

适用场景：一般适用于高吞吐量的场景

缺点：但是如果agent故障，会损失阶段性的数据。

**1.2必须配置参数**

- type，组件名称
  - 配置为`memory`

**1.3可选配置参数**

- capacity
  - 存放event的容量限制
  - 默认值，100





### Source

每个source，都会默认配置selector.type=replicating。所以1个source对应多个channel，不需要额外配置sourceSelector。



#### **1. NetCat TCP Source**

**1.1概念**

工作原理：非常类似nc-k -l [host][port]

工作原理举例：该source可以打开一个端口号，监听端口号收到的消息。将消息的每行，封装为一个event!

**1.2必须配置参数**

- tpye，组件名称，必须为`netcat`
- bind，要绑定的ip地址或主机名
- port，要绑定的端口号




#### 2. Exec Source

**2.1概念**

**作用：**Exec Source在启动后执行一个linux命令，期望这个命令可以持续地在标注输出中产生内容！

**要求：**

执行的linux命令

- 合适的命令，可以产生持续数据的命令，如cat ，tail -f 
- 不合适的命令，只能产生一条信息，之后就结束的命令，类似date

进程：一旦命令停止了，进程也就停止了

缺点：ExecSource是异步source，有丢失数据的风险

**2.2必须配置参数**

- type，组件名称
  - 配置为`exec`
- command，要执行的命令
  - 如linux命令，tail -f 文件路径，获取文件的更新信息

**2.3可选配置参数**

暂无

**2.4缺点**

异步source的问题
- 无法保证出现故障时，将event放入channel，并通知客户端
- 异步Source在异常情况下，若无法把从客户端读取的event进行缓存，则会丢失数据
- 建议替换为，Spooling Directory Source，Taildir Source






#### **3. Spooling Directory  Source**

**3.1概念**

**作用：**监控一个目录中新放入的文件的数据

原理：一旦发现监控目录有新放入的文件数据，就将数据封装为event! 在目录中，已经传输完成的数据，会使用重命名或删除来标识这些文件已经传输完成！

适用场景：已经在一个目录中生成了大量的离线日志，且日志不会再进行写入和修改。

优点：可靠的，不会丢文件。

**实现可靠的要求：**放入目录的文件必须是**一成不变**（不能修改）的文件，且**不能重名**。不满足要求，agent会报故障。

**3.2必须配置参数**

- type，spooldir
- spoolDir，监控的目录路径

**3.3可选配置参数**

- fileSuffix，文件后缀
  - 为已经读完的文件标识后缀
  - 默认值：.COMPLETED
- deletePolicy，删除的规则
  - 文件读完后，是立刻删除还是不删除 `never` or `immediate`
  - 默认值：never
- fileHeader，文件头
  - 是否在header中存放文件的绝对路径属性
  - 默认值：false
- fileHeaderKey，文件头key
  - 存放的绝对路径的key
  - 默认值：file





#### **4. Taildir Source**

**4.1概念**

**作用：**以接近实时的速度监控文件中写入的新行

**可靠性的要求：**检测文件中写入的新行，并且将每个文件tail的位置记录在一个JSON的文件中。agent挂掉，重启后，source从上次记录的位置继续执行tail操作

优点：可靠的，不会丢文件。

写入新行的判断：读取的行需要封闭，比如换行\n。否则不会读取未封闭的最后一行。

**记录读取的位置：**taildir_position.json，存储着读取的位置信息，“读取到了哪一行”。用户可以通过修改Position文件的参数，来改变source继续读取的位置！如果postion文件丢失了，那么source会重新从每个文件的第一行开始读取(重复读)！

**4.2必须配置参数**

- type
  - `taildir`
- filegroups，组名
- filegroups.filegroup.filename
  - 一个组中可以配置多个文件的路径







#### **5. Avro Source**

**5.1概念**

source读取avro格式的数据，反序列化为event对象

启动Avro Source时，会自动绑定一个RPC的端口！

**5.2必须配置参数**

- type
  - `avro`
- bind
  - 绑定的主机名或ip地址
- port
  - 绑定的端口号





### Sink Processor

#### 1.Default Sink Processor

agent中只有一个sink，此时就使用Default Sink Processor，不强制用户来配置Sink Processor和sink组！



#### 2.Failover Sink Processor

**2.1概念**

故障转移的sink处理器！ 

该sink处理器会维护一组有优先级的sink！默认挑选优先级最高（数值越大）的sink来处理数据！

故障的sink会放入池中冷却一段时间，恢复后，重新加入到存活的池中，此时在live pool(存活的池)中选优先级最高的，接替工作！

**2.2必须配置参数**

| 参数值                             | 默认值       | 解释                |
| ------------------------------- | --------- | ----------------- |
| **sinks**                       | –         | 空格分割的，多个sink组成的集合 |
| **processor.type**              | `default` | `failover`        |
| **processor.priority.**sinkName | –         | 优先级               |



#### 3.Load balancing Sink Processor（待补充）

**3.1概念**

2种均衡方法：round_robin` or `random

负载均衡的原理：让多个激活的sink间的负载均衡(多个sink轮流干活)！

**3.2必须配置参数**

- **processor.sinks**
  - 空格分割的，多个sink组成的集合  
- **processor.type** 
  - 默认值：  `default` 
  - 更改值：`load_balance`

**3.3可选配置参数**

- processor.selector
  - 默认值：round_robin
  - `round_robin`, `random` or FQCN of custom class that inherits from `AbstractSinkSelector`（待补充）




### Channel Selector

#### 1. replication Channel

每个source，给所有channel发送数据。





#### 2. Multiple Channel

1个source，只给指定的channel发送数据





## Flume事物

非常重要的，面试考原理的地方

### 理解事物

与mysql事物，是不一样的。

Flume的事物，是一批events。

这一批events（事物）的特点：原子性

事物所处的flume阶段：写入到channel和从channel中移出

分段的概念，source到channel，channel到sink。

这2段都有，read读和write写。读写数据不能丢失。



2个事物

put事物

take事物



take事物数据重复

channel中，takeList从channel拿走数据后，channel中被拿走的部分缓冲空间，会被冻结起来，不能put。

JUC，信号量



put事物数据重复

1个source对接多个channel，可能会出现put事物数据重复。如某些channel的put完成，但另一些channel的put未成功。

channel processor无法记录哪个channel的put成功，但是它可以处理channel的put异常。接收到异常，





事物的实现：由channel提供。source和sink在put和take数据时，从channel中获取已经定义好的事物。



rollback是谁发出的？channel中的put和take事物，在try...catch...中捕获到



channel processor的理解

它是一个调度者，并不参与source、channel、sink之间的实际操作，只是按顺序来启用他们。



事物中的数量关系

这里的数量，指的是什么数量？（有疑问）

source、sink中传输的数量

channel中缓存的数量

channel的put和take事物中，putList和takeList内存储的数量。

batchSize

transcationCapacity



一批数据，封装成多少个event，看source怎么写。



### 消息队列或传输框架的设计语义








常用的案例

读取tail的信息，多个文件的更新信息。

HDFSSink用得多。



properties文件中，多个属性值用空格` `分隔，而不是逗号。



简单案例二

HDFS上，没有出现对应的传输文件。

在启动flume的命令中，agent的name写错了。



复杂案例一

启动顺序，先启动agent2和agent3，因为若先启动agent1，数据读取时，由于agent2和3未启动，agent1的sink无法发送数据。



拓扑结构

简单串联

让外部机器无法直接连接到HDFS，而是指定特定的计算机可以连接HDFS。保护HDFS的数据安全。





英语单词

escape suquence转移序列

额外的Hadoop HA问题

如果有一个NN一直没有启动，在很久之后的一次服务器重启中，用ZK来选出Active的NN。若选到了这个NN，那么在这个NN没有启动的时间内的元数据，就都没有了？

若不重启，再启动这个NN，是否就直接从当前的Active的NN中获取这一段时时间的NN更改。
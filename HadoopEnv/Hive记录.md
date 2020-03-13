

# Hive

## --概念--

### 定义

**本质：**

是一个使用HQL语言实现业务逻辑，通过MR模版，对HDFS上的数据进行数据分析的软件工具。

hive的数据是存储在hdfs，hive建表后要和数据关联，关联后，每次对hive表的查询，实质是运行一个MR！

**需要做的事情：**

1. 原数据，要存在HDFS上
2. 元数据，要通过hive创建表结构，存储在MySQL上
3. HQL，直接实现业务逻辑



**作用：**

1. 分析HDFS分布式存储上的结构化数据。
2. 将结构化的数据文件映射为一张表，并提供类SQL查询功能。
3. 提供了JDBC驱动和命令行工具，让用户连接Hive。



**缺点：**

1. HQL表达能力有限。
2. 效率比较低。





**与RDMS的区别（Relational Data Management System关系型数据库管理系统，如MySQL）**

Hive：

1. 只能分析结构化数据
2. 不是RDMS关系型数据库，本身不存储数据。
3. 不支持行级别更新，如SQL更新类操作，不支持UPDATE SET，只支持INSERT。因为hdfs不支持随机写，只支持追加写，所以在hive中数据只能insert和select，不能delete和update。
4. 不支持事物，不是OLTP（On-Line Transaction Processing联机事务处理过程），而是OLAP（On-Line Analytical Processing联机分析处理过程）。



### 架构

![Snipaste_2020-03-04_19-50-19](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-04_19-50-19.png)

**主要的4大块**

1. 基于HQL语言，编写业务；
2. 基于MR，进行数据分析；
3. RDMS，存储表结构（元数据）；
4. HDFS，存储原数据。



**关心的3个部分**

1. Meta Data，元数据存储，
2. HDFS，原数据存储
3. YARN，执行MR



**Hive的Client解释**

- 3种用户接口，操作Hive的方式
  - CLI，命令行，hive shell
  - JDBC/ODBC，java访问hive
  - WEBUI，浏览器访问hive
- 编译器，负责将HQL语句编译成MR






### 数据类型

#### 1.基本数据类型

| Hive数据类型  | 长度                         | Java数据类型 |
| --------- | -------------------------- | -------- |
| TINYINT   | 1byte有符号整数                 | byte     |
| SMALINT   | 2byte有符号整数                 | short    |
| INT       | 4byte有符号整数                 | int      |
| BIGINT    | 8byte有符号整数                 | long     |
| BOOLEAN   | 布尔类型，true或者false           | boolean  |
| FLOAT     | 单精度浮点数                     | float    |
| DOUBLE    | 双精度浮点数                     | double   |
| STRING    | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | string   |
| TIMESTAMP | 时间类型                       |          |
| BINARY    | 字节数组                       |          |

**注意：**

1. byte, short, long这些整数类型的写法与Java中不同，需要注意别写错。
2. （经验）时间类型，一般不使用。因为表示时间可以使用String类型。Hive是Java写的，可以识别指定时间格式的String类型值。



#### 2.结构数据类型，输出与输入格式（待实现）

- Array，数组
  - **理解：**一组具有相同类型的变量的集合。
  - **hive定义格式：**`name array<dataType>`
  - **hive输入格式：**`value_value2_...`
  - **访问格式：**`name[loc]`
- Map，映射，键值对
  - **理解：**一组键值对的元组集合
  - **结构：**key:value，其中key和value都是可以改变的。
  - **hive定义格式：**`name map<key的dataType, value的dataType>`
  - **hive输入格式：**`key1:value1_key2:value2`
  - **访问方式：**`name[key]`
- Struct，结构体
  - **理解：**一组不同类型的数据容器，相当于将多种类型的Array数组放到了一个统一名称的容器中。
  - **结构：**propertyName:value，其中property是属性名，是不可以改变的。
  - **hive定义格式：**`name struct<property1: property1的dataType, property2: property2的dataType>`
  - **hive输入格式：**`value1_value2`
  - **访问方式：**`name.property`



**名词释义：**

- name，变量名；
- key，键；
- value，值；
- property，属性值，相当于元数据，定义后不可改变；
- dataType，数据类型，hive定义的数据类型；



**注意：**

- 要区分Map和Struct：Map的key是可以变化的，而Structstruct要求每条数据的property属性名是一致的，不可变化。
- JSON类型等其他类型的数据，在输入到Hive之前，需要进行数据转换。思路是：用Java的Json解释器API，提取数据，再按照Hive数据类型要求来拼接字符串。（待实现）
- 集合类型，元素的分隔符必须是一致的。



#### 3.类型转化

**3.1隐式转化**

1. 整数类型，可以隐式转化为范围更大的整数类型。
2. 所有整数类型、FLOAT、STRING类型，可以隐式转化为DOUBLE类型。
3. TINYINT、SMALLINT、INT，可以转化为FLOAT。
4. BOOLEAN，不能转化为任何其他类型。



**3.2强制转化**

cast('值'X as 类型Y)

**解释：**可将X的值，转化为Y类型。

**转化失败：**返回空值NULL。



#### 4.Hive分隔符

| 属性                   | hive默认分隔符 | 常用分隔符 |
| -------------------- | --------- | ----- |
| 行                    | \n        | \n    |
| 字段                   | ^A        | \t    |
| map,array和struct每个元素 | ^B        | _     |
| map中key-value的分隔符    | ^C        | :     |

^A： 在vim中，先进入编辑模式，再按ctrl+V,再按ctrl+A

大小写： hive中SQL语句是不区分大小写，除非是一些关键的属性名（严格区分大小写）！



**自定义分隔符**

```hive
row format delimited fields terminated by '\t'
collection items terminated by '_'
map keys terminated by ':';
```

自定义分隔符，可以用在建表，也可以用在查询语句中，规定查询结果的分隔符。

自定义分隔符写在哪？

- 建表语句中，自定义分隔符写在table后；

- 查询导出语句中，自定义分隔符写在insert语句后，select语句前。




### 数据管理

名词解释

metastore，即元数据

#### hive中原数据的存储位置

- 库
  - HDFS上的对应目录，`/user/hive/warehouse/库名.db`
  - 其他库，是在default库下建立`库名.db目录`的。
- 表
  - HDFS上对应的库目录的子目录，`/user/hive/warehouse/库名.db/表名`
- 表数据
  - HDFS上表目录下的文件




#### metastore的内容（元数据的内容）

- DBS表，每个库的记录；
- TBLS表，每个表的记录；
- COLUMNS_V2，每个字段信息的记录




#### metastore的数据库

**1.默认方式**：derby

- 缺点：
  - 默认只会从当前目录下读取metastore_db的库文件；
  - 不支持多实例同时使用一个库


- 需要改进：配置mysql作为元数据存储的数据库。



**2.改进方式**：MySQL

- 需要满足条件：
  - 提供一个可以从任意机器访问mysql服务实例的用户
  - 修改hive-site.xml中的参数`javax.jdo.option.ConnectionDriverName`
  - lib中有mysql连接驱动jar包，`mysql-connector-java-5.1.27-bin.jar`。注意这个jar包是通过解压`mysql-connector-java-5.1.27.tar.gz`获得的。









## --安装与配置--

### 环境要求

1. 安装了Java，并配置JAVA_HOME路径；
2. 安装了Hadoop，并配置了HADOOP_HOME路径；
3. 需要更换元数据的存储数据库，安装了mysql服务端、客户端，msyql连接jar包放在hive的lib文件夹中。
4. hive解压即可
5. 建议配置HIVE_HOME到/etc/profile
   1. 可以直接在任意路径使用bin目录下的工具
   2. 默认hive在启动时，读取HIVE_HOME/conf中的配置文件

可以运行的地方：hive只是个客户端，安装在任意机器(安装了java和hadoop)即可



### Hive常用配置属性

#### 数据存储位置

**1.default库的存储位置**

- 默认存储位置：/user/hive/warehouse
- 自定义存储位置
  - 更改了存储路径后，default库的路径，在元数据的DBS表中，查看DB_LOCATION_RUI，显示的还是原路径。但是在创建表时，表是在新的路径下创建的，这里与其他库有不同。
  - 配置文件：HIVE_HOME/conf/hive-site.xml
  - 修改配置：hive.metastore.warehouse.dir




**2.除default库外的其他库的存储位置**

- 其他的库，是在default库的目录下创建目录的。

- 自定义存储位置
  - 在更改之前创建的库，创建的表是按着之前的路径创建。在更改存储位置之后创建的库，创建的表是按着更改后的路径。这里与default表是不一样的。
  - 配置文件：HIVE_HOME/conf/hive-site.xml
  - 修改配置：hive.metastore.warehouse.dir




**3.元数据**

MySQL为数据库，存储在MySQL中。

derby为数据库，在启动当前进程的目录下生成一个metastore_db的库文件。

- 配置文件：HIVE_HOME/conf/hive-site.xml
- 修改配置：javax.jdo.option.ConnectionURL



#### 运行日志

- 默认存放位置：/tmp/atguigu（当前用户）/hive.log


- 自定影存放位置
  - 启用配置文件，/opt/module/hive/conf/hive-log4j.properties.template文件名称为hive-log4j.properties
  - 修改配置：hive.log.dir=/opt/module/hive/logs（自定义路径）




#### 压缩参数

1.中间传输数据压缩功能

`hive.exec.compress.intermediate`

默认为false，需要压缩设置为true



2.map输出压缩功能

`mapreduce.map.output.compress`

默认为false，需要压缩设置为true



3.map输出数据的压缩方式

`mapreduce.map.output.compress.codec`

设置为snappy全包名，`org.apache.hadoop.io.compress.SnappyCodec`



4.最终输出数据压缩功能

`hive.exec.compress.output`

默认为false，需要压缩设置为true



5.MapReduce最终输出数据压缩功能

`mapreduce.output.fileoutputformat.compress`

默认为false，需要压缩设置为true



6.MapReduce最终输出数据压缩为块压缩，输出为SEQUENCEFILE时，才需要设置为BLOCK

`mapreduce.output.fileoutputformat.compress.type=BLOCK`




### Hive配置属性的加载顺序

同名的配置属性从下到上，依次覆盖。4最优先。

配置文件<命令行参数<参数声明。

1. hive依赖于hadoop，hive启动时，默认会读取Hadoop的8个配置文件；
2. 加载hive默认的配置文件hive-default.xml；
3. 加载用户自定义的hive-site.xml；
4. 如果用户在输入hive命令时，指定了--hiveconf，那么用户指定的参数会覆盖之前已经读取的同名的参数。




### Hive元数据的存储配置

#### 启动前的配置

- 提供一个可以从任意机器访问mysql服务实例的用户
- 在$HIVE_HOME/conf/目录下编辑hive-site.xml中的参数，数据库驱动`javax.jdo.option.ConnectionDriverName`，（可选）元数据存储目录`javax.jdo.option.ConnectionURL`
- 最好自己创建metastore数据库，字符集为latin1
- lib中有mysql连接驱动jar包，`mysql-connector-java-5.1.27-bin.jar`。注意这个jar包是通过解压`mysql-connector-java-5.1.27.tar.gz`获得的。
- 如果是mysql5.5，请修改mysql服务端的配置文件/etc/my.cnf,在[mysqld]下添加`binlog_format=ROW` ，然后重启服务器




#### hive-site.xml模版

hive-site.xml需要自己在conf文件包下创建。

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://hadoop103:3306/metastore?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>root</value>
	  <description>username to use against metastore database</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>123</value>
	  <description>password to use against metastore database</description>
	</property>

        <property>
          <name>hive.metastore.warehouse.dir </name>
          <value>/xx</value>
          <description>HDFS file direction adder</description>
        </property>

        <property>
          <name>hive.cli.print.header</name>
          <value>true</value>
          <description>print the names of the columns</description>
        </property>

        <property>
          <name>hive.cli.print.current.db</name>
          <value>true</value>
          <description>print the names of the current database</description>
        </property>

</configuration>

```





#### MySQL配置步骤

##### 1 安装Mysql

上传rpm包，检测当前机器是否已经安装了mysql

```
rpm -qa | grep mysql
rpm -qa | grep MySQL
```

卸载之前安装的残留包

```
sudo rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_6
```

安装服务端

```
sudo rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
```

安装客户端

```
sudo rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
```

如果是5.6的mysql,需要先为root@localhost设置密码：

查看随机生成的密码：

```
sudo cat /root/.mysql_secret
```

启动服务：

```
sudo service mysql start
```

登录后修改密码：

```
mysql -uroot -p刚查询的随机密码
```

修改密码：

```
SET PASSWORD=password('密码')
```

之后退出，使用新密码登录！

##### 2 Mysql的卸载

查询当前安装的mysql版本

```
rpm -qa | grep MySQL
```

停止当前的mysql服务

```
sudo service mysql stop
```

卸载服务端

```
sudo rpm -e MySQL-server-5.6.24-1.el6.x86_64
```

删除之前mysql存放数据的目录

```
sudo rm -rf /var/lib/mysql/
```

##### 3 提供一个可以从任意机器访问mysql服务实例的用户

查询当前有哪些用户：

```
select host,user,password from mysql.user;
```

删除除了localhost的所有用户

```
delete from mysql.user where host <> 'localhost';
```

修改root用户可以从任意机器登录：

```
update mysql.user set host='%' where user='root';
```

刷新权限

```
flush privileges;
```

重启服务：

```
sudo service mysql restart
```

验证本机登录：

```
sudo mysql -uroot -p123456   
```

验证从外部地址登录：

```
sudo mysql -h hadoop103 -uroot -p123456
```

查看当前连接的线程：

```
sudo mysqladmin processlist -uroot -p123456
```



##### 4 复制mysql-connector的jar包

- 解压`mysql-connector-java-5.1.27.tar.gz`获得`mysql-connector-java-5.1.27-bin.jar`。
- 将jar包放在`${HIVE_HOME}/lib/`
- 配置hive-site.xml，`javax.jdo.option.ConnectionURL`配置参数。





##### 5 mysql注意事项

- mysql是开机自启动的，不要开机后重复启动！
- service命令需要使用sudo才可以运行
- 如果要开启mysql的bin_log日志功能，则要修改`binlog_format=row;`







### Hive启动问题集

- 在配置了Hadoop-HA集群上启动，会出现RuntimeException，由StandbyException抛出


**原因：**可能与Hadoop-HA的设置相关。导致Hive无法在状态为standby的nameNode服务器上启动。

![Snipaste_2020-03-04_12-09-04](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-04_12-09-04.png)



- SAXParseException


**原因：**xml配置文件的XML信息必须在第一行。

![Snipaste_2020-03-04_14-21-36](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-04_14-21-36.png)







mysql

root密码：123





## --使用与操作--

## 通过什么方式使用Hive？

### Hive shell（Linux中操作）

#### 启动前提

hive基于hadoop，在启动hive之前，必须先启动hadoop(hdfs,yarn)



#### 查看hive中的变量

- `set`
  - 查看hive启动后加载的所有变量
- `set 属性名`
  - 查看某个指定参数的值
- `set 属性名=属性值`
  - 对加载的参数进行修改，此次修改只有当前的cli有效，一旦退出就需要重新设置



#### hive 交互命令

	hive
	-d key=value:  定义一个变量名=变量值
	--database 库名： 让hive初始化连接指定的库
	-e <quoted-query-string> ： hive读取命令行的sql语句执行，结束后退出cli
	-f sql文件：   hive读取文件中的sql语句执行，结束后退出cli
	--hivevar <key=value> : 等价于-d
	--hiveconf <property=value>: 在启动hive时，定义hive中的某个属性设置为指定的值，多个属性值设置，可以用逗号分隔
	 -i <filename> ： 在启动hive后，先初始化地执行指定文件中的sql，不退出cli
	 -S,--silent :  静默模式，不输出和结果无关的信息


#### 使用CLI中的其他命令

CLI（command-line interface命令行界面）

- 访问hdfs，`dfs`
- 运行shell中的命令（如jps）,`!命令`




### JDBC方式

#### 启动前提（存疑）

启动Hive的hiveserver2服务，作为支持JDBC连接的服务器。

- 启动hiveserver2的方式
  - 前台启动，`hiveserver2`
  - 后台运行，`hiverserver2 &`
- 前台启动之会显示会卡住，没有下一行提示。实际上已经在运行接受请求。在接受到其他的JDBC客户端的请求时，会打印OK。



#### Beeline（存疑）

- 启动Beeline
  - Linux中，`HIVE_HOME/bin/beeline`
- 创建连接
  - `!connect 'jdbc:hive2://hadoop103:10000'`
  - 回车后输入用户名atguigu，密码随意（为什么密码是随意的？）



#### Java程序

- 创建Maven工程，引入hive-jdbc的驱动

```xml
 <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-jdbc</artifactId>
            <version>1.2.1</version>
 </dependency>
```

hive还依赖于hadoop，还需要将hadoop的依赖也加入进去！



- 示例程序：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

/**
 * Created by VULCAN on 2020/3/4
 */
public class HiveJdbcTest {

    public static void main(String[] args) throws Exception {

        //注册驱动  可选，只要导入hive-jdbc.jar，会自动注册驱动
        //Class.forName("org.apache.hive.jdbc.HiveDriver");

        //创建连接  url,driverClass,username,password
        Connection connection = DriverManager.getConnection("jdbc:hive2://hadoop103:10000", "atguigu", "");

        //准备sql
        String sql="select * from person";

        PreparedStatement ps = connection.prepareStatement(sql);

        //执行查询
        ResultSet resultSet = ps.executeQuery();

        //遍历结果
        while (resultSet.next()){

            System.out.println("name:"+resultSet.getString("name")+"   age:"+
                    resultSet.getInt("age"));
        }
        //关闭资源
        resultSet.close();
        ps.close();
        connection.close();
    }
}

```

####数据库集成工具（待补充）

**1.dbeaver**

数据库工具，开源，支持多种数据库。



**2.IDEA-Database**

url格式为：jdbc:hive2://ip地址:port端口号/database数据库





## 怎样使用Hive命令？（待实现）

### 常见的场景需求（待完善）

- wordCount案例

1. 将文件先上传到hdfs
2. 需要对数据进行ETL，转为结构化的数据
3. 根据数据进行建表
4. 将数据加载到表中，或和表产生关联
5. 编写HQL
6. 客户端返回HQL运行的结果



- 从其他数据格式，如Json，转换到Hive的数据存储格式，根据表的需求添加对应的分隔符。





- 按表的元数据指定的数据类型，用分隔符来读取HDFS上的数据。


```hive
create table t1(name string,friends array<string>,children map<string,int>,
              address struct<street:string,city:string> )
```



- HDFS上的数据与表结构产生关联，需要将HDFS上的数据文件放在表在HDFS上的存储目录中。
  - hadoop手动上传
    - `hadoop dfs -put input output`
  - Hive中使用load上传
    - `load data [local] inpath '路径名' into table 表名 `

[local]是什么作用（待完善）

路径名最好用全路径名，用家目录`~`会有问题。



查看表信息

describe extended 表名；

可展示如分隔符信息，字段信息





## ------预习

分区，paritioned，将数据根据一定规则（如每日数据）分开来存储

分桶，clustered，针对数据文件，进行抽样。这个不太理解？（疑问）

学习hive，一定要有元数据和原数据的概念，要区分开。**hive是用元数据找到原数据的。**



**分区表的架构问题**

**问题来源：**一开始，业务需求只建立（小时，分钟）的2级分区，但是后面需求要增加到（天，小时，分钟）的3级分区。扩展分区表级数，只能通过重新建立表结构解决。
重新建立表结构的话，就会出现一个问题，原来的数据在HDFS上存储的路径格式已经固定了如（/data/hour/min），现在要改成（/data/day/hour/min），那么就只能数据迁移。

**经验：**对于这类架构问题，要很小心。如果HDFS的路径没有断层，也就是说一开始HDFS上存储的路径格式就是（/data/day/hour/min），那么重新创建表结构，设置location，就可以实现分区表的级别扩展，不用进行数据迁移。



## DDL库操作

**1.增**

```sql
create database [if not exists] 库名 
[comment '库的注释']
[location '库在hdfs上存放的路径']
[with dbproperties(属性名=属性值，...)]
```

**注意：** 

- location可以省略，默认存放在/user/hive/warehouse/库名.db目录下
- dbproperties中只能存放string类型的属性
- 设置库的字符集为latin1，metastore库最好手动创建。`DEFAULT CHARACTER SET latin`



**2.删**

```sql
drop database [if exists] 库名 [cascade]
```

删除库时，是两步操作：

1. 在mysql的DBS表中删除库的元数据
2. 删除hdfs上库存放的路径

**注意：**

- 以上操作只能删除空库(库中没有表)！如果库中有表，是无法删除的，如果要强制删除，需要添加**cascade**关键字

强制删除，元数据会被删掉。那么元数据呢？强制删除相当于直接把表也都被全部删除了吗？（疑问）



**3.改**

Hive的1.X版本，只能改location和dbproperties属性！

```sql
ALTER DATABASE 库名 SET DBPROPERTIES (property_name=property_value, ...);
```

- 在改库的属性时，同名的属性会覆盖，不存在的属性会新增！

和hive的版本有关系。

1.X只能改，location和dbproperties属性。



**4.查**

切换库

```sql
use 库名
```

查看库的描述

```sql
desc database 库名
```

查看库的详细描述：

```sql
desc database extended  mydb2
```

查看库中的表

```sql
show tables in 库名
```

查看当前库下的表

```sql
show tables
```





## DDL表操作

### **1.外部表和管理表（内部表）**

**1.1定义区别：**

```sql
CREATE TABLE [EXTERNAL] 表名(); --简化，方便下面的说明
```

- 外部表，hive是不负责数据生命周期管理的、
  - 创建表时，加EXTERNAL，CREATE EXTERNAL 表名
  - 删除了hive中的外部表，只会删除表的schame信息，不会删除hdfs中的数据。
- 管理表，hive可以管理数据的生命周期
  - 创建表时，不加EXTERNAL，CREATE EXTERNAL 表名
  - 删除了hive中的管理表，不仅会删除表的schame信息，还会删除hdfs中的数据。

**生产情况：**

通常情况下，在公司创建的都是外部表。

因为表是廉价的，数据是珍贵的。



**1.2管理表和外部表的转换**

**转换操作**

MANAGED_TABLE--->EXTERNAL_TABLE

```sql
alter table 表名 set TBLPROPERTIES('EXTERNAL'='TRUE')
```

EXTERNAL_TABLE--->MANAGED_TABLE

```sql
alter table 表名 set TBLPROPERTIES('EXTERNAL'='FALSE')
```

可以通过查看表的Table Type属性，来判断表的类型。

注意：易错！hive的属性名和属性值，严格区分大小写。其他的不区分大小写。



### 2.分区表

**2.1定义**

分区表将表的数据，按照一个指定的分区字段，分散到表目录的**不同子目录**中。

**目的：**分散数据，在查询时，可以根据分区字段，过滤数据，**减少查询时MR输入的数据量**。

分区的元数据存放位置：在metastore.PARTITIONS表中。



**2.2注意**

- 新数据只能加在最后一级分区目录下；
- 分区目录的名称必须是**分区字段名=分区字段值**
- 2个注意：新数据只能加在**最后一级分区目录下**；
- 普通列字段的基础上自动添加分区字段。分区字段并不存储在分区表中，是在做表的字段查询时自动添加的。
- 分区字段和普通列字段，是不同的。






**2.3分区表的创建**

**2.3.1单级分区表**

```sql
create table t2(id int,name string,sex string) 
partitioned by(province string);
```

`partitioned by(part_col dataType)`

- part_col，是分区字段名，**不能与表中的字段相同**；
- dataType，是分区字段数字类型；




**2.3.2多级分区表**

表是一个分区表，但是有多个分区字段。

```sql
create table t3(id int,name string,sex string) partitioned by(province string,city string,area string)
```



**2.4分区表的数据导入**

多级分区表，要注意新增数据时，要写到最后一级分区字段。

**2.4.1 put方式，HDFS**

put方式无法生成分区表的元数据。

**解决方法：**

- 手动创建分区

  - 手动创建分区，不仅可以生成分区目录，还会生成分区的元数据

  ```sql
  alter table 表名 add partition(分区字段名=分区字段值)
  ```

  - 查看表的分区元数据

  ```sql
  show partitions 表名
  ```

  ​

- 自动修复分区命令msck。

  ```sql
  msck repair table 表名
  ```




**2.4.2 load方式**

可以做到

- 帮我们将数据上传到分区目录；
- 自动生成分区的元数据

```sql
load data local inpath '/home/atguigu/hivedatas/t2.data' into table t2 partition(province='guangxi')
```





**2.5删除分区**

```sql
alter table 表名 drop patition(),patition()
```

删除分区一定会删除分区的元数据，如果表是管理表，还会删除分区目录。



**分区字段的间隔**

增加分区，多个分区用空格` `间隔

删除分区，多个分区用逗号`,`分隔



### **3.分桶表**

**3.1定义**

分桶表也是为了分散数据，将分桶字段相同的数据分散到**同一个文件中**。

目的：分散数据，分散数据是为了更好地抽样调查。

- 分桶和MR中的分区是一个概念。
  - 要分桶，就必须要跑MR里的分区，只有insert才能触发，需要中间表。
  - 指在向表中使用insert 语句导入数据时， insert语句会翻译为一个MR程序，MR程序在运行时，可以根据分桶的字段，对数据进行分区。
- 同一种类型的数据，就可以分散到同一个文件中。
  - 可以对文件根据类型进行抽样查询。
- load和insert的区别
  - load，相当于put，只是上传数据
  - insert，相当于mr，对数据做了分区



**3.2注意**

- 如果需要实现分桶，那么必须使用Insert的方式向表中导入数据。只有insert会运行MR。
- 分桶的字段是基于表中的已有字段进行分桶。所以在clustered by中，只用写col_name，而不用像分区一样定义字段类型。
- 如果要实现分桶操作，那么reduceTask的个数需要>1。
- 分桶时，可以指定按照某个字段进行排序。



**3.3分桶表的创建**

**前提**

导入数据之前，需要打开强制分桶的开关：

```
set hive.enforce.bucketing=true;
```

**创建分桶表**

```sql
create table stu_buck(id int, name string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';
```

创建临时表

```sql
create table stu_buck_tmp(id int, name string)
row format delimited fields terminated by '\t';
```

先把数据load到临时表

```sql
oad data local inpath '/home/atguigu/hivedatas/t4.data' into table stu_buck_tmp;
```

使用insert 语句向分桶表导入数据

​				需要让reduceTask的个数=分的桶数，但是此时不需要额外设置！默认reduceTask的个数为-1，-1代表由hive自动根据情况设置reduceTask的数量！

```
mapreduce.job.reduces=-1
```

​		导入数据

```sql
insert overwrite table  stu_buck select * from  stu_buck_tmp
```





**3.4排序**

**前提**

如果需要执行排序，提前打开强制排序开关

```
set hive.enforce.sorting=true;

```

通过指定sorted by来指定分桶后的数据按照什么字段进行排序！

①创建分桶表，指定按照id进行降序排序

```sql
create table stu_buck2(id int, name string)
clustered by(id) 
SORTED BY (id desc)
into 4 buckets
row format delimited fields terminated by '\t';
```

②向表中导入数据

​				导入数据

```sql
insert overwrite table  stu_buck2 select * from  stu_buck_tmp
```

#### 



**3.5抽样查询**

基于分桶表进行抽样查询，表必须是分桶表！

```sql
select * from 分桶表 tablesample(bucket x out of y on 分桶字段);
```

假设当前分桶表，一共分了z桶！

- x: 代表从当前的第几桶开始抽样

- 0<x<=y

- y:  z/y 代表一共抽多少桶！

- y必须是z的因子或倍数！


怎么抽：  从第x桶开始抽，当y<=z每间隔y桶抽一桶，直到抽满 z/y桶。

y>z时，就无法完整的抽取一个桶的数据，很混乱，要避免。



### 4.表的通用操作

**4.1创建**

3种创建表的方式

4.1.1常规create

```sql
create [external] table [if not exists] 表名(col_name col_type comment,...)
[comment 表的注释]
[partitioned by (分区字段 字段数据类型, ...)]
[clustered by 分桶字段名]
[sorted by 排序字段名 (ASC|DESC)
into x buckets]
[row format ... 自定义分隔符]
[stored as 文件格式]
[location '表在hdfs存储的位置']
[tblproperties ('属性名'='属性值')]

```

4.1.2 基于现有表的创建

4.1.2.1like基于源表

基于源表，复制其表结构，创建新表，表中无数据

```sql
create table 表名 like 源表名
```

注意：只会复制表结构，不会复制表中属性值（如表的分区，存储格式等）；

不能通过此方法创建分区表。



4.1.2.2 as基于一条查询语句

基于一条查询语句，根据查询语句中字段的名称，类型和顺序创建新表,表中有数据

```sql
create table 表名 as ‘select语句’
```

注意：



4.1.3 as基于



**4.2删除**

```sql
drop table [if exists] 表名
```

清空表中的数据（表必须是管理表）

```sql
truncate table 表名
```



**4.3查询**

查看表的描述

```sql
desc  表名
```

查看表的详细描述

```sql
desc extended 表名
```

格式化表的详细描述

```sql
desc formatted 表名
```

查看表的建表语句

```sql
show create table 表名
```



**4.4修改**

修改表的某个属性

```sql
alter table 表名 set TBLPROPERTIES('属性名'='属性值')
```

修改列的信息

```sql
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
```

重命名表

```sql
ALTER TABLE table_name RENAME TO new_table_name
```

重置表的所有列

```sql
ALTER TABLE table_name REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
```

添加列

```sql
ALTER TABLE table_name ADD COLUMNS (col_name data_type [COMMENT col_comment], ...) 
```



### 5.视图view

**5.1视图的概念**

1. 视图是一种特殊(逻辑上存在，实际不存在)的表
2. 视图是只读的
3. 视图可以将敏感的字段进行保护，只将用户需要的字段暴露在视图中，保护数据的隐私

**5.2常用操作**

与表的DDL操作类似

创建： create view 视图名 as select 语句

删除：drop view 视图名

查询：desc view 视图名



int类型的hash值是它值的本身。



ISO/ICE 27001

ISO/ICE 27018

笔记中的 | ，是或者的意思。应该要加（）把选择括起来。老师的笔记要修改（补充）

## DML操作

### 导入

**1.load**

```sql
load data [local] inpath '数据路径' into table 表名 [partition]
```

local命令不走MR。

带local:从本地将数据put到hdfs上的表目录！

不带local: 代表将hdfs上的数据，mv到hdfs上的表的目录！

**2.insert**

insert导入数据会运行MR程序。

在特殊的场景下，只能使用insert不能用load!

例如：  

①分桶

②希望向hive表中导入的数据以SequnceFile或ORC等其他格式存储！

语法：

```sql
insert (into | overwrite)  table 表名 [partition()] (values(),(),() | select 语句)
```

insert into: 向表中追加写

insert overwrite: 覆盖写，清空表目录(hdfs层面，和外部表无关)，再向表中导入数据



多插入模式：从一张源表查询，执行多条insert语句，插入到多个目的表

```sql
from 源表
insert xxxx 目标表1 select xxxx
insert xxxx 目标表2 select xxxx
insert xxxx 目标表3 select xxxx
```

示例：

```sql
from t3
insert overwrite table t31 partition(province='henan',city='mianchi',area='chengguanzhen') select id,name,sex 
where province='guangdong' and city='shenzhen' and area='baoan'
insert overwrite table t32 partition(province='hebei',city='mianchi',area='chengguanzhen') select id,name,sex 
where province='guangxi' and city='liuzhou' and area='buzhidao'
insert overwrite table t33 partition(province='hexi',city='mianchi',area='chengguanzhen') select id,name,sex 
where province='guangxi' and city='nanning' and area='buzhidao'
```

**3.location**

建表时可以指定表的location属性（表在hdfs上的目录）。适用于数据已经存在在hdfs上了，只需要指定表的目录和数据存放的目录关联即可！

**4.import**

import必须导入的数据是由export命令导出的数据！

```sql
IMPORT [[EXTERNAL] TABLE new_or_original_tablename [PARTITION (part_column="value"[, ...])]]
  FROM 'source_path'
  [LOCATION 'import_target_path']
```

要求：  ①如果要导入的表不存在，那么hive会根据export表的元数据生成目标表，再导入数据和元数据

②如果表已经存在，在导入之前会进行schame的匹配检查，检查不复合要求，则无法导入！

③如果目标表已经存在，且schame和要导入的表结构匹配，那么要求要导入的分区必须不能存在！



location，在HDFS分区表的路径结构中，已经在create表时partitioned by了，为什么还要mscn repair？

import的问题还挺多的，和export配合使用，用于在不同的hive之间进行数据转移。



### 导出

**1.insert**

命令：

```
insert overwrite local directory '/home/atguigu/abc'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
select *, rank() over(partition by course order by score) from score;

```



```sql
insert overwrite [local] directory '导出的路径'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'  
select 语句;
```

带local导出到本地的文件系统，不带local代表导出到hdfs。

注意：输出的自定义分隔符的位置，写在insert语句后，select语句前。



**2.export导出**

命令：

```sql
EXPORT TABLE tablename [PARTITION (part_column="value"[, ...])]
  TO 'export_target_path' [ FOR replication('eventid') ]
```

优势： 

- 既导出数据还导出metastore(元数据，表结构)
- 导出的数据和表结构可以移动到其他的hadoop集群或hive中，使用import导入！



**3.`hive -e 'sql' > 文件`**

hadoop命令导出，是指的将分桶的数据，直接用HDFS命令传输文件到本地上。没什么用。



**4.总结**

insert和export的区别

- export不仅导出数据，还导出metastore元数据和表结构。
- export的数据可以移动到其他hadoop或hive集群中。





## DQL查询操作

记录与SQL有不同的地方，以及一些在SQL中的易错点

### 排序

hive独有的排序

**1. cluster by**

不支持降序，只支持升序

是一种简化，如果sort和distribute字段一致，并且按照asc进行排序，可以用cluster简写。

distribute by city sort by city 相当于 cluster by city



**2. distribute by**

使用某个字段进行分区。

需要指定reducetask个数，不能设置为-1自动设置吗？（疑问）不能，会报错的。sort by能设置为-1是因为。。。reducetask数量大于1时，Hive不知道要设置几个。（存疑，看视频的最后1分钟）

但是分区不能排序，只能用`hash值 % 分区数`



**3.sort by**

可以按指定字段排序，但是分区无法控制，随机分区。所以需要与distribute by一起使用。



### where语句

A<=>B

用法：A和B都是NULL，返回TRUE

A RILKE B

相当于A REGEXP B

用法：A与B匹配，匹配则返回TRUE，B一般是正则表达式。



### 分组

- where，在group by之前执行
- having，在group by之后执行
- group by语句中，select后面
  - 可以写group by的字段和聚集函数中的字段
  - 不能直接写其他字段。



### Join

Hive，只支持等值连接，不支持（<小于 >大于）的链接

连接谓词，不支持or



### 函数

#### **1.函数分类**



#### **2.函数查看**

用户自定义的函数是以库为单位的。

创建函数，必须在要使用的库进行创建。其他的库需要用`库名.函数名`来调用



#### **3.常用函数**

##### **3.1NVL**

**定义：**

```mysql
nvl(value,default_value) - Returns default value if value is null else returns value
当value是null值时，返回default_value,否则返回value
```

**使用场景：**计算前，处理nall值。

求有奖金人的平均奖金： avg聚集函数默认忽略Null

```sql
select avg(comm) from emp;
```

求所有人的平均奖金： 提前处理null值！

```sql
select avg(nvl(comm,0)) from emp;
```



在2个参数情况下，参数顺序是有要求的。（存疑）



##### **3.2行转列**

**3.2.1拼接单行字符**

对某字段进行字符拼接，不汇总。

**3.2.1.1CONCAT**

**定义：**

```mysql
concat(str1, str2, ... strN) - returns the concatenation of str1, str2, ... strN or concat(bin1, bin2, ... binN) - returns the concatenation of bytes in binary data  bin1, bin2, ... binN
Returns NULL if any argument is NULL
```

**用法：**拼接字符，字符中有NULL，则返回NULL



**3.2.1.2CONCAT_WS**

**定义：**

```mysql
concat_ws(separator, [string | array(string)]+) - returns the concatenation of the strings separated by the separator.
```

**用法：**拼接字符，用指定符号separator做分隔符拼接，不受NULL值影响。



**3.2.2拼接多行字符**

COLLECT系列，对某字段进行汇总，多行转一行。

**3.2.2.1COLLECT_SET**

返回的数据组成set集合，去重

**3.2.2.2COLLECT_LIST**

返回的数据组成list集合，不去重

**3.2.2.3与Distinct的区别**

distinct还是以多行形式呈现，而不是汇总一列的数据呈现为一行。



##### **3.3列转行**

**3.3.1定义**

1列1行 转为 N列N行

**3.3.2explode**

侧写格式，explode为例

```mysq
select 临时列名，其他字段
from 表名
-- 将 UDTF函数执行的返回的结果集临时用 临时表名代替，结果集返回的每一列，按照顺序分别以临时--列名代替
lateral view UDTF() 临时表名 as 临时列名,...
```

explode只能单独与select使用，无法与其他字段一起使用。可以用侧写解决这个问题。



**3.3.3LATERAL VIEW侧写**

炸裂的临时结果集中的每一行，可以和炸裂之前的所在行的其他字段进行join!

**解决的问题：**

1. explode只能单独与select使用，无法与其他字段一起使用。

2. explode后，explode出的表格和之前的表格没有关联字段，无法用on来使用join。

   使用侧写，那么explode出的表格实际上和之前的表格在id对应上是一致的。也就是说，从一行explode出的N行，都会被标记为一行，然后之前的一行与这N行一一做笛卡尔积。




##### **3.4判断句式**

**3.4.1 if**

定义：

```sql
if('条件判断','为true时','为false时')
```

类似三元运算符，做单层判断。



**3.4.2case when**

定义：

```mysql
case 列名
	when 值1 then  值2
	when 值3 then  值4
	when 值5 then  值6
	...
	else 值7
end
```

类似swith-case，做多层判断。



##### 3.5窗口函数

**3.5.1定义**

**窗口函数**=函数+窗口，指以下特定函数在运算时，可以自定义一个窗口（计算的范围）

**窗口：**函数在运行时，计算的结果集范围。

**函数：** 要运行的函数，只有以下函数称为窗口函数。

- 开窗函数：
  - LEAD
    - 用来返回当前行以下行的数据。
    - **用法：** LEAD (列名 ,offset ,default)
    - offset是偏移量，默认为1
    - default： 取不到值就使用默认值代替
  - LAG
    - 用来返回当前行以上行的数据
    - **用法：** LAG (列名 ,offset,default)
    - offset是偏移量，默认为1
    - default： 取不到值就使用默认值代替
  - FIRST_VALUE
    - 返回指定列的第一个值
    - **用法：** FIRST_VALUE(列名，[false是否忽略null值])
  - LAST_VALUE
    - 返回指定列的最后一个值
    - **用法：** LAST_VALUE(列名，[false是否忽略null值])
- 标准的聚集函数：MAX,MIN,AVG,COUNT,SUM
- 分析排名函数：
  - RANK
  - ROW_NUMBER
  - DENSE_RANK
  - CUME_DIST
  - PERCENT_RANK
  - NTILE


**3.5.2语法**

窗口函数，写在select这一行语句中的。

```mysql
函数  over( partition by 字段1,字段2 [window clause] )
```

partition by : 根据某些字段对整个数据集进行分区！

order by: 对分区或整个数据集中的数据按照某个字段进行排序！

注意： 如果对数据集进行了分区，那么窗口的范围不能超过分区的范围，即窗口必须在区内指定。



**3.5.3window clause窗口子句**

```mysql
--总的语法：
(ROWS | RANGE) BETWEEN ... AND ...
--3个范围：
(ROWS | RANGE) BETWEEN (UNBOUNDED | [num]) PRECEDING AND ([num] PRECEDING | CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
(ROWS | RANGE) BETWEEN CURRENT ROW AND (CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
(ROWS | RANGE) BETWEEN [num] FOLLOWING AND (UNBOUNDED | [num]) FOLLOWING
```

指定行或者列，窗口的前边界和后边界

UNBOUNDED，无边界

PRECEDING，前边界

FOLLOWING，后边界



window子句的特例

特例：1.未指定前边界 和 后边界，则默认为UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING，上无边界到下无边界	。

2.指定了order by，但没有指定window子句，则默认为UNBOUNDED PRECEDING AND CURRENT，上无边界到当前行。

3.`排名函数、LAG和LEAD窗口函数`，不支持在over中定义window子句。

若在窗口函数中，进行了分区partition by，则窗口是在每个分区中进行。



**3.5.4如何使用窗口函数？**

1. 根据需求使用什么函数
2. 确定窗口的大小



##### 总结

窗口函数 与 聚集函数的区别

- 聚集函数
  - 对整个表进行计算，N行变为1行。
  - 统计结果，不需要明晰，只需要一个统计的结果。
- 窗口函数
  - 对窗口中的每一条记录都进行计算，N行变为M行。
  - 统计明细，为每条明细都附加一个总的结果。即返回的结果是每一行数据和一个计算后的值。




##### 3.6排序函数

- RANK()：允许并列，并列后跳号
- ROW_NUMBER(): 连续，不并列，不跳号
- DENSE_RANK(): 连续，允许并列，并列不跳号！
- CUME_DIST(): 当前值以上的所有的值，占总数据集的比例！
- PERCENT_RANK():  rank()-1/总数据集-1
- NTILE(x): 将窗口中的数据平均分配到x个组中，返回当前数据的组号
  - 若分组有余，则数据从前往后插入组中。如12行数据，5个组，则前2个组多插入1行数据，即33222。

注意：排名函数可以跟over()，但是不能在over()中定义window_clause

排序的标准：按窗口的顺序排名的，需要用order by，这时rank排名就是行号。而不使用order by时，rank排名会出问题。



##### 3.7日期函数

默认解析的日期格式必须为：`2019-11-24 08:09:10`

日期计算函数

```mysql
unix_timestamp:返回当前或指定时间的时间戳	
from_unixtime：将时间戳转为日期格式
current_date：当前日期
current_timestamp：当前的日期加时间
*to_date：抽取日期部分，这个可以把'2017-1-7'这样的不是严格标准的日期，变成'2017-01-07'的严格标准日期，数据类型为string
year：获取年
month：获取月
day：获取日
hour：获取时
minute：获取分
second：获取秒
weekofyear：当前时间是一年中的第几周
dayofmonth：当前时间是一个月中的第几天
* months_between： 两个日期间的月份，前-后
* add_months：日期加减月
* datediff：两个日期相差的天数，前-后
* date_add：日期加天数
* date_sub：日期减天数
* last_day：日期的当月的最后一天
```

日期取整函数

```mysql
round： 四舍五入
ceil：  向上取整
floor： 向下取整
```





##### 3.8字符串操作函数

```mys
upper： 转大写
lower： 转小写
length： 长度
* trim：  前后去空格
lpad： 向左补齐，到指定长度
rpad：  向右补齐，到指定长度
* regexp_replace： SELECT regexp_replace('100-200', '(\d+)', 'num')='num-num
--这里的正则表达式 \d+，可能有问题，无法正确运行。
	使用正则表达式匹配目标字符串，匹配成功后替换！
```





##### 3.9集合操作函数

```mysql
size： 集合（map和list）中元素的个数
map_keys： 返回map中的key
map_values: 返回map中的value
* array_contains: 判断array中是否包含某个元素
sort_array： 将array中的元素排序
```



## 压缩

### Snappy压缩

**前提：**Hive使用snappy压缩，需要Hadoop支持snappy。

#### hadoop安装snappy

1.查看当前集群是否支持snappy

```shell
hadoop checknative
```

2.将snappy和hadoop编译后的so文件，放到`HADOOP_HOME/lib/native`目录下。

3.在集群中分发该so文件，不要忘记分发！

注意：此处省略了snappy的编译过程。具体知识，与native lib本地库有关。



#### hive启用压缩

启用要求：在Hadoop的mapred-site.xml或者hive中，配置好压缩参数。

具体参数的修改：参考本记录中的<Hive常用配置属性-压缩参数>

使用压缩的阶段：

shuffle阶段使用压缩，修改intermediate，和map.output.compress的2个参数，共3个参数。

reduce阶段使用压缩，修改output，fileoutputformat.compress的2个参数，共3个参数。



## 存储

### 表的存储类型

![Snipaste_2020-03-09_19-40-10](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-09_19-40-10.png)

- 行式存储
  - 特点：查询的速度更快
  - 存储格式有：TEXTFILE、SEQUENCEFILE
- 列式存储
  - 特点：可以针对性的设计更好的设计压缩算法.以列为单位进行压缩和编码
  - 存储格式有：ORC、PARQUET

在大数据领域，所有的数据格式，都采用**列式存储**。



### 存储文件格式

#### 格式类型（Parquet待补充）

**1.TextFile**

默认格式，数据不做压缩



**2.ORC**![Snipaste_2020-03-09_20-24-21](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-09_20-24-21.png)

- 使用范围：Hive独有，只能在Hive中使用。
- 原理：文件由1个或多个stripe组成。每个stripe有3部分组成
  - Index Data：一个轻量级的index，默认是每隔1W行做一个索引。这里做的索引应该只是记录某行的各字段在Row Data中的offset。
  - Row Data：存的是具体的数据，先取部分行，然后对这些行按列进行存储。对每个列进行了编码，分成多个Stream来存储。
  - Stripe Footer：存的是各个Stream的类型，长度等信息。

**3.Parquet（待补充，这个似乎很有用？）**

![Snipaste_2020-03-09_20-25-02](E:\itHeiMa\资料\课程截图类\Snipaste_2020-03-09_20-25-02.png)

- 使用范围：整个Hadoop体系。 clodera公司退出的一个旨在整个Hadoop体系中通用的一个高效的数据存储格式。
- 原理：二进制方式存储，不可直接读取，自解析。



#### 比较和总结

**1.ORC和Parquet都是行列结合存储。**

将部分行称为一个行组，每个行组内部使用列式存储。

**2.加载数据的方式**

- TextFile，可以使用load和insert
- ORC、Parquet，只能使用insert

**3.性能**

压缩比：ORC>Parquet>TextFile

查询速度：无明显差别

**4.hive常用的压缩和存储组合**

Hive独有：使用ORC文件格式，内部使用Snappy压缩。

通用：PARQUET

**5.是否支持切片**

ORC和PARQUET都支持切片。

ORC(snappy)，支持切片

PARQUET不使用压缩，支持切片。如果使用LZO压缩，此时是否切片取决于为



#### 配置文件格式

```hive
create table
...
store as 文件格式（默认为TEXTFILE）

```






​    

flume，抽取数据，传出数据。数据量太大，可以用多层flume。

kafka，数据传输，简单清理数据

Hadoop，存储，MR离线计算

Spark，离线计算core，sql，mlib，r，在线计算streaming

软件平台开发技术，spring，activeMQ，mysql，echar



会衍生出数据治理岗位，政府已经有相应的部门了。



## 优化

hive.mapred.mode，可以设置strict/nonstrict，严格模式。生产环境中，肯定是要开启的。





关键字key，值为null，则成为空key。

空key解决方法：ETL阶段，对null进行处理

查询阶段，null数据进行过滤，不要null数据；

null数据转换，将数据分散到不同reduce中，但是有前提，不能影响查询结果



group by聚合函数的优化

hive.groupby.skewindata，会将job，从1个变成2个。第1个job随机分区，先进行1次聚合，第1次聚合的结果文件会大大减少数据量，并且负载均衡（因为随机分区）。第2个job再进行聚合。





动态分区，很实用。

### 1.fetch抓取

一些HQL语句，可以不翻译为MR程序，而是使用FetchTask来运行，拉取数据！

启用了fetch抓取，可以节省某些HQL语句的查询效率！

默认fetch抓取的设置是开启的，为more

```
hive.fetch.task.conversion=more
```

一般不需要设置！



### 2.表的Join

#### 2.1 小表与大表join

在hive中，不管是 大表 join 小表还是 小表 Join 大表，hive都可以自动优化。（疑问）怎么优化的呢？

#### 2.2 大表之间的join

在MR中ReduceJoin的原理（回顾）：

```
Map阶段
①ReduceTask可以启动任意数量，必须保证关联字段相同的分到同一个区
		关联字段相同的数据，才能分到同一个ReduceTask
②数据源有两种不同类型的文件，都使用同一个map处理逻辑！
	因此在map()需要根据切片的来源，进行判断，从而进行不同的封装逻辑的选择！
③Mapper中封装数据的bean，应该可以覆盖不同类型文件中的所有字段
④需要在bean中打标记，标记bean封装的是哪个文件中的数据
		只需要将order.txt中的数据进行替换后输出！
Reduce阶段
⑤在reduce()中，根据数据的标记进行分类
		分为order.txt中的数据和pd.txt中的数据
⑥在cleanup()中，只讲order.txt中的数据，替换后写出
```

**问题原因：**大表之间的join，若关联字段，有大量的null数据。若不对null数据进行过滤，则null数据会被分到同一个区，造成数据倾斜，处理null数据的reduce就会负担很重。

**问题举例：**

左外连接，如果左表A表中有大量的c字段的值为null的数据，如果不对null的数据进行过滤，此时会产生数据倾斜！

```
A left join B on A.c=B.c
```

大量的为null的字段的数据，分配到同一个区，这个区最终是会被一个Reducer处理，此时这个Reducer的负载较大！

空key是指，关键字key，值为null。

**2.2.1 空Key过滤**（查询阶段）

如果关联字段为null的数据，不需要，可以对空Key进行过滤：

```sql
select n.* from (select * from nullidtable where id is not null ) n  left join ori o on n.id = o.id;
```

**2.2.2 空key分区平衡**（查询阶段）

如果为null的字段不能过滤！此时需要对nullkey进行转换！

**目的：** 转换后null的key可以随机地分配到多个不同的区中，负载均衡

**原则：** 转换后，不能影响SQL原本查询的结果！

**注意：** 注意转换后的类型必须能否正确赋值给原先的数据类型！

```sql
insert overwrite local directory '/home/atguigu/join3' 
select n.* from nullidtable n full join ori o on 
case when n.id is null then -floor(rand()*100) else n.id end = o.id;
```

**2.2.3 空key处理**（ETL阶段）

在ETL阶段，对null值进行处理。



#### 2.3 MapJoin和ReduceJoin（待补充）

小表 join 大表，用MapJoin

大表 join 大表，用ReduceJoin

判断标准（待补充）：





### 3.Group By优化（有疑问）

**作用：**解决group by只由1个job处理，可能的分区字段数据不平衡，造成reduceTask负载不均衡问题。

**原因：**不开启优化，则group by只用1个job来执行。此时MR的分区来实现group by操作。若group by的字段数据不均衡，则落入某1个分区的数据就会很多，那么处理这个分区的reduceTask压力就很大

**优化方式：**开启负载均衡配置

```
hive.groupby.skewindata = true
```

hive.map.aggr，Map端是否进行聚合。为什么会优化？（疑问）

**优化原理：**在Group by时，1个Job会变为两个Job。

第1个Job负责随机分区，reduce按detpno进行分组，进行了第1次集合；

第2个Job，将第1个Job输出的数据，再按照deptno进行分区，进行第2次聚合。



**举例：**

| deptno | name  |
| ------ | ----- |
| 10     | jack  |
| 10     | tom   |
| 20     | marry |
| 20     | nick  |
| 10     | jerry |

第一个Job负责随机分区，按detpno进行分组，先进行聚合！

ReduceTask1:

| deptno | name  |
| ------ | ----- |
| 10     | jack  |
| 10     | tom   |
| 20     | marry |

先进行聚合，以deptno作为分组(MR中的分组)的条件！

输出：(10,2),(20,1)

ReduceTask2:

| deptno | name  |
| ------ | ----- |
| 20     | nick  |
| 10     | jerry |

输出：(10,1),(20,1)

第二个Job再将第一个Job输出的数据，再按照deptno进行分区，进行最终的聚合！

ReduceTask1:（10，2）（10，1）----> (10,3)

ReduceTask2:（20，1）（20，1）----> (20,3)



### 4.动态分区（待补充）

当向一个分区表插入数据时，必须指定要插入数据的分区信息。

静态分区：insert into table 表名  partition(分区字段名=分区字段值)，此种方式称为静态分区。此时向哪个分区插入数据已知。

动态分区：在插入时，分区字段值根据要插入数据的某个字段动态生成。



操作步骤：（待补充）



示例：

创建一个和员工表类似的分区表，这个表以Job为分区字段

```sql
create external table if not exists default.emp_partition(
empno int,
ename string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
Partitioned by(job string)
row format delimited fields terminated by '\t';
```

动态分区：

①开启允许进行动态分区

```
set hive.exec.dynamic.partition.mode=nonstrict
```

②分区列必须作为向表中插入数据的最后一列,注意顺序

```
insert into table emp_partition partition(job) select empno,ename,mgr,hiredate,sal,comm,deptno,job from emp;
```



严格模式，和分区严格模式，要区分。



## 操作问题

- <Hive>Hive建表问题

**操作：**create table t5(name string); 

**报错：**FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:For direct MetaStore DB connections, we don't support retries at the client level.)

**原因1：**所选库的字符集有问题。

**解决1：**将字符集UTF-8修改为 latin1， alter database hive character set latin1;

**原因2：**使用的metastore数据库（如MySQL）开启了日志bin_log，且日志记录方式binlog_format为STATEMENT。

**解决2：**将binlog_format，更改为ROW或者MIXED



- <mysql>mysql服务重启问题

The server quit without updating PID file(/[失败]b/mysql/hadoop103.pid).

**错误日志信息：**

```linux 
200305 14:28:45 mysqld_safe mysqld from pid file /var/lib/mysql/hadoop103.pid ended
200305 14:28:46 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2020-03-05 14:28:46 0 [ERROR] /usr/sbin/mysqld: Error while setting value 'row;' to 'binlog_format'
2020-03-05 14:28:46 0 [ERROR] Aborting

```

**解决方法：**

mysql的配置文件my.cnf。在写配置信息时，每一行后面不用加分号`;` ，否则就报错。

**其他可能可行的解决方法：**（解决方法待验证）

1.可能是/home/data/mysql 文件夹没有写的权限
解决方法 ：给予权限，执行 “chown -R mysql:mysql /home/data/mysql” “chmod -R 755 /home/data/mysql”  然后重新启动mysqld！

2.可能进程里已经存在mysql进程
解决方法：用命令“ps -ef|grep mysqld”查看是否有mysqld进程，如果有使用“kill -9  进程号”杀死，然后重新启动mysqld！

3.可能是第二次在机器上安装mysql，有残余数据影响了服务的启动。
解决方法：去mysql的数据目录/data看看，如果存在mysql-bin.index，就赶快把它删除掉吧，它就是罪魁祸首了。

4.mysql在启动时没有指定配置文件时会使用/etc/my.cnf配置文件，请打开这个文件查看在[mysqld]节下有没有指定数据目录(datadir)。
解决方法：请在[mysqld]下设置这一行：datadir = /home/data/mysql

5.skip-federated字段问题
解决方法：检查一下/etc/my.cnf文件中有没有没被注释掉的skip-federated字段，如果有就立即注释掉吧。

6.错误日志目录不存在
解决方法：使用“chown” “chmod”命令赋予mysql所有者及权限

7.selinux惹的祸，如果是centos或redhat系统，默认会开启selinux
解决方法：关闭它，打开/etc/selinux/config，把SELINUX=enforcing改为SELINUX=disabled后存盘退出重启机器试试。



## 操作随记-解题技巧

- 临时改为本地模式

```hive
set hive.exec.mode.local.auto=true;
set hive.exec.mode.local.auto.input.files.max=30;
set hive.exec.mode.local.auto.inputbytes.max=5000000000;
```



- 连续类的题目

先将需要计算连续的列进行升序排序；

做辅助列，用辅助列与连续列的差值，此时，差值相同的为一组连续的数，用这个方法找出连续的数据。







group by要在where的后面。
聚合字段的条件过滤，用having，写在group by后面。
在某两个日期之间，可以用between and，用字典排序，但是需要日期格式一致，如1月，应该是01而不是1，否则05（5月）与1（1月）相比，1大，就判断错误了。

窗口函数，聚集函数等，不能对这些字段进行使用，而需要再嵌套才可以使用这些字段。

蚂蚁金服

第一题

```mysql
1.统计用户在2017.10.01之前，获得的carbon总数。
(
select user_id, sum(carbon) as t_carbon
from user_carbon
where to_date(data_dt) between '2017-01-01' and '2017-10-01'
group by user_id
) t1
2.获取p004胡杨的，carbon值
(
select plant_carbon as hy
from plant_carbon
where plant_id = 'p004'
) t2

3.获取p002沙柳的，carbon值
(
select plant_carbon as sl
from plant_carbon
where plant_id = 'p002'
) t3

4.判断用户的carbon总数，是否需要减掉1颗胡杨的carbon值。得到能够消费的carbon数。
(
select user_id, t_carbon, if(t_carbon > hy, t_carbon-hy, t_carbon) as u_carbon
from t1
join t2
join t3
) t4

5.用消费的carbon数，算出可以购买的沙柳数目slnum，并排序
(
select user_id, t_carbon, u_carbon, floor(u_carbon / sl) as slnum
from t4
join t3
order by slnum
) t5

6.计算与下一名的差值
select user_id, t_carbon, u_carbon, slnum, slnum - LEAD(slnum, 1, 0) over() as diff
from t5

select user_id, t_carbon, u_carbon, slnum, slnum - LEAD(slnum, 1, 0) over() as diff
from (
select user_id, t_carbon, u_carbon, floor(u_carbon / sl) as slnum
from (
select user_id, t_carbon, if(t_carbon > hy, t_carbon-hy, t_carbon) as u_carbon
from (
select user_id, sum(carbon) as t_carbon
from user_carbon
where to_date(data_dt) between '2017-01-01' and '2017-10-01'
group by user_id
) t1
join (
select plant_carbon as hy
from plant_carbon
where plant_id = 'p004'
) t2
join (
select plant_carbon
from plant_carbon
where plant_id = 'p002'
) t3
) t4
join (
select plant_carbon as sl
from plant_carbon
where plant_id = 'p002'
) t3
order by slnum
) t5

yarn模式，运行时间84.0秒
```



第二题

```mysql

1.计算出每个用户，每天的carbon总量，即group by用户和日期
选出carbon总量在100以上的，且data_dt为2017年的
(
select user_id, data_dt, sum(carbon) as total_carbon
from user_carbon
where year(data_dt) = 2017
group by user_id, data_dt
having total_carbon >= 100    
) t1



2.增加一列，按排序列，从每个用户开始自动排序。并将排序列做差。
这里不需要关心排序列是int类型而不是时间格式的string类型。
因为用date_sub求日期差，会得到日期。这个日期就是差值。相同日期就可以。
(
select user_id, data_dt, total_carbon, row_number() over(partition by user_id order by data_dt) as tmp_time,
date_sub(data_dt, row_number() over(partition by user_id order by data_dt)) as diff_time
from t1;
) t2

3.用差值列和用户列排序，数顺序日期的天数。
(
select user_id, data_dt, total_carbon, count(*) over(partition by user_id, diff_time) as day_num
from (
select user_id, data_dt, total_carbon, row_number() over(partition by user_id order by data_dt) as tmp_time,
date_sub(data_dt,  row_number() over(partition by user_id)) as diff_time
from (
select user_id, data_dt, sum(carbon) as total_carbon
from user_carbon
where year(data_dt) = 2017
group by user_id, data_dt
having total_carbon >= 100    
) t1
) t2
) t3


(
select user_id, data_dt, total_carbon, count(*) over(partition by user_id, diff_time) as day_num
from t2
) t3

4.过滤出大于等于3的
(
select user_id, data_dt, total_carbon, day_num
from (
select user_id, data_dt, total_carbon, count(*) over(partition by user_id, diff_time) as day_num
from (
select user_id, data_dt, total_carbon, row_number() over(partition by user_id order by data_dt) as tmp_time,
date_sub(data_dt,  row_number() over(partition by user_id order by data_dt)) as diff_time
from (
select user_id, data_dt, sum(carbon) as total_carbon
from user_carbon
where year(data_dt) = 2017
group by user_id, data_dt
having total_carbon >= 100    
) t1
) t2
) t3 
where day_num >= 3
) t4
;

5.查询出符合要求的用户流水
select t.*
from (
select user_id, data_dt, total_carbon, day_num
from (
select user_id, data_dt, total_carbon, count(*) over(partition by user_id, diff_time) as day_num
from (
select user_id, data_dt, total_carbon, row_number() over(partition by user_id order by data_dt) as tmp_time,
date_sub(data_dt,  row_number() over(partition by user_id order by data_dt)) as diff_time
from (
select user_id, data_dt, sum(carbon) as total_carbon
from user_carbon
where year(data_dt) = 2017
group by user_id, data_dt
having total_carbon >= 100    
) t1
) t2
) t3 
where day_num >= 3
) t4 join user_carbon as t
on t4.user_id = t.user_id
and t4.data_dt = t.data_dt
order by t.user_id, t.data_dt;

yarn模式，运行时间61.6秒
```


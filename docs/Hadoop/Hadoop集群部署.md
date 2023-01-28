# Hadoop集群部署

# 1 上传压缩包并解压

# 2 查看Hadoop支持的压缩方式以及本地库

```
cd /export/servers/hadoop-2.6.0-cdh5.14.0 bin/hadoop checknative
```

如果出现openssl为false，组需要安装`openssl-devel`服务，如果可以使用yum则执行以下命令，进行安装

```
yum -y install openssl-devel
```

# 3 修改配置文件

## **core-site.xml**

hadoop核心配置文件

```
vim /etc/hadoop/core-site.xml
```

配置如下：

```xml
<configuration>
    <!-- 指定HDFS访问的域名地址  -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node01:8020</value>
	</property>
    <!-- 临时文件存储目录  -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/data/HadoopDatas/tempDatas</value>
	</property>
	<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>
	</property>

	<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
	<property>
		<name>fs.trash.interval</name>
		<value>10080</value>
	</property>
</configuration>
```

## hdfs-site.xml

hdfs配置文件

```
vim /etc/hadoop/hdfs-site.xml
```

配置如下：

```xml
<configuration>
	<!-- NameNode存储元数据信息的路径，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割   --> 
	<!--   集群动态上下线 
	<property>
		<name>dfs.hosts</name>
		<value>/export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop/accept_host</value>
	</property>
	
	<property>
		<name>dfs.hosts.exclude</name>
		<value>/export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop/deny_host</value>
	</property>
	 -->
	 
	 <property>
			<name>dfs.namenode.secondary.http-address</name>
			<value>node01:50090</value>
	</property>
	<!--  hdfs的集群的web  访问地址  -->
	<property>
		<name>dfs.namenode.http-address</name>
		<value>node01:50070</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///data/HadoopDatas/namenodeDatas</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用逗号进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///data/HadoopDatas/datanodeDatas</value>
	</property>
	  <!-- edits产生的文件存放路径 -->
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///data/HadoopDatas/dfs/nn/edits</value>
	</property>
    
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///data/HadoopDatas/dfs/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///data/HadoopDatas/dfs/nn/snn/edits</value>
	</property>
    <!-- 默认文件存储的副本数量 -->
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
    <!-- 关闭hdfs的文件权限 -->
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
    <!-- 默认 文件的存储的块的大小 -->
	<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
</configuration>
```

## hadoop-env.sh

hadoop需要加载的环境变量

```
vim /etc/hadoop/hadoop-env.sh
```

修改JAVA_HOME的路径为jdk的安装路径

```
export JAVA_HOME=/usr/java/jdk1.8.0_141
```

## mapred-site.xml

MapReduce配置文件

```
vim /etc/hadoop/mapred-site.xml
```

配置如下：

```xml
<configuration>
    <!--指定运行mapreduce的环境是yarn -->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<!--开启job的小任务模式-->
	<property>
		<name>mapreduce.job.ubertask.enable</name>
		<value>true</value>
	</property>
	 <!-- MapReduce JobHistory Server IPC host:port -->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>node01:10020</value>
	</property>
	<!-- MapReduce JobHistory Server Web UI host:port -->
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>node01:19888</value>
	</property>
</configuration>
```

## yarn-site.xml

yarn配置文件

```
vim /etc/hadoop/yarn-site.xml
```

配置如下：

```xml
<configuration>
    <!-- yarn resourcesmanager 的主机地址 -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>node01</value>
	</property>
    <!-- 逗号隔开的服务列表，列表名称应该只包含a-zA-Z0-9_,不能以数字开始-->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<!-- 启用日志聚合功能，应用程序完成后，收集各个节点的日志到一起便于查看 -->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
    <property>
		 <name>yarn.log.server.url</name>
		 <value>http://node01:19888/jobhistory/logs</value>
</property>
    <!--多长时间将聚合删除一次日志 此处 30 day -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>2592000</value>
	</property>
    <!--时间在几秒钟内保留用户日志。只适用于如果日志聚合是禁用的-->
<property>
        <name>yarn.nodemanager.log.retain-seconds</name>
        <value>604800</value><!--7 day-->
</property>
    
    <!--指定文件压缩类型用于压缩汇总日志-->
    <property>
            <name>yarn.nodemanager.log-aggregation.compression-type</name>
            <value>gz</value>
    </property>
    <!-- nodemanager本地文件存储目录-->
    <property>
            <name>yarn.nodemanager.local-dirs</name>
            <value>/data/HadoopDatas/yarn/local</value>
    </property>
    <!-- resourceManager  保存最大的任务完成个数 -->
    <property>
            <name>yarn.resourcemanager.max-completed-applications</name>
            <value>1000</value>
    </property>
</configuration>
```

## slaves

集群节点列表

```
vim /etc/hadoop/slaves
```

追加集群节点列表

```
node01
node02
node03
```

# 4 创建文件存放的目录

每个节点创建对应的数据目录

```
mkdir -p /data/HadoopDatas/tempDatas
mkdir -p /data/HadoopDatas/namenodeDatas
mkdir -p /data/HadoopDatas/datanodeDatas 
mkdir -p /data/HadoopDatas/dfs/nn/edits
mkdir -p /data/HadoopDatas/dfs/snn/name
mkdir -p /data/HadoopDatas/dfs/nn/snn/edits
```

# 5 安装包分发

使用`scp`分发安装包

# 6 配置环境变量

各机器配置hadoop环境变量

```
vim  /etc/profile
```

添加`HADOOP_HOME`

```
export HADOOP_HOME=/data/hadoop-2.6.0-cdh5.14.0
export PATH=:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

使环境变量生效

```
source /etc/profile
```



# 集群启停

注意：

- 要启动 Hadoop 集群，需要启动 HDFS 和 YARN 两个集群。
- 首次启动HDFS时，**必须对其进行格式化操作**。本质上是一些清理和准备工作，因为此时的 HDFS 在物理上还是不存在的。格式化：`bin/hdfs namenode  -format` 或者 `bin/hadoop namenode –format`

## 启动方式一: 单机节点依次启动

(1)在**主节点**上使用以下命令启动 HDFS NameNode： 

```
sbin/hadoop-daemon.sh start namenode
```

(2)在**每个节点**上使用以下命令启动 HDFS DataNode： 

```
sbin/hadoop-daemon.sh start datanode
```

(3)在**主节点**上使用以下命令启动 HDFS secondaryNameNode： 

```
sbin/hadoop-daemon.sh start secondarynamenode
```

(4)在**主节点**上使用以下命令启动 YARN ResourceManager： 

```
sbin/yarn-daemon.sh  start resourcemanager
```

(5)在**每个节点**上使用以下命令启动 YARN nodemanager： 

```
sbin/yarn-daemon.sh start nodemanager
```

(6)在**主节点**上使用以下命令启动jobhistory：

```
sbin/mr-jobhistory-daemon.sh start historyserver
```

以上脚本位于$HADOOP_PREFIX/sbin/目录下。**如果想要停止某个节点上某个角色，只需要把命令中的start 改为stop 即可。**

## 启动方式二: 脚本一键启动

如果配置了 `etc/hadoop/slaves` 和 ssh 免密登录，则可以使用程序脚本启动所有Hadoop 两个集群的相关进程，在主节点所设定的机器上执行。

node01节点上执行以下命令:

```
sbin/start-dfs.sh sbin/start-yarn.sh 
sbin/mr-jobhistory-daemon.sh start historyserver
```

## 停止集群

**一般没什么问题, 不要停止集群**

```
sbin/stop-dfs.sh sbin/stop-yarn.sh
sbin/mr-jobhistory-daemon.sh stop historyserver
```
# HDFS常用指令

# HDFS基本命令

| 指令          | 功能                                         | 使用格式                                       | 示例                                                         |
| ------------- | -------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| help          | 查看命令参数                                 | `hdfs dfs -help 要查看的指令`                  | `hdfs dfs -help rm`                                          |
| ls            | 显示目录信息                                 | `hdfs dfs -ls 目录`                            | `hdfs dfs -ls /tmp`                                          |
| lsr           | 展示整个目录下的信息，等价于hdfs dfs -ls -R  | `hdfs dfs -lsr 目录`                           | `hdfs dfs -lsr /tmp`                                         |
| mkdir         | 创建目录                                     | `hdfs dfs -mkdir`                              | `hdfs dfs  -mkdir  -p  /a/b/c`                               |
| put           | 从本地文件系统中拷贝文件到hdfs目录           | `hdfs dfs -put 本地文件 hdfs目录`              | `hdfs dfs -put /a.txt  /user`                                |
| copyFromLocal | 同put                                        |                                                |                                                              |
| moveFromLocal | 从本地文件系统中移动文件到hdfs               | `hdfs dfs  -moveFromLocal 本地文件 hdfs目录`   | `hdfs dfs  -moveFromLocal  /root/error.log  /a/b/c/`         |
| appendToFile  | 追加一个本地文件到hdfs上已存在的文件末尾     | `hdfs dfs -appendToFile 本地文件 hdfs目标文件` | `hdfs dfs -appendToFile /a.txt /b.txt`                       |
| get           | 从hdfs下载文件到本地                         | `hdfs dfs -get hdfs文件 本地目录`              | `hdfs dfs -get /tmp/a.txt /localTmp`                         |
| copyToLocal   | 同get                                        |                                                |                                                              |
| getmerge      | 合并下载目录下的多个文件到本地一个文件       | `hdfs dfs -getmerge hdfs多个文件 本地目录`     | `hdfs dfs -getmerge /tmp/*.log /localTmp/error.log`          |
| mv            | 在hdfs目录中移动文件                         | `hdfs dfs -mv 源文件 目标目录`                 | `hdfs dfs -mv /a/a.txt /tmp`                                 |
| rm            | 在hdfs上删除文件或目录                       | `hdfs dfs -rm 要删除的文件`                    | `hdfs dfs -rm -f /tmp/a.log` 强制删除文件<br>`hdfs dfs -rm -r /tmp/a/` 删除目录 |
| cp            | 在hdfs目录中拷贝一个文件或目录到另一个目录中 | `hdfs dfs -cp 源文件或目录 目标目录`           | `hdfs dfs -cp /tmp/a.log /tmp/a/`                            |
| cat           | 显示文件内容                                 | `hdfs dfs -cat 文件`                           | `hdfs dfs -cat /tmp/a.log`                                   |
| tail          | 显示文件的末尾                               | `hdfs dfs -tail 文件`                          | `hdfs dfs -tail -f /tmp/a.log` 动态查看文件末尾              |
| chmod         | hdfs文件或目录权限修改                       | `hdfs dfs -chmod 权限 文件或目录`              | `hdfs dfs -chmod  666 /a.txt`                                |
| chown         | hdfs文件或目录拥有者或所属群组修改           | `hdfs dfs -chown 所有者:所有组 /a.txt`         | `hdfs dfs -chown someuser:somegrp   /a.txt`                  |
| df            | 统计hdfs可用空间                             | `hdfs dfs -df /`                               | `hdfs dfs -df -h /`                                          |
| du            | 统计目录大小                                 | `hdfs dfs -du 目录`                            | `hdfs dfs -du -s -h /tmp`                                    |
| count         | 统计一个指定目录下的文件节点数量             | `hdfs dfs -count 目录`                         | `hdfs dfs -count /tmp/`                                      |
| setrep        | 设置hdfs中文件的副本数量                     | `hdfs dfs -setrep 副本数 目标文件`             | `hdfs dfs -setrep 1 /tmp/a.log`                              |
| expunge       | 清空hdfs垃圾桶                               | `hdfs dfs -expunge`                            | `hdfs dfs -expunge`                                          |



# HDFS高级命令

## HDFS文件限额配置

对某一个目录进行数量和空间大小的限额

### 数量限额

```
hdfs dfs -mkdir -p /user/root/lisi   #创建hdfs文件夹
hdfs dfsadmin -setQuota 2 lisi   # 给该文件夹下面设置最多上传两个文件，上传文件，发现只能上传一个文件
hdfs dfsadmin -clrQuota /user/root/lisi  # 清除文件数量限制       
```

### 空间大小限额

```
hdfs dfsadmin -setSpaceQuota 4k /user/root/lisi   # 限制空间大小4KB
hdfs dfs -put  /data/bigtext.txt /user/root/lisi #上传超过4Kb的文件大小上去提示文件超过限额
hdfs dfsadmin -clrSpaceQuota /user/root/lisi   #清除空间限额
hdfs dfs -put  /data/bigtext.txt /user/root/lisi     
```

## HDFS安全模式

Hadoop的安全机制，用于保证集群中的数据块的安全性。

当Hadoop进行入到安全模式下，不允许用户对数据进行操作（增删改）。

Hadoop默认在启动之后的30s之内是在安全模式下的， 30s后自动离开安全模式。因为在30s内，Hadoop要进行自检。

### 查看安全模式状态

```
hdfs dfsadmin -safemode get
```

### 进入安全模式

```
hdfs dfsadmin -safemode enter
```

### 离开安全模式

```
hdfs dfsadmin -safemode leave
```


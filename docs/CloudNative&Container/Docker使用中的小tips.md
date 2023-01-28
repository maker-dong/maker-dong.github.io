# Docker使用中的小tips

# 修改国内镜像源

> 国内使用dockerhub拉去镜像速度缓慢，为提升效率可切换国内镜像仓库。

**编辑daemon.json**，如果没有则创建

```shell
vi /etc/docker/daemon.json
```

内容如下：

```json
{
    "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com"
    ]
}
```

此处我配置的是腾讯的镜像仓库，再提供几个常用的国内仓库，可根据实际情况选择最优的：

- docker中国：[https://registry.docker-cn.com](https://registry.docker-cn.com/)
- 网易：[http://hub-mirror.c.163.com](http://hub-mirror.c.163.com/)
- 中国科技大学：[https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn/)
- 阿里云：https://cr.console.aliyun.com/

**编辑完成后重启docker服务**

```shell
systemctl daemon-reload
systemctl restart docker
```

# 镜像编码格式

> 有些镜像的编码格式不支持中文，想要使其支持中文，需要修改为`C.UTF-8`格式。

**临时修改**
查看所有支持的编码格式：

```shell
locale -a
```

修改编码：

```shell
LANG=C.UTF-8
source /etc/profile
```

有些系统没有C.UTF-8，那就修改为zh_CN.UTF-8

**永久修改**
重新制作镜像，在Dockerfile中添加一行：

```language
ENV LANG C.UTF-8
```

# 镜像时区修改

> 一般docker镜像的时间与系统的时间是不一致的，原因是使用的时区不同，系统时区一般为Asia/Shanghai，需要对镜像的时区进行更改。

**复制本机时区文件**
将系统`/usr/share/zoneinfo/Asia/Shanghai`文件拷贝到Dockerfile所在的目录

**修改Dockerfile**

删除基础镜像的时间文件

```language
RUN rm -rf /etc/localtime
```

创建目录（可能有些基础镜像没有该目录）

```language
RUN mkdir -p /usr/share/zoneinfo/Asia
```

拷贝文件`Shanghai`到`/usr/share/zoneinfo/Asia/`

```language
COPY Shanghai /usr/share/zoneinfo/Asia/
```

创建软连接

```language
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

# 镜像的导入与导出

> 有时需要在一些特殊的环境下（如不能连接外网）使用docker，这就需要在一台能联网的机器上将镜像拉取后进行导出，拷贝到目标机器上再进行导入。

**导出镜像**
使用save指令将镜像导出到tar文件。

```shell
docker save -o 导出文件名.tar 镜像名:tag
```

示例：

```shell
docker save -o flink-image.tar apache/flink:1.12.4-scala_2.12
```

**加载镜像**
使用load指令将tar文件加载为docker镜像。

```shell
docker load < 导出文件名.tar
```

示例：

```shell
docker load < flink-image.tar
```

# 批量删除docker镜像/容器

> 有时我们需要批量删除docker中的容器或镜像，比如删库跑路的时候（不是）

**直接删除所有镜像**

```shell
docker rmi `docker images -q`
```

**直接删除所有容器**

```shell
docker rm `docker ps -aq`
```

**按条件筛选之后删除镜像**

```shell
docker rmi `docker images | grep xxxxx | awk '{print $3}'`
```

**按条件筛选之后删除容器**

```shell
docker rm `docker ps -a | grep xxxxx | awk '{print $1}'`
```

**删除所有停止的容器**

```shell
docker rm `docker ps -a | grep Exited | awk '{print $1}'`
```
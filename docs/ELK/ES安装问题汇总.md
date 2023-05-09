# ES安装问题汇总

# max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

## 问题描述

ES启动报错。其原因是ES需要的的最小max file descriptors为65535，我们设置的是4096，需要增大max file descriptors的值。

## 解决方案

调大系统的max file descriptors值

在`/etc/security/limits.conf`中新增两行配置：

```
* hard nofile 65536
* soft nofile 65536
```

这里`*`代表所有用户，如果要为指定用户调整参数，则将`*`替换为指定的用户名。

调整完毕后可查看参数：

```
$ ulimit -Hn
65536
$ ulimit -Sn
65536
```

# memory locking requested for elasticsearch process but memory is not locked

## 问题描述

ES启动报错。原因是ES进程请求内存锁定，但内存未锁定。

## 解决方案

### 方案一 关闭`bootstrap.memory_lock`

在`elasticsearch.yml`中配置`bootstrap.memory_lock: false`关闭内存锁。

此方案不推荐使用，关闭内存锁会影响性能。

### 方案二 开启memlock 

在`/etc/security/limits.conf`中追加配置：

```
* hard memlock unlimited
* soft memlock unlimited
```

这里`*`代表所有用户，如果要为指定用户调整参数，则将`*`替换为指定的用户名。

# max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

## 问题描述

ES启动报错。系统虚拟内存默认最大映射数为65530，无法满足ES系统要求，需要调整为262144以上。

## 解决方案

调大系统虚拟内存最大映射数。

在`/etc/sysctl.conf`中追加：

```
vm.max_map_count = 262144
```

重新加载系统设置：

```
sysctl -p
```


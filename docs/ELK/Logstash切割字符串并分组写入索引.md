# Logstash切割字符串并分组写入索引


# Logstash配置改造

改造之前我们将nginx日志按照日期进行了分割，做到每天一个索引，实现方法是在Logstash配置文件的output中将日期写入到索引名称中：

```ruby
  if [type] == "nginx-log" {
    elasticsearch {
      hosts => ["es06:9200","es07:9200","es08:9200"]
      index => "nginx-log-%{+YYYY.MM.dd}"
    }
```

现在我们要将索引按项目组划分索引，所以要在索引字段上再添加一个省份分组标识的字段。

那么省份分组标识字段如何获取？

我们要操作的是前置机日志，前置机日志中是有请求url的，在日志中我们叫`body_bytes_sent`，我们要做的是截取body_bytes_sent中的第一段，比如body_bytes_sent为：`/aaa/server/add`，我们需要截取第一段`aaa`，这就是所属的项目组名称。但目前我们拿到的是项目组的名称，但最终我们要得到省份的分组名称，有些省份下面会包含多个项目。这样的话我们就需要再将项目组名称与省份组标识做一个映射。

我们捋一下目前要做的事情：

1. 切分url，获取项目组名称
2. 将项目组名称与省份标识进行映射
3. 将省份标识拼接到索引名称

接下来我们将一步一步进行处理。

## 1 切分url，获取项目组名称

通过查看Logstash mutate的相关资料，mutate中提供一个split方法，可以对字符串进行切割。

用法是：

```ruby
mutate {
	split => [待切割字段 , 分隔符]
}
```

随即我尝试使用split方法对body_bytes_sent进行切割，并取下标为1的字段添加到新字段group。

此处注意数组下标是从0开始的，由于我们的url第一个字符是`/`，所以切割出来的第一个数组元素是""。

```ruby
mutate {
	split => ["body_bytes_sent" , "/"]
	add_field => ["group", "%{[body_bytes_sent][1]}"]
}
```

尝试将索引写入ES，确实group字段取到了值，但body_bytes_sent字段被切割成了逗号分割的数组，比如`/aaa/server/add`，group字段取值aaa，但body_bytes_sent切割成了`, aaa, server, add`。

为了避免这种情况，我决定先将body_bytes_sent复制一份出来，然后操作拷贝后的字段。

随后程序改为这样：

```ruby
mutate {
	copy => {"body_bytes_sent" => "body_bytes_sent_copy"}
	split => ["body_bytes_sent_copy" , "/"]
	add_field => ["group", "%{[body_bytes_sent_copy][1]}"]
}
```

讲道理说应该没问题了吧，但写入ES后，`%{[body_bytes_sent_copy][1]}`并没有取到值，仔细观察body_bytes_sent与body_bytes_sent_copy都没有被切割。非常奇怪，从逻辑上看似乎没什么问题啊。

经过查找资料，原来mutate中方法执行的优先级是有顺序的：

> mutate 在配置文件中的执行顺序
>
> 1. coerce
> 2. rename
> 3. update
> 4. replace
> 5. convert
> 6. gsub
> 7. uppercase
> 8. capitalize
> 9. lowercase
> 10. strip
> 11. remove
> 12. split
> 13. join merge
> 14. copy

所以，同一个mutate中，copy方法是最后执行的。也就是说在我的程序中，先执行了split与add_filed，此时body_bytes_sent_copy字段还是空的，所以group取值`%{[body_bytes_sent_copy][1]}`是取不到的，最后执行copy，所以此时body_bytes_sent_copy取到了body_bytes_sent的值，以至于两个字段都没有被切割。

解决这个问题很简单，给copy方法单独写到一个mutate里就可以了，此时程序是这样的：

```ruby
mutate {
    copy => {"body_bytes_sent" => "body_bytes_sent_copy"}
}

mutate {
  split => ["body_bytes_sent_copy" , "/"]
  add_field => ["group", "%{[body_bytes_sent_copy][1]}"]
}
```

此时group获取完成。

## 2 将项目组名称与省份标识进行映射

我们上面说过，省份标识与项目组名称是一对多的关系，所以需要进行项目组名与省份标识的映射。为了区分两个字段，由于省份标识是要体现在索引名称上的，这里我们为其命名为index-group。此处我用了个比较笨的方法进行映射，就是通过对不同的group进行条件判断，从而确定index-group的内容。


```ruby
if [group] in "mcs,ubp,ubp-2021" {
  mutate {
    add_field => {"index-group", "sd"}
  }
} else if [group] in "cmgs,gsubp,gsubp-prod" {
  mutate {
    add_field => {"index-group", "gs" }
  }
} else if [group] in "cmqh" {
  mutate {
    add_field => {"index-group", "qh" }
  }
} else if [group] in "cmnmg" {
  mutate {
    add_field => {"index-group", "nmg" }
  }
} else if [group] in "cmjl" {
  mutate {
    add_field => {"index-group", "jl" }
  }
} else if [group] in "nxsst" {
  mutate {
    add_field => {"index-group", "nx" }
  }
} else if [group] in "cmtj" {
  mutate {
    add_field => {"index-group", "tj" }
  }
} else {
  mutate {
	add_field => ["index-group", "default"]
  }
}
```

此处实现了如果group内容属于特定的几个组名，则为其添加新字段index-group，并赋值省份标识。

比如第一段中：

```ruby
if [group] in "mcs,ubp,ubp-2021" {
  mutate {
    add_field => {"index-group", "sd"}
  }
} 
```

如果组名称是mcs或ubp或ubp-2021的则为该条数据添加新字段index-group，值为sd。

最后再通过else进行兜底，将不属于上述提到的组的数据统一设置给default组。

## 3 将省份标识拼接到索引名称

到此我们已经将数据按照省份进行了“打标签”，接下来在写入索引时将index-group字段写入索引名称就大功告成了。

```ruby
if [type] == "nginx-log" {
  elasticsearch {
    hosts => ["es06:9200","es07:9200","es08:9200"]
    index => "nginx-log-%{index-group}-%{+YYYY.MM.dd}"
  }
```

启动Logstash，进入ES查看数据，索引已经按照省份标识进行了创建，数据一切正常。

此时在kibana中创建索引模式`nginx-log*`可以查看索引省份的数据，如果想查看某一省份的数据可以按省份标识创建索引模式，比如`nginx-log-sd*`。



# 参考资料

- [elastic官方文档plugins-filters-mutate](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html)
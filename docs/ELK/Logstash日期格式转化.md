# Logstash日期格式转化

## 使用示例
logstash的date过滤插件，支持从字段里分析日期格式，然后放入@timestamp字段里。

```
filter {
    date {
      match => ["create_at", "yyyy-MM-dd HH:mm:ss,SSS", "UNIX"]
      target => "@timestamp"
      locale => "cn"
    }
}
```



## match

- 第一个参数是字段名。
- 第二个参数是格式化模式
- 第三个参数是要转换的时间格式，如下表

| 格式    | 示例                       |
| ------- | -------------------------- |
| ISO8601 | 2011-04-19T03:44:01.103Z   |
| UNIX    | 1326149001.132或1326149001 |
| TAI64N  | 64位时间戳                 |


参考：
https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html#plugins-filters-date-match
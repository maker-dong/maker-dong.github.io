# ES索引模板

## 创建索引模板

````
PUT _template/fruits_template
{
    "index_patterns": ["shop*", "bar*"],       // 可以通过"shop*"和"bar*"来适配, template字段已过期
    "order": 0,                // 模板的权重, 多个模板的时候优先匹配用, 值越大, 权重越高
    "settings": {
        "number_of_shards": 1  // 分片数量, 可以定义其他配置项
    },
    "aliases": {
        "alias_1": {}          // 索引对应的别名
    },
    "mappings": {
        // ES 6.0开始只支持一种type, 名称为“_doc”
        "_doc": {
            "_source": {            // 是否保存字段的原始值
                "enabled": false
            },
            "properties": {        // 字段的映射
                "@timestamp": {    // 具体的字段映射
                    "type": "date",           
                    "format": "yyyy-MM-dd HH:mm:ss"
                },
                "@version": {
                    "doc_values": true,
                    "index": "false",   // 设置为false, 不索引
                    "type": "text"      // text类型
                },
                "logLevel": {
                    "type": "long"
                }
            }
        }
    }
}
````

## 查看索引模板

````
GET _template // 查看所有模板

GET _template/temp* // 查看与通配符相匹配的模板

GET _template/temp1,temp2 // 查看多个模板

GET _template/fruits_template // 查看指定模板
````

## 判断模板是否存在:

````
HEAD _template/fruits_tem
````

结果说明:

- 如果存在, 响应结果是: 200 - OK
- 如果不存在, 响应结果是: 404 - Not Found

## 删除索引模板

````
DELETE _template/fruits_template // 删除上述创建的模板
````

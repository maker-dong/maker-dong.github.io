# ES导入导出工具elasticdump简单使用说明


elasticdump是一款开源的es数据导入与导出工具。

项目地址：[elasticdump](https://github.com/elasticsearch-dump/elasticsearch-dump)

# 安装

elasticdump基于npm实现，可以使用npm进行安装，官方也提供docker镜像。

npm安装：

```
npm i elasticdump
```

docker安装：

```
docker pull taskrabbit/elasticsearch-dump
```

在此为了方便，我使用docker进行安装。

# 导出

## 数据导出

先给一个**示例**：

```shell
docker run --rm -ti -v /root/mnt/elasticsearch/data:/tmp taskrabbit/elasticsearch-dump \
--input=http://es用户名:密码@es地址:9200/索引名称 \
--output=/tmp/output.json \
--type=data
```

**参数详解**：

```
--rm									运行完成后删除容器
-ti										前台运行，如果想后台运行则高改为 -d
-v /root/mnt/elasticsearch/data:/tmp	将本地目录映射到容器中的/tmp目录
taskrabbit/elasticsearch-dump			镜像名称
--input									给出要导出的es地址、端口、用户名、密码、索引名称
--output								给出导出文件的存储地址路径与文件名（注意，此处使用docker容器进行导出，所以要写容器内的地址，建议直接使用/tmp目录）
--type									要导出的是数据，所以填data
--limit 10000							限制一次导出最大行数，不写默认是1000
```

以上是一个比较简单的导出。

## 按条件导出数据

如果要想在导出时加入查询筛选，需要添加`--searchBody`字段。

给出**示例**：

```shell
docker run --rm -d -v /root/mnt/elasticsearch/data:/tmp taskrabbit/elasticsearch-dump \
--input=http://es用户名:密码@es地址:9200/索引名称 \
--output=/tmp/output.json \
--type=data \
--searchBody='{"query":{"bool":{"must":[{"match_phrase":{"body_bytes_sent":"cmgs"}},{"range":{"@timestamp":{"gte":"2022-05-01T00:00:00.000+08:00","lt":"2022-06-01T00:00:00.000+08:00"}}}]}}}' \
--limit 10000
```

这里在`searchBody`参数中可以添加一个DSL查询

```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "match_phrase": {
                        "body_bytes_sent": "cmgs"
                    }
                },
                {
                    "range": {
                        "@timestamp": {
                            "gte": "2022-05-01T00:00:00.000+08:00",
                            "lt": "2022-06-01T00:00:00.000+08:00"
                        }
                    }
                }
            ]
        }
    }
}
```

此处我就是要导出`"body_bytes_sent": "cmgs"`并且时间范围在2022-05-01T00:00:00.000+08:00~2022-06-01T00:00:00.000+08:00的数据。以此类推，可以添加各种的查询作为导出条件。

## 模板导出

索引模板的导出与索引数据的导出几乎是一样的，给出一个实例：

```shell
docker run --rm -ti -v /root/mnt/elasticsearch/data:/tmp taskrabbit/elasticsearch-dump \
--input=http://es用户名:密码@es地址:9200/索引模板名称 \
--output=/tmp/output_template.json \
--type=template
```

注意此处要将`--type`指定为`template`类型。

# 导入

## 数据导入

数据的导入就是要把导出时的input与output进行调换，将导出的文件作为导入时的input。

```shell
docker run --rm -ti -v /root/mnt/elasticsearch/data:/tmp taskrabbit/elasticsearch-dump \
--input=/tmp/input.json \
--output=http://es用户名:密码@es地址:9200/索引名称 \
--type=data
```

## 模板导入

索引模板的导入跟数据的导入几乎一样，注意要把`--type`指定为`template`类型。

```shell
docker run --rm -ti -v /root/mnt/elasticsearch/data:/tmp taskrabbit/elasticsearch-dump \
--input=/tmp/zhyd_business_template.json \
--output=http://es用户名:密码@es地址:9200 \
--type=template
```



# 附 elasticdump参数表

```
命令

elasticdump: Import and export tools for elasticsearch
version: %%version%%

Usage: elasticdump --input SOURCE --output DESTINATION [OPTIONS]

--input
                    Source location (required)
--input-index
                    Source index and type
                    (default: all, example: index/type)
--output
                    Destination location (required)
--output-index
                    Destination index and type
                    (default: all, example: index/type)
--overwrite
                    Overwrite output file if it exists
                    (default: false)                    
--limit
                    How many objects to move in batch per operation
                    limit is approximate for file streams
                    (default: 100)
--size
                    How many objects to retrieve
                    (default: -1 -> no limit)
--concurrency
                    The maximum number of requests the can be made concurrently to a specified transport.
                    (default: 1)       
--concurrencyInterval
                    The length of time in milliseconds in which up to <intervalCap> requests can be made
                    before the interval request count resets. Must be finite.
                    (default: 5000)       
--intervalCap
                    The maximum number of transport requests that can be made within a given <concurrencyInterval>.
                    (default: 5)
--carryoverConcurrencyCount
                    If true, any incomplete requests from a <concurrencyInterval> will be carried over to
                    the next interval, effectively reducing the number of new requests that can be created
                    in that next interval.  If false, up to <intervalCap> requests can be created in the
                    next interval regardless of the number of incomplete requests from the previous interval.
                    (default: true)                                                                                       
--throttleInterval
                    Delay in milliseconds between getting data from an inputTransport and sending it to an
                    outputTransport.
                     (default: 1)
--debug
                    Display the elasticsearch commands being used
                    (default: false)
--quiet
                    Suppress all messages except for errors
                    (default: false)
--type
                    What are we exporting?
                    (default: data, options: [settings, analyzer, data, mapping, alias, template])
--delete
                    Delete documents one-by-one from the input as they are
                    moved.  Will not delete the source index
                    (default: false)
--searchBody
                    Preform a partial extract based on search results
                    when ES is the input, default values are
                      if ES > 5
                        `'{"query": { "match_all": {} }, "stored_fields": ["*"], "_source": true }'`
                      else
                        `'{"query": { "match_all": {} }, "fields": ["*"], "_source": true }'`
--headers
                    Add custom headers to Elastisearch requests (helpful when
                    your Elasticsearch instance sits behind a proxy)
                    (default: '{"User-Agent": "elasticdump"}')
--params
                    Add custom parameters to Elastisearch requests uri. Helpful when you for example
                    want to use elasticsearch preference
                    (default: null)
--sourceOnly
                    Output only the json contained within the document _source
                    Normal: {"_index":"","_type":"","_id":"", "_source":{SOURCE}}
                    sourceOnly: {SOURCE}
                    (default: false)
--ignore-errors
                    Will continue the read/write loop on write error
                    (default: false)
--scrollTime
                    Time the nodes will hold the requested search in order.
                    (default: 10m)
--maxSockets
                    How many simultaneous HTTP requests can we process make?
                    (default:
                      5 [node <= v0.10.x] /
                      Infinity [node >= v0.11.x] )
--timeout
                    Integer containing the number of milliseconds to wait for
                    a request to respond before aborting the request. Passed
                    directly to the request library. Mostly used when you don't
                    care too much if you lose some data when importing
                    but rather have speed.
--offset
                    Integer containing the number of rows you wish to skip
                    ahead from the input transport.  When importing a large
                    index, things can go wrong, be it connectivity, crashes,
                    someone forgetting to `screen`, etc.  This allows you
                    to start the dump again from the last known line written
                    (as logged by the `offset` in the output).  Please be
                    advised that since no sorting is specified when the
                    dump is initially created, there's no real way to
                    guarantee that the skipped rows have already been
                    written/parsed.  This is more of an option for when
                    you want to get most data as possible in the index
                    without concern for losing some rows in the process,
                    similar to the `timeout` option.
                    (default: 0)
--noRefresh
                    Disable input index refresh.
                    Positive:
                      1. Much increase index speed
                      2. Much less hardware requirements
                    Negative:
                      1. Recently added data may not be indexed
                    Recommended to use with big data indexing,
                    where speed and system health in a higher priority
                    than recently added data.
--inputTransport
                    Provide a custom js file to use as the input transport
--outputTransport
                    Provide a custom js file to use as the output transport
--toLog
                    When using a custom outputTransport, should log lines
                    be appended to the output stream?
                    (default: true, except for `$`)
--transform
                    A javascript, which will be called to modify documents
                    before writing it to destination. global variable 'doc'
                    is available.
                    Example script for computing a new field 'f2' as doubled
                    value of field 'f1':
                        doc._source["f2"] = doc._source.f1 * 2;
                    May be used multiple times.
                    Additionally, transform may be performed by a module. See [Module Transform](#module-transform) below.
--awsChain
                    Use [standard](https://aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/) location and ordering for resolving credentials including environment variables, config files, EC2 and ECS metadata locations
                    _Recommended option for use with AWS_
--awsAccessKeyId
--awsSecretAccessKey
                    When using Amazon Elasticsearch Service protected by
                    AWS Identity and Access Management (IAM), provide
                    your Access Key ID and Secret Access Key.
                    --sessionToken can also be optionally provided if using temporary credentials
--awsIniFileProfile
                    Alternative to --awsAccessKeyId and --awsSecretAccessKey,
                    loads credentials from a specified profile in aws ini file.
                    For greater flexibility, consider using --awsChain
                    and setting AWS_PROFILE and AWS_CONFIG_FILE
                    environment variables to override defaults if needed
--awsIniFileName
                    Override the default aws ini file name when using --awsIniFileProfile
                    Filename is relative to ~/.aws/
                    (default: config)
--support-big-int   
                    Support big integer numbers
--retryAttempts  
                    Integer indicating the number of times a request should be automatically re-attempted before failing
                    when a connection fails with one of the following errors `ECONNRESET`, `ENOTFOUND`, `ESOCKETTIMEDOUT`,
                    ETIMEDOUT`, `ECONNREFUSED`, `EHOSTUNREACH`, `EPIPE`, `EAI_AGAIN`
                    (default: 0)
                    
--retryDelay   
                    Integer indicating the back-off/break period between retry attempts (milliseconds)
                    (default : 5000)            
--parseExtraFields
                    Comma-separated list of meta-fields to be parsed  
--fileSize
                    supports file splitting.  This value must be a string supported by the **bytes** module.     
                    The following abbreviations must be used to signify size in terms of units         
                    b for bytes
                    kb for kilobytes
                    mb for megabytes
                    gb for gigabytes
                    tb for terabytes
                    
                    e.g. 10mb / 1gb / 1tb
                    Partitioning helps to alleviate overflow/out of memory exceptions by efficiently segmenting files
                    into smaller chunks that then be merged if needs be.
--fsCompress
                    gzip data before sending outputting to file 
--s3AccessKeyId
                    AWS access key ID
--s3SecretAccessKey
                    AWS secret access key
--s3Region
                    AWS region
--s3Endpoint        
                    AWS endpoint can be used for AWS compatible backends such as
                    OpenStack Swift and OpenStack Ceph
--s3SSLEnabled      
                    Use SSL to connect to AWS [default true]
                    
--s3ForcePathStyle  Force path style URLs for S3 objects [default false]
                    
--s3Compress
                    gzip data before sending to s3  

--retryDelayBase
                    The base number of milliseconds to use in the exponential backoff for operation retries. (s3)
--customBackoff
                    Activate custom customBackoff function. (s3)
--tlsAuth
                    Enable TLS X509 client authentication
--cert, --input-cert, --output-cert
                    Client certificate file. Use --cert if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--key, --input-key, --output-key
                    Private key file. Use --key if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--pass, --input-pass, --output-pass
                    Pass phrase for the private key. Use --pass if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--ca, --input-ca, --output-ca
                    CA certificate. Use --ca if source and destination are identical.
                    Otherwise, use the one prefixed with --input or --output as needed.
--inputSocksProxy, --outputSocksProxy
                    Socks5 host address
--inputSocksPort, --outputSocksPort
                    Socks5 host port
--help
                    This page

```


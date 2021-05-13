---
title:ElasticSearch的简单使用
---

> ​    这篇文章使用的ES都是直接运行在Windows里的，并不是使用docker启动的，虽然在之前的博客里面我在docker上安装了ES并且正常启动了，但是目前我docker使用的非常不熟练，es上要安装ik分词器，但是我根本不知道怎么修改docker里image文件的内容和向image里面添加文件。其次，我之前搜索引擎使用的是solr，里面需要对每一个core配置一个结构，我想es也应该是这样，而且在我百度了一些文章之后发现的确如此，但是如果要我在docker里面频繁的修改我可能要花很多时间在docker的学习上。docker的确很有用，容器技术也是一个很方便的东西，但是不是我现在要专注的东西，所以我决定先直接在Windows上运行es，就像我之前选择在tomcat上运行solr一样。
>
> ```这篇文章写的没啥用，但是没办法，浪费了我的时间，所以我还是发出来充数了```

## 1.安装ElasticSearch

​	在Windows环境中安装ElasticSearch很简单，直接在官网下载zip文件，解压即可。但是电脑中要提前安装好好Java，在环境变量中配置好JAVA_HOME，然后直接进入bin目录中运行elasticsearch即可。启动之后可以在浏览器中访问[http://localhost:9200]()，如果es正常启动会返回如下数据：

```json
{
  "name" : "DESKTOP-C36HRED",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "mXdKNKyiSFyDVQB8Z3rwJQ",
  "version" : {
    "number" : "7.9.2",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "d34da0ea4a966c4e49417f2da2f244e3e97b4e6e",
    "build_date" : "2020-09-23T00:45:33.626720Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

​	目前在安装运行中并未遇到无法启动的问题，但是好像会有虚拟内存不够的情况，就是再docker启动中的问题一，但是我现在没有遇到，我也没有自己动手处理，之后我会再在我个人的电脑上安装一次，如果那个时候出现别的问题我再补充上来。

---

## 2.安装ik分词器

​	IK分词器是非常优秀的一个中文分词器，ElasticSearch是有一些分词功能的，但是我不太清楚它是否像solr一样内置了中文分词功能，而且即便是solr我也选择使用ik分词器。IK分词器的安装也非常简单，可以访问[IK分词器Github项目](https://github.com/medcl/elasticsearch-analysis-ik/releases)直接下载对应版本的IK分词器，一般来说IK分词器的版本号和ES版本号对应。

​	下载压缩文件后将其在ElasticSearch目录下的plugins文件夹中新建一个文件夹，将其解压到该文件夹内即可。重启ElasticSearch即可加载ik分词器了。

### 安装IK分词器出现的错误

​	笔者当时没有在plugins文件夹下新建文件夹，直接就解压到plugins文件中了，导致出现了以下的错误，解决方法也很简单，不要直接把压缩文件的内容直接解压到plugins文件内即可。本来我解压的时候就很奇怪，还在想如果有多个插件这个文件夹内容不就很乱了，结果报错我还找了半天问题出在哪里，本来我安装的是7.12.1的版本，最新版，我还以为是版本太新了，有删除了重新安装了7.9.2的版本，结果还不可以，最后才发现是我自己傻了·····

```
[2021-05-10T14:50:39,910][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [DESKTOP-C36HRED] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.IllegalStateException: Could not load plugin descriptor for plugin directory [commons-codec-1.9.jar]
```

​	还有其他的错误信息，很长，但是对于我也没啥用，我也看不懂，一般就保留最上面的错误信息用来百度就可以了。

**OK至此我的elasticsearch就可以运行了，并且包含了ik分词器，接下来就是一些简单的使用**

---

## 3.ElasticSearch中的基本概念

> 这一节的内容是摘自```阮一峰的网络日志```中的```全文搜索引擎 Elasticsearch 入门教程```

### 3.1 Node 与 Cluster

Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。

### 3.2 Index

Elastic 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。

所以，Elastic 数据管理的顶层单位就叫做 Index（索引）。它是单个数据库的同义词。每个 Index （即数据库）的名字必须是小写。

下面的命令可以查看当前节点的所有 Index。

> ```bash
> $ curl -X GET 'http://localhost:9200/_cat/indices?v'
> ```

### 3.3 Document

Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。

Document 使用 JSON 格式表示，下面是一个例子。

> ```javascript
> {
>   "user": "张三",
>   "title": "工程师",
>   "desc": "数据库管理"
> }
> ```

同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。

### 3.4 Type

Document 可以分组，比如`weather`这个 Index 里面，可以按城市分组（北京和上海），也可以按气候分组（晴天和雨天）。这种分组就叫做 Type，它是虚拟的逻辑分组，用来过滤 Document。

不同的 Type 应该有相似的结构（schema），举例来说，`id`字段不能在这个组是字符串，在另一个组是数值。这是与关系型数据库的表的[一个区别](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html)。性质完全不同的数据（比如`products`和`logs`）应该存成两个 Index，而不是一个 Index 里面的两个 Type（虽然可以做到）。

下面的命令可以列出每个 Index 所包含的 Type。

> ```bash
> $ curl 'localhost:9200/_mapping?pretty=true'
> ```

根据[规划](https://www.elastic.co/blog/index-type-parent-child-join-now-future-in-elasticsearch)，Elastic 6.x 版只允许每个 Index 包含一个 Type，7.x 版将会彻底移除 Type。

---

## 4.通过curl命令访问ElasticSearch

> Windows里面也可以使用curl命令，虽然也可以直接在浏览器里面输入路径来操作，浏览器的地址框应该是默认为GET请求，无法使用其他的PUT，DELETE请求。

### 4.1  新建Index

```bash
curl -X PUT "http://localhost:9200/weather"
```

​	可以新建一个名称为```weather```的Index，如果创建成功，服务器会返回如下的JSON数据

```JSON
{
  "acknowledged":true,
  "shards_acknowledged":true
}
```

### 4.2 删除Index

```bash
curl -X DELETE "http://localhost:9200/weather"
```

​	该命令可以删除名称为weather的Index，如果该Index不存在则会报错，这个错误信息是我删除名为test的Index是产生的。

```Json
{
    "error":{
        "root_cause":[
            {
                "type":"index_not_found_exception",
                "reason":"no such index [test]",
                "resource.type":"index_or_alias",
                "resource.id":"test",
                "index_uuid":"_na_",
                "index":"test"
            }
        ],
        "type":"index_not_found_exception",
        "reason":"no such index [test]",
        "resource.type":"index_or_alias",
        "resource.id":"test",
        "index_uuid":"_na_",
        "index":"test"
    },
    "status":404
}
```

### 4.3 新增Type

​	通过下面这条命令可以在名为accounts的Index内新增一个名为person的Type，person有三个field，分别为user，类型为text并且使用ik分词器进行索引，title和desc亦是如此。

```bash
$ curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}'
```

​	```到这里还有下面的新增记录本彩笔就没办法使用curl命令运行了，总是报错，搞得我有点懵，这边我打算直接找一下在配置文件里直接配置type的方法，我觉得肯定是有的，毕竟开发环境和生产环境都用命令来创建type非常容易出错，也不现实，现在数据库表都是选择字段自动生成了，solr也有配置文件配置结构，而且真的是生成不了，不知道是不是Windows命令行的问题。```

```bash
#这是我格式化输入的命令
curl -X PUT "localhost:9200/accounts" -d '{"mappings": {"person": {"properties": {"user": {"type": "text","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},"title": {"type": "text","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},"desc": {"type": "text","analyzer": "ik_max_word","search_analyzer": "ik_max_word"}}}}}'

#这是返回的错误信息
{"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}

#后来我又百度说加这个参数 -H "Content-Type: application/json"
curl -H "Content-Type: application/json" -X PUT "localhost:9200/accounts" -d '{"mappings": {"person": {"properties": {"user": {"type": "text","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},"title": {"type": "text","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},"desc": {"type": "text","analyzer": "ik_max_word","search_analyzer": "ik_max_word"}}}}}'
#但是还是错误
{"error":{"root_cause":[{"type":"parse_exception","reason":"Failed to parse content to map"}],"type":"parse_exception","reason":"Failed to parse content to map","caused_by":{"type":"json_parse_exception","reason":"Unexpected character ('m' (code 109)): was expecting double-quote to start field name\n at [Source: (org.elasticsearch.common.bytes.AbstractBytesReference$MarkSupportingStreamInputWrapper); line: 1, column: 3]"}},"status":400}

#先就这样吧，我看看能不能直接写配置文件来创建type
```

### 4.4 新增记录（Document）

```bash
$ curl -X PUT 'localhost:9200/accounts/person/1' -d '{user: 张三,title: 程师,desc: 数据库管理}' 
```

​	该命令会向accounts数据库（Index）的person表（Type）中添加一条记录（Document）。该记录的Id为1，既路径最后的1。Id并不一定需要是数字，也可以是字符串。如果不填写的话系统会自动生成Id。

​    这个命令我也没有实际运行，看了4.3就可以用发现，我连Type都没有创建成功，之后我找到可以使用配置文件生成Type的方法了我在尝试一下，不过不知道为什么，-d传json数据总是失败，有点无语，估计我也不会再测试，一会可能就直接去看使用java操作elasticsearch的方法了。

### 4.5 查询记录

#### 	4.5.1 根据id查询记录

​	看了上面两个小结，很明显，这个我也没办法用，就是搬了大佬的博客，希望对你能有所帮助。

​	向`/Index/Type/Id`发出 GET 请求，就可以查看这条记录。

```bash
$ curl 'localhost:9200/accounts/person/1?pretty=true'
```

​	上面代码请求查看`/accounts/person/1`这条记录，URL 的参数`pretty=true`表示以易读的格式返回。

​	返回的数据中，`found`字段表示查询成功，`_source`字段返回原始记录。

```json
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理"
  }
}
```

​	如果 Id 不正确，就查不到数据，`found`字段就是`false`。

```bash
$ curl 'localhost:9200/weather/beijing/abc?pretty=true'

{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "abc",
  "found" : false
}
```



#### 	4.5.2 返回所有记录

​	使用 GET 方法，直接请求`/Index/Type/_search`，就会返回所有记录。

```bash
$ curl 'localhost:9200/accounts/person/_search'

{
  "took":2,
  "timed_out":false,
  "_shards":{"total":5,"successful":5,"failed":0},
  "hits":{
    "total":2,
    "max_score":1.0,
    "hits":[
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"AV3qGfrC6jMbsbXb6k1p",
        "_score":1.0,
        "_source": {
          "user": "李四",
          "title": "工程师",
          "desc": "系统管理"
        }
      },
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"1",
        "_score":1.0,
        "_source": {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理，软件开发"
        }
      }
    ]
  }
}
```

​	上面代码中，返回结果的 `took`字段表示该操作的耗时（单位为毫秒），`timed_out`字段表示是否超时，`hits`字段表示命中的记录，里面子字段的含义如下。

> - `total`：返回记录数，本例是2条。
> - `max_score`：最高的匹配程度，本例是`1.0`。
> - `hits`：返回的记录组成的数组。

​	返回的记录中，每条记录都有一个`_score`字段，表示匹配的程序，默认是按照这个字段降序排列。

​	

### 4.6 删除记录

​	删除记录就是发出 DELETE 请求。

```bash
$ curl -X DELETE 'localhost:9200/accounts/person/1'
```

​	这里先不要删除这条记录，后面还要用到。

### 4.7 更新记录

​	更新记录就是使用 PUT 请求，重新发送一次数据。

```bash
$ curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理，软件开发"
}' 

{
  "_index":"accounts",
  "_type":"person",
  "_id":"1",
  "_version":2,
  "result":"updated",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":false
}
```

​	上面代码中，我们将原始数据从"数据库管理"改成"数据库管理，软件开发"。 返回结果里面，有几个字段发生了变化。

```json
"_version" : 2,
"result" : "updated",
"created" : false
```

​	可以看到，记录的 Id 没变，但是版本（version）从`1`变成`2`，操作类型（result）从`created`变成`updated`，`created`字段变成`false`，因为这次不是新建记录

## 5.全文搜索

​	全文搜索是搜索引擎的一个重要的功能，这里也是先使用curl命令介绍es全文搜索的形式，当然，这些命令我还是没有使用过，全部都是照搬大神的博客，如果使用有什么问题可以大家一起讨论。

### 	5.1 全文搜索命令结构

​	Elastic 的查询非常特别，使用自己的[查询语法](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl.html)，要求 GET 请求带有数据体。

> ```bash
> $ curl 'localhost:9200/accounts/person/_search'  -d '
> {
>   "query" : { "match" : { "desc" : "软件" }}
> }'
> ```

上面代码使用 [Match 查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-match-query.html)，指定的匹配条件是`desc`字段里面包含"软件"这个词。返回结果如下。

> ```javascript
> {
>   "took":3,
>   "timed_out":false,
>   "_shards":{"total":5,"successful":5,"failed":0},
>   "hits":{
>     "total":1,
>     "max_score":0.28582606,
>     "hits":[
>       {
>         "_index":"accounts",
>         "_type":"person",
>         "_id":"1",
>         "_score":0.28582606,
>         "_source": {
>           "user" : "张三",
>           "title" : "工程师",
>           "desc" : "数据库管理，软件开发"
>         }
>       }
>     ]
>   }
> }
> ```

Elastic 默认一次返回10条结果，可以通过`size`字段改变这个设置。

> ```bash
> $ curl 'localhost:9200/accounts/person/_search'  -d '
> {
>   "query" : { "match" : { "desc" : "管理" }},
>   "size": 1
> }'
> ```

上面代码指定，每次只返回一条结果。

还可以通过`from`字段，指定位移。

> ```bash
> $ curl 'localhost:9200/accounts/person/_search'  -d '
> {
>   "query" : { "match" : { "desc" : "管理" }},
>   "from": 1,
>   "size": 1
> }'
> ```

上面代码指定，从位置1开始（默认是从位置0开始），只返回一条结果。

### 5.2 逻辑运算

如果有多个搜索关键字， Elastic 认为它们是`or`关系。

> ```bash
> $ curl 'localhost:9200/accounts/person/_search'  -d '
> {
>   "query" : { "match" : { "desc" : "软件 系统" }}
> }'
> ```

上面代码搜索的是`软件 or 系统`。

如果要执行多个关键词的`and`搜索，必须使用[布尔查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-bool-query.html)。

> ```bash
> $ curl 'localhost:9200/accounts/person/_search'  -d '
> {
>   "query": {
>     "bool": {
>       "must": [
>         { "match": { "desc": "软件" } },
>         { "match": { "desc": "系统" } }
>       ]
>     }
>   }
> }'
> ```
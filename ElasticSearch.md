### 一、基本认识

### 1.1 和ES的基本交互

```
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        24,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}

GET /megacorp/employee/1
DELETE /megacorp/employee/1
```




####1.1.1 轻量搜索

```
GET /megacorp/employee/_search


GET /megacorp/employee/_search?q=last_name:Smith
```



#### 1.1.2 使用表达式查询

```
GET /megacorp/employee/_search
{
  "query":{
    "match":{
      "last_name": "Smith"
    }
  }
}
```



#### 1.1.3 更复杂的搜索

```
GET /megacorp/employee/_search
{
  "query":{
    "bool":{
      "must":{
        "match":{
          "last_name": "smith"
        }
      },
      "filter":{
        "range":{
          "age":{"gt":30}
        }
      }
    }
  }
}
```



### 1.2 全文搜索 （一项传统数据库确实很难搞定的任务）

#### 1.2.1 相关性概念

```
GET /megacorp/employee/_search
{
  "query":{
    "match":{
      "about":"rock climbing"
    }
  }
}
```




#### 1.2.2 短语搜索

```
GET /megacorp/employee/_search
{
  "query":{
    "match_phrase":{
      "about":"rock climbing"
    }
  }
}
```



### 1.3 高亮搜索

```
GET /megacorp/employee/_search
{
  "query":{
    "match_phrase": {
      "about": "rock climbing"
    }
  },
  "highlight":{
    "fields":{
      "about":{}
    }
  }
}
```



###1.4 分析

#### 1.4.1聚合

```
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```





## 二、分布式特性

ElaticSearch 尽可能屏蔽了分布式系统的复杂性。

* 集群内的原理
  * 集群扩容
  * 故障转移
* 分布式文档存储
* 分布式文档检索
* 分区（shard）
* 内部分片结构



## 三 集群内的原理



目标： 如何按需配置集群、节点和分片，并在硬件故障时保证数据的安全。

### 3.1 空集群

### 3.2 集群健康

```
GET /_cluster/health

{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 9,
  "active_shards" : 9,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 3,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 75.0
}


status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：

green
所有的主分片和副本分片都正常运行。
yellow
所有的主分片都正常运行，但不是所有的副本分片都正常运行。
red
有主分片没能正常运行。

```

### 3.3 添加故障转移

### 3.4 水平扩容

### 3.5 应对故障

## 四、数据输入和输出

无论我们写什么样的程序，目的都是一样的：以某种方式组织数据服务我们的目的。 但是数据不仅仅由随机位和字节组成。我们建立数据元素之间的关系以便于表示实体，或者现实世界中存在的 *事物* 。 如果我们知道一个名字和电子邮件地址属于同一个人，那么它们将会更有意义。

尽管在现实世界中，不是所有的类型相同的实体看起来都是一样的。 一个人可能有一个家庭电话号码，而另一个人只有一个手机号码，再一个人可能两者兼有。 一个人可能有三个电子邮件地址，而另一个人却一个都没有。一位西班牙人可能有两个姓，而讲英语的人可能只有一个姓。

面向对象编程语言如此流行的原因之一是对象帮我们表示和处理现实世界具有潜在的复杂的数据结构的实体，到目前为止，一切都很完美！

但是当我们需要存储这些实体时问题来了，传统上，我们以行和列的形式存储数据到关系型数据库中，相当于使用电子表格。 正因为我们使用了这种不灵活的存储媒介导致所有我们使用对象的灵活性都丢失了。

但是否我们可以将我们的对象按对象的方式来存储？这样我们就能更加专注于 *使用* 数据，而不是在电子表格的局限性下对我们的应用建模。 我们可以重新利用对象的灵活性。

一个 *对象* 是基于特定语言的内存的数据结构。为了通过网络发送或者存储它，我们需要将它表示成某种标准的格式。 [JSON](http://en.wikipedia.org/wiki/Json) 是一种以人可读的文本表示对象的方法。 它已经变成 NoSQL 世界交换数据的事实标准。当一个对象被序列化成为 JSON，它被称为一个 *JSON 文档* 。

Elastcisearch 是分布式的 *文档* 存储。它能存储和检索复杂的数据结构—序列化成为JSON文档—以 *实时* 的方式。 换句话说，一旦一个文档被存储在 Elasticsearch 中，它就是可以被集群中的任意节点检索到。

当然，我们不仅要存储数据，我们一定还需要查询它，成批且快速的查询它们。 尽管现存的 NoSQL 解决方案允许我们以文档的形式存储对象，但是他们仍旧需要我们思考如何查询我们的数据，以及确定哪些字段需要被索引以加快数据检索。

在 Elasticsearch 中， *每个字段的所有数据* 都是 *默认被索引的* 。 即每个字段都有为了快速检索设置的专用倒排索引。而且，不像其他多数的数据库，它能在 *同一个查询中* 使用所有这些倒排索引，并以惊人的速度返回结果。

在本章中，我们展示了用来创建，检索，更新和删除文档的 API。就目前而言，我们不关心文档中的数据或者怎样查询它们。 所有我们关心的就是在 Elasticsearch 中怎样安全的存储文档，以及如何将文档再次返回。

### 4.1 什么是文档 



在大多数应用中，多数实体或对象可以被序列化为包含键值对的 JSON 对象。 一个 *键* 可以是一个字段或字段的名称，一个 *值* 可以是一个字符串，一个数字，一个布尔值， 另一个对象，一些数组值，或一些其它特殊类型诸如表示日期的字符串，或代表一个地理位置的对象：

```js
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```



通常情况下，我们使用的术语 *对象* 和 *文档* 是可以互相替换的。不过，有一个区别： 一个对象仅仅是类似于 hash 、 hashmap 、字典或者关联数组的 JSON 对象，对象中也可以嵌套其他的对象。 对象可能包含了另外一些对象。在 Elasticsearch 中，术语 *文档* 有着特定的含义。它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

字段的名字可以是任何合法的字符串，但 *不可以* 包含英文句号(.)。

#### 4.1.1 文档元数据

一个文档不仅仅包含它的数据 ，也包含 *元数据* —— *有关* 文档的信息。 三个必须的元数据元素如下：

- **`_index`**

  **文档在哪存放**

  一个 *索引* 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 `products` 中，而存储所有销售的交易到索引 `sales` 中。 虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。

  实际上，在 Elasticsearch 中，我们的数据是被存储和索引在 *分片* 中，而一个索引仅仅是逻辑上的命名空间， 这个命名空间由一个或者多个分片组合在一起。 然而，这是一个内部细节，我们的应用程序根本不应该关心分片，对于应用程序而言，只需知道文档位于一个 *索引* 内。 Elasticsearch 会处理所有的细节。

  我们将在 [*索引管理*](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/index-management.html) 介绍如何自行创建和管理索引，但现在我们将让 Elasticsearch 帮我们创建索引。 所有需要我们做的就是选择一个索引名，这个名字必须小写，不能以下划线开头，不能包含逗号。我们用 `website` 作为索引名举例。

- **`_type`**

  **文档表示的对象类别**

  数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。 例如，所有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" 和 "lawn-care"。

  这些文档共享一种相同的（或非常相似）的模式：他们有一个标题、描述、产品代码和价格。他们只是正好属于“产品”下的一些子类。

  Elasticsearch 公开了一个称为 *types* （类型）的特性，它允许您在索引中对数据进行逻辑分区。不同 types 的文档可能有不同的字段，但最好能够非常相似。 我们将在 [类型和映射](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/mapping.html) 中更多的讨论关于 types 的一些应用和限制。

  一个 `_type` 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符. 我们使用 `blog` 作为类型名举例。

- **`_id`**

  **文档唯一标识**

  *ID* 是一个字符串，当它和 `_index` 以及 `_type` 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 `_id` ，要么让 Elasticsearch 帮你生成。

#### 4.1.2 其他元数据

还有一些其他的元数据元素，他们在 [类型和映射](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/mapping.html) 进行了介绍。通过前面已经列出的元数据元素， 我们已经能存储文档到 Elasticsearch 中并通过 ID 检索它—换句话说，使用 Elasticsearch 作为文档的存储介质。

### 4.2 索引文档

通过使用 `index` API ，文档可以被 *索引* —— 存储和使文档可被搜索。 但是首先，我们要确定文档的位置。正如我们刚刚讨论的，一个文档的 `_index` 、 `_type` 和 `_id` 唯一标识一个文档。 我们可以提供自定义的 `_id` 值，或者让 `index` API 自动生成。

#### 4.2.1 使用自定义的ID

```json
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

```
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/02"
}
```

返回

```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

```



⚠️ 在 Elasticsearch 中每个文档都有一个版本号。当每次对文档进行修改时（包括删除）， _version 的值会递增。 在 处理冲突 中，我们讨论了怎样使用 _version 号码确保你的应用程序中的一部分修改不会覆盖另一部分所做的修改。

#### 4.2.2 Auto generating IDs

```
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

返回

```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "PHdBvXcBTZbNVD8_UDqx",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

```

### 4.3 取回一个文档

#### 4.3.1 获取一个文档

```
GET /website/blog/123?pretty
```

返回

```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "My first blog entry",
    "text" : "Just trying this out...",
    "date" : "2014/01/02"
  }
}
```

注意：在请求的查询串参数中加上 `pretty` 参数，正如前面的例子中看到的，这将会调用 Elasticsearch 的 *pretty-print* 功能，该功能 使得 JSON 响应体更加可读。但是， `_source` 字段不能被格式化打印出来。相反，我们得到的 `_source` 字段中的 JSON 串，刚好是和我们传给它的一样。



#### 4.3.2 获取文档的一部分

```
GET /website/blog/123?_source=title,text
```

返回

```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "text" : "Just trying this out...",
    "title" : "My first blog entry"
  }
}
```

#### 4.3.3 检查文档是否存在

如果只想检查一个文档是否存在--根本不想关心内容—那么用 `HEAD` 方法来代替 `GET` 方法。 `HEAD` 请求没有返回体，只返回一个 HTTP 请求报头：

```
curl --HEAD http://localhost:9200/website/_doc/123
```

如果存在则返回

```
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 214
```

如果不存在则返回

```
HTTP/1.1 404 Not Found
content-type: application/json; charset=UTF-8
content-length: 61
```

当然，一个文档仅仅是在检查的时候不存在，并不意味着一毫秒之后它也不存在：也许同时正好另一个进程就创建了该文档。

#### 4.3.4 更新整个文档

在 Elasticsearch 中文档是 *不可改变* 的，不能修改它们。相反，如果想要更新现有的文档，需要 *重建索引* 或者进行替换， 我们可以使用相同的 `index` API 进行实现

```
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```

在响应体中，我们能看到 Elasticsearch 已经增加了 `_version` 字段值：

```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 5,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}
```

在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 尽管你不能再对旧版本的文档进行访问，但它并不会立即消失。当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。

在本章的后面部分，我们会介绍 `update` API, 这个 API 可以用于 [partial updates to a document](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/partial-updates.html) 。 虽然它似乎对文档直接进行了修改，但实际上 Elasticsearch 按前述完全相同方式执行以下过程：

1. 从旧文档构建 JSON
2. 更改该 JSON
3. 删除旧文档
4. 索引一个新文档

唯一的区别在于, `update` API 仅仅通过一个客户端请求来实现这些步骤，而不需要单独的 `get` 和 `index` 请求。

### 4.4  创建新文档

当我们索引一个文档，怎么确认我们正在创建一个完全新的文档，而不是覆盖现有的呢？

请记住， `_index` 、 `_type` 和 `_id` 的组合可以唯一标识一个文档。所以，确保创建一个新文档的最简单办法是，使用索引请求的 `POST` 形式让 Elasticsearch 自动生成唯一 `_id` :

```js
POST /website/blog/
{ ... }
```



然而，如果已经有自己的 `_id` ，那么我们必须告诉 Elasticsearch ，只有在相同的 `_index` 、 `_type` 和 `_id` 不存在时才接受我们的索引请求。这里有两种方式，他们做的实际是相同的事情。使用哪种，取决于哪种使用起来更方便。

第一种方法使用 `op_type` 查询-字符串参数：

```js
PUT /website/blog/123?op_type=create
{ ... }
```



第二种方法是在 URL 末端使用 `/_create` :

```js
PUT /website/blog/123/_create
{ ... }
```



如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 `201 Created` 的 HTTP 响应码。

另一方面，如果具有相同的 `_index` 、 `_type` 和 `_id` 的文档已经存在，Elasticsearch 将会返回 `409 Conflict` 响应码，以及如下的错误信息：

```sense
{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```



### 4.5 删除文档

删除文档的语法和我们所知道的规则相同，只是使用 `DELETE` 方法：

```
DELETE /website/blog/123
```

如果找到该文档，Elasticsearch 将要返回一个 `200 ok` 的 HTTP 响应码，和一个类似以下结构的响应体。注意，字段 `_version` 值已经增加:

```js
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```

如果文档没有找到，我们将得到 `404 Not Found` 的响应码和类似这样的响应体：

```js
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```

即使文档不存在（ `Found` 是 `false` ）， `_version` 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。

正如已经在[更新整个文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/update-doc.html)中提到的，删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。

### 4.6 处理冲突

当我们使用 `index` API 更新文档 ，可以一次性读取原始文档，做我们的修改，然后重新索引 *整个文档* 。 最近的索引请求将获胜：无论最后哪一个文档被索引，都将被唯一存储在 Elasticsearch 中。如果其他人同时更改这个文档，他们的更改将丢失。

很多时候这是没有问题的。也许我们的主数据存储是一个关系型数据库，我们只是将数据复制到 Elasticsearch 中并使其可被搜索。 也许两个人同时更改相同的文档的几率很小。或者对于我们的业务来说偶尔丢失更改并不是很严重的问题。

但有时丢失了一个变更就是 *非常严重的* 。试想我们使用 Elasticsearch 存储我们网上商城商品库存的数量， 每次我们卖一个商品的时候，我们在 Elasticsearch 中将库存数量减少。

有一天，管理层决定做一次促销。突然地，我们一秒要卖好几个商品。 假设有两个 web 程序并行运行，每一个都同时处理所有商品的销售，如图 [Figure 7, “Consequence of no concurrency control”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/version-control.html#img-data-lww) 所示。

![Consequence of no concurrency control](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0301.png)

Figure 7. Consequence of no concurrency control

`web_1` 对 `stock_count` 所做的更改已经丢失，因为 `web_2` 不知道它的 `stock_count` 的拷贝已经过期。 结果我们会认为有超过商品的实际数量的库存，因为卖给顾客的库存商品并不存在，我们将让他们非常失望。

变更越频繁，读数据和更新数据的间隙越长，也就越可能丢失变更。

在数据库领域中，有两种方法通常被用来确保并发更新时变更不会丢失：

- **悲观并发控制**

  这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。

- **乐观并发控制**

  Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

  Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并且到达目的地时也许 *顺序是乱的* 。 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。

  当我们之前讨论 `index` ， `GET` 和 `delete` 请求时，我们指出每个文档都有一个 `_version` （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 `_version` 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

  我们可以利用 `_version` 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 `version` 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。

  让我们创建一个新的博客文章：

  ```sense
  PUT /website/blog/1/_create
  {
    "title": "My first blog entry",
    "text":  "Just trying this out..."
  }
  ```

  拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/40_Concurrency.json) 

  响应体告诉我们，这个新创建的文档 `_version` 版本号是 `1` 。现在假设我们想编辑这个文档：我们加载其数据到 web 表单中， 做一些修改，然后保存新的版本。

  首先我们检索文档:

  ```sense
  GET /website/blog/1
  ```

  拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/40_Concurrency.json) 

  响应体包含相同的 `_version` 版本号 `1` ：

  ```js
  {
    "_index" :   "website",
    "_type" :    "blog",
    "_id" :      "1",
    "_version" : 1,
    "found" :    true,
    "_source" :  {
        "title": "My first blog entry",
        "text":  "Just trying this out..."
    }
  }
  ```

  现在，当我们尝试通过重建文档的索引来保存修改，我们指定 `version` 为我们的修改会被应用的版本：

  ```sense
  PUT /website/blog/1?version=1 
  {
    "title": "My first blog entry",
    "text":  "Starting to get the hang of this..."
  }
  ```

  拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/40_Concurrency.json) 

  |      | 我们想这个在我们索引中的文档只有现在的 `_version` 为 `1` 时，本次更新才能成功。 |
  | ---- | ------------------------------------------------------------ |
  |      |                                                              |

  此请求成功，并且响应体告诉我们 `_version` 已经递增到 `2` ：

  ```sense
  {
    "_index":   "website",
    "_type":    "blog",
    "_id":      "1",
    "_version": 2
    "created":  false
  }
  ```

  拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/40_Concurrency.json) 

  然而，如果我们重新运行相同的索引请求，仍然指定 `version=1` ， Elasticsearch 返回 `409 Conflict` HTTP 响应码，和一个如下所示的响应体：

  ```sense
  {
     "error": {
        "root_cause": [
           {
              "type": "version_conflict_engine_exception",
              "reason": "[blog][1]: version conflict, current [2], provided [1]",
              "index": "website",
              "shard": "3"
           }
        ],
        "type": "version_conflict_engine_exception",
        "reason": "[blog][1]: version conflict, current [2], provided [1]",
        "index": "website",
        "shard": "3"
     },
     "status": 409
  }
  ```

  拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/40_Concurrency.json) 

  这告诉我们在 Elasticsearch 中这个文档的当前 `_version` 号是 `2` ，但我们指定的更新版本号为 `1` 。

  我们现在怎么做取决于我们的应用需求。我们可以告诉用户说其他人已经修改了文档，并且在再次保存之前检查这些修改内容。 或者，在之前的商品 `stock_count` 场景，我们可以获取到最新的文档并尝试重新应用这些修改。

  所有文档的更新或删除 API，都可以接受 `version` 参数，这允许你在代码中使用乐观的并发控制，这是一种明智的做法。

  ### 通过外部系统使用版本控制

  一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch ，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。

  如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 `timestamp` — 那么你就可以在 Elasticsearch 中通过增加 `version_type=external` 到查询字符串的方式重用这些相同的版本号， 版本号必须是大于零的整数， 且小于 `9.2E+18` — 一个 Java 中 `long` 类型的正值。

  外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， Elasticsearch 不是检查当前 `_version` 和请求中指定的版本号是否相同， 而是检查当前 `_version` 是否 *小于* 指定的版本号。 如果请求成功，外部的版本号作为文档的新 `_version` 进行存储。

  外部版本号不仅在索引和删除请求是可以指定，而且在 *创建* 新文档时也可以指定。

  例如，要创建一个新的具有外部版本号 `5` 的博客文章，我们可以按以下方法进行：

  ```sense
  PUT /website/blog/2?version=5&version_type=external
  {
    "title": "My first external blog entry",
    "text":  "Starting to get the hang of this..."
  }
  ```

  在响应中，我们能看到当前的 `_version` 版本号是 `5` ：

  ```js
  {
    "_index":   "website",
    "_type":    "blog",
    "_id":      "2",
    "_version": 5,
    "created":  true
  }
  ```

  现在我们更新这个文档，指定一个新的 `version` 号是 `10` ：

  ```sense
  PUT /website/blog/2?version=10&version_type=external
  {
    "title": "My first external blog entry",
    "text":  "This is a piece of cake..."
  }
  ```

  

  请求成功并将当前 `_version` 设为 `10` ：

  ```js
  {
    "_index":   "website",
    "_type":    "blog",
    "_id":      "2",
    "_version": 10,
    "created":  false
  }
  ```

  如果你要重新运行此请求时，它将会失败，并返回像我们之前看到的同样的冲突错误， 因为指定的外部版本号不大于 Elasticsearch 的当前版本号。

### 4.7文档部分更新

在 [更新整个文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/update-doc.html) , 我们已经介绍过 更新一个文档的方法是检索并修改它，然后重新索引整个文档，这的确如此。然而，使用 `update` API 我们还可以部分更新文档，例如在某个请求时对计数器进行累加。

我们也介绍过文档是不可变的：他们不能被修改，只能被替换。 `update` API 必须遵循同样的规则。 从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， `update` API 简单使用与之前描述相同的 *检索-修改-重建索引* 的处理过程。 区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。通过减少检索和重建索引步骤之间的时间，我们也减少了其他进程的变更带来冲突的可能性。

`update` 请求最简单的一种形式是接收文档的一部分作为 `doc` 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。 例如，我们增加字段 `tags` 和 `views` 到我们的博客文章，如下所示：

```sense
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/45_Partial_update.json) 

如果请求成功，我们看到类似于 `index` 请求的响应：

```js
{
   "_index" :   "website",
   "_id" :      "1",
   "_type" :    "blog",
   "_version" : 3
}
```

检索文档显示了更新后的 `_source` 字段：

```sense
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  3,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags": [ "testing" ], 
      "views":  0 
   }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/45_Partial_update.json) 

|      | 新的字段已被添加到 `_source` 中。 |
| ---- | --------------------------------- |
|      |                                   |

#### 4.7.1 使用脚本部分更新文档

脚本可以在 `update` API中用来改变 `_source` 的字段内容， 它在更新脚本中称为 `ctx._source` 。 例如，我们可以使用脚本来增加博客文章中 `views` 的数量：

```sense
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/45_Partial_update.json) 

**用 Groovy 脚本编程**

对于那些 API 不能满足需求的情况，Elasticsearch 允许你使用脚本编写自定义的逻辑。 许多API都支持脚本的使用，包括搜索、排序、聚合和文档更新。 脚本可以作为请求的一部分被传递，从特殊的 .scripts 索引中检索，或者从磁盘加载脚本。

默认的脚本语言 是 [Groovy](http://groovy.codehaus.org/)，一种快速表达的脚本语言，在语法上与 JavaScript 类似。 它在 Elasticsearch V1.3.0 版本首次引入并运行在 *沙盒* 中，然而 Groovy 脚本引擎存在漏洞， 允许攻击者通过构建 Groovy 脚本，在 Elasticsearch Java VM 运行时脱离沙盒并执行 shell 命令。

因此，在版本 v1.3.8 、 1.4.3 和 V1.5.0 及更高的版本中，它已经被默认禁用。 此外，您可以通过设置集群中的所有节点的 `config/elasticsearch.yml` 文件来禁用动态 Groovy 脚本：

```yaml
script.groovy.sandbox.enabled: false
```

这将关闭 Groovy 沙盒，从而防止动态 Groovy 脚本作为请求的一部分被接受， 或者从特殊的 `.scripts` 索引中被检索。当然，你仍然可以使用存储在每个节点的 `config/scripts/` 目录下的 Groovy 脚本。

如果你的架构和安全性不需要担心漏洞攻击，例如你的 Elasticsearch 终端仅暴露和提供给可信赖的应用， 当它是你的应用需要的特性时，你可以选择重新启用动态脚本。

你可以在 [scripting reference documentation](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/modules-scripting.html) 获取更多关于脚本的资料。

我们也可以通过使用脚本给 `tags` 数组添加一个新的标签。在这个例子中，我们指定新的标签作为参数，而不是硬编码到脚本内部。 这使得 Elasticsearch 可以重用这个脚本，而不是每次我们想添加标签时都要对新脚本重新编译：

```sense
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/45_Partial_update.json) 

获取文档并显示最后两次请求的效果：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  5,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags":  ["testing", "search"], 
      "views":  1 
   }
}
```

|      | `search` 标签已追加到 `tags` 数组中。 |
| ---- | ------------------------------------- |
|      | `views` 字段已递增。                  |

我们甚至可以选择通过设置 `ctx.op` 为 `delete` 来删除基于其内容的文档：

```sense
POST /website/blog/1/_update
{
   "script" : "ctx.op = ctx._source.views == count ? 'delete' : 'none'",
    "params" : {
        "count": 1
    }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/45_Partial_update.json) 

#### 4.7.2 更新的文档可能尚不存在

假设我们需要在 Elasticsearch 中存储一个页面访问量计数器。 每当有用户浏览网页，我们对该页面的计数器进行累加。但是，如果它是一个新网页，我们不能确定计数器已经存在。 如果我们尝试更新一个不存在的文档，那么更新操作将会失败。

在这样的情况下，我们可以使用 `upsert` 参数，指定如果文档不存在就应该先创建它：

```sense
POST /website/pageviews/1/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/45_Upsert.json) 

我们第一次运行这个请求时， `upsert` 值作为新文档被索引，初始化 `views` 字段为 `1` 。 在后续的运行中，由于文档已经存在， `script` 更新操作将替代 `upsert` 进行应用，对 `views` 计数器进行累加。

#### 4.7.3更新和冲突

在本节的介绍中，我们说明 *检索* 和 *重建索引* 步骤的间隔越小，变更冲突的机会越小。 但是它并不能完全消除冲突的可能性。 还是有可能在 `update` 设法重新索引之前，来自另一进程的请求修改了文档。

为了避免数据丢失， `update` API 在 *检索* 步骤时检索得到文档当前的 `_version` 号，并传递版本号到 *重建索引* 步骤的 `index` 请求。 如果另一个进程修改了处于检索和重新索引步骤之间的文档，那么 `_version` 号将不匹配，更新请求将会失败。

对于部分更新的很多使用场景，文档已经被改变也没有关系。 例如，如果两个进程都对页面访问量计数器进行递增操作，它们发生的先后顺序其实不太重要； 如果冲突发生了，我们唯一需要做的就是尝试再次更新。

这可以通过设置参数 `retry_on_conflict` 来自动完成， 这个参数规定了失败之前 `update` 应该重试的次数，它的默认值为 `0` 。

```sense
POST /website/pageviews/1/_update?retry_on_conflict=5 
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 0
   }
}
```

失败之前重试该更新5次。

在增量操作无关顺序的场景，例如递增计数器等这个方法十分有效，但是在其他情况下变更的顺序 *是* 非常重要的。 类似 [`index` API](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/index-doc.html) ， `update` API 默认采用 *最终写入生效* 的方案，但它也接受一个 `version` 参数来允许你使用 [optimistic concurrency control](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/optimistic-concurrency-control.html) 指定想要更新文档的版本。



### 4.8 获取多个文档

Elasticsearch 的速度已经很快了，但甚至能更快。 将多个请求合并成一个，避免单独处理每个请求花费的网络延时和开销。 如果你需要从 Elasticsearch 检索很多文档，那么使用 *multi-get* 或者 `mget` API 来将这些检索请求放在一个请求中，将比逐个文档请求更快地检索到全部文档。

`mget` API 要求有一个 `docs` 数组作为参数，每个元素包含需要检索文档的元数据， 包括 `_index` 、 `_type` 和 `_id` 。如果你想检索一个或者多个特定的字段，那么你可以通过 `_source` 参数来指定这些字段的名字：

```sense
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```



该响应体也包含一个 `docs` 数组， 对于每一个在请求中指定的文档，这个数组中都包含有一个对应的响应，且顺序与请求中的顺序相同。 其中的每一个响应都和使用单个 [`get` request](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/get-doc.html) 请求所得到的响应体相同：

```sense
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```



如果想检索的数据都在相同的 `_index` 中（甚至相同的 `_type` 中），则可以在 URL 中指定默认的 `/_index` 或者默认的 `/_index/_type` 。

你仍然可以通过单独请求覆盖这些值：

```sense
GET /website/blog/_mget
{
   "docs" : [
      { "_id" : 2 },
      { "_type" : "pageviews", "_id" :   1 }
   ]
}
```



事实上，如果所有文档的 `_index` 和 `_type` 都是相同的，你可以只传一个 `ids` 数组，而不是整个 `docs` 数组：

```js
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```



注意，我们请求的第二个文档是不存在的。我们指定类型为 `blog` ，但是文档 ID `1` 的类型是 `pageviews` ，这个不存在的情况将在响应体中被报告：

```sense
{
  "docs" : [
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "2",
      "_version" : 10,
      "found" :    true,
      "_source" : {
        "title":   "My first external blog entry",
        "text":    "This is a piece of cake..."
      }
    },
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "found" :    false  
    }
  ]
}
```



未找到该文档。

事实上第二个文档未能找到并不妨碍第一个文档被检索到。每个文档都是单独检索和报告的。

即使有某个文档没有找到，上述请求的 HTTP 状态码仍然是 `200` 。事实上，即使请求 *没有* 找到任何文档，它的状态码依然是 `200` --因为 `mget` 请求本身已经成功执行。 为了确定某个文档查找是成功或者失败，你需要检查 `found` 标记。

### 4.9 代价较小的批量操作

与 `mget` 可以使我们一次取回多个文档同样的方式， `bulk` API 允许在单个步骤中进行多次 `create` 、 `index` 、 `update` 或 `delete` 请求。 如果你需要索引一个数据流比如日志事件，它可以排队和索引数百或数千批次。

`bulk` 与其他的请求体格式稍有不同，如下所示：

```js
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```



这种格式类似一个有效的单行 JSON 文档 *流* ，它通过换行符(`\n`)连接到一起。注意两个要点：

- 每行一定要以换行符(`\n`)结尾， *包括最后一行* 。这些换行符被用作一个标记，可以有效分隔行。
- 这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个 JSON *不* 能使用 pretty 参数打印。

在 [为什么是有趣的格式？](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/distrib-multi-doc.html#bulk-format) 中， 我们解释为什么 `bulk` API 使用这种格式。

`action/metadata` 行指定 *哪一个文档* 做 *什么操作* 。

`action` 必须是以下选项之一:

- **`create`**

  如果文档不存在，那么就创建它。详情请见 [创建新文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/create-doc.html)。

- **`index`**

  创建一个新文档或者替换一个现有的文档。详情请见 [索引文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/index-doc.html) 和 [更新整个文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/update-doc.html)。

- **`update`**

  部分更新一个文档。详情请见 [文档的部分更新](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/partial-updates.html)。

- **`delete`**

  删除一个文档。详情请见 [删除文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/delete-doc.html)。

`metadata` 应该指定被索引、创建、更新或者删除的文档的 `_index` 、 `_type` 和 `_id` 。

例如，一个 `delete` 请求看起来是这样的：

```js
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```



`request body` 行由文档的 `_source` 本身组成—文档包含的字段和值。它是 `index` 和 `create` 操作所必需的，这是有道理的：你必须提供文档以索引。

它也是 `update` 操作所必需的，并且应该包含你传递给 `update` API 的相同请求体： `doc` 、 `upsert` 、 `script` 等等。 删除操作不需要 `request body` 行。

```js
{ "create":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
```



如果不指定 `_id` ，将会自动生成一个 ID ：

```js
{ "index": { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
```



为了把所有的操作组合在一起，一个完整的 `bulk` 请求 有以下形式:

```sense
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } 
```

|      | 请注意 `delete` 动作不能有请求体,它后面跟着的是另外一个操作。 |
| ---- | ------------------------------------------------------------ |
|      | 谨记最后一个换行符不要落下。                                 |

这个 Elasticsearch 响应包含 `items` 数组，这个数组的内容是以请求的顺序列出来的每个请求的结果。

```sense
{
   "took": 4,
   "errors": false, 
   "items": [
      {  "delete": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 2,
            "status":   200,
            "found":    true
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 3,
            "status":   201
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "EiwfApScQiiy7TIKFxRCTw",
            "_version": 1,
            "status":   201
      }},
      {  "update": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 4,
            "status":   200
      }}
   ]
}
```

|      | 所有的子请求都成功完成。 |
| ---- | ------------------------ |
|      |                          |

每个子请求都是独立执行，因此某个子请求的失败不会对其他子请求的成功与否造成影响。 如果其中任何子请求失败，最顶层的 `error` 标志被设置为 `true` ，并且在相应的请求报告出错误明细：

```sense
POST /_bulk
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
```

在响应中，我们看到 `create` 文档 `123` 失败，因为它已经存在。但是随后的 `index` 请求，也是对文档 `123` 操作，就成功了：

```sense
{
   "took": 3,
   "errors": true, 
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, 
            "error":    "DocumentAlreadyExistsException 
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 
      }}
   ]
}
```



|      | 一个或者多个请求失败。                       |
| ---- | -------------------------------------------- |
|      | 这个请求的HTTP状态码报告为 `409 CONFLICT` 。 |
|      | 解释为什么请求失败的错误信息。               |
|      | 第二个请求成功，返回 HTTP 状态码 `200 OK` 。 |

这也意味着 `bulk` 请求不是原子的： 不能用它来实现事务控制。每个请求是单独处理的，因此一个请求的成功或失败不会影响其他的请求。

#### 不要重复指定Index和Type

也许你正在批量索引日志数据到相同的 `index` 和 `type` 中。 但为每一个文档指定相同的元数据是一种浪费。相反，可以像 `mget` API 一样，在 `bulk` 请求的 URL 中接收默认的 `/_index` 或者 `/_index/_type` ：

```sense
POST /website/_bulk
{ "index": { "_type": "log" }}
{ "event": "User logged in" }
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/55_Bulk_defaults.json) 

你仍然可以覆盖元数据行中的 `_index` 和 `_type` , 但是它将使用 URL 中的这些元数据值作为默认值：

```sense
POST /website/log/_bulk
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/030_Data/55_Bulk_defaults.json) 

#### 多大是太大了？

整个批量请求都需要由接收到请求的节点加载到内存中，因此该请求越大，其他请求所能获得的内存就越少。 批量请求的大小有一个最佳值，大于这个值，性能将不再提升，甚至会下降。 但是最佳值不是一个固定的值。它完全取决于硬件、文档的大小和复杂度、索引和搜索的负载的整体情况。

幸运的是，很容易找到这个 *最佳点* ：通过批量索引典型文档，并不断增加批量大小进行尝试。 当性能开始下降，那么你的批量大小就太大了。一个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 如果你的文档非常大，那么就减少批量的文档个数。

密切关注你的批量请求的物理大小往往非常有用，一千个 1KB 的文档是完全不同于一千个 1MB 文档所占的物理大小。 一个好的批量大小在开始处理后所占用的物理大小约为 5-15 MB。



## 五、分布式文档存储



在前面的章节，我们介绍了如何索引和查询数据，不过我们忽略了很多底层的技术细节， 例如文件是如何分布到集群的，又是如何从集群中获取的。 Elasticsearch 本意就是隐藏这些底层细节，让我们好专注在业务开发中，所以其实你不必了解这么深入也无妨。

在这个章节中，我们将深入探索这些核心的技术细节，这能帮助你更好地理解数据如何被存储到这个分布式系统中。

**注意**

这个章节包含了一些高级话题，上面也提到过，就算你不记住和理解所有的细节仍然能正常使用 Elasticsearch。 如果你有兴趣的话，这个章节可以作为你的课外兴趣读物，扩展你的知识面。

如果你在阅读这个章节的时候感到很吃力，也不用担心。 这个章节仅仅只是用来告诉你 Elasticsearch 是如何工作的， 将来在工作中如果你需要用到这个章节提供的知识，可以再回过头来翻阅。

### 5.1 路由一个文件到一个分片中

当索引一个文档的时候，文档会被存储到一个主分片中。 Elasticsearch 如何知道一个文档应该存放到哪个分片中呢？当我们创建文档时，它如何决定这个文档应当被存储在分片 `1` 还是分片 `2` 中呢？

首先这肯定不会是随机的，否则将来要获取文档的时候我们就不知道从何处寻找了。实际上，这个过程是根据下面这个公式决定的：

```
shard = hash(routing) % number_of_primary_shards
```

`routing` 是一个可变值，默认是文档的 `_id` ，也可以设置成一个自定义的值。 `routing` 通过 hash 函数生成一个数字，然后这个数字再除以 `number_of_primary_shards` （主分片的数量）后得到 **余数** 。这个分布在 `0` 到 `number_of_primary_shards-1` 之间的余数，就是我们所寻求的文档所在分片的位置。

这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

你可能觉得由于 Elasticsearch 主分片数量是固定的会使索引难以进行扩容。实际上当你需要时有很多技巧可以轻松实现扩容。我们将会在[*扩容设计*](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/scale.html)一章中提到更多有关水平扩展的内容。

所有的文档 API（ `get` 、 `index` 、 `delete` 、 `bulk` 、 `update` 以及 `mget` ）都接受一个叫做 `routing` 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。我们也会在[*扩容设计*](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/scale.html)这一章中详细讨论为什么会有这样一种需求。



### 5.2 主分片和副分片如何交互

为了说明目的, 我们 假设有一个集群由三个节点组成。 它包含一个叫 `blogs` 的索引，有两个主分片，每个主分片有两个副本分片。相同分片的副本不会放在同一节点，所以我们的集群看起来像 [Figure 8, “有三个节点和一个索引的集群”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/how-primary-and-replica-shards-interact.html#img-distrib)。

![有三个节点和一个索引的集群](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0401.png)

Figure 8. 有三个节点和一个索引的集群

我们可以发送请求到集群中的任一节点。 每个节点都有能力处理任意请求。 每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。 在下面的例子中，将所有的请求发送到 `Node 1` ，我们将其称为 *协调节点(coordinating node)* 。

当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点。

### 5.3 新建、索引和删除文档

新建、索引和删除 请求都是 *写* 操作， 必须在主分片上面完成之后才能被复制到相关的副本分片，如下图所示 [Figure 9, “新建、索引和删除单个文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/distrib-write.html#img-distrib-write).

![新建、索引和删除单个文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0402.png)

Figure 9. 新建、索引和删除单个文档

以下是在主副分片和任何副本分片上面 成功新建，索引和删除文档所需要的步骤顺序：

1. 客户端向 `Node 1` 发送新建、索引或者删除请求。
2. 节点使用文档的 `_id` 确定文档属于分片 0 。请求会被转发到 `Node 3`，因为分片 0 的主分片目前被分配在 `Node 3` 上。
3. `Node 3` 在主分片上面执行请求。如果成功了，它将请求并行转发到 `Node 1` 和 `Node 2` 的副本分片上。一旦所有的副本分片都报告成功, `Node 3` 将向协调节点报告成功，协调节点向客户端报告成功。

在客户端收到成功响应时，文档变更已经在主分片和所有副本分片执行完成，变更是安全的。

有一些可选的请求参数允许您影响这个过程，可能以数据安全为代价提升性能。这些选项很少使用，因为Elasticsearch已经很快，但是为了完整起见，在这里阐述如下：

- **`consistency`**

  consistency，即一致性。在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求 必须要有 *规定数量(quorum)*（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行_写_操作(其中分片副本可以是主分片或者副本分片)。这是为了避免在发生网络分区故障（network partition）的时候进行_写_操作，进而导致数据不一致。_规定数量_即：

  `int( (primary + number_of_replicas) / 2 ) + 1`

  `consistency` 参数的值可以设为 `one` （只要主分片状态 ok 就允许执行_写_操作）,`all`（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或 `quorum` 。默认值为 `quorum` , 即大多数的分片副本状态没问题就允许执行_写_操作。注意，*规定数量* 的计算公式中 `number_of_replicas` 指的是在索引设置中的设定副本分片数，而不是指当前处理活动状态的副本分片数。如果你的索引设置中指定了当前索引拥有三个副本分片，那规定数量的计算结果即：

  `int( (primary + 3 replicas) / 2 ) + 1 = 3`

  如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此您将无法索引和删除任何文档。

- **`timeout`**

  如果没有足够的副本分片会发生什么？ Elasticsearch会等待，希望更多的分片出现。默认情况下，它最多等待1分钟。 如果你需要，你可以使用 `timeout` 参数 使它更早终止： `100` 100毫秒，`30s` 是30秒。

新索引默认有 `1` 个副本分片，这意味着为满足 `规定数量` *应该* 需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当 `number_of_replicas` 大于1的时候，规定数量才会执行。

### 5.4  取回一个文档



可以从主分片或者从其它任意副本分片检索文档 ，如下图所示 [Figure 10, “取回单个文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/distrib-read.html#img-distrib-read).

![取回单个文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0403.png)

Figure 10. 取回单个文档

以下是从主分片或者副本分片检索文档的步骤顺序：

1、客户端向 `Node 1` 发送获取请求。

2、节点使用文档的 `_id` 来确定文档属于分片 `0` 。分片 `0` 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 `Node 2` 。

3、`Node 2` 将文档返回给 `Node 1` ，然后将文档返回给客户端。

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。

在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的。

### 5.5 局部更新文档

如 [Figure 11, “局部更新文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/_partial_updates_to_a_document.html#img-distrib-update) 所示，`update` API 结合了先前说明的读取和写入模式。

![局部更新文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0404.png)

Figure 11. 局部更新文档

以下是部分更新一个文档的步骤：

1. 客户端向 `Node 1` 发送更新请求。
2. 它将请求转发到主分片所在的 `Node 3` 。
3. `Node 3` 从主分片检索文档，修改 `_source` 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 `retry_on_conflict` 次后放弃。
4. 如果 `Node 3` 成功地更新文档，它将新版本的文档并行转发到 `Node 1` 和 `Node 2` 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， `Node 3` 向协调节点也返回成功，协调节点向客户端返回成功。

`update` API 还接受在 [新建、索引和删除文档](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/distrib-write.html) 章节中介绍的 `routing` 、 `replication` 、 `consistency` 和 `timeout` 参数。

**基于文档的复制**

当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。 如果Elasticsearch仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

### 5.6.多文档模式

`mget` 和 `bulk` API 的模式类似于单文档模式。区别在于协调节点知道每个文档存在于哪个分片中。 它将整个多文档请求分解成 *每个分片* 的多文档请求，并且将这些请求并行转发到每个参与节点。

协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端，如 [Figure 12, “使用 `mget` 取回多个文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/distrib-multi-doc.html#img-distrib-mget) 所示。

![“使用 `mget` 取回多个文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0405.png)

Figure 12. 使用 `mget` 取回多个文档

以下是使用单个 `mget` 请求取回多个文档所需的步骤顺序：

1. 客户端向 `Node 1` 发送 `mget` 请求。
2. `Node 1` 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复， `Node 1` 构建响应并将其返回给客户端。

可以对 `docs` 数组中每个文档设置 `routing` 参数。

bulk API， 如 [Figure 13, “使用 `bulk` 修改多个文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/distrib-multi-doc.html#img-distrib-bulk) 所示， 允许在单个批量请求中执行多个创建、索引、删除和更新请求。

![“使用 `bulk` 修改多个文档”](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/images/elas_0406.png)

Figure 13. 使用 `bulk` 修改多个文档

`bulk` API 按如下步骤顺序执行：

1. 客户端向 `Node 1` 发送 `bulk` 请求。
2. `Node 1` 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

`bulk` API 还可以在整个批量请求的最顶层使用 `consistency` 参数，以及在每个请求中的元数据中使用 `routing` 参数。

### 为什么是有趣的格式？

当我们早些时候在[代价较小的批量操作](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/bulk.html)章节了解批量请求时，您可能会问自己， "为什么 `bulk` API 需要有换行符的有趣格式，而不是发送包装在 JSON 数组中的请求，例如 `mget` API？" 。

为了回答这一点，我们需要解释一点背景：在批量请求中引用的每个文档可能属于不同的主分片， 每个文档可能被分配给集群中的任何节点。这意味着批量请求 `bulk` 中的每个 *操作* 都需要被转发到正确节点上的正确分片。

如果单个请求被包装在 JSON 数组中，那就意味着我们需要执行以下操作：

- 将 JSON 解析为数组（包括文档数据，可以非常大）
- 查看每个请求以确定应该去哪个分片
- 为每个分片创建一个请求数组
- 将这些数组序列化为内部传输格式
- 将请求发送到每个分片

这是可行的，但需要大量的 RAM 来存储原本相同的数据的副本，并将创建更多的数据结构，Java虚拟机（JVM）将不得不花费时间进行垃圾回收。

相反，Elasticsearch可以直接读取被网络缓冲区接收的原始数据。 它使用换行符字符来识别和解析小的 `action/metadata` 行来决定哪个分片应该处理每个请求。

这些原始请求会被直接转发到正确的分片。没有冗余的数据复制，没有浪费的数据结构。整个请求尽可能在最小的内存中处理。



## 六、 搜索——最基本的工具

现在，我们已经学会了如何使用 Elasticsearch 作为一个简单的 NoSQL 风格的分布式文档存储系统。我们可以将一个 JSON 文档扔到 Elasticsearch 里，然后根据 ID 检索。但 Elasticsearch 真正强大之处在于可以从无规律的数据中找出有意义的信息——从“大数据”到“大信息”。

Elasticsearch 不只会_存储（stores）_ 文档，为了能被搜索到也会为文档添加_索引（indexes）_ ，这也是为什么我们使用结构化的 JSON 文档，而不是无结构的二进制数据。

*文档中的每个字段都将被索引并且可以被查询* 。不仅如此，在简单查询时，Elasticsearch 可以使用 *所有（all）* 这些索引字段，以惊人的速度返回结果。这是你永远不会考虑用传统数据库去做的一些事情。

*搜索（search）* 可以做到：

- 在类似于 `gender` 或者 `age` 这样的字段上使用结构化查询，`join_date` 这样的字段上使用排序，就像SQL的结构化查询一样。
- 全文检索，找出所有匹配关键字的文档并按照_相关性（relevance）_ 排序后返回结果。
- 以上二者兼而有之。

很多搜索都是开箱即用的，为了充分挖掘 Elasticsearch 的潜力，你需要理解以下三个概念：

- ***映射（Mapping）\***

  描述数据在每个字段内如何存储

- ***分析（Analysis）\***

  全文是如何处理使之可以被搜索的

- ***领域特定查询语言（Query DSL）\***

  Elasticsearch 中强大灵活的查询语言

以上提到的每个点都是一个大话题，我们将在 [深入搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/search-in-depth.html) 一章详细阐述它们。本章节我们将介绍这三点的一些基本概念——仅仅帮助你大致了解搜索是如何工作的。

数据：

```
POST /_bulk
{"create":{"_index":"us","_id":"1"}}
{"email":"john@smith.com","name":"John Smith","username":"@john"}
{"create":{"_index":"gb","_id":"2"}}
{"email":"mary@jones.com","name":"Mary Jones","username":"@mary"}
{"create":{"_index":"gb","_id":"3"}}
{"date":"2014-09-13","name":"Mary Jones","tweet":"Elasticsearch means full text search has never been so easy","user_id":2}
{"create":{"_index":"us","_id":"4"}}
{"date":"2014-09-14","name":"John Smith","tweet":"@mary it is not just text, it does everything","user_id":1}
{"create":{"_index":"gb","_id":"5"}}
{"date":"2014-09-15","name":"Mary Jones","tweet":"However did I manage before Elasticsearch?","user_id":2}
{"create":{"_index":"us","_id":"6"}}
{"date":"2014-09-16","name":"John Smith","tweet":"The Elasticsearch API is really easy to use","user_id":1}
{"create":{"_index":"gb","_id":"7"}}
{"date":"2014-09-17","name":"Mary Jones","tweet":"The Query DSL is really powerful and flexible","user_id":2}
{"create":{"_index":"us","_id":"8"}}
{"date":"2014-09-18","name":"John Smith","user_id":1}
{"create":{"_index":"gb","_id":"9"}}
{"date":"2014-09-19","name":"Mary Jones","tweet":"Geo-location aggregations are really cool","user_id":2}
{"create":{"_index":"us","_id":"10"}}
{"date":"2014-09-20","name":"John Smith","tweet":"Elasticsearch surely is one of the hottest new NoSQL products","user_id":1}
{"create":{"_index":"gb","_id":"11"}}
{"date":"2014-09-21","name":"Mary Jones","tweet":"Elasticsearch is built for the cloud, easy to scale","user_id":2}
{"create":{"_index":"us","_id":"12"}}
{"date":"2014-09-22","name":"John Smith","tweet":"Elasticsearch and I have left the honeymoon stage, and I still love her.","user_id":1}
{"create":{"_index":"gb","_id":"13"}}
{"date":"2014-09-23","name":"Mary Jones","tweet":"So yes, I am an Elasticsearch fanboy","user_id":2}
{"create":{"_index":"us","_id":"14"}}
{"date":"2014-09-24","name":"John Smith","tweet":"How many more cheesy tweets do I have to write?","user_id":1}
```

### 6.1 空搜索

搜索API的最基础的形式是没有指定任何查询的空搜索，它简单地返回集群中所有索引下的所有文档：

```sense
GET /_search
```



拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/050_Search/05_Empty_search.json) 

返回的结果（为了界面简洁编辑过的）像这样：

```js
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```



### hits

返回结果中最重要的部分是 `hits` ，它包含 `total` 字段来表示匹配到的文档总数，并且一个 `hits` 数组包含所查询结果的前十个文档。

在 `hits` 数组中每个结果包含文档的 `_index` 、 `_type` 、 `_id` ，加上 `_source` 字段。这意味着我们可以直接从返回的搜索结果中使用整个文档。这不像其他的搜索引擎，仅仅返回文档的ID，需要你单独去获取文档。

每个结果还有一个 `_score` ，它衡量了文档与查询的匹配程度。默认情况下，首先返回最相关的文档结果，就是说，返回的文档是按照 `_score` 降序排列的。在这个例子中，我们没有指定任何查询，故所有的文档具有相同的相关性，因此对所有的结果而言 `1` 是中性的 `_score` 。

`max_score` 值是与查询所匹配文档的 `_score` 的最大值。

### took

`took` 值告诉我们执行整个搜索请求耗费了多少毫秒。

### shards

`_shards` 部分告诉我们在查询中参与分片的总数，以及这些分片成功了多少个失败了多少个。正常情况下我们不希望分片失败，但是分片失败是可能发生的。如果我们遭遇到一种灾难级别的故障，在这个故障中丢失了相同分片的原始数据和副本，那么对这个分片将没有可用副本来对搜索请求作出响应。假若这样，Elasticsearch 将报告这个分片是失败的，但是会继续返回剩余分片的结果。

### timeout

`timed_out` 值告诉我们查询是否超时。默认情况下，搜索请求不会超时。如果低响应时间比完成结果更重要，你可以指定 `timeout` 为 10 或者 10ms（10毫秒），或者 1s（1秒）：

```js
GET /_search?timeout=10ms
```

在请求超时之前，Elasticsearch 将会返回已经成功从每个分片获取的结果。

应当注意的是 `timeout` 不是停止执行查询，它仅仅是告知正在协调的节点返回到目前为止收集的结果并且关闭连接。在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。

使用超时是因为 SLA(服务等级协议)对你是很重要的，而不是因为想去中止长时间运行的查询。

### 6.2  多索引、多类型

你有没有注意到之前的 [empty search](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/empty-search.html) 的结果，不同类型的文档— `user` 和 `tweet` 来自不同的索引— `us` 和 `gb` ？

如果不对某一特殊的索引或者类型做限制，就会搜索集群中的所有文档。Elasticsearch 转发搜索请求到每一个主分片或者副本分片，汇集查询出的前10个结果，并且返回给我们。

然而，经常的情况下，你想在一个或多个特殊的索引并且在一个或者多个特殊的类型中进行搜索。我们可以通过在URL中指定特殊的索引和类型达到这种效果，如下所示：

- **`/_search`**

  在所有的索引中搜索所有的类型

- **`/gb/_search`**

  在 `gb` 索引中搜索所有的类型

- **`/gb,us/_search`**

  在 `gb` 和 `us` 索引中搜索所有的文档

- **`/g\*,u\*/_search`**

  在任何以 `g` 或者 `u` 开头的索引中搜索所有的类型

- **`/gb/user/_search`**

  在 `gb` 索引中搜索 `user` 类型

- **`/gb,us/user,tweet/_search`**

  在 `gb` 和 `us` 索引中搜索 `user` 和 `tweet` 类型

- **`/_all/user,tweet/_search`**

  在所有的索引中搜索 `user` 和 `tweet` 类型

当在单一的索引下进行搜索的时候，Elasticsearch 转发请求到索引的每个分片中，可以是主分片也可以是副本分片，然后从每个分片中收集结果。多索引搜索恰好也是用相同的方式工作的—只是会涉及到更多的分片。

搜索一个索引有五个主分片和搜索五个索引各有一个分片准确来所说是等价的。

接下来，你将明白这种简单的方式如何灵活的根据需求的变化让扩容变得简单。

### 6.3  分页

在之前的 [空搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/empty-search.html) 中说明了集群中有 14 个文档匹配了（empty）query 。 但是在 `hits` 数组中只有 10 个文档。如何才能看到其他的文档？

和 SQL 使用 `LIMIT` 关键字返回单个 `page` 结果的方法相同，Elasticsearch 接受 `from` 和 `size` 参数：

- **`size`**

  显示应该返回的结果数量，默认是 `10`

- **`from`**

  显示应该跳过的初始结果数量，默认是 `0`

如果每页展示 5 条结果，可以用下面方式请求得到 1 到 3 页的结果：

```sense
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

考虑到分页过深以及一次请求太多结果的情况，结果集在返回之前先进行排序。 但请记住一个请求经常跨越多个分片，每个分片都产生自己的排序结果，这些结果需要进行集中排序以保证整体顺序是正确的。

**在分布式系统中深度分页**

理解为什么深度分页是有问题的，我们可以假设在一个有 5 个主分片的索引中搜索。 当我们请求结果的第一页（结果从 1 到 10 ），每一个分片产生前 10 的结果，并且返回给 *协调节点* ，协调节点对 50 个结果排序得到全部结果的前 10 个。

现在假设我们请求第 1000 页—结果从 10001 到 10010 。所有都以相同的方式工作除了每个分片不得不产生前10010个结果以外。 然后协调节点对全部 50050 个结果排序最后丢弃掉这些结果中的 50040 个结果。

可以看到，在分布式系统中，对结果排序的成本随分页的深度成指数上升。这就是 web 搜索引擎对任何查询都不要返回超过 1000 个结果的原因。

### 6.4 轻量搜索

有两种形式的 `搜索` API：一种是 “轻量的” *查询字符串* 版本，要求在查询字符串中传递所有的参数，另一种是更完整的 *请求体* 版本，要求使用 JSON 格式和更丰富的查询表达式作为搜索语言。

查询字符串搜索非常适用于通过命令行做即席查询。例如，查询在 `tweet` 类型中 `tweet` 字段包含 `elasticsearch` 单词的所有文档：

```sense
GET /_all/tweet/_search?q=tweet:elasticsearch
```



拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/050_Search/20_Query_string.json) 

下一个查询在 `name` 字段中包含 `john` 并且在 `tweet` 字段中包含 `mary` 的文档。实际的查询就是这样

```
+name:john +tweet:mary
```

但是查询字符串参数所需要的 *百分比编码* （译者注：URL编码）实际上更加难懂：

```sense
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```



拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/050_Search/20_Query_string.json) 

`+` 前缀表示必须与查询条件匹配。类似地， `-` 前缀表示一定不与查询条件匹配。没有 `+` 或者 `-` 的所有其他条件都是可选的——匹配的越多，文档就越相关。

#### 6.4.1 _all 字段

这个简单搜索返回包含 `mary` 的所有文档：

```sense
GET /_search?q=mary
```

之前的例子中，我们在 `tweet` 和 `name` 字段中搜索内容。然而，这个查询的结果在三个地方提到了 `mary` ：

- 有一个用户叫做 Mary
- 6条微博发自 Mary
- 一条微博直接 @mary

Elasticsearch 是如何在三个不同的字段中查找到结果的呢？

当索引一个文档的时候，Elasticsearch 取出所有字段的值拼接成一个大的字符串，作为 `_all` 字段进行索引。例如，当索引这个文档时：

```js
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```

这就好似增加了一个名叫 `_all` 的额外字段：

```js
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```

除非设置特定字段，否则查询字符串就使用 `_all` 字段进行搜索。

在刚开始开发一个应用时，`_all` 字段是一个很实用的特性。之后，你会发现如果搜索时用指定字段来代替 `_all` 字段，将会更好控制搜索结果。当 `_all` 字段不再有用的时候，可以将它置为失效，正如在 [元数据: _all 字段](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/root-object.html#all-field) 中所解释的。

#### 6.4.2 更复杂的查询

下面的查询针对tweents类型，并使用以下的条件：

- `name` 字段中包含 `mary` 或者 `john`
- `date` 值大于 `2014-09-10`
- `_all` 字段包含 `aggregations` 或者 `geo`

```sense
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```



查询字符串在做了适当的编码后，可读性很差：

```js
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```

从之前的例子中可以看出，这种 *轻量* 的查询字符串搜索效果还是挺让人惊喜的。 它的查询语法在相关参考文档中有详细解释，以便简洁的表达很复杂的查询。对于通过命令做一次性查询，或者是在开发阶段，都非常方便。

但同时也可以看到，这种精简让调试更加晦涩和困难。而且很脆弱，一些查询字符串中很小的语法错误，像 `-` ， `:` ， `/` 或者 `"` 不匹配等，将会返回错误而不是搜索结果。

最后，查询字符串搜索允许任何用户在索引的任意字段上执行可能较慢且重量级的查询，这可能会暴露隐私信息，甚至将集群拖垮。

因为这些原因，不推荐直接向用户暴露查询字符串搜索功能，除非对于集群和数据来说非常信任他们。

相反，我们经常在生产环境中更多地使用功能全面的 *request body* 查询API，除了能完成以上所有功能，还有一些附加功能。但在到达那个阶段之前，首先需要了解数据在 Elasticsearch 中是如何被索引的。



## 七、映射和分析

当摆弄索引里面的数据时，我们发现一些奇怪的事情。一些事情看起来被打乱了：在我们的索引中有12条推文，其中只有一条包含日期 `2014-09-15` ，但是看一看下面查询命中的 `总数` （total）：

```sense
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```



为什么在 [`_all` ](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/search-lite.html#all-field-intro)字段查询日期返回所有推文，而在 `date` 字段只查询年份却没有返回结果？为什么我们在 `_all` 字段和 `date` 字段的查询结果有差别？

推测起来，这是因为数据在 `_all` 字段与 `date` 字段的索引方式不同。所以，通过请求 `gb` 索引中 `tweet` 类型的*映射*（或模式定义），让我们看一看 Elasticsearch 是如何解释我们文档结构的：

```sense
GET /gb/_mapping/tweet
```



这将得到如下结果：

```js
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

基于对字段类型的猜测， Elasticsearch 动态为我们产生了一个映射。这个响应告诉我们 `date` 字段被认为是 `date` 类型的。由于 `_all` 是默认字段，所以没有提及它。但是我们知道 `_all` 字段是 `string` 类型的。

所以 `date` 字段和 `string` 字段索引方式不同，因此搜索结果也不一样。这完全不令人吃惊。你可能会认为核心数据类型 strings、numbers、Booleans 和 dates 的索引方式有稍许不同。没错，他们确实稍有不同。

但是，到目前为止，最大的差异在于代表 *精确值* （它包括 `string` 字段）的字段和代表 *全文* 的字段。这个区别非常重要——它将搜索引擎和所有其他数据库区别开来。



### 7.1 精确值VS全文

当摆弄索引里面的数据时，我们发现一些奇怪的事情。一些事情看起来被打乱了：在我们的索引中有12条推文，其中只有一条包含日期 `2014-09-15` ，但是看一看下面查询命中的 `总数` （total）：

```sense
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```

为什么在 [`_all` ](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/search-lite.html#all-field-intro)字段查询日期返回所有推文，而在 `date` 字段只查询年份却没有返回结果？为什么我们在 `_all` 字段和 `date` 字段的查询结果有差别？

推测起来，这是因为数据在 `_all` 字段与 `date` 字段的索引方式不同。所以，通过请求 `gb` 索引中 `tweet` 类型的*映射*（或模式定义），让我们看一看 Elasticsearch 是如何解释我们文档结构的：

```sense
GET /gb/_mapping/tweet
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/052_Mapping_Analysis/25_Data_type_differences.json) 

这将得到如下结果：

```js
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

基于对字段类型的猜测， Elasticsearch 动态为我们产生了一个映射。这个响应告诉我们 `date` 字段被认为是 `date` 类型的。由于 `_all` 是默认字段，所以没有提及它。但是我们知道 `_all` 字段是 `string` 类型的。

所以 `date` 字段和 `string` 字段索引方式不同，因此搜索结果也不一样。这完全不令人吃惊。你可能会认为核心数据类型 strings、numbers、Booleans 和 dates 的索引方式有稍许不同。没错，他们确实稍有不同。

但是，到目前为止，最大的差异在于代表 *精确值* （它包括 `string` 字段）的字段和代表 *全文* 的字段。这个区别非常重要——它将搜索引擎和所有其他数据库区别开来。

### 7.2 倒排索引

Elasticsearch 使用一种称为 *倒排索引* 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

例如，假设我们有两个文档，每个文档的 `content` 域包含如下内容：

1. The quick brown fox jumped over the lazy dog
2. Quick brown foxes leap over lazy dogs in summer

为了创建倒排索引，我们首先将每个文档的 `content` 域拆分成单独的 词（我们称它为 `词条` 或 `tokens` ），创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。结果如下所示：

```
Term      Doc_1  Doc_2
-------------------------
Quick   |       |  X
The     |   X   |
brown   |   X   |  X
dog     |   X   |
dogs    |       |  X
fox     |   X   |
foxes   |       |  X
in      |       |  X
jumped  |   X   |
lazy    |   X   |  X
leap    |       |  X
over    |   X   |  X
quick   |   X   |
summer  |       |  X
the     |   X   |
------------------------
```

现在，如果我们想搜索 `quick brown` ，我们只需要查找包含每个词条的文档：

```
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
quick   |   X   |
------------------------
Total   |   2   |  1
```

两个文档都匹配，但是第一个文档比第二个匹配度更高。如果我们使用仅计算匹配词条数量的简单 *相似性算法* ，那么，我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。

但是，我们目前的倒排索引有一些问题：

- `Quick` 和 `quick` 以独立的词条出现，然而用户可能认为它们是相同的词。
- `fox` 和 `foxes` 非常相似, 就像 `dog` 和 `dogs` ；他们有相同的词根。
- `jumped` 和 `leap`, 尽管没有相同的词根，但他们的意思很相近。他们是同义词。

使用前面的索引搜索 `+Quick +fox` 不会得到任何匹配文档。（记住，`+` 前缀表明这个词必须存在。）只有同时出现 `Quick` 和 `fox` 的文档才满足这个查询条件，但是第一个文档包含 `quick fox` ，第二个文档包含 `Quick foxes` 。

我们的用户可以合理的期望两个文档与查询匹配。我们可以做的更好。

如果我们将词条规范为标准模式，那么我们可以找到与用户搜索的词条不完全一致，但具有足够相关性的文档。例如：

- `Quick` 可以小写化为 `quick` 。
- `foxes` 可以 *词干提取* --变为词根的格式-- 为 `fox` 。类似的， `dogs` 可以为提取为 `dog` 。
- `jumped` 和 `leap` 是同义词，可以索引为相同的单词 `jump` 。

现在索引看上去像这样：

```
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
dog     |   X   |  X
fox     |   X   |  X
in      |       |  X
jump    |   X   |  X
lazy    |   X   |  X
over    |   X   |  X
quick   |   X   |  X
summer  |       |  X
the     |   X   |  X
------------------------
```

这还远远不够。我们搜索 `+Quick +fox` *仍然* 会失败，因为在我们的索引中，已经没有 `Quick` 了。但是，如果我们对搜索的字符串使用与 `content` 域相同的标准化规则，会变成查询 `+quick +fox` ，这样两个文档都会匹配！

这非常重要。你只能搜索在索引中出现的词条，所以索引文本和查询字符串必须标准化为相同的格式。

分词和标准化的过程称为 *分析* ， 我们会在下个章节讨论。



### 7.3 分词器

#### 7.3.1 分析与分析器

*分析* 包含下面的过程：

- 首先，将一块文本分成适合于倒排索引的独立的 *词条* ，
- 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 *recall*

分析器执行上面的工作。 *分析器* 实际上是将三个功能封装到了一个包里：

- **字符过滤器**

  首先，字符串按顺序通过每个 *字符过滤器* 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 `&` 转化成 `and`。

- **分词器**

  其次，字符串被 *分词器* 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

- **Token 过滤器**

  最后，词条按顺序通过每个 *token 过滤器* 。这个过程可能会改变词条（例如，小写化 `Quick` ），删除词条（例如， 像 `a`， `and`， `the` 等无用词），或者增加词条（例如，像 `jump` 和 `leap` 这种同义词）。

Elasticsearch提供了开箱即用的字符过滤器、分词器和token 过滤器。 这些可以组合起来形成自定义的分析器以用于不同的目的。我们会在 [自定义分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/custom-analyzers.html) 章节详细讨论。

#### 7.3.2 内置分析器

但是， Elasticsearch还附带了可以直接使用的预包装的分析器。接下来我们会列出最重要的分析器。为了证明它们的差异，我们看看每个分析器会从下面的字符串得到哪些词条：

```
"Set the shape to semi-transparent by calling set_trans(5)"
```

- **标准分析器**

  标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 [Unicode 联盟](http://www.unicode.org/reports/tr29/) 定义的 *单词边界* 划分文本。删除绝大部分标点。最后，将词条小写。它会产生`set, the, shape, to, semi, transparent, by, calling, set_trans, 5`

- **简单分析器**

  简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生`set, the, shape, to, semi, transparent, by, calling, set, trans`

- **空格分析器**

  空格分析器在空格的地方划分文本。它会产生`Set, the, shape, to, semi-transparent, by, calling, set_trans(5)`

- **语言分析器**

  特定语言分析器可用于 [很多语言](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-lang-analyzer.html)。它们可以考虑指定语言的特点。例如， `英语` 分析器附带了一组英语无用词（常用单词，例如 `and` 或者 `the` ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 *词干* 。`英语` 分词器会产生下面的词条：`set, shape, semi, transpar, call, set_tran, 5`注意看 `transparent`、 `calling` 和 `set_trans` 已经变为词根格式。

#### 7.3.3 什么时候使用分析器

当我们 *索引* 一个文档，它的全文域被分析成词条以用来创建倒排索引。 但是，当我们在全文域 *搜索* 的时候，我们需要将查询字符串通过 *相同的分析过程* ，以保证我们搜索的词条格式与索引中的词条格式一致。

全文查询，理解每个域是如何定义的，因此它们可以做正确的事：

- 当你查询一个 *全文* 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。
- 当你查询一个 *精确值* 域时，不会分析查询字符串，而是搜索你指定的精确值。

现在你可以理解在 [开始章节](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/mapping-analysis.html) 的查询为什么返回那样的结果：

- `date` 域包含一个精确值：单独的词条 `2014-09-15`。
- `_all` 域是一个全文域，所以分词进程将日期转化为三个词条： `2014`， `09`， 和 `15`。

当我们在 `_all` 域查询 `2014`，它匹配所有的12条推文，因为它们都含有 `2014` ：

```sense
GET /_search?q=2014              # 12 results
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/052_Mapping_Analysis/25_Data_type_differences.json) 

当我们在 `_all` 域查询 `2014-09-15`，它首先分析查询字符串，产生匹配 `2014`， `09`， 或 `15` 中 *任意* 词条的查询。这也会匹配所有12条推文，因为它们都含有 `2014` ：

```sense
GET /_search?q=2014-09-15        # 12 results !
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/052_Mapping_Analysis/25_Data_type_differences.json) 

当我们在 `date` 域查询 `2014-09-15`，它寻找 *精确* 日期，只找到一个推文：

```sense
GET /_search?q=date:2014-09-15   # 1  result
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/052_Mapping_Analysis/25_Data_type_differences.json) 

当我们在 `date` 域查询 `2014`，它找不到任何文档，因为没有文档含有这个精确日志：

```sense
GET /_search?q=date:2014         # 0  results !
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/052_Mapping_Analysis/25_Data_type_differences.json) 

#### 7.3.4测试分析器

有些时候很难理解分词的过程和实际被存储到索引中的词条，特别是你刚接触Elasticsearch。为了理解发生了什么，你可以使用 `analyze` API 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本：

```sense
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/052_Mapping_Analysis/40_Analyze.json) 

结果中每个元素代表一个单独的词条：

```js
{
   "tokens": [
      {
         "token":        "text",
         "start_offset": 0,
         "end_offset":   4,
         "type":         "<ALPHANUM>",
         "position":     1
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}
```

`token` 是实际存储到索引中的词条。 `position` 指明词条在原始文本中出现的位置。 `start_offset` 和 `end_offset` 指明字符在原始字符串中的位置。

每个分析器的 `type` 值都不一样，可以忽略它们。它们在Elasticsearch中的唯一作用在于[`keep_types` token 过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-keep-types-tokenfilter.html)。

`analyze` API 是一个有用的工具，它有助于我们理解Elasticsearch索引内部发生了什么，随着深入，我们会进一步讨论它。

#### 7.3.5指定分析器

当Elasticsearch在你的文档中检测到一个新的字符串域，它会自动设置其为一个全文 `字符串` 域，使用 `标准` 分析器对它进行分析。

你不希望总是这样。可能你想使用一个不同的分析器，适用于你的数据使用的语言。有时候你想要一个字符串域就是一个字符串域—不使用分析，直接索引你传入的精确值，例如用户ID或者一个内部的状态域或标签。

要做到这一点，我们必须手动指定这些域的映射。

### 7.4 映射

### 7.5 复杂核心域类型

除了我们提到的简单标量数据类型， JSON 还有 `null` 值，数组，和对象，这些 Elasticsearch 都是支持的。

#### 7.5.1 多值域

很有可能，我们希望 `tag` 域包含多个标签。我们可以以数组的形式索引标签：

```js
{ "tag": [ "search", "nosql" ]}
```

对于数组，没有特殊的映射需求。任何域都可以包含0、1或者多个值，就像全文域分析得到多个词条。

这暗示 *数组中所有的值必须是相同数据类型的* 。你不能将日期和字符串混在一起。如果你通过索引数组来创建新的域，Elasticsearch 会用数组中第一个值的数据类型作为这个域的 `类型` 。

当你从 Elasticsearch 得到一个文档，每个数组的顺序和你当初索引文档时一样。你得到的 `_source` 域，包含与你索引的一模一样的 JSON 文档。

但是，数组是以多值域 *索引的*—可以搜索，但是无序的。 在搜索的时候，你不能指定 “第一个” 或者 “最后一个”。 更确切的说，把数组想象成 *装在袋子里的值* 。

#### 7.5.2 空域

当然，数组可以为空。这相当于存在零值。 事实上，在 Lucene 中是不能存储 `null` 值的，所以我们认为存在 `null` 值的域为空域。

下面三种域被认为是空的，它们将不会被索引：

```js
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]
```

#### 7.4.3 多层级对象

我们讨论的最后一个 JSON 原生数据类是 *对象* -- 在其他语言中称为哈希，哈希 map，字典或者关联数组。

*内部对象* 经常用于嵌入一个实体或对象到其它对象中。例如，与其在 `tweet` 文档中包含 `user_name` 和 `user_id` 域，我们也可以这样写：

```js
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
```

#### 7.5.4 内部对象的映射

Elasticsearch 会动态监测新的对象域并映射它们为 `对象` ，在 `properties` 属性下列出内部域：

```js
{
  "gb": {
    "tweet": { 
      "properties": {
        "tweet":            { "type": "string" },
        "user": { 
          "type":             "object",
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   { 
              "type":         "object",
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```

|      | 根对象   |
| ---- | -------- |
|      | 内部对象 |

`user` 和 `name` 域的映射结构与 `tweet` 类型的相同。事实上， `type` 映射只是一种特殊的 `对象` 映射，我们称之为 *根对象* 。除了它有一些文档元数据的特殊顶级域，例如 `_source` 和 `_all` 域，它和其他对象一样。

#### 7.5.5 内部对象是如何索引的

Lucene 不理解内部对象。 Lucene 文档是由一组键值对列表组成的。为了能让 Elasticsearch 有效地索引内部类，它把我们的文档转化成这样：

```js
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
```

*内部域* 可以通过名称引用（例如， `first` ）。为了区分同名的两个域，我们可以使用全 *路径* （例如， `user.name.first` ） 或 `type` 名加路径（ `tweet.user.name.first` ）。

在前面简单扁平的文档中，没有 `user` 和 `user.name` 域。Lucene 索引只有标量和简单值，没有复杂数据结构。

#### 7.5.6 内部对象数组

最后，考虑包含内部对象的数组是如何被索引的。 假设我们有个 `followers` 数组：

```js
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```

这个文档会像我们之前描述的那样被扁平化处理，结果如下所示：

```js
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```

`{age: 35}` 和 `{name: Mary White}` 之间的相关性已经丢失了，因为每个多值域只是一包无序的值，而不是有序数组。这足以让我们问，“有一个26岁的追随者？”

但是我们不能得到一个准确的答案：“是否有一个26岁 *名字叫 Alex Jones* 的追随者？”

相关内部对象被称为 *nested* 对象，可以回答上面的查询，我们稍后会在[*嵌套对象*](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/nested-objects.html)中介绍它。



## 八 请求体查询

### 8.1 空查询

让我们以最简单的 `search` API 的形式开启我们的旅程，空查询将返回所有索引库（indices)中的所有文档：

```sense
GET /_search
{} 
```



|      | 这是一个空的请求体。 |
| ---- | -------------------- |
|      |                      |

只用一个查询字符串，你就可以在一个、多个或者 `_all` 索引库（indices）和一个、多个或者所有types中查询：

```js
GET /index_2014*/type1,type2/_search
{}
```



同时你可以使用 `from` 和 `size` 参数来分页：

```js
GET /_search
{
  "from": 30,
  "size": 10
}
```





**一个带请求体的 GET 请求？**

某些特定语言（特别是 JavaScript）的 HTTP 库是不允许 `GET` 请求带有请求体的。事实上，一些使用者对于 `GET` 请求可以带请求体感到非常的吃惊。

而事实是这个RFC文档 [RFC 7231](http://tools.ietf.org/html/rfc7231#page-24)— 一个专门负责处理 HTTP 语义和内容的文档 — 并没有规定一个带有请求体的 `GET` 请求应该如何处理！结果是，一些 HTTP 服务器允许这样子，而有一些 — 特别是一些用于缓存和代理的服务器 — 则不允许。

对于一个查询请求，Elasticsearch 的工程师偏向于使用 `GET` 方式，因为他们觉得它比 `POST` 能更好的描述信息检索（retrieving information）的行为。然而，因为带请求体的 `GET` 请求并不被广泛支持，所以 `search` API同时支持 `POST` 请求：

```js
POST /_search
{
  "from": 30,
  "size": 10
}
```

类似的规则可以应用于任何需要带请求体的 `GET` API。

我们将在聚合 [聚合](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/aggregations.html) 章节深入介绍聚合（aggregations），而现在，我们将聚焦在查询。

相对于使用晦涩难懂的查询字符串的方式，一个带请求体的查询允许我们使用 *查询领域特定语言（query domain-specific language）* 或者 Query DSL 来写查询语句。

### 8.2 查询表达式

查询表达式(Query DSL)是一种非常灵活又富有表现力的 查询语言。 Elasticsearch 使用它可以以简单的 JSON 接口来展现 Lucene 功能的绝大部分。在你的应用中，你应该用它来编写你的查询语句。它可以使你的查询语句更灵活、更精确、易读和易调试。

要使用这种查询表达式，只需将查询语句传递给 `query` 参数：

```js
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```



*空查询（empty search）* —`{}`— 在功能上等价于使用 `match_all` 查询， 正如其名字一样，匹配所有文档：

```sense
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```



拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/054_Query_DSL/60_Empty_query.json) 

#### 8.2.1 查询语句的结构

一个查询语句的典型结构：

```js
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

如果是针对某个字段，那么它的结构如下：

```js
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

举个例子，你可以使用 `match` 查询语句 来查询 `tweet` 字段中包含 `elasticsearch` 的 tweet：

```js
{
    "match": {
        "tweet": "elasticsearch"
    }
}
```

完整的查询请求如下：

```sense
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/054_Query_DSL/60_Match_query.json) 

#### 8.2.2 合并查询语句

*查询语句(Query clauses)* 就像一些简单的组合块，这些组合块可以彼此之间合并组成更复杂的查询。这些语句可以是如下形式：

- *叶子语句（Leaf clauses）* (就像 `match` 语句) 被用于将查询字符串和一个字段（或者多个字段）对比。
- *复合(Compound)* 语句 主要用于 合并其它查询语句。 比如，一个 `bool` 语句 允许在你需要的时候组合其它语句，无论是 `must` 匹配、 `must_not` 匹配还是 `should` 匹配，同时它可以包含不评分的过滤器（filters）：

```sense
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }},
        "filter":   { "range": { "age" : { "gt" : 30 }} }
    }
}
```

拷贝为 cURL[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/snippets/054_Query_DSL/60_Bool_query.json) 

一条复合语句可以合并 *任何* 其它查询语句，包括复合语句，了解这一点是很重要的。这就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

例如，以下查询是为了找出信件正文包含 `business opportunity` 的星标邮件，或者在收件箱正文包含 `business opportunity` 的非垃圾邮件：

```js
{
    "bool": {
        "must": { "match":   { "email": "business opportunity" }},
        "should": [
            { "match":       { "starred": true }},
            { "bool": {
                "must":      { "match": { "folder": "inbox" }},
                "must_not":  { "match": { "spam": true }}
            }}
        ],
        "minimum_should_match": 1
    }
}
```

到目前为止，你不必太在意这个例子的细节，我们会在后面详细解释。最重要的是你要理解到，一条复合语句可以将多条语句 — 叶子语句和其它复合语句 — 合并成一个单一的查询语句。







### 








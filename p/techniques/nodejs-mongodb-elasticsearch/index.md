# nodejs + mongodb + elasticsearch 实现简单的搜索功能

## 大概

![在这里输入图片描述][1]

 1. 数据存储在 mongodb
 2. 使用 elasticsearch 作为搜索数据库
 3. nodejs 用官方客户端 elasticsearch-js 请求 elasticsearch
 4. 使用 mongo-connector 同步 mongodb 的数据到 elasticsearch
 5. 前端随便请求一下

## elasticsearch

本文使用的是 [elasticsearch-rtf][2]，这个库使用的是 elasticsearch 5.1 版本，并且预装了一些常用插件，比如常用分词器。

在安装完 java 环境后，就可以直接运行 elasticsearch 目录里的 bin/elasticsearh。**不过在虚拟机中**，会有各种像内存不足，文件描述符限制等问题。所以在虚拟机中，我们需要修改一些配置:

 1. 修改 config/jvm.options 里的 jvm 内存为 512m
    ````
    # Xms represents the initial size of total heap space
    # Xmx represents the maximum size of total heap space

    #-Xms2g
    #-Xmx2g

    -Xms512m
    -Xmx512m
    ````

 2. 修改文件描述符上限，需要 root 权限，然后运行
    ````
    ulimit -n 65536
    ````

 3. 为了让别的机器能连上 elasitcsearch 机器，需要把 config/elasticsearch.yml 里的 network.host 修改为 0.0.0.0。另外，还需要运行如下命令
    ````
    sysctl -w vm.max_map_count=262144
    ````
    不然会报 vm.max_map_count 不足的错误。

## mongodb 

安装就好，没有坑

## mongo-connector

[mongo-connector][3] 是一个 python 编写的，用来复制 mongodb 中数据到各种搜索数据库的工具，支持 elasticsearch。

使用这个库前，要先在 mongodb 中设置 replSet，具体可以查看[这里][4]。

另外，在运行中需要用到 elastic_doc_manager 这个库，需要另外安装，在他的 README 里没有明确指明需要另外安装，具体可以查看[这里][5]。

最后，运行
````
mongo-connector -m localhost:27017 -t localhost:9200 -d elastic_doc_manager
````

mongo-connector 会把 mongodb 里的数据同步到 elasticsearch。

## 测试数据

测试数据使用 [Faker-zh-cn.js][6] 生成。这个库是 [faker.js][7] 的中文版。代码如下:

````

const { MongoClient } = require('mongodb')
const faker = require('faker-zh-cn')
const DB_URL = 'mongodb://192.168.134.125:27017/test'

faker.locale = 'zh_CN'

const insertData = function(db, callback) {

  let collection = db.collection('data')

  let data = []

  for (let i = 0; i <= 1000; i++) {
    data.push({
      "name": faker.Name.findName(),
      "address": `${faker.Address.city()},${faker.Address.streetName()},${faker.Address.streetAddress()}`,
      "description": faker.Lorem.paragraph()
    })
  }

  collection.insert(data, function(err, result) {
    if(err) {
      console.log('Error:'+ err)
      return
    }
    console.log('success')
    callback && callback(result)
  })
}

MongoClient.connect(DB_URL, function (err, db) {
  if (err) {
    return console.log(err)
  }
  console.log('连接成功')
  insertData(db)
})

````

## 替换分词器

生成测试数据后，使用 elasticsearch 的 [rest api][8] 测试搜索，发现搜索结果并不理想。比如使用查询条件 q=王者农药，出来的结果除了王者、农药相关的结果，连包含王、者、农或药的结果都出来了，说明搜索默认对中文以字为单位分词，我们需要替换分词器。

[elasticsearch-analysis-ik][9] 是一个流行的中文分词库，在 elasticsearch-rtf 中已经集成，可直接使用。

更换分词器需要对 elasticsearch 的[映射][10]重定义。使用 elasticsearch 的 rest api 进行设置，会报错，大概意思就是已经存在数据的索引的映射不能被修改，所以要另辟蹊径。

后来查到，只需要根据旧的[索引和类型][11]，重新建立一个索引，然后把想要修改的索引，[reindex][12] 到新索引即可。新索引的映射带上了 analyzer: "ik_smart" 的声明，就可以用上这个分词器了。

最后，搜索结果变得正常了一点，只会出现王者和农药相关的结果了，说明分词器起效了。

## nodejs 代码

用的 express 随便意思一下
````

const { Client } = require('elasticsearch')
const INDEX = 'index'
const express = require('express')
const app = express()

let client = new Client({
  host: '192.168.134.125:9200',
  log: 'trace'
})

app.use(express.static(__dirname + '/public'))

app.get('/', (req, res) => {
  res.send('Hello World')
})

app.get('/search/:keyword?', (req, res) => {
  client.search({
    q: req.params.keyword,
    index: INDEX,
    size: 999
  }).then(function (body) {
    var hits = body.hits.hits
    res.send(hits)
  }, function (error) {
    console.trace(error.message)
  })
})

console.log('listening port: 3000')
app.listen(3000)


````

前端就是一个简单的页面，调用 nodejs 的接口，看起来是这样的：

![在这里输入图片描述][13]

## 最后
本人才疏学浅，如有纰漏，还望指正。

## 参考
https://segmentfault.com/a/1190000003773614

  [1]: https://dn-coding-net-production-pp.qbox.me/531fbce3-ea36-4492-b235-54b3de7bf51a.png
  [2]: https://github.com/medcl/elasticsearch-rtf
  [3]: https://github.com/mongodb-labs/mongo-connector
  [4]: https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/
  [5]: https://github.com/mongodb-labs/mongo-connector/wiki/Usage%20with%20ElasticSearch
  [6]: https://github.com/layerssss/Faker-zh-cn.js
  [7]: https://github.com/Marak/faker.js
  [8]: https://es.xiaoleilu.com/010_Intro/15_API.html
  [9]: https://github.com/medcl/elasticsearch-analysis-ik
  [10]: https://es.xiaoleilu.com/052_Mapping_Analysis/45_Mapping.html
  [11]: https://es.xiaoleilu.com/030_Data/10_Index.html
  [12]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html
  [13]: https://dn-coding-net-production-pp.qbox.me/9e3080fb-2525-4266-9230-5056f3396c3d.png
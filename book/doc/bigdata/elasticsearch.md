[TOC]

# 1、创建索引

```
PUT /testindex
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
      }
}
```

# 2、查询索引

```
GET /testindex
```

# 3、删除索引

```
DELETE /testindex
```

# 4、创建类型

```
PUT testindex/_mapping/goods
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "ik_max_word"
    },
    "images": {
      "type": "keyword",
      "index": "false"
    },
    "price": {
      "type": "float"
    }
  }
}
```

# 5、新增数据

```
POST /testindex/goods/
{
    "title":"小米手机",
    "images":"1.jpg",
    "price":111.00
}
```

# 6、新增数据（动态加字段、自定义id）

```
POST /testindex/goods/3
{
    "title":"超米手机",
    "images":"3.jpg",
    "price":333,
    "stock": 200
}
```

# 7、修改数据

```
PUT /testindex/goods/3
{
    "title":"超米手机",
    "images":"3.jpg",
    "price":333,
    "stock": 100
}
```

# 8、删数据

```
DELETE /testindex/goods/3
```

# 9、查数据

```
GET /testindex/goods/_search
```

更多搜索功能，参考：https://www.jianshu.com/p/28b803a88d40



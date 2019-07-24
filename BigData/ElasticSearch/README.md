## ElasticSearch
    ```
    PUT http://127.0.0.1:9200/megacorp/employee/1
    {
       "first_name":"John",
       "last_name":"Smith",
       "age":25,
       "about":"I love to go rock climbing",
       "interests":["sports","music"]
    }
    PUT http://127.0.0.1:9200/megacorp/employee/2
    {
       "first_name":"Jane",
       "last_name":"Smith",
       "age":32,
       "about":"I like to go collect rock albums",
       "interests":["music"]
    }
    PUT http://127.0.0.1:9200/megacorp/employee/3
    {
       "first_name":"Douglas",
       "last_name":"Fir",
       "age":35,
       "about":"I like to build cabinets",
       "interests":["forestry"]
    }
    ```
## 请求
```
#获取员工1
GET http://127.0.0.1:9200/megacorp/employee/1

#搜索全部员工
GET http://127.0.0.1:9200/megacorp/employee/_search

#搜索姓氏中包含"Smith"人员工
GET http://127.0.0.1:9200/megacorp/employee/_search?q=last_name:Smith

#DSL语句查询
GET GET http://127.0.0.1:9200/megacorp/employee/_search
{
   "query":{
       "match":{
          "last_name":"Smith"
       }
   }
}

#添加过滤器
GET http://127.0.0.1:9200/megacorp/employee/_search
{
   "query":{
       "bool":{
          "filter":{
             "range":{
                "age":{
                   "gt":30
                }
             }
          },
          "must":{
             "match":{
                "last_name":"Smith"
             }
          }
       }
   }
}

#全文搜索
GET http://127.0.0.1:9200/megacorp/employee/_search
{
   "query":{
       "match":{
          "about":"rock climbing"
       }
   }
}

#短语搜索
GET http://127.0.0.1:9200/megacorp/employee/_search
{
   "query":{
       "match_phrase":{
          "about":"rock climbing"
       }
   }
}

#聚合 
#开启聚合
PUT http://127.0.0.1:9200/megacorp/_mapping/
{
  "properties":{
    "interests": { 
      "type":     "text",
      "fielddata": true
    }
   }
}
#找到所有职员中最大的共同点(兴趣爱好)是什么:
GET http://127.0.0.1:9200/megacorp/employee/_search
{
  "aggs":{
     "all_insterests":{
        "terms:{
           "field":"interests"
        }
     }
  }
}

#查询所有姓"Smith"的人最大的共同点(兴趣爱好)
{
  "query":{
    "match":{
      "last_name":"smith"
    }
  },
  "aggs":{
     "all_insterests":{
        "terms:{
           "field":"interests"
        }
     }
  }
}

#聚合也允许分级汇总 ,统计每种兴趣下职员的平均年龄
GET http://127.0.0.1:9200/megacorp/employee/_search

{
  "aggs":{
     "all_insterests":{
        "terms:{
           "field":"interests"
        },
     },
     "aggs":{
       "avg_age":{
         "avg":{"field":"age"}
       }
     }
  }
}

```

## 分布式集群
  - 集群健康  
 ` GET /_cluster/health`  
    - `green` 所有主要分片和复制分片都可用  
    - `yellow` 所有主要分片可用，但不是所有复制分片都可用
    - `red` 不是所有的主要分片都可用
    


## 检索文档
  - `pretty` `格式化输出JSON`
  curl -i -XGET http://localhost:9200/website/blog/124?pretty  
  
  - 检索文档一部分
  GET http://localhost:9200/website/blog/123?_source=title,text  
  
  - 只获取_source字段，不要其他元数据
   GET http://localhost:9200/website/blog/123/_source
   
  - 检查文档是否存在
  curl -i -XHEAD http://localhost:9200/website/blog/123
  
  - 更新整个文档
      ```
      PUT http://localhost:9200/website/blog/123
      {
         "title":"My first blog entry",
         "text":"I am starting to get the hang of this...",
         "date": "2014/01/02"
      }
      ```
  - 创建文档
    - 第一种方式
      ```
      POST http://localhost:9200/website/blog/1
      {...}
      ```
    - 第二种方式
        ```
        PUT http://localhost:9200/website/blog/1?op_type=create
        {...}
        ```
    - 第三种方式
        ```
        PUT http://localhost:9200/website/blog/1/_create
        {...}
        ```
  - 删除文档
  DELETE http://localhost:9200/website/blog/1/
  
  - 检索多个文档
      ```
      GET  http://localhost:9200/_mget  
      {
         "docs":[
          {
             "_index": "website",
             "_type": "blog",
             "_id": 2
          },
          {
             "_index": "website",
             "_type": "pageviews",
             "_id": 1
           }
         ]  
       }
       ```
  - 批量执行
      ```
      POST http://localhost:9200/_bulk
      { "delete":{"_index":"website","_type":"blog","_id":"123}}
      { "create":{"_index":"website","_type":"blog","_id":"123}}
      { "title": "My first blog post" }
      { "index": { "_index": "website","_type":"blog"}}
      { "title": "My first blog post" }
      { "update": {"_index":"website","_type":"blog","_id":"123","_retry_on_conflict":3}}
      { "doc":{"title":"My updated blog post"}}
      ```
  - 分页
  GET http://localhost:9200/website/blog/_search?size=5  
  GET http://localhost:9200/website/blog/_search?size=5&from=5  
  
  
  
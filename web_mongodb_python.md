## mongodb

#### 源生语句和pymongo查询的区别

 ### pymongo连接
```
from config.config import Config
from pymongo import MongoClient
mongo = MongoClient(Config.MONGO_URI)
db = mongo[Config.MONGO_DBNAME]

# config文件
class Config:
    DEBUG = False
    MONGO_URI = "mongodb://xxxxxxxxxx"
    MONGO_DBNAME = "xxxxx"

    RULE_MODEL_IP_LIST = [
            'xx.xxx.xx.xxx',
            'xx.xxx.xx.xxx',
            'xx.xxx.xx.xxx',
            'xx.xxx.xx.xxx',
        ]
```
### pymongo查询

**比较运算查询**
```
# lt、lte、gt、gte、in、nin、ne
db.customers.find({id:{$lte:100,$gte:50}})
# $nin 和 $ne是低效查询器，最好不要单独使用
```
**连接运算查询**
```
# or and exists
db.xxx.find({$or:[{id:1},{age:100}]})
# 查询id不存在的文档
db.xxx.find({id:{$exists:flase}})
db.xxx.find({'id':null})  //同上功能
```
**嵌套查询**
```
#示例
{
    "_id" : ObjectId("5b66c09b7506cbadbc344c2e"),
    "name" : "zhang",
    "sex" : "不男不女",
    "age" : ISODate("2018-08-15T17:05:26.974Z"),
    "score" : 99.0,
    "address" : "--",
    "group" : [ 
        {
            "age" : 5,
            "ss" : 1
        }, 
        {
            "age" : 1
        }
    ]
}

# 查询
db.getCollection('students').find({'group.0.age':5}) //单引号不要缺少
# 不指定索引时，返回所有包括5的age文档
db.xxx.find({'group.age':5})
# 查询数组字段中的第一个或者最后一个
db.getCollection('students').find({'group.age':5},{'group':{'$slice': 1}})

```

```python
db.表名.find()
# 查询时，所有字段需要控制格式，字符串加引号，返回一个pymongo对象，做遍历查询时需要list做转换成列表

db.表明.find_one()
返回一个mongodb文档，可以直接用字典方法提取

# 排序
db.getCollection('#####').find({}).sort({date:-1})

# python中查询id值时可以通过前段传来的id值通过objectid对象来传递查询
kw = db.表名.find_one({'_id': ObjectId(str(_id)})

# 查询数组中是否包含
kw = db.表名.find_one({'_id': ObjectId(str(_id),'manager':{'$in': '包含字段'}})
```
### pymongo插入

**支持批量插入**
```
for(var i=1;i<11;i++) db.students.insert({id:i,name:'ss',age:100+i})
```
mongodb的数据格式是BSON格式
```python
#  生成唯一ID值的方法
from bson import ObjectId

#  ObjectId前4个字节表示时间戳，接下来的3个字节是机器标识码，紧接的两个字节由进程id组成（PID），最后三个字节是随机数。
a = ObjectId()

#  可以通过getTimestamp函数来获取文档的创建时间
ObjectId("5349b4ddd2781d08c09890f4").getTimestamp()
#  返回对象是ISO格式的文档穿件时间：
ISODate("2014-04-12T21:49:17Z")

```
### mongodb更新
```
doc = db.表明.find_one()
doc['first'] = 4
db.表名.save(doc)
```
mongodb 有固定集合可以覆盖之前的纪录
#### 聚合查询 pymongo和源生通用
*aggregate*
```python
# $match 匹配

    alarm = db.alarm.aggregate([
        {'$match': {'seat': seat}},
        {'$group': {
            #  按条件聚合查询
            '_id': {'company_id': '$company_id', 'seat': '$seat', 'customer': '$customer', 'date': '$date'},
            #  增加词组
            'wangwang': {'$addToSet': '$wangwang'},
            'alert_time': {'$addToSet': '$alert_time'},
            'emotion': {'$addToSet': '$emotion'},
            'keyword': {'$addToSet': '$keyword'},
            'rule': {'$addToSet': '$rule'},
        }},
        # 从第几条开始
        {'$skip': startIndex},
        # 返回数量限制
        {'$limit': limit}
    ])

```

### mongodb查询分析
开启 Profiling 功能，纪录性能日志
>   level = 0 不纪录
    level = 1 纪录阀值
    level = 2 所有都纪录
```
drug:PRIMARY> db.system.profile.find().pretty()
{
    "op" : "query",    #操作类型，有insert、query、update、remove、getmore、command   
    "ns" : "mc.user",  #操作的集合
    "query" : {        #查询语句
        "mp_id" : 5,
        "is_fans" : 1,
        "latestTime" : {
            "$ne" : 0
        },
        "latestMsgId" : {
            "$gt" : 0
        },
        "$where" : "new Date(this.latestNormalTime)>new Date(this.replyTime)"
    },
    "cursorid" : NumberLong("1475423943124458998"),
    "ntoreturn" : 0,   #返回的记录数。例如，profile命令将返回一个文档（一个结果文件），因此ntoreturn值将为1。limit(5)命令将返回五个文件，因此ntoreturn值是5。如果ntoreturn值为0，则该命令没有指定一些文件返回，因为会是这样一个简单的find()命令没有指定的限制。
    "ntoskip" : 0,     #skip()方法指定的跳跃数
    "nscanned" : 304,  #扫描数量
    "keyUpdates" : 0,  #索引更新的数量，改变一个索引键带有一个小的性能开销，因为数据库必须删除旧的key，并插入一个新的key到B-树索引
    "numYield" : 0,    #该查询为其他查询让出锁的次数
    "lockStats" : {    #锁信息，R：全局读锁；W：全局写锁；r：特定数据库的读锁；w：特定数据库的写锁
        "timeLockedMicros" : {     #锁
            "r" : NumberLong(19467),
            "w" : NumberLong(0)
        },
        "timeAcquiringMicros" : {  #锁等待
            "r" : NumberLong(7),
            "w" : NumberLong(9)
        }
    },
    "nreturned" : 101,        #返回的数量
    "responseLength" : 74659, #响应字节长度
    "millis" : 19,            #消耗的时间（毫秒）
    "ts" : ISODate("2014-02-25T02:13:54.899Z"), #语句执行的时间
    "client" : "127.0.0.1",   #链接ip或则主机
    "allUsers" : [ ],     
    "user" : ""               #用户
}

```


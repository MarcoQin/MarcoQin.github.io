---

layout: post
title: 关于mongodb中collection的设计问题
category : Dev
tagline: "Supporting tagline"
tags : [mongodb, database, nosql]

---

最近一段时间mongodb使用比较频繁, 公司好多数据都从MySQL迁移到mongo. 因此, 便出现了一些数据库的设计问题.

比如有业务需求, 存储通讯录. 那么问题来了: 每个用户记录通讯录的方式都不相同, 如何将移动端的通讯录较为合理地保存, 并以前段容易解析的结构返回response.

mongodb是文档化的数据库, 所有的collection都可以随意定义.

刚开始设计是按照如下方式:

{% highlight python %}
_id: userid, int
phonebook:[
    {
        'base_info':{
            'name':'',  # string, 通讯录中的名称
            'telephones':['15611001733'],  # 通讯录中的电话号码
            'extra': '',  string
        },
        'app_info':{
            # some app_info here
        },
    },
]
{% endhighlight %}

出来后发现, 这种结构对数据更新太不友好.

无论是在mongodb还是在日常Python脚本中, 哈希表都是一种非常友好非常强大的数据结构.

于是, 改良后, collection设计如下:

```python
{
    '_id': userid,  # int, 用户id作为主键
    'phonebook':{  # 电话簿存储
        '{{telephone}}':{  # 通讯录中的号码作为主键
            'ref': 'telephone',  # 该号码指向的号码, 被指向号码持有该号码的全部信息. 信息持有者指向自己
            'app_info':{
                    'key':'value', # 用户在应用中的一些信息
                }
            },
            'base_info':{
                'name': '',  # string, 姓名
                'extra': '',  # string,
            }
        }
    }
}
```

变成如上结构之后, 无论是查找, 更新, 还是包装成response返回给客户端, 操作方便性差不多是平衡的.

但是由此还是产生了新的问题: 若是我不知道其中一个key的parent key的信息, 查询就异常困难.

mongodb有对应的对未知键的数据查询方式, 通过关键字'$where'放入一段javascript函数进行查询. 应用到此处便如下所示:

```javascript
{
    '_id': 0,
    "$where" : function(){
        for( var c in this ){
            if( c == "phonebook" ){
                for(var i in this[c]){
                    if('app_info' in this[c][i]){
                        if(this[c][i]['app_info']['user_info']['userid']==xxx){return true;
                        }
                    }
                }
            };
        }
        return false;
    }
}
```

**注意:** 在Python中使用时, '$where'后面跟的是string. 可以先用三引号将js函数包围.

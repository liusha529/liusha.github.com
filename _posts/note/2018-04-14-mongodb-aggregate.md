---
layout: post
title: "Mongodb聚合、联表、分组"
date: 2018-04-14 10:25:15 +0800 
categories: 我的笔记
tag: 笔记
---
* content
{:toc}

<!-- more -->


# Mongodb聚合管道，多表查询，项目实例

## 官方文档地址：

[https://docs.mongodb.com/manual/reference/operator/aggregation/](https://docs.mongodb.com/manual/reference/operator/aggregation/)

-------

## Aggregate

聚合管道是一个基于数据处理管道概念的框架。   通过使用一个多阶段的管道，将一组文档转换为最终的聚合结果


-------

### 相关操作介绍：
下面表达式是在aggregate里面使用

| 表达式 | 描述 |  
| --- | --- | 
| $project |修改流中的文档，比如增加或者移除一个字段。对于每一个输入文档，相应输出一个文档。| 
| $match |对流中的文档进行过滤，仅允许符合条件的文档进入下一个阶段，过滤操作不会修改文档。 $match 操作使用Mongodb标准的查询条件。对于每一个输入文档，如果符合条件则输出这个文档，否则就丢弃该文档。|
|$sort|通过指定的排序键重新排序文档流。只有顺序发生了变化;文档保持不变。对于每个输入文档，输出一个文档。|
|$skip|在聚合管道中跳过指定数量的文档，并返回余下的文档|
|$limit|用来限制MongoDB聚合管道返回的文档数| 
|$unwind|从输入文档中解析一个数组字段，并为每个元素输出一个文档。每个输出文档都用一个元素值替换这个数组。对于每个输入文档，输出n个文档，其中n是数组元素的数量，而空数组可以为0。|
|$sample|随机返回指定文档数量|
|$group|根据指定的标识符表达式分组输入文档，如果指定，则将累加器表达式(s)应用到每个组。使用所有输入文档，并为每个不同的组输出一个文档。输出文档只包含标识符字段，如果指定的话，则是累积字段|
|$lookup|对同一数据库中的另一个集合执行左外联接，以便从“加入”集合中的文档中进行处理|
|$facet|在同一组输入文档的同一阶段内处理多个聚合管道。允许创建多面聚合，在单个阶段中，可以跨多个维度或多个方面来描述数据|
|$replaceRoot|用指定的嵌入文档替换文档。操作代替输入文档中的现有的所有领域，包括_id场。指定嵌入在输入文档中的文档，以将嵌入式文档提升到顶层|

-------


### $group相关操作介绍 
根据指定的标识符表达式分组输入文档，如果指定，则将累加器表达式(s)应用到每个组。使用所有输入文档，并为每个不同的组输出一个文档。输出文档只包含标识符字段，如果指定的话，则是累积字段

下面表达式是在$group里面使用

| 表达式 | 描述 |
| --- | --- |
|$sum|计算总和 | 
|$avg|计算平均值|
|$min|获取集合中所有文档对应值得最小值|
|$max|获取集合中所有文档对应值得最大值|
|$push|在结果文档中插入值到一个数组中|
|$addToSet|在结果文档中插入值到一个数组中，但不创建副本|
|$first|根据资源文档的排序获取第一个文档数据|
|$last|根据资源文档的排序获取最后一个文档数据|

-------

### $lookup 连接查询相关操作介绍
对同一数据库中的另一个集合执行左外联接，以便从“加入”集合中的文档中进行处理。

| 字段 | 描述 | 
| --- | --- | 
|from| 需要关联的表 |  
|localField|当前表关联的字段|
|foreignField| 关联表对应的字段|
|as| 存放关联集合的字段名 |

-------

### 操作符详细介绍：

#### $project  挑选字段，过滤字段
```javascript 
mongodb shell命令:
 
db.user.aggregate(
	{$project:{'name':1}}
)
    	
结果如下：
    	
{
	"_id" : ObjectId("57ec90a207a9bd3a458b456b"),
	"name" : "测试"
},
{
	"_id" : ObjectId("57ecd22207a9bd1c4e8b4567"),
	"name" : "gua"
}
```    	
    	
#### $match 过滤数据
```javascript 
mongodb shell命令:
     

db.user.aggregate(
	{$project:{name:1,type:1}},
	{$match:{type:'admin'}} // 过滤数据时，$project中需要有该数据得字段名；
)
    	
结果如下：
{
	"_id" : ObjectId("576b969cef38b39540ba3742"),
	"type" : "admin",
	"name" : "测试"
},
{
	"_id" : ObjectId("57edcd7007a9bd5b4e8b456b"),
	"name" : "测试",
	"type" : "admin"
}
```    	
 
#### $sort 排序

#### $skip 跳过文档数 

#### $limit 返回文档数



```javascript 
mongodb shell命令:
     
db.user.aggregate(
	{$project:{'name':1}},
	{$sort:{name:-1}},
	{$skip:0},
	{$limit:3}
)
	
	
结果如下：
{
	"_id" : ObjectId("585c9a3a07a9bd69108b4569"),
	"name" : "测试"
},
{
	"_id" : ObjectId("58e3912807a9bdcc1a8b4b43"),
	"name" : "测试"
},
{
	"_id" : ObjectId("58edc8b607a9bdcc1a8b4c9f"),
	"name" : "测试"
}        
```   
    
#### $unwind 将文档中的某一个数组类型字段拆分成多条数据
```javascript 
mongodb shell命令:
     

拆分前
    	
db.orderDriver.aggregate(
   	{$match:{orderNo:'XY161205054704199'}},
   	{$project:{orderNo:1,_id:1,statusRecord:1}}
)
    	
结果如下：
{
"_id" : ObjectId("5844ff8307a9bd2b1c8b4569"),
"orderNo" : "XY161205054704199",
"statusRecord" : [
   		[
   			NumberLong("100"),
   			NumberLong("1480916824")
   		],
   		[
   			NumberLong("200"),
   			NumberLong("1480916867")
   		],
   		[
   			NumberLong("300"),
   			NumberLong("1480916940")
   		],
   		[
   			NumberLong("400"),
   			NumberLong("1480916986")
   		],
   		[
   			NumberLong("500"),
   			NumberLong("1480916992")
   		],
   		[
   			NumberLong("600"),
   			NumberLong("1480916998")
   		]
   ]
}
    	
拆分后
	db.orderDriver.aggregate(
	   {$project:{orderNo:1,_id:1,statusRecord:1}},
	{$match:{orderNo:'XY161205054704199'}},
	{$unwind:'$statusRecord'} //statusRecord数组字段，此处需要在字段名前增加 $ 符号
)
    	
结果如下：
{
	"_id" : ObjectId("5844ff8307a9bd2b1c8b4569"),
	"orderNo" : "XY161205054704199",
	"statusRecord" : [
		NumberLong("100"),
		NumberLong("1480916824")
	]
},
{
	"_id" : ObjectId("5844ff8307a9bd2b1c8b4569"),
	"orderNo" : "XY161205054704199",
	"statusRecord" : [
		NumberLong("200"),
		NumberLong("1480916867")
	]
}
``` 	

#### $sample 随机返回指定文档数量 

```javascript 
mongodb shell命令:
     

db.user.aggregate(
	{$project:{name:1}},
	{$sample:{size:5}} //指定返回文档数量
)
	
结果如下：
{
	"_id" : ObjectId("585c9a3a07a9bd69108b4569"),
	"name" : "测试"
},
{
	"_id" : ObjectId("58e3912807a9bdcc1a8b4b43"),
	"name" : "测试"
},
{
	"_id" : ObjectId("58edc8b607a9bdcc1a8b4c9f"),
	"name" : "测试"
}
```   	
    
#### $group  分组 
    
    
##### $sum 计算总和
```javascript     	
mongodb shell命令:
     
db.user.aggregate(
	{$project:{name:1,type:1}},
	{$group:{
		_id:'$type',
		userName:{$sum:1}
	}}
)
    	
结果如下：
{
	"_id" : "admin",
	"userName" : 7
}
{
	"_id" : "attache",
	"userName" : 9
}
```
 	
        
##### `$min` `$max` 最小 最大值
```javascript     	
mongodb shell命令:
     

db.orderShipper.aggregate(
	{$project:{orderNo:1,shipperId:1,realTotalMoney:1}},
	{$group:{
		_id:'$shipper',
		realTotalMoney:{$max:'$realTotalMoney'}
	}}
)
    	
db.orderShipper.aggregate(
	{$project:{orderNo:1,shipperId:1,realTotalMoney:1}},
	{$group:{
		_id:'$shipper',
		realTotalMoney:{$min:'$realTotalMoney'}
	}}
)
    	
结果如下：
    	
$max 最大
{
	"_id" : ObjectId("58d0989007a9bdce1a8b49e3"),
	"realTotalMoney" : 47200
},
$min 最小
{
	"_id" : ObjectId("58d0989007a9bdce1a8b49e3"),
	"realTotalMoney" : 555
}
```
    
    
##### $push 在结果文档中插入值到一个数组中
```javascript     	
mongodb shell命令:
     

db.orderShipper.aggregate(
	{$project:{orderNo:1,shipperId:1,realTotalMoney:1}},
	{$group:{
		_id:'$shipperId',
		realTotalMoney:{$push:'$realTotalMoney'}
	}}
)
    	
结果如下：
{
	"_id" : ObjectId("58d0989007a9bdce1a8b49e3"),
	"realTotalMoney" : [
		555,
		1400,
		2100,
		47200
	]
}
```

##### $first 根据资源文档的排序获取第一个文档数据
```javascript     
mongodb shell命令:
     

db.orderShipper.aggregate(
	{$project:{orderNo:1,shipperId:1,realTotalMoney:1}},
	{$group:{
		_id:'$shipperId',
		realTotalMoney:{$first:'$realTotalMoney'}
	}}
)
    	
结果如下：
{
	"_id" : ObjectId("58d0989007a9bdce1a8b49e3"),
	"realTotalMoney" : 555
}
```    
    
##### $last 根据资源文档的排序获取最后一个文档数据
```javascript         
mongodb shell命令:
     

db.orderShipper.aggregate(
	{$project:{orderNo:1,shipperId:1,realTotalMoney:1}},
	{$group:{
		_id:'$shipperId',
		realTotalMoney:{$last:'$realTotalMoney'}
	}}
)
    	
结果如下：
{
	"_id" : ObjectId("58d0989007a9bdce1a8b49e3"),
	"realTotalMoney" : 47200
}
```
    
#### $lookup 连接查询相关操作介绍
    
```javascript     
mongodb shell命令:
     

db.orderShipper.aggregate([
   {$limit:1},
	{$lookup:{
		from:'user',
		localField:'shipperId',
		foreignField:'_id',
		as : 'userInfo'
	}},
	{$project:{shipperId:1,orderNo:1,'userInfo._id':1,'userInfo.name':1,'userInfo.phone':1}}
])
    	
结果如下：
{
"_id" : ObjectId("58193bce07a9bdc8258b4576"),
"shipperId" : ObjectId("58193ad807a9bdc8258b4574"),
"orderNo" : "XY161102010518181",
"userInfo" : [
    	{
    		"_id" : ObjectId("58193ad807a9bdc8258b4574"),
    		"phone" : "18357132821",
    		"name" : "测试"
    	}
    ]
}
```    
    
    
#### $facet 在同一组输入文档的同一阶段内处理多个聚合管道
```javascript 
mongodb shell命令:
     
查询4月份所有交易员名下已确认送达订单收款，付款情况
var star = new Date("2017-04-01 00:00:00").getTime() /1000
var end  = new Date("2017-04-31 23:59:59").getTime() /1000
    
db.user.aggregate([
   {
       $facet:{
           "no" :[
               {$match:{ 
                   type:'sched'}
               },
               {$lookup:{
                   from:'orderShipper',
                   localField:'_id',
                   foreignField: 'schedulerId',
                   as: 'orderList'
               }},
               {$unwind:'$orderList'},
               {$match:{ 
                   'orderList.status':700,
                   'orderList.finishTime':{'$gte':star,'$lt':end}
                   
               }},
               {$group:{
                   _id:'$name',
                   dd:{$push:'$orderList.orderNo'},
                   shipperOrder:{$push:{status:'$orderList.receiveMoneyTime',mong:'$orderList.realTotalMoney'}}
               }},
               {$unwind:'$shipperOrder'},
               {$match:{
                   'shipperOrder.status':{'$in':[0,1]}
                   
               }},
               {$group:{
                   _id:'$_id',
                   shipperCnt:{$sum:1},
                   orderNos:{$first:'$dd'},
                   shipperMoney:{$sum:'$shipperOrder.mong'}
                   
               }},
               {$unwind:'$orderNos'},
               {$lookup:{
                   from:'orderDriver',
                   localField:'orderNos',
                   foreignField: 'orderNo',
                   as: 'driverList'
               }},
               {$unwind:'$driverList'},
               {$match:{'driverList.win':true,'driverList.payedStatus':{'$in':[0,1]}}},
               {$group:{
                   _id:'$_id',
                   shipperCnt:{$first:'$shipperCnt'},
                   shipperMoney:{$first:'$shipperMoney'},
                   driverCnt:{$sum:1},
                   driverrMoney:{$sum:'$driverList.realTotalMoney'}
               }}
           ],
           'yes' : [
               {$match:{ 
                   type:'sched'}
               },
               {$lookup:{
                   from:'orderShipper',
                   localField:'_id',
                   foreignField: 'schedulerId',
                   as: 'orderList'
               }},
               {$unwind:'$orderList'},
               {$match:{ 
                   'orderList.status':700,
                   'orderList.finishTime':{'$gte':star,'$lt':end}
               }},
               {$group:{
                   _id:'$name',
                   dd:{$push:'$orderList.orderNo'},
                   shipperOrder:{$push:{status:'$orderList.receiveMoneyTime',mong:'$orderList.realTotalMoney'}}
               }},
               {$unwind:'$shipperOrder'},
               {$match:{
                   'shipperOrder.status':{'$eq':2}
                   
               }},
               {$group:{
                   _id:'$_id',
                   shipperCnt:{$sum:1},
                   orderNos:{$first:'$dd'},
                   shipperMoney:{$sum:'$shipperOrder.mong'}
               }},
               {$unwind:'$orderNos'},
               {$lookup:{
                   from:'orderDriver',
                   localField:'orderNos',
                   foreignField: 'orderNo',
                   as: 'driverList'
               }},
               {$unwind:'$driverList'},
               {$match:{'driverList.win':true,'driverList.payedStatus':{'$eq':2}}},
               {$group:{
                   _id:'$_id',
                   shipperCnt:{$first:'$shipperCnt'},
                   shipperMoney:{$first:'$shipperMoney'},
                   driverCnt:{$sum:1},
                   driverrMoney:{$sum:'$driverList.realTotalMoney'}
               }}
           ]
       }
       
   }
  ]
)
    
结果如下：
{
"no" : [
	{
		"_id" : "测试1",
		"shipperCnt" : 6,
		"shipperMoney" : 32577.16,
		"driverCnt" : 8,
		"driverrMoney" : 31119.45
	},
	{
		"_id" : "测试2",
		"shipperCnt" : 6,
		"shipperMoney" : 25822.260000000002,
		"driverCnt" : 6,
		"driverrMoney" : 24025.25
	},
	{
		"_id" : "测试3",
		"shipperCnt" : 11,
		"shipperMoney" : 22427.49,
		"driverCnt" : 11,
		"driverrMoney" : 18925.45
	},
	{
		"_id" : "测试4",
		"shipperCnt" : 3,
		"shipperMoney" : 14389.54,
		"driverCnt" : 3,
		"driverrMoney" : 13135
	},
	{
		"_id" : "测试5",
		"shipperCnt" : 42,
		"shipperMoney" : 117065.02,
		"driverCnt" : 43,
		"driverrMoney" : 105611.175
	},
	{
		"_id" : "测试6",
		"shipperCnt" : 44,
		"shipperMoney" : 118917.04,
		"driverCnt" : 44,
		"driverrMoney" : 102525.62
	},
	{
		"_id" : "测试7",
		"shipperCnt" : 16,
		"shipperMoney" : 36755.04,
		"driverCnt" : 17,
		"driverrMoney" : 34193.12
	},
	{
		"_id" : "测试8",
		"shipperCnt" : 9,
		"shipperMoney" : 29653.12,
		"driverCnt" : 10,
		"driverrMoney" : 28830.16
	}
],
"yes" : [
	{
		"_id" : "测试1",
		"shipperCnt" : 6,
		"shipperMoney" : 39822.53,
		"driverCnt" : 2,
		"driverrMoney" : 14532.880000000001
	},
	{
		"_id" : "测试2",
		"shipperCnt" : 2,
		"shipperMoney" : 3347.4,
		"driverCnt" : 1,
		"driverrMoney" : 700
	},
	{
		"_id" : "测试3",
		"shipperCnt" : 2,
		"shipperMoney" : 2800,
		"driverCnt" : 2,
		"driverrMoney" : 3544.05
	}
 ]
}
```    

#### $replaceRoot 用指定的嵌入文档替换文档。操作代替输入文档中的现有的所有领域，包括_id场。指定嵌入在输入文档中的文档，以将嵌入式文档提升到顶层

```javascript 
原数据：
{
	"_id" : ObjectId("58193bce07a9bdc8258b4576"),
	"shipperId" : ObjectId("58193ad807a9bdc8258b4574"),
	"orderNo" : "XY161102010518181",
	"userInfo" : {
		"_id" : ObjectId("58193ad807a9bdc8258b4574"),
		"phone" : "xxxx1111",
		"name" : "测试"
	}
},
{
	"_id" : ObjectId("5823e79707a9bd72288b4569"),
	"shipperId" : ObjectId("5822778107a9bdd2148b456c"),
	"orderNo" : "XY161110032055107",
	"userInfo" : {
		"_id" : ObjectId("5822778107a9bdd2148b456c"),
		"phone" : "xxxx1111",
		"name" : "测试1"
	}
}
```

```javascript	
mongodb shell命令:
	
db.orderShipper.aggregate([
   {$limit:2},
	{$lookup:{
		from:'user',
		localField:'shipperId',
		foreignField:'_id',
		as : 'userInfo'
	}},
	{$project:{shipperId:1,orderNo:1,'userInfo._id':1,'userInfo.name':1,'userInfo.phone':1}},
	{$unwind:"$userInfo"},
	{$replaceRoot:{newRoot:'$userInfo'}}
	
])
	

结果如下：
{
	"_id" : ObjectId("58193ad807a9bdc8258b4574"),
	"phone" : "xxxx1111",
	"name" : "测试1"
},
{
	"_id" : ObjectId("5822778107a9bdd2148b456c"),
	"phone" : "xxxx1111",
	"name" : "测试2"
} 
```


### 实例

下面是运用在项目中的实列：
以用户的类型分配对应的功能模块，实现动态地增减功能模块，对于多级模块，用到了无限级分类方法，在获取数据时，将对应的模块嵌套其中，最后生成功能导航条


#### 表结构：
```javascript 
authPrivilege表
{
"_id" : ObjectId("59588f1fc7f9878b070041c3"),
"name" : "审核流程",
"urlPath" : "/process/order-web/review",
"parentId" : NumberLong("0"),
"sortNumber" : NumberLong("1"),
"icoPath" : "glyphicon glyphicon-send",
"privilegeId" : NumberLong("1"),
"submitTime" : NumberLong("1498976031")
},
{
"_id" : ObjectId("59589039c7f9875a03004241"),
"name" : "顺风车",
"urlPath" : "/process/order-web/tail-wind",
"parentId" : NumberLong("0"),
"sortNumber" : NumberLong("4"),
"icoPath" : "glyphicon glyphicon-transfer",
"privilegeId" : NumberLong("6"),
"submitTime" : NumberLong("1498976313")
},
    
authRolePrivilege表
{
"_id" : ObjectId("595a0765c7f9875d03004357"),
"privilegeId" : [
	NumberLong("2"),
	NumberLong("12"),
	NumberLong("13"),
	NumberLong("14"),
	NumberLong("15"),
	NumberLong("17"),
	NumberLong("37"),
	NumberLong("38"),
	NumberLong("40"),
	NumberLong("41")
],
"type" : "attache",
"submitTime" : NumberLong("1499072357")
}
```
    
#### php代码：
 PHP框架  YII2
 详情介绍在这里 ： [http://www.yiiframework.com/doc-2.0/yii-mongodb-collection.html](http://www.yiiframework.com/doc-2.0/yii-mongodb-collection.html)
   
    
    以用户类型查询到该类型所拥有的模块，其中关联了2张表，
    authRolePrivilege表 对应该类型拥有那些模块的ID,是以数组形式存入模块ID
    authPrivilege表 模块信息
       
    关联字段为 
    user表中的type 对应authRolePrivilege表中 type
    authRolePrivilege表中的 privilegeId 对应 authPrivilege表 privilegeId
       
    mongodb\ActiveQuery中无法使用aggregate方法，需要在mongodb\Collection才能使用
    
```php
$userRols = User::getCollection()->aggregate(
      ['$match' => [
          '_id'=> MongoId::__set_state(array(
              '$id' => '59588f1fc7f9878b070041c3',
           )) // 查询该用户ID的数据
      ]], 
      ['$lookup' => [
          'from' => 'authRolePrivilege', //需要关联的表名
          'localField' => 'type', // 当前表中关联字段
          'foreignField' => 'type', // 关联表中对应的字段
          'as' => 'roles', // 存放关联集合的字段名
      ]],
      ['$unwind'=>'$roles'], // 将关联集合的字段展开（类似循环，把数组里的没有个元素，循环出来）
      ['$unwind'=>'$roles.privilegeId'],// 将关联集合中的privilegeId字段展开，用于关联模块表使用
      ['$lookup' => [
          'from' => 'authPrivilege', // 模块表
          'localField' => 'roles.privilegeId', //当前表中关联字段
          'foreignField' => 'privilegeId',// 关联表中对应的字段
          'as' => 'list' // 存放关联集合的字段名
    
      ]],
      ['$unwind'=>'$list'], // 将结果展开 （这里需要在字段名前加上 '$' 符）
      ['$group'=>[
          '_id' => '$_id', // 分组以那个一段分组，这里的_ID 是用户ID （这里的字段名_id,为固定值，不能自定义） 这里需要在字段名前加上 '$' 符）
          'list' => ['$push'=>'$list'] // 将分组结果的数据插入到一个数组中 （这里的字段名list 可以自定义） 这里需要在字段名前加上 '$' 符）
      ]],
      ['$unwind'=>'$list'], //将分组号的数据展开
      ['$sort'=>['list.sortNumber' => 1]], //对关联到模块表用数据，排序
      ['$unwind'=>'$list'],
      ['$replaceRoot'=>['newRoot'=>'$list']] // 将每一个list里面的数据提到顶层
  ); 
 ```      
 
 
  
        
#### 输出结果：
 ```javascript       
 array (
     0 => 
     array (
       '_id' => 
       MongoId::__set_state(array(
          '$id' => '59588f1fc7f9878b070041c3',
       )),
       'name' => '审核流程',
       'urlPath' => '/process/order-web/review',
       'parentId' => 0,
       'sortNumber' => 1,
       'icoPath' => 'glyphicon glyphicon-send',
       'privilegeId' => 1,
       'submitTime' => 1498976031,
     ),
     1 => 
     array (
       '_id' => 
       MongoId::__set_state(array(
          '$id' => '59588f51c7f9876e03004239',
       )),
       'name' => '发货',
       'urlPath' => '/process/order-web/deliver-goods',
       'parentId' => 0,
       'sortNumber' => 2,
       'icoPath' => 'glyphicon glyphicon-plus-sign',
       'privilegeId' => 2,
       'submitTime' => 1498976081,
     ),
     2 => 
     array (
       '_id' => 
       MongoId::__set_state(array(
          '$id' => '59588f6fc7f987570300423a',
       )),
       'name' => '询价列表',
       'urlPath' => '',
       'parentId' => 0,
       'sortNumber' => 3,
       'icoPath' => 'glyphicon glyphicon-eye-open',
       'privilegeId' => 3,
       'submitTime' => 1498976111,
     ),
     3 => 
     array (
       '_id' => 
       MongoId::__set_state(array(
          '$id' => '59588ff1c7f98746070041ce',
       )),
       'name' => '在线询价',
       'urlPath' => '/process/order-web/order-inquiry',
       'parentId' => 3,
       'sortNumber' => 3.1000000000000001,
       'icoPath' => '',
       'privilegeId' => 4,
       'submitTime' => 1498976241,
     ),
     4 => 
     array (
       '_id' => 
       MongoId::__set_state(array(
          '$id' => '5958900ec7f9875e070041c9',
       )),
       'name' => '语音询价',
       'urlPath' => '/process/order-web/order-inquiry?type=app',
       'parentId' => 3,
       'sortNumber' => 3.2000000000000002,
       'icoPath' => '',
       'privilegeId' => 5,
       'submitTime' => 1498976270,
     )
   )
```



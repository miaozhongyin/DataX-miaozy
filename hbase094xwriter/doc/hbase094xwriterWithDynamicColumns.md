# Hbase094XWriterWithDynamicColumns 动态列模式插件文档说明
* 新增及修改功能
#1， 动态列模式是基于Normal模式改进，暂时支持将所有的列插入到一个列簇中。
#2，在生成rowkey方面支持单一主健和联合rowkey，其中单一主健有列值构成，符合主健由列值和常量字符构成
#3，增加主健MD5加密方式，
#4，新增NULL值自定义处理方式


### 3.1 配置样例

* 配置一个从本地写入hbase1.1.x的作业：

```
{
  "job": {
    "setting": {
      "speed": {
        "channel": 5
      }
    },
    "content": [
      {
        "reader": {
          "name": "txtfilereader",
          "parameter": {
            "path": "/Users/shf/workplace/datax_test/hbase11xwriter/txt/normal.txt",
            "charset": "UTF-8",
            "column": ["*"],
            "fieldDelimiter": ","
          }
        },
        "writer": {
          "name": "hbase11xwriter",
          "parameter": {
            "hbaseConfig": {
              "hbase.rootdir": "hdfs: //ip: 9000/hbase",
              "hbase.cluster.distributed": "true",
              "hbase.zookeeper.quorum": "***"
            },
            "table": "writer",
            "mode": "dynamic",
            "rowkeyColumn": [
                {
                  "index":0,
                  "type":"string"
                },
                {
                 "index":1,
                  "type":"string"
                },
                {
                  "index":-1,
                  "type":"string",
                  "value":"_"
                }
            ],
            "column": [ "familyName":"cf"],
            "versionColumn":{
              "index": -1,
              "value":"123456789"
            },
            "encoding": "utf-8",
            "nullMode":{
               "mode":"comstorm",
               "replaceValue":""
             }
            }
          }
        }
      }
    ]
  }
}
```


### 3.2 参数说明

* **hbaseConfig**

	* 描述：每个HBase集群提供给DataX客户端连接的配置信息存放在hbase-site.xml，请联系你的HBase PE提供配置信息，并转换为JSON格式。同时可以补充更多HBase client的配置，如：设置scan的cache、batch来优化与服务器的交互。
 
	* 必选：是 <br />
 
	* 默认值：无 <br />
 
* **mode**
 
	* 描述：写hbase的模式，目前只支持normal ,dynamic模式，<br />
 
	* 必选：是 <br />
 
	* 默认值：无 <br />
	
* **table**
 
	* 描述：要写的 hbase 表名（大小写敏感） <br />
 
	* 必选：是 <br />
 
	* 默认值：无 <br />

* **encoding**

	* 描述：编码方式，UTF-8 或是 GBK，用于 String 转 HBase byte[]时的编码 <br />
 
	* 必选：否 <br />
 
	* 默认值：UTF-8 <br />
 
  
* **column**

	* 描述：要写入的hbase字段,在dynamic模式中默认将reader数据源的所有值写到familyName列簇中。
"column": [ "familyName":"cf" ］
		            
	```

	* 必选：是<br />
 
	* 默认值：无 <br />

* **rowkeyColumn**

	* 描述：要写入的hbase的rowkey列。index：指定该列对应reader端column的索引，从0开始，常量index为－1标识复合主健拼接字符，常量index为－2标识所有列簇为主健,常量index为－3标识加密方式（默认不加密）；type：指定写入数据类型，用于转换HBase byte[]；value：配置常量，常作为多个字段的拼接符。hbasewriter会将rowkeyColumn中所有列按照配置顺序进行拼接作为写入hbase的rowkey，不能全为常量。
	* 新增功能： 支持对拼接后的rowkey进行加密，暂时支持对MD5加密方式。（强烈建议对复合主健进行加密，因为如果后期有增量数据同步功能，一但复合主健值被更新，将无法同步该记录）
	配置格式如下：

    * 单一主健
    rowkeyColumn": [
                {
                  "index":0,
                  "type":"string"
                },
                {
                   "index":-3,
                   "value":""
                },
                {
                  "index":-1,
                  "type":"string",
                  "value":"_"
                }
            ]
    * 复合主健 （1）
   "rowkeyColumn": [
                    {
                      "index":0,
                      "type":"string"
                    },
                    {
                     "index":3,
                      "type":"string"
                    },
                    {
                      "index":-1,
                      "type":"string",
                      "value":"_"
                    }
                ]
	* 复合主健 （2）
    "rowkeyColumn": [
                    {
                       "index":-2,
                    },
                    {
                       "index":-3,
                       "value":"MD5"
                    },
                    {
                        "index":-1,
                        "type":"string",
                        "value":"_"
                    }
                    ]


	* 必选：是<br />
 
	* 默认值：无 <br />
	
* **versionColumn**

	* 描述：指定写入hbase的时间戳。支持：当前时间、指定时间列，指定时间，三者选一。若不配置表示用当前时间。index：指定对应reader端column的索引，从0开始，需保证能转换为long,若是Date类型，会尝试用yyyy-MM-dd HH:mm:ss和yyyy-MM-dd HH:mm:ss SSS去解析；若为指定时间index为－1；value：指定时间的值,long值。配置格式如下：
	
	```
"versionColumn":{
	"index":1
}
		            
	```
	
	或者
	
	```
"versionColumn":{
	"index":－1,
	"value":123456789
}
		            
	```

	* 必选：否<br />
 
	* 默认值：无 <br />
	

* **nullMode**

	* 描述：读取的null值时，如何处理。支持两种方式：（1）skip：表示不向hbase写这列；（2）empty：写入HConstants.EMPTY_BYTE_ARRAY，即new byte [0] ，（3）comstorm自定义替换值：该值为字符串<br />
	  
	* 必选：否<br />
 
	* 默认值：skip<br />	

* **walFlag**

	* 描述：在HBae client向集群中的RegionServer提交数据时（Put/Delete操作），首先会先写WAL（Write Ahead Log）日志（即HLog，一个RegionServer上的所有Region共享一个HLog），只有当WAL日志写成功后，再接着写MemStore，然后客户端被通知提交数据成功；如果写WAL日志失败，客户端则被通知提交失败。关闭（false）放弃写WAL日志，从而提高数据写入的性能。<br />
	  
	* 必选：否<br />
 
	* 默认值：false<br />

* **writeBufferSize**

	* 描述：设置HBae client的写buffer大小，单位字节。配合autoflush使用。autoflush，开启（true）表示Hbase client在写的时候有一条put就执行一次更新；关闭（false），表示Hbase client在写的时候只有当put填满客户端写缓存时，才实际向HBase服务端发起写请求<br />
	  
	* 必选：否<br />
 
	* 默认值：8M<br />

### 3.3 HBase支持的列类型
* BOOLEAN
* SHORT
* INT
* LONG
* FLOAT
* DOUBLE
* STRING



请注意:

* `除上述罗列字段类型外，其他类型均不支持`。

## 4 性能报告

略

## 5 约束限制

略

## 6 FAQ

***

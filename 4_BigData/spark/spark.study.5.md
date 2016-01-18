#spark学习笔记5


## spark && hbase





## before
本文主要记录spark的一些实践

* 1. order_info表读取进来转化为 order class
* 2. merge_order_info表读取进来 转化为 order class
* 3. 将order_info和pay_info表merge起来(方式多样: 多条支付纪录merge成list；独立成多条数据)



spark－shell 连接集群：

```
m1: spark-shell --master spark://master:7077
 
m2: MASTER=spark://master:7077   ./bin/spark-shell

/data/server/spark-1.5.1/bin/spark-shell --master spark://master:7077 --driver-memory 10G --executor-memory 5G --executor-cores 1 --total-executor-cores 20
```


```
val fileName = "/user/spark/basedata/wd/vshop_order_info/daily_export.wd.order_info.20151101"

val textFile = sc.textFile(fileName)
val orderRDD = textFile.map(one => scala.util.parsing.json.JSON.parseFull(one).get.asInstanceOf[Map[String, Any]])
```






val order_info_0 = sqlContext.read.json(fileName)
order_info_0.printSchema()



sqlContext.read.json("/user/spark/basedata/wd/vshop_order_info/daily_export.wd.order_info.20150915").printSchema()


订单数据操作记录

```
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

val order_info_0 = sqlContext.read.json("/user/spark/basedata/wd/vshop_order_info/daily_export.wd.order_info.20151115")

val pay_info = sqlContext.read.json("/user/spark/basedata/wd/vshop_pay_info/daily_export.wd.pay_info.20151115")

val tostr: (Any => String) = (arg: Any) => arg.toString
val sqlToStr = udf(tostr)

//将order表中的orderid转成 str
val order_info = order_info_0.withColumn("str_id", sqlToStr(col("order_id")))
 val order_info_1 = order_info_0.withColumn("str_order_id", sqlToStr(order_info_0("order_id"))).withColumn("str_seller_id", sqlToStr(order_info_0("seller_id")))



val order_merge_info = order_info.join(pay_info, order_info("str_id") === pay_info("order_ids"))

// 慎用，占用内存
// order_merge_info.cache()  

order_merge_info.count()

order_merge_info.select("order_id", "buyer_id", "seller_id", "third_userid").show(100)


val order_filter_zfb = order_merge_info.filter("third_userid is not null").filter("pay_type=2").select("order_id", "buyer_id", "seller_id", "pay_type", "third_userid")


order_filter_zfb.groupBy("third_userid").count().orderBy(desc("count")).show()


val third_userid_count = order_filter_zfb.groupBy("third_userid").count()



df.write.mode("append").json(/user/vc/yangliang)


import org.apache.spark.sql.functions._

df.orderBy(asc("col1"))
df.sort(desc("col1"))
```


聚合操作

```
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

val order_info_0 = sqlContext.read.json("/user/spark/basedata/wd/vshop_order_info/daily_export.wd.order_info.20151115")

val tostr: (Any => String) = (arg: Any) => arg.toString
val sqlToStr = udf(tostr)


val order_info_1 = order_info_0.withColumn("str_order_id", sqlToStr(col("order_id"))).withColumn("str_seller_id", sqlToStr(col("seller_id")))

filter condition[订单号非0，order_status有效(= 2 or 3)]

c1: 有效订单数>=10   
c2: 有效订单数(小于10RMB的在线支付+大于500RMB的货到付款) / 总有效订单数 >= 70%

c1: count(buyer_id)>=10
c2: count([price<=10 && tag in set(2,3)] or [price>=500 && tag in set(1)]) / count(buyer_id) >= 0.7



val order_info = order_info_1.select("order_id", "buyer_id", "str_seller_id").show(100)

dataframe -> rdd

val order_info_rdd = order_info.map(x => (x.getString(1), x.getString(0)))
val order_info_rdd = order_info.map(x => (x.getString(1), (x.getString(0), x.getString(2))))

val buyer_orders = order_info_rdd.groupByKey()

val buyer_orders_count = buyer_orders.map {case (a, b) => (a, b.size)}

order_info_rdd.groupByKey().map( row => (row._1, row._2.size)) 


getInt(2) 
```


```
import org.apache.spark.sql.types.{StructType,StructField,StringType}
import scala.collection.mutable

   val schema: StructType = StructType(mutable.ArraySeq(

      StructField("order_id", IntegerType, true),

      StructField("seller_id", StringType, true)));
      
      
     val fileName = "/user/spark/basedata/wd/vshop_order_info/daily_export.wd.order_info.20151116"
      
    val jsonFile = sqlContext.read.schema(schema).json(fileName)

    jsonFile.printSchema()
```

map filter

x.retain((k,v) => k > 1)




15号订单数据：

* 总订单：263790
* order_id=0: 22184

* 有效订单:181749
* 相关买家数：134751




|编号|字段名称|含义|
|:--:|:--:|:--:|
|0|_id||
|1|browser||
|2|buyer_address||
|3|buyer_id||
|4|buyer_ip||
|5|buyer_name||
|6|||
|7|||
|8|||

|6|item_list||
|7|level||
|8|mer_id||
|9|order_add_time||
|10|order_id||
|11|order_pay_time||
|12|order_status||

```
val order_info = sc.textFile("/user/vc/vshop/vshop_order_info.csv")

scala> val rdd1 =  order_info.filter(line => line.contains("325724246"))

val rdd1 =  order_info.filter(line => line.contains("325724246"))
rdd1.cache()
```


# Zeppelin学习
```
val orders = sqlContext.read.json("/user/spark/basedata/wd/vshop_order_info/daily_export.wd.order_info.2015122*")
orders.registerTempTable("orderSql")
select seller_id, count(*) from orderSql  group by seller_id

```


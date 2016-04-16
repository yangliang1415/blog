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




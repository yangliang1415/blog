# spark学习笔记1


本文主要介绍spark在mac下的安装和简单开发环境搭建，以及简单例子的运行。


## spark开发环境搭建
开发环境的搭建，我主要参考的`这里`。

## spark启动

spark的启动，在spark主目录中
>* 运行./sbin/spark-all.sh，启动spark
>* 输入jps命令(查看java相关进程)，观察到Maskter和Worker两个进程，说明spark启动成功
>* 使用spark shell测试，运行./bin/spark-shell，就可以和spark简单的交互编程了。可以用官网上的一个简单例子测试如下

```
scala> val textFile = sc.textFile("README.md")
textFile: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[1] at textFile at <console>:21

scala> textFile.count()
res0: Long = 9

scala> textFile.first()
res2: String = # Apache Spark
```
## spark任务编写
spark官网已经为我们写了很多expample program，这里先运行样例程序，然后自己编写简单任务。

bin目录下面的spark-submit主要就是用于提交我们的任务的，一个例子如下：

```
./bin/spark-submit  examples/src/main/python/logistic_regression.py  data/mllib/lr_data.txt  100
```
其中lr_data.txt文件是训练数据，格式是：label v1 v2 v3 ... v10

下面我们对`logistic_regression.py`文件简要分析

```
"""
 A logistic regression implementation that uses NumPy (http://www.numpy.org)
 to act on batches of input data using efficient matrix operations.
 
 In practice, one may prefer to use the LogisticRegression algorithm in
 MLlib, as shown in examples/src/main/python/mllib/logistic_regression.py.
 """
 from __future__ import print_function
 
 import sys
 
 import numpy as np
 from pyspark import SparkContext
 
 
 D = 10  # Number of dimensions
 
 # Read a batch of points from the input file into a NumPy matrix object. We operate on batches to
 # make further computations faster.
 # The data file contains lines of the form <label> <x1> <x2> ... <xD>. We load each block of these
 # into a NumPy array of size numLines * (D + 1) and pull out column 0 vs the others in gradient().
 def readPointBatch(iterator):
     strs = list(iterator)
     matrix = np.zeros((len(strs), D + 1))
     for i, s in enumerate(strs):
         matrix[i] = np.fromstring(s.replace(',', ' '), dtype=np.float32, sep=' ')
     return [matrix]

 if __name__ == "__main__":
 
     if len(sys.argv) != 3:
         print("Usage: logistic_regression <file> <iterations>", file=sys.stderr)
         exit(-1)
 
     print("""WARN: This is a naive implementation of Logistic Regression and is
       given as an example! Please refer to examples/src/main/python/mllib/logistic_regression.py
       to see how MLlib's implementation is used.""", file=sys.stderr)
 
     sc = SparkContext(appName="PythonLR")
     points = sc.textFile(sys.argv[1]).mapPartitions(readPointBatch).cache()
     iterations = int(sys.argv[2])
 
     # Initialize w to a random value
     w = 2 * np.random.ranf(size=D) - 1
     print("Initial w: " + str(w))
     
          # Compute logistic regression gradient for a matrix of data points
     def gradient(matrix, w):
         Y = matrix[:, 0]    # point labels (first column of input file)
         X = matrix[:, 1:]   # point coordinates
         # For each point (x, y), compute gradient function, then sum these up
         return ((1.0 / (1.0 + np.exp(-Y * X.dot(w))) - 1.0) * Y * X.T).sum(1)
 
     def add(x, y):
         x += y
         return x
 
     for i in range(iterations):
         print("On iteration %i" % (i + 1))
         w -= points.map(lambda m: gradient(m, w)).reduce(add)
 
     print("Final w: " + str(w))
 
     sc.stop()
```

* 首先`sc.textFile(sys.argv[1])`依赖lr_data.txt文件定义了一个RDD，然后通过parsePoint函数转换定义了另外一个RDD points，并将points这个RDD缓存在内存中。(**原因**: 由于在迭代计算更新梯度时，需要用到全部的points，这里将points缓存在内存中，可以大大加快计算速度。)

* 接着定义并随机初始化`权重w`，定义`梯度计算函数`。

* 最后不断迭代更新`权重w`，直到达到设定的迭代次数上限。

	```
for i in range(iterations):
	print("On iteration %i" % (i + 1))
	w -= points.map(lambda m: gradient(m, w)).reduce(add)
	```


## Ref
* ref1: http://www.cnblogs.com/yangmu/p/4216102.html




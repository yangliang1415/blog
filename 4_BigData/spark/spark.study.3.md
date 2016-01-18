#spark学习笔记3

本文主要介绍spark的机器学习库`MLlib`，该库实现了一些常用的机器学习算法，包含`分类`、`回归`、`聚类`、`协同过滤`、`降维`等几大类。`MLlib`被分为两个`packages`:

* `mllib`: 建立在`RDD`基础上的各种原生API 
* `ml`: 提供 higher_level 的 API，建立在 DataFrames 基础上，致力于打造 ML pipelines 

这里我们主要分析 mllib package，首先将 mllib 按照各个目录分类内容分类如下

|目录|内容|详情|
|:-:|:-:|:-:|
| classification|分类算法| 
|tree|决策树算法|
|recommendation|推荐算法|
|clustering|聚类算法|
|regression|回归算法|
|optimization|最优化算法|
|feature|数据预处理模块|
|evaluation|算法效果衡量模块|
|stat|统计模块|
|util|常用工具模块|



## 0. 模型算法的各种基类关系
回归分类不分家。文件 regression/GeneralizedLinearAlgorithm.scala定义了2个虚基类:

* GeneralizedLinearModel(广义线性模型)：数据包含一个权重向量和常数项；方法包含两个预测函数，分别用于单点预测和批量预测。

	```
 abstract class GeneralizedLinearModel(val weights: Vector, val intercept: Double)
   extends Serializable {
   // 单点预测函数
   protected def predictPoint(dataMatrix: Vector, weightMatrix: Vector, intercept: Double): Double
   // 在testData数据集预测
    def predict(testData: RDD[Vector]): RDD[Double] = ｛｝
   }
	```

* GeneralizedLinearAlgorithm(广义线性算法)：数据包含一个优化算法求解器optimizer；方法包含模型训练方法run，实际就是根据数据input和initialWeights，通过优化算法optimizer来求解得到最终的model。

```
 abstract class GeneralizedLinearAlgorithm[M <: GeneralizedLinearModel]
   extends Logging with Serializable {
   /** The optimizer to solve the problem. */
   def optimizer: Optimizer
   def run(input: RDD[LabeledPoint]): M = {｝
   def run(input: RDD[LabeledPoint], initialWeights: Vector): M = {}
   }
```

文件 classfication/ClassificationModel.scala中定义了 分类算法模型ClassificationModel

	``` 
	trait ClassificationModel extends Serializable {
	   // 数据集预测
	   def predict(testData: RDD[Vector]): RDD[Double]
	   // 单点预测
	   def predict(testData: Vector): Double
	```



## 1.分类算法

分类模块下面有以下文件

* ClassificationModel.scala: 分类算法模型基类
* LogisticRegression.scala: LR算法模型和LR算法的实现
* NaiveBayes.scala: NB算法模型和LR算法的实现
* SVM.scala: SVM算法模型和LR算法的实现
* StreamingLogisticRegressionWithSGD.scala: 流式的LR
* impl/GLMClassificationModel.scala: GLM分类模型的导入导出



下面从最熟悉的分类算法入手，结合 **算法和代码实例** 来通读代码。
classification下面实现了 `LogisticRegression`、`NaiveBayes`、`SVM`三种常见的分类算法。

### LR算法
首先分析LogisticRegression算法的实现。

下面是 **LR模型** LogisticRegressionModel 的声明，数据主要包含一个权重向量 weights 和常数项 intercept; 方法主要包含 predictPoint。

```
 class LogisticRegressionModel (
     override val weights: Vector,
     override val intercept: Double,
     val numFeatures: Int,
     val numClasses: Int)
   extends GeneralizedLinearModel(weights, intercept) with ClassificationModel with Serializable
   with Saveable with PMMLExportable{
   
     override protected def predictPoint(
       dataMatrix: Vector,
       weightMatrix: Vector,
       intercept: Double) = {
     require(dataMatrix.size == numFeatures)
 
       if (numClasses == 2) {
         // 2分类预测
         val margin = dot(weightMatrix, dataMatrix) + intercept
         val score = 1.0 / (1.0 + math.exp(-margin))
         threshold match {
         case Some(t) => if (score > t) 1.0 else 0.0
         case None => score
       }
     } else {
      // 多分类...
     }
     // 其他方法的实现...
}
```

接下来是 **LR算法** 的实现，根据LR的求解算法的不同(SGD、LBFGS)，将LR分为两种: `LogisticRegressionWithSGD``LogisticRegressionWithLBFGS`。

如下是运用随机梯度下降算法 SGD 求解的 `LogisticRegressionWithSGD` 算法声明。

  ```
 class LogisticRegressionWithSGD private[mllib] (
     private var stepSize: Double,
     private var numIterations: Int,
     private var regParam: Double,
     private var miniBatchFraction: Double)
   extends GeneralizedLinearAlgorithm[LogisticRegressionModel] with Serializable {
   // 算法成员声明：梯度计算；更新函数；优化目标； 
   private val gradient = new LogisticGradient()
   private val updater = new SquaredL2Updater()
   override val optimizer = new GradientDescent(gradient, updater)
     .setStepSize(stepSize)
     .setNumIterations(numIterations)
     .setRegParam(regParam)
     .setMiniBatchFraction(miniBatchFraction）
     // 其他方法的实现...
  }
  ```

声明 `LogisticRegressionWithSGD` object，模型训练的逻辑主要在 run函数中，该函数的声明实现在 虚类 `GeneralizedLinearAlgorithm` 中，利用`optimizer`来求解目标函数。

```
  val weightsWithIntercept = optimizer.optimize(data, initialWeightsWithIntercept)
```


```
 object LogisticRegressionWithSGD {
  def train(
       input: RDD[LabeledPoint],
       numIterations: Int,
       stepSize: Double,
       miniBatchFraction: Double,
       initialWeights: Vector): LogisticRegressionModel = {
     new LogisticRegressionWithSGD(stepSize, numIterations, 0.0, miniBatchFraction)
       .run(input, initialWeights)
   }
```



如下是运用随机梯度下降算法 LBFGS 求解的 `LogisticRegressionWithLBFGS ` 算法声明。

```
 class LogisticRegressionWithLBFGS
   extends GeneralizedLinearAlgorithm[LogisticRegressionModel] with Serializable {
   
   this.setFeatureScaling(true)
   // 利用LBFGS算法求解优化目标函数
   override val optimizer = new LBFGS(new LogisticGradient, new SquaredL2Updater)
   }
```

### NB算法



## Ref
http://spark.apache.org/docs/latest/mllib-guide.html#fn:1
http://yanbohappy.sinaapp.com/?p=498

gbdt: http://blog.csdn.net/w28971023/article/details/8240756
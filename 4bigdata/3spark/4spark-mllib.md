













### 随机数

```scala
import org.apache.spark.{SparkContext, SparkConf}
import org.apache.spark.mllib.random.RandomRDDs._
object RandomRDDTest {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("RandomRDDTest")
        val sc = new SparkContext(conf)
        
        // 100万 0-1 随机数,均匀分布在10个分区中。
        val u = normalRDD(sc, 1000000L, 10)
        val v = u.map(x => 1.0 + 2.0 * x)
        v.foreach(println)  
    }
}
```



### 假设检验？

用于确定结果是否具有统计学意义

```scala
//卡方检验
import org.apache.spark.{SparkContext, SparkConf}
import org.apache.spark.mllib.linalg._
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.stat.Statistics
import org.apache.spark.mllib.stat.test.ChiSqTestResult
import org.apache.spark.rdd.RDD


/**
  * spark.mllib目前支持Pearson的卡方（χ2χ2）测试，以获得适合度和独立性。 输入数据类型确定是否进行拟合优度或独立性测试。
  * 适合度测试需要输入类型的Vector，而独立性测试需要一个Matrix作为输入。
  * spark.mllib还支持输入类型RDD [LabeledPoint]，通过卡方独立测试来启用特征选择。
  */
object HypothesisTestingTestAPI {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testLabeledPoint2") 
        val sc = new SparkContext(conf) 
        
        val vec: Vector = Vectors.dense(0.1, 0.15, 0.2, 0.3, 0.25)
        val goodnessOfFitTestResult = Statistics.chiSqTest(vec)

        val mat: Matrix = Matrices.dense(3, 2, Array(1.0, 3.0, 5.0, 2.0, 4.0, 6.0))
        val independenceTestResult = Statistics.chiSqTest(mat)
        
        val obs: RDD[LabeledPoint] =sc.parallelize(Seq(
            LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0)),
            LabeledPoint(1.0, Vectors.dense(1.0, 2.0, 0.0)),
            LabeledPoint(-1.0, Vectors.dense(-1.0, 0.0, -0.5)))) 
       
        val featureTestResults: Array[ChiSqTestResult] = Statistics.chiSqTest(obs)
        featureTestResults.zipWithIndex.foreach { 
            case (k, v) =>println("Column " + (v + 1).toString + ":")
            println(k)
        }
    }
}
```

```scala
// Kolmogorov-Smirnov（KS 测试）
import org.apache.spark.mllib.stat.Statistics
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
object HypothesisTestingTestAPI2 {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testLabeledPoint2") 
        val sc = new SparkContext(conf) 
        val data: RDD[Double] = sc.parallelize(Seq(0.1, 0.15, 0.2, 0.3, 0.25))  

        val testResult = Statistics.kolmogorovSmirnovTest(data, "norm", 0, 1)
        println(testResult)
        
        val myCDF = Map(0.1 -> 0.2, 0.15 -> 0.6, 0.2 -> 0.05, 0.3 -> 0.05, 0.25 -> 0.1)
        val testResult2 = Statistics.kolmogorovSmirnovTest(data, myCDF)
    }
}
```



### 分层采样

```scala
import org.apache.spark.{SparkContext, SparkConf}
object StratifiedSamplingTestAPI {
    def main(args: Array[String]) {
        val conf = new SparkConf().setMaster("local").setAppName("testLabeledPoint2") 
        val sc = new SparkContext(conf)
        
        val data = sc.parallelize(Seq((1, 'a'), (1, 'b'), (2, 'c'), (2, 'd'), (2, 'e'), (3, 'f')))
        val fractions = Map(1 -> 0.1, 2 -> 0.6, 3 -> 0.3)

        val approxSample = data.sampleByKey(withReplacement = false, fractions = fractions)
        approxSample.foreach(println)
        //    (1,b)
        //    (2,d)

        val exactSample = data.sampleByKeyExact(withReplacement = false, fractions = fractions)
        exactSample.foreach(println)
        //    (2,e)
        //    (3,f)
  
    }
}
```



# 特征抽取和转化

### TF-IDF

```scala
import org.apache.spark.mllib.feature.{HashingTF, IDF}
import org.apache.spark.mllib.linalg.Vector
import org.apache.spark.rdd.RDD

// Load documents (one per line).
val documents: RDD[Seq[String]] = sc.textFile("data/mllib/kmeans_data.txt").map(_.split(" ").toSeq)

val hashingTF = new HashingTF()
val tf: RDD[Vector] = hashingTF.transform(documents)

// While applying HashingTF only needs a single pass to the data, applying IDF needs two passes:
tf.cache()
val idf = new IDF().fit(tf)
val tfidf: RDD[Vector] = idf.transform(tf)

val idfIgnore = new IDF(minDocFreq = 2).fit(tf)
val tfidfIgnore: RDD[Vector] = idfIgnore.transform(tf)
```



### Word2Vec

```scala
import org.apache.spark.mllib.feature.{Word2Vec, Word2VecModel}

val input = sc.textFile("data/mllib/sample_lda_data.txt").map(line => line.split(" ").toSeq)

val word2vec = new Word2Vec()
val model = word2vec.fit(input)

val synonyms = model.findSynonyms("1", 5)
for((synonym, cosineSimilarity) <- synonyms) {
  println(s"$synonym $cosineSimilarity")
}

model.save(sc, "myModelPath")
val sameModel = Word2VecModel.load(sc, "myModelPath")
```



### StandardScaler

```scala
import org.apache.spark.mllib.feature.{StandardScaler, StandardScalerModel}
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.util.MLUtils

val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")

val scaler1 = new StandardScaler().fit(data.map(x => x.features))
//scaler2&scaler3 模型相同
val scaler2 = new StandardScaler(withMean = true, withStd = true).fit(data.map(x => x.features))
val scaler3 = new StandardScalerModel(scaler2.std, scaler2.mean)

val data1 = data.map(x => (x.label, scaler1.transform(x.features)))
val data2 = data.map(x => (x.label, scaler2.transform(Vectors.dense(x.features.toArray))))
```



### Normalizer

```scala
import org.apache.spark.mllib.feature.Normalizer
import org.apache.spark.mllib.util.MLUtils

val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")

val normalizer1 = new Normalizer()
val normalizer2 = new Normalizer(p = Double.PositiveInfinity)

val data1 = data.map(x => (x.label, normalizer1.transform(x.features)))
```



### ChiSqSelector

```scala
import org.apache.spark.mllib.feature.ChiSqSelector
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.util.MLUtils

val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")

// Discretize data in 16 equal bins since ChiSqSelector requires categorical features
// Even though features are doubles, the ChiSqSelector treats each unique value as a category
val discretizedData = data.map { lp =>
  LabeledPoint(lp.label, Vectors.dense(lp.features.toArray.map { x => (x / 16).floor }))
}

val selector = new ChiSqSelector(50)
val transformer = selector.fit(discretizedData)

val filteredData = discretizedData.map { lp =>
  LabeledPoint(lp.label, transformer.transform(lp.features))
}
```



# 分类&回归

| Problem Type              | Supported Methods                                            |
| ------------------------- | ------------------------------------------------------------ |
| Binary Classification     | linear SVMs, logistic regression, decision trees, random forests, gradient-boosted trees, naive Bayes |
| Multiclass Classification | logistic regression, decision trees, random forests, naive Bayes |
| Regression                | linear least squares, Lasso, ridge regression, decision trees, random forests, gradient-boosted trees, isotonic regression |



### 支持向量机分类

```scala
import org.apache.spark.mllib.classification.{SVMModel, SVMWithSGD}
import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
import org.apache.spark.mllib.util.MLUtils

val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")

val splits = data.randomSplit(Array(0.6, 0.4), seed = 11L)
val training = splits(0).cache()
val test = splits(1)

val numIterations = 100
val model = SVMWithSGD.train(training, numIterations)


model.clearThreshold()

val scoreAndLabels = test.map { 
    point => val score = model.predict(point.features)
    (score, point.label)
}

val metrics = new BinaryClassificationMetrics(scoreAndLabels)
val auROC = metrics.areaUnderROC()

model.save(sc, "target/tmp/scalaSVMWithSGDModel")
val sameModel = SVMModel.load(sc, "target/tmp/scalaSVMWithSGDModel")
```



### 逻辑回归 分类

```scala
import org.apache.spark.mllib.classification.{LogisticRegressionModel, LogisticRegressionWithLBFGS}
import org.apache.spark.mllib.evaluation.MulticlassMetrics
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.util.MLUtils

val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")
val splits = data.randomSplit(Array(0.6, 0.4), seed = 11L)
val training = splits(0).cache()
val test = splits(1)

val model = new LogisticRegressionWithLBFGS().setNumClasses(10).run(training)

val predictionAndLabels = test.map { 
    case LabeledPoint(label, features) => val prediction = model.predict(features)
    (prediction, label)
}

val metrics = new MulticlassMetrics(predictionAndLabels)
val accuracy = metrics.accuracy

model.save(sc, "target/tmp/scalaLogisticRegressionWithLBFGSModel")
val sameModel = LogisticRegressionModel.load(sc,"target/tmp/scalaLogisticRegressionWithLBFGSModel")
```



### 流式线性回归 回归

```scala
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.regression.StreamingLinearRegressionWithSGD

val trainingData = ssc.textFileStream(args(0)).map(LabeledPoint.parse).cache()
val testData = ssc.textFileStream(args(1)).map(LabeledPoint.parse)

val numFeatures = 3
val model = new StreamingLinearRegressionWithSGD().setInitialWeights(Vectors.zeros(numFeatures))

model.trainOn(trainingData)
model.predictOnValues(testData.map(lp => (lp.label, lp.features))).print()

ssc.start()
ssc.awaitTermination()
```



# 聚类

### K-means

```scala
import org.apache.spark.mllib.clustering.{KMeans, KMeansModel}
import org.apache.spark.mllib.linalg.Vectors

val data = sc.textFile("data/mllib/kmeans_data.txt")
val parsedData = data.map(s => Vectors.dense(s.split(' ').map(_.toDouble))).cache()

val numClusters = 2
val numIterations = 20
val clusters = KMeans.train(parsedData, numClusters, numIterations)

// Evaluate clustering by computing Within Set Sum of Squared Errors
val WSSSE = clusters.computeCost(parsedData)

clusters.save(sc, "target/org/apache/spark/KMeansExample/KMeansModel")
val sameModel = KMeansModel.load(sc, "target/org/apache/spark/KMeansExample/KMeansModel")
```



### Gaussian mixture

```scala
import org.apache.spark.mllib.clustering.{GaussianMixture, GaussianMixtureModel}
import org.apache.spark.mllib.linalg.Vectors

val data = sc.textFile("data/mllib/gmm_data.txt")
val parsedData = data.map(s => Vectors.dense(s.trim.split(' ').map(_.toDouble))).cache()

// k是聚类群数
val gmm = new GaussianMixture().setK(2).run(parsedData)

// Save and load model
gmm.save(sc, "target/org/apache/spark/GaussianMixtureExample/GaussianMixtureModel")
val sameModel = GaussianMixtureModel.load(sc,"target/org/apache/spark/GaussianMixtureExample/GaussianMixtureModel")

//输出聚类结果
for (i <- 0 until gmm.k) {
  println("weight=%f\nmu=%s\nsigma=\n%s\n" format(gmm.weights(i), gmm.gaussians(i).mu, gmm.gaussians(i).sigma))
}
```

Latent Dirichlet allocation (LDA)

```scala
import org.apache.spark.mllib.clustering.{DistributedLDAModel, LDA}
import org.apache.spark.mllib.linalg.Vectors

val data = sc.textFile("data/mllib/sample_lda_data.txt")
val parsedData = data.map(s => Vectors.dense(s.trim.split(' ').map(_.toDouble)))
val corpus = parsedData.zipWithIndex.map(_.swap).cache()

val ldaModel = new LDA().setK(3).run(corpus)

// Output topics. Each is a distribution over words (matching word count vectors)
println(s"Learned topics (as distributions over vocab of ${ldaModel.vocabSize} words):")
val topics = ldaModel.topicsMatrix
for (topic <- Range(0, 3)) {
    print(s"Topic $topic :")
    for (word <- Range(0, ldaModel.vocabSize)) {
        print(s"${topics(word, topic)}")
    }
    println()
}

ldaModel.save(sc, "target/org/apache/spark/LatentDirichletAllocationExample/LDAModel")
val sameModel = DistributedLDAModel.load(sc,"target/org/apache/spark/LatentDirichletAllocationExample/LDAModel")
```



### Streaming k-means

```scala
import org.apache.spark.mllib.clustering.StreamingKMeans
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setAppName("StreamingKMeansExample")
val ssc = new StreamingContext(conf, Seconds(args(2).toLong))

val trainingData = ssc.textFileStream(args(0)).map(Vectors.parse)
val testData = ssc.textFileStream(args(1)).map(LabeledPoint.parse)

val model = new StreamingKMeans()
  .setK(args(3).toInt)
  .setDecayFactor(1.0)
  .setRandomCenters(args(4).toInt, 0.0)

model.trainOn(trainingData)
model.predictOnValues(testData.map(lp => (lp.label, lp.features))).print()

ssc.start()
ssc.awaitTermination()
```



# 降维

### SVD

Singular value decomposition 

```scala
import org.apache.spark.mllib.linalg.Matrix
import org.apache.spark.mllib.linalg.SingularValueDecomposition
import org.apache.spark.mllib.linalg.Vector
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.linalg.distributed.RowMatrix

val data = Array(
  Vectors.sparse(5, Seq((1, 1.0), (3, 7.0))),
  Vectors.dense(2.0, 0.0, 3.0, 4.0, 5.0),
  Vectors.dense(4.0, 0.0, 0.0, 6.0, 7.0))
val rows = sc.parallelize(data)
val mat: RowMatrix = new RowMatrix(rows)

// Compute the top 5 singular values and corresponding singular vectors.
val svd: SingularValueDecomposition[RowMatrix, Matrix] = mat.computeSVD(5, computeU = true)
val U: RowMatrix = svd.U  // The U factor is a RowMatrix.
val s: Vector = svd.s     // The singular values are stored in a local dense vector.
val V: Matrix = svd.V     // The V factor is a local dense matrix.
```



###  PCA

Principal component analysis

```scala
import org.apache.spark.mllib.linalg.Matrix
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.linalg.distributed.RowMatrix

val data = Array(
  Vectors.sparse(5, Seq((1, 1.0), (3, 7.0))),
  Vectors.dense(2.0, 0.0, 3.0, 4.0, 5.0),
  Vectors.dense(4.0, 0.0, 0.0, 6.0, 7.0))
val rows = sc.parallelize(data)
val mat: RowMatrix = new RowMatrix(rows)

// Principal components are stored in a local dense matrix.
val pc: Matrix = mat.computePrincipalComponents(4)

// Project the rows to the linear space spanned by the top 4 principal components.
val projected: RowMatrix = mat.multiply(pc)
```

```scala
import org.apache.spark.mllib.feature.PCA
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.rdd.RDD

val data: RDD[LabeledPoint] = sc.parallelize(Seq(
  new LabeledPoint(0, Vectors.dense(1, 0, 0, 0, 1)),
  new LabeledPoint(1, Vectors.dense(1, 1, 0, 1, 0)),
  new LabeledPoint(1, Vectors.dense(1, 1, 0, 0, 0)),
  new LabeledPoint(0, Vectors.dense(1, 0, 0, 0, 0)),
  new LabeledPoint(1, Vectors.dense(1, 1, 0, 0, 0))))

val pca = new PCA(5).fit(data.map(_.features))

val projected = data.map(p => p.copy(features = pca.transform(p.features)))
```



# ALS

```scala
import org.apache.spark.mllib.recommendation.ALS
import org.apache.spark.mllib.recommendation.MatrixFactorizationModel
import org.apache.spark.mllib.recommendation.Rating

val data = sc.textFile("data/mllib/als/test.data")
val ratings = data.map(_.split(',') match { case Array(user, item, rate) =>
  Rating(user.toInt, item.toInt, rate.toDouble)
})

val rank = 10
val numIterations = 10
val model = ALS.train(ratings, rank, numIterations, 0.01)

val usersProducts = ratings.map { case Rating(user, product, rate) =>
  (user, product)
}

val predictions = model.predict(usersProducts).map { case Rating(user, product, rate) =>
    ((user, product), rate)
}

val ratesAndPreds = ratings.map { case Rating(user, product, rate) =>
  ((user, product), rate)
}.join(predictions)

val MSE = ratesAndPreds.map { 
    case ((user, product), (r1, r2)) => val err = (r1 - r2)
    err * err
}.mean()

model.save(sc, "target/tmp/myCollaborativeFilter")
val sameModel = MatrixFactorizationModel.load(sc, "target/tmp/myCollaborativeFilter")
```



# FP-growth

```scala
import org.apache.spark.mllib.fpm.FPGrowth
import org.apache.spark.rdd.RDD

val data = sc.textFile("data/mllib/sample_fpgrowth.txt")
val transactions: RDD[Array[String]] = data.map(s => s.trim.split(' '))

val fpg = new FPGrowth().setMinSupport(0.2).setNumPartitions(10)
val model = fpg.run(transactions)

model.freqItemsets.collect().foreach { itemset =>
  println(s"${itemset.items.mkString("[", ",", "]")},${itemset.freq}")
}

val minConfidence = 0.8
model.generateAssociationRules(minConfidence).collect().foreach { rule =>
  println(s"${rule.antecedent.mkString("[", ",", "]")}=> " + s"${rule.consequent .mkString("[", ",", "]")},${rule.confidence}")
}
```



# 评价指标

### Binary classification

```scala
import org.apache.spark.mllib.classification.LogisticRegressionWithLBFGS
import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.util.MLUtils

val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_binary_classification_data.txt")

val Array(training, test) = data.randomSplit(Array(0.6, 0.4), seed = 11L)
training.cache()

val model = new LogisticRegressionWithLBFGS().setNumClasses(2).run(training)

// Clear the prediction threshold so the model will return probabilities
model.clearThreshold


val predictionAndLabels = test.map { case LabeledPoint(label, features) =>
  val prediction = model.predict(features)
  (prediction, label)
}


val metrics = new BinaryClassificationMetrics(predictionAndLabels)

val precision = metrics.precisionByThreshold
precision.collect.foreach { case (t, p) =>
  println(s"Threshold: $t, Precision: $p")
}

val recall = metrics.recallByThreshold
recall.collect.foreach { case (t, r) =>
  println(s"Threshold: $t, Recall: $r")
}

val PRC = metrics.pr

val f1Score = metrics.fMeasureByThreshold
f1Score.collect.foreach { case (t, f) =>
  println(s"Threshold: $t, F-score: $f, Beta = 1")
}

val beta = 0.5
val fScore = metrics.fMeasureByThreshold(beta)
fScore.collect.foreach { case (t, f) =>
  println(s"Threshold: $t, F-score: $f, Beta = 0.5")
}

val auPRC = metrics.areaUnderPR
println(s"Area under precision-recall curve = $auPRC")

val thresholds = precision.map(_._1)

val roc = metrics.roc

val auROC = metrics.areaUnderROC
println(s"Area under ROC = $auROC")
```



### Multiclass classification

```scala
import org.apache.spark.mllib.classification.LogisticRegressionWithLBFGS
import org.apache.spark.mllib.evaluation.MulticlassMetrics
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.util.MLUtils

// Load training data in LIBSVM format
val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_multiclass_classification_data.txt")

// Split data into training (60%) and test (40%)
val Array(training, test) = data.randomSplit(Array(0.6, 0.4), seed = 11L)
training.cache()

// Run training algorithm to build the model
val model = new LogisticRegressionWithLBFGS()
  .setNumClasses(3)
  .run(training)

// Compute raw scores on the test set
val predictionAndLabels = test.map { case LabeledPoint(label, features) =>
  val prediction = model.predict(features)
  (prediction, label)
}

// Instantiate metrics object
val metrics = new MulticlassMetrics(predictionAndLabels)

// Confusion matrix
println("Confusion matrix:")
println(metrics.confusionMatrix)

// Overall Statistics
val accuracy = metrics.accuracy
println("Summary Statistics")
println(s"Accuracy = $accuracy")

// Precision by label
val labels = metrics.labels
labels.foreach { l =>
  println(s"Precision($l) = " + metrics.precision(l))
}

// Recall by label
labels.foreach { l =>
  println(s"Recall($l) = " + metrics.recall(l))
}

// False positive rate by label
labels.foreach { l =>
  println(s"FPR($l) = " + metrics.falsePositiveRate(l))
}

// F-measure by label
labels.foreach { l =>
  println(s"F1-Score($l) = " + metrics.fMeasure(l))
}

// Weighted stats
println(s"Weighted precision: ${metrics.weightedPrecision}")
println(s"Weighted recall: ${metrics.weightedRecall}")
println(s"Weighted F1 score: ${metrics.weightedFMeasure}")
println(s"Weighted false positive rate: ${metrics.weightedFalsePositiveRate}")
```



### Multilabel classification

```scala
import org.apache.spark.mllib.evaluation.MultilabelMetrics
import org.apache.spark.rdd.RDD

val scoreAndLabels: RDD[(Array[Double], Array[Double])] = sc.parallelize(
  Seq((Array(0.0, 1.0), Array(0.0, 2.0)),
    (Array(0.0, 2.0), Array(0.0, 1.0)),
    (Array.empty[Double], Array(0.0)),
    (Array(2.0), Array(2.0)),
    (Array(2.0, 0.0), Array(2.0, 0.0)),
    (Array(0.0, 1.0, 2.0), Array(0.0, 1.0)),
    (Array(1.0), Array(1.0, 2.0))), 2)

// Instantiate metrics object
val metrics = new MultilabelMetrics(scoreAndLabels)

// Summary stats
println(s"Recall = ${metrics.recall}")
println(s"Precision = ${metrics.precision}")
println(s"F1 measure = ${metrics.f1Measure}")
println(s"Accuracy = ${metrics.accuracy}")

// Individual label stats
metrics.labels.foreach(label =>
  println(s"Class $label precision = ${metrics.precision(label)}"))
metrics.labels.foreach(label => println(s"Class $label recall = ${metrics.recall(label)}"))
metrics.labels.foreach(label => println(s"Class $label F1-score = ${metrics.f1Measure(label)}"))

// Micro stats
println(s"Micro recall = ${metrics.microRecall}")
println(s"Micro precision = ${metrics.microPrecision}")
println(s"Micro F1 measure = ${metrics.microF1Measure}")

// Hamming loss
println(s"Hamming loss = ${metrics.hammingLoss}")

// Subset accuracy
println(s"Subset accuracy = ${metrics.subsetAccuracy}")
```



### Ranking systems

```scala
import org.apache.spark.mllib.evaluation.{RankingMetrics, RegressionMetrics}
import org.apache.spark.mllib.recommendation.{ALS, Rating}

// Read in the ratings data
val ratings = spark.read.textFile("data/mllib/sample_movielens_data.txt").rdd.map { line =>
  val fields = line.split("::")
  Rating(fields(0).toInt, fields(1).toInt, fields(2).toDouble - 2.5)
}.cache()

// Map ratings to 1 or 0, 1 indicating a movie that should be recommended
val binarizedRatings = ratings.map(r => Rating(r.user, r.product,
  if (r.rating > 0) 1.0 else 0.0)).cache()

// Summarize ratings
val numRatings = ratings.count()
val numUsers = ratings.map(_.user).distinct().count()
val numMovies = ratings.map(_.product).distinct().count()
println(s"Got $numRatings ratings from $numUsers users on $numMovies movies.")

// Build the model
val numIterations = 10
val rank = 10
val lambda = 0.01
val model = ALS.train(ratings, rank, numIterations, lambda)

// Define a function to scale ratings from 0 to 1
def scaledRating(r: Rating): Rating = {
  val scaledRating = math.max(math.min(r.rating, 1.0), 0.0)
  Rating(r.user, r.product, scaledRating)
}

// Get sorted top ten predictions for each user and then scale from [0, 1]
val userRecommended = model.recommendProductsForUsers(10).map { case (user, recs) =>
  (user, recs.map(scaledRating))
}

// Assume that any movie a user rated 3 or higher (which maps to a 1) is a relevant document
// Compare with top ten most relevant documents
val userMovies = binarizedRatings.groupBy(_.user)
val relevantDocuments = userMovies.join(userRecommended).map { case (user, (actual,
predictions)) =>
  (predictions.map(_.product), actual.filter(_.rating > 0.0).map(_.product).toArray)
}

// Instantiate metrics object
val metrics = new RankingMetrics(relevantDocuments)

// Precision at K
Array(1, 3, 5).foreach { k =>
  println(s"Precision at $k = ${metrics.precisionAt(k)}")
}

// Mean average precision
println(s"Mean average precision = ${metrics.meanAveragePrecision}")

// Mean average precision at k
println(s"Mean average precision at 2 = ${metrics.meanAveragePrecisionAt(2)}")

// Normalized discounted cumulative gain
Array(1, 3, 5).foreach { k =>
  println(s"NDCG at $k = ${metrics.ndcgAt(k)}")
}

// Recall at K
Array(1, 3, 5).foreach { k =>
  println(s"Recall at $k = ${metrics.recallAt(k)}")
}

// Get predictions for each data point
val allPredictions = model.predict(ratings.map(r => (r.user, r.product))).map(r => ((r.user,
  r.product), r.rating))
val allRatings = ratings.map(r => ((r.user, r.product), r.rating))
val predictionsAndLabels = allPredictions.join(allRatings).map { case ((user, product),
(predicted, actual)) =>
  (predicted, actual)
}

// Get the RMSE using regression metrics
val regressionMetrics = new RegressionMetrics(predictionsAndLabels)
println(s"RMSE = ${regressionMetrics.rootMeanSquaredError}")

// R-squared
println(s"R-squared = ${regressionMetrics.r2}")
```



# PMML

```scala
import org.apache.spark.mllib.clustering.KMeans
import org.apache.spark.mllib.linalg.Vectors

val data = sc.textFile("data/mllib/kmeans_data.txt")
val parsedData = data.map(s => Vectors.dense(s.split(' ').map(_.toDouble))).cache()

// Cluster the data into two classes using KMeans
val numClusters = 2
val numIterations = 20
val clusters = KMeans.train(parsedData, 2, 20)

clusters.toPMML("/tmp/kmeans.xml")
clusters.toPMML(sc, "/tmp/kmeans")
clusters.toPMML(System.out)
```



# 问题排查

高维数据pmml不支持问题：https://blog.csdn.net/u013227785/article/details/117993748?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1.topblog&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1.topblog&utm_relevant_index=1



自定义pmml不支持的transformer：https://blog.csdn.net/NOT_GUY/article/details/100054852














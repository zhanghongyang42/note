官网：https://scikit-learn.org/stable/

​			http://www.studyai.cn/index.html

书籍：《机器学习实战》《Hands-On Machine Learning with Scikit-Learn and TensorFlow》

​			《统计学习方法》《深度学习》



# 概述

sklearn 主要实现    特征工程、监督学习、聚类、降维、模型选择、模型评价、pipeline持久化 等。

![v2-a27ca7566aa6d028114421f4acff610c_r](picture/v2-a27ca7566aa6d028114421f4acff610c_r.jpg)



### 核心对象

Sklearn 里有 estimator（估计器） 、 transformer（转换器）  、元估计器。

```
继承estimator 和 transformer 是特征工程
继承 estimator 是算法
元估计器把估计器当成参数
```



五大元估计器：

集成功能的 **ensemble**

- ensemble.BaggingClassifier
- ensemble.VotingClassifier

多分类和多标签的 **multiclass**

- *multiclass.OneVsOneClassifier*
- *multiclass.OneVsRestClassifier*

多输出的 **multioutput**

- multioutput.MultiOutputClassifier

选择模型的 **model_selection**

- model_selection.GridSearchCV

- model_selection.RandomizedSearchCV

流水线的 **pipeline**

- pipeline.Pipeline



### 接口分类

**sklearn.datasets** - 用于得到数据集



**sklearn.feature_extraction** - 特征提取

**sklearn.impute** - 缺失值填充

**sklearn.preprocessing** - 特征处理

**sklearn.feature_selection** - 特征选择



**sklearn.linear_model** - 线性模型

**sklearn.tree** - 树模型

**sklearn.ensemble** - 集成模型

**sklearn.multioutput** - 多输出模型



 **sklearn.cluster** - 聚类模型

**sklearn.decomposition** - 降维模型



**sklearn.model_selection** - 模型选择

**sklearn.metrics** - 模型评价

**sklearn.pipeline** - 管道



# 特征工程

## 缺失值处理

删除或者填充



##### SimpleImputer

```python
import numpy as np
from sklearn.impute import SimpleImputer

imp = SimpleImputer(missing_values=np.nan, strategy='mean')
imp.fit([[1, 2], [np.nan, 3], [7, 6]])

X = [[np.nan, 2], [6, np.nan], [7, 6]]
X = imp.transform(X)
```



##### iterative_imputer

其他列作为x，缺失列作为y，多次预测，进行填充

```python
import numpy as np
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

imp = IterativeImputer(max_iter=10, random_state=0)
imp.fit([[1, 2], [3, 6], [4, 8], [np.nan, 3], [7, np.nan]])

X_test = [[np.nan, 2], [6, np.nan], [np.nan, 6]]
print(np.round(imp.transform(X_test)))
```



##### KNNImputer

```python
from sklearn.impute import KNNImputer

imputer = KNNImputer(n_neighbors=2, weights="uniform")
imputer.fit_transform(X)
```



##### MissingIndicator

```python
from sklearn.impute import MissingIndicator

indicator = MissingIndicator(missing_values=-1)  #值为-1代表缺失
mask_missing_values_only = indicator.fit_transform(X)
```



## 数据处理

### 标准化

##### Normalizer

```python
from sklearn.preprocessing import Normalizer

normalizer = Normalizer().fit(X)  # fit does nothing
X = normalizer.transform(X)
```



##### StandardScaler

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler().fit(X_train)
print(scaler.transform(X_train))
```



##### MinMaxScaler

```python
from sklearn.preprocessing import MinMaxScaler

X_train_minmax = MinMaxScaler().fit_transform(X_train)
```



##### MaxAbsScaler

设置参数  with_mean=False  ，可以缩放稀疏数据。

```python
from sklearn.preprocessing import MaxAbsScaler

X_train_maxabs =  MaxAbsScaler().fit_transform(X_train)
```



### 编码方式

labelencoder/OrdinalEncoder

onehotencoder



### 离散化

##### KBinsDiscretizer

```python
from sklearn.preprocessing import KBinsDiscretizer
est = KBinsDiscretizer(n_bins=[3, 2, 2], encode='ordinal').fit(X)
X = est.transform(X)
```



##### Binarizer  二值化

```python
from sklearn.preprocessing import Binarizer

binarizer = Binarizer().fit(X)
X = binarizer.transform(X)

binarizer = Binarizer(threshold=1.1)
X = binarizer.transform(X)
```



### 数据变换

##### QuantileTransformer

把数据变成01之间的均匀分布

```python
from sklearn.preprocessing import QuantileTransformer

quantile_transformer = QuantileTransformer(random_state=0)
X_train_trans = quantile_transformer.fit_transform(X)
```



##### PowerTransformer

从对数正态分布映射到正态分布上

```python
X_lognormal = np.random.RandomState(616).lognormal(size=(3, 3))

pt = preprocessing.PowerTransformer(method='box-cox', standardize=False)
X_lognormal = pt.fit_transform(X_lognormal)
```



## 特征选择

### 方差选择法

```python
#方差计算(s2) = Σ [(xi - x̅)2]/n - 1
from sklearn.feature_selection import VarianceThreshold

X = [[0, 0, 1], [0, 1, 0], [1, 0, 0], [0, 1, 1], [0, 1, 0], [0, 1, 1]]
sel = VarianceThreshold(threshold=(.8 * (1 - .8)))
X_new = sel.fit_transform(X)
```



### 单变量特征选择

```python
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2

selector = SelectKBest(chi2, k=2).fit(X, y)

#相似性得分越大越好，pvalue越小越好
print(selector.scores_)
print(selector.pvalues_)
print(selector.get_support(True))

X_new = selector.transform(X)
```

注：稀疏格式数据只能使用卡方检验和互信息法



### 递归特征消除（RFE）

```python
from sklearn.svm import SVC
from sklearn.feature_selection import RFE

svc = SVC(kernel="linear", C=1)
rfe = RFE(estimator=svc, n_features_to_select=1, step=1) #n_features_to_select：剩几个，step：每次删几个
df = rfe.fit_transform(X, y)
```



### SelectFromModel

##### 基于 L1 的特征选取

```python
from sklearn.feature_selection import SelectFromModel
from sklearn.svm import LinearSVC

lsvc = LinearSVC(C=0.01, penalty="l1", dual=False).fit(X, y)

model = SelectFromModel(lsvc, prefit=True)
X_new = model.transform(X)
```



##### 基于 树的特征选取

基于树的 estimators （查阅 [`sklearn.tree`](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.tree) 模块和树的森林 在 [`sklearn.ensemble`](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.ensemble) 模块） 可以用来计算特征的重要性。

```python
from sklearn.feature_selection import SelectFromModel
from sklearn.ensemble import ExtraTreesClassifier

clf = ExtraTreesClassifier().fit(X, y)

model = SelectFromModel(clf, prefit=True)
X_new = model.transform(X)
```



# 监督学习

监督学习就是有标签的算法。

lable为离散值是分类算法，label为连续值是回归算法。

很多监督学习算法即可用于回归，也可以用于分类。



### LinearRegression

回归算法

```python
from sklearn import linear_model

reg = linear_model.LinearRegression()
reg.fit ([[0, 0], [1, 1], [2, 2]], [0, 1, 2])

print(reg.coef_)
```



### LogisticRegression

是**二分类**算法。已经封装好用于**多分类**的改进。也可以用分类概率值进行**排序**。

```python
from sklearn.linear_model import LogisticRegression

clf = LogisticRegression(random_state=0).fit(X, y)

clf.predict(X[:2, :])
clf.predict_proba(X[:2, :])
clf.score(X, y)
```



### DecisionTree

即可用于回归，也可以用于分类。

```python
#回归
from sklearn import tree

clf = tree.DecisionTreeRegressor()
clf = clf.fit(X, y)
clf.predict([[1, 1]])
```

```python
#二分类，多分类，排序
from sklearn import tree

clf = tree.DecisionTreeClassifier()
clf = clf.fit(X, Y)

clf.predict([[2., 2.]])
clf.predict_proba([[2., 2.]])
```

```python
#决策树可视化
import graphviz

dot_data = tree.export_graphviz(clf, out_file=None)
graph = graphviz.Source(dot_data)
graph.render("iris")
```



### 其他算法

```python
### SVM回归 ###
from sklearn import svm
model_SVR = svm.SVR()

### KNN回归 ###
from sklearn import neighbors
model_KNeighborsRegressor = neighbors.KNeighborsRegressor()
```



# 集成模型

bagging：各个模型并行跑出结果后投票或平均概率。

`如：随机森林`

boosting：各个模型串行跑出结果进行效果提升。

`如：GBDT、AdaBoost`				

stacking：bagging和boosting混合使用。



bagging 方法可以减小过拟合，所以通常在强分类器和复杂模型上使用时表现的很好（例如，完全生长的决策树）

boosting 方法则在弱模型上表现更好（例如，浅层决策树）。



### RandomForest

分类，回归

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import RandomForestRegressor

clf = RandomForestClassifier(n_estimators=10)
clf = RandomForestRegressor(n_estimators=20)

clf = clf.fit(X, Y)

clf.feature_importances_
```



### AdaBoost

```python
#二分类
from sklearn.ensemble import AdaBoostClassifier
clf = AdaBoostClassifier(n_estimators=100)

#回归
from sklearn import ensemble
model_AdaBoostRegressor = ensemble.AdaBoostRegressor(n_estimators=50)
```



### GBDT

```python
#分类
from sklearn.ensemble import GradientBoostingClassifier

clf = GradientBoostingClassifier(n_estimators=100,learning_rate=1.0,max_depth=1,random_state=0)
clf = clf.fit(X_train, y_train)
clf.score(X_test, y_test)                 
```

```python
#回归
from sklearn.ensemble import GradientBoostingRegressor
est = GradientBoostingRegressor(n_estimators=100, learning_rate=0.1,max_depth=1,random_state=0, loss='ls').fit(X_train, y_train)
```

```python
#支持热启动，在已经训练好的模型基础上继续加入数据
est.set_params(n_estimators=200, warm_start=True)  
est.fit(X_train, y_train) 
```



### Bagging

相同模型进行 bagging

```python
from sklearn.ensemble import BaggingClassifier
from sklearn.neighbors import KNeighborsClassifier
bagging = BaggingClassifier(KNeighborsClassifier(),max_samples=0.5, max_features=0.5)

from sklearn.ensemble import BaggingRegressor
model_BaggingRegressor = BaggingRegressor()
```



### Voting

不同模型进行 bagging

```python
#分类
from sklearn.linear_model import LogisticRegression
from sklearn.naive_bayes import GaussianNB
from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import VotingClassifier
from sklearn.model_selection import cross_val_score

clf1 = LogisticRegression(solver='lbfgs', multi_class='multinomial',random_state=1)
clf2 = RandomForestClassifier(n_estimators=50, random_state=1)
clf3 = GaussianNB()

eclf=VotingClassifier(estimators=[('lr', clf1),('rf', clf2),('gnb', clf3)],voting='soft',weights=[2, 1, 2])
```

```python
#回归
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.ensemble import RandomForestRegressor
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import VotingRegressor

reg1 = GradientBoostingRegressor(random_state=1, n_estimators=10)
reg2 = RandomForestRegressor(random_state=1, n_estimators=10)
reg3 = LinearRegression()

ereg = VotingRegressor(estimators=[('gb', reg1), ('rf', reg2), ('lr', reg3)])
ereg = ereg.fit(X, y)
```



# 多标签/多输出

### 概述

Binary classification（二分类）

Multiclass classification（多分类）

Multioutput regression （多输出回归）

Multilabel classification（多标签分类/多输出二分类）

Multioutput-multiclass classification（多输出多分类）/ multi-task classification（多任务分类）



*tips：*

**多分类** 和 **多标签分类** 和 **多输出回归** 原理：通过 multiclass 把这些问题转化为二分类问题



### 二分类、多分类

二分类、多分类： sklearn 的所有分类器对这两个类 开箱即用

```
固有多分类:
        sklearn.naive_bayes.BernoulliNB
        sklearn.tree.DecisionTreeClassifier
        sklearn.tree.ExtraTreeClassifier
        sklearn.ensemble.ExtraTreesClassifier
        sklearn.naive_bayes.GaussianNB
        sklearn.neighbors.KNeighborsClassifier
        sklearn.semi_supervised.LabelPropagation
        sklearn.semi_supervised.LabelSpreading
        sklearn.discriminant_analysis.LinearDiscriminantAnalysis
        sklearn.svm.LinearSVC (setting multi_class=”crammer_singer”)
        sklearn.linear_model.LogisticRegression (setting multi_class=”multinomial”)
        sklearn.linear_model.LogisticRegressionCV (setting multi_class=”multinomial”)
        sklearn.neural_network.MLPClassifier
        sklearn.neighbors.NearestCentroid
        sklearn.discriminant_analysis.QuadraticDiscriminantAnalysis
        sklearn.neighbors.RadiusNeighborsClassifier
        sklearn.ensemble.RandomForestClassifier
        sklearn.linear_model.RidgeClassifier
        sklearn.linear_model.RidgeClassifierCV
1对1-多分类:
        sklearn.svm.NuSVC
        sklearn.svm.SVC.
        sklearn.gaussian_process.GaussianProcessClassifier (setting multi_class = “one_vs_one”)
1对多-多分类:
        sklearn.ensemble.GradientBoostingClassifier
        sklearn.gaussian_process.GaussianProcessClassifier (setting multi_class = “one_vs_rest”)
        sklearn.svm.LinearSVC (setting multi_class=”ovr”)
        sklearn.linear_model.LogisticRegression (setting multi_class=”ovr”)
        sklearn.linear_model.LogisticRegressionCV (setting multi_class=”ovr”)
        sklearn.linear_model.SGDClassifier
        sklearn.linear_model.Perceptron
        sklearn.linear_model.PassiveAggressiveClassifier
```



### 多标签分类

y有多个标签，如新闻主题可以为 政治、金融、 教育 中的一个或几个。

```python
#将多标签分类转化为多输出二分类的格式
from sklearn.preprocessing import MultiLabelBinarizer
y = [[2, 3, 4], [2], [0, 1, 3], [0, 1, 2, 3, 4], [0, 1, 2]]

MultiLabelBinarizer().fit_transform(y)

>>> array([[0, 0, 1, 1, 1],
       [0, 0, 1, 0, 0],
       [1, 1, 0, 1, 0],
       [1, 1, 1, 1, 1],
       [1, 1, 1, 0, 0]])
```

```python
#多标签分类/多输出二分类
from sklearn.datasets import make_multilabel_classification
from sklearn.model_selection import train_test_split
X, Y = make_multilabel_classification(n_samples=10, n_features=5, n_classes=3, n_labels=2)
X_train, X_test, Y_train ,Y_test = train_test_split(X, Y, test_size=0.2)

from sklearn.tree import DecisionTreeClassifier
cls = DecisionTreeClassifier()
cls.fit(X_train, Y_train)

Y_pred = cls.predict(X_test)

from sklearn import metrics
metrics.f1_score(Y_test, Y_pred, average="macro")
```

```
多标签分类/固有的多输出二分类:
        sklearn.tree.DecisionTreeClassifier
        sklearn.tree.ExtraTreeClassifier
        sklearn.ensemble.ExtraTreesClassifier
        sklearn.neighbors.KNeighborsClassifier
        sklearn.neural_network.MLPClassifier
        sklearn.neighbors.RadiusNeighborsClassifier
        sklearn.ensemble.RandomForestClassifier
        sklearn.linear_model.RidgeClassifierCV
```



### 多输出回归

1.固有多输出回归

```
sklearn.linear_model.LinearRegression
sklearn.neighbors.KNeighborsRegressor
sklearn.tree.DecisionTreeRegressor
sklearn.ensemble.RandomForestRegressor
```

```python
from sklearn.datasets import make_regression
from sklearn.linear_model import LinearRegression

X, y = make_regression(n_samples=1000, n_features=10, n_informative=5, n_targets=2, random_state=1)
data_in = [[-2.02220122, 0.31563495, 0.82797464, -0.30620401, 0.16003707, -1.44411381, 0.87616892, -0.50446586, 0.23009474, 0.76201118]]

model = LinearRegression()
model.fit(X, y)

yhat = model.predict(data_in)
```



2.每一输出单独建立一个回归模型

MultiOutputRegressor

```python
#假设每个输出之间都是相互独立的
from sklearn.datasets import make_regression
from sklearn.multioutput import MultiOutputRegressor
from sklearn.ensemble import GradientBoostingRegressor

X, y = make_regression(n_samples=10, n_targets=3, random_state=1)

MultiOutputRegressor(GradientBoostingRegressor(random_state=0)).fit(X, y).predict(X)

>>> array([[-154.75474165, -147.03498585,  -50.03812219],
       [   7.12165031,    5.12914884,  -81.46081961],
       [-187.8948621 , -100.44373091,   13.88978285],
       [-141.62745778,   95.02891072, -191.48204257],
       [  97.03260883,  165.34867495,  139.52003279],
       [ 123.92529176,   21.25719016,   -7.84253   ],
       [-122.25193977,  -85.16443186, -107.12274212],
       [ -30.170388  ,  -94.80956739,   12.16979946],
       [ 140.72667194,  176.50941682,  -17.50447799],
       [ 149.37967282,  -81.15699552,   -5.72850319]])
```



3.每个输出都参与的链接模型

RegressorChain

```python
#第一个模型的输入和输出作为第二个模型的输入，以此类推
from sklearn.datasets import make_regression
from sklearn.multioutput import RegressorChain
from sklearn.svm import LinearSVR

X, y = make_regression(n_samples=1000, n_features=10, n_informative=5, n_targets=2, random_state=1)
data_in = [[-2.02220122, 0.31563495, 0.82797464, -0.30620401, 0.16003707, -1.44411381, 0.87616892, -0.50446586, 0.23009474, 0.76201118]]

model = LinearSVR()
wrapper = RegressorChain(model)

wrapper.fit(X, y)

yhat = wrapper.predict(data_in)
```



### 多输出分类

目前,sklearn.metrics中没有评估方法能够支持多输出多分类任务。



1.固有的多输出多分类

```
sklearn.tree.DecisionTreeClassifier
sklearn.tree.ExtraTreeClassifier
sklearn.ensemble.ExtraTreesClassifier
sklearn.neighbors.KNeighborsClassifier
sklearn.neighbors.RadiusNeighborsClassifier
sklearn.ensemble.RandomForestClassifier
```



2.每一个输出单独建立一个分类模型

```python
#假设每个输出之间都是相互独立的
from sklearn.datasets import make_classification
from sklearn.multioutput import MultiOutputClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.utils import shuffle
import numpy as np

X, y1 = make_classification(n_samples=10, n_features=100, n_informative=30, n_classes=3, random_state=1)
y2 = shuffle(y1, random_state=1)
y3 = shuffle(y1, random_state=2)
Y = np.vstack((y1, y2, y3)).T

forest = RandomForestClassifier(n_estimators=100, random_state=1)

multi_target_forest = MultiOutputClassifier(forest, n_jobs=-1)

multi_target_forest.fit(X, Y).predict(X)

>>> array([[2, 2, 0],
       [1, 2, 1],
       [2, 1, 0],
       [0, 0, 2],
       [0, 2, 1],
       [0, 0, 2],
       [1, 1, 0],
       [1, 1, 1],
       [0, 0, 2],
       [2, 0, 0]])
```



3.ClassifierChain

见多输出回归对应部分或者官网。



# 半监督学习

`LabelPropagation`和`LabelSpreading`可以预测y缺失的数据集，只要把缺失部分换成-1即可预测

```python
#以下代码用于比较LabelSpreading预测y缺失数据集的准确度
#点的颜色是真实值。背景色是预测值
import numpy as np
import matplotlib.pyplot as plt
from sklearn import datasets
from sklearn import svm
from sklearn.semi_supervised import LabelSpreading

iris = datasets.load_iris()
X = iris.data[:, :2]
y = iris.target
print(y)

#准备缺失y
rng = np.random.RandomState(0)
y_30 = np.copy(y)
y_30[rng.rand(len(y)) < 0.3] = -1
y_50 = np.copy(y)
y_50[rng.rand(len(y)) < 0.5] = -1

#准备模型
ls30 = (LabelSpreading().fit(X, y_30), y_30)
ls50 = (LabelSpreading().fit(X, y_50), y_50)
ls100 = (LabelSpreading().fit(X, y), y)
rbf_svc = (svm.SVC(kernel='rbf', gamma=.5).fit(X, y), y)

#准备画图
h = .02
x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
xx, yy = np.meshgrid(np.arange(x_min, x_max, h),np.arange(y_min, y_max, h))
titles = ['Label Spreading 30% data',
          'Label Spreading 50% data',
          'Label Spreading 100% data',
          'SVC with rbf kernel']
color_map = {-1: (1, 1, 1), 0: (0, 0, .9), 1: (1, 0, 0), 2: (.8, .6, 0)}

for i, (clf, y_train) in enumerate((ls30, ls50, ls100, rbf_svc)):
    plt.subplot(2, 2, i + 1)

    #根据预测结果绘制背景
    Z = clf.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    plt.contourf(xx, yy, Z, cmap=plt.cm.Paired)

    #每个点的x，y轴是特征，颜色是真实y
    colors = [color_map[y] for y in y_train]
    plt.scatter(X[:, 0], X[:, 1], c=colors, edgecolors='black')
    plt.title(titles[i])

plt.suptitle("Unlabeled points are colored white", y=0.1)
plt.show()
```



# 无监督学习

## 聚类

 scikit-learn 中的 聚类算法 的比较：

| Method name（方法名称）                                      | Parameters（参数）                                           | Scalability（可扩展性）                                      | Usecase（使用场景）                                          | Geometry (metric used)（几何图形（公制使用））               |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [K-Means（K-均值）](https://sklearn.apachecn.org/docs/master/22.html#k-means) | number of clusters（聚类形成的簇的个数）                     | 非常大的 `n_samples`, 中等的 `n_clusters` 使用 [MiniBatch 代码）](https://sklearn.apachecn.org/docs/master/22.html#mini-batch-kmeans) | 通用, 均匀的 cluster size（簇大小）, flat geometry（平面几何）, 不是太多的 clusters（簇） | Distances between points（点之间的距离）                     |
| [Affinity propagation](https://sklearn.apachecn.org/docs/master/22.html#affinity-propagation) | damping（阻尼）, sample preference（样本偏好）               | Not scalable with n_samples（n_samples 不可扩展）            | Many clusters, uneven cluster size, non-flat geometry（许多簇，不均匀的簇大小，非平面几何） | Graph distance (e.g. nearest-neighbor graph)（图距离（例如，最近邻图）） |
| [Mean-shift](https://sklearn.apachecn.org/docs/master/22.html#mean-shift) | bandwidth（带宽）                                            | Not scalable with `n_samples` （`n_samples`不可扩展）        | Many clusters, uneven cluster size, non-flat geometry（许多簇，不均匀的簇大小，非平面几何） | Distances between points（点之间的距离）                     |
| [Spectral clustering](https://sklearn.apachecn.org/docs/master/22.html#spectral-clustering) | number of clusters（簇的个数）                               | 中等的 `n_samples`, 小的 `n_clusters`                        | Few clusters, even cluster size, non-flat geometry（几个簇，均匀的簇大小，非平面几何） | Graph distance (e.g. nearest-neighbor graph)（图距离（例如最近邻图）） |
| [Ward hierarchical clustering](https://sklearn.apachecn.org/docs/master/22.html#hierarchical-clustering) | number of clusters（簇的个数）                               | 大的 `n_samples` 和 `n_clusters`                             | Many clusters, possibly connectivity constraints（很多的簇，可能连接限制） | Distances between points（点之间的距离）                     |
| [Agglomerative clustering](https://sklearn.apachecn.org/docs/master/22.html#hierarchical-clustering) | number of clusters（簇的个数）, linkage type（链接类型）, distance（距离） | 大的 `n_samples` 和 `n_clusters`                             | Many clusters, possibly connectivity constraints, non Euclidean distances（很多簇，可能连接限制，非欧氏距离） | Any pairwise distance（任意成对距离）                        |
| [DBSCAN](https://sklearn.apachecn.org/docs/master/22.html#dbscan) | neighborhood size（neighborhood 的大小）                     | 非常大的 `n_samples`, 中等的 `n_clusters`                    | Non-flat geometry, uneven cluster sizes（非平面几何，不均匀的簇大小） | Distances between nearest points（最近点之间的距离）         |
| [Gaussian mixtures（高斯混合）](https://sklearn.apachecn.org/docs/master/mixture.html#mixture) | many（很多）                                                 | Not scalable（不可扩展）                                     | Flat geometry, good for density estimation（平面几何，适用于密度估计） | Mahalanobis distances to centers（ 与中心的马氏距离）        |
| [Birch](https://sklearn.apachecn.org/docs/master/22.html#birch) | branching factor（分支因子）, threshold（阈值）, optional global clusterer（可选全局簇）. | 大的 `n_clusters` 和 `n_samples`                             | Large dataset, outlier removal, data reduction.（大型数据集，异常值去除，数据简化） | Euclidean distance between points（点之间的欧氏距离）        |



### K-means

```python
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans

X, y = make_blobs(n_samples=1500, random_state=42)
y_pred = KMeans(n_clusters=2, random_state=42).fit_predict(X)
```



### 层次聚类

AgglomerativeClustering

```python
#Ward, complete,average,single linkage四种策略
from sklearn.cluster import AgglomerativeClustering
import numpy as np

X = np.array([[1, 2], [1, 4], [1, 0],[4, 2], [4, 4], [4, 0]])
clustering = AgglomerativeClustering(linkage='ward', n_clusters=2).fit(X)

clustering.labels_
```



### DBSCAN

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn import datasets

X1, y1=datasets.make_circles(n_samples=5000, factor=.6,noise=.05)
X2, y2 = datasets.make_blobs(n_samples=1000, n_features=2, centers=[[1.2,1.2]], cluster_std=[[.1]], random_state=9)

X = np.concatenate((X1, X2))
plt.scatter(X[:, 0], X[:, 1], marker='o')
plt.show()

from sklearn.cluster import DBSCAN
y_pred = DBSCAN(eps = 0.1, min_samples = 10).fit_predict(X)
plt.scatter(X[:, 0], X[:, 1], c=y_pred)
plt.show()
```



## 降维

### pca（主成分分析）

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=3)
pca.fit(X)

X_new = pca.transform(X)
```



### Random Projection

```python
import numpy as np
from sklearn import random_projection

X = np.random.rand(100,10000)

transformer = random_projection.SparseRandomProjection()

X_new = transformer.fit_transform(X)
print(X_new.shape)
```



### FeatureAgglomeration

聚类后降维，把同一类的特征降维。

```python
import numpy as np
from sklearn.datasets import load_digits

digits = load_digits()
images = digits.images
X = np.reshape(images, (len(images), -1))
print(X.shape)

from sklearn.cluster import FeatureAgglomeration
agglo = FeatureAgglomeration(n_clusters=32)

agglo.fit(X)
X_reduced = agglo.transform(X)

print(X_reduced.shape)
```



## 异常检测

### 孤立森林

```python
from sklearn.ensemble import IsolationForest
import numpy as np

X = np.array([[-1, -1], [-2, -1], [-3, -2], [0, 0], [-20, 50], [3, 5]])

clf = IsolationForest(n_estimators=10, warm_start=True)
clf.fit(X)

clf.set_params(n_estimators=20)
clf.fit(X)  
```



# 交叉验证-超参数搜索

交叉验证是为了得到模型最好的参数。

超参数搜索是为了得到模型最好的超参数（同时搜索了超参数，进行了交叉验证）。使用了超参数搜索就不必有验证集，也不必交叉验证



### 交叉验证

**训练集**训练好模型后直接用**测试集**进行预测，可能过拟合，所以需要把训练集分为训练集和**验证集**。

但是这就减少了训练数据，而且验证结果受数据集划分影响。

所以要使用交叉验证（多次划分训练集和验证集）。



交叉验证的作用：

1.可以在划分训练集和验证集过程中使用**全量的 数据**进行训练。

2.得到模型的验证**平均评分**（各种指标）。

3.验证模型（超参数）稳定性。



交叉验证还很多验证方法，详见官网。下面列举了两种

```python
#k折交叉验证/分层k折（用于数据不平衡）
from sklearn.datasets import load_iris
from sklearn import svm
from sklearn.model_selection import cross_validate

iris = load_iris()

clf = svm.SVC(kernel='linear', C=1)
scores = cross_validate(clf, iris.data, iris.target, cv=5, scoring='f1_macro',return_estimator=True)

print(scores.keys())

print(scores['fit_time'])
print(scores['score_time'])
print(scores['estimator'])
print('Accuracy : %0.2f'%scores['test_score'].mean())
```

```python
#自定义交叉验证
from sklearn.datasets import load_iris
from sklearn import svm
from sklearn.model_selection import ShuffleSplit
from sklearn.model_selection import cross_validate

iris = load_iris()

clf = svm.SVC(kernel='linear', C=1)

cv = ShuffleSplit(n_splits=5, test_size=0.3, random_state=0)
print(cross_validate(clf, iris.data, iris.target, cv=cv))
```

```python
#时间序列数据交叉验证，数据需按照时间排列好
#原理是拿出前一部分数据，用后一部分验证。看结果就懂了
import numpy as np
from sklearn.model_selection import TimeSeriesSplit

X = np.array([[1, 2], [3, 4], [1, 2], [3, 4], [1, 2], [3, 4]])
y = np.array([1, 2, 3, 4, 5, 6])
tscv = TimeSeriesSplit(n_splits=3)

for train, test in tscv.split(X,y):
     print("%s %s" % (train, test))
```

在pipeline中，pipeline整体作为一个模型，应用在交叉验证中



### 超参数搜索

网格搜索

```python
from sklearn.datasets import load_iris
from sklearn.svm import SVC
from sklearn.model_selection import GridSearchCV, KFold

iris = load_iris()
X_iris = iris.data
y_iris = iris.target

p_grid = {"C": [1, 10, 100],"gamma": [.01, .1]}
svm = SVC(kernel="rbf")
cv = KFold(n_splits=4, shuffle=True, random_state=i)
 
clf = GridSearchCV(estimator=svm, param_grid=p_grid, cv=cv)
clf.fit(X_iris, y_iris)
print(clf.best_score_)
```



随机搜索

```python
from scipy.stats import randint as sp_randint
from sklearn.model_selection import RandomizedSearchCV
from sklearn.datasets import load_digits
from sklearn.ensemble import RandomForestClassifier

digits = load_digits()
X, y = digits.data, digits.target

clf = RandomForestClassifier(n_estimators=20)
param_dist = {"max_depth": [3, None],"max_features": sp_randint(1, 11),"min_samples_split": sp_randint(2, 11),
              "bootstrap": [True, False],"criterion": ["gini", "entropy"]}
n_iter_search = 20

random_search = RandomizedSearchCV(clf, param_distributions=param_dist,n_iter=n_iter_search, cv=5, iid=False)
random_search.fit(X, y)
```



# 模型评价

1.模型自带score方法

2.网格搜索和交叉验证的评分方法：可以同时进行多指标评价

3.sklearn.metrics的指标：`_score` 结尾返回值越高越好，`_error` 或 `_loss` 结尾返回值越低越好



所有涉及到多标签，多输出的都搞不太懂



### 网格搜索评价

1.现有的*网格得分*函数

| Scoring（得分）                | Function（函数）                                             | Comment（注解）                                              |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Classification（分类）**     |                                                              |                                                              |
| ‘accuracy’                     | [`metrics.accuracy_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html#sklearn.metrics.accuracy_score) |                                                              |
| ‘average_precision’            | [`metrics.average_precision_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html#sklearn.metrics.average_precision_score) |                                                              |
| ‘f1’                           | [`metrics.f1_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score) | for binary targets（用于二进制目标）                         |
| ‘f1_micro’                     | [`metrics.f1_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score) | micro-averaged（微平均）                                     |
| ‘f1_macro’                     | [`metrics.f1_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score) | macro-averaged（宏平均）                                     |
| ‘f1_weighted’                  | [`metrics.f1_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score) | weighted average（加权平均）                                 |
| ‘f1_samples’                   | [`metrics.f1_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score) | by multilabel sample（通过 multilabel 样本）                 |
| ‘neg_log_loss’                 | [`metrics.log_loss`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html#sklearn.metrics.log_loss) | requires `predict_proba` support（需要 `predict_proba` 支持） |
| ‘precision’ etc.               | [`metrics.precision_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html#sklearn.metrics.precision_score) | suffixes apply as with ‘f1’（后缀适用于 ‘f1’）               |
| ‘recall’ etc.                  | [`metrics.recall_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html#sklearn.metrics.recall_score) | suffixes apply as with ‘f1’（后缀适用于 ‘f1’）               |
| ‘roc_auc’                      | [`metrics.roc_auc_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html#sklearn.metrics.roc_auc_score) |                                                              |
| **Clustering（聚类）**         |                                                              |                                                              |
| ‘adjusted_mutual_info_score’   | [`metrics.adjusted_mutual_info_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.adjusted_mutual_info_score.html#sklearn.metrics.adjusted_mutual_info_score) |                                                              |
| ‘adjusted_rand_score’          | [`metrics.adjusted_rand_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.adjusted_rand_score.html#sklearn.metrics.adjusted_rand_score) |                                                              |
| ‘completeness_score’           | [`metrics.completeness_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.completeness_score.html#sklearn.metrics.completeness_score) |                                                              |
| ‘fowlkes_mallows_score’        | [`metrics.fowlkes_mallows_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.fowlkes_mallows_score.html#sklearn.metrics.fowlkes_mallows_score) |                                                              |
| ‘homogeneity_score’            | [`metrics.homogeneity_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.homogeneity_score.html#sklearn.metrics.homogeneity_score) |                                                              |
| ‘mutual_info_score’            | [`metrics.mutual_info_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mutual_info_score.html#sklearn.metrics.mutual_info_score) |                                                              |
| ‘normalized_mutual_info_score’ | [`metrics.normalized_mutual_info_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.normalized_mutual_info_score.html#sklearn.metrics.normalized_mutual_info_score) |                                                              |
| ‘v_measure_score’              | [`metrics.v_measure_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.v_measure_score.html#sklearn.metrics.v_measure_score) |                                                              |
| **Regression（回归）**         |                                                              |                                                              |
| ‘explained_variance’           | [`metrics.explained_variance_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.explained_variance_score.html#sklearn.metrics.explained_variance_score) |                                                              |
| ‘neg_mean_absolute_error’      | [`metrics.mean_absolute_error`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html#sklearn.metrics.mean_absolute_error) |                                                              |
| ‘neg_mean_squared_error’       | [`metrics.mean_squared_error`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html#sklearn.metrics.mean_squared_error) |                                                              |
| ‘neg_mean_squared_log_error’   | [`metrics.mean_squared_log_error`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html#sklearn.metrics.mean_squared_log_error) |                                                              |
| ‘neg_median_absolute_error’    | [`metrics.median_absolute_error`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html#sklearn.metrics.median_absolute_error) |                                                              |
| ‘r2’                           | [`metrics.r2_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.r2_score.html#sklearn.metrics.r2_score) |                                                              |

```python
from sklearn import svm, datasets
from sklearn.model_selection import cross_val_score

iris = datasets.load_iris()
X, y = iris.data, iris.target
clf = svm.SVC(probability=True, random_state=0)
cross_val_score(clf, X, y, scoring='neg_log_loss')
```



2.从[`sklearn.metrics`](https://sklearn.apachecn.org/docs/master/classes.html#module-sklearn.metrics)模块构造*网格得分*函数

因为有些评价指标有参数，所以需要自行构造

```
from sklearn.metrics import fbeta_score, make_scorer
from sklearn.model_selection import GridSearchCV
from sklearn.svm import LinearSVC

ftwo_scorer = make_scorer(fbeta_score, beta=2)

grid = GridSearchCV(LinearSVC(), param_grid={'C': [1, 10]}, scoring=ftwo_scorer)
```



3.自己写评价函数 构造*网格得分*函数

```python
import numpy as np
from sklearn.metrics import make_scorer
from sklearn.dummy import DummyClassifier

#构建了一个评分函数
def my_custom_loss_func(y_true, y_pred):
    diff = np.abs(y_true - y_pred).max()
    return np.log1p(diff)

#数据集
X = [[1], [1]]
y = [0, 1]

#训练模型
clf = DummyClassifier(strategy='most_frequent', random_state=0)
clf = clf.fit(X, y)

#进行评价
print(my_custom_loss_func(clf.predict(X), y))

#构造网格得分函数
score = make_scorer(my_custom_loss_func, greater_is_better=False)
#greater_is_better返回的是损失得分，即函数返回值的相反数
print(score(clf, X, y))
```



### 分类指标

其中一些仅限于二分类示例:

| 调用                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`precision_recall_curve`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_recall_curve.html#sklearn.metrics.precision_recall_curve)(y_true, probas_pred) | Compute precision-recall pairs for different probability thresholds |
| [`roc_curve`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_curve.html#sklearn.metrics.roc_curve)(y_true, y_score[, pos_label, …]) | Compute Receiver operating characteristic (ROC)              |

其他也可以在多分类示例中运行:

| 调用                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`cohen_kappa_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.cohen_kappa_score.html#sklearn.metrics.cohen_kappa_score)(y1, y2[, labels, weights, …]) | Cohen’s kappa: a statistic that measures inter-annotator agreement. |
| [`confusion_matrix`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.confusion_matrix.html#sklearn.metrics.confusion_matrix)(y_true, y_pred[, labels, …]) | Compute confusion matrix to evaluate the accuracy of a classification |
| [`hinge_loss`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.hinge_loss.html#sklearn.metrics.hinge_loss)(y_true, pred_decision[, labels, …]) | Average hinge loss (non-regularized)                         |
| [`matthews_corrcoef`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.matthews_corrcoef.html#sklearn.metrics.matthews_corrcoef)(y_true, y_pred[, …]) | Compute the Matthews correlation coefficient (MCC)           |

有些还可以在 multilabel case （多重示例）中工作:

| 调用                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`accuracy_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html#sklearn.metrics.accuracy_score)(y_true, y_pred[, normalize, …]) | Accuracy classification score.                               |
| [`classification_report`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.classification_report.html#sklearn.metrics.classification_report)(y_true, y_pred[, …]) | Build a text report showing the main classification metrics  |
| [`f1_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score)(y_true, y_pred[, labels, …]) | Compute the F1 score, also known as balanced F-score or F-measure |
| [`fbeta_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.fbeta_score.html#sklearn.metrics.fbeta_score)(y_true, y_pred, beta[, labels, …]) | Compute the F-beta score                                     |
| [`hamming_loss`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.hamming_loss.html#sklearn.metrics.hamming_loss)(y_true, y_pred[, labels, …]) | Compute the average Hamming loss.                            |
| [`jaccard_similarity_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.jaccard_similarity_score.html#sklearn.metrics.jaccard_similarity_score)(y_true, y_pred[, …]) | Jaccard similarity coefficient score                         |
| [`log_loss`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html#sklearn.metrics.log_loss)(y_true, y_pred[, eps, normalize, …]) | Log loss, aka logistic loss or cross-entropy loss.           |
| [`precision_recall_fscore_support`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_recall_fscore_support.html#sklearn.metrics.precision_recall_fscore_support)(y_true, y_pred) | Compute precision, recall, F-measure and support for each class |
| [`precision_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html#sklearn.metrics.precision_score)(y_true, y_pred[, labels, …]) | Compute the precision                                        |
| [`recall_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html#sklearn.metrics.recall_score)(y_true, y_pred[, labels, …]) | Compute the recall                                           |
| [`zero_one_loss`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.zero_one_loss.html#sklearn.metrics.zero_one_loss)(y_true, y_pred[, normalize, …]) | Zero-one classification loss.                                |

一些通常用于 ranking:

| 调用                                                         | 功能                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| [`dcg_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.dcg_score.html#sklearn.metrics.dcg_score)(y_true, y_score[, k]) | Discounted cumulative gain (DCG) at rank K.             |
| [`ndcg_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.ndcg_score.html#sklearn.metrics.ndcg_score)(y_true, y_score[, k]) | Normalized discounted cumulative gain (NDCG) at rank K. |

有些工作与 binary 和 multilabel （但不是多类）的问题:

| 调用                                                         | 功能                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| [`average_precision_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html#sklearn.metrics.average_precision_score)(y_true, y_score[, …]) | Compute average precision (AP) from prediction scores     |
| [`roc_auc_score`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html#sklearn.metrics.roc_auc_score)(y_true, y_score[, average, …]) | Compute Area Under the Curve (AUC) from prediction scores |



![4](picture/407341c3d4d055b857bb3229003b9daf.png)

![3771db7af1e3b7bf33e15ec20d278f39](picture/3771db7af1e3b7bf33e15ec20d278f39.png)

![b](picture/b3edbb24837112f795a22e3574457416.png)



confusion_matrix	混淆矩阵

```python
#二分类，多分类
from sklearn.metrics import confusion_matrix
y_true = [2, 0, 2, 2, 0, 1]
y_pred = [0, 0, 2, 2, 0, 2]
confusion_matrix(y_true, y_pred)
```



多标签混淆矩阵

```python
y_true = np.array([[0, 0, 1],[0, 1, 0],[1, 1, 0]])
y_pred = np.array([[0, 1, 0],[0, 0, 1],[1, 1, 0]])

mcm = multilabel_confusion_matrix(y_true, y_pred)
tn = mcm[:, 0, 0]
tp = mcm[:, 1, 1]
fn = mcm[:, 1, 0]
fp = mcm[:, 0, 1]
recall = tp / (tp + fn)
```



分类报告

```python
from sklearn.metrics import classification_report
y_true = [0, 1, 2, 2, 0]
y_pred = [0, 0, 2, 1, 0]
target_names = ['class 0', 'class 1', 'class 2']
print(classification_report(y_true, y_pred, target_names=target_names))
```



accuracy	准确度

```python
#应用于二分类，多分类，多标签评价
import numpy as np
from sklearn.metrics import accuracy_score

y_pred = [0, 2, 1, 3]
y_true = [0, 1, 2, 3]

accuracy_score(y_true, y_pred)
```

```python
#二分类多标签情况，每条数据的几个y有一个没对应，即为该条数据的y没对应。根据对应结果计算准确度
import numpy as np
from sklearn.metrics import accuracy_score

y_true = np.array([[0, 0], [1, 1],[1,1]])
y_pred = np.ones((3, 2))

print(accuracy_score(y_true,y_pred))
```



精确率、召回率、f1

```python
#二分类、多分类、多标签
from sklearn import metrics
y_pred = [0, 1, 0, 0]
y_true = [0, 1, 0, 1]

#average，评价多分类或者多标签分类的必须参数
#https://blog.csdn.net/hlang8160/article/details/78040311
metrics.precision_score(y_true, y_pred,average='macro')

metrics.recall_score(y_true, y_pred)

metrics.f1_score(y_true, y_pred)

#也是f1分数，beta>1，说明recall更重要
metrics.fbeta_score(y_true, y_pred, beta=2)
```

```python
import numpy as np

y_true = np.array([0, 0, 1, 1])
y_scores = np.array([0.1, 0.4, 0.35, 0.8])

#计算出不同阈值下的精确率和召回率
from sklearn.metrics import precision_recall_curve
precision, recall, threshold = precision_recall_curve(y_true, y_scores)
```



ROC曲线下的AUC得分

```python
#ROC曲线绘制
import numpy as np
from sklearn.metrics import roc_curve

y = np.array([1, 1, 2, 2])
scores = np.array([0.1, 0.4, 0.35, 0.8])

fpr, tpr, thresholds = roc_curve(y, scores, pos_label=2)
```

```python
#AUC得分计算
import numpy as np
from sklearn.metrics import roc_auc_score

y_true = np.array([0, 0, 1, 1])
y_scores = np.array([0.1, 0.4, 0.35, 0.8])

roc_auc_score(y_true, y_scores)
```



Cohen’s kappa

```python
#二分类，多分类
#用于比较不同人工标注的准确性，0.8以上说明两个人标注的都准确
from sklearn.metrics import cohen_kappa_score
y_true = [2, 0, 2, 2, 0, 1]
y_pred = [0, 0, 2, 2, 0, 2]
cohen_kappa_score(y_true, y_pred)
```



Jaccard 距离计算

```python
import numpy as np
from sklearn.metrics import jaccard_score

y_true = np.array([[0, 1, 1],[1, 1, 0]])
y_pred = np.array([[1, 1, 1],[1, 0, 0]])

jaccard_score(y_true[0], y_pred[0])  
```



log损失

```python
#logistic 回归损失  或者  交叉熵损失
from sklearn.metrics import log_loss

y_true = [0, 0, 1, 1]
y_pred = [[.9, .1], [.8, .2], [.3, .7], [.01, .99]]

log_loss(y_true, y_pred)  
```



### 回归指标

最大误差

```python
from sklearn.metrics import max_error
y_true = [3, 2, 7, 1]
y_pred = [9, 2, 7, 1]
max_error(y_true, y_pred)
```



平均绝对误差 (MAE) 

```python
from sklearn.metrics import mean_absolute_error
y_true = [3, -0.5, 2, 7]
y_pred = [2.5, 0.0, 2, 8]
mean_absolute_error(y_true, y_pred)

#多标签
y_true = [[0.5, 1], [-1, 1], [7, -6]]
y_pred = [[0, 2], [-1, 2], [8, -5]]
mean_absolute_error(y_true, y_pred)
```



均方误差（MSE）

```python
from sklearn.metrics import mean_squared_error

y_true = [3, -0.5, 2, 7]
y_pred = [2.5, 0.0, 2, 8]

mean_squared_error(y_true, y_pred)

#多标签
y_true = [[0.5, 1], [-1, 1], [7, -6]]
y_pred = [[0, 2], [-1, 2], [8, -5]]
mean_squared_error(y_true, y_pred)  
```



均方误差对数（MSLE）

```python
from sklearn.metrics import mean_squared_log_error

y_true = [3, 5, 2.5, 7]
y_pred = [2.5, 5, 4, 8]
mean_squared_log_error(y_true, y_pred)  

y_true = [[0.5, 1], [1, 2], [7, 6]]
y_pred = [[0.5, 2], [1, 2.5], [8, 8]]
mean_squared_log_error(y_true, y_pred)  
```



中位绝对误差（MedAE）

```python
from sklearn.metrics import median_absolute_error
y_true = [3, -0.5, 2, 7]
y_pred = [2.5, 0.0, 2, 8]
median_absolute_error(y_true, y_pred)
```



### 聚类指标

兰德指数

```python
#已知真实标签时 对聚类算法的评价
#真实数据一般没有，作为聚类模型选择过程中共识索引(Consensus Index)的一个构建模块是非常有用的

from sklearn import metrics
labels_true = [0, 0, 0, 1, 1, 1]
labels_pred = [0, 0, 1, 1, 2, 2]

metrics.adjusted_rand_score(labels_true, labels_pred)  
```



Silhouette 系数

![8f839ebe5b506fef19bd8cc121b3f557](picture/8f839ebe5b506fef19bd8cc121b3f557.png)

- **a**: 样本与同一类别中所有其他点之间的平均距离。
- **b**: 样本与 *下一个距离最近的簇* 中的所有其他点之间的平均距离。

```python
from sklearn import datasets
from sklearn.cluster import KMeans
from sklearn import metrics

dataset = datasets.load_iris()
X = dataset.data
y = dataset.target

kmeans_model = KMeans(n_clusters=3, random_state=1).fit(X)
labels = kmeans_model.labels_
metrics.silhouette_score(X, labels, metric='euclidean')
```



# 模型持久化

pickle

```python
import pickle

pickle.dump(model, open('model', 'wb'))
model = pickle.load(open('model', 'rb'))
```

joblib

```python
from sklearn.externals import joblib

joblib.dump(clf, 'filename.pkl')
clf = joblib.load('filename.pkl')
```



# Pipeline

### Pipeline

管道的所有评估器必须是转换器（ `transformer` ），最后一个评估器的类型不限。



Pipeline（串联）

```python
from sklearn.pipeline import Pipeline
from sklearn.svm import SVC
from sklearn.decomposition import PCA

pipe = Pipeline([('reduce_dim', PCA()), ('clf', SVC())])
```



make_pipeline（快速pipeline）

```python
from sklearn.pipeline import make_pipeline
from sklearn.naive_bayes import MultinomialNB
from sklearn.preprocessing import Binarizer

make_pipeline(Binarizer(), MultinomialNB())
```



```python
#查询
pipe.steps 
pipe['reduce_dim']

#设置嵌套参数
#clf 是我们在pipeline中设置的评估器名称，C 是对应的评估期参数名称
pipe.set_params(clf__C=10)
#在网格搜索中应用
from sklearn.model_selection import GridSearchCV
param_grid = dict(reduce_dim__n_components=[2, 5, 10],clf__C=[0.1, 10, 100])
grid_search = GridSearchCV(pipe, param_grid=param_grid)

#删除或者替换模型
pipe.set_params(clf='drop')
```



缓存 

```python
#默认不缓存，当开启缓存后，同样的数据，同样参数的中间transformer不会重新训练，只会重新训练改变了参数的transformer
from tempfile import mkdtemp
from shutil import rmtree

from sklearn.decomposition import PCA
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline

#缓存fit后的transformers
#应用在网格搜索
cachedir = mkdtemp()
pipe = Pipeline([('reduce_dim', PCA()), ('clf', SVC())], memory=cachedir)

#清除缓存
#rmtree(cachedir)
```



### FeatureUnion

把多个transformer 组合成一个transformer，输出组合了他们的输出。

每一个处理步骤都要处理所有的输入数据，然后把输出组合

```python
from sklearn.pipeline import FeatureUnion
from sklearn.decomposition import PCA
from sklearn.decomposition import KernelPCA

combined = FeatureUnion([('linear_pca', PCA()), ('kernel_pca', KernelPCA())])
```



### ColumnTransformer

把不同的列用不同的方法处理

```python
import pandas as pd
X = pd.DataFrame(
    {'city': ['London', 'London', 'Paris', 'Sallisaw'],
     'title': ["His Last Bow", "How Watson Learned the Trick",
               "A Moveable Feast", "The Grapes of Wrath"],
     'expert_rating': [5, 3, 4, 5],
     'user_rating': [4, 5, 4, 3]})

from sklearn.compose import ColumnTransformer
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.preprocessing import OneHotEncoder
from sklearn.preprocessing import MinMaxScaler

column_trans = ColumnTransformer([('city_category', OneHotEncoder(dtype='int'),['city']),('title_bow', CountVectorizer(), 'title')],remainder=MinMaxScaler())

print(column_trans.fit_transform(X))
```



#  特征提取

sklearn.feature_extraction



## 文本特征提取

Text feature extraction



### 词袋模型

Bag of words model



**词集模型 DictVectorizer**：单词构成的集合，集合中每个元素只有一个，即词集中的每个单词都只有一个。

**词袋模型 CountVectorizer**：在词集的基础上加入了频率这个维度，即统计单词在文档中出现的次数（令牌化和出现频数统计），通常我们在应用中都选用词袋模型。



##### DictVectorizer

把字典转换成   NumPy/SciPy   格式

```python
measurements = [
    {'city': 'Dubai', 'temperature': 33.},
    {'city': 'London', 'temperature': 12.},
    {'city': 'San Francisco', 'temperature': 18.}
]

from sklearn.feature_extraction import DictVectorizer
vec = DictVectorizer()

print(vec.fit_transform(measurements).toarray())
print(vec.get_feature_names())
```



##### Feature hashing

本类是DictVectorizer和CountVectorizer的低内存替代品。

把dict、pair、string转换成    NumPy/SciPy

```python
from sklearn.feature_extraction import FeatureHasher
h = FeatureHasher(n_features=10, input_type='string', dtype=int, alternate_sign=False)
#n_features输出特征个数

d = [{'dog': 1, 'cat': 2, 'elephant': 4}, {'dog': 2, 'run': 5}]
d = [[('dog', 1), ('cat', 2), ('elephant', 4)], [('dog', 2), ('run', 5)]]
d = [['dog', 'cat', 'cat', 'elephant', 'elephant' ,'elephant' ,'elephant' ,],
     ["dog", "dog", "run", 'run', 'run', 'run', 'run'],
     ["run", "run"]]

f = h.transform(d)
print(f.toarray())
```



##### CountVectorizer

CountVectorizer将每句话（文章）看做一条数据，每个token（字、词）是一个特征，值是token出现的次数。

```python
from sklearn.feature_extraction.text import CountVectorizer
vectorizer = CountVectorizer()

corpus = [
    'This is the first document.',
    'This is the second second document.',
    'And the third one.',
    'Is this the first document?',
]

X = vectorizer.fit_transform(corpus)
print(X.toarray())
print(vectorizer.get_feature_names())

print(vectorizer.transform(['Something completely new.']).toarray())

#ngram_range默认1个词为1个token，可以设置1，2，3个词都是1个token，这样可以把词的顺序信息训练进来
bigram_vectorizer = CountVectorizer(ngram_range=(1, 3))
X_2 = bigram_vectorizer.fit_transform(corpus).toarray()

print(X_2)
print(bigram_vectorizer.get_feature_names())
```



### TfidfTransformer

用于搜索、文档分类 、聚类



TF 词频

![20180806135836378](picture/20180806135836378.png)



IDF 逆文档频率 ：即词频的权重

![20180806140023860](picture/20180806140023860.png)

```python
from sklearn.feature_extraction.text import TfidfTransformer
transformer = TfidfTransformer(smooth_idf=False)

#一个列表代表一篇文章，共有6篇文章
#每一列代表一个词，3代表第一个词在第一篇文章里出现了3次。
#每一篇文章的每一个词都可以计算出tf-idf，最后把结果归一化
counts = [[3, 0, 1],
          [2, 0, 0],
          [3, 0, 0],
          [4, 0, 0],
          [3, 2, 0],
          [3, 0, 2]]


tfidf = transformer.fit_transform(counts)
print(tfidf.toarray())
```



### TfidfVectorizer

**CountVectorizer**  +  **TfidfTransformer**

```python
from sklearn.feature_extraction.text import TfidfVectorizer
vectorizer = TfidfVectorizer()
vectorizer.fit_transform(corpus)
```



## 图像特征提取

Image feature extraction



# 特征衍生

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures
X = np.arange(6).reshape(3, 2)

poly = PolynomialFeatures(2)
poly.fit_transform(X)
```



# 自定义特征处理

如果不使用函数，而使用  lambda  表达式，transformer  不能被  pickle

```python
import numpy as np
from sklearn.preprocessing import FunctionTransformer
transformer = FunctionTransformer(np.log1p)
X = np.array([[0, 1], [2, 3]])
transformer.transform(X)
```






















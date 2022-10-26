官网：https://scikit-learn.org/stable/modules/compose.html



# 概述

pipeline 把多个 模型&特征工程 组合成一个模型。便于上线调用。

pipeline 和 模型一样，可以训练、预测、评价、超参数搜索、持久化、部署。



pipeline的所有评估器必须是转换器（ `transformer` ），最后一个评估器的类型不限。



pipeline是流水线管道的意思。

下面有3个类用于转换X,，分别是**Pipeline、FeatureUnion、ColumnTransformer**，这三个类可以互相嵌套，用于设计流水线。

**TransformedTargetRegressor** 用于转换y。



当sklearn的评估器不能满足我们的需要时，简单计算可以使用 **FunctionTransformer**，复杂计算需要**自定义 Transformer 和 Estimator**。



构建pipeline的基本思路，Pipeline全部特征流式计算，FeatureUnion全部特征并行计算，ColumnTransformer选取特征并行计算，任意组合即可。



# TransformedTargetRegressor

用于转换 y ，详见 https://scikit-learn.org/stable/modules/compose.html



# Pipeline（串联）

```python
from sklearn.pipeline import Pipeline
from sklearn.svm import SVC
from sklearn.decomposition import PCA

pipe = Pipeline([('reduce_dim', PCA()), ('clf', SVC())])
```



# pipeline删改查

```python
#查询
pipe.steps 
pipe['reduce_dim']

#删除或者替换模型
pipe.set_params(clf='drop')

#设置参数
#clf 是我们在pipeline中设置的评估器名称，C 是对应的参数名称
pipe.set_params(clf_C=10)
```



# Pipeline网格搜索

```python
from sklearn.decomposition import PCA
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from tempfile import mkdtemp
from shutil import rmtree
from sklearn.model_selection import GridSearchCV

#默认不缓存，当开启缓存后，同样的数据，同样参数的中间transformer不会重新训练，只会重新训练改变了参数的transformer
cachedir = mkdtemp()	#清除缓存	rmtree(cachedir)
pipe = Pipeline([('reduce_dim', PCA()), ('clf', SVC())], memory=cachedir)

#在网格搜索中应用
param_grid = dict(reduce_dim_n_components=[2, 5, 10],clf_C=[0.1, 10, 100])
grid_search = GridSearchCV(pipe, param_grid=param_grid)
```



# FeatureUnion

把多个transformer 并联成一个transformer，每一个transformer 都要处理所有的输入数据，然后把他们的输出组合。

```python
from sklearn.pipeline import FeatureUnion
from sklearn.decomposition import PCA
from sklearn.decomposition import KernelPCA

combined = FeatureUnion([('linear_pca', PCA()), ('kernel_pca', KernelPCA())])
```



# ColumnTransformer

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

# remainder='drop',remainder='passthrough',verbose_feature_names_out=False
column_trans = ColumnTransformer(
    [
        ('city_category', OneHotEncoder(dtype='int'),['city']),
        ('title_bow', CountVectorizer(), 'title')
    ],
    remainder=MinMaxScaler())

print(column_trans.fit_transform(X))
```



# FunctionTransformer

简单的数值计算，类型转换等操作 可以用 FunctionTransformer 直接变成符合pipeline输入的 Transformer。

```python
def tofloat(x):
    return x.astype(np.float64)

tofloat_transformer = FunctionTransformer(tofloat, accept_sparse=True)
```



# 自定义 Transformer 

自定义 Transformer 和 Estimator，本部分内容仅供参考，还需整理。

```python
#异常值处理
#指定值替换
class ValueReplace(BaseEstimator,TransformerMixin):
    def __init__(self,column,orig_value,rep_value):
        self.column = column
        self.orig_value = orig_value
        self.rep_value = rep_value
    def fit(self,X,y=None):
        return self
    def transform(self,X):
        X[self.column].replace(self.orig_value,self.rep_value,inplace=True)
        return X
    
#特征衍生
#经验手动衍生
class FeatureDivision(BaseEstimator,TransformerMixin):
    def __init__(self):
        pass
    def fit(self,X,y=None):
        return self
    def transform(self,X):
        X["zd_bal_rat"] = X["zd_bal"]/X["acct_cred_limit"]
        X["in_bal_rat"] = X["in_bill_bal"]/X["acct_cred_limit"]
        X["all_bal_rat"] = (X["in_bill_bal"]+X["zd_bal"])/X["acct_cred_limit"]
        X["mean_deposit_3"] = (X["deposit"]+X["lm_deposit"]+X["lm2_deposit"])/3 
        X["deposit_bal_rat"] = X["deposit"]/X["zd_bal"]
        X["deposit_bal_bill_rat"] = X["deposit"]/(X["zd_bal"]+X["in_bill_bal"])
        return X
    def fit_transform(self,X,y=None):
```



# 自定义 futureunion

不知道干什么用的，先总结在这

```python
#返回dataframe的futureunion
class PandasFeatureUnion(FeatureUnion):
    def fit_transform(self,X,y=None,**fit_params):
        self._validate_transformers()
        result = Parallel(n_jobs=self.n_jobs)(
            delayed(_fit_transform_one)(
                transformer=trans,
                X=X,
                y=y,
                weight=weight,
                **fit_params)
            for name,trans,weight in self._iter())
        if not result:
            return np.zeros(x.shape[0],0)
        Xs,transformers = zip(*result)
        self._update_transformer_list(transformers)
        if any(sparse.issparse(f) for f in Xs):
            Xs = sparse.hstack(Xs).tocsr()
        else:
            Xs = self.merge_dataframes_by_column(Xs)
        return Xs
    
    def merge_dataframes_by_column(self,Xs):
        return pd.concat(Xs,axis='columns',copy=False)
    def transform(self,X):
        Xs = Parallel(n_jobs=self.n_jobs)(
            delayed(_transform_one)(
                transformer=trans,
                X=X,
                y=None,
                weight=weight)
            for name,trans,weight in self._iter())
        if not Xs:
            return np.zeros((x.shape[0],0))
        if any(sparse.issparse(f) for f in Xs):
            Xs=sparse.hstack(Xs).tocsr()
        else:
            Xs=self.merge_dataframes_by_column(Xs)
        return Xs
```












































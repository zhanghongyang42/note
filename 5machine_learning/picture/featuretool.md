https://zhuanlan.zhihu.com/p/93109455



https://docs.featuretools.com/en/stable/

https://www.kaggle.com/willkoehrsen/start-here-a-gentle-introduction/data



```python
import pandas as pd
import numpy as np
import featuretools as ft
```

pandas 与 ft 的 dtype 类型转换

```python
app_types[col] = ft.variable_types.Boolean
app_types['REGION_RATING_CLIENT'] = ft.variable_types.Ordinal
```

日期格式化

```python
example['date'].dt.days
df['purchase_date'] = pd.to_datetime(df['purchase_date'])
df['year'] = df['purchase_date'].dt.year
df['weekofyear'] = df['purchase_date'].dt.weekofyear
df['month'] = df['purchase_date'].dt.month
df['dayofweek'] = df['purchase_date'].dt.dayofweek
df['weekend'] = (df.purchase_date.dt.weekday >=5).astype(int)
df['hour'] = df['purchase_date'].dt.hour
```

自动构建特征

```python
import gc
import pandas as pd
import numpy as np
import featuretools as ft
import seaborn as sns
from featuretools.tests.selection.test_selection import feature_matrix
from sklearn.metrics import roc_auc_score
from sklearn.model_selection import KFold
from sklearn.preprocessing import LabelEncoder

plt.rcParams['font.size'] = 22
warnings.filterwarnings('ignore')

app_train = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\application_train.csv').sort_values('SK_ID_CURR').reset_index(drop = True).loc[:1000, :]
app_test = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\application_test.csv').sort_values('SK_ID_CURR').reset_index(drop = True).loc[:1000, :]
bureau = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\bureau.csv').sort_values(['SK_ID_CURR', 'SK_ID_BUREAU']).reset_index(drop = True).loc[:1000, :]
bureau_balance = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\bureau_balance.csv').sort_values('SK_ID_BUREAU').reset_index(drop = True).loc[:1000, :]
cash = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\POS_CASH_balance.csv').sort_values(['SK_ID_CURR', 'SK_ID_PREV']).reset_index(drop = True).loc[:1000, :]
credit = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\credit_card_balance.csv').sort_values(['SK_ID_CURR', 'SK_ID_PREV']).reset_index(drop = True).loc[:1000, :]
previous = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\previous_application.csv').sort_values(['SK_ID_CURR', 'SK_ID_PREV']).reset_index(drop = True).loc[:1000, :]
installments = pd.read_csv('C:\\Users\\zetyun\\PycharmProjects\\xgblr\\data\\installments_payments.csv').sort_values(['SK_ID_CURR', 'SK_ID_PREV']).reset_index(drop = True).loc[:1000, :]


app_train['set'] = 'train'
app_test['set'] = 'test'
app_test["TARGET"] = np.nan
app = app_train.append(app_test, ignore_index = True)


# 存储entity(类似df)及其relationship的集合,EntitySet,
# 为了自动的构造出特征，并执行单表中的transformer或者多表间的aggregation
es = ft.EntitySet(id = 'clients')

# entity
# 有唯一索引的entity
es = es.entity_from_dataframe(entity_id = 'app', dataframe = app, index = 'SK_ID_CURR')
es = es.entity_from_dataframe(entity_id = 'bureau', dataframe = bureau, index = 'SK_ID_BUREAU')
es = es.entity_from_dataframe(entity_id = 'previous', dataframe = previous, index = 'SK_ID_PREV')
# 没有唯一索引entity，自动生成索引
es = es.entity_from_dataframe(entity_id = 'bureau_balance', dataframe = bureau_balance, make_index = True, index = 'bureaubalance_index')
es = es.entity_from_dataframe(entity_id = 'cash', dataframe = cash, make_index = True, index = 'cash_index')
es = es.entity_from_dataframe(entity_id = 'installments', dataframe = installments,make_index = True, index = 'installments_index')
es = es.entity_from_dataframe(entity_id = 'credit', dataframe = credit,make_index = True, index = 'credit_index')

# relationship(确定父子表及其关联列)
print('Parent: app, Parent Variable: SK_ID_CURR\n\n', app.iloc[:, 111:115].head())
print('\nChild: bureau, Child Variable: SK_ID_CURR\n\n', bureau.iloc[10:30, :4].head())

r_app_bureau = ft.Relationship(es['app']['SK_ID_CURR'], es['bureau']['SK_ID_CURR'])
r_bureau_balance = ft.Relationship(es['bureau']['SK_ID_BUREAU'], es['bureau_balance']['SK_ID_BUREAU'])
r_app_previous = ft.Relationship(es['app']['SK_ID_CURR'], es['previous']['SK_ID_CURR'])
r_previous_cash = ft.Relationship(es['previous']['SK_ID_PREV'], es['cash']['SK_ID_PREV'])
r_previous_installments = ft.Relationship(es['previous']['SK_ID_PREV'], es['installments']['SK_ID_PREV'])
r_previous_credit = ft.Relationship(es['previous']['SK_ID_PREV'], es['credit']['SK_ID_PREV'])

es = es.add_relationships([r_app_bureau, r_bureau_balance, r_app_previous,r_previous_cash, r_previous_installments, r_previous_credit])
print(es)
# 不要创建菱形图


# Feature Primitives一些进行特征组合的基元
primitives = ft.list_primitives()
    #配置显示长度
pd.options.display.max_colwidth = 100

primitives[primitives['type'] == 'aggregation'].head(10)
primitives[primitives['type'] == 'transform'].head(10)


# Deep Feature Synthesis 深度为几就是几个操作
default_agg_primitives =  ["sum", "std", "max", "skew", "min", "mean", "count", "percent_true", "num_unique", "mode"]
default_trans_primitives =  ["day", "year", "month", "weekday", "haversine", "numwords", "characters"]

# 1
# feature_names = ft.dfs(entityset = es, target_entity = 'app',
#                        trans_primitives = default_trans_primitives,
#                        agg_primitives=default_agg_primitives,
#                        max_depth = 2, features_only=True)
#
# print('%d Total Features' % len(feature_names))
#
# 2
# feature_matrix, feature_names = ft.dfs(entityset = es, target_entity = 'app',
#                                        trans_primitives = default_trans_primitives,
#                                        agg_primitives=default_agg_primitives,
#                                         max_depth = 2, features_only=False, verbose = True)
# pd.options.display.max_columns = 1700
# feature_matrix.head(10)
#
# feature_names[-20:]
#1，2开销太大，基本不能执行

# 也不咋地
print(app)
feature_matrix_spec, feature_names_spec = ft.dfs(entityset = es, target_entity = 'app',
                                                 agg_primitives = ['sum', 'count', 'min', 'max', 'mean', 'mode'],
                                                 max_depth = 2, features_only = False, verbose = True)
pd.options.display.max_columns = 1000
feature_matrix_spec.head(10)

# 相关性检查
correlations = pd.read_csv('correlations_spec.csv', index_col = 0)
correlations.index.name = 'Variable'
correlations.head()


correlations_target = correlations.sort_values('TARGET')['TARGET']
correlations_target.head()
correlations_target.dropna().tail()


features_sample = pd.read_csv('feature_matrix.csv', nrows = 20000)
features_sample = features_sample[features_sample['set'] == 'train']
features_sample.head()

def kde_target_plot(df, feature):
    """Kernel density estimate plot of a feature colored
    by value of the target."""
    # Need to reset index for loc to workBU
    df = df.reset_index()
    plt.figure(figsize=(10, 6))
    plt.style.use('fivethirtyeight')

    # plot repaid loans
    sns.kdeplot(df.loc[df['TARGET'] == 0, feature], label='target == 0')
    # plot loans that were not repaid
    sns.kdeplot(df.loc[df['TARGET'] == 1, feature], label='target == 1')

    # Label the plots
    plt.title('Distribution of Feature by Target Value')
    plt.xlabel('%s' % feature)
    plt.ylabel('Density')
    plt.show()

kde_target_plot(features_sample, feature = 'MAX(previous_app.MEAN(credit.CNT_DRAWINGS_ATM_CURRENT))')



# 共线特征检查
threshold = 0.9

correlated_pairs = {}

# Iterate through the columns
for col in correlations:
    # Find correlations above the threshold
    above_threshold_vars = [x for x in list(correlations.index[correlations[col] > threshold]) if x != col]
    correlated_pairs[col] = above_threshold_vars
correlated_pairs['MEAN(credit.AMT_PAYMENT_TOTAL_CURRENT)']
correlations['MEAN(credit.AMT_PAYMENT_TOTAL_CURRENT)'].sort_values(ascending=False).head()

plt.plot(features_sample['MEAN(credit.AMT_PAYMENT_TOTAL_CURRENT)'], features_sample['MEAN(previous_app.MEAN(credit.AMT_PAYMENT_CURRENT))'], 'bo')
plt.title('Highly Correlated Features')

# 特征重要性查看
fi = pd.read_csv('spec_feature_importances_ohe.csv', index_col = 0)
fi = fi.sort_values('importance', ascending = False)
fi.head(15)
kde_target_plot(features_sample, feature = 'MAX(bureau.DAYS_CREDIT)')
original_features = list(pd.get_dummies(app).columns)
created_features = []

# Iterate through the top 100 features
for feature in fi['feature'][:100]:
    if feature not in original_features:
        created_features.append(feature)

print('%d of the top 100 features were made by featuretools' % len(created_features))

import matplotlib.pyplot as plt
plt.rcParams['font.size'] = 22

def plot_feature_importances(df):
    # Sort features according to importance
    df = df.sort_values('importance', ascending=False).reset_index()

    # Normalize the feature importances to add up to one
    df['importance_normalized'] = df['importance'] / df['importance'].sum()

    # Make a horizontal bar chart of feature importances
    plt.figure(figsize=(14, 10))
    ax = plt.subplot()

    # Need to reverse the index to plot most important on top
    ax.barh(list(reversed(list(df.index[:15]))),
            df['importance_normalized'].head(15),
            align='center', edgecolor='k')

    # Set the yticks and labels
    ax.set_yticks(list(reversed(list(df.index[:15]))))
    ax.set_yticklabels(df['feature'].head(15))

    # Plot labeling
    plt.xlabel('Normalized Importance')
    plt.title('Feature Importances')
    plt.show()

    return df
fi = plot_feature_importances(fi)
print('There are %d features with 0 importance' % sum(fi['importance'] == 0.0))

# 特征选择
from featuretools import selection
feature_matrix2 = selection.remove_low_information_features(feature_matrix)

print('Removed %d features' % (feature_matrix.shape[1]- feature_matrix2.shape[1]))


train = feature_matrix2[feature_matrix2['set'] == 'train']
test = feature_matrix2[feature_matrix2['set'] == 'test']
train = pd.get_dummies(train)
test = pd.get_dummies(test)


print('Final Training Shape: ', train.shape)
print('Final Testing Shape: ', test.shape)



# -----------------------moxing
def model(features, test_features, encoding = 'ohe', n_folds = 5):

    """Train and test a light gradient boosting model using
    cross validation.

    Parameters
    --------
        features (pd.DataFrame):
            dataframe of training features to use
            for training a model. Must include the TARGET column.
        test_features (pd.DataFrame):
            dataframe of testing features to use
            for making predictions with the model.
        encoding (str, default = 'ohe'):
            method for encoding categorical variables. Either 'ohe' for one-hot encoding or 'le' for integer label encoding
            n_folds (int, default = 5): number of folds to use for cross validation

    Return
    --------
        submission (pd.DataFrame):
            dataframe with `SK_ID_CURR` and `TARGET` probabilities
            predicted by the model.
        feature_importances (pd.DataFrame):
            dataframe with the feature importances from the model.
        valid_metrics (pd.DataFrame):
            dataframe with training and validation metrics (ROC AUC) for each fold and overall.

    """

    # Extract the ids
    train_ids = features['SK_ID_CURR']
    test_ids = test_features['SK_ID_CURR']

    # Extract the labels for training
    labels = features['TARGET']

    # Remove the ids and target
    features = features.drop(columns = ['SK_ID_CURR', 'TARGET'])
    test_features = test_features.drop(columns = ['SK_ID_CURR'])


    # One Hot Encoding
    if encoding == 'ohe':
        features = pd.get_dummies(features)
        test_features = pd.get_dummies(test_features)

        # Align the dataframes by the columns
        features, test_features = features.align(test_features, join = 'inner', axis = 1)

        # No categorical indices to record
        cat_indices = 'auto'

    # Integer label encoding
    elif encoding == 'le':

        # Create a label encoder
        label_encoder = LabelEncoder()

        # List for storing categorical indices
        cat_indices = []

        # Iterate through each column
        for i, col in enumerate(features):
            if features[col].dtype == 'object':
                # Map the categorical features to integers
                features[col] = label_encoder.fit_transform(np.array(features[col].astype(str)).reshape((-1,)))
                test_features[col] = label_encoder.transform(np.array(test_features[col].astype(str)).reshape((-1,)))

                # Record the categorical indices
                cat_indices.append(i)

    # Catch error if label encoding scheme is not valid
    else:
        raise ValueError("Encoding must be either 'ohe' or 'le'")

    print('Training Data Shape: ', features.shape)
    print('Testing Data Shape: ', test_features.shape)

    # Extract feature names
    feature_names = list(features.columns)

    # Convert to np arrays
    features = np.array(features)
    test_features = np.array(test_features)

    # Create the kfold object
    k_fold = KFold(n_splits = n_folds, shuffle = False, random_state = 50)

    # Empty array for feature importances
    feature_importance_values = np.zeros(len(feature_names))

    # Empty array for test predictions
    test_predictions = np.zeros(test_features.shape[0])

    # Empty array for out of fold validation predictions
    out_of_fold = np.zeros(features.shape[0])

    # Lists for recording validation and training scores
    valid_scores = []
    train_scores = []

    # Iterate through each fold
    for train_indices, valid_indices in k_fold.split(features):

        # Training data for the fold
        train_features, train_labels = features[train_indices], labels[train_indices]
        # Validation data for the fold
        valid_features, valid_labels = features[valid_indices], labels[valid_indices]

    #     # Create the model
    #     model = lgb.LGBMClassifier(n_estimators=10000, boosting_type = 'goss',
    #                objective = 'binary',
    #                                class_weight = 'balanced', learning_rate = 0.05,
    #                                reg_alpha = 0.1, reg_lambda = 0.1, n_jobs = -1, random_state = 50)
    #
    #     # Train the model
    #     model.fit(train_features, train_labels, eval_metric = 'auc',
    #               eval_set = [(valid_features, valid_labels), (train_features, train_labels)],
    #               eval_names = ['valid', 'train'], categorical_feature = cat_indices,
    #               early_stopping_rounds = 100, verbose = 200)
    #
    #     # Record the best iteration
    #     best_iteration = model.best_iteration_
    #
    #     # Record the feature importances
    #     feature_importance_values += model.feature_importances_ / k_fold.n_splits
    #
    #     # Make predictions
    #     test_predictions += model.predict_proba(test_features, num_iteration = best_iteration)[:, 1] / k_fold.n_splits
    #
    #     # Record the out of fold predictions
    #     out_of_fold[valid_indices] = model.predict_proba(valid_features, num_iteration = best_iteration)[:, 1]
    #
    #     # Record the best score
    #     valid_score = model.best_score_['valid']['auc']
    #     train_score = model.best_score_['train']['auc']
    #
    #     valid_scores.append(valid_score)
    #     train_scores.append(train_score)
    #
    #     # Clean up memory
    #     gc.enable()
    #     del model, train_features, valid_features
    #     gc.collect()
    #
    # # Make the submission dataframe
    # submission = pd.DataFrame({'SK_ID_CURR': test_ids, 'TARGET': test_predictions})
    #
    # # Make the feature importance dataframe
    # feature_importances = pd.DataFrame({'feature': feature_names, 'importance': feature_importance_values})
    #
    # # Overall validation score
    # valid_auc = roc_auc_score(labels, out_of_fold)
    #
    # # Add the overall scores to the metrics
    # valid_scores.append(valid_auc)
    # train_scores.append(np.mean(train_scores))
    #
    # # Needed for creating dataframe of validation scores
    # fold_names = list(range(n_folds))
    # fold_names.append('overall')
    #
    # # Dataframe of validation scores
    # metrics = pd.DataFrame({'fold': fold_names,
    #                         'train': train_scores,
    #                         'valid': valid_scores})
    #
    # return submission, feature_importances, metrics





```



```python
#定义es
es = ft.EntitySet(id = 'clients')
#加入df
es = es.entity_from_dataframe(entity_id = 'app_train', dataframe = app_train, index = 'SK_ID_CURR', variable_types = app_types)
es = es.entity_from_dataframe(entity_id = 'app_test', dataframe = app_test, index = 'SK_ID_CURR', variable_types = app_test_types)
es = es.entity_from_dataframe(entity_id = 'installments', dataframe = installments,make_index = True, index = 'installments_index',time_index = 'installments_paid_date')
es = es.entity_from_dataframe(entity_id = 'credit', dataframe = credit,make_index = True, index = 'credit_index',time_index = 'credit_balance_date')
#定义关系
r_app_bureau = ft.Relationship(es['app_train']['SK_ID_CURR'], es['bureau']['SK_ID_CURR'])
r_test_app_bureau = ft.Relationship(es['app_test']['SK_ID_CURR'], es['bureau']['SK_ID_CURR'])
#关系添加
es = es.add_relationships([r_app_bureau, r_test_app_bureau, r_bureau_balance, r_app_previous, r_test_app_previous])
#自动构建
time_features, time_feature_names = ft.dfs(entityset = es, target_entity = 'app_train',
                                           trans_primitives = ['cum_sum', 																'time_since_previous'], max_depth = 2,
                                           agg_primitives = ['trend'] ,#一些值随时间变化的趋													势(是个斜率)
                                           features_only = False, verbose = True,
                                           chunk_size = len(app_train),
                                           ignore_entities = ['app_test'])
```

```python
#创建一个自定义的基元
from featuretools.variable_types import (Boolean, Datetime,DatetimeTimeIndex,Discrete,Index,Numeric,Variable, Id)
from featuretools.primitives import AggregationPrimitive, make_agg_primitive
from datetime import datetime, timedelta
from collections import Counter



#返回x的出现最多的值的比例
def normalized_mode_count(x):
    #mode是一个基元，代表得到x列表中出现最多的数
    if x.mode().shape[0] == 0:
        return np.nan
    #生成一个{值：次数}的字典
    counts = dict(Counter(x.values))
    #得到次数最多的数的值
    mode = x.mode().iloc[0]
    return counts[mode] / np.sum(list(counts.values()))
NormalizedModeCount = make_agg_primitive(function = normalized_mode_count,
                                         input_types = [Discrete],#所有分类值的超类，也是ft中的一种数据类型
                                         return_type = Numeric)
#返回中连续出现最多的数
def longest_repetition(x):
    x = x.dropna()
    if x.shape[0] < 1:
        return None

    longest_element = current_element = None
    longest_repeats = current_repeats = 0

    for element in x:
        if current_element == element:
            current_repeats += 1
        else:
            current_element = element
            current_repeats = 1
        if current_repeats > longest_repeats:
            longest_repeats = current_repeats
            longest_element = current_element
    return longest_element
LongestSeq = make_agg_primitive(function = longest_repetition,
                                     input_types = [Discrete],
                                     return_type = Discrete)

custom_features, custom_feature_names = ft.dfs(entityset = es,
                                              target_entity = 'app_train',
                                              agg_primitives = [NormalizedModeCount, LongestSeq],
                                              max_depth = 2,
                                              trans_primitives = [],
                                              features_only = False, verbose = True,
                                              chunk_size = len(app_train),
                                              ignore_entities = ['app_test'])

custom_features.iloc[:, -40:].head()

plt.figure(figsize = (8, 6))
plt.bar(custom_features['LONGEST_REPETITION(previous.NAME_YIELD_GROUP)'].value_counts().index, custom_features['LONGEST_REPETITION(previous.NAME_YIELD_GROUP)'].value_counts(), edgecolor = 'k')
plt.xlabel('NAME_YIELD_GROUP'); plt.ylabel('Counts'); plt.title('Longest Repetition of Previous Name Yield Group')

plt.figure(figsize = (8, 6))
sns.kdeplot(custom_features['NORMALIZED_MODE_COUNT(previous.NAME_PRODUCT_TYPE)'], label = 'NORMALIZED_MODE_COUNT(previous.NAME_PRODUCT_TYPE)')
sns.kdeplot(custom_features['NORMALIZED_MODE_COUNT(bureau.CREDIT_ACTIVE)'], label = 'NORMALIZED_MODE_COUNT(bureau.CREDIT_ACTIVE)')


```






















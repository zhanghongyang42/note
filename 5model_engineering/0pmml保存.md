官网：https://github.com/jpmml?page=1

比较重要的项目包括：[sklearn2pmml](https://github.com/jpmml/sklearn2pmml)、[jpmml-sklearn](https://github.com/jpmml/jpmml-sklearn)、[sklearn2pmml-plugin](https://github.com/jpmml/sklearn2pmml-plugin)

介绍：https://www.cnblogs.com/pinard/p/9220199.html



# 简介

pmml 是一种跨多语言平台（包括java和python）的序列化格式。可以用来跨语言环境部署模型。

但是这种格式对模型来说有一定的精度损失。

pmml本质上就是将模型序列化成的XML文件。

实际上线时，上百维的结构化数据可以使用pmml这种方式上线。



# 使用

pmml 一般使用方法如下，直接构建好一个pipeline（见上一节pipeline），然后转换成pmml_pipeline,直接保存。

```python
# 保存
from sklearn2pmml import make_pmml_pipeline,sklearn2pmml

pipeline_pmml = make_pmml_pipeline(pipeline)
sklearn2pmml(pipeline_pmml, "./model.pmml", with_repr = True, debug = True)

#加载，一般用java
```



pipeline能支持的transformer很多，详见上一节 pipeline。

但是pmml支持的transformer不多，详见 https://github.com/jpmml/jpmml-sklearn#supported-packages 。



还有pmml关于一些非sklearn标准库使用的方法示例：https://github.com/jpmml/sklearn2pmml#documentation ，如改变数据域防止java调用报错。

更多pmml自己支持的方法，详见源码：https://github.com/jpmml/sklearn2pmml/tree/master/sklearn2pmml



# 自定义

若是sklearn标准库中没有，pmml也没有补充的数据处理方法，只能自己自定义。

自定义方法如下https://github.com/jpmml/sklearn2pmml-plugin



自定义原理是：

sklearn2pmml 中的每一个类，在java中都有一个对应的类去实现操作。

通过把python中类对象 名称和参数 序列化到 xml 文件中，java 去解析 xml 文件。

如果想要自定义新的transformer，只能按照上述方法，在python端和java端分别实现一个进行xml文件的序列化和解析。

任何在java端没有对应名称的方法，都无法被解析。
































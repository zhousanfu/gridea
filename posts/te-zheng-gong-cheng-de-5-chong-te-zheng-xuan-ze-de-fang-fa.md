---
title: '特征工程的5 种特征选择的方法'
date: 2021-04-07 11:53:05
tags: [数据挖掘]
published: true
hideInList: false
feature: 
isTop: false
---
特征选择是从原始特征中选择出一些最有效特征以降低数据集维度、提高法性能的方法。

我们知道模型的性能会随着使用特征数量的增加而增加。但是，当超过峰值时，模型性能将会下降。这就是为什么我们只需要选择能够有效预测的特征的原因。

特征选择类似于降维技术，其目的是减少特征的数量，但是从根本上说，它们是不同的。区别在于要素选择会选择要保留或从数据集中删除的要素，而降维会创建数据的投影，从而产生全新的输入要素。

特征选择有很多方法，在本文中我将介绍 Scikit-Learn 中 5 个方法，因为它们是最简单但却非常有用的，让我们开始吧。

### **1、方差阈值特征选择**

具有较高方差的特征表示该特征内的值变化大，较低的方差意味着要素内的值相似，而零方差意味着您具有相同值的要素。

方差选择法，先要计算各个特征的方差，然后根据阈值，选择方差大于阈值的特征，使用方法我们举例说明：

```
import pandas as pd
import seaborn as sns
mpg = sns.load_dataset('mpg').select_dtypes('number')
mpg.head()

```


对于此示例，我仅出于简化目的使用数字特征。在使用方差阈值特征选择之前，我们需要对所有这些数字特征进行转换，因为方差受数字刻度的影响。

```
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
mpg = pd.DataFrame(scaler.fit_transform(mpg), columns = mpg.columns)
mpg.head()

```


所有特征都在同一比例上，让我们尝试仅使用方差阈值方法选择我们想要的特征。假设我的方差限制为一个方差。

```
from sklearn.feature_selection import VarianceThreshold
selector = VarianceThreshold(1)
selector.fit(mpg)
mpg.columns[selector.get_support()]

```

方差阈值是一种无监督学习的特征选择方法。如果我们希望出于监督学习的目的而选择功能怎么办？那就是我们接下来要讨论的。

### **2、SelectKBest特征特征**

单变量特征选择是一种基于单变量统计检验的方法，例如：chi2，Pearson等等。

SelectKBest 的前提是将未经验证的统计测试与基于 X 和 y 之间的统计结果选择 K 数的特征相结合。

```
mpg = sns.load_dataset('mpg')
mpg = mpg.select_dtypes('number').dropna()
#Divide the features into Independent and Dependent Variable
X = mpg.drop('mpg' , axis =1)
y = mpg['mpg']

```

由于单变量特征选择方法旨在进行监督学习，因此我们将特征分为独立变量和因变量。接下来，我们将使用SelectKBest，假设我只想要最重要的两个特征。

```
from sklearn.feature_selection import SelectKBest, mutual_info_regression
#Select top 2 features based on mutual info regression
selector = SelectKBest(mutual_info_regression, k =2)
selector.fit(X, y)
X.columns[selector.get_support()]

```

### **3、递归特征消除(RFE)**

递归特征消除或RFE是一种特征选择方法，利用机器学习模型通过在递归训练后消除最不重要的特征来选择特征。

根据Scikit-Learn，RFE是一种通过递归考虑越来越少的特征集来选择特征的方法。

- 首先对估计器进行初始特征集训练，然后通过coef_attribute或feature_importances_attribute获得每个特征的重要性。
- 然后从当前特征中删除最不重要的特征。在修剪后的数据集上递归地重复该过程，直到最终达到所需的要选择的特征数量。

在此示例中，我想使用泰坦尼克号数据集进行分类问题，在那里我想预测谁将生存下来。

```
#Load the dataset and only selecting the numerical features for example purposes
titanic = sns.load_dataset('titanic')[['survived', 'pclass', 'age', 'parch', 'sibsp', 'fare']].dropna()
X = titanic.drop('survived', axis = 1)
y = titanic['survived']

```

我想看看哪些特征最能帮助我预测谁可以幸免于泰坦尼克号事件。让我们使用LogisticRegression模型获得最佳特征。

```
from sklearn.feature_selection import RFE
from sklearn.linear_model import LogisticRegression
# #Selecting the Best important features according to Logistic Regression
rfe_selector = RFE(estimator=LogisticRegression(),n_features_to_select = 2, step = 1)
rfe_selector.fit(X, y)
X.columns[rfe_selector.get_support()]

```

默认情况下，为RFE选择的特征数是全部特征的中位数，步长是1.当然，你可以根据自己的经验进行更改。

### **4、SelectFromModel 特征选择**

Scikit-Learn 的 SelectFromModel 用于选择特征的机器学习模型估计，它基于重要性属性阈值。默认情况下，阈值是平均值。

让我们使用一个数据集示例来更好地理解这一概念。我将使用之前的数据。

```
from sklearn.feature_selection import SelectFromModel
sfm_selector = SelectFromModel(estimator=LogisticRegression())
sfm_selector.fit(X, y)
X.columns[sfm_selector.get_support()]

```

与RFE一样，你可以使用任何机器学习模型来选择功能，只要可以调用它来估计特征重要性即可。你可以使用随机森林模或XGBoost进行尝试。

### **5、顺序特征选择(SFS)**

顺序特征选择是一种贪婪算法，用于根据交叉验证得分和估计量来向前或向后查找最佳特征，它是 Scikit-Learn 版本0.24中的新增功能。方法如下：

- SFS-Forward 通过从零个特征开始进行功能选择，并找到了一个针对单个特征训练机器学习模型时可以最大化交叉验证得分的特征。
- 一旦选择了第一个功能，便会通过向所选功能添加新功能来重复该过程。当我们发现达到所需数量的功能时，该过程将停止。

让我们举一个例子说明。

```
from sklearn.feature_selection import SequentialFeatureSelector

sfs_selector = SequentialFeatureSelector(estimator=LogisticRegression(), n_features_to_select = 3, cv =10, direction ='backward')
sfs_selector.fit(X, y)
X.columns[sfs_selector.get_support()]

```

### **结论**

特征选择是机器学习模型中的一个重要方面，对于模型无用的特征，不仅影响模型的训练速度，同时也会影响模型的效果。
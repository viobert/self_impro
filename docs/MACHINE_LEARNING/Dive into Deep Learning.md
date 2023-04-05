# Dive into Deep Learning

## Install

Install Pytorch（CPU or GPU）

```shell
mkdir d2l-zh && cd d2l-zh
curl https://zh-v2.d2l.ai/d2l-zh-2.0.0.zip -o d2l-zh.zip
unzip d2l-zh.zip && rm d2l-zh.zip
cd pytorch
```

Install d2l package

```shell
pip install d2l==0.17.6
```



## Key Components and Classification

### Key Components

1. Data
    + 每个**数据集**`Data set`由一个个**样本**`example,sample`组成
    + **样本**也叫做**数据点**`data point`或者**数据实例**`data instance`
    + 通常每一个**样本**由一组成为**特征**`feature,或称协变量 covariates`的属性组成。
    + **模型**`Model`会根据这些属性进行预测，预测的属性称为**标签**`label, 目标target`
    + 正常的，数据集也有两类：一类是由为了训练而收集的样本组成，叫做**训练集**`training set`; 另一种是为了测试的，新的数据集，称为**测试集**`test set`
    + 但是，一个模型在训练集上表现良好，未必可以在测试集上表现的很好，这时这个模型称为过拟合`overfitting`的
2. Models
    + 使用数据集来选择参数的元程序被称为**学习算法**`learning algorithm`
    + 根据参数调整后的程序称为**模型**`Model`
3. Objective Functions
    + 在机器学习中，我们需要定义模型的优劣程度的度量，这个度量在大多数情况是“可优化”的，这被称之为**目标函数**`objective function`。
    + 我们通常定义一个目标函数，并希望优化它到最低点。 因为越低越好，所以这些函数有时被称为损失函数`loss function，或cost function`。
4. Optimization Algorithm
    + **优化算法**是指：当我们获得了一些数据源及其表示、一个模型和一个合适的损失函数，接下来就需要一种算法，它能够搜索出最佳参数，以最小化损失函数。 深度学习中，大多流行的**优化算法**通常基于一种基本方法——**梯度下降**`gradient descent`
    + 。 简而言之，在每个步骤中，梯度下降法都会检查每个参数，看看如果仅对该参数进行少量变动，训练集损失会朝哪个方向移动。 然后，它在可以减少损失的方向上优化参数。

#### classfication

##### 1. Supervised learning

##### Regression

回归问题：有多少

##### Classification

分类问题：哪一个；输出被规定在有限个离散值中。

##### Tagging

标记问题：是分类问题的推广。

分类问题的输出是一个值，而标注问题输出是一个向量，向量的每个值属于一种标记类型。

##### Search

搜索：结果集很重要，但是有些问题更需要的是结果集内部的排序，排名

##### Recommender

推荐系统：个性化的搜索

##### Sequence Learning

序列学习：

如果输入的样本之间没有任何关系，以上模型可能完美无缺。 但是如果输入是连续的，模型可能就需要拥有“记忆”功能。 比如，我们该如何处理视频片段呢？ 在这种情况下，每个视频片段可能由不同数量的帧组成。 通过前一帧的图像，我们可能对后一帧中发生的事情更有把握。 语言也是如此，机器翻译的输入和输出都为文字序列。

##### 2. Unsupervised learning 

##### 3. Interacting with an Environment 

##### 4. Reinforcement Learning


























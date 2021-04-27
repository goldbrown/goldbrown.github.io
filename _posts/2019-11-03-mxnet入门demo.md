---
layout:     post
title:      mxnet入门demo
subtitle:   
date:       2019-11-03
author:     Chris
header-img: img/post-bg-dl.jpeg
catalog: true
tags:
    - 深度学习
    - 神经网络
    - mxnet
---


今天在PC上简单跑了一个mxnet入门例子，记录下来。

# mxnet简介
mxnet是一款深度学习框架，在几款常用的深度学习框架中比较靠前。它的优点是：   
* 消耗的内存少   
这一点，可以让mxnet运行在内存较小的设备上面，比如手机或者IOT设备。
* 易扩展   
可以方便扩展到多个gpu和多个计算机。
* 灵活性   
提供命令式和符号式两种类型的编程API。命令式API方便开发者学习，容易调试和修改超参数等。

# 通用的深度学习训练过程

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8mcdyjybxj30ko0podg1.jpg" height="50%" width="50%">

# 入门demo
下面的例子，是使用mxnet训练一个两层的深度学习网络。网络结构如下。

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8mc0lnucrj30gu0xgdg8.jpg" height="50%" width="50%">

遵循上面的步骤，我们来一步步训练一个模型，并使用模型进行预测。

## 步骤1：收集数据
下面的代码从某个函数中生成数据。

```python
import pickle
import numpy as np
# define boundary
def cos_curve(x):
    return 0.25*np.sin(2*x*np.pi+0.5*np.pi) + 0.5
# samples store point, labels store label
np.random.seed(1234)
samples = []
labels = []
# sample number
sample_density = 50
for i in range(sample_density):
    x1, x2 = np.random.random(2)
    # computer boundary
    bound = cos_curve(x1)
    # ignore the sample near the boundary
    if bound - 0.1 < x2 <= bound + 0.1:
        continue
    else:
        samples.append((x1, x2))
        # set the label
        if x2 > bound:
            labels.append(1)
        else:
            labels.append(0)

# store the samples and labels
with open('data.pkl', 'wb') as f:
    pickle.dump((samples, labels), f)
# visualize
import matplotlib.pyplot as plt
for i, sample in enumerate(samples):
    plt.plot(sample[0], sample[1], 'o' if labels[i] else '^', 
            mec='r' if labels[i] else 'b',
            mfc='none',
            markerSize=10)
x1 = np.linspace(0, 1)
plt.plot(x1, cos_curve(x1), 'k--')
plt.show()
```

## 步骤2：建立模型

```python
import numpy as np
import mxnet as mx
# define data input
data = mx.sym.Variable('data')
# data flow through a fully connected layer with two outputs
fc1 = mx.sym.FullyConnected(data=data, name='fc1', num_hidden=2)
# a sigmoid activation layer
sigmoid1 = mx.sym.Activation(data=fc1, name='sigmoid1', act_type='sigmoid')
# another fully connected layer with two outputs
fc2 = mx.sym.FullyConnected(data=sigmoid1, name='fc2', num_hidden=2)
# output by softmax
mlp = mx.sym.SoftmaxOutput(data=fc2, name='softmax')
# visualize
shape = {'data': (2,)}
mlp_dot = mx.viz.plot_network(symbol=mlp, shape=shape)
mlp_dot.render('simple_mlp.gv', view=True)
```


## 步骤3：训练模型参数

```python
# train model
import pickle
import logging
# read data

with open('data.pkl', 'rb') as f:
    samples, labels = pickle.load(f)
# logging level
logging.getLogger().setLevel(logging.WARN)
# batch grandient descend
batch_size = len(labels)
samples = np.array(samples)
labels = np.array(labels)
# configure model param
train_iter = mx.io.NDArrayIter(samples, labels, batch_size)
model = mx.model.FeedForward(
    symbol=mlp,
    num_epoch=1000,
    learning_rate=0.1,
    momentum=0.99
)
# fit model
model.fit(X=train_iter)
# predict
print(model.predict(mx.nd.array([[0.5, 0.5]])))
```

## 步骤4：预测以及可视化训练结果


```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

# axes grid length
X = np.arange(0, 1.05, 0.05)
Y = np.arange(0, 1.05, 0.05)
X, Y = np.meshgrid(X, Y)
grids = mx.nd.array([[X[i][j], Y[i][j]] for i in range(X.shape[0]) for j in range(X.shape[1])])
# get result of predicted
grid_probs = model.predict(grids)[:, 1].reshape(X.shape)
# drawing
fig = plt.figure('sample surface')
ax = fig.gca(projection='3d')
ax.plot_surface(X, Y, grid_probs, alpha=0.15, color='k',rstride=2, cstride=2, lw=0.5)
# select samples
samples0 = samples[labels==0]
samples0_probs = model.predict(samples0)[:, 1]
samples1 = samples[labels==1]
samples1_probs = model.predict(samples1)[:, 1]
# plot sample figure
ax.scatter(samples0[:, 0], samples0[:, 1], samples0_probs, c='r', marker='o', s=50)
ax.scatter(samples1[:, 0], samples1[:, 1], samples1_probs, c='b', marker='^', s=50)
plt.show()
```

注意，上面的代码中，仍然用模型来预测训练样本的结果，这样的做法其实是不规范的。只不过，我们为了方便，所以这么做了。

# 参考
《深度学习与计算机视觉：算法原理、框架应用与代码实现》




> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NTUwNjQ1Mw==&mid=2247503131&idx=1&sn=f466a18f9f2169b8b7ba57db7da2617f&chksm=fcb835e1cbcfbcf76cf21026d609bd40917a3ffad66741023b5fcce0d6430c5520ea57620a54&mpshare=1&scene=1&srcid=0605jK5Fr4AWVIo4EEqAo0RR&sharer_sharetime=1622826188227&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

摘要

在今天的报告中，我们将分析食品以预测它们是否可以食用。我们应用最先进的 迁移学习方法和 Tensorflow 框架来构建用于食品分类的机器学习模型。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ABvEnMciauWvr7XBBKmM6wjUdqgZs3JF4UOCsFnjrqgkITT5eibiaYXvYBRKJrdMlM07CWhocrEZ0aBxC7ibeE07ZA/640?wx_fmt=jpeg)

### 介绍

图像分类是机器预测图片属于哪个类别的工作。在深度学习开始蓬勃发展之前，图像分类等任务无法达到人类水平。

这是因为机器学习模型无法确定图像的邻域知识。模型只接收像素级命令。

由于深度学习的潜力，图像分类任务可以利用被描述为卷积神经网络 (CNN) 的模型来传递人类水平的性能。

CNN 是一种研究图像表征的深度学习模型。该模型无需个人参与即可确定从平面到高级的特征。

该模型不仅接收像素级别的数据。该模型还通过称为卷积的机制从图像中获取相邻数据。

卷积将通过将范围内的像素的编译相乘并将它们相加为一个值来聚合邻域数据。ML 模型将接受这些特征以将图片分类为一组。

深度学习虽然可以完成人类级别的生产，但需要大量的数据。

**如果我们没有它们怎么办？我们可以应用一种称为迁移学习的理论。**

迁移学习是一种在海量数据上为我们的查询训练模型的技术。因此，我们仅通过微调模型来准备它们。我们将注意到的优势是模型将在一段时间内学会。

本文将帮助你练习使用 TensorFlow (Python) 进行食品图像分类的迁移学习。因为预处理步骤是基本过程，我还将解释如何将数据提供给我们的深度学习模型。

### 执行

##### 步骤 1：导入库

我们要求做的第一步是导入库。我们想要 TensorFlow、NumPy、os 和 pandas。如果你还没有修复包，你可以应用 pip 命令来安装所需的库。

> 请注意，我命令使用的 TensorFlow 是 2.4.1 版本，因此请安装该版本。

另外，如果你希望采用 GPU 来训练深度学习模型，请安装 11.0 版的 CUDA，因为该版本包含 2.4.1 版的 TensorFlow。

> 下载 CUDA 软件 https://developer.nvidia.com/cuda-11.0-download-archive

**下面是安装和加载所需库的代码。**

```
# ! pip install tensorflow==2.4.1
# ! pip install pandas
# ! pip install numpy
import os
import numpy as np
import pandas as pd
import tensorflow as tf
```

##### 步骤 2：准备数据

安排好库后，下一步是修复我们的数据集。在此示例中，我们将应用名为 Food-5K 的数据集。

*   https://www.kaggle.com/binhminhs10/food5k
    

该数据集由 5000 张图片组成，分为两类，即食物和非食物。FOOD-5K 分为训练、验证和测试数据集。

**数据集文件夹的格式如下：**

正如你在上面看到的，每个文件夹都由图片组成。每个图片文件名包括类别和由下划线分隔的标识符。

我们需要产生带有图像文件名的列和带有该文件夹排列的标签的数据框。

**修复数据集的代码如下所示，**

```
def dframe(dtype):
    X = []
    y = []
    path = 'Food-5K/' + dtype + '/'
    for i in os.listdir(path):
        # Image
        X.append(i)
        # Label
        y.append(i.split('_')[0])
    X = np.array(X)
    y = np.array(y)
    df = pd.DataFrame()
    df['filename'] = X
    df['label'] = y
    return df
df_train = dframe('training')
df_val = dframe('validation')
df_test = dframe('evaluation')
```

```
df_train.head()
```

这是数据框的视图，

下一步就是制作一个对象，将图片放入模型中。我们将练习 _tf.keras.preprocessing.image_ 库的 _ImageDataGenerator_ 对象。

使用此对象，我们将生成图像批次。此外，我们可以扩充我们的图片以扩大数据集的乘积。因为我们还扩展了这些图片，我们进一步设置了图像增强技术的参数。

此外，因为我们应用了一个数据帧作为关于数据集的知识，因此我们将使用 flow_from_dataframe 方法生成批处理并增强图片。

**上面的代码看起来像这样**

```
from tensorflow.keras.preprocessing.image import ImageDataGenerator
```

```
# Create the ImageDataGenerator object
train_datagen = ImageDataGenerator(
    featurewise_center=True,
    featurewise_std_normalization=True,
    rotation_range=20,
    width_shift_range=0.2,
    height_shift_range=0.2,
    horizontal_flip=True,
)
val_datagen = ImageDataGenerator(
    featurewise_center=True,
    featurewise_std_normalization=True,
    rotation_range=20,
    width_shift_range=0.2,
    height_shift_range=0.2,
    horizontal_flip=True,
)
# Generate batches and augment the images
train_generator = train_datagen.flow_from_dataframe(
    df_train,
    directory='Food-5K/training/',
    x_col='filename',
    y_col='label',
    class_mode='binary',
    target_size=(224, 224),
)
val_generator = train_datagen.flow_from_dataframe(
    df_val,
    directory='Food-5K/validation/',
    x_col='filename',
    y_col='label',
    class_mode='binary',
    target_size=(224, 224),
)
```

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWvr7XBBKmM6wjUdqgZs3JF48ibsy7cKYNFS3vo4WBnE8gtIxmCwETAicxvSzd4qBMZmtId7w0vEMnLw/640?wx_fmt=png)

##### 步骤 3：训练模型

决定好批次之后，我们可以通过迁移学习技术来训练模型。因为我们应用了这种方法，所以我们不需要从头开始执行 CNN 架构。相反，我们将使用当前和以前预训练的架构。

我们将应用 ResNet-50 作为我们新模型的脊椎。我们将生成输入并根据类别数量调整 ResNet-50 的最后一个线性层 ResNet-50。

**构建模型的代码如下**

```
from tensorflow.keras.applications import ResNet50

# Initialize the Pretrained Model
feature_extractor = ResNet50(weights='imagenet', 
                             input_shape=(224, 224, 3),
                             include_top=False)

# Set this parameter to make sure it's not being trained
feature_extractor.trainable = False

# Set the input layer
input_ = tf.keras.Input(shape=(224, 224, 3))

# Set the feature extractor layer
x = feature_extractor(input_, training=False)

# Set the pooling layer
x = tf.keras.layers.GlobalAveragePooling2D()(x)

# Set the final layer with sigmoid activation function
output_ = tf.keras.layers.Dense(1, activation='sigmoid')(x)

# Create the new model object
model = tf.keras.Model(input_, output_)

# Compile it
model.compile(optimizer='adam',
             loss='binary_crossentropy',
             metrics=['accuracy'])

# Print The Summary of The Model
model.summary()
```

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWvr7XBBKmM6wjUdqgZs3JF4eHJrrfunUrJ83u2tZPCyia0Jshm6sWMEicRG0Y5l1jMiaBceXiagTtkuCw/640?wx_fmt=png)

为了训练模型，我们采用拟合的方法来准备模型。这是代码，

```
model.fit(train_generator, epochs=20, validation_data=val_generator)
```

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWvr7XBBKmM6wjUdqgZs3JF4MgQ7ttzQpiaVdIUeqBwrpS8wBpzEsaWxB6icZAJnAtNLuAV6gYem6HvA/640?wx_fmt=png)

##### 步骤 4：测试模型

在训练模型之后，现在让我们在测试数据集上检查模型。在扩展中，我们需要结合一个 pillow 库来加载和调整图片大小，以及 scikit-learn 来确定模型性能。

我们将练习来自 scikit-learn 库的分类报告，以生成关于模型执行的报告。此外，我们会喜欢它的混淆矩阵。

**这是预测实验数据及其决策的代码**，

```
from PIL import Image
from sklearn.metrics import classification_report, confusion_matrix

y_true = []
y_pred = []

for i in os.listdir('Food-5K/evaluation'):
    img = Image.open('Food-5K/evaluation/' + i)
    img = img.resize((224, 224))
    img = np.array(img)
    img = np.expand_dims(img, 0)
    
    y_true.append(int(i.split('_')[0]))
    y_pred.append(1 if model.predict(img) > 0.5 else 0)
    
print(classification_report(y_true, y_pred))
print()
print(confusion_matrix(y_true, y_pred))
```

![](https://mmbiz.qpic.cn/mmbiz_png/ABvEnMciauWvr7XBBKmM6wjUdqgZs3JF4byhP1jACy32CfyFJgYcbdalmXVfSTiaQoHkQEIlQLgN8ZruyXUkxu0Q/640?wx_fmt=png)

> 从前面的内容可以看出，该模型的性能已超过 95％。因此，我们可以在建立图像分类器 API 的情况下接受此模型。

##### 步骤 5：保存模型

如果你希望将模型用于后续使用或部署，你可以使用 save 方法保存模型，

```
model.save('./resnet50_food_model')
```

如果你需要加载模型，你可以像这样练习 load_model 方法，

```
model = tf.keras.models.load_model('./resnet50_food_model')
```

### 下一步是什么

做得好！现在你已了解如何使用 TensorFlow 执行迁移学习。我希望这项研究能鼓励你，尤其是那些渴望在数据不足的情况下训练深度学习模型的人。

  

**☆ END ☆**
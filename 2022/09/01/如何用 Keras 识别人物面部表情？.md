> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903612817080328) 可以获取。 该数据集压缩后大小为 92 M，未压缩版本大小为 295 M。数据集中包含 2 万 8 千张训练照片和 3 万张测试照片。每一张照片都存储为 48 X 48 像素。纯数据集包含图像像素（48 X 48 = 2304 个值），每张图像中人脸的表情，和使用类型（用作训练或测试）。

把数据集下载到本地数据文件夹后，用如下代码读取数据集内容：

```
with open("/data/fer2013.csv") as f:
    content = f.readlines()
 
lines = np.array(content)
 
num_of_instances = lines.size
print("number of instances: ",num_of_instances)
复制代码

```

### 学习过程

最近几年，深度学习技术在计算机视觉研究占据了主要地位，即便是学术性的计算机视觉大会也紧密转为深度学习研讨会。在这里，我们会使用深度学习中的卷积神经网络完成本次任务。然后我们会利用 TensorFlow 的后端，使用 Keras 构建卷积神经网络。

我们已经将数据集下载到了本地，现在可以把训练集和测试集存储为专用变量了。

```
x_train, y_train, x_test, y_test = [], [], [], []
 
for i in range(1,num_of_instances):
 emotion, img, usage = lines[i].split(",")
 
 val = img.split(" ")
 pixels = np.array(val, 'float32')
 
 emotion = keras.utils.to_categorical(emotion, num_classes)
 
 if 'Training' in usage:
     y_train.append(emotion)
     x_train.append(pixels)
 elif 'PublicTest' in usage:
     y_test.append(emotion)
     x_test.append(pixels)
复制代码

```

然后构造卷积神经网络架构：

```
model = Sequential()
 
#第一个卷积层
model.add(Conv2D(64, (5, 5), activation='relu', input_shape=(48,48,1)))
model.add(MaxPooling2D(pool_size=(5,5), strides=(2, 2)))
 
#第二个卷积层
model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(AveragePooling2D(pool_size=(3,3), strides=(2, 2)))
 
#第三个卷积层
model.add(Conv2D(128, (3, 3), activation='relu'))
model.add(Conv2D(128, (3, 3), activation='relu'))
model.add(AveragePooling2D(pool_size=(3,3), strides=(2, 2)))
 
model.add(Flatten())
 
#全连接神经网络
model.add(Dense(1024, activation='relu'))
model.add(Dropout(0.2))
model.add(Dense(1024, activation='relu'))
model.add(Dropout(0.2))
 
model.add(Dense(num_classes, activation='softmax'))
复制代码

```

模型搭建完毕后，我们开始训练神经网络。为了能缩短训练时间，我比较喜欢随机选择训练集让模型学习，这也是在 Keras 中使用 fit_generator 的原因。同样，损失函数会是交叉熵，因为我们的任务是多类型分类问题。

```
gen = ImageDataGenerator()
train_generator = gen.flow(x_train, y_train, batch_size=batch_size)
 
model.compile(loss='categorical_crossentropy'
    , optimizer=keras.optimizers.Adam()
    , metrics=['accuracy']
)
 
model.fit_generator(train_generator, steps_per_epoch=batch_size, epochs=epochs)
复制代码

```

执行完 fit 函数后，我们对模型进行评估:

```
train_score = model.evaluate(x_train, y_train, verbose=0)
print('Train loss:', train_score[0])
print('Train accuracy:', 100*train_score[1])
 
test_score = model.evaluate(x_test, y_test, verbose=0)
print('Test loss:', test_score[0])
print('Test accuracy:', 100*test_score[1])
复制代码

```

为了不出现过拟合，最终得到如下结果，但当我增加 epoch 次数时，出现了过拟合现象：

```
Test loss: 2.27945706329
Test accuracy: 57.4254667071
 
Train loss: 0.223031098232
Train accuracy: 92.0512731201
复制代码

```

最终搭建的面部表情识别模型准确率为 57%。

### 用自定义照片测试模型

我们现在试着让模型识别我们自己找来的照片中的面部表情，因为单凭测试集中的错误率说明不了什么。

```
img = image.load_img("/data/pablo.png", grayscale=True, target_size=(48, 48))
 
x = image.img_to_array(img)
x = np.expand_dims(x, axis = 0)
 
x /= 255
 
custom = model.predict(x)
emotion_analysis(custom[0])
 
x = np.array(x, 'float32')
x = x.reshape([48, 48]);
 
plt.gray()
plt.imshow(x)
plt.show()
复制代码

```

将面部表情存储为数值，并按照从 0 到 6 打上标签。Keras 会生成一个输出数组，包括这 7 个不同的表情分值。我们可以将每个表情的预测可视化为条形图。

```
def emotion_analysis(emotions):
    objects = ('angry', 'disgust', 'fear', 'happy', 'sad', 'surprise', 'neutral')
    y_pos = np.arange(len(objects))
 
    plt.bar(y_pos, emotions, align='center', alpha=0.5)
    plt.xticks(y_pos, objects)
    plt.ylabel('percentage')
    plt.title('emotion')
 
    plt.show()
复制代码

```

如果你看过 NetFlix 出品的《毒枭》系列，你一定很熟悉这张照片。下面这张照片是大毒枭巴勃罗 · 埃斯科瓦尔被捕入狱时照的照片。用我们刚才搭建的模型对这张照片识别后，显示巴勃罗当时的心情非常不错。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/25/163969b8c20f6586~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

接着我们用第二张照片，马龙 · 白兰度在《教父》中饰演的 “老头子” 考利昂看到儿子尸体时的画面。测试结果显示模型也能识别出考利昂痛苦悲伤的表情。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/25/163969be64bbe84e~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

我还想测试一下发怒的表情，提到发怒，休 · 杰克曼演的 “金刚狼” 可是个典型，就是他了。我选了《X 战警》中金刚狼暴怒的图片。测试结果显示非常成功。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/25/163969c587c8a69c~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

最后，让模型挑战一个高难度的照片：**蒙娜丽莎的微笑**。至今艺术家们仍没有搞明白达芬奇的画作《蒙娜丽莎》中人物当时的心情。用我们的模型识别后，显示当时蒙娜丽莎的心情非常平和自然。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/25/163969cc4af863e5~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

### 结语

在本文我们构建了一个卷积神经网络识别人类的面部表情。最终模型的准确率为 57%，这个结果还可以接受吧，因为当年 Kaggle 的那次比赛最高准确率才 34%。如果对检测到的图像中的人脸进行处理，而非处理整个图像，能进一步提高准确率。因此我在运行神经模型前，对图像中的人脸做了些裁切工作。我计划下一步训练 Inception 中的预构建模型完成这个任务，到时候再和大家分享。

点击[这里](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fserengil%2Ftensorflow-101%2Fblob%2Fmaster%2Fpython%2Ffacial-expression-recognition.py "https://github.com/serengil/tensorflow-101/blob/master/python/facial-expression-recognition.py")查看本项目代码。

欢迎关注我们，学习资源，AI 教程，论文解读，趣味项目，你想看的都在这里！


-----------------------------------------
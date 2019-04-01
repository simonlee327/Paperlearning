# 模型结构
![Lenet](http://static.zybuluo.com/feixian15/bo9cgzyuomzn73jiq6o9hsuu/ScreenShot2017-07-06at12.24.42PM.png)

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_1.png)

# 各层参数详解（计算过程）
## 数据集和模型任务解读：
## 涉及到的运算
* Convolution
![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_12.gif)
* Maxpooling

    Use maxpooling to subsampling 
    
![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_3.png)

  池化是非线性下采样的一种形式，主要作用是通过减少网络的参数来减小计算量，并且能够在一定程度上控制过拟合。通常在卷积层的后面会加上一个池化层。池化包括最大池化、平均池化等。其中最大池化是用不重叠的矩形框将输入层分成不同的区域，对于每个矩形框的数取最大值作为输出层，如上图所示。
* Full connection


##各层参数说明

输入图片：32*32

卷积核大小：5*5

卷积核种类：6

输出featuremap大小：28*28 （32-5+1）=28

神经元数量：28*28*6

可训练参数：（5*5+1) * 6（每个滤波器5*5=25个unit参数和一个bias参数，一共6个滤波器）

连接数：（5*5+1）*6*28*28=122304

详细说明：对输入图像进行第一次卷积运算（使用 6 个大小为 5*5 的卷积核），得到6个C1特征图（6个大小为28*28的 feature maps, 32-5+1=28）。我们再来看看需要多少个参数，卷积核的大小为5*5，总共就有6*（5*5+1）=156个参数，其中+1是表示一个核有一个bias。对于卷积层C1，C1内的每个像素都与输入图像中的5*5个像素和1个bias有连接，所以总共有156*28*28=122304个连接（connection）。有122304个连接，但是我们只需要学习156个参数，主要是通过权值共享实现的。

# 参数总数，模型效果



# 为什么使用这种结构



# 我的思考（疑惑，改进的地方）

# reference
[csdn blog1](https://cuijiahua.com/blog/2018/01/dl_3.html)

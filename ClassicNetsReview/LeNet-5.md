# Architecture 
![Lenet](http://static.zybuluo.com/feixian15/bo9cgzyuomzn73jiq6o9hsuu/ScreenShot2017-07-06at12.24.42PM.png)

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_1.png)

# 各层参数详解（计算过程）

* Input:

  an image of size[32,32],only has one channel
* C1

  Kernal：6 convolution kernals of size [5,5]So C1  

  output ：feature maps of size 32-2-2=28 ,[28,28,6]

  parameter： 6\*（5\*5+1）=156
    
  连接数：（5\*5+1）\*6\*28\*28=122304
  后期被 感受也取代，没有人计算连接数了，因为这个就是为，让产生的feature 能够对应到原始图像的更大区域
* S2 (Pooling but not maxpooling)

  calculate: Pooling layer,4个输入相加，乘以一个可训练参数，再加上一个可训练偏置。结果通过sigmoid激活

  kernal：[2,2] 

  output matric size(neuron number): [14,14,6] (14\*14\*6)

  output featuermap size :[14,14]

  parameter: (14\*14+1)\*6=1182
  
  后来就不用了，都是最大池化，我也不知道这个叫什么

* C3 (convolution layer)

  Input：[14,14,6]

  Kernal: [5,5,3/4]
  
  output: [10,10,16]
  ![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_5.png)

  C3的前6个feature map（对应上图第一个红框的6列）与S2层相连的3个feature map相连接（上图第一个红框），后面6个feature map与S2层相连的4个feature map相连接（上图第二个红框），后面3个feature map与S2层部分不相连的4个feature map相连接，最后一个与S2层的所有feature map相连。卷积核大小依然为5*5，所以总共有6*(3*5*5+1)+6*(4*5*5+1)+3*(4*5*5+1)+1*(6*5*5+1)=1516个参数。而图像大小为10*10，所以共有151600个连接。

  Like this

  The first colume

  ![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_6.png)

  为什么采用上述这样的组合了？论文中说有两个原因：1）减少参数，2）
这种不对称的组合连接的方式有利于提取多种组合特征。????

* S4
输入：10*10

采样区域：2*2

采样方式：4个输入相加，乘以一个可训练参数，再加上一个可训练偏置。结果通过sigmoid

采样种类：16

输出featureMap大小：5*5（10/2）

神经元数量：5*5*16=400

连接数：16*（2*2+1）*5*5=2000

S4中每个特征图的大小是C3中特征图大小的1/4

详细说明：S4是pooling层，窗口大小仍然是2*2，共计16个feature map，C3层的16个10x10的图分别进行以2x2为单位的池化得到16个5x5的特征图。有5x5x5x16=2000个连接。连接的方式与S2层类似。

* C5

输入：S4层的全部16个单元特征map（与s4全相连）

卷积核大小：5*5

卷积核种类：120

输出featureMap大小：1*1（5-5+1）

可训练参数/连接：120*（16*5*5+1）=48120

详细说明：C5层是一个卷积层。由于S4层的16个图的大小为5x5，与卷积核的大小相同，所以卷积后形成的图的大小为1x1。这里形成120个卷积结果。每个都与上一层的16个图相连。所以共有(5x5x16+1)x120 = 48120个参数，同样有48120个连接。C5层的网络结构如下：

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_7.png)

* F6

输入：c5 120维向量

计算方式：计算输入向量和权重向量之间的点积，再加上一个偏置，结果通过sigmoid函数输出。

可训练参数:84*(120+1)=10164

详细说明：6层是全连接层。F6层有84个节点，对应于一个7x12的比特图，-1表示白色，1表示黑色，这样每个符号的比特图的黑白色就对应于一个编码。该层的训练参数和连接数是(120 + 1)x84=10164。ASCII编码图如下：

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_8.png)

F6层的连接方式如下：

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_9.png)

* L7(output layer)

Output层也是全连接层，共有10个节点，分别代表数字0到9，且如果节点i的值为0，则网络识别的结果是数字i。采用的是径向基函数（RBF）的网络连接方式。假设x是上一层的输入，y是RBF的输出，则RBF输出的计算方式是：

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_10.png)

上式w_ij 的值由i的比特图编码确定，i从0到9，j取值从0到7*12-1。RBF输出的值越接近于0，则越接近于i，即越接近于i的ASCII编码图，表示当前网络输入的识别结果是字符i。该层有84x10=840个参数和连接。

![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_11.png)
## 数据集和模型任务解读：


## 涉及到的运算
* Convolution
![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_12.gif)
* Maxpooling

    Use maxpooling to subsampling 
    
![](https://cuijiahua.com/wp-content/uploads/2018/01/dl_3_3.png)

  池化是非线性下采样的一种形式，主要作用是通过减少网络的参数来减小计算量，并且能够在一定程度上控制过拟合。通常在卷积层的后面会加上一个池化层。池化包括最大池化、平均池化等。其中最大池化是用不重叠的矩形框将输入层分成不同的区域，对于每个矩形框的数取最大值作为输出层，如上图所示。
* Full connection



# 参数总数，模型效果



# 为什么使用这种结构



# 我的思考（疑惑，改进的地方）
主要有以下几个不同，和现代的方法
1 对于duochannel 的卷积，他是人工选择的对各级几个channel 应用卷积，有的是连着的，有的是不连着的，有连着两个的，还有连着三个的，我们是对所有的通道用卷积，他说这样可以1）减少参数，2） 这种不对称的组合连接的方式有利于提取多种组合特征，减少参数确实没问题，但是第二点是什么意思呢？就算全都使用，训练出来的那个参数会比较小，和没什么问题啊我觉得

2 池化，我们用的是最大池化，他用的池化方式需要训练参数，都能达到减少参数的作用，但是现代的方法更不需要训练参数

3 评估方法，最后对于输出的解释，是同样的内容，但是他解释的很复杂RBF函数是什么？我还需要仔细看一下

# 总结

  这是最早的关于卷积网络的一片文章，他里面用了很多传统的概念去解释，比如最后的layer，很多的现代概念还没有形成，比如loss function，这种，但是思想是一样的，很多细节的地方也和我们不一样，比如池化和多通道的卷积。他里面的思想还是值得研究的，更能帮助我们理解现代的方法的目的是什么，有什么更好的地方。
  
# reference

[csdn blog1](https://cuijiahua.com/blog/2018/01/dl_3.html)

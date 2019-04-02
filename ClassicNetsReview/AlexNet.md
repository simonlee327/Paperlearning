# Architect
![](https://img-blog.csdn.net/20180518202244353)
* input

    [224,224,3] [227,227,3]

* conv 1：

    kernal [11,11,3,96] stride[4,4]
    output:[55,55,96] （227-11）/4+1=56 ,[55,55,48]every GPU.

    and parameter about max-pooling:
    
    max pooling size [3,3] stride [2,2] 

    output: [27,27,96] (55-3)/2+1=27

    局部响应归一化层(LRN)：最后经过局部响应归一化处理，归一化运算的尺度为5*5；第一层卷积层结束后形成的图像层的规模为27*27*96.分别由96个卷积核对应生成，这96层数据氛围2组，每组48个像素层，每组在独立的GPU下运算

* conv 2：

    Input :[27,27,96=48\*2]

    kernal [5,5,48,256=128\*2] stride[1,1] padding: 2(same)

    output:[27,27,256=128\*2] (27-5+2\*2)/1+1=27 

    and parameter about max-pooling:
    
    max pooling size [3,3] stride [2,2] 

    output: [13,13,256=128\*2] (27-3)/2+1=13

    局部响应归一化层(LRN)：最后经过归一化处理，分别对应2组128个卷积核所运算形成。每组在一个GPU上进行运算。即共256个卷积核，共2个GPU进行运算。

* conv 3：

    Input :[13,13,256=128\*2]

    kernal [3,3,256,384=192\*2] stride[1,1] padding: 1

    output:[13,13,384=192\*2] (13-3+1\*2)/1+1=13 
    
* conv 4：

    Input :[13,13,384=192\*2]

    kernal [3,3,192,384=192\*2] stride[1,1] padding: 1

    output:[13,13,384=192\*2] (13-3+1\*2)/1+1=13 

* conv 5：

    Input :[13,13,384=192\*2]

    kernal: [3,3,192,256=128\*2] stride[1,1] padding: 1

    output:[13,13,256=128\*2] (13-3+1\*2)/1+1=13 

    Max Pooling:

    size [3,3] stride [2,2]

    output: [6,6,256=128\*2] (13-3)/2+1=6

* FC layer 6:

    Input : [6,6,256=128\*2]

    Kernal : [6,6,256,4096=2048\*2]

    Output : [4096=2048\*2]

* FC layer 7:

    Input : [4096=2048\*2]

    Kernal : [4096,4096=2048\*2]

    Output : [4096=2048\*2]

* Output Layer 8 :

    Input : [4096=2048\*2]

    Output : [1000]

# Parameters Number
第一个层计算错误
![](https://vimsky.com/wp-content/uploads/2017/11/AlexNet-2.jpg)

From : (https://vimsky.com/article/3664.html)

(https://blog.csdn.net/superyang198608/article/details/82876445)


# 相关知识点

* non-saturating neurons (https://www.zhihu.com/question/264163033)

  使用relu，而不用sigmoid 或者tanh ，规避vanishing, exploding of gradients 带来的gradient值过大过小，导致训练效率低下如上图所示，使用了RELU后，训练效率大幅提升（但论文中似乎没有提及规避vanishing, exploding of gradients的问题）
* 

# 创新之处：
 * Relu Nonlinearity

    这是最重要的，首先他加快了收敛速度，训练速度大大的加快了。
    (现在还在用)，其次，能够减少梯度消失的问题。？现在都是用Relu了。


    参考：
    
    1 https://blog.csdn.net/wangkun1340378/article/details/74517252


* DropOut
    随机使
    大大的减少了过拟合发生的情况，但是加倍了迭代的次数
    (现在还在用)
    
    参考：

    1 https://blog.csdn.net/program_developer/article/details/80737724
 * Overlapping Pooling

    相较于之前 Le-Net5 中采用的平均池化，AlexNet 首次采用了重叠的最大池化，避免了平均池化的模糊化效果？？，AlexNet中提出让步长比池化核的尺寸小，这样池化层的输出之间会有重叠和覆盖，提升了特征的丰富性

    论文中说，实验证明比起普通的（是说不重叠的？），起到一点点减少过拟合的作用

    为什么现在很少见到用重叠的？

    我觉得可能是，过拟合的作用太小了，但是比起平均池化更有用，

    参考：
    https://www.zhihu.com/question/36686900/answer/130890492
* LRN 
    不懂
    对局部神经元的活动创建竞争机制，使得其中响应比较大的值变得相对更大，并抑制其他反馈较小的神经元，增强了模型的泛化能力。
# 我不懂的部分

* 什么是稀疏性，为什么说人类的大脑具有稀疏
* 什么是饱和和不饱和函数，为什么Relu能起到加速的作用
* label-preserving transformation 是什么
* 为什么和反馈神经网络比，CNNS有更少的连接和参数
* LRN 的部分，后来有人说这个没什么用
* 各种池化的选择，区别，工作原理

    https://blog.csdn.net/sinat_29552923/article/details/72179553

* 数据增强的部分
* 各种空间的不同，距离度量的不同
* 还有其他我在文章中标注出来的，有时间再进行详细的学习研究

# 总结
    我认为AlexNet最主要的工作，是训练了一个比较深的卷积网络，虽然没有办法和现代的这些比较，但是在他这个时候是非常有意义的，之所以能够出现这样的网络，有几个点:
    1

    Relu 激活函数的使用，能够增加训练效率
    2

    ImageNet数据集的出现
    3
    GPU的使用
    4 
    DropOut技术的使用，能够显著的减少过拟和

    以上任何一个因素的缺少，都会导致，无法出现这么好的分类效果，比如，Relu缺少，速度就会很慢，慢五倍以上，没有数据及会过拟合，没有DropOut也会过拟合
    同时，这些技术对网络的影响很大，所以能够一直应用到现在，不是微小的影响

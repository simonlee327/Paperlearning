# PointNet

![](https://img-blog.csdn.net/20180517215110916)
## 网络架构

输入n*3的点集，每个点多次使用MLP，提取特征，然后再n点的相同位置的特征中找最大值，得到全局特征，对于分类，将这个全局特征经过MLP压缩成K个分数，对于分割，将这个全局特征copy n份复制到经过的MLP提取的每个点的64 c的后面得到了n\*1088的特征，1088通道然后经过MLP压缩成m类，对应m类的分数

其中T-Net是学习到的仿射变换的矩阵，将原始的数据做一个仿射变换，将这个模块嵌入到网络的第一层，相当于使网络能够学习到最有利于任务的变换，将这个模块嵌入到更高级的特征层中，作用类似。

## Insight:

1 输入无序性问题

从数据结构的角度来讲，点云数据知识一组无序的向量集合，若不考虑其他诸如颜色等因素，只考虑点的坐标，则点云数据只是一组n * 3的点集合。那么当对这n个点进行不同顺序的读入时，无论是图像分类抑或是部分分割，都必须输出相同的结果，在2D图像中，因为点的位置是固定的，所以不存在这样的问题，但是在点云数据中，点的输入组合中共有n！种，所以必须进行处理。

在论文中，处理的方式也比较简单，使用了最大池化，在对n个点进行卷积等操作后，生成n * 1024维的矩阵，1024维的1024个整体特征，对于每个维度，求其最大值，即解决了输入无序性的问题。此处可举例有max（a,b,c,d,e,f,g） =  max (a,d,b,e,c,f,g) = ...

在论文的4.3部分，作者就使用最大池化进行证明，当特征维数足够大时，最大池化可以对模拟论文中所述的任意对称函数f，同时在此基础上论述了模型的稳定性。对于任意数据集S，Cs表示critical set，Ns表示upper-bound set，即对于相同全局特征f(s)，所可能的最大点云集，此时有f(Ns) = f(S), 故而当添加的点属于Ns - S时，对于整体点云特征并无影响，故而满足稳重所述的robust to small pertubation of input points。此处其实与svm中的支持向量有点类似，以及凸包中，被包住的点。最终作者的结论是网络学习到了概括形状的关键点，即点云的骨架。？

2 点与点之间的相关性

（在分类问题中，此问题其实不是特别重要，在分类中，只要处理出全局特征，通过对全局特征进行mlp或者用svm分类等，即可得到最终答案，但是在part segmentation 以及 scene semantic parsing中，此问题就显得极其重要。

论文中进行处理时，通过将提取出的全局特征以及点特征结合，得到n * 1088（64 + 1024）的特征矩阵，然后在此矩阵的居处上，重新提取每个点的特征，得到包含局部及全局特征的点信息。？
## 问题
1 不捕捉局部结构，识别精细的模式和复杂的场景很弱

Pointnet的基本思想是对输入点云中的每一个点学习其对应的空间编码，之后再利用所有点的特征得到一个全局的点云特征。这里欠缺了对局部特征的提取及处理，比如说点云空间中临近点一般都具有相近的特征，同属于一个物体空间中的点的概率也很大，就好比二维图像中，同一个物体的像素值都相近一样。

2 密度问题

再者现实场景中的点云往往是疏密不同的，而Pointnet是基于均匀采样的点云进行训练的，导致了其在实际场景点云中的准确率下降

PointNet++中的评论：

However, by design PointNet does not capture local structures induced by the metric space points live in, limiting its ability to recognize fine-grained patterns and generalizability to complex scenes

3 Pooling 丢失信息

    Pooling丢失信息，本身卷积这种提取去局部信息的方式是不丢失信息的，但是PointNet用了Pooing，就会丢失信息？？
这就是Pooling的副作用
但是是他的维度还变多了，这些信息是不是隐藏在这些维度中了？

    这个观点在PointCNN中提出来的
# PointNet++

![](https://img-blog.csdnimg.cn/20181109173211143.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDY2NDA5NA==,size_16,color_FFFFFF,t_70)

## 网络架构

整个网络就是模仿的FCN，其中使用采样加分组的方式来多次运用PointNet ，相当于把pointnet当成卷积核用了，这是下采样，上采样使用KNN插值，然后对每个点和之前的特征层拼接上，用PointNet融合这些特征，多次重复这两个过程，得到每个点的K的分数

## Insight

1 使用采样加分组的方式来多次运用PointNet ，相当于把pointnet当成卷积核用了，解决了Point没有局部结构捕捉的问题

2 使用 不同大小的分组来抵抗密度不均匀的问题 （MSG，MRG）

![](https://img-blog.csdn.net/2018082217250689?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM3MDExODEy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3 相对于PointNet，精度提高，模型体积减小

## 问题
1 推理速度比PointNet慢4-7倍，比Vanilla 慢8-16倍

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/Image%203.png)

## Reference

https://blog.csdn.net/weixin_40664094/article/details/83902950

https://blog.csdn.net/weixin_40664094/article/details/83932046

# PointSIFT

## 网络架构

![](https://img-blog.csdn.net/2018101611361175?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdiaW5feHU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

主要作用是提出了一个模块（PointSIFT），能够捕捉一个点周围的局部信息，输入一些具有特征的点云，输出具有更强表达能力的特征，同时，特征维度不改变。能够提高精度。


## 解释

1 OE单元

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/5.png)

Stage1:

对于一个输入的点P0，它的维度是d，以他为中心，找一个半径r，空间中划分出8各区域，在每个区域中找到一个距离最近的点，总共找到八个点，(如果某一个区域没有点，就认为P0是这个方向的点)

Stage2：

对这些点进行卷积，按照XYZ的方向进行，最终八个特征生成一个特征
如上图所示，这八个立方体代表8个特征，首先用2*d维度的卷积核对X轴进行卷积4次，生成4个d维度的，然后对y轴，生成2个d维度的，最后对z轴，生成一个d维度的。

这里有要训练的参数，也就是三个卷积核的参数。

能够编码每个点的局部信息，也就是能够提取局部结构信息

2 PointFIFT 模块

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/9.png)

这是一个OE单元，方向编码模块，然后如上图，整个SIFT模块有多个OE组成，对n\*d 的输入，如果用一次，输出的每个点的感受野比较小，只有8个，但是用两次就有64个，用三次就更大了，为了能够学习到最有力的scale组合，堆叠多次OE单元，然后把他们的特征整合，最后输出一个d维的特征。

## Insight

### 1 实现了更高的mIoU，Accuracy

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/6.png)

### 2 OE单元，能够捕捉来自不同方向的信息，相比于Pointnet++，Grouping的时候选择的点更加合理

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/7.png)

如图，这种方向找点的方法，比PointNet++的ball qural 更加能够捕捉到合适的信息，图上就是这种情况

### 3 下采样的时候能够不丢失点，与PointNet++比较

https://github.com/simonlee327/Paperlearning/blob/master/Pictures/8.png

PointNet++下采样的时候，Grouping的时候有的点不属于任何一个质心，但是PointSIFT就不存在这个问题，他对每一个点都找了8个最近的点

论文中说PointNet++这样Grouping，少了20%的点，影响性能，而PointFIFT中的每个点都对最终的预测有贡献

### 4 Scale -aware

作者说他做了一个实验，产生大小不同的基本形状，然后，观察到，底层的OE单元对小尺度的有反应，高层的对大尺度的有反应，证明了Scale-aware,堆叠多个OE有用


## 问题

1 效率低，运行慢

2 复现困难？？

3 我认为PointSIFT和SA层中的结构重复了，PointSIFT本省已经能够提取点的局部特征，那就可以代替PointNet，只需要做一个减少Point的数量的操作，下采样的过程就完成了。

更详细的说明，就是PointNet能够提取点云的特征，这是在论文中已经证明过了的，可以逼近任意一个连续函数，但是PointSIFT能不能这样做呢？PointSIFT论文中说它也可以提取特征

另外，因为没有看代码，还有架构上的细节没弄清楚，比如有一段说将FP用在下采样中，FP是上采样的模块怎么用在上采样中，还有他证明PointSIFT有效的对比试验，他没有说清楚表格中的说明是什么意思
![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/10.png)
![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/11.png)
## Reference

https://www.sohu.com/a/241154838_129720

https://blog.csdn.net/hongbin_xu/article/details/83071273

https://github.com/MVIG-SJTU/pointSIFT

https://blog.csdn.net/shengyan5515/article/details/82965659


# PointCNN

提出了用于点云数据的卷积运算方法，使用了这种类似于二维的卷积算子，然后整个网络类似于Unet，达到了高于PointNet++的效果。

## 文章内容

在论文里面写的很详细，我就大概记录一下

首先，他用直接在点云数据上面用卷积，发现了一些问题，比如排序不变性，还有无法记录点云的局部形状的信息

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/12.png)

比如2和3就是这样，本来应该是不同的，但是放到矩阵中就相同了，而3和4也是，本来应该是相同的结构，但是因为在矩阵中的顺序是不固定的，就不同了，这就是上面提到的两个问题

然后作者希望学习到一个变换矩阵，能够使输入的特征具有排序不变性，同时又能够反应局部的结构信息，就像二维的卷积那样。然后搭建一个分层的网络结构如下图：

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/14.png)

具体是这样做的：

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/16.png)
## 卷积运算
输入就是找到代表点的邻域的点，然后对这些点进行处理，最终输出一个特征

首先搜索用的是随机搜索，但是对于分割任务用的是FPS，找出代表点，然后找出他的K个近邻

1 坐标变化

2 对坐标进行升维，进行更抽象的表示，来和特征向量进行匹配

3 把升维之后的坐标矩阵和特征矩阵拼接

4 将坐标变化之后的坐标矩阵MLP,学到X变化矩阵

5 对拼接之后的特征矩阵进行X变换

6 常规的卷积，也就是加权和运算，最终生成一个特征

## 整体架构

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/17.png)

普通的分层卷积，b里面使用了扩张卷积的方法，c里面的反卷积需要看一下

Unet

## 实验结果

1 分类：

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/18.png)

2 分割

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/19.png)

3 消融实验

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/21.png)

第二个是直接使用卷积，没有X变换的，3和4是为了保持参数量一致，增加了网络的宽度或者是深度，最终证明使用了X变换的效果好，而不是因为参数两增加了

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/20.png)

这个实验很有意思，他证明了能够对不同的排序输出相似的结果，排序不变性

图中的每个点都代表和卷积进行加权和的一个矩阵，也就是算法里面坐标提升维度之后和它对应的特征拼接生成的矩阵，但是对于同样的结构，有不同的排序，也就是矩阵的行有可能是交换了的，所以有不同数据点，途中的不同颜色他们的代表点不同，相同颜色哪些点都属于同一个代表点，也就是同一个Patch，二维图像中的，

在理想情况下，对于同一个Patch，不同的排序所产生的特征矩阵应该是相同的，也就是对途中的相同的点，

第三张图代表经过了变换的特征，他们高度相似，距离很近,证明了排序不变性。

## 总结

1 这个和PointNet相比，提出了不同的解决排序不变性的方法，也就是学习一个矩阵，作者说Pooling的方式会丢掉一些信息，这样做更好，同时参数也会减少，有点像T-Net

2 更高的精度

3 更快的速度：（对比PointNet和PointNet++）

![](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/22.png)

## 问题

1 作者也说，对于排序不变性没有解决的很好

2 搜索的时候也可能漏掉点？

3 
## reference

https://www.cnblogs.com/elliottzheng/p/9100254.html

https://blog.csdn.net/qq_15602569/article/details/79560614


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

![](https://img-blog.csdn.net/20181016113513202?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmdiaW5feHU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

对于一个输入的点P0，它的维度是d，以他为中心，找一个半径r，空间中划分出8各区域，在每个区域中找到一个距离最近的点，总共找到八个点，然后对
## Insight

## 问题

## Reference

https://blog.csdn.net/hongbin_xu/article/details/83071273

https://github.com/MVIG-SJTU/pointSIFT

https://blog.csdn.net/shengyan5515/article/details/82965659
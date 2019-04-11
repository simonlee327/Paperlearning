#   Reference
  (https://blog.csdn.net/shenxiaolu1984/article/details/51066975)
  (https://blog.csdn.net/Tomxiaodai/article/details/81749845)
  (https://baike.baidu.com/item/R-CNN/23197553?fr=aladdin)
# Architect

![p](https://github.com/simonlee327/Paperlearning/blob/master/Pictures/Image%201.png)

1. 输入一张图片，

1. Ex2000+ Region ProposalsS

    Selective Search： traditonal aproach ? 

    No parameter to learn ?

    Imporvement

1. 使用CNN 计算这些 2000+ Region Proposals的Features 

1. 将这些Features 送入一些SVM ，得到每个类概率，这样，每个Proposal都会有各个类的概率

1. 使用 非最大值抑制去掉很多Proposal，这样每张图片就只剩下几个Proposal。

1. 接下来有两种方法一种是直接将剩下的Proposal 作为结果，另一种是将剩下的Proposal做rgression,矫正根据Slective Search 提出的框的位置 。

    论文里面 有一个图说明了，这种方法，用CNN 可以精准的判断类别，但是框的位置不太好，因此可以使用Regression 矫正框的位置。毕竟，这些框都是没有经过学习算法的直接提出来的。


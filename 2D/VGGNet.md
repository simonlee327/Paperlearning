# Reference
(https://blog.csdn.net/qq_31531635/article/details/71170861)

VGG 在深度学习领域中非常有名，很多人 fine-tune 的时候都是下载 VGG 的预训练过的权重模型，然后在次基础上进行迁移学习。VGG 是 ImageNet 2014 年目标定位竞赛的第一名，图像分类竞赛的第二名，需要注意的是，图像分类竞赛的第一名是大名鼎鼎的 GoogLeNet，那么为什么人们更愿意使用第二名的 VGG 呢？

因为 VGG 够简单

# 网络结构
![](https://img-blog.csdnimg.cn/20181107170256266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JyaWJsdWU=,size_16,color_FFFFFF,t_70)



他主要是讨论了网络深度对于模型性能的影响
上面那张图就是他们试验过的6中网络结构

# 实验结论
![](https://img-blog.csdnimg.cn/20181107175255860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JyaWJsdWU=,size_16,color_FFFFFF,t_70)
1. LRN没用（A与A-LRN）比较
1. 网络越深效果越好 （A与B）(D与E饱和了，没有明显变化)
1. 1*1卷积能够增加非线性，增加网络辨别效果（B-C）
1. 1*1卷积不如3\*3卷积效果好，它用最小的代价实现了非线性的增加，但是不如正常的卷积(C D)
1. 用多个小卷积代替一个大卷积更好，原因如下 

    a:感受野相同

    b：参数个数少

    c: 非线性增加(使用了多个激活函数)
# 其他重点
1. 全卷积网络可以用于推理，不同大小的输入，但是他们最后的输出也会大小不同，这时用平均Pooling的方式可以转换成概率分布。
1. 处理图像的方式也会影响模型的性能，（数据增强），如Table3 

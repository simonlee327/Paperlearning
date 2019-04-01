<!-- vscode-markdown-toc -->
* 1. [2.1网络结构](#)
* 2. [2.2网络特点](#-1)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->---
title：
category
---

# 1 LeNet

![Lenet](http://static.zybuluo.com/feixian15/bo9cgzyuomzn73jiq6o9hsuu/ScreenShot2017-07-06at12.24.42PM.png)

上图为LeNet结构图，是一个6层网络结构：三个卷积层，两个下采样层和一个全连接层（图中C代表卷积层，S代表下采样层，F代表全连接层）。其中，C5层也可以看成是一个全连接层，因为C5层的卷积核大小和输入图像的大小一致，都是5*5（可参考LeNet详细介绍）

网络特点 
每个卷积层包括三部分：卷积、池化和非线性激活函数

（sigmoid激活函数

使用卷积提取空间特征

降采样层采用平均池化

# 2 Alex Net

##  1. <a name=''></a>2.1网络结构

![Alex Net](http://static.zybuluo.com/feixian15/frh61ulks4075fsuplbr3k77/1689929-063fb60285b6ed42.png)

##  2. <a name='-1'></a>2.2网络特点

使用两块GPU并行加速训练，大大降低了训练时间

成功使用ReLu作为激活函数，解决了网络较深时的梯度弥散问题

使用数据增强、dropout和LRN层来防止网络过拟合，增强模型的泛化能力

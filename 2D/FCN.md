## Reference
    (https://blog.csdn.net/qq_36269513/article/details/80420363)

    (https://blog.csdn.net/shenxiaolu1984/article/details/51348149#fn:1)
## 核心思想

1. 把Alex net，VGGnet，GoogLeNet改成全卷积，可以实现任意尺度的输入

1. 结合深层粗糙的语义信息和浅层精细的外表信息来分割

    Combine semantic information from a deep ,coarse layer with appearance information from a shallow, fine layer to produce accurance and detailed segmentations

    (Skip 的方式：反卷积之后再和下采样时候相应尺寸的特征图相加)
    如下图：

    ![](https://raw.githubusercontent.com/simonlee327/Paperlearning/master/Pictures/Image%202.png?token=ATcesqoG7s1HZK_r41ZEmMRlny3mCFCJks5crCiEwA%3D%3D)
    
    上图中，32x即为扩大32倍。Pool5扩大32倍就可以得到原来图像大小了。
    
    Pool5扩大2倍与Pool4融合得到，再扩大16倍也可以得到原来图像大小了。
    
    Pool5扩大2倍与Pool4融合,再扩大2倍与Pool3融合再扩大8倍也可以得到原来图像大小了

    这就得到了三个（FCN-32s，FCN-16s,FCN-8S），反卷积的步长S

1. 
## Reference
    (https://blog.csdn.net/qq_36269513/article/details/80420363)
## 核心思想

1. 把Alex net，VGGnet，GoogLeNet改成全卷积，可以实现任意尺度的输入

1. 结合深层粗糙的语义信息和浅层精细的外表信息来分割

    Combine semantic information from a deep ,coarse layer with appearance information from a shallow, fine layer to produce accurance and detailed segmentations

    (Skip 的方式：反卷积之后再和下采样时候相应尺寸的特征图相加)
    如下图：

    

1. 
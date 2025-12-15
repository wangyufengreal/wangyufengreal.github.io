---
title: OpenCV 中的形态学变换
date: 2025-12-12 08:00:00 +0800
categories: [技术]
tags: [机器视觉]
---

## 背景

在工业场景下的二值化图片的预处理流程中，往往需要涉及到 `开运算` 与 `闭运算`，知道如何调用 OpenCV API 以及说出它们发挥的作用很容易，但是要了解为什么这样，底层的原理是怎样的，就需要对 OpenCV 中的形态学变换有一个基本的认知，本篇文章就是在做这样的事情。

## Erosion

侵蚀，官方文档使用 `soil erosion` 隐喻，非常恰当。因为对于水土流失，不仅仅是形态较小而整体不太稳固的沙土结构会被冲走，对于很稳固的山地结构，洪水袭来，它的边缘也会被带走一层土壤，而这，正是 Erosion 操作所做的事情。这里需要引入 `kernel` 的概念，以矩形 kernel 为例（事实上这和卷积核是类似的），当指定大小（如 5 * 5）的 kernel 遍历每一个像素点（具体这个被决定 `生死` 的像素点在哪里由 `anchor` 指定，正常使用默认也就是 kernel 的中心点），当 kernel 范围以内的区域存在 `0`, 也就是相对于 `foreground` 的 `background`，那么这个 anchor 决定的像素点应该也变为 0，这也就是，只有 kernel 范围内所有的像素值都为 `1`(假设是 foreground 的像素值) ,那么那一个被决定的像素点才是 1。

通过上述的定义，我们能够知道。只要选择了合适的 kernel，Erosion 的作用实际上是去除噪点，也就是面积小于 kernel 的噪点会被抹去（形态上无法盖住 kernel 的 foreground 同样是这个下场）。另外，就算形态远远大于 kernel 范围的 foreground 也会被削去一层边，原因是在 kernel 经过这些像素的时候，kernel 会检测到 foreground 边缘以外的 background。这正是 `Erosion` 去噪的 `"副作用"`，当然这里的副作用是打引号的，本身这层边很有可能也会包含一些毛刺，而去除一部分信息的同时，这些毛刺的噪声也消失了。因此整体来看，Erosion 就是一个整体提升二值化图片 SNR 的一个操作，但是我们不得不在意这个过程中损失的有效信息。这也是后续我们要介绍的 `Opening` 存在的意义。

## Dilation

膨胀，恰恰和 `Erosion` 相反，简单说就是在 `kernel` 以内，凡是有一个属于 `foreground` 的像素点，那么就认为这个目标像素点是 foreground，因此这个最终的结果就是，噪点（如果有的话）变成了大球，大的 foreground 实体变胖了一大圈（size 由 kernel 的 size 决定），乃至于明明一开始是分开的多个 foreground 实体，连接到了一起！这就是 Dilation。

`Dilation` 的问题和 `Erosion` 是类似的，这种情况下，我们关注的是不小心断开的多个主体或者内部的空白。执行了膨胀操作以后，有效信息确实提升了，但同时全局的膨胀带来无效信息的增长，SNR 变化方向也很难说。

## Opening

第一个魔法来了，`Opening` 就是先执行 `Erosion`，然后接着执行 `Dilation`。效果就是，去除噪点以后，补回原有的大的主体的被削去的外边缘，完美解决了我们刚刚在 Erosion 部分提到的有效信息被擦去的问题，我们实现了完美的 SNR 提升！（如果我们关注的是留下来的主体的话）

## Closing

第二个魔法，闭运算，`Closing` 就是先执行 `Dilation`，然后接着执行 `Erosion`。效果是，使得断开的区域连接或者内部的空缺填补以后，把多余的膨胀信息消除了，由于原始断开或空缺的区域只是一种瑕疵，因此在 `Erosion` 后得到了保留。这个操作同样实现了 SNR 的有效提升。

## cv.erode(...), cv.dilate(...), cv.morphologyEx(...)

API 的划分很有趣，也正说明了 OpenCV 简单形态学操作的本质，一个负责 `Erosion`，一个负责 `Dilation`，其它的额外操作都是 Erosion 和 Dilation 以及原始图片的组合，在 `cv.morphologyEx(...)` 中使用 `op` 参数区分。

## 衍生的操作

这些操作针对于不同的场景，同样使用得很多，也很有用。

`Gradient` 就是 `Dilation` 和 `Erosion` 的差，体现出来就是大的实体的轮廓和对噪点的扩宽。

`Top Hat` 就是原图和 `Opening` 的差，也就是得到了噪点本身（看似是噪点，实际可能是更关键的信息，比如星空中的星星，对于天文研究来说很有用）。简单说就是，如果我们关注特别小的 foreground，用 `Top Hat` 没错。

`Black Hat` 就是原图和 `Closing` 的差，也就是被填充或连通的区域本身。和 `Top Hat` 相反，这是如果我们关注小的 Background，那就使用使用 `Black Hat`。（为什么会有人想要关注 background？事实上， foreground 和 background 是相对的，从光学来看，255 是 foreground， 0 是 background，但是对于某些事物如某些国家的车牌而言，它的背景是白色的，数字是黑色的，反而这里我们可以利用 `Black Hat` 来对这些数字进行提取，然后进行进一步的处理。）

## kernel

又叫做 `structure element`, 除了矩形，还有圆形，十字形等等。当然，本质上它们都是矩形的矩阵，我们说的形状是它们内部 `0/1` 的分布的形态。

对于文字，表格，条形码等人造物体，通常使用矩形，因为人造物体通常有直角和直线，比如多变的文字实际占用着一个田字框形的几何结构，使用矩形有利于后一步的处理（文字可以膨胀为一个矩形）。

对于细胞，颗粒等自然物体，通常使用椭圆，这是因为自然界本身很少有完美的直角，更多的都是包含一个弧度，使用椭圆的 kernel 有助于保留这些物体的形态特征（比如侵蚀细胞的时候，如果是矩形，那么细胞可能会被侵蚀为一个类矩形的形态，而使用原型的 kernel，则能够保留原始的圆形的特性）。

十字线则适用于如需要去除噪点，但是保留网格线的场景。

当然，这个 kernel 的形态完全可以自定义（毕竟本质上是 numpy.uint8 类型的 0/1 矩阵），要由具体的场景而定。

## 对 anchor 参数的理解

实际就是在 kernel 中，我们需要决定它到底是存在还是消除的那个像素点的位置。常常取中心点（默认）。我们这里主要讨论，如果这个点在其它位置会发生生么。

比如在左上角，那么在侵蚀中，foreground 的左上角会得到更多的保留，而右下角会被更多的侵蚀。原因是，实体左上角的像素点，它在计算时 kernel 因为整体偏右下，而得到了更多的 foreground 的点，因此只需要稍微靠里一点，就能够使 kernel 中充满 foreground 的像素，使得像素得到保留。而对于右小角的点，不巧，不巧它们就得到了更多的 background 的点，因此被削去更多。这种影响在膨胀上相反，左上角得到更多膨胀，右下角膨胀较少。这是可以通过图像直观理解的情况。

在右下角和上述左上角的情况相反，推理方法完全一致，在此不再赘述。

## borderType 和 borderValue

实际就是当我们对于图片边界的像素点，很有可能 kernel 是位于图片外部的，那么该如何计算是一个很重大的问题。borderType 给出了几种方案，首先是重复，也就是把图片边界值简单重复。其次是镜像。用的比较多的是指定值（使用 borderValue 指定），具体取 foreground 的值，还是 background 的值。这个和我们要解决的问题有关。这本质上也就是我们到底关注 foreground 还是 background 的问题。

## 后记

这就是对 OpenCV 形态学的基本认识，上述内容是基于二值化图片考虑的，但是形态学操作同样可以在灰度图等不同类型的图像进行操作，虽然算法的复杂度有差异，但是本质上是一样。


## 同面积不同形状 kernel 在形态学操作中的速度差异和原因

实验：测试 30*30 和 1*900 的 kernel 在 4096*4096 大小的图上的闭运算效率。

```python
import cv2
import numpy as np

img = np.zeros(shape=(4096, 4096), dtype=np.uint8)

kernel_1 = cv2.getStructuringElement(cv2.MORPH_RECT, (30, 30))
kernel_2 = cv2.getStructuringElement(cv2.MORPH_RECT, (1, 900))

%timeit cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel_1)
%timeit cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel_2)

# kernel_1: 21.6 ms ± 470 μs per loop (mean ± std. dev. of 7 runs, 10 loops each)
# kernel_2: 424 ms ± 12.7 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

实测后者的速度是前者的 19.6 倍，为什么同一面积的 kernel，计算的速度却天差地别呢？首先，opencv 的形态学操作对实心矩阵有优化，这里的差距是决定性的。在 opencv/modules/imgproc/src/morph.dispatch.cpp 中可以看到：

```c++
Ptr<FilterEngine> createMorphologyFilter(
        int op, int type, InputArray _kernel,
        Point anchor, int _rowBorderType, int _columnBorderType,
        const Scalar& _borderValue)
{
    // 其它代码...

    if( countNonZero(kernel) == kernel.rows*kernel.cols )
    {
        // rectangular structuring element
        rowFilter = getMorphologyRowFilter(op, type, kernel.cols, anchor.x);
        columnFilter = getMorphologyColumnFilter(op, type, kernel.rows, anchor.y);
    }
    else
        filter2D = getMorphologyFilter(op, type, kernel, anchor);
    
    // 其它代码...
}
```

在上面的代码中，可以看到，当 kernel 是实心矩阵时（`countNonZero(kernel) == kernel.rows*kernel.cols`），会拆分为一种行矩阵和列矩阵的形式（能够进行这种操作的情况被称作可分离滤波 Separable Filter），而当对所有像素点的行矩阵计算最大值（膨胀操作即是找最大值的操作）以后，下一次再对所有矩阵求列矩阵最大值操作的时候，实际就得到了 kernel 中的最大值，这是一个非常有趣的 trick，导致实际的单像素计算量由 W * H 变为了 W + H。

那么对于 `（30*30 vs 1*900）`，其实两者都进入了是实心矩阵的分支，但是无疑效果差异迥异。就是 `(900+1) / (30+30) ≈ 15.0` 倍的性能差异。这个差异无疑是决定性的。当然，这毕竟限制了 kernel 的形态，因此也并非所有的应用场景都能够享受到这个巨量的性能提升。另外需要注意的是，我们实测的 19.6 倍依然大于理论的提升 15.0 倍，这是为什么呢？这里需要考虑 numpy 矩阵内存管理的模式，而这也正是我们在条形码增强场景下的核心可提升点。

对于二维的 numpy 矩阵，一行的数据存储在相连接的内存空间，因此在计算行矩阵的最大值运算时，实际上有一个寻址的优势（就是空间局部性 Spatial Locality 和缓存行填充 Cache Line Filling）。进而这种内存空间上的优势提升了性能。而此时，我们只需要利用这种行数据相连的特性，将条形码图片进行转置，然后再使用行 900*1 的 kernel 进行闭运算，就能够得到更快的运算速度。下面是基准测试：

```python
import cv2
import numpy as np

img = np.zeros((4096, 4096), dtype=np.uint8)
cv2.randn(img, 0, 255)

# 方案 A：直接跑垂直大核 (Cache Killer)
kernel_v = cv2.getStructuringElement(cv2.MORPH_RECT, (1, 900))
%timeit cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel_v)

# 方案 B：旋转 -> 水平跑 -> 旋转 (Cache Friendly)
def cache_friendly_by_transpose(img):
    kernel_h = cv2.getStructuringElement(cv2.MORPH_RECT, (900, 1))
    # 1. Transpose (利用内存拷贝优化)
    img_t = cv2.transpose(img)
    # 2. Run Horizontal (Cache 命中率极高)
    res_t = cv2.morphologyEx(img_t, cv2.MORPH_CLOSE, kernel_h)
    # 3. Transpose Back
    res_b = cv2.transpose(res_t)

    return res_b

%timeit cache_friendly_by_transpose(img)

# 704 ms ± 169 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
# 220 ms ± 7.05 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

可以看到，性能上的提升的幅度是非常显著的，因此在实际的生产环境下我们最终选择了利用 transpose 的方式带来这里的性能上的提升。


## 需要改进的内容

结合项目编写

加入数学理论定义

加入性能分析

展示corner case（不做形态学ocr识别错误，做了之后识别是正确的）

## 待研究的问题

问题 1：请用数学公式定义膨胀 (Dilation) 和 腐蚀 (Erosion)。它们和闵可夫斯基和 (Minkowski Sum) / 闵可夫斯基差 (Minkowski Difference) 有什么关系？
问题 2：在二值图像中，腐蚀操作等价于对图像取反后的什么操作？（提示：对偶性 Duality）。这在工程实现上有什么意义？
问题 4：OpenCV 底层是如何优化大尺寸 Kernel 的？
问题 5：你的水表读数（特别是老式字轮）经常出现字符断裂（导致 OCR 识别成错误的数字）或者光照导致的噪点粘连。

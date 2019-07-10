#  《基于OPENPAI平台的云智能POSSTER拍照系统》 ——实训结题报告

## 1. 团队成员

组长：游增

组员：邱润韬、林会东、王永乐、陈梓轩

## 2. 第一次尝试

## 3. 研读论文

将老师给我们的参考论文分工研读，并做分析报告，我们主要对一下论文做了深入阅读和分析：

（论文标题和Abstract截图）

**林会东**：

1. Deep High Dynamic Range Imaging with Large Foreground Motions：

   <http://openaccess.thecvf.com/content_ECCV_2018/papers/Shangzhe_Wu_Deep_High_Dynamic_ECCV_2018_paper.pdf>（论文Abstract 截图）

林会东研读的这篇论文，是可以将多张不同曝光的照片做融合生成HDR图片，即使在这几张照片人物姿势相差很大，也不会出现鬼影现象，（论文中的延时图片）具体是采集多张低、中、高三张不同曝光的照片，选取其中一张作为前景参考（论文中是选取中等曝光的图片作为参考，将他们经过由encoder、decoder组成的U net模型（论文模型结构），最后生成，前景信息于选取图片一致，曝光适度的HDR图片。这是论文中的效果图（）

可是当我们应用到我们的图片上时发现，虽然总体光照好了很多，但是效果仍然不好（我们的效果图）：

1. ppt泛白，明显过曝
2. 原图中本来存在但不明显的噪点更加突出的显现
3. 过暗处的区域信息仍然是丢失的

**邱润韬**：

1. HDR Deghosting: How to deal with Saturation?：

   <https://www.cv-foundation.org/openaccess/content_cvpr_2013/papers/Hu_HDR_Deghosting_How_2013_CVPR_paper.pdf>

**王永乐**：

1. DeepFuse: A Deep Unsupervised Approach for Exposure Fusion with Extreme Exposure Image Pairs

   <http://val.serc.iisc.ernet.in/DeepFuseICCV17/files/DF_iccv17.pdf>

**游增**：

1. Deep Reverse Tone Mapping

   <http://www.npal.cs.tsukuba.ac.jp/~endo/projects/DrTMO/paper/DrTMO_SIGGRAPHAsia_light.pdf>

   <https://blog.csdn.net/vn9PLgZvnPs1522s82g/article/details/79694900?tdsourcetag=s_pcqq_aiomsg>

我研读的这篇论文，是针对单张图片的，他的网络结构不是用来合成HDR的，而是将单张图片映射成多张不同曝光的图片，网络是为了让图片映射的更加自然。我看了一下他的代码，他最后融合的时候是直接用的OpenCV的MergeMertens做融合，对我们的场景不适用，不同曝光根本不需要用到网络，我们可以直接通过调整参数获得，肯定更自然，它应该主要是针对网络上的图片进行优化

**陈梓轩**：

1. Learning a Deep Single Image Contrast Enhancer from Multi-Exposure Images

   <https://ieeexplore.ieee.org/document/8259342>

## 3. 第二次尝试

在应用我们的照片到别人训练好的网络上发现总体光照改善，局部细节糟糕后，在老师和师兄的建议下设想能不能自己构造数据集，先从单一场景的小数据集开始训练我们自己的网络，在拍摄了几十组样品照片后我们发现一个致命的问题，就是即使要求人不动，可是最后得到的照片还是会有些微的差别，这对数据集影响很大，而且即使是我们想要充当Ground truth的图片也还是不好，训出来的效果自然不可能更好。

（给几张照片）

## 4. 从比较简单的融合开始

根据老师的建议，我们准备从简单的融合下手，而我们想试试最后用到的OpenCV的融合算法，于是拍摄了几组不同曝光的照片，发现效果并不好。噪点严重，ppt有些过曝，且容易产生鬼影。因为不可能要求演讲人不动。

（效果图）

然后我们想到之前应用到的林会东看到那片论文的网络出来的效果除了ppt区域有些过曝以外，其他部分的光照还算可以，我们如果可以针对ppt区域，将融合好的图片和看的清ppt区域的中等曝光区域做融合就好了。

而我们发现DeepLab做的语义分割的效果（给出效果图）能够准确的将PPT区域标识出来，我们想到，将标识为TV类的点的坐标找出来，在求一个凸包，用这个凸包作为蒙版，（给出蒙版图）将融合好的图片和中等曝光的图片做第二步融合

在我将算法写好后测试效果，发现，ppt区域是能看的很清楚了，而且整体光照也不错，就是ppt区域有些突兀，与周围光照完全不一致（效果图）

我们对蒙版做了高斯模糊，将蒙版边缘做了模糊化，起到过度的效果，然后用这个作为蒙版进行融合，发现不会显得突兀了，效果还算不错（效果图）

但是这个网络用时比较久，然后噪点有些严重，加上去噪，在电脑上用cpu跑的话，需要近半分钟。

我们继而发现，高曝光的那张照片跟融合好的图片相比，除了ppt信息完全丢失，其他部分曝光正常，即使是过暗的地方，仍然能够看清。所以我们想能不能直接将高曝光的图片，和中等曝光的图片以那个蒙版做融合，语义分割网络只需要几百毫秒就可以完成，前后融合也不过几秒钟。

完成后发现，效果确实很好，没有明显噪点，暗处信息没有丢失，人像和ppt都能看清效果图（）



## 5. PPT矫正区域

这个算法也是由我完成，这个算法设计的比较简单，主要是步骤是先用边缘检测将轮廓勾勒出来，在检测所有外边缘轮廓，接着计算面积，如果其面积最大且构成四边形，我们则认定是ppt，并对他求透视矩阵，在对ppt部分做逆变换。



## 5. 移植Android平台

紧接着我们便开始分工：主界面、相册界面、预览界面由王永乐、陈梓轩完成，DeepLab模型的移植由林会东完成，而融合和ppt矫正算法我已经设计好了，所以只需要重构成Java就行，所以融合的算法重构由我完成，而ppt矫正部分有我和邱润韬一起完成，最后模块的整合由我完成
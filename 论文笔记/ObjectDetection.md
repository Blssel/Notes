[ObjectOverview](https://github.com/hoya012/deep_learning_object_detection)

# 1. RCNN
## Step1:区域建议（Region Proposal）
论文中说有很多region proposal的方法，本文无法从原理上证明哪些更好，为了方便和其他方法比较，就直接选择selective search(ss)。ss流程本文并没有再展开介绍。
 
此步骤不需要训练。

## Step2:特征抽取（Feature extraction）
有了一堆**类别无关**的region proposal box，下一步需要对这些框框中的内容进行特征提取。本文使用AlexNet进行特征的抽取：将图片resize到227*227大小，送入AlexNet（5个卷积层和2个全连接层），得到一个4096维的特征向量。

**注:**resize的时候，

**此步骤需要对backbone网络进行预训练**。
1. **先在ILSVRC2012上训练一个image-level的分类网络（官方是用的CNN的caffe开源代码）**。这一步比较简单，没什么可说的。据论文解释称，他们在ILSVRC2012上的训练效果和Krizhevsky等的效果相仿，在验证集上的top-1精度还要高2.2个百分点，这主要是因为他们简化了训练过程。
2. **再对预训练网络在检测数据集上做进一步的finetune**。在image-level的图片上训练的分类网络backbone直接拿来用在检测任务的特征提取上效果并不好，因为用在检测任务上网络接收的输入是一个warped（或者叫resized）ROI区域，虽然尺寸上没有差别，但是ROI中的图像区域或多或少都存在一些形变，数据分布发生了改变，因此需要进一步微调参数。**具体做法是：（以VOC为例）**1) 将原分类网络的分类层由1000类输出调整为21类（20+1，VOC有20类，另外一类是background类）；2) 挑选合适的ROI作为训练样本：结合groundtruth，将selective search的得到的所有region proposals进行正负样本的分类。对每一个region的bounding box，寻找与它重叠度最大的groundtruth，如果它与该groundtruth bounding box之间的IOU大于阈值0.5，则赋予此groundtruth的类别标签，否则赋予类别标签为background。**训练时**base lr设为0.001，每个mini batch由32个positive sample（从所有类别中选32个）和96个背景样本组成，共计128个样本，这主要是因为kbackground的样本总量更多。

## Step3:检测框的分类与回归
RCNN的想法是，将上一步提取的特征送入SVM中，训练若干个SVM分类器（多分类器），就达到了对每个bounding box分类的目的。想法既然没问题，那么下一步就是要考虑如何训练SVM了。

**如何训练SVM分类器，最重要的是如何构建训练样本集。**这里，作者采用了与finetune阶段稍微不同的策略，即仅仅使用groundtruth作为positive sample，negtive sample则是那些与groundtruth box IOU低于0.3的样本，相当于有一部分“中间样本”是不被使用的。至于为什么这么做，将在后续讨论。

## 讨论
- Q1:finetune和SVM训练阶段**对positive和negtive样本的定义为什么不一样**?

  A1:首先简单总结一下二者的区别：finetune阶段选择那些与groundtruth之间IOU足够大（大于0.5）的proposal作为positive样本（如果存在多个则选择最大的那个），剩余的一律作为negtive样本（background）；SVM训练阶段，选择groundtruth区域本身作为positive样本，与所有groundtruth box之间IOU低于0.3的作为negtive样本，其他介于两者之间的样本全部作废。**为什么这么做?**其实这是一个实验得出的结论，并没有什么理论依据。据作者所述，他们一开始是使用的在ImageNet上预训练的模型作为特征抽取器，并在此基础上训练SVM，尝试了多种正负样本选择的方案（包括上述的两种），最后发现0.3阈值的那一种效果最好；在此基础上，他们又进一步加入了finetune的步骤，最初直接使用的0.3阈值方案，后来发现效果不如0.5阈值的那种，所以才最终确定。

- Q2:

# 2. OverFeat
## Introduction

# M2Det
## Proposed Method

<div align="center"><img src="pic/picture_2020-09-20-17-52-58.png" alt="Image" style="zoom:100%;" /></div>

## 多层次特征金字塔(Multi-level Feature Pyramid Network)
该模块的**输入**是卷积神经网络不同层的feature map（例如VGG的conv4_3和conv5_3）。

**首先**，FFMv1将输入feature map的深层和浅层特征进行融合，得到**base feature**；
**FFMv1**，FFMv1的输入是形状为(1024,20,20)和(512,40,40)的两个feature map，经过处理(各自经过卷积，然后前者做一个上采样)concat在一起，形成(768,40,40)形状的feature map，即为**base feature**.
<div align="center"><img src="pic/picture_2020-09-20-18-51-32.png" alt="Image" style="zoom:50%;" /></div>

**然后**，**base feature**被送入**多个**堆叠的<TUM, FFMv2>模块当中，其中TUM是个U型网络，上采样部分的feature map可以认为是多层不同尺度的feature map；FFMv2的作用是**将TUM最后一层feature map和base feature进行融合**，输出的结果送入下一个<TUM, FFMv2>模块中



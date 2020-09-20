# Proposed Method

<div align="center"><img src="pic/picture_2020-09-20-17-52-58.png" alt="Image" style="zoom:%;" /></div>

## 多层次特征金字塔(Multi-level Feature Pyramid Network)
该模块的**输入**是卷积神经网络不同层的feature map（例如VGG的conv4_3和conv5_3）。

**首先**，FFMv1将输入feature map的深层和浅层特征进行融合，得到**base feature**；
**FFMv1**，FFMv1的输入是形状为(1024,20,20)和(512,40,40)的两个feature map，经过处理(各自经过卷积，然后前者做一个上采样)concat在一起，形成(768,40,40)形状的feature map，即为**base feature**.
<div align="center"><img src="pic/picture_2020-09-20-18-51-32.png" alt="Image" style="zoom:%;" /></div>

**然后**，**base feature**被送入**多个**堆叠的<TUM, FFMv2>模块当中，其中TUM是个U型网络，上采样部分的feature map可以认为是多层不同尺度的feature map；FFMv2的作用是**将TUM最后一层feature map和base feature进行融合**，输出的结果送入下一个<TUM, FFMv2>模块中


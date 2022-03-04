# 【论文精读】Face-to-Parameter Translation for Game Character Auto-Creation

作者： 
<div align=center><img src="https://user-images.githubusercontent.com/54792870/156784714-21d0597a-3d4c-40f1-b515-09b9d0a74d6b.png"/></div>  

论文原文：[ICCV2019](https://arxiv.org/pdf/1909.01064.pdf)  
升级版：[TPAMI](http://levir.buaa.edu.cn/publications/Neural-Rendering-for-Game_Character_Auto-creation.pdf)  
本篇精读总结主要以TPAMI上发表的最新版为准

## Motivation
角色定制化系统是RPG（Role-Playing Games）游戏中的重要组成，玩家可以根据自己的喜好来编辑游戏角色的长相。但是纯手动编辑耗时耗力。
这篇文章提出了一种方法，根据输入的脸部照片自动创建与之长相对应的3D人脸，玩家后续还可以在其基础上进一步调整。  

## Method
方法框架如下图所示  

<div align=center><img src="https://user-images.githubusercontent.com/54792870/156780046-0cf527f2-4ffd-4e90-89f6-23a44e1204b3.png"/></div>  

主要由三部分组成：
1. Facial Parameter Translator T: 根据输入的人脸照片 I 预测脸部参数 x
2. Imitator G: 根据输入的面部参数 x 合成游戏人物面部的2D图像 G(x)
3. Descriptor F: 衡量Imitator G生成的人物面部2D图像 G(x) 和 输入的人脸照片 I 之间的差异，计算loss

* 典型的游戏引擎的渲染过程是不可微的，但是本文设计的Imitator G通过模拟游戏引擎的行为，使得本文的方法可以端到端地通过梯度下降优化参数。

### Imitator G
Imitator G 的任务便是模拟游戏引擎的行为，根据输入的面部参数 x 生成游戏人脸，为了任务简洁性，这里只生成游戏人脸的正面的2D图像。  
Imitator G 的训练如下图所示：  

<div align=center><img src="https://user-images.githubusercontent.com/54792870/156783941-404d543b-e51f-421e-9087-51deafb37e20.png"/></div>  

非常直观，就是通过计算游戏引擎输出的面部图像和Imitator G生成的面部图像的差异作为loss，来训练Imitator G。
loss为L1 loss，如下所示：
<div align=center><img src="https://user-images.githubusercontent.com/54792870/156785050-04324522-a576-4017-b865-5099081925ef.png"/></div>

网络结构：  
<div align=center><img src="https://user-images.githubusercontent.com/54792870/156785244-eff3d5ea-030a-4698-a50c-1b02e182bf4e.png"/></div>

### Translator T
训练一个神经网络 T' 将facial embeddings映射到面部参数 x, 其中的facial embeddings有一个面部识别网络F<sub>recg</sub>生成：
<div align=center><img height=30 src="https://user-images.githubusercontent.com/54792870/156788622-45b6205d-27fc-4443-90b6-c13a701df4df.png"/></div>
网络细节如下图所示：
<div align=center><img src="https://user-images.githubusercontent.com/54792870/156789369-1b4a2c2a-6dce-41da-875f-0c0ac91cca70.png"/></div>
先经过面部识别网络得到facial embeddings，然后经过一系列的全连接层已经作者这几的残差注意力block后映射到面部参数x

### Descriptor F （Facial Similarity Measurement）
分三个方面衡量Imitator G生成的人物面部2D图像 G(x) 和 输入的人脸照片 I 之间的差异
<div align=center><img height=30 src="https://user-images.githubusercontent.com/54792870/156790902-dd8cd580-3772-4d46-8ebc-40ceec1c602a.png"/></div>
网络结果如下图所示：
<div align=center><img src="https://user-images.githubusercontent.com/54792870/156791535-1baa0b6b-dd38-4d7e-881d-f2cb9dac216b.png"/></div>

* Facial identity loss (L<sub>idt</sub>): global loss, 从高层语义角度衡量两张图像的相似度。具体的做法就是将两张图送入人脸识别网络得到facial embeddings，计算两个facial embeddings之间的余弦相似度：
<div align=center><img height=40 src="https://user-images.githubusercontent.com/54792870/156791911-eeb20667-d1e5-41dc-ab14-d8268276ecb6.png"/></div>

* Facial content loss (L<sub>ctt</sub>): local loss, 计算pixelwise的图像相似度。具体地，将两张图送入人脸分割网络获得人脸特征表达，此外以分割结果作为attention 权重将图像特征加权后计算二者的差异：
<div align=center><img height=40 src="https://user-images.githubusercontent.com/54792870/156792689-da186f4f-1294-4af6-8a53-40571941d4db.png"/></div>

* Loopback loss (L<sub>loop</sub>): 为提升预测的鲁棒性，将渲染好的游戏人脸图像I’输入到Translator T 来得到面部参数x'，然后强迫在循环前后的面部参数不变：
<div align=center><img height=30 src="https://user-images.githubusercontent.com/54792870/156793528-0ec7a689-1f5d-470f-a02a-3c773c073a46.png"/></div>


### 实验结果
略，见原文

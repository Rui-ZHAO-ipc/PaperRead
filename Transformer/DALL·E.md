# 【论文精读】DALL·E: Creating Images from Text

We decided to name our model using a portmanteau of the artist Salvador Dalí and Pixar’s WALL·E.  
命名由来：两位艺术家Salvador Dalí和Pixar’s WALL·E的名字结合

We extend these findings to show that manipulating visual concepts through language is now within reach.  
通过语言来操控视觉概念已经变得触手可及！！！

## DALL·E相关
项目网站：https://openai.com/blog/dall-e/  
论文：[Zero-Shot Text-to-Image Generation](https://arxiv.org/pdf/2102.12092.pdf)  
代码（官方，只放出了VAE部分）：https://github.com/openai/dall-e  
代码（pytorch版复现）：https://github.com/lucidrains/DALLE-pytorch  
视频讲解：[Youtube Yannic Kilcher](https://www.youtube.com/watch?v=j4xgkjWlfL4)  
Blog讲解：[知乎](https://zhuanlan.zhihu.com/p/394467135)，[CSDN](https://nakaizura.blog.csdn.net/article/details/116903995)  

## 一句话总结
[DALL·E](https://arxiv.org/pdf/2102.12092.pdf) [1] 是一种基于[GPT-3](https://arxiv.org/abs/2005.14165)的Text to Image生成器，其参数量高达12-billion.

## Overview
输入：1. Text Prompt or 2. Text & Image Prompt  
输出：视觉概念的合理组合，渲染文本，图像转换  
可实现：  
1. **Controlling Attributes**  控制对象的属性以及它出现的次数
2. **Drawing Multiple Objects**  对象的定位、堆叠和多个属性的控制
3. **Visualizing Perspective and Three-Dimensionality**  控制场景的视点和场景渲染的3D风格
4. **Visualizing Internal and External Structure**  用横断面视图渲染内部结构，用微距照片渲染外部结构
5. **Inferring Contextual Details**  当Text Prompt表明图像必须包含某个没有明确说明的细节时，可以“fill in the blanks”
6. **Combining Unrelated Concepts**  将两个视觉概念结合展现出来，即使这两个概念毫不相关
7. **Animal Illustrations**  拟人化的动物和物体，动物嵌合体和表情符号
8. **Zero-Shot Visual Reasoning** Zero-Shot视觉推理
9. **Geographic Knowledge** 知晓某些合理的地理信息
10. **Temporal Knowledge** 知晓某些合理的时间信息

之前的一些Text to Image方法：Conditional GAN based [2], StackGAN [3] and StackGAN++ [4], AttnGAN [5], 增加额外监督信息的方法 [6, 7, 8], 基于采样的图像生成+预训练的多模态判别模型 [9, 10]  
更多的论文整理：[2019-2021 Text To Image 论文整理](https://blog.csdn.net/qq_26136211/article/details/115206130)

## 方法
目标: 训练一个Transformer来自回归地将文本和图像tokens建模为单流（single stream）数据  
问题:  
1. 对于高分辨率图像来说，直接将像素作为图像tokens会占用大量内存  
2. 似然目标函数倾向于优先考虑像素之间的短期依赖关系：大量的模型capacity消耗在捕捉高频细节信息，而不是低频的结构。而低频结构往往对我们从视觉上识别物体更重要  

<div align=center><img src="https://user-images.githubusercontent.com/54792870/156366839-b2452b40-be07-437b-b9e6-392c08753a04.png" width="  "></div>


解决方法：two-stage training procedure  
**stage1** 训练一个dVAE将![](http://latex.codecogs.com/svg.latex?256\times256)的图像压缩到![](http://latex.codecogs.com/svg.latex?32\times32)个图像tokens，其中每个token映射成8192维的词表。这将trensformer的上下文尺寸（context size）减少了192倍（![](http://latex.codecogs.com/svg.latex?3\times256\times256\div32\div32)），不会对视觉质量造成很大的影响。  
> dVAE: discrete Variational AutoEncoder, 离散变分自编码器，详情见原[paper](https://arxiv.org/pdf/2102.12092.pdf) P12，拓展：[VAE+VQVAE](https://zhuanlan.zhihu.com/p/388299884)

**stage2** BPE Encoder编码得到的256个文本tokens和![](http://latex.codecogs.com/svg.latex?32\times32=1024)个图像tokens拼接，并训练自回归transformer来建模文本和图像tokens的联合分布。 
> [BPE](https://zhuanlan.zhihu.com/p/86965595): Byte Pair Encoding, 字节对编码


## 补充
[Transformer结构及其应用详解--GPT、BERT、MT-DNN、GPT-2](https://zhuanlan.zhihu.com/p/69290203) 知乎：Ph0en1x  
[Pretraning in NLP (预训练ELMo，GPT，BERT，XLNet)](https://nakaizura.blog.csdn.net/article/details/102136315?spm=1001.2014.3001.5502) CSDN:上杉翔二  
[Vision Transformer(iGPT，ViT，DERT，IPT，TransReID，TransGAN，TNT，CvT)](https://nakaizura.blog.csdn.net/article/details/113095927) CSDN:上杉翔二  
[预训练新范式 (Prompt-tuning，Prefix-tuning，P-tuning)](https://nakaizura.blog.csdn.net/article/details/121036309) CSDN:上杉翔二  



## 参考文献
[1] Ramesh, Aditya, et al. "[Zero-shot text-to-image generation](https://arxiv.org/pdf/2102.12092.pdf)." International Conference on Machine Learning. PMLR, 2021.  
[2] Reed, Scott, et al. "[Generative adversarial text to image synthesis](https://arxiv.org/abs/1605.05396)." International conference on machine learning. PMLR, 2016.  
[3] Zhang, Han, et al. "[Stackgan: Text to photo-realistic image synthesis with stacked generative adversarial networks](https://arxiv.org/abs/1612.03242)." Proceedings of the IEEE international conference on computer vision. 2017.  
[4] Zhang, Han, et al. "[Stackgan++: Realistic image synthesis with stacked generative adversarial networks](https://arxiv.org/abs/1710.10916)." IEEE transactions on pattern analysis and machine intelligence 41.8 (2018): 1947-1962.  
[5] Xu, Tao, et al. "[Attngan: Fine-grained text to image generation with attentional generative adversarial networks](https://arxiv.org/abs/1711.10485)." Proceedings of the IEEE conference on computer vision and pattern recognition. 2018.
[6] Reed, Scott E., et al. "[Learning what and where to draw](https://arxiv.org/abs/1610.02454)." Advances in neural information processing systems 29 (2016).  
[7] Li, Wenbo, et al. "[Object-driven text-to-image synthesis via adversarial training](https://arxiv.org/abs/1902.10740)." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2019.  
[8] Koh, Jing Yu, et al. "[Text-to-image generation grounded by fine-grained user attention](https://arxiv.org/abs/2011.03775)." Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision. 2021.  
[9] Nguyen, Anh, et al. "[Plug & play generative networks: Conditional iterative generation of images in latent space](https://arxiv.org/abs/1612.00005)." Proceedings of the IEEE conference on computer vision and pattern recognition. 2017.  
[10] Cho, Jaemin, et al. "[X-lxmert: Paint, caption and answer questions with multi-modal transformers](https://arxiv.org/abs/2009.11278)." arXiv preprint arXiv:2009.11278 (2020).

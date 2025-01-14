<div align='center'>
    CLIP-对比图文多模态预训练的读后感
</div>


<div align='right'>
    FesianXu 20210724 at Baidu Search Team
</div>

# 前言

CLIP是近年来在多模态方面的经典之作，其用大量的数据和算力对模型进行预训练，使得模型的zero-shot性能甚至可以匹敌众多数据集上的监督SOTA，实在让人惊叹不已，本文简要纪录下笔者阅读该文后的读后感以及一些启发。**如有谬误请联系指出，本文遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明并且联系笔者，谢谢**。

$\nabla$ 联系方式：
**e-mail**: FesianXu@gmail.com

**github**: https://github.com/FesianXu

**知乎专栏**: [计算机视觉/计算机图形理论与应用](https://zhuanlan.zhihu.com/c_1265262560611299328)

**微信公众号**：

![qrcode][qrcode]

----



# 再说到语义标签

之前在博文[1,2]中曾经简单地介绍过语义标签（semantic label）与多模态模型之间的一些关系，为了对这个话题有着更全面的了解，请读者先移步[1,2]对语义标签进行简单了解。在[1]的最后，我们谈到

> 这些任务都是需要很高层次的语义标注才能实现的。通常来说，此时人工标注能做到的就是给定一个图片，让多个人以相近的标准去进行描述，然后形成图文对`<image, text#1, text#2...text#n>`，让模型进行学习

作为语义标签中最为切实可行的方案，对图片进行文字描述（caption）是可行的，而且互联网上也存在着海量这样的数据。通常来说，这类型的数据如Fig 1.1所示。

![image_caption_pair][image_caption_pair]

<div align='center'>
    <b>
        Fig 1.1 图片-文本对的形式的多模态数据集，常用于进行预训练。通常对于一张图片，会有一个以上的文本描述。
    </b>
</div>

在CLIP[3]这篇工作中，作者提出了**对比图文预训练** （Contrastive Language-Image Pretraining）方法对从网络中收集到的4亿（400M）条图文对进行预训练，因此这个工作也是采用语义标签对模型进行预训练的经典例子。在这个过程中，值得注意的是作者团队采用了巨大的batch size，一个batch size竟达到了32,768，当然这也需要巨量的算力资源。这种“大力出奇迹”的做法，使得CLIP模型的zero-shot能力惊人地出色，在众多数据集中甚至超过了采用全监督的SOTA方法。我们接下来的篇幅主要看看CLIP的模型设计，训练策略以及最主要的，作者团队在论文中的实验结果和分析。



# 对比图文预训练 CLIP

CLIP的模型结构并没有特别多值得注意的地方，其采用的是经典的双塔结构，对于图片域和文本域有着不同的图片编码器（Image Encoder）和文本编码器（Text Encoder）。其中文本编码器采用了经典的Transformer结构[4]，而图片编码器则采用了两种：第一种是改进后的ResNet，作者选择用基于注意力的池化层去替代ResNet的全局池化层，此处的注意力机制同样是与Transformer类似的多头QKV注意力；作者同样采用ViT结构[5]作为第二种图片编码器进行实验。本文用$f_{\mathrm{Text}}(\cdot)$表示文本编码器，$f_{\mathrm{Img}}(\cdot)$表示图片编码器，$\mathbf{x}_{Img} \in \mathbb{R}^{N \times H \times W \times C}$表示一个batch的图片，而$\mathbf{x}_{\mathrm{Text}} \in \mathbb{R}^{N \times S}$表示一个batch的文本，那么有：
$$
\begin{aligned}
\mathbf{f}_{\mathrm{img}} &= f_{\mathrm{Img}}(\mathbf{x}_{Img}) \in \mathbb{R}^{N \times D_{i}} \\
\mathbf{f}_{\mathrm{text}} &= f_{\mathrm{Text}}(\mathbf{x}_{Text}) \in \mathbb{R}^{N \times D_{t}}
\end{aligned}
\tag{2.1}
$$
通过线性映射层将图片特征$\mathbf{f}_{\mathrm{img}}$和文本特征$\mathbf{f}_{\mathrm{text}}$都映射到相同的嵌入特征维度$D_{e}$，那么有：
$$
\begin{aligned}
\mathbf{f}_{\mathrm{img}}^{e} &= \mathbf{f}_{\mathrm{img}} \mathbf{W}_{\mathrm{img}} \in \mathbb{R}^{N \times D_{e}} \\
\mathbf{f}_{\mathrm{text}}^{e} &= \mathbf{f}_{\mathrm{text}} \mathbf{W}_{\mathrm{text}} \in \mathbb{R}^{N \times D_{e}} 
\end{aligned}
\tag{2.2}
$$
为了保证数值尺度的一致性，对其进行L2标准化，即是：
$$
G_{L2}(\mathbf{x}) = \dfrac{\mathbf{x}_i}{\sqrt{\sum_{i}^{D}\mathbf{x}_i^2}}
\tag{2.3}
$$
那么有：
$$
\begin{aligned}
\mathbf{f}^{\mathrm{norm}}_{\mathrm{img}} &= G_{L2}(\mathbf{f}_{\mathrm{img}}) \\
\mathbf{f}^{\mathrm{norm}}_{\mathrm{text}} &= G_{L2}(\mathbf{f}_{\mathrm{text}})
\end{aligned}
\tag{2.4}
$$
![clip_training_model][clip_training_model]

<div align='center'>
    <b>
        Fig 2.1 CLIP的负样本采样，采用了in-batch负采样的方法。其CLIP模型也是经典的双塔结构。
    </b>
</div>

此时如Fig 2.1所示，对图片嵌入特征和文本嵌入特征进行矩阵相乘。那么形成的打分矩阵上，对角线上都是配对的正样本对打分，而矩阵的其他元素，则是由同个batch内的图片和不配对的文本（相反亦然）组成的负样本。这种策略可以形成$N^2-N$个负样本。整个过程可以用公式(2.5)描述。
$$
\mathbf{M} = (\mathbf{f}^{\mathrm{norm}}_{\mathrm{img}}) (\mathbf{f}^{\mathrm{norm}}_{\mathrm{text}})^{\mathrm{T}} \in \mathbb{R}^{N \times N}
\tag{2.5}
$$
而后只需要对$\mathbf{M}$的每一行和每一列求交叉熵损失，并且加和起来即形成了总损失了。其中每一行可以视为是同个图片，与同个batch内其他所有样本对的文本进行组合构成的负样本对形成的损失，而每一列自然就是同个文本，对于每个图片进行组合而构成的损失了。整个过程如下面的伪代码所示。

```python
# image_encoder - ResNet or Vision Transformer
# text_encoder - CBOW or Text Transformer
# I[n, h, w, c] - minibatch of aligned images
# T[n, l] - minibatch of aligned texts
# W_i[d_i, d_e] - learned proj of image to embed
# W_t[d_t, d_e] - learned proj of text to embed
# t - learned temperature parameter
# extract feature representations of each modality
I_f = image_encoder(I) #[n, d_i]
T_f = text_encoder(T) #[n, d_t]
# joint multimodal embedding [n, d_e]
I_e = l2_normalize(np.dot(I_f, W_i), axis=1)
T_e = l2_normalize(np.dot(T_f, W_t), axis=1)
# scaled pairwise cosine similarities [n, n]
logits = np.dot(I_e, T_e.T) * np.exp(t)
# symmetric loss function
labels = np.arange(n)
loss_i = cross_entropy_loss(logits, labels, axis=0)
loss_t = cross_entropy_loss(logits, labels, axis=1)
loss = (loss_i + loss_t)/2
```

CLIP的模型结构和正负样本组成策略并不复杂，其负样本构成方式是经典的batch negative方式，也即是从batch内部去构成负样本，而CLIP的贡献点在于采用了海量的图文对数据和超大batch size进行预训练，并不在于其模型结构。我们看一下CLIP是如何去进行zero-shot任务的。如Fig 2.2所示，考虑到大部分的数据集的标签都是以单词的形式存在的，比如“bird”，“cat”等等，然而在预训练阶段的文本描述大多都是某个短句，为了填补这种数据分布上的差别，作者考虑用“指示上下文”（guide context）对标签进行扩展。以Fig 2.2为例子，可以用`a photo of a <LABEL>.`作为文本端的输入，其中的`<LABEL>`恰恰是需要预测的zero-shot标签。

![zero_shot_frame][zero_shot_frame]

<div align='center'>
    <b>
        Fig 2.2 将CLIP应用到zero-shot中，需要文本端采用“指示上下文”的形式。
    </b>
</div>

考虑到以单词作为标签存在多义的情况，比如在Oxford-IIIT Pet dataset 数据集中`boxer`表示斗牛犬，而在其他数据集中则可能表示拳击运动；在ImageNet中，`crane`同时表示了起重机和鹤。这种词语的多义显然对是因为缺少对标签的上下文描述导致的。为了解决这种问题，作者在指示上下文中添加了一些提示标签类型的词语，比如`A photo of a <LABEL>, a type of pet. `。作者将这个方法称之为“prompt engineering”。在合适地选取了不同的指示上下文，并且将其打分进行ensemble之后。作者发现这些Tricks竟能在zero-shot实验上提高5个绝对百分位，如Fig 2.3所示。

![prompt_engineering_ensemble][prompt_engineering_ensemble]

<div align='center'>
    <b>
        Fig 2.3 采用了prompt engineering和ensemble之后，可以在zero-shot指标上提高5个绝对百分位。这个说明了通过指示上下文，提供标签的上下文信息可以有效地提高zero-shot效果。
    </b>
</div>
在zero-shot这块，作者也做了充足的实验。首先作者通过和经过强监督学习的Resnet-50提取的特征对比，任务都是分类任务，因此作者基于Resnet-50和CLIP提取出的特征，只是训练了最后的分类器，分类结果如Fig 2.4所示。可以发现仅仅通过无监督的对比学习预训练得到的特征，即便是和强监督模型特征相比也是不分伯仲的。同时可以发现，zero-shot CLIP在一些动作识别任务中，比如Kinetics 700，UCF 101中有着比较大的提高，作者认为这可能是因为目前的文本描述中有很多以动词，动作为中心的句子导致的。


![zero_shot_vs_linear_probe_resnet50][zero_shot_vs_linear_probe_resnet50]

<div align='center'>
    <b>
        Fig 2.4 CLIP的zero-shot性能，与Resnet-50的特征进行对比。
    </b>
</div>

作者同样将CLIP应用在了few-shot中，如Fig 2.5所示，横坐标是few-shot中的X-ways，指的是用多少有标注的样本进行Linear Probe的分类器训练（这个过程中仅仅采用了预训练模型的特征，而线性分类器需要重新训练）。从图中可以发现zero-shot CLIP大概相当于 4-ways few-shot CLIP的结果，而显然的，Linear Probe CLIP比其他few-shot基线模型有着更为优秀而稳定的表现，领先了有10多个绝对百分点。这个实验同时也揭露了zero-shot和few-shot的区别，在监督学习中模型需要基于标签学习出相同类别样本的共同特征，不同类样本的差别，因此通常都需要更多的标注数据才能达到较理想的效果。如果没有太多的标注数据提供类别的上下文信息，那么性能可能比zero-shot还差。而且，从预训练中学习到的概念可能是很通用的，比如一张图中可能有很多视觉实体，如果不提供一定的标注去声明任务需要注意的视觉实体，那么就无法完成任务。而zero-shot由于是直接学习概念，并且和概念进行对比，似乎就不会存在这个问题。

![zero_shot_labeld_example_per_class][zero_shot_labeld_example_per_class]

<div align='center'>
    <b>
        Fig 2.5 few-shot CLIP与其他few-shot基线模型的对别。
    </b>
</div>



# 笔者的个人启示

基于双塔结构的CLIP模型在数据量足够的情况下，可以在预训练阶段学习到很多通用的视觉语义概念，并且给下游任务提供非常大的帮助。受限于端到端的方式，目前的负样本数量仍然是受限于mini batch size的大小，在MoCo中给出了一个可行的解决方案[6,7]。这种预训练方式能在大规模的商业搜索，推荐广告系统中广泛应用，值得我们研究。

# Reference

[1]. https://fesian.blog.csdn.net/article/details/114958239

[2]. https://fesian.blog.csdn.net/article/details/118256321

[3]. Radford, A., Kim, J. W., Hallacy, C., Ramesh, A., Goh, G., Agarwal, S., ... & Sutskever, I. (2021). Learning transferable visual models from natural language supervision. *arXiv preprint arXiv:2103.00020*.

[4]. Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones,L., Gomez, A. N., Kaiser, Ł., and Polosukhin, I. Attention is all you need. In Advances in neural information processing systems, pp. 5998–6008, 2017  

[5]. Dosovitskiy, Alexey, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani et al. “An image is worth 16x16 words: Transformers for image recognition at scale.” arXiv preprint arXiv:2010.11929 (2020).

[6]. https://fesian.blog.csdn.net/article/details/119515146

[7]. He, K., Fan, H., Wu, Y., Xie, S., & Girshick, R. (2020). Momentum contrast for unsupervised visual representation learning. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (pp. 9729-9738).







[qrcode]: ./imgs/qrcode.jpg
[image_caption_pair]: ./imgs/image_caption_pair.png
[clip_training_model]: ./imgs/clip_training_model.png
[zero_shot_frame]: ./imgs/zero_shot_frame.png
[prompt_engineering_ensemble]: ./imgs/prompt_engineering_ensemble.png
[zero_shot_vs_linear_probe_resnet50]: ./imgs/zero_shot_vs_linear_probe_resnet50.png
[zero_shot_labeld_example_per_class]: ./imgs/zero_shot_labeld_example_per_class.png
[natural_distribution_shift]: ./imgs/natural_distribution_shift.png








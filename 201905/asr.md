<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


>提示：在github中无法正常显示公式，download repo到本地用markdown打开可以正常显示，也可以使用[GitHub with MathJax](https://chrome.google.com/webstore/detail/mathjax-plugin-for-github/ioemnmodlmafdkllaclgeombjnmnbima/related)插件来解决此问题

# [Very Deep Self-Attention Networks for End-to-End Speech Recognition](https://arxiv.org/pdf/1904.13377.pdf)

这篇文章算是少数用纯transformer来做ASR的了，结果还算不错，有几个结论值得借鉴：
> #transformer的层数一定要足够深，论文中的层数都达到了48层(encoder尤其需要更深一些，encoder用了36层，decoder用了12层)；同时，作者做了另外的实验，把层数减少一半，模型节点数目double，发现性能掉了挺多，说明transformer的确需要更深一些，而不是模型容量大就足够

> #为了防止transformer过拟合，中间的一些drop，stochastic layers的设计也挺重要
作者的工作目测还是很solid的，[代码](https://github.com/quanpn90/NMTGMinor/tree/audio-encoder/)可循


# [AISHELL-2: Transforming Mandarin ASR Research Into Industrial Scale](https://arxiv.org/pdf/1808.10583.pdf)
国内一家专注做语音交互的公司[希尔贝克](http://www.aishelltech.com/sy)release的1000h的中文语音数据，非常高的质量，数据量也足够大，并且作为学术研究可以免费申请

# [Hard Sample Mining for the Improved Retraining of Automatic Speech Recognition](https://arxiv.org/pdf/1904.08031.pdf) 
作者的出发点我觉得还是可以的，使用一种类似半监督的方式，来进行数据筛选。
流程如下：
> 1.使用300h SB数据训练End2End的ASR系统

> 2.使用100h的Fisher数据，送入ASR中，得到识别结果$\hat y$，同时也有golden $y$，再训练一个模型，分辨$\hat y$和$y$。再使用500h的Fisher语料，一个是随机挑的，一个是"hard" samples的数据，然后发现这种"harde"语料的性能比随机挑选的更好

文章的出发点我还是很认可，不过论文中有很多关键步骤根本没有讲清楚，所以我觉得有2种原因: 1.中间的trick作者并不愿意详细讲出来；2.另外一种嘛，emmmm

# [Automatic Spelling Correction with Transformer for CTC-based End-to-End Speech Recognition](https://arxiv.org/pdf/1904.10045.pdf)
多阶段的ASR模型，首先训练一个 “声学模型”，再训练一个纠错模型

# [Sequence-to-Sequence Speech Recognition with Time-Depth Separable Convolutions](https://arxiv.org/pdf/1904.02619.pdf)
Facebook家做ASR的一篇文章，这篇文章的一个亮点是使用time-depth separable convolution
其中TDS网络结构主要有下面3个部分网络组成
<img src="./figures/asr_fig1.jpg" width="550">

论文使用了很多的一些工程trick，这些trick主要有以下几个方面
### Decoder
去除了scheduled sampling, input feeding, location-based attention等时序相关的的一些操作，这样在训练过程中，由于是teacher-forcing的，所以无需等待，直接一并算完，调cudnn的RNN，节省多次调研kernel的开销，并且attention使用的也是点积attnetion
### Random samping
Decoder端的token使用了很少比例的random sampling以此来缓解训练和将来解码的mismatch和overfitting
### Soft window pre-training
<img src="./figures/asr_fig2.jpg" width="300">

其中$W_{ij}=(i-\frac{T}{U}j)$用于约束attention的大致范围，加速收敛
### Regularization
#### Dropout
#### label smoothing
使用的是0.05的smooth factor
#### word piece sampling
decoder端的分词不仅仅是BPE的argmax，而是多组后选，这个多后选来自google 的[sentence piece](https://arxiv.org/pdf/1804.10959.pdf)方法 on-the-fly进行，缓解过拟合，增加模型泛化能力，我个人还是非常推崇这类正则方法的，至少是对条件概率连乘做了另外一种break方式，用这种方法解码的时候可能还能够通过解码出大单元来解决特定字不准确的问题
### BeamSearch的稳定性trick
#### attention限制
当前attention的peak离上一个attention peak 不能相隔太远
#### EOS
要大于一个threshold才结束防止句子过短

总体来说，这篇文章也算是干货满满的

# [Transformers with convolutional context for ASR](https://arxiv.org/pdf/1904.11660.pdf)
又是Facebook家做的ASR的一个工作，这个工作非常简单粗暴，就是用transformer来做ASR，一个非常重要的就是，不再向NMT那样使用Position Embedding，而是使用几层卷积代替position embedding，然后再累深层的transformer；实验结论是Encoder端需要多层transformer，Decoder端有6~8层就够了
网络结构图为
<img src="./figures/asr_fig3.jpg" width="500">

# [Language Modeling with Deep Transformers](https://arxiv.org/pdf/1905.04226.pdf)
使用深层次的self-attention来做language model建模的一篇文章，文章有个很有意思的说法是使用transformer来进行language model任务时，可以不需要position embedding，甚至convolution也不需要来进行relative信息的建模，这个结论我暂时表示有点疑惑，作者解释模型可能通过monotonic信息增益来确定position相关的信息，emmm

# [DYNAMIC EVALUATION OF TRANSFORMER LANGUAGE MODELS](https://arxiv.org/pdf/1904.08378.pdf)
这篇文章在两个非常重要的工作中展开的[transformer-xl](https://arxiv.org/pdf/1901.02860.pdf) [dynamic evaludation](https://arxiv.org/pdf/1709.07432.pdf)

首先说一下dynamic eval这个方法，其原理图为：

<img src="./figures/asr_fig4.jpg" width="600">

说一下大致的过程，就是把一个sequence给break成多个片段，我们一般更新模型参数都是整个sequence的loss average之后再更新模型参数，这个方法是，前一个片段计算loss之后直接把lstm的模型参数给更新了，然后拿着更新之后的模型参数，去跑下一个片段的forward过程

然后我们再看一下transformer-xl这篇文章做的事情，其实就是一个Block RNN，重用历史，截断历史梯度，从而能够有很长的历史ctx信息

<img src="./figures/asr_fig5.jpg" width="600">

这篇文章做的工作就是把这两个language model中非常强大的方法融合再一起，在多个lm任务上取得不错的效果。

# [RWTH ASR Systems for LibriSpeech: Hybrid vs Attention - w/o Data Augmentation](https://arxiv.org/pdf/1905.03072.pdf)
在帧级建模上去取得了非常不错的效果，而且看github上面的repo，发现这是一个组织，里面有非常多的ASR相关的实验，很值得关注，并且还有一个专门的工具，其[文档](https://returnn.readthedocs.io/en/latest/)


# [Generating Long Sequences with Sparse Transformers](https://arxiv.org/pdf/1904.10509.pdf)
Open AI出的sparse transformer，其主要的优点在于，可以使用transformer建模非常长的历史，文章为了实现此功能，sparsr的选取还是非常有意思的，很类似TTS合成预测16bit 65536将其分成高8位256和低8位256预测方法；作者对于local历史不进行sparse的self-attention，对于很长的历史，变使用跳帧的方式稀疏做self-attention

<img src="./figures/asr_fig6.jpg" width="600">

# [Deep Residual Output Layers for Neural Language Generation](https://arxiv.org/pdf/1905.05513.pdf)
一篇PMLP的论文，思路非常棒，跟我们CPC从bilinear到dnn交互非常类似。而且效果也很好。
首先介绍一下这篇文章的出发点或者说期望解决的问题，

> shallow fusion 问题，即在最后一层是一个线性交互

> 过拟合问题

一般我们在做分类的时候，最后一层的softmax，是用softmax weight和最终的hidden做一个linear；这里就有一个非常强的假设，我们网络已经把特征提取得非常牛逼，非常线性可分了，然后使用线性分类器就可以搞定了。用公式表示为

<img src="./figures/asr_fig7.jpg" width="300">

可以看到，这样的方式，学习到的$W_i^T$非常的不具备丰富表征能力，一个最简单的例子，对于$w_i$和$w_j$，两个模型参数老死不相往来，这个是有害的，所以最让人容易想到的一个改进方案是*weight tying*，这个方法在语言模型，机器翻译上面已经是一个标配做法，即使得target端的embedding和softmax weight进行共享，这样就可以变成

<img src="./figures/asr_fig8.jpg" width="300">

可以看到，这里仍然是点积来衡量相似度的，而一个更好的办法是bilinear，所以最后的公式用Bilinear又可以表示为

<img src="./figures/asr_fig9.jpg" width="300">

这里就跟我们CPC很类似的，这里的bilinear如果用非线性来进行交互，是可以得到很大的gain的，作者将这里的非线性
交互拆解成让$E$和$h_t$分别做非线性，用$g_out(.)$和$g_in(.)$表示非线性，整个公式可以表示为

<img src="./figures/asr_fig10.jpg" width="500">

网络结构图为

<img src="./figures/asr_fig11.jpg" width="600">

根据上面的公式，作者先解决了shallow fusion问题，作者还提出另外一个解决过拟合问题的方法
一种变体的dropout，作者仅仅在$E$这个分支这里引入了残差和dropout，在每一层都有dropout,而且不同之处在于，采样一次dropout mask，多层残差都用一个dropout mask，类似zoneout这种正则方式

当然，作者这种方法在语言模型和NMT(transformer)中都取得了比较明显的提升，结论还是非常solid


# naivebayes
## 朴素贝叶斯模型
使用贝叶斯模型对训练集执行垃圾邮件分类任务，用测试集检验模型的运行效果，垃圾邮件数据集为公开的数据集。
## 公开数据集
我们使用Ling-Spam数据集，可以通过链接http://www.aueb.gr/users/ion/data/lingspam_public.tar.gz 下载。
## 估计模型参数
我们划分Ling-Spam数据集的一部分为测试集来进行模型参数估计，测试集中的每一封信件都被标记为垃圾邮件（标记为$C=c_1​$）或非垃圾邮件（标记为 $C=c_2​$ ），从信件的命名可以看出来。我们选择已经经过文本清洗的数据集，位于lemm_stop文件夹中。我们将数据集中所有的出现次数超过20次的单词作为一系列二项分布的伯努利随机变量，记为${W_1,...,W_k}​$，这些随机变量表示单词$W_i​$是否在一封信件中存在，我们取$W_i=1​$表示存在，反之，$W_i=0​$表示不存在。这些随机变量作为朴素贝叶斯模型的特征，并且假定这些特征在已知信件的类别C下是相互独立的。
信件类别的先验概率$P(C)​$使用极大似然估计求得，公式如下：
$$
P(C=c_i) = \frac{n(c_i)}{n}
$$
其中$n(c_i)$是类别$c_i$的信件数，n是训练集中的总信件数。因为在本例中只有两个类别所以$\sum_{i=1}^2 P(C=c_i)=1$。
估计已知信件类别C求信件特征$X={W_1,...,W_k}$的先验概率$P(X\mid C)$，因为特征过多，我们无法求出所有的联合分布，所以我们采用朴素或者天真的方式对模型进行假设。我们假设在已知信件类别C的时候，信件的特征之间是相互独立的，即信件特征${W_1,...,W_k}$是关于类别C的条件独立。所以先验概率$P(X \mid C)$的估计公式如下：
$$
P(X\mid C)=\prod_{i_1}^k P(W_i\mid C)
$$
其中连乘的条件概率$P(W_i|C)​$的估计公式也采用极大似然估计：
$$
P(W_i=1\mid C=c_j)=\frac{n(w_i,c_j)}{n(c_j)}
$$
$$
P(W_i=0 \mid C=c_j)=1-P(W_i=1\mid C=c_j)
$$
其中$n(w_i,c_j)$是在类别$c_j$中包含单词$w_i$的信件数。
使用极大似然估计可能会造成零概率问题，比如在本例中我们选择的特征为整个邮件数据集中的高频词${W_1,...,W_k}$，而这些高频词不一定都在垃圾邮件中存在，而根据我们模型中计算条件概率$P(W_i|C)$的公式可以知道，当$n(w_i,c_j)=0$时，我们算出的概率也为零，这样就不合理，因为这只能说明我们的训练集不够完全。所以我们需要将极大似然进行smooth，比如相对频率计算公式为：
$$
\hat{\theta}_{yi} = \frac{ N_{yi} + \alpha}{N_y + \alpha n}
$$
$\hat{\theta}_{yi}$指的是本例中的$P(W_i=1|C=c_j)$，$N_{yi}$对应$n(w_i,c_j)$，$N_y$对应$n(c_j)$，$\alpha$是一个平滑参数，当$a=1$时就是Laplace smoothing，即公式如下：
$$
P(W_i=1\mid C=c_j) = \frac{ n(w_i,c_j) + 1}{n(c_j) + n}
$$


## 使用模型进行分类

根据贝叶斯定理进行分类，我们使用最大后验概率求解：
$$
\begin{align}\begin{aligned}P(y \mid x_1, \dots, x_n) \propto P(y) \prod_{i=1}^{n} P(x_i \mid y)\\\Downarrow\\\hat{y} = \arg\max_y P(y) \prod_{i=1}^{n} P(x_i \mid y),\end{aligned}\end{align}
$$
在本例中，$y$指的是我们要预测邮件的类别$C=c_j$，$x_i$是指邮件选取高频词的二项分布的随机变量$W_i$，我们要求使得如下公式值最大的邮件类别$c_j$的值：
$$
\arg\max_j P(C=c_j\mid W_1,\dots,W_k)=\\arg\max_j P(C=c_j)\prod_i^kP(W_i\mid C=c_j)
$$
”宁可放过三千，不可错过一封正常邮件“，拦下一封正常的邮件比起让一封垃圾邮件通过测试更让人无法接受。所以，我们可以调整一封邮件是垃圾邮件的概率与一封邮件正常的概率的比率是否超过一个阈值$t$：
$$
\frac{P(C=c_1\mid W_1,\dots,W_k)}{P(C=c_2\mid W_1,\dots,W_k)}>t
$$
当$t=1$时，我们就认为当一封邮件是垃圾邮件的概率超过0.5就可以判断其为垃圾邮件，但是我们取$t=3$，即当一封邮件是垃圾邮件的概率超过0.75时我们才判定它是垃圾邮件。

## 评价模型

我们用数据集的其他部分作为测试集，计算出识别垃圾邮件的准确率。

## 模型参数

贝叶斯分类邮件的参数包括高频次的次数（20），平滑常数（$\alpha$)以及阈值$t=3$


---
layout: post
title:  机器学习（三十五）——Probabilistic Robotics, 推荐算法中的常用排序算法, NLP机器翻译常用评价度量, Adaboost
category: ML 
---

# Probabilistic Robotics

这篇心得主要根据Sebastian Thrun的Probabilistic Robotics课程的ppt来写。

>注：Sebastian Thrun，德国波恩大学博士（1995年）。先后执教于CMU和Stanford。

网址：

http://robots.stanford.edu/probabilistic-robotics/ppt/

## 贝叶斯过滤器

假定我们需要根据测量值z来判断门的开关。显然，这里的$$P(open\mid z)$$是诊断式（**diagnostic**）问题，而$$P(z\mid open)$$是因果式（**causal**）问题。通常来说，后者比较容易获取，而前者可以基于后者使用贝叶斯公式计算得到。

一般将$$P(z\mid x)$$称为**Sensor model**。

针对多相关测量值问题，这里有一个和朴素贝叶斯假设相仿的**Markov assumption**——假设$$z_n$$独立于$$z_1,\dots,z_{n-1}$$（即“现在”不依赖于“过去”），则：

$$P(x\mid z_1,\dots,z_n)=\frac{P(z_n\mid x)P(x\mid z_1,\dots,z_{n-1})}{P(z_n\mid z_1,\dots,z_{n-1})}(\text{Bayes})
\\=\eta P(z_n\mid x)P(x\mid z_1,\dots,z_{n-1})=\eta_{1,\dots,n}\prod_{i=1}^nP(z_i\mid x)P(x)(\text{Markov})$$

>注：以下的推导过程注释中，如无特别说明。均以Bayes指代Bayes' theorem，以Markov指代Markov assumption。

上式中的$$\eta$$表示概率的归一化系数。

除了测量值z之外，一般的控制系统中还有动作（Action）的概念。比如打开门就是一个Action。Action会导致系统的状态发生改变（也可不变）。如下图所示：

![](/images/article/state_trans.png)

通常，将$$P(x\mid u,x')$$称作**Action Model**。其中，u表示Action，而x'表示系统的上一个状态。

一般的，**新的测量值会减少系统的不确定度，而新的Action会增加系统的不确定度。**

综上，一个贝叶斯过滤器（Bayes Filters）的框架包括：

输入：

1.观测值z和Action u的序列：$$d_t=\{u_1,z_1,\dots,u_t,z_t\}$$

2.Sensor model：$$P(z\mid x)$$

3.Action model：$$P(x\mid u,x')$$

4.系统状态的先验概率：$$P(x)$$

输出：

1.估计动态系统的状态X。

2.状态的后验概率，也叫**Belief**：

$$\begin{align}
\mathbf{Bel(x_t)}&=P(x_t\mid u_1,z_1,\dots,u_t,z_t)
\\&=\eta P(z_t\mid x_t,u_1,z_1,\dots,u_t)P(x_t\mid u_1,z_1,\dots,u_t)(\text{Bayes})
\\&=\eta P(z_t\mid x_t)P(x_t\mid u_1,z_1,\dots,u_t)(\text{Markov})
\\&=\eta P(z_t\mid x_t)\int P(x_t\mid u_1,z_1,\dots,u_t,x_{t-1})P(x_{t-1}\mid u_1,z_1,\dots,u_t)\mathrm{d}x_{t-1}(\text{Total prob.})
\\&=\eta P(z_t\mid x_t)\int P(x_t\mid u_t,x_{t-1})P(x_{t-1}\mid u_1,z_1,\dots,u_t)\mathrm{d}x_{t-1}(\text{Markov})
\\&=\eta P(z_t\mid x_t)\int P(x_t\mid u_t,x_{t-1})P(x_{t-1}\mid u_1,z_1,\dots,z_{t-1})\mathrm{d}x_{t-1}(\text{Markov})
\\&=\eta P(z_t\mid x_t)\int P(x_t\mid u_t,x_{t-1})\mathbf{Bel(x_{t-1})}\mathrm{d}x_{t-1}
\end{align}$$

上式也可以写作：

**预测**：

$$\overline{\mathbf{Bel(x_t)}}=\int P(x_t\mid u_t,x_{t-1})\mathbf{Bel(x_{t-1})}\mathrm{d}x_{t-1}$$

**修正**：

$$\mathbf{Bel(x_t)}=\eta P(z_t\mid x_t)\overline{\mathbf{Bel(x_t)}}$$

熟悉卡尔曼滤波的同学大概已经看出来了。没错！贝叶斯过滤器是一大类算法的统称。这些算法包括Kalman filters、Particle filters、Hidden Markov models、Dynamic Bayesian networks、Partially Observable Markov Decision Processes (POMDPs)等。

## 递归最小二乘法

http://www.blog.huajh7.com/adaptive-filters-lms-rls-kalman-filter-1/

## 卡尔曼滤波

>注：Rudolf (Rudi) Emil Kálmán，1930～2016，匈牙利出生的美国科学家。哥伦比亚大学博士（1957），先后执教于斯坦福大学和佛罗里达大学。现代控制理论的里程碑人物，美国科学院院士。   
>卡尔曼滤波从纯数学的角度讲，并没有多大意义。因此，主流数学家们在很长一段时间内，并不承认Kálmán是数学家。只是由于卡尔曼滤波在工程界的巨大影响力，才不得不于2012年，授予其美国数学协会院士。

Kalman filters是一种高斯线性滤波器。

参考：

http://www.cs.unc.edu/~welch/media/pdf/kalman_intro.pdf

Gregory Francis Welch写的卡尔曼滤波科普文。

>注：Gregory Francis Welch，北卡罗莱娜大学博士（1997）。中佛罗里达大学教授。

http://www.cs.unc.edu/~welch/kalman/media/misc/kalman_intro_chinese.zip

上文的中文版。

https://zhuanlan.zhihu.com/p/21294526

知乎诸位大神的科普文。

http://www.docin.com/p-976961701.html

动态相对定位中自适应滤波方法的研究

《自适应动态导航定位》，杨元喜著。

>注：杨元喜，1956年生，大地测量学家。中国科学院院士。

# 推荐算法中的常用排序算法

## Pointwise方法

Pranking (NIPS 2002), OAP-BPM (EMCL 2003), Ranking with Large Margin Principles (NIPS 2002), Constraint Ordinal Regression (ICML 2005)。

## Pairwise方法

Learning to Retrieve Information (SCC 1995), Learning to Order Things (NIPS 1998), Ranking SVM (ICANN 1999), RankBoost (JMLR 2003), LDM (SIGIR 2005), RankNet (ICML 2005), Frank (SIGIR 2007), MHR(SIGIR 2007), Round Robin Ranking (ECML 2003), GBRank (SIGIR 2007), QBRank (NIPS 2007), MPRank (ICML 2007), IRSVM (SIGIR 2006)。

## Listwise方法

LambdaRank (NIPS 2006), AdaRank (SIGIR 2007), SVM-MAP (SIGIR 2007), SoftRank (LR4IR 2007), GPRank (LR4IR 2007), CCA (SIGIR 2007), RankCosine (IP&M 2007), ListNet (ICML 2007), ListMLE (ICML 2008) 。

## 参考

https://mp.weixin.qq.com/s/YjYVE6jzySVsZmXSPivB5w

达观数据搜索引擎排序实践（上篇）

https://mp.weixin.qq.com/s/UpN7tAMjbFLSPcDYsWaykg

达观数据搜索引擎排序实践（下篇）

https://mp.weixin.qq.com/s/xigME-griWFwEvvPNqWuvg

美团点评联盟广告的场景化定向排序机制

# NLP机器翻译常用评价度量

机器翻译的评价指标主要有：BLEU、NIST、Rouge、METEOR等。

参考：

http://blog.csdn.net/joshuaxx316/article/details/58696552

BLEU，ROUGE，METEOR，ROUGE-浅述自然语言处理机器翻译常用评价度量

http://blog.csdn.net/guolindonggld/article/details/56966200

机器翻译评价指标之BLEU

http://blog.csdn.net/han_xiaoyang/article/details/10118517

机器翻译评估标准介绍和计算方法

http://blog.csdn.net/lcj369387335/article/details/69845385

自动文档摘要评价方法---Edmundson和ROUGE

https://mp.weixin.qq.com/s/XiZ6Uc5cHZjczn-qoupQnA

对话系统评价方法综述

https://zhuanlan.zhihu.com/p/33088748

数据集和评价指标介绍

# Adaboost

Adaboost是Yoav Freund和Robert Schapire于1997年提出的算法。两人后来因为该算法被授予Gödel Prize（2003）。

>Yoav Freund，UCSC博士，UCSD教授。

>Robert Elias Schapire，MIT博士。先后供职于Princeton University、AT&T Labs和Microsoft Research。

>Gödel Prize，由欧洲计算机学会（EATCS）与美国计算机学会基础理论专业组织（ACM SIGACT）于1993年共同设立，颁给理论计算机领域最杰出的学术论文。其名称取自Kurt Gödel。

>Kurt Friedrich Gödel，1906～1978，奥地利逻辑学家，数学家，哲学家，后加入美国藉。维也纳大学博士（1930）。在逻辑学方面，他是继Aristotle、Gottlob Frege之后最伟大的逻辑学家。在数学方面，他以哥德尔不完备定理著称，和Bertrand Russell、 David Hilbert、Georg Cantor齐名。

Adaboost既可用于分类问题，也可用于回归问题。这里仅针对二分类问题进行讨论。

假设我们有数据集$$\{(x_1, y_1), \ldots, (x_N, y_N)\}$$，其中$$y_i \in \{-1, 1\}$$，还有一系列弱分类器$$\{k_1, \ldots, k_L\}$$。

由于Boost算法是个串行算法，每次迭代就会加入一个弱分类器。这样m-1次迭代之后的分类器如下所示：

$$C_{(m-1)}(x_i) = \alpha_1k_1(x_i) + \cdots + \alpha_{m-1}k_{m-1}(x_i)$$

而m次迭代之后的分类器则为：

$$C_{m}(x_i) = C_{(m-1)}(x_i) + \alpha_m k_m(x_i)$$

如何选择新加入的弱分类器$$k_m$$和对应的权重$$\alpha_m$$呢？我们可以定义误差E如下所示：

$$E = \sum_{i=1}^N e^{-y_i C_m(x_i)}$$

令$$w_i^{(1)} = 1,w_i^{(m)} = e^{-y_i C_{m-1}(x_i)}$$，则：

$$E = \sum_{i=1}^N w_i^{(m)}e^{-y_i\alpha_m k_m(x_i)}$$

因为$$k_m$$分类正确时，$$y_i k_m(x_i) = 1$$，分类错误时，$$y_i k_m(x_i) = -1$$。所以：

$$E = \sum_{y_i = k_m(x_i)} w_i^{(m)}e^{-\alpha_m} + \sum_{y_i \neq k_m(x_i)} w_i^{(m)}e^{\alpha_m}\\= \sum_{i=1}^N w_i^{(m)}e^{-\alpha_m} + \sum_{y_i \neq k_m(x_i)} w_i^{(m)}(e^{\alpha_m}-e^{-\alpha_m})$$

可以看出和$$k_m$$相关的实际上只有上式的右半部分。显然，使得$$\sum_{y_i \neq k_m(x_i)} w_i^{(m)}$$最小的$$k_m$$，也会令E最小，这也就是我们选择加入的$$k_m$$。

对E求导，得：

$$\frac{d E}{d \alpha_m} = \frac{d (\sum_{y_i = k_m(x_i)} w_i^{(m)}e^{-\alpha_m} + \sum_{y_i \neq k_m(x_i)} w_i^{(m)}e^{\alpha_m}) }{d \alpha_m}$$

令导数为0，可得：

$$\alpha_m = \frac{1}{2}\ln\left(\frac{\sum_{y_i = k_m(x_i)} w_i^{(m)}}{\sum_{y_i \neq k_m(x_i)} w_i^{(m)}}\right)$$

令$$\epsilon_m = \sum_{y_i \neq k_m(x_i)} w_i^{(m)} / \sum_{i=1}^N w_i^{(m)}$$，则：

$$\alpha_m = \frac{1}{2}\ln\left( \frac{1 - \epsilon_m}{\epsilon_m}\right)$$

参考：

https://mp.weixin.qq.com/s/G06VDc6iTwmNGsH4IfSeJQ

Adaboost从原理到实现

https://mp.weixin.qq.com/s/PZ-1fkNvdJmv_8zLbvoW1g

Adaboost算法原理小结

https://mp.weixin.qq.com/s/KoOUgwXLOfJfOjWhbFX52Q

如果Boosting你懂，那Adaboost你懂么？

https://mp.weixin.qq.com/s/Joz2FpGgBY0tC8lpoFz8Mw

AdaBoost元算法如何提高分类性能——机器学习实战

https://mp.weixin.qq.com/s/MLEVUKse5usmKIWJF-yfOQ

通俗易懂讲解自适应提升算法AdaBoost


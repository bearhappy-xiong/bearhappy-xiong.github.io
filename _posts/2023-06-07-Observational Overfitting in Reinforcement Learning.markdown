---
title: TEST
date: 2023-06-06 14:43:00 +0800
categories: [Animal, Insect]
tags: [bee]   # TAG names should always be lowercase
toc: false
comments: false
math: true
mermaid: true  # 图表生成工具
---

![test-img](../assets/images/img-test.jpg){: width="700" height="400" }
_Image 1_



# Observational Overfitting in Reinforcement Learning


> 这篇是知乎`张楚珩`的[文章](https://zhuanlan.zhihu.com/p/449120642)复制粘贴的

会议/期刊: ICLR2019
标题: Observational Overfitting in Reinforcement Learning
作者信息: Google, MIT

**特色**: 强化学习的泛化性问题, 大家从各种不同的角度来研究，这里的角度是：***如果状态中包含很多 spurious features，会导致神经网络依赖这些不本质的特征来做判断，从而影响泛化性***.
这里用一些简单的例子和实验来说明不同的神经网络产生的隐式正则化(implicit regularization) 可以影响神经网络对于这些 spurious features 的依赖。

联系论文: [[IDAAC]], [[Local feature swapping for generalization in reinforcement learning]] 都研究了这个问题.

> [!question]+ 什么是隐式正则化(implicit regularization)?
> 神经网络是属于过参数化的模型（包含比所需参数数目更多的参数），所以有很多组参数都能最小化相应的误差，那么最后神经网络为什么学习到某些解，而不是另一些解呢？这是因为神经网络的优化过程包含了隐含的正则化，类似于**在所有能最小化相应误差的解中，返回范数最小的那个解**。但是这个**约束是由于神经网络优化自带形成的，而不是我们显式加上来的**，我们把它称作 implicit regularization。

## 一、背景
Observational overfitting 是啥意思？设想一种情况：Observation 中本来也包含足够我们做出正确决策的信息，但是由于其中还包含了一些*有关联，但是并不本质的特征（spurious features）*。**如果智能体主要依赖于 spurious features 来做决策，那么其泛化性就会比较差.** 
比如下面这个索尼克游戏(滚动的刺猬球)中，可以使用 [[saliency map]] 的技术观察智能体究竟是根据那些特征来做决策的；依赖度比较高的区域用红色阴影标注出来了。
我们可以看到，学习出来的智能体很依赖于时间、背景中的云，而不是依赖于场景中的障碍物和要控制的索尼克刺猬；这说明相应的智能体是根据非本质的一些特征来做决策的，因此泛化性可能会不好。
为什么自动学习的过程中会使得智能体依赖于这些非本质的特征呢？*依赖于时间来做决策其实相当于是学习到一个开环控制，即完全不管场景是啥，智能体学习到一个依赖于时间的策略，等于是闭着眼睛数着时间来操作*。
![[Pasted image 20220518144414.png]]

## 二、问题分析

![](https://pic4.zhimg.com/v2-fbd2d0f914f48bc7e3804be6d11b0587_r.jpg)

这里提出了一个 `(f,g)-sheme` 用于故意向 observation 里面掺入一些 spurious feature 的方法来测试强化学习算法的泛化性。
> (f,g)-sheme: $\phi_{\theta}(s)=h\left(f(s), g_{\theta}(s)\right)$ 
> 上面这个图里把`raw state`分解成$f$和$g_\theta$两个图像, 再进行连接`concatenation`.
```ad-note
这里是怎么做到分解的呢? 又为什么要先分开又合上呢?
```

![](https://pic1.zhimg.com/v2-80663dc35bfe32af59a002461d6f84d4_r.jpg)

这里引用了之前的一个 generalization upper bound 结论，说在这个设定下，如果函数族里面的函数都不受到 theta 的影响（泛化性好），那我们相应的 gap 就小。不过我把公式和他们说的没太对应上来，求大佬解释。

![](https://pic2.zhimg.com/v2-8dbdb1158d8681d802514d6e48a4c5f5_r.jpg)

下面文章通过在 LQR 场景下的理论分析，说明**最后策略的泛化性和训练时所见到的场景/任务的数量、注入 spurious features 的数目有关**，这些是由于 implicit regularization 导致的；实验上还说明泛化性还和策略网络的结构有关。不过仍然，前后的图、公式和文字，我还是没太对应上来。

![](https://pic2.zhimg.com/v2-55abd540ab41f783549163bc0e7d2c19_r.jpg)

![](https://pic2.zhimg.com/v2-80bdd599103a740d72ddae50fddb7f2d_r.jpg)

## 三、实验

最后，本文在 (f,g)-scheme 下跑了一些更复杂的实验。

![](https://pic4.zhimg.com/v2-968f776d0857104d1fd61e8b8c03f7d3_r.jpg)

![](https://pic2.zhimg.com/v2-44c3f775b11b0070dbdc05d64476702d_r.jpg)

![](https://pic1.zhimg.com/v2-d4e11140efae5137197d2308384d4c80_r.jpg)

![](https://pic4.zhimg.com/v2-70bfcd68f99774ae484db80c287ccbc3_r.jpg)

![](https://pic1.zhimg.com/v2-e35baaebe8d27e8866b869e971eb8460_r.jpg)

![](https://pic1.zhimg.com/v2-c097753c5624aab01f1d24924db941bc_r.jpg)

![](https://pic1.zhimg.com/v2-74d8cc00149b3480e96391caeb617d5c_r.jpg)

![](https://pic4.zhimg.com/v2-de18ada11442316b12a7448869263417_r.jpg)

## 四、总结

这篇文章还是感觉好乱（我没理解到= =），所以需要一个总结：

-   Observation 中的 spurious features 会使得策略泛化性降低；
-   模型是去拟合这些 spurious features 还是 invariant features 和 implicit regularization 有关，因此可以通过使用不同的神经网络结构来得到更好的泛化性能；
-   传统分析有监督深度学习泛化性的一些泛化误差上界在强化学习中不太好用（比如和神经网络结构有关的一些 smoothness bound 和可以用于直接监测泛化性能的 margin distribution）。


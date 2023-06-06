---
title: The Primacy Bias in Deep Reinforcement Learning
date: 2023-06-06 15:00:00 +0800
categories: [RL, 网络可塑性]
tags: []
toc: true
comments: true
math: true
mermaid: true  # 图表生成工具
---



## 1. 标题+作者


会议/期刊: _Proceedings of the 39 th International Conference on Machine Learning, Baltimore, Maryland, USA, PMLR 162, 2022._

作者在Mila上写的一个博客: [The Primacy Bias in Deep Reinforcement Learning](https://mila.quebec/en/article/the-primacy-bias-in-deep-reinforcement-learning/)

生物领域:
[Forgetting Enhances Episodic Control With Structured Memories遗忘增强了结构化记忆的情景控制](https://www.frontiersin.org/articles/10.3389/fncom.2022.757244/full)


## 2. 摘要

> 小明聪明、勤奋、挑剔、冲动和嫉妒
> 小刚嫉妒、冲动、挑剔、勤奋和聪明

首先指出DRL中的一个常见缺陷: `primacy bias首因偏差`
==agent总是倾向于依赖早期交互而忽略后来遇到的(可能更加)有用的证据.==
然后给出解决方案: ==定期重置agent的一部分==


*“Your assumptions are your windows on the world. Scrub them off every once in a while, or the light won’t come in.” 
										    			–Isaac Asimov
“你的假设是你了解世界的窗口。每隔一段时间就把它们擦掉，否则光线不会进来。” 
																——艾萨克·阿西莫夫



## 3. 结论
- 确定了DRL中的首因偏差--AI agent过度适应早期经验的破坏性趋势.
- 展示了与这种形式的过拟合相关的危险.
- 提出了一个基于重设agent的一部分的简单解决方案.
- 实验论证有效且普适.
---

title: Think More
多年来, 强化学习研究的问题都是先在表格case中建立一个合理的算法, 然后再使用神经网络来参数化agent的各个部分. 神经网络参数化的过程总是被当做技术细节. 但是, 本文提出的首因偏差就是一种在参数化过程中引入的问题. 在表格case中无论如何研究也不能解决这一问题.
像定期重置这样简单的技术就可以显著提高性能, 表明对于DRL还有很多不同于RL的研究领域.



## 4. 导言
---
1. 证明了DRL中首要偏差的存在;
2. 揭示这种现象的原因; 展示DRL算法如何放大了这个影响;
3. 提出通过定期重置部分agent来缓解首因偏差.
4. 将重置应用于强baseline算法, 实验证明了性能的定性和定量改进.
---
### 人类的首因偏差例子
Bob练习吉他一段时间, 但总是弹得不好, Alice给他展示了一种更好听的演奏方式. 但是, 在随后的排练中, 即使Bob已经知道了弹得更好的Alice的方法, 但他的潜意识还是会按自己之前的方式去弹. Bob就经历了 **首因偏差** 的问题. 这是人类存在的一种认知偏见--第一次的经验会对以后的学习和行为产生长久的影响(==这一点非常不同于我们已知的神经网络的遗忘特性==).

在这种认知偏见的影响下, 即使Bob已经收集了足够的新数据来改善他在吉他演奏中的表现, 但是, ***由于长时间的训练和糟糕的技术给他提供了启蒙, 他无法利用他的新经验来学习.*** 这导致了恶性循环➡️***由于Bob在面对新的数据时没能提高他的吉他技巧, 他就无法通过弹奏更具挑战的吉他段落来收集更多有用的数据, 这就导致了整体学习水平的下降.***
### IN DRL
DRL算法很容易受到类似偏见的影响. DRL中的首因偏差--本文称为`a tendency to overfit early interactions with the environment`. 早期与环境互动的过拟合倾向. 
这种倾向会阻碍agent通过之后的经验来改善行为的学习过程. 
恶性循环: 性能差的agent收集质量差的数据➡️质量差的数据放大了从过拟合agent中恢复的难度.

同时, DRL中的一些组件也放大了首因偏差的影响--经验回放机制允许样本重用, 但是, agent暴露在初始样本的次数多于最近的样本. 而DRL算法经常额外使用高重放率, 在相同数据上更新agent更多次. _这虽然可以提高agent的性能, 但是会加剧早期交互的影响_.

作者认为, 受到这种首因偏差影响的agent应该在继续学习过程之前, 忘记一部分solution, 因为这些solution来自于对早期经验的过拟合.
这个忘记的过程, 作者叫做`resetting mechanism`(重置机制), 让agent忘记部分知识. 具体来说, ***📍定期重新初始化agent的神经网络的最后几层, 同时保持buffer中的经验📍.*** 📈这种重置机制能***使训练机制具有更高的replay radio和更长的n-step 目标***--如果没有重置机制的话, 更高的replay radio和更长的n-step 目标会导致agent过拟合📈.

这个机制不引入额外计算成本, 且只有两个问题需要抉择:
1️⃣重置神经网络的哪些部分
2️⃣重置的频率


## 5. 相关工作
包含replay buffer的算法中, 数据重采样的次数由`replay radio`控制, 这对算法性能起着至关重要的作用. 
`replay radio`太低➡️无法充分利用数据➡️采样效率低下
`replay radio`太高➡️过度利用数据➡agent过拟合

- 1993年, Anderson et al.提出基于方差准则重置Q网络的 **单个神经元** 并观察到了更快的收敛.
- 非均匀抽样的形式，包括重新加权最近的样本(`Wang, C., Wu, Y., Vuong, Q., and Ross, K. Striving for simplicity and performance in off-policy drl: Output normalization and non-uniform sampling. In International Conference on Machine Learning, pp. 10070–10080. PMLR, 2020.`)和优先经验回放(`PER`)可以被视为减轻首因偏差的一种方式. 
- ITER在监督案例中证明从预训练的网络中学习, 可能比从头开始学习非平稳数据集更糟糕. 并提出了一种方案ITER, 在on-policy和buffer-free的设置中完全重置agent网络，并将上一代的蒸馏作为知识转移机制.
> ITER和reset的不同之处是ITER不保留buffer.

#### 遗忘机制
与`灾难性遗忘`现象相比, 很多论文指出了`灾难性记忆`, 类似首因偏差.
Achille et al. (2018)观察到 **学习中存在一个对网络有决定性影响的关键阶段**. 
Erhan et al. (2010)观察到 **经过训练的网络对第一个数据点的敏感性更高**.
***在监督学习里已经发现了早期样本的影响, 这个问题在DRL中对初始经验过度拟合的后果更加严重, 因为agent要自己收集数据来学习, 即数据收集过程处于训练循环中*** 

#### 重置网络
Taha et al. (2021)从进化学习的角度研究了重置子网络的过程, 并表明CV任务的性能有所提高.
Zhou et al. (2022)表明一定程度的遗忘会改善泛化.
Zhang et al. (2019)证明重置不同的层对网络的性能有不同的影响.
[Fortuitous Forgetting in Connectionist Networks](https://openreview.net/forum?id=ei3SY1_zYsE)
Alabdulmohsin et al. (2021)证明重置增加了训练实例的边际, 并能诱导收敛到更平坦的最小值.

#### 认知科学领域
primacy bias(primacy effect)是人类学习中的一种认知偏差.
给定一系列事实, 人类通常会根据第一个事实形成概括. 这会导致有害的偏见.
这可能就是自然大脑中遗忘机制的作用--一定程度的遗忘有利于决策的制定.

[Why do we only remember the first things on our grocery list?](https://thedecisionlab.com/biases/primacy-effect) 
[Primacy and Recency Biases: Be First or Last](https://leversofpersuasion.medium.com/primacy-and-recency-biases-be-first-or-last-cff4c7f64b7e)
比如第一印象决定了对一个人的整体印象.
primary bias 和 recency bias 有关. 后者叫近因效应, 表示我们能更好的回忆起最新信息. 比如, 需要投票的比赛中最后几名的票数总是很高(不考虑水平问题)



## 6. The Primacy Bias

> a tendency to overfit initial experiences that damages the rest of the learning process.
> 过度拟合初始经验的倾向会损害学习过程的其余部分。

### 在第一次交互时对agent进行过度训练可能会致命地破坏此后的学习过程
agent对早期数据的依赖程度是决定首要偏差对学习过程影响程度的关键因素. 但是, 还需要通过充分利用初始经验来加快训练. 所以这里存在一个trade-off⚖️⚖️.

##### 实验设置: 对`a single batch`的早期数据的过拟合是否足以完全破坏agent的学习过程?
<kbd>baseline</kbd>: SAC--默认超参, 即每一步对策略和价值函数进行一次更新.
<kbd>env</kbd>: quadruped-run (四足跑)
<kbd>实验方法</kbd>:
`heavy priming 重启动`--收集 $100$ 个数据点后, 使用生成的replay buffer更新agent $10^5$ 次, 之后恢复标准训练.
<kbd>实验结果</kbd>:
即使恢复标准训练, 在收集和训练近一百万个transitions后, ***具有`heavy priming`的agent也无法从初始过拟合中恢复***.
下图表示, 在前100个transitions中, 有或没有`heavy priming`的无折扣回报.
![[Pasted image 20221118194638.png]]
这个实验非常显然的传达出一个信息: 对早期经验的过拟合, 会非常严重的损害学习过程的其余部分. 

> 虽然不会真的有哪个算法的实现会在100个数据点上训练$10^5$步, 但是之后的实验也会表明, 即使每个步骤的更新数量相对较少, 也会导致类似的问题.

---
### 受首因偏差影响的agent收集的数据***足以用于学习***，但是这个agent由于其累积的过拟合而无法利用它
<kbd>baseline</kbd>: `SAC`--默认超参, 即每一步对策略和价值函数进行一次更新.
<kbd>env</kbd>: `quadruped-run` (四足跑)
<kbd>实验方法</kbd>:
在MDP中训练一个每步*更新9次*的SAC, 由于首要偏差, 这个agent表现会不好. 然后, 我们从头开始初始化同一个agent, 但是使用前一个SAC收集的数据作为初始replay buffer.
<kbd>实验结果</kbd>:
第二个agent得到的汇报迅速提高, 接近最佳表现.
下图中, `SAC-failing`是一个标准agent, `SAC with failing agent buffer`是使用第一个agent的replay buffer初始化的agent. 


这个实验说明, 首因偏差本身是可以收集到适当的数据的, 他只是早早的过拟合了, 导致不能从这些数据中吸取教训了. 换句话说, ***存储在replay buffer立的数据本身是足以获得更好的性能的, 但是过度拟合的agent缺乏将其提炼为更好的策略的能力.*** 而 **随机初始化的神经网络** 不受首要偏差的影响, 因此能够充分利用收集到的经验.

## 7.首要偏差的解决--resetting

>> 给定agent的神经网络, 定期重新初始化这个网络最后几层的参数, 同时保留replay buffer

<kbd>baselines</kbd>: `SPR` for Atari, `SAC` for DMC-states, `DrQ` for DMC-pixels
<kbd>问题域</kbd>: 离散控制--Atari 100k; 连续控制--DM Control Suite;

不同算法的重置策略稍有不同. 
对`SPR`: 
1️⃣重置神经网络的哪些部分➡️仅重置5层Q网络的最后一个线性层
2️⃣重置的频率➡️每 $2×10^4$ 步
对`SAC`:
1️⃣重置神经网络的哪些部分➡️**完全重置**agent的网络
2️⃣重置的频率➡️每 $2×10^5$ 步
3️⃣需要对 target network 和 两个 Q network 都进行重置
对`DrQ`:
1️⃣重置神经网络的哪些部分➡️重置策略网络和价值网络7层中的最后3层
2️⃣重置的频率➡️每 $2×10^5$ 步
3️⃣需要对 target network 和 两个 Q network 都进行重置

> 对其他部件的重置在之后的ablation中

***重置之后, 不对新参数进行任何形式的预训练, 直接返回标准训练. 以保持环境交互和agent更新之间的完整循环.***  

本文对算法评估使用的是跨task的`四分位数平均值`(IQM)
来自[[Deep reinforcement learning at the edge of the statistical precipice.]]
关于`IQM`: 2021-NeurIPS-deep-reinforcement-learning-at-the-edge-of-the-statistical-precipice
![[Pasted image 20221119192437.png|500]]


### 重置可以持续提高性能
下面两个表报告了`SPR`,`SAC`,`DrQ`的结果.

上面两个表中, 报告了不同raplay radio和 `n`值的最佳结果.
实验数据可以看出, reset能减轻首要偏差. 
并且可以用于:
1. 离散或连续动作空间.
2. 状态输入或图像输入.
3. 各种replay buffer配置.
4. 优先经验回放或随机抽样
5. 各种神经网络深度

### 带reset的agent的学习动态
乍一看, 重置可能是一种很drastic的措施. 因为agent每次都必须从头开始学习随机初始化的那些层的参数. 但本节展示了在各种情况下, 这种策略仍然能带来性能的提高和快速学习. 
下图是state-based SAC在DMC中的四个代表环境上的示例.
> 由于SAC只使用三层策略和价值函数的架构, 所以本文选择重置整个网络.

1. `cheetah-run`环境中首因偏差不是问题, 所以重置会导致学习曲线中的一些性能下降的峰值.但是**agent能够在短短的几千步内返回到以前的状态**.
2. `hopper-hop`和`humanoid-run`环境中算法容易受到首因偏差的影响, 所以重置不仅使性能恢复到以前的水平, 而且还使agent超越了它.
##### 为什么RLagent每次重置之后能恢复的如此快速?
关键在于保留了(跨迭代的)replay buffer.
**定期清空replay buffer对性能是非常不利的**. 
作者猜测, 可以用model-based的观点可以提供一个解释：由于replay buffer可以被视为非参数化的世界模型，在重置后，agent忘记了过去学到的行为，但是保留了buffer中的模型作为其知识核心。
从神经网络训练的角度, 也有人观察到学习的本质是恢复正确的表征.也就是说, 使用最新的经验池, 重新学习一个策略(执行器actuator)可能相对更简单.
之前, 我们提到首因偏差导致了一个恶性循环: 性能差的agent收集质量差的数据➡️质量差的数据放大了从过拟合agent中恢复的难度.
这里, 作者认为网络重置引发了一个良性循环: 重置后的agent摆脱了过去训练迭代产生的负面priming➡️可以更好地利用最新收集的数据, 从而提高性能➡️为未来的更新产生更高质量的数据提供可能性.
***如果首因偏差是一种特殊形式的过拟合, 那么重置就可以看成是一种量身定制的正则化形式.专门用于解决首因偏差.*** 
下图可以看出, 重置的特殊性质能让agent克服首因偏差, 这是dropout和L2正则化做不到的.


### 重置的成功背后的因素
> 什么条件下重置是最有用的?
###### Raplay radio
前面已经证明了, **对早期数据的依赖程度** 是首因偏差影响强度的关键决定因素. 
所以, 很自然的想到, 重置的影响很大程度上取决于`replay radio`, 即每个环境步的梯度更新次数.
下图说明, ==replay radio越大, 重置的影响就越大==. 
- 在每步更新四次的情况下, 将`SPR`的性能提高了40%.
- 在replay radio高达32的情况下, `SAC`实现了最高的性能. 也就是说`SAC`的性能提高了100%
- 即使把`SAC`推到128和256的极端replay radio下, 在大多数环境中几乎不可能进行有效的学习了, 重置仍然能使agent获得合理的性能.
***也就是说, 重置允许replay radio这个超参进行不那么仔细的调整, 并且通过对每个数据点进行更多的更新来提高采样效率, 而不会受到首因偏差的严重影响.***

###### n-step targets
$n$ 控制的是TD learning中bias-variance的trade-off.
$n$ 越大, 偏差越小, 方差越大.
作者通过实验证明, 从高方差目标学习的agent会很容易对初始数据过拟合, ==n越大, 重置的影响就越大.==

###### TD failure modes-sparse & divergence
TD learning的致命三元组: **自举, 函数逼近, off policy**
致命三元组可能导致学习不稳定.
在稀疏奖励环境中, critic可能会收敛到一个退化解❓. 因为他是自举的, 在自己的输出上进行引导. 所以极少量的非零奖励数据不足以从这种崩溃中逃逸. 
下图是DrQ在`cartpole-swingup sparse`环境中的表现. 可以看出, *崩溃的DrQ不会再恢复了*. 但是, 经过对replay buffer的*人工检查*, 我们发现agent在大约2%的轨迹中达到了目标状态. 
**减轻首因偏差解决的是优化问题, 而不是探索问题.** 
![[Pasted image 20221121210341.png]]

---
如果在TD learning中发生了 divergence(分歧), agent基本上无法通过标准优化恢复. 预测值在数十万步后无法衰减到正常幅度.
> divergence指的应该就是高估低估

下图中, 添加reset为agent提供第二次机会找到稳定解决方案来解决此问题.


###### What and how to reset❓
再复制一遍结论, 消融测试在附录B中:
对`SPR`: 
1️⃣重置神经网络的哪些部分➡️仅重置5层Q网络的最后一个线性层
2️⃣重置的频率➡️每 $2×10^4$ 步
对`SAC`:
1️⃣重置神经网络的哪些部分➡️**完全重置**agent的网络
2️⃣重置的频率➡️每 $2×10^5$ 步
3️⃣需要对 target network 和 两个 Q network 都进行重置
对`DrQ`:
1️⃣重置神经网络的哪些部分➡️重置策略网络和价值网络7层中的最后3层
2️⃣重置的频率➡️每 $2×10^5$ 步
3️⃣需要对 target network 和 两个 Q network 都进行重置

造成上述差异的原因是什么❓
作者推测, 差异在于每个领域所需的**表示学习程度**, Atari中的大量知识包含在agent的表示过程中, **在DMC中学习特征可能特别容易**. 在DrQ中, 当同时重置actor和critic时, 事实证明, 重置critic比actor更加重要. 这可能是因为DrQ编码器仅从critic loss中学习.

关于重置频率，最佳选择应*取决于算法恢复的速度以及它受首要偏差影响的程度*；我们发现有时即使是一次重置也会提高基线代理的性能。

***本文在重置网络时会使用新的随机种子来对新网络的权重进行采样***. 但同时作者提到, 即使使用相同的种子进行重置, 也可以减轻首因偏差. 因为在`Bjorck, J., Gomes, C. P., and Weinberger, K. Q. Is high variance unavoidable in RL? a case study in continuous control. In International Conference on Learning Representations, 2022`中提到, DRL中的病态不是由于初始化问题.

###### whether to reset the state of the optimizer❓
作者发现重置优化器对训练几乎没有影响. 因为矩估计更新很快.
- [x] 修改官方代码, 改为不重置优化器 🔼 ✅ 2023-03-21

---
简而言之，实验结果表明，在不同的环境和算法中，重置可以提高预期收益。它们作为一种正则化的形式，从而防止了对早期数据的过度拟合，开启了新的超参数配置，可能具有更高的性能和样本效率，并解决了深度RL中出现的优化挑战。虽然通过更彻底的超参数搜索和额外的修改，有可能进一步提高基线的性能，但令人振奋的是，所提出的简单重置方案带来的好处可与以前的算法进步相媲美。

## 8. 附录内容
### 消融实验
##### 1.Replay buffer
<kbd>实验设置</kbd>: 通过定期重置replay buffer来测试它在DrQ中的重要性.
<kbd>实验结果</kbd>: 结果证明保留buffer是必须的. 重置它相当于从头开始学习.
<kbd>结论</kbd>: 这些结果表明，对于防止首要偏见的负面影响并同时能够从重置中恢复，知识保留比行为保留重要得多。

##### 2.Initialization
<kbd>实验设置</kbd>: 为了理解agent对首因偏差的敏感性是否只是神经内网络中的某一层的不幸运的初始化的结果, 我们把DrQ的reset操作改为使用与训练一开始相同的种子进行初始化.
<kbd>实验结果</kbd>: 这个变体的性能与原始版本几乎相同.
<kbd>结论</kbd>: 通过重置解决的问题不是病态初始化问题, 而是从不断增长的相互作用数据集中学习时发生的特殊相互作用导致的问题.

##### 3.Optimizer state
<kbd>实验设置</kbd>: 为了判断重置神经网络参数时同时重置优化器几乎没有效果, 在DrQ中进行了实验.
<kbd>结论</kbd>: 同时重置优化器和参数的性能与只重置参数差不多, 但是只重置优化器是不行的.

##### 4.Reset depth
关于重置深度的消融

##### 5.Which networks to reset
`DrQ`: 同时重置所有神经网络通常是提高骨干算法性能的最稳健技术, 而重置 critic 对性能的影响最大

##### 6.Number of resets
<kbd>实验设置</kbd>: 为了判断有限次数的重置甚至一次重置, 是否足以克服初始经验过度拟合的影响, 使用`DrQ`测试了这个假设
<kbd>实验结果</kbd>: 尽管第一次重置对减轻首因偏差的贡献最大, 但是并不足以达到标准连续重置策略的相同性能.
<kbd>结论</kbd>: 在训练过程中应该进行3-10次重置.

##### 7.Other regularizers
重置可以看作是正则化的一种形式，因为它们隐含地将最终解决方案限制为与初始参数相差不远.
- [x] 尝试不同的初始化方法对结果的影响. ✅ 2023-03-21
这部分证明了其他正则化方法不能解决首因偏差问题.


## 9.评论
reset可以从两个角度解释:
1. <kbd>监督学习视角</kbd>: reset相当于周期性的将模型参数从局部极小值点拉出来，通过不断地随机，保证不会陷入局部极小值点
	> reset实质上反映了神经网络自身可能因为进入了较差的参数范围，即便给什么数据都不能更好，很难跳出坑了（或者说稳定在了一个比较差的次优点）
	> 重新给他一次再次为人的机会
2. <kbd>强化学习视角</kbd>: reset相当于让一个新的网络去拟合现在的新数据分布. 随着训练的进行, 策略网络和环境本身的分布都在改变, 而初始的agent还受着初始数据分布的影响, 使用新的网络来拟合现在的新分布, 能有比较好的拟合效果. 而一般情况下, 新分布比初始分布更加合理, 所以agent性能可能得到提升.
	> ITER研究的是这个角度, 所以可以进行ITER与reset的对比实验

本文的一个缺陷是没有给出这种重置机制的理论推导和收敛性证明.
同时, 从regret最小化的角度来看, 重置必然会导致的性能短暂崩溃是不可取的.
可能的补救措施包括在每次重置后进行一段时间的 offline post-training(与pre-training相对), 或者在每次重置后的一段时间内从重置前后的agent之间的插值中抽样操作.
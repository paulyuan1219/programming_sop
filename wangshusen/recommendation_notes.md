# [别人做的笔记](https://www.yuque.com/yuejiangliu/recommended-system-in-the-industry/resource)

# 工业界的推荐系统



## 1. **概要** [[讲义](https://github.com/wangshusen/RecommenderSystem/blob/main/Notes/01_Basics.pdf)]

### 小红书的推荐系统

##### 转化流程

- 曝光
  - 点击
    - 滑动到底
      - 评论
    - 点赞
    - 收藏
    - 转发

##### 消费指标

- 点击率 = 点击次数 / 曝光次数
- 点赞率 = 点赞次数 / 点击次数
- 收藏率 = 收藏次数 / 点击次数
- 转发率 = 转发次数 / 点击次数
- 阅读完成率 = 滑动到底次数 / 点击次数 × 𝑓(笔记长度)


#### 北极星指标

- ⽤户规模：
  - ⽇活⽤户数（DAU）、⽉活⽤户数（MAU）。
- 消费：
  - ⼈均使⽤推荐的时长、⼈均阅读笔记的数量。
- 发布：
  - 发布渗透率、⼈均发布量。

##### 实验流程
1. 离线实验  
  收集历史数据，在历史数据上做训练、测试。算法没有部署到产品中，没有跟⽤户交互。
2. 小流量AB测试  
  把算法部署到实际产品中，⽤户实际跟算法做交互。
3. 全流量上线  

### 推荐系统的链路

- 召回 billion to thousands
  - N条召回通道：协同过滤、双塔模型、关注的作者、等等
  - 每一条召回通道返回几百
  - ⽤多条通道，取回⼏千篇笔记。
- 粗排 thousands to hundreds
  - 用户特征，物品特征，统计特征，全部输入神经网络，预测点击率，点赞率等等，最后排序
  - ⽤⼩规模神经⽹络，给⼏千篇笔记打分，选出分数最⾼的⼏百篇。
- 精排 thousands to hundreds
  - ⽤⼤规模神经⽹络，给⼏百篇笔记打分
- 重排 hundreds to tens
  - 做多样性抽样（⽐如MMR、DPP），从⼏百篇中选出⼏⼗篇。
  - ⽤规则打散相似笔记。
  - 插⼊广告、运营推广内容，根据⽣态要求调整排序。


这节课以小红书为例讲解推荐系统的链路。链路包括召回、粗排、精排、重排。
- 召回（retrieval）：快速从海量数据中取回几千个用户可能感兴趣的物品。
- 粗排：用小规模的模型的神经网络给召回的物品打分，然后做截断，选出分数最高的几百个物品。
- 精排：用大规模神经网络给粗排选中的几百个物品打分，可以做截断，也可以不做截断。
- 重排：对精排结果做多样性抽样，得到几十个物品，然后用规则调整物品的排序。

#### QA

1. 下拉刷新是怎么实现的，每次刷新都会走一遍推荐链路吗？那这样推荐结果不会重复吗？短时间内模型参数应该没变，模型是输入应该也不变吧

> 对，重做一遍. 不会重复的，召回加了bloom filter

2. 王老师，统计特征能不能给几个例子？谢谢！


> 物品的点击率、用户的点击率、物品分男女用户两桶的点击率、用户按物品类目分桶的点击率


3. 王老师，请教一下，架构层面是怎么实现的，模型训练和在线部署用的什么框架？

> embedding太大了，所以用自研的，基于TF


### AB测试
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/01_Basics_03.pdf)]
    [[YouTube](https://youtu.be/7Jz3VY8SCR4)]
    [[B站](https://www.bilibili.com/video/BV1J44y1o7gf)].

#### 互斥 vs 正交
- 如果所有实验都正交，则可以同时做无数组实验。
- 同类的策略（例如精排模型的两种结构）天然互斥，对于⼀个⽤户，只能⽤其中⼀种。
- 同类的策略（例如添加两条召回通道）效果会相互增强（1+1>2）或相互抵消（1+1<2）。互斥可以避免同类策略相互⼲扰。
- 不同类型的策略（例如添加召回通道、优化粗排模型）通常不会相互⼲扰（1+1=2），可以作为正交的两层。

#### 总结
- 分层实验：同层互斥（不允许两个实验同时影响⼀位⽤户）、不同层正交（实验有重叠的⽤户）。
- Holdout：保留 10% 的⽤户，完全不受实验影响，可以考察整个部门对业务指标的贡献。
- 实验推全：新建⼀个推全层，与其他层正交。
- 反转实验：在新的推全层上，保留⼀个⼩的反转桶，使⽤旧策略。长期观测新旧策略的 diff。

#### QA AB测试

1. 老师，分层实验的意思并不是必须按照推荐系统的分层来分层实验啊。Layer是一个虚拟的概念，并不对应推荐系统中具体的层次关系，其含义等同于一个实验。这一点在Facebook开发的A/B实验平台PlanOut中体现得非常清楚。在PlanOut中，根本没有Layer的概念，只有Experiment，流量在进入一个新的Experiment前就会被重新打散。比如两个实验：在召回里是否使用X路召回；在召回里是否使用Y路召回，这两个实验可以正交的，虽然都在召回层。
> 多谢多谢～这个我知道的哈。分层还是比较tricky的，要是有两个召回，总体quota分配变了，两个实验会互斥的。还有，正交到底会怎么样互相干扰，其实我不太确定。在前司发现分层被滥用，很多人每个实验开一层，利用流量没调平，随便做点啥就报实验显著 发launch。

2. 谢谢分享。我也不太明白为什么推全要开新层，新层的意思是比原来多了一步吗？
> 旧实验已经让用户体验发生变化，需要打散用户重新做实验。

3. 看完整个课程后，发现王老师好像没有涉及离线的评测？

> 召回离线测评没有多大价值。排序离线就是看auc和gauc

4. 
> 灼眼的零时迷子

按照论文 Tang, Diane, et al. "Overlapping experiment infrastructure: More, better, faster experimentation." Proceedings of the 16th ACM SIGKDD international conference on Knowledge discovery and data mining. 2010. 中的解释，对王老师提到的“新层”的定义是发布层（launch layer）。对于发布层，有以下性质：
1. 从用户流量的角度，发布层包含全部流量。
2. 发布层不是瞬间建成的，是逐渐扩展到所有流量的。
3. 从实验参数的角度，发布层是参数的一个独立划分。一个实验参数可以同时出现出现在至多一个发布层和至多一个非发布层（实验层）中。
4. 如王老师视频（12:03）中所示，以重排层的一个实验为例。该实验（模型）通过了测试，需要推全。该模型参数会保存在新建的发布层中，作为默认模型参数的一个代替。当一个流量发生，对该流量的处理要用到某一个参数，会按照如下优先级去选择：非发布层（实验层）中的参数值>发布层中的参数值>默认参数值，因此能做到不干扰现有实验的情况下逐步向所有用户推出更改，并以标准化方式跟踪这些推出。
5. 当发布层建立完全，即通过测试的参数成功推全到所有流量后，发布层的参数会覆写默认参数，发布层会被删除。
6. 原文中还提到发布层的一个优点，因为发布层是全流量的，在其上进行的实验规模通常较大，能够更好的观测实验之间的交互。

总结一下，新建的发布层的概念不是指某一流量要重复经过两个模型，而是该模型的参数需要按照 非发布层（实验层）中的参数值>发布层中的参数值>默认参数值 这一优先级进行选择。使用发布层可以逐渐推全策略，能做到看起来“无缝”的服务升级，同时也能跟踪策略实际情况下的状态，若出现意外可以及时停止、补救。

最后叠个甲，以上是参考论文获得的结论，纯属个人理解，如果有业内前辈发现问题麻烦纠正一下[脱单doge]~
2023-05-31 19:01

17

回复

UP主觉得很赞

R崽哗啦啦

 个人想法：发布层的参数应该是静态并且全覆盖的，也就是在使用发布层（即非反转桶）的路径上一定使用新发布的预推全参数而不可能使用默认参数，而反转桶一定使用默认参数。非发布层（实验层）的参数是基于上述初始参数进一步做优化的。也就是【非反转桶：实验参数>发布参数】【反转桶：实验参数>默认参数】不知道理解对不对。


## 2. **召回**
    
    * 基于物品的协同过滤（ItemCF）
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_01.pdf)]
    [[YouTube](https://youtu.be/QtmunNLeDvo)]
    [[B站](https://www.bilibili.com/video/BV1mA4y1Q7RN)].
    
    * Swing模型
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_02.pdf)]
    [[YouTube](https://youtu.be/DUUMNTDuJ3Q)]
    [[B站](https://www.bilibili.com/video/BV1DA4y1Q7rB)].
    
    * 基于用户的协同过滤（UserCF）
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_03.pdf)]
    [[YouTube](https://youtu.be/7O9zFMNdrZ8)]
    [[B站](https://www.bilibili.com/video/BV1HY4y1Y7P1)].
    
    * 离散特征处理
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_04.pdf)]
    [[YouTube](https://youtu.be/Wiqfn0BIcJs)]
    [[B站](https://www.bilibili.com/video/BV1pS4y1a7QT)].
    
    * 矩阵补充
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_05.pdf)]
    [[YouTube](https://youtu.be/phpIjr8_C7g)]
    [[B站](https://www.bilibili.com/video/BV1b34y1e7En)].
    
    * 双塔模型：模型和训练
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_06.pdf)]
    [[YouTube](https://youtu.be/2Mc10LZ-DB0)]
    [[B站](https://www.bilibili.com/video/BV1YA4y1D75Q)].
    
    * 双塔模型：正负样本
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_07.pdf)]
    [[YouTube](https://youtu.be/KOpl2cJyKOg)]
    [[B站](https://www.bilibili.com/video/BV133411T7ue)].
    
    * 双塔模型：线上服务
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_08.pdf)]
    [[YouTube](https://youtu.be/3qOvHfW1A-8)]
    [[B站](https://www.bilibili.com/video/BV1KY4y1h73Y)].

    * 双塔模型+自监督学习
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_09.pdf)]
    [[YouTube](https://youtu.be/Ra3MVhneR9E)]
    [[B站](https://www.bilibili.com/video/BV1v24y1B7JH)].
    
    * Deep Retrieval 召回
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_10.pdf)]
    [[YouTube](https://youtu.be/BYtzZ48hRFM)]
    [[B站](https://www.bilibili.com/video/BV1Fu4y1b7PL)].
    
    * 其它召回通道
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_11.pdf)]
    [[YouTube](https://youtu.be/7CKBjx7bw7k)]
    [[B站](https://www.bilibili.com/video/BV1m5411R7nd)].

    * 曝光过滤
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/02_Retrieval_12.pdf)]
    [[YouTube](https://youtu.be/cM76ZbkqrFU)]
    [[B站](https://www.bilibili.com/video/BV1sa4y137LF)]



## 3. **排序**

    * 多目标排序模型
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/03_Rank_01.pdf)]
    [[YouTube](https://youtu.be/kY4W46MQqsg)]
    [[B站](https://www.bilibili.com/video/BV19t4y1p7UM)].
    
    * Multi-gate Mixture-of-Experts (MMoE)
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/03_Rank_02.pdf)]
    [[YouTube](https://youtu.be/JIEwaPARjfk)]
    [[B站](https://www.bilibili.com/video/BV14Y411M74v)].
    
    * 预估分数融合
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/03_Rank_03.pdf)]
    [[YouTube](https://youtu.be/D2iqM2puJ2I)]
    [[B站](https://www.bilibili.com/video/BV1YT411578u)].
    
    * 播放时长建模
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/03_Rank_04.pdf)]
    [[YouTube](https://youtu.be/SiyvcJzr2bg)]
    [[B站](https://www.bilibili.com/video/BV1394y1277M)].
    
    * 推荐系统的特征
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/03_Rank_05.pdf)]
    [[YouTube](https://youtu.be/J7N4xjqg0rk)]
    [[B站](https://www.bilibili.com/video/BV1gN4y157TM)].
    
    * 粗排三塔模型
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/03_Rank_06.pdf)]
    [[YouTube](https://youtu.be/0CvouPv47SA)]
    [[B站](https://www.bilibili.com/video/BV1Dd4y1m7KT)].


    
## 4. **交叉结构**
    
    * Factorized Machine (FM)
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/04_Cross_01.pdf)]
    [[YouTube](https://youtu.be/exVPXVFPMDk)]
    [[B站](https://www.bilibili.com/video/BV15V4y1x7Ht)].
    
    * Deep & Cross Network (深度交叉网络)
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/04_Cross_02.pdf)]
    [[YouTube](https://youtu.be/yNeRO5m63JQ)]
    [[B站](https://www.bilibili.com/video/BV1LP411L7Z2)].
    
    * LHUC
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/04_Cross_03.pdf)]
    [[YouTube](https://youtu.be/TxIedW94hu0)]
    [[B站](https://www.bilibili.com/video/BV1jU4y1z7Tc)].
    
    * SENet & FiBiNET
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/04_Cross_04.pdf)]
    [[YouTube](https://youtu.be/nF37qtNvw1E)]
    [[B站](https://www.bilibili.com/video/BV1SY4y1M7bD)].




## 5. **用户行为序列建模** 

    * 用户行为序列特征
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/05_LastN_01.pdf)]
    [[YouTube](https://youtu.be/Stbc9goPKXQ)]
    [[B站](https://www.bilibili.com/video/BV1GG4y1B7Yh)].
    
    * DIN 模型
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/05_LastN_02.pdf)]
    [[YouTube](https://youtu.be/0hPep80Oy6k)]
    [[B站](https://www.bilibili.com/video/BV1bT411T7u4)].
        
    * SIM 模型
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/05_LastN_03.pdf)]
    [[YouTube](https://youtu.be/_4J9aF5KR84)]
    [[B站](https://www.bilibili.com/video/BV1Ze4y1B7JL)].



## 6. **多样性** [[讲义](https://github.com/wangshusen/RecommenderSystem/blob/main/Notes/06_Rerank.pdf)]
    
    * 多样性的度量
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/06_Rerank_01.pdf)]
    [[YouTube](https://youtu.be/uCIlk7N1dvk)]
    [[B站](https://www.bilibili.com/video/BV1ne4y1v7mC)].
    
    * MMR 算法
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/06_Rerank_02.pdf)]
    [[YouTube](https://youtu.be/tCa4yackga0)]
    [[B站](https://www.bilibili.com/video/BV1dV4y1V7Kg)].
    
    * 规则约束
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/06_Rerank_03.pdf)]
    [[YouTube](https://youtu.be/84kK1h0FS3Y)]
    [[B站](https://www.bilibili.com/video/BV1om4y1F7y5)].
    
    * DPP：数学基础
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/06_Rerank_04.pdf)]
    [[YouTube](https://youtu.be/HjpJeUSekKs)]
    [[B站](https://www.bilibili.com/video/BV1re411F7cp)].
    
    * DPP：多样性算法
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/06_Rerank_05.pdf)]
    [[YouTube](https://youtu.be/wi8xVHiZZr4)]
    [[B站](https://www.bilibili.com/video/BV1Md4y1c7uB)].




## 7. **物品冷启动** [[讲义](https://github.com/wangshusen/RecommenderSystem/blob/main/Notes/07_ColdStart.pdf)]

	 * 评价指标
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/07_ColdStart_01.pdf)]
    [[YouTube](https://youtu.be/EEQ4qwjezRc)]
    [[B站](https://www.bilibili.com/video/BV1eZ4y1a7tG)].
	 	 
    * 简单的召回通道
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/07_ColdStart_02.pdf)]
    [[YouTube](https://youtu.be/lboewzsA8DU)]
    [[B站](https://www.bilibili.com/video/BV1HY4y157Ri)].

    * 聚类召回
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/07_ColdStart_03.pdf)]
    [[YouTube](https://youtu.be/Tm4SlB9A8BQ)]
    [[B站](https://www.bilibili.com/video/BV1YT4y1q7zC)].
    
    * Look-Alike人群扩散
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/07_ColdStart_04.pdf)]
    [[YouTube](https://youtu.be/pjmRo8Uzzqg)]
    [[B站](https://www.bilibili.com/video/BV1U5411X7ud)].
    
    * 流量调控
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/07_ColdStart_05.pdf)]
    [[YouTube](https://youtu.be/QGD-1Feq1ZQ)]
    [[B站](https://www.bilibili.com/video/BV1vS4y1z7sC)].
    
    * 冷启动的AB测试
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/07_ColdStart_06.pdf)]
    [[YouTube](https://youtu.be/oEUi4mSAv8Q)]
    [[B站](https://www.bilibili.com/video/BV12341137Cq)].


## 8. **涨指标的方法** [[参考文献](https://arxiv.org/abs/2308.01204)]

    * 概述
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/08_Improvement_01.pdf)]
    [[YouTube](https://youtu.be/YptRKEZZ0gY)]
    [[B站](https://www.bilibili.com/video/BV1fc41167jK)].

    * 召回
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/08_Improvement_02.pdf)]
    [[YouTube](https://youtu.be/imi73gPNCFA)]
    [[B站](https://www.bilibili.com/video/BV13H4y127Tt)].
    
    * 排序
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/08_Improvement_03.pdf)]
    [[YouTube](https://youtu.be/2lXflLLEwfA)]
    [[B站](https://www.bilibili.com/video/BV1fQ4y1G72F)].
    
    * 多样性
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/08_Improvement_04.pdf)]
    [[YouTube](https://youtu.be/wW_BZ11hXOY)]
    [[B站](https://www.bilibili.com/video/BV1eN4y1z7vs)].

    * 特殊人群
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/08_Improvement_05.pdf)]
    [[YouTube](https://youtu.be/2hcM-jV84ho)]
    [[B站](https://www.bilibili.com/video/BV15g4y1m7P4)].

    * 交互行为
    [[slides](https://github.com/wangshusen/RecommenderSystem/blob/main/Slides/08_Improvement_06.pdf)]
    [[YouTube](https://youtu.be/CUpwivLDgEA)]
    [[B站](https://www.bilibili.com/video/BV1tg4y127rS)].










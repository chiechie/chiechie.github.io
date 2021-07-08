---
title: 《Advances in Financial Machine Learning》读书笔记1 数据分析
author: chiechie
mathjax: true
date: 2021-07-06 19:50:44
tags: 
- 量化
- 投资
categories:
- AFML
---



## chapter 2 金融数据结构

1. 金融数据经常分为4类，基本面数据，市场交易数据，分析数据，另类数据（Alternative）
![img.png](image.png)
2. 基本面数据包括公司每个季度发布的会计报告，要注意发布时间和统计时间段的区别。
3. 另类数据（Alternative）包括个人数据，商业过程数据，卫星，天气数据。
4. 数据处理，为了让ml方法可用，需要将原始数据处理为表数据（bars）。两种处理方法，标准bar methods和信息驱动的方法。第二种在实践中用的多。
5. 标准bars方法：将不等间隔处理成等间隔数据，很多数据厂商都是提供这种格式。标准bars方法包括Time Bars，Tick Bars，Volume Bars，Dollar Bars

    > 一个tick表示一次成交事件
7. 【Tick Bars】每隔多少笔交易采样一次，属于一种基于信息的采样方法，其理论依据是：固定时间内的价格变化服从方差无限大的Paretian分布；固定交易笔数内的价格变化服从高斯分布。
   
    > Price changes over a fixed number of transactions may have a Gaussian distribution. 
    > Price changes over a fixed time period may follow a stable Paretian distribution, whose variance is infinite. 
    > Since the number of transactions in any time period is random, the above statements are not necessarily in disagreement
    > --Mandelbrot and Taylor 
8. 上面的假设很重要，因为很多统计方法的依赖于假设--样本来自IID高斯过程。
9. 【Tick Bars】构造tick bars要留意异常点，很多交易所在看盘前和收盘后都有竞价（auction），这段时间，order book 积累 bids 和 offers单，并不撮合（match）。当竞价结束，有一笔数量很大的交易会公开，这一笔交易可等价于成千上万个ticks，虽然现实的是一个tick。
10. 【Volume Bars】tick bars的缺陷在于，真实情况下，我们下的一笔单子会被拆分成多笔交易去成交。因此看到的tick比我们实际下的tick变多了。Volume bars可以解决这个问题，他是按照一定证券价值变动的时间段，进行抽样。举个例子，we could sample prices every time a futures contract exchanges 1,000 units, regardless of the number of ticks involved.
11. 【Dollar Bars】每隔一段时间，市场上交易价值达到某个给定值（bar size），就进行抽样，the bar size could be adjusted dynamically as a function of the free-floating market capitalization of a company (in the case of stocks), or the outstanding amount of issued debt (in the case of fixed-income securities)
12. tick bars， volumn bars， dollar bars 三者对比： If you compute tick bars and volume bars on E-mini S&P 500 futures for a given bar size, the number of bars per day will vary wildly over the years. That range and speed of variation will be reduced once you compute the number of dollar bars per day over the years, for a constant bar size. 结论是前面两者每天的变动范围和变动速度，要远高于dollar bars
   ![img_1.png](./img_1.png)
13. 信息驱动的bars，目的在于，当有信息到达时，采样更频繁。信息驱动bars有几种方法：Tick Imbalance Bars，Volume/Dollar Imbalance Bars，Tick Runs Bars，Volume/Dollar Runs Bars
14. 【Tick Imbalance Bars】背后的想法是只要tick数据超过我们的期望，就去采样。这样设置index，累计的交易信号超过某个阈值，没看懂
15. 【Volume/Dollar Imbalance Bars】？
16. 【Tick Runs Bars】？
16. 【Volume/Dollar Runs Bars】？
14. 处理多产品序列：The ETF Trick、PCA Weights，Single Future Roll
    ```python
    def pcaWeights(cov,riskDist=None,riskTarget=1.):
        # Following the riskAlloc distribution, match riskTarget
        eVal,eVec = np.linalg.eigh(cov) # must be Hermitian              
        indices = eVal.argsort()[::-1] # arguments for sorting eVal desc
        eVal,eVec=eVal[indices],eVec[:,indices]
        if riskDist is None:
            riskDist=np.zeros(cov.shape[0])
            riskDist[-1]=1. 
        loads=riskTarget*(riskDist/eVal)**.5 
        wghts=np.dot(eVec,np.reshape(loads,(-1,1))) 
        #ctr= (loads/riskTarget)**2*eVal # verify riskDist 
    # return wghts
    ```
17. 直接让ml预测涨跌很难， after certain catalytic conditions算法会更容易表现好。
18. 对特征进行采样的方法-Event-Based Sampling，其中一种方法叫The CUSUM Filter，利用CUSUM可以构造交易策略（Fama and Blume [1966]的filter trading strategy），同事也可以用来采样：当累计收益$S_t$超过某个阈值时，进行采样，并将$S$置为0，

```python
import pandas as pd
def getTEvents(gRaw,h): 
    # gRaw： raw time series
    # h: thresh
    tEvents,sPos,sNeg=[],0,0 
    diff=gRaw.diff()
    for i in diff.index[1:]:
        sPos,sNeg=max(0,sPos+diff.loc[i]),min(0,sNeg+diff.loc[i]) 
        if sNeg<-h:
            sNeg=0
            tEvents.append(i) 
        elif sPos>h:
            sPos=0
            Events.append(i) 
    return pd.DatetimeIndex(tEvents)
```
19. $S_t$可以是structural break statistics, entropy, or market microstructure measurement。比如，我们可以定义一个时间，之哟啊r SADF远离之前的取值足够远。
20. 使用event-based的方法获得了一个子集之后，可以让ml算法来分析，这些特殊事件有没有蕴含一些有值得决策的信息。



## chapter 3 标记

在监督学习中，需要输入label，那么在金融领域，如何定义label？
固定时间范围方法不够准确（可用动态阈值来改进），同时未考虑价格变化的路径，更好的方法是三边界法；此外，元标签能结合各种先验知识，是基金公司做模型、裁员工必备工具。



### 固定时间段方法

1. 大部分论文都是采用的这个方法，即固定的一段时间收益率是否超过/低于某个取值。
    ![img_2.png](./img_2.png)
2. 虽然大部分人这么用，但是这个方法跟固定时间段采样有一样的毛病，就是固定时间段内的样本并不服从gaussian分布。第二个缺陷是，这个阈值是固定的，无视当前市场波动率的变化，可能会导致错失很多有价值的正样本。
3. 有更优的标记方法：动态阈值（类似异常检测）和  volume /dollar bars（波动率更固定）， 
4. 即使改进了fixed time 和 fixed thresh，还有一个很显现实的问题就是，要考虑到价格路径，如果在半路触发margin call，那么预测得再准也没有用。


### 三边界方法（THE TRIPLE-BARRIER METHOD）

1. 简单说，固定一个窗口，价格先达到上沿就标记1，先达到下沿就标记-1，到窗口结束都被碰到就标记0。
1. 具体说，首先设置2个水平障碍和1个垂直障碍。2个水平障碍是基于变动的日波动率算出来的，1个垂直障碍是说，离上一次position take，经过了bars的个数。
2. 如果upper障碍最先触发，返回1；如果lower障碍最先触发，返回-1；如果垂直的障碍触发，返回-1/+1，或者0，具体情况具体分析.三重障碍方法是路径依赖的标记方法。
```python
def applyPtSlOnT1(close,events,ptSl,molecule):
    # apply stop loss/profit taking, if it takes place before t1 (end of event)
    events_=events.loc[molecule] 
    out=events_[['t1']].copy(deep=True)
    if ptSl[0]>0:
        pt=ptSl[0]*events_['trgt'] 
    else:
        pt=pd.Series(index=events.index) # NaNs
    if ptSl[1]>0:
        sl=-ptSl[1]*events_['trgt'] 
    else:
        sl=pd.Series(index=events.index) # NaNs
    for loc,t1 in events_['t1'].fillna(close.index[-1]).iteritems():
        df0=close[loc:t1] # path prices df0=(df0/close[loc]-1)*events_.at[loc,'side'] # path returns out.loc[loc,'sl']=df0[df0<sl[loc]].index.min() # earliest stop loss. out.loc[loc,'pt']=df0[df0>pt[loc]].index.min() # earliest profit taking.
    return out
```
3 根据需要，三边界方法也可以有其他合理变体：下边界+右边界：我们会在一定时间后平仓，除非触发止损点提前平仓；上边界+下边界：如果没有触发盈利点和止损点，则一直持有股票；（见下图）
![img_4.png](img_4.png)
这部分讨论了三边界方法的代码实现，即如何给样本打标签使得 ML 算法可以同时学习到一笔交易的方向和规模

### 同时学习方向和规模（LEARNING SIDE AND SIZE）

1. 这种标记可以让ml算法从side和size中学习到一些信息
2. 如果没有side信息，我们没法区分profit-taking 障碍 和 stop-loss 障碍。

### META-LABELING

假设你有个模型能决定交易方向，你只需要确定交易的规模（包括不交易，即规模为0）。这是金融从业者经常需要考虑的问题，我们确定要买或者卖，唯一的问题是这笔交易值得冒多大风险；同时，我们不需要 ML 模型学习交易方向，只需要它告诉我们合适的交易规模是多少。

假如有一个基于金融理论的模型，告诉我们交易方向，那我们的标签就变成了 [公式] （ML模型只需要决定是否执行这个操作），而不是 [公式] （ML模型同时决定交易方向和规模）

> 元标签的含义:
> 
> 金融中用ML的另一常见错误是同时学习仓位的方向和规模（据我所知很多论文仅对买/卖方向做决策，每笔交易的金额/股数是固定的）。具体而言，方向决策（买/卖）是最基本的决策，规模决策（size decision）是风险管理决策，即我们的风险承受能力有多大，以及对于方向决策有多大信心。我们没必要用一个模型处理两种决策，更好的做法是分别构建两个模型：第一个模型来做方向决策，第二个模型来预测第一个模型预测的准确度。很多ML模型表现出高精确度（precision）和低召回率（recall），这意味着这些模型过于保守，大量交易机会被错过。F1-score 综合考虑了精确度和召回率，是更好的衡量指标，元标签（META-LABELING）有助于构建高 F1-score 模型。首先（用专家知识）构建一个高召回率的基础模型，即对交易机会宁可错杀一千，不可放过一个。随后构建一个ML模型，用于决定我们是否应该执行基础模型给出的决策。元标签+ML有以下4个优势：1. 大家批评ML是黑箱，而元标签+ML则是在白箱（基础模型）的基础上构建的，具有更好的可解释性；2. 元标签+ML减少了过拟合的可能性，即ML模型仅对交易规模决策不对交易方向决策，避免一个ML模型对全部决策进行控制；3. 元标签+ML的处理方式允许更复杂的策略架构，例如：当基础模型判断应该多头，用ML模型来决定多头规模；当基础模型判断应该空头，用另一个ML模型来决定空头规模；4. 赢小输大会得不偿失，所以单独构建ML模型对规模决策是有必要的

### 量化 + 基本面方法

THE QUANTAMENTAL WAY

很多对冲基金——包括一些老牌基金——正在拥抱量化方法。元标签正是这些公司需要的：假设你有了一系列有预测力的特征，你既可以同时预测交易方向和规模；也可以用元标签方法。元标签方法中确定方向的基础模型可以是 ML模型、计量公式、交易规则、基本面分析，也可以是人类基于直觉的预测结果，可见元标签方法的普适性。

举个例子，元标签方法可能会发现基金经理能及时预测市场风格转换，但无法在疲倦、压力下准确预测。由于基金经理必然会受生理心理等因素影响，元标签方法能评价基金经理的预测能力。综上所述，元标签方法为基金公司的量化之路指明了方向（做模型 & 评价基金经理），它应该成为基金公司的基本工具。

### 丢掉不必要的label

it is preferable to drop extremely rare labels and focus on the more common outcomes.
当标签很多且类别不均衡（imbalance）时，一些ML模型表现不好。这种情况下，最好丢掉非常罕见的标签并专注于更常见的结果。这样做有另一个原因，即用bagging方法时罕见的标签可能无法采集到，这是 sklearn 的一个bug，短期难以解决，建议读者写自己的 class，扩展 sklearn的功能。


## chapter 4 样本权重

训练ML 模型需要抽取样本，本章我们会考虑抽样时如何给样本加权，以更好地训练模型。

1. 大部分ML算法都是基于IID假设，而金融时序不是IID的，所以大部分ml应用直接套用到金融场景会失败。
2. 很多时候数据难免出现交叉（如三标签方法一段数据结束时间不确定），当两段数据出现交叉，标签序列就不再是IID了。这种场景经常出现在非time bars中。
![img_3.png](img_3.png)
3. 对此我们有三种解决方案：一是丢弃重复数据，这会造成信息损失，不推荐；二是根据独特性加权抽样——一段数据与其他数据交叉越少，独特性越高，应该给予更多权重；三是 Sequential Bootstrap，即序列有放回抽样，每抽出一个样本，相应地减少与该样本有重叠的样本被抽取的概率，这样抽取的样本比普通 Bootstrap 更接近 IID。
4. 此外，绝对收益率（absolute return）大的样本应该给予更多权重，原因是对 ML 算法来说，绝对收益率小的样本不好预测，作为训练样本价值不大。
    
    > The “neutral” case is unnecessary, as it can be implied by a “−1” or “1” prediction with low confidence.
5. 市场是常为新的，越新的数据与当前市场相关度越高，价值越大。

    > Markets are adaptive systems (Lo [2017]). As markets evolve, older examples are less relevant than the newer ones.
6. 最后，我们还应该考虑类别权重。金融中不均衡数据集很常见，而且这些罕见的标签往往非常重要。在 sklearn 等科学计算包中可以设置为 class_weight='balanced' 。
7. label表示 买/卖 信号, 两个label 如果是基于相同时间段的收益率 计算出来的，就说是 concurrent的.
8. 对样本使用bootstrap方法抽样，以期得到iid样本。
9. 基于uniqueness和absolute return对样本赋予权重。绝对收益高的的labels应该被给予更高的权重；收益取值越unique的也要给予更高的权重
10. 市场是演化着的，所以我们希望给新忘本更多的权重，给老样本更少的权重。
11. 怎么量化这个事件衰减效应？设计一个时间衰减因子（所有元素加起来为1），用这个因子乘以样本权重，
12. 使用机器学习做分类时，有的稀有事件（比如金融危机）出现次数很少，为了保证ml算法能重视这类事件，可以调整sample_weight
13. 具体来说，在scikit learn中，设置class_weight='balanced'，或者在bagging trees中设置class_weight='balanced_subsample'，小心[bug](https://github.com/scikit-learn/scikit-learn/issues/4324)

## chapter 5 分数差分

分数差分--Fractionally Differentiated Features

如何兼顾平稳性（adf）和 记忆性（跟price的相关性）？--分数差分

### STATIONARITY VS. MEMORY的两难问题

1. 金融序列大部分非平稳，且有很低信噪比，标准的平稳变换，例如差分变换，会丢失信息。
2. 价格序列有记忆，但是差分后的序列没有记忆了。
3. 接下来理论家们会从剩下的残差信号中使用各种fancy的工具去提取信息。
4. 金融序列不平稳的原因是，它有很长的记忆.所以要使用传统的方法的话要做invariant processes，例如看价格的收益率或者取对数差，波动性变化
5. 在信号处理中，我们是不希望所有的记忆都被抹除的，因为记忆是信号模型的basis。例如，均衡平稳模型需要一些记忆，来获取截止目前为止，结果偏离长期预测值多远，来预测。矛盾在于，收益是平稳的，但是没有记忆。价格有记忆，但是不是平稳的。
那么问题就来了：最小的差分阶数是什么？既能满足一个价格序列平稳，又能保留尽可能多的信息？
6. 协整（cointergration）方法可以使用记忆来建模。
7. 平稳性只是ml算法的必要不充分条件，但是通过差分变换的方法虽然获得了平稳性却丢失了记忆性，会导致ml基本上没有什么记忆能力。

下面会介绍一些转换方法，在保留记忆的同时，又能实现平稳变换。


### 分数差分方法

1. 如何解决平稳和记忆两难的问题？Hosking [1981]提出了分数差分的方法。
    ![img.png](fd.png)
2. 使用迭代法计算权重向量
    ![img.png](fd1.png)
    ![img_1.png](fd2.png)
3. 在SP500上面做实验，当差分d=0.35时，跟原始价格序列的相关性仍然很高, 0.995，d=1时候，相关性只有0.03, 基本上丢失了记忆。从adf上看，d=0.35时, 序列的黏稠度也不高，adf约等于 –2.8623， 原始的adf是–0.3387,d=1对应的adf是–46.9114。
4. Expanding Window 和 固定宽度窗口分数差分方法(Fixed-Width Window Fracdiff)


## 参考

1. 《Advances in Financial Machine Learning》
2. https://blog.csdn.net/weixin_38753422/article/details/100179559
3. https://zhuanlan.zhihu.com/p/69231390
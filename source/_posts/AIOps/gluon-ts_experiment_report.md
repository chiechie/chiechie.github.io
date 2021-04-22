---
title: 使用gluon-ts做时序预测
author: chiechie
mathjax: true
date: 2021-04-22 17:35:59
tags:
categories:
---

## 背景

- [electricity数据集--Electricity dataset from UCI](https://archive.ics.uci.edu/ml/datasets/ElectricityLoadDiagrams20112014)： 
- 参数
    - input_size： window_size-stride_size，14
    - stride
    - window_size： 192 hour or 8天
    - stride_size:  24 hour，滑窗步长
    

## 问题

- gluon-ts里面将输入做对数处理效果会不会好一点？
    - 不用，mxnet已经做了， 并且额外做预处理，可能还会带来负面影响，
        - loss_weights 是根据observed_values的min确定的---所以会忽略小量岗的曲线，学的不好的，
    - 参考论文--context-length内的所有z求平均值，作为scale来归一化原始输入并且记下来，模型输出的u 和 sigma 再 使用这个scale来逆归一化回去。
- 为什么pytorch的训练要7个小时？mxnet训练只要10min？
    - 主要在样本采样这块
- 自己能不用tensorflow复现mxnet的效果？
    - 对比参数量，准确率，计算时间，内存消耗
- 为什DeepAR和DeepFator的loss scale 差这么大?
    - loss的含义都不一样
        - deepAR的loss计算方式：
 ```python
def loss(self, x: Tensor) -> Tensor:
	return -self.log_prob(x)
```
    - deepFator的loss计算方式：
 ```python
def negative_normal_likelihood(self, F, y, mu, sigma):
    return (
        F.log(sigma)
        + 0.5 * math.log(2 * math.pi)
        + 0.5 * F.square((y - mu) / sigma)
    )
```
- 对不同曲线预测的时候，为什么有的曲线效果很， 好有的曲线效果很差？
    - deepAR会针对不同曲线，给loss 赋予不同的权重：loss_weights 是根据observed_values的min确定的---所以会忽略小量岗的曲线，学的不好的，

## 算法前期调研中的一些小插曲

网上找到了pytorch版本的deepAR版，但是很慢

- Student： 训练过程很慢 要7个小时，虽然paper里面也号称要7个小时，是优化效率还是直接上线呢？
- Teacher： 看了amazon已经将deepAR做了产品化，按照算法开发的流程肯定经历了效率优化的，并且deepAR的文章发于2017年4月，到目前为止amazon应该做了多次迭代的，所以去查一下近3年的amazon deepAR 相关的资料，看有没有开源的稳定的算法代码。
- Student：找到了--基于MXNET的gluon-ts（于2019年6月发表），mxnet在electricty上的训练和预测时间大概是10min左右
- Teacher：很好，接下来可以考虑更多的产品化方面的事情了，结合运维场景；另外注意到CNN-QR的训练效率也很不错，把这几个算法的准确率对比看一下
- Student：整体上准确率 CNN-QR略逊于DeepAR，但是考虑到计算效率，两者还是不相上下。
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FUD6-nafIsi.png?alt=media&token=a359d00d-bde2-41b2-bb96-d9222cb4514c)
- Teacher：下一步，试着CNN-QR的拓展。
- Student：好的
- Teacher： DeepAR用于异常检测 的效果也测试下
- Student：使用数据平台的两台主机的cpu指标测试，发现cpu的效果还不错，体现在 不同类型的曲线，他们的方差趋势都预测的很准。
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FxofMTtcgaV.png?alt=media&token=26cd83f2-717e-4c9c-8a93-da589dcfe63d)
- Teacher：使用有异常标注的数据测试具体的指标，两种都测下--批量预测 和 实时预测
- Student：在获取数据阶段踩了一些坑，因为维度太多，全部数据维度都要 会导致拉数据接口快挂了
- Teacher：找出其中最重要的维度（重要度 即每个维度的记录的条数或者指标求和）
- Student： 从5w个维度过滤出来了100个维度，剩下的大部分维度怎么弄呢？
- Teacher：先管好这个100个维度，跑一跑模型，剩下的数据质量差可以暂时不监控。
- Student： 这批数据有个特点--不同曲线形态模式很不一样，我怀疑1个模型很容易学混，接下来就证实了这一点（100个曲线上面的图，待补充）![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FHF_Qc1aO1g.png?alt=media&token=3591cf24-bfdf-44b7-894b-ba03652522f1)
- Teacher： 尝试加入feat_static_cat特征，来区分不同曲线。
- Student：有些许变化，但是改进还不太明显。另外在做实验过程中，发现几个优化的方向
    - **epoch**： 从 5 到 50，带来了大量的提升
    - **lstm**的网络结构参数：layers从2到1，units从40到20，准确率有部分提升
- Teacher：因为样本较少（3k个训练样本）更适合用简单的模型
- Student：除了直接使用deepar的默认scale以及概率分布的参数，还尝试了另外一个方法： 因为cpu使用率本身是0～1，将数据预处理（除以100）到0～1 之间后跑，但是直接跑模型会报错， 做了以下设置才能跑：
    - scale设置为False
    - value 为边界点（0或者1）时，加入扰动项
    - 输出的概率分布设置为beta分布
    输出的上下界，非常符合比率型设定（上界不会超过1，下界不会低于0），但是就是准确率不高，一看就是学混了（移花接木，局部的模式 太弱了）

- 出bug了，排查问题

## deepAR实验v1

- [code](https://github.com/zhykoties/TimeSeries)： pytorch版本

    
```python
    for series in trange(num_series):
        cov_age = stats.zscore(np.arange(total_time-data_start[series]))
        if train:
            covariates[data_start[series]:time_len, 0] = cov_age[:time_len-data_start[series]]
        else:
            covariates[:, 0] = cov_age[-time_len:]
        for i in range(windows_per_series[series]):
            if train:
                window_start = stride_size*i+data_start[series]
            else:
                window_start = stride_size*i
```
        - num_covariates： 4个
        - pred_days : 7
        - num_vovariates: 4个
            - 周几
            - 几点
            - 月份
    - [整理](https://iwiki.woa.com/pages/viewpage.action?pageId=326062234)
- deepFactor vs deepAR效果对比：
```python
1台主机的网络入流量数据：
deepFactor
{
    "MSE": 303732622110915.06,
    "abs_error": 2275600640.0,
    "abs_target_sum": 16922566656.0,
    "abs_target_mean": 100729563.42857143,
    "seasonal_error": 7606565.400503778,
    "MASE": 1.7807303548412023,
    "MAPE": 0.12016319151568138,
    "sMAPE": 0.13137161232512512,
    "OWA": NaN,
    "MSIS": 71.22905232846578,
    "QuantileLoss[0.5]": 2275600712.0,
    "Coverage[0.5]": 0.23214285714285715,
    "RMSE": 17427926.500617195,
    "NRMSE": 0.17301699627612852,
    "ND": 0.1344713651456159,
    "wQuantileLoss[0.5]": 0.13447136940028964,
    "mean_wQuantileLoss": 0.13447136940028964,
    "MAE_Coverage": 0.26785714285714285
}
deepAR结果：
{
    "MSE": 103720750278558.47,
    "abs_error": 1316331008.0,
    "abs_target_sum": 16922566656.0,
    "abs_target_mean": 100729563.42857143,
    "seasonal_error": 7606565.400503778,
    "MASE": 1.0300711564944531,
    "MAPE": 0.07665473185019729,
    "sMAPE": 0.07928408221885551,
    "OWA": NaN,
    "MSIS": 16.15919346419098,
    "QuantileLoss[0.5]": 1316330924.0,
    "Coverage[0.5]": 0.27976190476190477,
    "RMSE": 10184338.480164457,
    "NRMSE": 0.10110575419485757,
    "ND": 0.07778554132822911,
    "wQuantileLoss[0.5]": 0.07778553636444309,
    "mean_wQuantileLoss": 0.07778553636444309,
    "MAE_Coverage": 0.22023809523809523
}
结论：看来deepAR还是略胜deepFactor一范畴
```
- lol整体预测
- lol分商品预测 
- pct预测
- cpu使用率
- 磁盘使用率
- 网路流量检测

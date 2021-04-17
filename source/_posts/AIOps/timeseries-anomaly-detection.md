---
title: 时序异常检测方案概述
author: chiechie
mathjax: true
date: 2021-04-16 15:26:42
tags:
- 异常检测
- AIOps
- 时间序列
categories:
- AIOps
---

## 模型配置

- 历史依赖：取值为2天，还可以设置，取值越大，越不敏感。
- 平滑窗口：取值为5个点，还可以设置，取值越大，越敏感。


## 主要处理流程

1. 对历史数据做平滑处理, 处理的函数为```numpy_ewma_vectorized_v2```
```python
import numpy as np
def numpy_ewma_vectorized_v2(data, window):
    alpha = 2 / (window + 1.0)
    alpha_rev = 1 - alpha
    n = data.shape[0]

    pows = alpha_rev ** (np.arange(n + 1))

    scale_arr = 1 / pows[:-1]
    offset = data[0] * pows[1:]
    pw0 = alpha * alpha_rev ** (n - 1)

    mult = data * pw0 * scale_arr
    cumsums = mult.cumsum()
    out = offset + cumsums * scale_arr[::-1]
    return out
```

2. 计算动态上下界： 
   - 计算正常的波动区间：
      - 计算残差序列 = 平滑后的历史依赖 - 原始的历史依赖
        ```python
        residual = history - history_s
        ```

      - 取残差序列中大于0的部分，计算均值history_std_u，即为正常的向上波动范围
        ```python
         history_std_u = np.nanmean(residual[residual > 0])
         ```
      - 取残差序列中小于0的部分，计算均值history_std_l, 即为正常的向下波动范围
        ```python
         history_std_l = np.nanmean(residual[residual < 0])
        ```
   - 计算当前基线```history_mean```：历史依赖平滑后的最新一个时刻的取值，即为当前的基线。
     ```python
     history_mean = history_s[-1]
     ```
   - 计算上下界： 上界 = 当前基线 + N_sigma * 向上波动范围
              下界 = 当前基线 - N_sigma * 向上波动范围
     ```python
      upper_bound = history_mean + N * history_std_u
      lower_bound = history_mean + N * history_std_l
      ```


### 3. 将敏感度映射为N

小trick1--将敏感度映射为N（N可以控制上下波动范围）, N = 1 -convert_losseness_to_n(敏感度)
   ```python
   def convert_losseness_to_n(l):
       looseness = [0.3, 0.5, 0.7]
       n_sigma = [4, 8, 16]
       n_sigma_sqrt = np.sqrt(n_sigma)
       z1 = np.polyfit(looseness, n_sigma_sqrt, 2)
       p1 = np.poly1d(z1)
       return np.square(p1(l))
   ```

### 4. 清洗极端值

小trick2--对历史数据做平滑处理之前，以及计算正常的上下波动范围之前，都应该对输入序列清洗极端值。

   ```python
   def replace_etv(arr):
       arr = np.array(arr)
       arr_mean, arr_std = np.nanmedian(arr), np.max([np.std(arr), eps])
       u, l = arr_mean + 5 * arr_std, arr_std - 5 * arr_std
       anom = (arr > u) | (arr < l)
       arr[anom] = arr_mean
       return arr
   ```

### 总结

总结一下，整个处理逻辑放在```gaussian_detector```中

```python
def gaussian_detector(history, point, N=3, alert_upward=1, alert_down=1, exp_window=5):
    history_s = copy.deepcopy(history)
    
    history_s = replace_etv(history_s)
    history_s = numpy_ewma_vectorized_v2(history_s, exp_window)

    residual = history - history_s
    residual = replace_etv(residual)

    history_mean = history_s[-1]
    history_std_u = np.nanmean(residual[residual > 0])
    history_std_l = np.nanmean(residual[residual < 0])

    history_std_u = np.max([history_std_u, eps])
    history_std_l = np.min([history_std_l, -eps])

    upper_bound = history_mean + N * history_std_u
    lower_bound = history_mean + N * history_std_l

    is_up_anomaly = point > upper_bound
    is_down_anomaly = point < lower_bound
    is_anomaly = int(is_up_anomaly | is_down_anomaly)

    return is_anomaly, upper_bound, lower_bound, history_mean, history_std_u
```

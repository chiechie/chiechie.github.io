---
title: 使用gluon-ts做时序预测(2)
author: chiechie
mathjax: true
date: 2021-04-22 19:36:28
tags: 
- 的
categories: 
- 技术
---

## 对象关系

gluon-ts 对象关系（自上而下；从系统顶层 到 底层）：

- 基类-gluonts.model.Estimator：相当于平台层，实现主流程, 核心方法有
    - train: 执行训练，返回predictor，抽象方法
- 父类-gluonts.model.GluonEstimator：
    - create_transformation: 抽象方法
    - create_training_network：抽象方法
    - create_predictor：抽象方法
    - train_model： 执行训练，返回TrainOutput, 依次调用：
        - self.create_transformation()： 返回`transformation`
        - self.create_training_network()：返回`trained_net`
        - self.trainer()：```trainer = Trainer(epochs=epochs, batch_size=batch_size)```
        - self.create_predictor(): 输入`transformation`和 `trained_net`， 返回`predictor`，```self.create_predictor(transformation, trained_net)```
        - train：执行训练，返回TrainOutput.predictor，self.train_model()： 
- 子类（gluonts.model.deepar.DeepAREstimator）：相当于用户层，负责具体算法的逻辑, 需要实现3个函数，其他的调度类方法继承父类，例如train，和它的兄弟类共享。
    - **create_transformation**： 返回`Transformation`对象, 依次调用
        - AddObservedValuesIndicator： 
        - AddTimeFeatures：
        - AddAgeFeature
    - **create_training_network**： 返回 `trained_network`对象，调用
        - DeepARTrainingNetwork 实例化
    - **create_predictor**：返回：`Predictor`对象
        - DeepARPredictionNetwork 实例化
        - copy_parameters：将训练好的网络参数传给预测网络
```python
copy_parameters(trained_network, prediction_network)
```
            - RepresentableBlockPredictor：transform + 预测网络
```python
RepresentableBlockPredictor(
            input_transform=transformation,
            prediction_net=prediction_network,
            batch_size=self.trainer.batch_size,
            freq=self.freq,
            prediction_length=self.prediction_length,
            ctx=self.trainer.ctx,
            dtype=self.dtype,
        )
```

### 辅助类-DeepARTrainingNetwork 和 DeepARPredictionNetwork

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FVuuzwiLxCK.png?alt=media&token=151a163b-35ae-407e-8ac1-a6381a545f5a)
- DeepARNetwork：父类
    - unroll_encoder：
        - get_lagged_subsequences：
        - scaler：
        - self.rnn.unroll：
- DeepARTrainingNetwork：
    - hybrid_forward： 返回 loss 和 weighted_loss
        - distribution：
            - unroll_encoder
        - distr.loss
            - loss_weights 是根据observed_values的min确定的---所以会忽略小量岗的曲线，学的不好的，
                - observed_values 是 past_observed_values 和 future_observed_values 联合的结果
```python
        # (batch_size, seq_len, *target_shape)
        observed_values = F.concat(
            past_observed_values.slice_axis(
                axis=1,
                begin=self.history_length - self.context_length,
                end=self.history_length,
            ),
            future_observed_values,
            dim=1,
        )
    
  		# loss_weights 是根据observed_values的min确定的
  		loss_weights = (
            observed_values
            if (len(self.target_shape) == 0)
            else observed_values.min(axis=-1, keepdims=False)
        )

        weighted_loss = weighted_average(
            F=F, x=loss, weights=loss_weights, axis=1
        )
        
      	loss = F.where(condition=loss_weights, x=loss, y=F.zeros_like(loss))

    return weighted_loss, loss
```
        - DeepARPredictionNetwork：
            - hybrid_forward： unroll_encoder 和 sampling_decoder


## 构建概率分布

- StudentT分布，有三个参数：mu，sigma，mu
```python
def s(mu: Tensor, sigma: Tensor, nu: Tensor) -> Tensor:
    F = self.F
    gammas = F.sample_gamma(
        alpha=nu / 2.0, beta=2.0 / (nu * F.square(sigma)), dtype=dtype
    )
    normal = F.sample_normal(
        mu=mu, sigma=1.0 / F.sqrt(gammas), dtype=dtype
    )
    return normal
```
- TransformedDistribution: 将近似正态分布 转换为 带scale的 真实分布
- distr_args：StudentT分布的3个参数<mu，sigma，nu>
- TrainDataLoader: 将TS分解成小片段
- sequence = F.concat(past_target, future_target, dim=1)
- sequence_length： self.history_length + self.prediction_length
- subsequences_length = self.context_length + self.prediction_length
- data_entry是：一个样本
```javascript
data_entry.keys()
----
Out[10]: dict_keys(
  ['start', 'source', 
	'feat_static_cat', 'feat_static_real', 
  'past_time_feat', 'future_time_feat', 
  'past_observed_values', 'future_observed_values',
   'past_target', 'future_target',
   'past_is_pad', 'forecast_start'])
```
- history_length：真实用到的历史依赖数据，self.history_length = self.context_length（就是72） + max(self.lags_seq)：lags是滞后项，用来捕捉长期的周期性,793
- extract_pred_target：时间序列中找到非预测区间的片段
```python
    def extract_pred_target(
        time_series: Union[pd.Series, pd.DataFrame], forecast: Forecast
    ) -> np.ndarray:
        """

        Parameters
        ----------
        time_series
        forecast

        Returns
        -------
        np.ndarray
            time series cut in the Forecast object dates
        """
        assert forecast.index.intersection(time_series.index).equals(
            forecast.index
        ), (
            "Cannot extract prediction target since the index of forecast is outside the index of target\n"
            f"Index of forecast: {forecast.index}\n Index of target: {time_series.index}"
        )

        # cut the time series using the dates of the forecast object
        return np.atleast_1d(
            np.squeeze(time_series.loc[forecast.index].transpose())
        )
```

- lags_seq: 滞后项的个数
    - len(self.lags_seq)是40，
- seq_len 是什么？rnn的序列长度，72
- target是什么？
    - target = past_target[721:] + future_target
    - 形状为(batch_size, seq_len, *target_shape)
- past_observed_values是什么？
    - 指示变量（observed_indicator）： 0表示改位置的取值是缺失的，1表示是真实的观测值， 一般跟另外一个形状相同的实值向量配套实用。（以past_observed_values为例，就是跟past_values一起使用）
    - 形状为<batch_size, history_length >， eg 1x793
- past_target是什么？
    - past_target 是长度为history_length的序列
    - 原始的TS序列变成1个past_target 还是多个 past_target ？
        - 从节省样本的角度应该是多个
        - 测试的时候确实只用了最新的窗口，但是训练的时候不是，详细见下面
        - 看训练过程的日志：
            - 根据信息-测试用例1
                - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F8TYYuBdntP.png?alt=media&token=90dbf3d6-2a89-499e-8316-af08b540c41c)
                - 2/50 [00:06<00:20,  1.85it/s, epoch=3/10, avg_epoch_loss=17.6]
                - 50/50 [00:24<00:00,  2.04it/s, epoch=1/10, avg_epoch_loss=19]
                - 每次迭代/梯度更新 使用了1/1.85s = 0.54s；
                - 完整的一轮学习耗时预估为20s
                - 批量大小为64
            - 可以推测：
                - 批量个数为50
                - 总样本个数为：50 * 64 = 3200个
                - 检验： 0.54s * 50  == 20？ 差不多
            - 再测试下--测试用例2 
                -  32% █████████████| 16/50 [00:03<00:05,  6.44it/s, epoch=1/10, avg_epoch_loss=19.7]
                - 批量大小为1
                - 每次迭代使用1 / 6.44s = 0.155 s
                - 完整一轮学习使用了8s
                - 推断：批量个数为50；总样本个数 50 * 1 =50个
                - 检验：0.155  * 50 == 8s?---是对的
            - 综合测试用例1 和 测试用例2：
                - batch_size 为64时， 每次迭代时间为0.54s；
                - batch_size 为8时， 每次迭代时间为1/7.00 = 0.145s；
                - batch_size 为1时， 每次迭代时间为0.155s；
                - batch_size 为128时， 每次迭代时间为1 / 2.08==0.48s
            - 思考：为什么只能有50个batch？这不是很浪费样本吗？
                - 因为每次样本都是随机采样，所以增加epochs的效果， 等价于提升样本使用率。（不是很确定）
    - 这是因为给的原始的TS长度只有context_length, 所以history_length的更早位置的数据是0
- past_target 和 past_observed_values有什么联系？
- past_time_feat是什么？<batch_size, history_length, 6>，形状为1x793x6
- loss的长度是多少？形状为(batch_size, seq_len)，loss = distr.loss(target)
- unroll_encoder的输入输出是什么？
    - 输入：
        - feat_static_cat，1x1，0
        - feat_static_real，1x1，0
        - past_time_feat，1x793x6
        - past_target，1x793
        - past_observed_values，1x793
    - 输出：
        - a: 形状为1x72x40
        - state: list,  长度为4, 4个1*40的array
            - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FlQP5XUIibI.png?alt=media&token=d0dc0aed-bf50-43b0-8ced-f580af0d593c)
        - scale：1*1， 89839590
        - static_feat： 1*3 的矩阵
            - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FyUrVcW7cVK.png?alt=media&token=6af2ee80-183e-4aed-acdc-877bf09e4859)
- rnn.unroll
    - update一点理解： 可以把rnn看成一个带状态的函数，最原子的操作（对应的主体取名叫单元rnn把）是输入m个特征（注意 都是一个时间点）输出num_cell个实数，同时更新自己的状态（也就是num_cell个参数），拓展到多个时间点，只是把单元rnn 加上了一个计算调度，自动的运行seq_len次（seq2seq中encoder序列长度，或做推理的时候 encoder序列长度 + k）
    - 输入：
        - inputs：1x72x49， 【input_lags, time_feat, repeated_static_feat】
            - input_lags：
                -  1x72x40，seq_len = 72,   self.lags_seq_len = 40
            - time_feat： 1x72x6
            - repeated_static_feat：1x72x3，把static_feat 拷贝72份（context_length）
                - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FN9r8UcjmWW.png?alt=media&token=f10dde5d-71e6-4de8-9483-f95ad213fb8a)
        - length： subsequences_length==144, 表示seq2seq架构中，encoder序列的长度（72） + decoder序列的长度（72）
    - 输出：
        - outputs： 1x72x40
        - state：1x40 
- sampling_decoder的输入输出是什？
    - 输入：rnn输出的 output 就不要了，只要state相关信息，以及未来要用到的信息
        - past_target： 1x793
        - future_time_feat：1x72x6 ，这个是需要针对future period 计算的
        - static_feat：1x3， [[-1.0517105  0.        18.313536 ]， (batch_size, num_features + prod(target_shape)),  num_features包括嵌入特征（-1.0517105）和静态特征（0），以及target的scale （ 18.313536 ]）， 这个就是拷贝 past 算好的

```python
# in addition to embedding features, 
#  use the log scale as it can help prediction too

static_feat = F.concat(
            embedded_cat,
            feat_static_real,
            F.log(scale)
            if len(self.target_shape) == 0
            else F.log(scale.squeeze(axis=1)),
            dim=1,
        )
```
            - scale：1*1，89839590，(batch_size, 1, *target_shape)
            - state：4个1*40的向量， list of (batch_size, num_cells) tensors
        - 输出：
            - 1x100x72，Shape: (batch_size, num_sample_paths, prediction_length).
            - A tensor containing sampled paths. 
        - 中间处理逻辑：
            -  for k in range(prediction_length):
                - 构造inputs: 
                    - get_lagged_subsequences构造lags： 返回[72, 40]
                        - sequence：repeated_past_target, 形状为  100x793
                        - sequence_length=self.history_length + k, 表示rnn处理的长度
                        - indices=self.shifted_lags, 就是原始的滞后项，长度为40
                        - subsequences_length=1,
                        - 输出：
                - rnn.unroll(...) ： 这个rnn.unroll 是将单元rnn更新多次 
    - future_observed_values是什么？
        - 形状为(batch_size, prediction_length, *target_shape)
    - ObservedValue是什么？
        - 形状为(batch_size, seq_len, *target_shape)
        - observed_values  =  past_observed_values[721:793]  + future_observed_values 
- 归一化，获取lags，特征工程，特征拼接，这四个步骤的顺序是怎么样的？
    - 概览
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fh2sU85bMJq.png?alt=media&token=1c752076-827a-469a-af0e-a51fbe3c4e9e)
    - deepar 细节图：
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FbFiYx7kM9g.png?alt=media&token=83eccdba-5fd3-4f73-9804-ae8536ba16a7)
- DeepAR的神经网络没法可视化？-有没有替代方案？
- 看一下deepAR的周期性误差算的是什么？有做归一化吗？
- 【借鉴】归一化是怎么做的？
    - ```python
_, scale = self.scaler(
    past_target.slice_axis(
        axis=1, begin=-self.context_length, end=None
    ),
    past_observed_values.slice_axis(
        axis=1, begin=-self.context_length, end=None
    ),
)```
    - 只用history_length中最近context_length的subTS作为样本，并且剔除掉缺失值， 求均值，作为scale
        -  这个只是一个示范，可以看一下输入输出的结构，他实际上并没有做归一化
```python
class NOPScaler(Scaler):
    """
    The ``NOPScaler`` assigns a scale equals to 1 to each input item,
    i.e.,
    no scaling is applied upon calling the ``NOPScaler``.
    """

    @validated()
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    # noinspection PyMethodOverriding
    def compute_scale(
        self, F, data: Tensor, observed_indicator: Tensor
    ) -> Tensor:
        """
        Parameters
        ----------
        F
            A module that can either refer to the Symbol API or the NDArray
            API in MXNet.

        data
            tensor containing the data to be scaled.

        observed_indicator
            observed_indicator: binary tensor with the same shape as
            ``data``, that has 1 in correspondence of observed data points,
            and 0 in correspondence of missing data points.

        Returns
        -------
        Tensor
            shape (N, C), identically equal to 1.
        """
        return F.ones_like(data).mean(axis=self.axis)
```
    - 然后 对所有的lags项（72 * 40） 除以 scale。

## gluon中怎么实现概率层？

- gluon中怎么实现概率层？class ArgProj(gluon.HybridBlock):A block that can be used to project from a dense layer to distribution
- estimator训练的时候会调用predict吗？
- forecast是什么？为什么要定义这个对象？
    - 是一个生成器，这个对象 表示“预备算”的状态
    - 使用next 或者 for 循环的时候，才真的开始计算。
- 动态特征和 静态 特征  都是全局的还是局部的，还是混合的？
- HybridBlock 和 Predictor的 区别是什么？
    - A `Predictor` wrapping a `HybridBlock` used for inference.
- 既然有了预测网络，为什么还要定义predictor对象？
    - `Predictor`对象 == `Transformation`   + 预测网络 
- `Predictor`输出概率分布的参数 还是 采样的结果？
    - 采样结果, btw 概率分布是中间变量
- MyPredNetwork.predict()可以处理多个样本吗？
    - 看PredNetwork的hybrid_ward 的 注释是可以的呀，都支持输入batch的数据了，之前怎么想的
```python
class DeepARPredictionNetwork(DeepARNetwork):
    
	def hybrid_forward(
        self,
        F,
        feat_static_cat: Tensor,  # (batch_size, num_features)
        feat_static_real: Tensor,  # (batch_size, num_features)
        past_time_feat: Tensor,  # (batch_size, history_length, num_features)
        past_target: Tensor,  # (batch_size, history_length, *target_shape)
        past_observed_values: Tensor,  # (batch_size, history_length, *target_shape)
        future_time_feat: Tensor,  # (batch_size, prediction_length, num_features)
    ) -> Tensor:
        """
        Predicts samples, all tensors should have NTC layout.
        Parameters
        ----------
        F
        feat_static_cat : (batch_size, num_features)
        feat_static_real : (batch_size, num_features)
        past_time_feat : (batch_size, history_length, num_features)
        past_target : (batch_size, history_length, *target_shape)
        past_observed_values : (batch_size, history_length, *target_shape)
        future_time_feat : (batch_size, prediction_length, num_features)

        Returns
        -------
        Tensor
            Predicted samples
        """
```
    - spark MLlib是不是这么做的还要再确认下
    - 最底层的rnn.unroll层是支持的
```python
# output shape: (batch_size * num_samples, 1, num_cells)
# state shape: (batch_size * num_samples, num_cells)
rnn_outputs, repeated_states = self.rnn.unroll(
    inputs=decoder_input,
    length=1,
    begin_state=repeated_states,
    layout="NTC",
    merge_outputs=True,
)
### decoder_input： <NDArray 100x1x49 @cpu(0)>
### rnn_outputs： <NDArray 100x1x40 @cpu(0)>
### repeated_states： <NDArray 100x40 @cpu(0)>]
```

- model_analysis（画图） 和  evaluate （`make_evaluation_predictions`来计算评估指标的函数），对应的不是一个预测时间段， 导致指标 和 图对应不起来
    - 真正评估的时候以model_analysis为准
    - dataflow上面调试的时候以`make_evaluation_predictions`这种方式为准
- `make_evaluation_predictions`内部的逻辑是什么样的？它的预测 和 预测网络的预测有什么不一样？--时间段不一样
    - 以回测为目的的预测：就是说 它一定要保证评估有充足的数据（最尾上的长度为 预测区间 的一截）；坏处是，会存在模型预测的输入不够的情况（eg 测试数据长度为future_size, 结果全被拿去做 评估集了，模型预测输入就没了，当然模型会padding为0）

```python
def make_evaluation_predictions(
    dataset: Dataset, predictor: Predictor, num_samples: int
) -> Tuple[Iterator[Forecast], Iterator[pd.Series]]:
    """
    Return predictions on the last portion of predict_length time units of the
    target. 
    Such portion is cut before making predictions, 
    such a function can be used in evaluations where accuracy is evaluated on the last portion of
    the target.

    Parameters
    ----------
    dataset
        Dataset where the evaluation will happen. Only the portion excluding
        the prediction_length portion is used when making prediction.
    predictor
        Model used to draw predictions.
    num_samples
        Number of samples to draw on the model when evaluating.

    Returns
    -------
    """
    prediction_length = predictor.prediction_length
    freq = predictor.freq
    
    #lead_time: 预测跳点，预测的时间段为lead_time ～ prediction_length + lead_time
    lead_time = predictor.lead_time

    def add_ts_dataframe(
        data_iterator: Iterator[DataEntry],
    ) -> Iterator[DataEntry]:
        for data_entry in data_iterator:
            data = data_entry.copy()
            index = pd.date_range(
                start=data["start"],
                freq=freq,
                periods=data["target"].shape[-1],
            )
            data["ts"] = pd.DataFrame(
                index=index, data=data["target"].transpose()
            )
            yield data

    def ts_iter(dataset: Dataset) -> pd.DataFrame:
        for data_entry in add_ts_dataframe(iter(dataset)):
            yield data_entry["ts"]

    # 将target序列截断： 
    #     扔掉最后的prediction_length + lead_time条记录        
    def truncate_target(data):
        data = data.copy()
        target = data["target"]
        assert (
            target.shape[-1] >= prediction_length
        )  # handles multivariate case (target_dim, history_length)
        data["target"] = target[..., : -prediction_length - lead_time]
        return data

    # TODO filter out time series with target shorter than prediction length
    # TODO or fix the evaluator so it supports missing values instead (all
    # TODO the test set may be gone otherwise with such a filtering)
	## step 1 - 使用truncate_target 截断测试集中 最后一个“future”窗口的值
    ##          并使用AdhocTransform将truncate_target逻辑映射到每一个序列数据上
    dataset_trunc = TransformedDataset(
        dataset, transformations=[transform.AdhocTransform(truncate_target)]
    )
    ## step 2 - 使用predictor的predict方法 对截断后剩下的序列 做预测，以sample_path的格式返回预测值
	## step 3 - 输出dataset的pd.DataFrame格式
    return (
        predictor.predict(dataset_trunc, num_samples=num_samples),
        ts_iter(dataset),
    )
```


### forcast是什么？`forcast.mean`形状是什么？

```javascript
forecast.mean.shape
Out[25]: (168,)
```

### 如果只做预测 不做评估要怎么搞？

```python
# step 1： 将context长度的测试数据输入predictor.predict()
forecast = predictor.predict(test_data, num_samples=10)
forecast_list = list(forecast)
# step 2： 获取多个序列的输出（forecast对象包含了这些信息），
# 通过 forecast_list[0].samples 获取到 对每个序列的 预测值，也就是sample_path
# sample_path的形状为(采样个数，预测窗口长度)
forecast_list[0].samples.shape
   ...: 
Out[35]: (10, 168)
```
- 特征模块（gluonts.transform.feature module）有什么功能？
    - **AddAgeFeature**：Adds an ‘age’ feature to the data_entry.
    - **AddConstFeature**：将一个常数扩充到时间轴上
        - If is_train=True，the feature matrix has the same length as the target field. If is_train=False，the feature matrix has length len(target) + pred_length
    - **AddObservedValuesIndicator**：将缺失值用dummy值替换，增加一个index变量，区分有值 和 没有值 的情况。
    - **AddTimeFeatures**：
        - Ifis_train=True the feature matrix has the same length as the target field. If is_train=False the feature matrix has length len(target) + pred_length
    - **target_transformation_length**：
        - If is_train=True，return len(target) 
        -  If is_train=False, eturn len(target) + pred_length

## Estimator，GluonEstimator，DeepAREstimator三者的关系

> update 10月26 日-- 重新理解Estimator，GluonEstimator，DeepAREstimator三者的关系

- 本质 是 通过定义抽象类  以及  规范 来 实现 应用时候 的 可扩展性。
- Estimator 是各种Estimator的最本质的功能的抽象 定义成的类，即”训练“（train方法），拿到训练数据和验证数据（optional），返回__Predictor__。
- GluonEstimator 和 DeepAREstimator 都是在这个 框架下，功能的 细化 和 扩展。
- Estimator的 直属子类有 DummyEstimator 和 GluonEstimator，DummyEstimator 的功能就是在训练的时候 直接返回一个预先构造好的__Predictor__ 。
- Estimator的另外一个直属子类 GluonEstimator 的定义就规范多了，是一个标准的完整的算法类，参考：
    - 父类（gluonts.model.GluonEstimator）：
        - create_transformation: 抽象方法
        - create_training_network：抽象方法
        - create_predictor：抽象方法
        - train_model： 执行训练，返回TrainOutput, 依次调用：
            - self.create_transformation()： 返回`transformation`
            - self.create_training_network()：返回`trained_net`
            - self.trainer()：```trainer = Trainer(epochs=epochs, batch_size=batch_size)```
            - self.create_predictor(): 输入`transformation`和 `trained_net`， 返回`predictor`，```self.create_predictor(transformation, trained_net)```
        - train：执行训练，返回TrainOutput.predictor
            - self.train_model()： 
### 为什么要定义这两层抽象类呢（Estimator 和 GluonEstimator 都是抽象类，实际执行的是的DeepAREstimator）？

- **我的猜测是，很多组件 是 对 某一组对象生效的，这两层的抽象 实际上是在 定义“组”，以及 对 每一组成员的行为 制定规范（通过抽象方法）。**
- 设想一个场景，现在要实现一个逻辑，调用10个实际类的fit方法（DeepAREstimator和 它的同伴们），注意！ 在执行之前，我是不知道具体要调用哪个类的fit的：
    - 不抽象的话 需要写 10次处理逻辑：遍历10个类，写10 个if else来 调用某个方法。（后面增加到11类，这个if else又要改）
    - 抽象的话 就只需要写1次处理逻辑： 调用抽象类，以及抽象类的方法。
- 而如果这个逻辑 要实现N次，按照不抽象的写法，就要写10 * N 次  if else。



## 自定义一个点预测模型（从系统 底层 到 顶层，or从 里层 到 外层 ）：

### 第1步：定义训练网络 和 预测网络 

- 训练网络 和 预测网络 都要实现hybrid_forward方法，该方法就是在定义训练网络 和 预测网络被调用时的逻辑
    - 训练网络 的hybrid_forward方法 应该返回  预测值 和 真实值 的loss
    - 预测网络 的hybrid_forward 返回 预测值
- 构造一个训练网络：
    
```python
from gluonts.model.estimator import GluonEstimator
from gluonts.model.predictor import Predictor, RepresentableBlockPredictor
from gluonts.core.component import validated
from gluonts.support.util import copy_parameters
from gluonts.transform import ExpectedNumInstanceSampler, Transformation, InstanceSplitter
from gluonts.dataset.field_names import FieldName
from mxnet.gluon import HybridBlock

class MyTrainNetwork(gluon.HybridBlock):
    def __init__(self, prediction_length, **kwargs):
        super().__init__(**kwargs)
        self.prediction_length = prediction_length

        with self.name_scope():
            # Set up a 3 layer neural network that directly predicts the target values
            self.nn = mx.gluon.nn.HybridSequential()
            self.nn.add(mx.gluon.nn.Dense(units=40, activation='relu'))
            self.nn.add(mx.gluon.nn.Dense(units=40, activation='relu'))
            self.nn.add(mx.gluon.nn.Dense(units=self.prediction_length, activation='softrelu'))

    def hybrid_forward(self, F, past_target, future_target):
        prediction = self.nn(past_target)
        # calculate L1 loss with the future_target to learn the median
        return (prediction - future_target).abs().mean(axis=-1)
```

- 内嵌网络（真实的神经网络，是训练网络的一个成员，self.nn）的输入输出：
    - 输入：past value
    - 输出：prediction，长度为prediction_length
- hybrid_forward函数的输入输出：
    - 输入：past target，future target
    - 输出：prediction（self.nn(past_target)） 和 future target的 l1距离
- 代码示范
- 构造一个预测网络：
- 网络结构应该和训练网络保持一致（通过继承inheriting 训练网络类 来做到）
- 网络的输入输出：
    - 输入：past target
    - 输出：prediction, 长度为prediction_length
- hybrid_forward的输入输出：
    - 输入：past_target
    - 输出：prediction, 长度为prediction_length
    
```python
class MyPredNetwork(MyTrainNetwork):
    # The prediction network only receives past_target and returns predictions
    def hybrid_forward(self, F, past_target):
        prediction = self.nn(past_target)
        return prediction.expand_dims(axis=1)
```

###  第2步：构造一个estimator，须实现3个方法：

- create_transformation方法：包括特征转换逻辑 和 训练时的数据切分方式
    - 返回`Transformation`  对象
- create_training_network方法：返回训练网络，也会把超参存下来
    -  返回`trained_network`对象
- create_predictor方法：创建预测网络，并返回一个Predictor对象
    - 返回`Predictor`对象：定义了predict方法，transformer + 预测网络 。

```python
class MyEstimator(GluonEstimator):
    @validated()
    def __init__(
        self,
        freq: str,
        context_length: int,
        prediction_length: int,
        trainer: Trainer = Trainer()
    ) -> None:
        super().__init__(trainer=trainer)
        self.context_length = context_length
        self.prediction_length = prediction_length
        self.freq = freq

    def create_transformation(self):
        # Feature transformation that the model uses for input.
        # Here we use a transformation that randomly select training samples from all time series.
        return InstanceSplitter(
                    target_field=FieldName.TARGET,
                    is_pad_field=FieldName.IS_PAD,
                    start_field=FieldName.START,
                    forecast_start_field=FieldName.FORECAST_START,
                    train_sampler=ExpectedNumInstanceSampler(num_instances=1),
                    past_length=self.context_length,
                    future_length=self.prediction_length,
                )

    def create_training_network(self) -> MyTrainNetwork:
        return MyTrainNetwork(
            prediction_length=self.prediction_length
        )

    def create_predictor(
        self, transformation: Transformation, trained_network: HybridBlock
    ) -> Predictor:
        prediction_network = MyPredNetwork(
            prediction_length=self.prediction_length
        )

        copy_parameters(trained_network, prediction_network)

        return RepresentableBlockPredictor(
            input_transform=transformation,
            prediction_net=prediction_network,
            batch_size=self.trainer.batch_size,
            freq=self.freq,
            prediction_length=self.prediction_length,
            ctx=self.trainer.ctx,
        )
```
   
###  第3步：训练estimator，得到predictor

```python
estimator = MyEstimator(
    prediction_length=dataset.metadata.prediction_length,
    context_length=100,
    freq=dataset.metadata.freq,
    trainer=Trainer(ctx="cpu",
                    epochs=5,
                    learning_rate=1e-3,
                    num_batches_per_epoch=100
                   )
)
predictor = estimator.train(dataset.train)
```

        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FhvZeUsSk1L.png?alt=media&token=234da86d-ad2b-4b58-a191-d8d5ccba5c82)

### 第4步 ：创建forecasts

- `make_evaluation_predictions` 函数会调用`Predictor`的predict方法，从而获取预测值

```python
forecast_it, ts_it = make_evaluation_predictions(
    dataset=dataset.test,
    predictor=predictor,
    num_samples=100)

forecasts = list(forecast_it)
tss = list(ts_it)

## tss 是最完整的曲线，forecasts只有后面一截
plot_prob_forecasts(tss[0], forecasts[0])
```
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FmPC-fmVxIs.png?alt=media&token=f9b7f8c0-2717-47f7-930a-cc76a30d8855)
- 这里的num_samples设置为100，就会获得100次相同的预测结果（点预测没有随机性）

###  第5步：评估forecasts

```python
evaluator = Evaluator(quantiles=[0.1, 0.5, 0.9])
agg_metrics, item_metrics = evaluator(iter(tss), iter(forecasts), num_series=len(dataset.test))
```
返回两部分： 整体指标 以及 每个商品指标
    - 整体指标（agg_metrics）
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F8Yt16rezmJ.png?alt=media&token=68cbc539-27a7-4edc-ab76-20606b9a85b8)
    - 每个商品评估指标（item_metrics）
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FxEvAhUsdiL.png?alt=media&token=a3c18b94-a391-4009-ada4-ff57c1976c6d)


## 使用过程中的bug

- ujson dumps NaN but doesn't load it 3511，https://github.com/micropython/micropython/issues/3511
```python
def serialize_data_entry(data_entry: DataEntry) -> Dict:
    """
    Encode the numpy values in the a DataEntry dictionary into lists so the
    dictionary can be json serialized.
    """
    def serialize_field(field):
        if isinstance(field, np.ndarray):
            # circumvent https://github.com/micropython/micropython/issues/3511
            nan_ix = np.isnan(field)
            field = field.astype(np.object_)
            field[nan_ix] = "NaN"
            return field.tolist()
        return str(field)

    return {
        k: serialize_field(v)
        for k, v in data_entry.data.items()
        if v is not None
    }
```
使用simplednn的时候，报错

```javascript
GluonTSUserError: Got NaN in first epoch.
Try reducing initial learning rate.
```
- 原因1：target中含有nan值，有的算法处理了，有的算法没有处理，deepAR处理了的.
    - solution： 清洗target中的nan。
- 原因2: gluonts的容错机制不行，第一个epho挂了。。不知道为啥, best_epoch_no 为-1，就报错了
    - 解决方案：epoch从1-->2+
- 原因3: 可能是随机种子设置的不好，重新跑一遍就好了。。。无语

### 测试多个算法

包括"transformer", "gp",  "deepfactor", "deepar", "dnn", "prophet", "deepstate","wavenet"

#### 已通过

- deepar：效果很稳定
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FFq-O8QcOd2.png?alt=media&token=0a7c6572-919f-4b1d-92e7-76fb03bd5298)
- prophet： 效果很稳定，小数据量时 效果比deepar好。
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F1dIVCi8Mpg.png?alt=media&token=7d838f06-5a69-49ed-8f2f-ff6b7d240a5e)
- deepfactor： 效果很不稳定
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fx642RB3BnL.png?alt=media&token=ae3bce56-8d23-451f-9d42-2ef0cb61282a)
- transformer： 训练时间过长5min，训练集才1400个点
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FqKMkZR5D1f.png?alt=media&token=cfa9cf7f-5285-463a-a02f-f088a67b80c0)
- dnn：效果一般，不推荐。应该就是裸跑dnn
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FV9lPxmzhet.png?alt=media&token=3e05ec82-c5d7-4499-84f1-21a38a529661)

#### 未通过

- deepstate： 训练报错
    ```python
    value = data[self.field]
    KeyError: 'feat_static_cat'
    ```
- wavenet：训练时间过长
- gp：输出怎么是贴零的？

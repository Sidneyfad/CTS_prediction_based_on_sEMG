基于表面肌电信号的腕管综合征诊断
===

腕管综合征 (Carpal tunnel syndrome, CTS) 是由于腕管中的正中神经受到压迫导致的一系列症状和体征。 典型症状包括：拇指和桡侧手指为主的麻木和刺痛、掌侧腕部和前臂掌侧酸痛和疼痛以及手部笨拙。

肌电图（EMG）是首选的诊断检测，有助于诊断腕管综合征、排除或识别其他神经学诊断、评估正中神经损伤的严重程度、有助于选择治疗方法、判断干预的成功性、确定预后，并可作为随后参照的基准。

肌电图是将针电极插入肌肉记录电位变化的一种电生理检查，相比针极肌电图，表面肌电图（sEMG）的采集过程成本更低且给病人带来的痛苦更小，但由于肌电信号本身是一种较微弱的电信号，加之皮肤和组织对肌电均有衰减作用, 因此在皮肤表面记录的表面肌电信号比针电极记录的信号更微弱, 也更易受干扰影响。

本项目旨在将机器学习技术和表面肌电信号数据相结合，来达到较为准确地诊断腕管综合征的目标。

## 数据集简介

本项目中的实验数据包括病例样本和正常样本。病例样本以临床诊断中的病人作为受试者，采集方法分为两种：自由对掌和维持对掌；正常样本以未患病的人作为受试者，采集方法均采用自由对掌。所有样本的采集均使用相同设备，使用两个电极（即双通道）对受试者进行表面肌电信号的采集，采样率为1000，每个受试者重复对掌动作5次，作为一个样本。

目前数据集存在的问题有：

* 样本数量较少。可能使得模型难以学习到对疾病诊断有用的特征信息。
* 样本采集流程不够规范。首先是采集流程不一致，自由对掌和维持对掌得到的样本是否可以混合使用尚不明确；其次是动作分割问题，目前是让受试者连续做5次动作来作为一个样本，更加科学的形式应该是把受试者的一次动作作为一个样本，并固定这一次动作的采集时长。
* 正常样本相比病例样本数量较少，且数据质量较差。

## 数据预处理（去噪）

由于表面肌电信号较为微弱，在采集过程中会受到各种信号源的干扰，包括工频干扰、其它生理信号干扰以及仪器本身的干扰。因此在使用表面肌电信号进行腕管综合征的诊断时，我们需要先对原始信号数据进行去噪处理。

采用Butterworth数字滤波器对信号进行去噪，过滤掉低于20HZ的低频噪声和高于250HZ的高频噪声。此外，经计算，50Hz及倍频（n*50Hz)的功率值之和占总功率很小，因此认为信号中的工频干扰很微小，为避免陷波引起的肌电信号本身信息丢失，我们这里不对信号进行陷波处理。

## 特征提取

常用的表面肌电特征分为时域特征和频域特征，如下表所示：

<br>

| 时域特征  | 频域特征 |
| ------------- | ------------- |
| Zero corssing  | Mean frequency  |
| Slope sign change  | Median frequency  |
| Wilson amplitude  | Peak frequency  |
| Waveform length  |  |
| Integrated EMG  |   |
| Mean absolute value  |  |
| Variance  |  |
| Root-mean-square  |  |

<br>

对提取到的双通道表面肌电信号分别提取表中的11个常用特征并使用各经典机器学习分类器进行实验，实验表明随机森林的分类效果最好：

<br>

| Method  | accuracy |
| ------------- | ------------- |
| KNN  | 87.1% |
| SVM  | 55.2%  |
| RF  | 88.8%  |
| MLP  | 60.0% |

<br> 

上述特征可视为依赖专家知识的高阶特征，这些特征虽被用于很多的表面肌电信号应用场景，但未必适用于腕管综合征的诊断，因此我们希望提取依赖较少专家知识的低阶特征，并让模型自动从低阶特征中提取出对诊断有用的高阶特征。

采用的特征提取方法为计算功率谱密度（Power Spectral Density, PSD）和局部二值模式（Local Binary Pattern, LBP）直方图。

功率谱密度，实际就是通过一定方法求解信号的功率随频率变化的曲线。我们采用 Welch 方法[34] 来计算功率谱密度，Welch 方法的思路是：先把信号分成多段（各段之间可以有重叠），然后把窗口函数（Hamming、Hanning 等）加到每一段信号上，求出每一段信号的功率谱，最后对每段数据的功率谱进行平均，得到整个信号的功率谱。相比传统的快速傅里叶变换（FFT）方法，Welch 方法的分辨率更高，在脑电信号、眼电信号、肌电信号等生物信号的处理中有着广泛的应用。功率谱密度表示了信号功率随着频率的变化情况，或者说信号功率在频域的分布状况，因此可以将其视作信号的频域特征。

下面给出了一个原始表面肌电信号和对应功率谱密度的示意图：

<br>

![image](https://github.com/Sidneyfad/CTS_prediction_based_on_sEMG/blob/main/images/psd.png)

<br>

局部二值模式简单来说就是通过将每个像素的相邻像素与该像素值进行比较，并将其作为二进制编码来表示局部纹理。由于 LBP 在很多场景的应用中都有良好的表现，尤其是在计算机视觉和图像处理领域，因此得到了研究人员的广泛关注。LBP 通过对各中心像素与相邻像素进行简单的计算来刻画图像中存在的纹理模式。计算的结果通常被汇总成直方图形式，作为刻画图像纹理的特征使用。由于二维 LBP 在获取图像基本纹理特征方面计算简单且效果好，因此 LBP 的一维版本（简称 1DLBP）近年来也被用于一维信号的特征提取。

对于一维的信号数据，我们将中心振幅值与左右各 4 个共 8 个 (也可以是其它数值，通常默认值为 8，与二维的情形一致) 相邻振幅值进行比较，若某个相邻振幅值小于中心振幅值，则该位置记为 0；若某个相邻振幅值大于中心振幅值，则该位置记为 1。将上述 0、1 组成的二进制数转化为 0～255 的十进制数作为该位置的特征，遍历整个信号数据后我们对所有特征值进行计数就可以得到一个直方图，最终我们把得到的直方图（256 维）作为当前信号数据的特征向量，就完成了特征提取过程。1DLBP 提取特征的过程如图所示：

<br>

![image](https://github.com/Sidneyfad/CTS_prediction_based_on_sEMG/blob/main/images/sEMG_LBP.png)

<br>

对提取到的双通道表面肌电信号分别提取PSD和LBP特征（共770维）并使用各经典机器学习分类器进行实验，实验表明依旧是随机森林的分类效果最好：

<br>

| Method  | accuracy |
| ------------- | ------------- |
| KNN  | 84.2% |
| SVM  | 50.1%  |
| RF  | 91.6%  |
| MLP  | 45.1% |

<br> 

## 深度森林

深度森林是一种受深度学习和集成学习的特点启发，将集成方法与深度神经网络的框架相结合，既具备深度神经网络学习能力和表征能力强的优点，又具备集成学习方法易于训练、有效防止过拟合的优点。其框架图如下：

<br>

<div align=center>![image](https://github.com/Sidneyfad/CTS_prediction_based_on_sEMG/blob/main/images/级联森林.png)</div>

<br>

深度森林已被验证在基于表面肌电信号的手势分类中具有出色的表现，而在腕管综合征的诊断这一场景中和随机森林的表现差异不大，这可能是由于数据量过小和数据质量不佳导致的。可以推测，当有足够数量的高质量数据时，深度森林的表现会超过随机森林，从而达到可以在应用中指导医疗实践的效果。

为了使得深度森林更加适用于解决高维小样本数据上的多分类问题，我基于深度森林做了几点改进，这里给出论文和代码实现。

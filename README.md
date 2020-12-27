基于表面肌电信号的腕管综合征诊断
===

腕管综合征 (Carpal tunnel syndrome, CTS) 是由于腕管中的正中神经受到压迫导致的一系列症状和体征。 典型症状包括：拇指和桡侧手指为主的麻木和刺痛、掌侧腕部和前臂掌侧酸痛和疼痛以及手部笨拙。

肌电图（EMG）是首选的诊断检测，有助于诊断腕管综合征、排除或识别其他神经学诊断、评估正中神经损伤的严重程度、有助于选择治疗方法、判断干预的成功性、确定预后，并可作为随后参照的基准。

肌电图是将针电极插入肌肉记录电位变化的一种电生理检查，相比针极肌电图，表面肌电图（sEMG）的采集过程成本更低且给病人带来的痛苦更小，但由于肌电信号本身是一种较微弱的电信号，加之皮肤和组织对肌电均有衰减作用, 因此在皮肤表面记录的表面肌电信号比针电极记录的信号更微弱, 也更易受干扰影响。

本项目旨在将机器学习技术和表面肌电信号数据相结合，来达到较为准确地诊断腕管综合征的目标。

## 数据集简介

本项目中的实验数据包括病例样本和正常样本。病例样本以临床诊断中的病人作为受试者，采集方法分为两种：自由对掌和维持对掌。其中2019年采集的病例样本均使用自由对掌（共50例），2020年采集的病例样本均使用维持对掌（共26例）；正常样本以未患病的人作为受试者，采集方法均采用自由对掌（共21例）。所有样本的采集均使用相同设备，使用两个电极（即双通道）对受试者进行表面肌电信号的采集，采样率为1000，每个受试者重复对掌动作5次，作为一个样本。

目前数据集存在的问题有：

* 样本数量较少。可能使得模型难以学习到对疾病诊断有用的特征信息。
* 样本采集流程不够规范。首先是采集流程不一致，自由对掌和维持对掌得到的样本是否可以混合使用尚不明确；其次是动作分割问题，目前是让受试者连续做5次动作来作为一个样本，更加科学的形式应该是把受试者的一次动作作为一个样本，并固定这一次动作的采集时长。
* 正常样本相比病例样本数量较少，且数据质量较差。

## 数据预处理（去噪）

由于表面肌电信号较为微弱，在采集过程中会受到各种信号源的干扰，包括工频干扰、其它生理信号干扰以及仪器本身的干扰。因此在使用表面肌电信号进行腕管综合征的诊断时，我们需要先对原始信号数据进行去噪处理。

采用Butterworth数字滤波器对信号进行去噪，过滤掉低于20HZ的低频噪声和高于250HZ的高频噪声。此外，经计算，50Hz及倍频（n*50Hz)的功率值之和占总功率很小，因此认为信号中的工频干扰很微小，为避免陷波引起的肌电信号本身信息丢失，我们这里不对信号进行陷波处理。

## 特征提取

常用的表面肌电特征分为时域特征和频域特征，如下表所示：

| Attempt | #1 | #2 |
| :---: | :---: | :---: |
| Seconds | 301 | 283 |

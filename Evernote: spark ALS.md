Title: Spark ALS最小二乘法
Tags: spark,算法
Notebook: 技术相关

[TOC]

# 相关指标
+ ROC 反应了抽样的方案对各种质量的产品的辨识能力
+ AUC 等于ROC曲线下面的区域面积，是随机选择的好推荐比岁间选择的差推荐的排名高的概率

AUC指标也用于评分分类器， MLlib的BinaryClassificationMetrics 类实现了这个指标
其他的评分系统相关的评价指标在RankingMetrics类中实现， 这些指标包括准确率，召回率和平均准确率(Mean Average Precision, MAP), MAP更强调排在最前面的推荐的质量

通常数据被分为三个子集: 训练集， 检验(CV)集， 测试集

---
title: Spark MLlib学习笔记-分类和回归
notebook: 技术相关
tags:spark
---
spark.mllib  提供了 二分类，多分类和回归分析的方法。 

| Problem Type        | Supported Methods           |
| ------------- |:-------------:|
| 二分类     | 线性svm, 逻辑回归, 决策树, 随机森林, gradient-boosted树， 朴素贝叶斯 |
| 多分类      | 逻辑回归，决策树，随机森林，朴素贝叶斯      |
| 回归 | 线性最小二乘法， Lasso，岭回归, 决策树, 随机森林, gradient-boosted树, 保序回归    |


+ [线性模型](http://spark.apache.org/docs/1.6.0/mllib-linear-methods.html)
	* [分类(svm, 逻辑回归)](http://spark.apache.org/docs/1.6.0/mllib-linear-methods.html#classification)
	* [线性回归(最小二乘法， Lasso，岭回归)](http://spark.apache.org/docs/1.6.0/mllib-linear-methods.html#linear-least-squares-lasso-and-ridge-regression)
+ [决策树](http://spark.apache.org/docs/1.6.0/mllib-decision-tree.html)
+ [Ensembles of decision trees](http://spark.apache.org/docs/1.6.0/mllib-ensembles.html)
	* [随机森林](http://spark.apache.org/docs/1.6.0/mllib-ensembles.html#random-forests)
	* [gradient-boosted树](http://spark.apache.org/docs/1.6.0/mllib-ensembles.html#gradient-boosted-trees-gbts)
+ [朴素贝叶斯](http://spark.apache.org/docs/1.6.0/mllib-naive-bayes.html)
+ [保序回归](http://spark.apache.org/docs/1.6.0/mllib-isotonic-regression.html)
# 裸特征 0.868! 2021 CCF 个贷违约预测赛道 baseline

赛道链接: https://www.datafountain.cn/competitions/530

## 碎碎念

典型的风控赛题，就不用多做介绍啦

赛方提到了迁移学习，提供了 train_public 和 train_internet_public 两个表

个人感觉是可以拼起来直接做的，至于为啥叫迁移学习，没搞懂

## 思路

本 baseline 主要是看下两个表一起是不是比单表分数要高

- 只使用两表共有的字段；
- 做了点数据整理；
- 未做特征工程，留给各位大佬发挥；
- 五折 LGB

## 线上

线下比较低分：0.806，但是线上却有 0.86832636137，据说是比裸特征 train_public 单表(0.85X) 分数要好。
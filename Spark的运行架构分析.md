---
title: Spark的运行架构分析
weblog_mt_keywords: "spark,运行架构"
---
[toc]
# 架构概述
## 一：Spark的运行模式
# 运行模式详解
## 一：Spark On Local
此种模式下，我们只需要在安装Spark时不进行hadoop和Yarn的环境配置，只要将Spark包解压即可使用，运行时Spark目录下的bin目录执行bin/spark-shell即可
具体可参考这篇[博客](https://blog.csdn.net/happyanger6/article/details/47070223)

## 二：Spark On Local Cluster（Spark Standalone）

即Spark的伪分布模式，安装部署可参考：http://blog.csdn.net/gamer_gyt/article/details/51638023

Standalone模式是Spark实现的资源调度框架，其主要的节点有Client节点、Master节点和Worker节点。其中Driver既可以运行在Master节点上中，也可以运行在本地Client端。当用spark-shell交互式工具提交Spark的Job时，Driver在Master节点上运行；当使用spark-submit工具提交Job或者在Eclips、IDEA等开发平台上使用”new SparkConf.setManager(“spark://master:7077”)”方式运行Spark任务时，Driver是运行在本地Client端上的。

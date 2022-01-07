---
title: "数值优化"
subtitle: ""
description: 
date: 2022-01-07T14:58:42+08:00
lastmod: 2022-01-07T14:58:42+08:00

comments: true
hidden: false
draft: true

image: 
tags: []
categories: []
---
本文主要介绍了基本的数值优化原理和方案。

<!--more-->

## 优化问题基本概念
优化问题主要包含三个部分：
1. 目标函数，$f(x)$，该函数用于最大化或最小化。
2. 决策变量
3. 约束条件，如等式约束或不等式约束。   
数学形式如下：
$$
\begin{array}{cl}
\underset{x \in \mathbb{R}^{n}}{\operatorname{minimize}} & f(x) \\
\text { subject to } & g(x)=0, \\
& h(x) \geq 0 .
\end{array}
$$
上式中，$f: \mathbb{R}^{n} \rightarrow \mathbb{R}, g: \mathbb{R}^{n} \rightarrow \mathbb{R}^{p}, h: \mathbb{R}^{n} \rightarrow \mathbb{R}^{q}$，通常被认为是可微的。


## 无约束优化和牛顿型方法

## 等式约束优化

## 不等式约束优化
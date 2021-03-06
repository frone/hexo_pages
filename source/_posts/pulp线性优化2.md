---
title: PulP线性优化（二）优化概念 
date: 2018-08-03 16:23:51
tags:
    - PULP
    - 线性优化
categories: 算法
---
*本文根据PuLP文档翻译而来，原文请参考*
*https://pythonhosted.org/PuLP/main/optimisation_concepts.html*

### 线性编程
___
最简单的数学程序类型是线性程序。要使您的数学程序成为线性程序，您需要满足以下条件：

- 决策变量必须是实变量;
- 目标必须是线性表达;
- 约束必须是线性表达式。

线性表达式是以下形式的表达式

$$ a_1x_1 + a_2x_2 + a_3x_3 + ... a_nx_n\{<=,=,>=\}b $$

其中$a_i$和b是已知的常数$x_i$是变量。求解线性程序的过程称为线性编程。线性编程通过修订的单纯形法（也称为原始单纯形法），双单纯形法或内点法进行。像cplex这样的解算器允许您指定使用哪种方法，但我们在此不再详述。

### 整数编程
___
整数程序几乎与线性程序相同，但有一个非常重要的例外。整数程序中的某些决策变量可能只需要包含整数值。变量称为整数变量。由于大多数整数程序包含连续变量和整数变量的混合，因此它们通常称为混合整数程序。虽然线性编程的变化很小，但对解决方案过程的影响是巨大的。整数程序可能是非常难以解决的问题，并且目前有许多研究发现解决整数程序的“好”方法。可以使用分支绑定过程来解决整数程序。

注意对于任何合理大小的MIP，解决方案时间随着整数变量数量的增加呈指数增长。
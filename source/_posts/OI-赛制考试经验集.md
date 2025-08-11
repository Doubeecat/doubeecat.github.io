---
title: OI 赛制考试经验集
categories: 学习笔记
tags: []
date: 2022-08-28 15:30:21
---
多谢梅花，伴我微吟。

<!--more-->

0. **永远不要放弃。无论何时。**

1. 在开场的时候，应先用 30 min 以内的时间读完所有题目，对每个题有一个初步的认知。

2. 优先找出签到题以及自己最有感觉的题，首先先用 40 min 以内时间搞定签到题并拍上。根据另外一题难度决定策略。

3. 如果另一题较简单，那么考虑在 1 h 30 min 内切掉并拍上这个题。**一个做法只有在过拍的情况下才可以无脑交上去。否则必须数据分治**

4. 暴力分数+特殊性质启发正解思考。

5. 观察答案是否超出了 `int` 范围，如果超出了记得**只对需要的数组开 `ll`**

6. $n \leq 10,\mathcal{O}(n!)$

   $n \leq 20,\mathcal{O}(2^n)$

   $n \leq 500,\mathcal{O}(n^3)$

   $n \leq 5000,\mathcal{O}(n^2)/\mathcal{O}(n^2\log n)$

   $n \leq 10^5,\mathcal{O}(n\log ^2 n)/\mathcal{O}(n\log n)$

   $n \leq 5\times 10^5,\mathcal{O}(n\log n)/\mathcal{O}(n \log^2 n)$ （小常数）

   $n \leq 10^7,\mathcal{O}(n)$

   $n \leq 10^{12},\mathcal{O}(\sqrt n)$

   $n \leq 10^{18},\mathcal{O}(1)/\mathcal{O}(\log n)$

7. 一定要保证思考的做法写的出来。

8. 在 7 的前提下，如果碰到一个思维难度高的题，想想能不能通过上数据结构优化掉一部分思考量。


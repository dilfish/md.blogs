---
title: "方向导数思考"
date: 2023-09-24T17:29:54+08:00
draft: false
---

https://m.youtube.com/watch?v=GJODOGq7cAY

https://betterexplained.com/articles/understanding-pythagorean-distance-and-the-gradient/

![如图](/pics/directional.dr.png)

在上面的推导和解释中有一个核心因素，就是 x y 两个分量是独立变化的。

因此选用 cos(theta) 和 sin(theta) 是为了在其夹角的方向上任意选择，同时，因为平方和总是1，也符合导数极限的定义。

因此梯度的方向就比较自然，因为 x y 独立变化，而方向的定义要求平方和为1。方向导数最大的时候，方向与偏导数也就是梯度同向，此时 cos(theta) 为1，最大值等于梯度的长度。

同向可以理解为，方向向量的分量和偏导数成比例。即例如 x 偏导为3，而 y 偏导为 2，则梯度的向量就是(3，2)，最大方向导数为 √(3²+2²)。因为 cos 和 sin 的平方和要为1，所以真正使用的方向向量是归一化之后的 (3，2)/√13。

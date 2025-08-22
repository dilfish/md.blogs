---
title: "方向导数思考"
date: 2023-09-24T17:29:54+08:00
draft: false
---

https://m.youtube.com/watch?v=GJODOGq7cAY

https://betterexplained.com/articles/understanding-pythagorean-distance-and-the-gradient/

![如图](/pics/directional.dr.png)

在上面的推倒和解释中有一个核心因素，就是 x y 两个分量是独立变化的。

因此选用 cos(theta) 和 sin(theta) 是为了在其夹角的方向上任意选择，同时，因为平方和总是1，也符合导数极限的定义。

因此梯度的方向就比较自然，因为 x y 独立变化，而导数的定义要求平方和为1。所以他们值最大的时候就是和偏导数方向垂直。此时 cos(theta) 为1。

垂直可以理解为，两个向量的分量互换。即例如 x 偏导为3，而 y 偏导为 2，则梯度的向量应该是(2，3)，因为x 方向变化速度快，所以给他一个较小的系数，y 方向较大的系数。这样可以使得他们组成的向量的长度最大。

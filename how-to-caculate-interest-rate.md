---
title: "如何计算利率"
date: 2022-06-07T13:28:24+08:00
draft: false
---

最近我偶然看到招行信用卡 app 里面有个叫 e 招贷的东西，号称年化利率低至3.3。我对这个计算方法不太了解，也不知道是不是真的。

于是问了群友[^1]，测试了几个样例[^2]，群友说 excel 有个 irr 的函数，刚好我知道 python 也有个这样的函数。文档在这里 [numpy_financial.irr](https://numpy.org/numpy-financial/latest/irr.html)。这个文档来龙去脉讲得非常详细，刚好符合这个场景。

于是我写代码验证了一下：
```python
# dilfish for irr

import numpy_financial as nf
# money loaned
moneyList=[-40000]
# paid back in 18 monthes
for i in range(18):
     moneyList.append(2282.22)
# monthly rate * 12 = yearly rate
print(nf.irr(moneyList)*12)
```

如上，最后结果是 0.033834104514713914，和 app 标记的一样，不过总额度只有4万，房贷的利率是4.5, 算下来一年能省400块钱，还是不折腾了。

看起来虽然省不了几个钱，但是知识学到了啊。

[^1]: <大唐先进动力研究院>的渣男对本文亦有贡献，在此表示感谢
[^2]: 这个测试带来了严重后果，招行给我打了好几个电话问要不要贷款

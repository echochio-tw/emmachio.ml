---
layout: post
title: 直線回歸 linear regression
date: 2022-08-19
tags: Python
---

直線回歸 用 DataFrame把圖畫出來

```
x = [6,5,10,8,2]
y = [7,1,9,5,3]
dat = {'X':x,'Y':y}
import pandas as pd
df = pd.DataFrame(dat)
import seaborn as sb
from matplotlib import pyplot as plt
sb.lmplot(x = "X", y = "Y", data = df)
plt.show()
```
計算式：
<img src="/images/posts/regression/1.png">

<img src="/images/posts/regression/2.png">

<img src="/images/posts/regression/3.png">

<img src="/images/posts/regression/4.png">

---
layout: post
title: python streamlit
date: 2022-03-22
tags: streamlit
---

streamlit
```
import streamlit as st
import pandas as pd
import numpy as np
import time

# 設定網頁標題
st.title('My App')

# 加入網頁文字內容
st.write("My first app")

# 加入 pandas 資料表格
st.write(pd.DataFrame({
    'first column': [1, 2, 3, 4],
    'second column': [10, 20, 30, 40]
}))


chart_data = pd.DataFrame(
    np.random.randn(20, 3),
    columns=['a', 'b', 'c'])
st.line_chart(chart_data)


option = st.selectbox(
    '你喜歡哪種動物？',
    ['狗', '貓', '鸚鵡', '天竺鼠'])
'你的答案：', option

latest_iteration = st.empty()
bar = st.progress(0)
for i in range(100):
    latest_iteration.text(f'目前進度 {i+1} %')
    bar.progress(i + 1)
    time.sleep(0.1)

```
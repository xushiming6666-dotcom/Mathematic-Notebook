# 一、数据导入
最常见的数据
```python
df.columns
#stock_code
#trade_date
#close
#volume
#factor
#ret
```
第一件事情一定是把字符串转为可识别时间
```python
df["trade_date"]=pd.to_datetime(df["trade_date"])

df=df.sort_values(
                  ["stock_code","trade_date"]
                   ).reset_index(drop=True)
#把股票先按照stock_date分类 然后按照trade_date来排序这就是最基本的数据清理步骤
```
# 二、收益率Return
```python
df["ret"]=(
           df.groupby("stock_code")["close"]
            .pct_change()
           )
#groupby这里的作用就是分割，防止A股的最后一日数据和B股第一日数据串起来无意义。
```

# 三、Shift()函数使用
shift函数功能可以实现dataframe中数据的上下移动，其中若Positive就代表现在拿到过去的信息；而Negative代表拿到了未来的信息，常在计算未来收益率中使用。未来收益率则和IC IR等量化指标有关。
例如，生成一列昨日收盘价:
```python
df["close_lag1"]=(
                  df.groupby("stock_code")["close"].shift(1)
                  )
#依然要记住，不想股票混用那就得加个.groupby来分割表
```
计算未来一天收益率:
```python
df["future_ret_1d"]=(
                     df.groupby("stock_code")["close"].shift(-1)/df["close"]-1
                     )
#注意这里用到了shift(-1)，如果你的因子涉及这个信息，那么就是典型的look-ahead-bias 切记切记
```

#四、rolling() 函数使用
rolling()函数和滚动窗口有很大的关联，Alpha_101中大量的因子难以实现，我们可以采用滚动窗口的方式来生产因子，同样这部分是量化笔试面试中重点的部分。
## 1.过去20天的均值
```python
df["ma20"]=(
             df.groupby["stock_code"].("close")
              .transform(lambda x:
                          x.rolling(20).mean()
                         )
              )
#transform的好处就是既操作了比如求mean()，还不压缩行，例如原来100万行数据，处理完了还是100万行
```
## 2.过去20天的波动率
```python
df["vol20"]=(
              df.groupby("stock_code")[ret]
              .transform(lambda x:x.rolling(20).std())
            )
```

## 3.过去20天的累积动量
```python
df["mom20"]=(
             df.groupby("stock_code")["close"]
             .transform(lambda x:x/x.shift(20)-1)
             )
#累积动量就是一个特殊的收益率，可以看作期末期初这样的
```

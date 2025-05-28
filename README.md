# Jupyter Notebook基础教程

## 创建一个新的Notebook

新建一个Notebook Python 3 (ipykernel)


## 关于cell 和 kernel。

## cell

单元格（cell）
主要分为两类：
代码单元格：包含可执行代码，运行后在下方显示输出结果
Markdown 单元格：用于编写 Markdown 格式的文本内容
试着输入一行代码，查看执行效果：

```python
print('Hello World!')
```

    Hello World!
    


## 快捷键

单元格模式
存在两种操作模式：
编辑模式：按 Enter 进入，单元格边框为绿色
命令模式：按 Esc 进入，单元格边框为蓝色
常用快捷键
通过 Ctrl + Shift + P 可查看所有命令，以下是命令模式下的高频操作：
上下箭头：在单元格间移动
A / B：在当前单元格上方 / 下方插入新单元格
M：将当前单元格转为 Markdown 类型
Y：将当前单元格转为代码类型
D+D（连续按两次 D）：删除当前单元格
Z：撤销删除操作
H：打开快捷键帮助文档
编辑模式下，按 Ctrl + Shift + - 可从光标处拆分单元格。

## Kernel

每个 Notebook 依赖一个内核运行，代码在单元格中执行时由内核处理，且内核状态在整个文档中保持一致。这意味着在某个单元格定义的函数或变量，可在其他单元格直接使用。例如：
python

import numpy as np
def square(x):
    return x * x

执行上述代码后，后续单元格可调用 np 和 square：
python

x = np.random.randint(1, 10)
y = square(x)
print(f'{x} squared is {y}')



## 简单的Python程序的例子

### 定义selection_sort函数执行选择排序功能。

```python
# 选择排序算法定义
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_index = i  # 初始化最小元素索引
        for j in range(i + 1, n):
            if arr[j] < arr[min_index]:
                min_index = j  # 更新最小元素索引
        arr[i], arr[min_index] = arr[min_index], arr[i]  # 交换元素
    return arr

```

### 定义test函数进行测试，执行数据输入

```python

# 测试函数定义
def test():
    # 数据输入处理
    raw_input = input("请输入一组数字，用空格分隔：")
    arr = list(map(int, raw_input.strip().split()))
    
    # 输出原始数据
    print("原始数据：", arr)
    
    # 执行排序
    sorted_arr = selection_sort(arr)
    
    # 输出排序结果
    print("排序后结果：", sorted_arr)

# 调用测试函数
test()


```


## 数据分析的例子

#### 设置

导入相关的工具库

```python
%matplotlib inline
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

加载数据集。

```python
df = pd.read_csv('fortune500.csv')
```

#### 检查数据集

```python
df.head()
```

![image](https://github.com/user-attachments/assets/8ca63e29-72ef-499a-a980-1cf68bc45c34)


```python
df.tail()
```

![image-20250507091205007](C:\Users\18328\Desktop\软件项目实践3\project 4\img\image-7.png)


检查数据条目是否加载完整。

```
len(df)
```

![image](https://github.com/user-attachments/assets/9c1a7bb0-658f-4160-a25f-82224cc6e8c2)


从1955至2055年总共有25500条目录。

对于profit属性，期望的结果是float类型，因此其可能包含非数字的值，利用正则表达式进行检查。

```python
non_numberic_profits = df.profit.str.contains('[^0-9.-]')
df.loc[non_numberic_profits].head()

```


![image](https://github.com/user-attachments/assets/3062c79c-667c-491f-9015-35daf866a755)


确实存在这样的记录，profit这一列为字符串，统计一下到底存在多少条这样的记录。

```
len(df.profit[non_numberic_profits])
```

```
369
```

总体来说，利润（profit）列包含非数字的记录相对来说较少。更进一步，使用直方图显示一下按照年份的分布情况。

![image](https://github.com/user-attachments/assets/88e66f5e-a356-45ae-80ef-020495a6d5f0)


可见，单独年份这样的记录数都少于25条，即少于4%的比例。这在可以接受的范围内，因此删除这些记录。

```python
df = df.loc[~non_numberic_profits]
df.profit = df.profit.apply(pd.to_numeric)

```

再次检查数据记录的条目数。

```
len(df)
```

```
25131
```

### 使用matplotlib进行绘图

接下来，以年分组绘制平均利润和收入。首先定义变量和方法。

```python
group_by_year = df.loc[:, ['year', 'revenue', 'profit']].groupby('year')
avgs = group_by_year.mean()
x = avgs.index
y1 = avgs.profit
def plot(x, y, ax, title, y_label):
    ax.set_title(title)
    ax.set_ylabel(y_label)
    ax.plot(x, y)
    ax.margins(x=0, y=0)

```

开始绘图

```python
fig, ax = plt.subplots()
plot(x, y1, ax, 'Increase in mean Fortune 500 company profits from 1955 to 2005', 'Profit (millions)')

```

![image](https://github.com/user-attachments/assets/58fe6d29-59dc-4abc-8377-c0a17e6e347e)



```python
y2 = avgs.revenue
fig, ax = plt.subplots()
plot(x, y2, ax, 'Increase in mean Fortune 500 company revenues from 1955 to 2005', 'Revenue (millions)')

```

![image](https://github.com/user-attachments/assets/f3c00005-ac3d-43fe-97f9-b41d386fb954)



公司收入曲线并没有出现急剧下降，可能是由于财务会计的处理。对数据结果进行标准差处理。

```python
def plot_with_std(x, y, stds, ax, title, y_label):
    ax.fill_between(x, y - stds, y + stds, alpha=0.2)
    plot(x, y, ax, title, y_label)
fig, (ax1, ax2) = plt.subplots(ncols=2)
title = 'Increase in mean and std Fortune 500 company %s from 1955 to 2005'
stds1 = group_by_year.std().profit.values
stds2 = group_by_year.std().revenue.values
plot_with_std(x, y1.values, stds1, ax1, title % 'profits', 'Profit (millions)')
plot_with_std(x, y2.values, stds2, ax2, title % 'revenues', 'Revenue (millions)')
fig.set_size_inches(14, 4)
fig.tight_layout()

```

![image](https://github.com/user-attachments/assets/da1cc822-9498-4218-a63a-7dafca43f63b)




可见，不同公司之间的收入和利润差距惊人，那么到底前10%和后10%的公司谁的波动更大了？此外，还有很多有价值的信息值得进一步挖掘。

#### 在同一张图上展示收入和利润

```python
import matplotlib.pyplot as plt

# 使用同一个图和坐标轴
fig, ax = plt.subplots()

# 画利润曲线
plot(x, y1, ax, 'Mean Fortune 500 Company Revenue and Profit (1955–2005)', 'Millions USD')
# 再画收入曲线
ax.plot(x, y2, label='Revenue')  # 添加label方便图例区分
ax.plot(x, y1, label='Profit')   # 再画一次Profit并加label（上面plot函数没有加label）

# 添加图例
ax.legend()

```

需要复用同一个 `ax`（坐标轴）并调用 `plot()` 两次，分别传入利润和收入数据。

![image](https://github.com/user-attachments/assets/5b0c3d1f-7a1e-47e7-b7e4-ec1ab25f575e)


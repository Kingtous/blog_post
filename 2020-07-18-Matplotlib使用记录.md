---
layout: post
title: Matplotlib使用记录
subtitle: python、matplotlib
author: Kingtous
date: 2020-07-18 19:34:50
header-img: img/post-bg-universe.jpg
catalog: True
tags:
- Python
---

```python
x = np.array([1,2,3,4,5])
y1 = np.array([3,4,5,1,2])
y2 = np.array([2,1,4,6,5])
plt.fill(x, y1, 'b', x, y2, 'r', alpha=0.3)
```



```python
[<matplotlib.patches.Polygon at 0x7f0efff73bd0>,
 <matplotlib.patches.Polygon at 0x7f0efff586d0>]
```



![](http://img.kingtous.cn/img/20200718193632.png)



```python
colors=np.random.rand(5)
area=(30*np.random.rand(5))**2
print(area)
plt.scatter(x,y1,s=area,alpha=0.6,c=colors)
[4.90499346e+02 3.70129905e+02 4.63484403e-03 5.01227914e+02
 1.57868686e+00]
```





```python
<matplotlib.collections.PathCollection at 0x7f0efbf9a8d0>
```



![](http://img.kingtous.cn/img/20200718193648.png)



```python
# 3D
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure(figsize=(12, 8))
ax = Axes3D(fig)
'''
fig = plt.figure(figsize=(12, 8))
ax = fig.gca(projection='3d')
'''

delta = 0.125
# 生成代表X轴数据的列表
x = np.arange(-3.0, 3.0, delta)
# 生成代表Y轴数据的列表
y = np.arange(-2.0, 2.0, delta)
# 对x、y数据执行网格化
X, Y = np.meshgrid(x, y)
Z1 = np.exp(-X**2 - Y**2)
Z2 = np.exp(-(X - 1)**2 - (Y - 1)**2)
# 计算Z轴数据（高度数据）
Z = (Z1 - Z2) * 2
# 绘制3D图形
surf = ax.plot_surface(X, Y, Z,
                       rstride=1,  # rstride（row）指定行的跨度
                       cstride=1,  # cstride(column)指定列的跨度
                       cmap=plt.get_cmap('rainbow'))  # 设置颜色映射
# 设置Z轴范围
ax.set_zlim(-2, 2)
# 设置标题
plt.title("3D图")
fig.colorbar(surf, shrink=0.5, aspect=5)
```



```python
<matplotlib.colorbar.Colorbar at 0x7f0efbec8f50>
```



```python
/usr/local/lib/python3.7/dist-packages/matplotlib/backends/backend_agg.py:214: RuntimeWarning: Glyph 22270 missing from current font.
  font.set_text(s, 0.0, flags=flags)
/usr/local/lib/python3.7/dist-packages/matplotlib/backends/backend_agg.py:183: RuntimeWarning: Glyph 22270 missing from current font.
  font.set_text(s, 0, flags=flags)
```



![](http://img.kingtous.cn/img/20200718193702.png)
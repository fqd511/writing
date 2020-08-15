# Tile Map System | 瓦片地图

### 简介

---

瓦片地图就是在渲染地图时将地图切割成多个小块，以便检索和显示，因为切割方式形似瓦片（Tile），故称为瓦片地图（Map Tile System）。说人话就是对地图进行网格化处理，本文主要内容包括瓦片地图使用的投影方式、坐标体系等。
### 投影法

---

由于地球是一个赤道略宽两极略扁的不规则梨形球体，表面是一个不可展平的曲面，所以运用任何数学方法进行投影都会产生误差和变形，因此实际情况中会根据使用场景选取误差较小的投影方式。
常见的投影法有墨卡托(Mercator)投影(85%)、Lambert等角正割圆锥投影(5%)、Albers等积正割圆锥投影、等距圆锥投影等，本文涉及到的瓦片地图使用墨卡托投影法。
Mercator 投影有以下几点需要注意：

- 缺点：Mercator 投影对实际情况的扭曲会随着维度增加而放大，在极点时逼近无限大，因此在处理时只考虑±85.05°维度以内的区域；

- 优点：Mercator 投影是共形映射，可以避免扭曲建筑物的形状（共形映射是一个保持角度不变的映射，保持了角度以及无穷小物体的形状，但是不一定保持它们的尺寸）

- 优点：Mercator 投影是圆柱投影，纬线和经线正好能对应成平面坐标中的横轴和纵轴


实际处理时将地球的椭球面处理成球面，带来纵轴方向约 0.33%的失真
### 比例换算

---

比例尺表示图上距离与实际距离的比值，计算公式为：
1 ： 每像素表示的实际距离*每单位图上距离包含的像素数
> 下面以 Bing地图举例：

#### 缩放级别
最低级别（level 1）地图的尺寸是 512*512 像素，每增加一级，地图长宽都乘以 2，计算公式为：
map width = map height = 256 * 2 ^level pixels
#### 地面分辨率
地面分辨率表示地面上的距离，由地图中的单个像素表示（比如：10米 / 像素）。地面分辨率与缩放比例和维度有关。以米 / 像素为单位可以计算为:
地面分辨率 = cos (纬度 * π / 180) * 地球周长 / 地图宽度
#### 比例尺
比例尺表示地图距离与实际距离之间的比值，与缩放比例和维度有关，计算公式为：
比例尺 = 1: (地面分辨率 * 屏幕 dpi / 0.0254米 / 英寸)
注意上面公式右边的单位计算是：米/像素  * 像素/英寸  *  米/英寸，单位会互相消掉
#### 坐标体系
##### 像素坐标
参考网页定位常见方式，左上角为（0，0），右下角为（宽度-1，高度-1）。结合前文的缩放比例就是：
(256 * 2^level-1,256 * 2^level-1)
加入经纬度的转换，先计算：
sinLatitude = sin (维度 * π / 180)
像素横坐标：
((经度 + 180) / 360) * 256 * 2 ^level   （实际上就是根据经度进行平移，因为经线竖直）
像素纵坐标：
(0.5 – log((1 + sinLatitude) / (1 – sinLatitude)) / (4 * π)) * 256 * 2 ^level
此处的经纬度参考 WGS 84 标准
##### 网格坐标和quadkey表示法
如图给每一个网格分配一个坐标（网格大小为 256*256）
![](https://cdn.nlark.com/yuque/0/2020/jpeg/164181/1577974890397-d6bdfa4b-1625-4beb-bcda-6b0fb9ab2daf.jpeg#align=left&display=inline&height=334&originHeight=334&originWidth=336&size=0&status=done&style=none&width=336)
根据像素坐标计算包含该像素点的网格坐标：
tileX = floor(pixelX / 256)

tileY = floor(pixelY / 256)
###### 转化为quadkey
![](https://cdn.nlark.com/yuque/0/2020/jpeg/164181/1577974890292-0f6eeb70-3138-4779-b219-b7f9698f5b90.jpeg#align=left&display=inline&height=365&originHeight=365&originWidth=623&size=0&status=done&style=none&width=623)
> **具体方法**：以网格坐标（3,5）为例，先转化为二进制（011,101），然后交叉组合，顺序是：纵坐标左数第一位→横坐标左数第一位→纵坐标左数第二位→横坐标左数第二位→。。。→横坐标最右位，组合完毕为100111，转化为 4 进制为 213，即为（3，5）的 quadkey 表示法

**quadkey 注意点**：

- quadkey 位数等于 level 数

- 任何网格的 quadkey 都以其父块的 quadkey 开头

- quadkey 作为一维索引，保留了网格的接近程度信息，便于数据库性能优化

### 参考资料

---

- [Bing Maps Tile System](https://docs.microsoft.com/en-us/bingmaps/articles/bing-maps-tile-system)

- [八类地图常用的投影方法](http://blog.sciencenet.cn/blog-2637373-974051.html)

- [共形映射](https://zh.wikipedia.org/wiki/%E5%85%B1%E5%BD%A2%E6%98%A0%E5%B0%84)

- [WGS](https://zh.wikipedia.org/wiki/%E4%B8%96%E7%95%8C%E5%A4%A7%E5%9C%B0%E6%B5%8B%E9%87%8F%E7%B3%BB%E7%BB%9F)


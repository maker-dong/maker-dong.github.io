# postgis常用函数

# 安装postgis扩展

```sql
CREATE EXTENSION postgis;
```



# ST_AsText 获取几何图形的文本

>  ST_AsText 获取一个几何，然后返回其可识别的文本表示。

- 语法：

```
st_astext (geometry input)
```

- 参数：
  - geometry：一个geometry类型的几何图形
- 返回值：文本类型的集合描述，例如：
  - `POINT (1020.12000000 324.02000000)`
  -  `MULTIPOLYGON(((117.660971 36.262423,117.661244 36.263544,117.662778 36.263477,117.662708 36.261338,117.662667 36.259048,117.660318 36.259408,117.660971 36.262423)))`

# ST_GeomFromText 将WKT字符串转化为几何图形

> 返回一个与给定的WKT字符串相对应的Geometry对象。

- 语法：

```
geometry ST_GeomFromText(text WKT);
geometry ST_GeomFromText(text WKT , integer srid);
```

- 参数：
  - WKT：WKT字符串
  - srid：Geometry对象的空间参考系ID
- 返回值：一个geometry类型的几何图形
  - `MULTIPOLYGON (((117.065207 36.670192, 117.065203 36.670149, 117.065289 36.670147, 117.065295 36.670204, 117.065207 36.670192)))`

# ST_Point 构造Point对象

> 使用给定的坐标值构造Point对象。

- 语法：

```
ST_Point(float  xLon , float  yLat, integer srid);
```

- 参数：
  - xLon：经度
  - yLat：维度
  - srid： Geometry对象的空间参考系ID
- 返回值：一个geometry的point点
  - `POINT (119.949313 36.361594)`

# ST_MakeValid 修复无效的几何图形

> 尝试创建给定无效几何体的有效表示，而不丢失任何输入顶点。有效几何图形原封不动地返回。
>
> - 如果有部分或者全部维度损失，输出的Geometry对象是一个更低维度Geometry对象的集合或者一个更低维度的Geometry对象。

- 语法：

```
ST_MakeValid(geometry input);
```

- 参数：
  - input：一个geometry类型的几何图形
- 返回值：修复后的geometry

# ST_Distance 计算两点间距离

>  ST_Distance 用于返回两个几何之间的距离。这一距离是两个几何的最近折点之间的距离。

- 语法：

```
st_distance (geometry input1, geometry input2)
st_distance (geometry input1, geometry input2, text unit_name)
```

- 参数：

  - input1：一个geometry类型的点

  - input2：一个geometry类型的点

  - unit_name：非必传，距离单位，默认是公里

- 返回值：float

# ST_Within 包含关系

> 如果给定的Geometry对象A完全在对象B之内，则返回True。
>
> - 为使此函数有意义，两个Geometry对象必须都具有相同的投影方式，且具有相同的空间参考（SRID）。
> - 如果ST_Within(A,B)为True且ST_Within(B,A)为True，则认为这两个Geometry对象在空间上相等。

- 语法：

```
ST_Within(geometry  A , geometry  B);
```

- 参数：
  - A：一个geometry类型的对象
  - B：一个geometry类型的对象
- 返回值：布尔值
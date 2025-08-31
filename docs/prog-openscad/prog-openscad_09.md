# 第九章：A

OpenSCAD 语言参考

![](img/chapterart.png)

本语言参考提供了对大多数 OpenSCAD 功能的简短描述，作为快速提醒，帮助使用本书中描述的功能或发现新的 OpenSCAD 特性。请参考官方 OpenSCAD 文档 [`openscad.org/documentation`](https://openscad.org/documentation) 获取完整手册。

## 语法

1.  使用一组参数创建一个 2D 或 3D 形状。用分号（`;`）结束命令：

    ```
    shape(...);
    ```

1.  创建一个已通过一系列操作转换的形状。用分号（`;`）结束语句：

    ```
    transformation2(...) transformation1(...) shape(...);
    ```

1.  创建一个变量来命名和引用一个重要的值；值一旦赋值后无法更改：

    ```
    var_name = value;
    ```

1.  创建一个名为 `name` 的用户定义形状，具有零个或多个参数。用户定义的形状与内建形状的工作方式相同：

    ```
    module name(...) { ... } 
    name(...); 
    ```

1.  创建一个名为 `name` 的用户定义数学运算，具有零个或多个参数：

    ```
    function name(...) = ...;
    name(...);
    or
    name = function(...) ...;
    name(...);
    ```

1.  导入并立即执行 *filename.scad* 中的 OpenSCAD 代码：

    ```
    include `<filename.scad>`
    ```

1.  导入并使（但不立即执行）*filename.scad* 中的 OpenSCAD 函数和模块可用：

    ```
    use `<filename.scad>`
    ```

## 运算符

运算符按优先级降序排列。当多个相同优先级的运算符出现在表达式中时，运算符按出现顺序（从左到右）进行计算：

1.  `^`

1.  `*`, `/`, `%`

1.  `+`, `-`

1.  `<`, `>`, `<=`, `>=`

1.  `==`, `!=`

1.  `&&`

1.  `||`

## 2D 形状

1.  绘制一个指定半径或直径的圆：

    ```
    circle(`radius` | d=`diameter`)
    ```

1.  绘制一个边长为 *size* 的正方形，宽度 = *size*，高度 = *size*（相等边长）；可选择将正方形居中于 (0,0)：

    ```
    square(`size`, `center`)
    ```

1.  绘制一个矩形，宽度沿 x 轴，长度/深度沿 y 轴，由向量定义；可选择将正方形居中于 (0,0)：

    ```
    square([`width`, `height`], *center*)
    ```

1.  绘制一个连接所有由 [x, y] 点向量定义的点的多边形：

    ```
    polygon([*[*`x1`, `y2`*]**,* *[*`x2, y2`*]**, ...,* *[*`xn, yn`*]*])
    ```

1.  绘制一个连接所有由 [x, y] 点向量定义的点的多边形；可选择定义一个包含有孔多边形路径的集合：

    ```
    polygon(*[*`points`*]*, *[*`paths`*]*)
    ```

1.  绘制由 *text* 字符串定义的文字；可选择指定文字的大小、字体、水平对齐、垂直对齐、字母间距、方向、语言和脚本：

    ```
    text(`text`, *size*, *font*, *halign*, *valign*, 
    *spacing*, *direction*, *language*, *script*)
    ```

1.  导入一个 2D SVG 或 DXF 文件：

    ```
    import("`filename.svg`")
    ```

## 3D 形状

1.  绘制一个以 (0, 0, 0) 为中心，指定半径或直径的球体：

    ```
    sphere(`radius` | d=`diameter`)
    ```

1.  绘制一个立方体，长度 = *size*，宽度 = *size*，高度 = *size*（相等边长）；可选择将立方体居中于 (0,0,0)：

    ```
    cube(`size`, *center*)
    ```

1.  绘制一个由向量定义的宽度沿 x 轴，长度/深度沿 y 轴，高度沿 z 轴的立方体；可选择将立方体居中于 (0,0,0)：

    ```
    cube([`width`, `depth`, `height`], center)
    ```

1.  绘制一个指定高度和半径或直径的圆柱体；可选择将圆柱体居中于 (0,0,0)：

    ```
    cylinder(*h*, *r*|*d*, center)
    ```

1.  绘制一个指定高度和半径或直径的圆锥体；可选择将圆锥体居中于 (0,0,0)：

    ```
    cylinder(*h*, *r1*|*d1*, *r2*|*d2*, center)
    ```

1.  绘制一个由点和面向量定义的 3D 实体；可选择使用凸度来改进复杂凹形的预览：

    ```
    polyhedron(*[*`points`*]*, *[*`faces`*]*, convexity)
    ```

1.  导入一个 STL、OFF、3MF 或 AMF 文件：

    ```
    import("`filename.stl`")
    ```

1.  绘制数据文件的 3D 高度图；可选地将形状居中于 (0,0) 并使用凸性来改善复杂凹形状的预览：

    ```
    surface(file = "`filename.dat`", center, convexity)
    ```

## 布尔操作

1.  将多个形状组合成一个形状：

    ```
    union() { ... }
    ```

1.  从初始形状中减去一个或多个形状：

    ```
    difference() { ... }
    ```

1.  绘制多个形状的重叠区域：

    ```
    intersection() { ... }
    ```

## 形状变换

1.  根据 2D 或 3D 向量平移形状：

    ```
    translate([`x`, `y`, `z`])
    ```

1.  根据向量定义的角度围绕每个轴旋转形状：

    ```
    rotate([`x`, `y`, `z`])
    ```

1.  沿 z 轴旋转形状特定的角度：

    ```
    rotate(`angle`)
    ```

1.  根据由 2D 或 3D 向量定义的缩放因子缩放形状：

    ```
    scale([`x`, `y`, `z`])
    ```

1.  根据由 2D 或 3D 向量定义的维度调整形状大小；可选地使用 `auto` 保持在未指定维度中的对象纵横比：

    ```
    resize([`x`, `y`, `z`], auto, convexity)
    ```

1.  根据通过原点的对称平面垂直向量反射形状：

    ```
    mirror([`x`, `y`, `z`])
    ```

1.  用给定的 4 × 4 仿射变换矩阵将所有子元素的几何形状相乘：

    ```
    multmatrix(`matrix`)
    ```

1.  根据预定义的颜色名称或十六进制颜色值改变形状的颜色；可选地使颜色（半）透明：

    ```
    color("`colorname` | `#hex`", alpha)
    ```

1.  根据 RGB 或 RGBA 向量改变形状的颜色。向量中的每个值范围从 0 到 1，表示颜色中红、绿、蓝和 alpha 的比例。

    ```
    color([`r`, `g`, `b`, `a`])
    ```

1.  按给定的半径（用于圆角）或增量 + 倒角（用于锐角或切角）将 2D 外框向外或向内移动：

    ```
    offset(`r`|delta, chamfer)
    ```

1.  通过将 3D 形状投影到 xy 平面上创建 2D 形状；当 `cut = true` 时，创建 3D 物体与 xy 平面的交集的 2D 切片；可选地，当 `cut = true` 时：

    ```
    projection(cut)
    ```

1.  在一个或多个形状周围创建凸包：

    ```
    hull() { ... }
    ```

1.  绘制多个形状的 Minkowski 和：

    ```
    minkowski() { ... }
    ```

1.  将 2D 形状沿 z 轴挤压成 3D 形状；可选地将形状居中于 (0,0) 或指定挤压的凸性、扭曲、切片和缩放：

    ```
    linear_extrude(*height*, *center*, *convexity*, *twist*, *slices*, *scale*)
    ```

1.  将 2D 形状围绕 z 轴挤压成具有旋转对称性的固体：

    ```
    rotate_extrude(angle, convexity)
    ```

## 循环、决策和列表推导

1.  根据控制变量的起始、步长和结束（包含）值重复一组形状：

    ```
    for (`var_name` = [`start`:`step`:`end`]) { ... }
    ```

1.  绘制 `for` 循环生成的所有形状的交集：

    ```
    intersection_for(`var_name` = [`start`:`step`:`end`]) { ... }
    ```

1.  仅在布尔测试为真时执行命令：

    ```
    if (`boolean_test`) { ... }
    ```

1.  如果布尔测试为真，则执行一组命令；否则，执行备用命令：

    ```
    if (`boolean_test`) { ... } else { ... }
    ```

1.  根据 `for` 循环生成一个值的列表：

    ```
    `list_var` = [ for (i = `range`|`list`) `func`(`i`) ]
    ```

1.  根据 `for` 循环生成一个值的列表，但仅在该值导致某个条件为真时：

    ```
    `list_var` = [ for (`i` = ...) if (`condition`(`i`)) `func`(`i`) else ... ]
    ```

1.  根据 `for` 循环生成一个列表的列表：

    ```
    `list_var` = [ for (`i` = ...) let (`assignments`) `func`(...) ]
    ```

## 其他形状操作

1.  即使在预览模式下也强制生成网格：

    ```
    render(convexity) { ... }
    ```

1.  在用户定义的模块内，选择由索引、向量或范围指定的子元素：

    ```
    children(`index` | `vector` | `range`)
    ```

## 修饰符字符

1.  `*` 禁用绘制形状。

1.  `!` 仅显示特定形状。

1.  `#` 将形状高亮为红色用于调试；高亮的形状将被渲染。

1.  `%` 将形状高亮为灰色；高亮的形状将不会被渲染。

## 特殊变量

**可写：**

1.  `$fa` 弧段的最小角度。

1.  `$fs` 弧段的最小大小。

1.  `$fn` 用于定义弧段的碎片数；忽略`$fa`和`$fs`。

1.  `$vpr` 视口旋转角度，单位为度。

1.  `$vpt` 视口平移。

1.  `$vpd` 视口相机的距离。

1.  `$vpf` 视口视野。

**只读：**

1.  `$t` 当前动画步骤，归一化为 0 到 1 之间的值。

1.  `$children` 模块子元素的数量。

1.  `$preview` 如果使用了预览模式，返回真。

## 数学函数

1.  `sin(``ANGLE``)` 计算一个角度的正弦，单位为度。

1.  `cos(``ANGLE``)` 计算一个角度的余弦，单位为度。

1.  `tan(``ANGLE``)` 计算一个角度的正切，单位为度。

1.  `acos(``NUMBER``)` 计算一个数的弧（反）余弦，单位为度。

1.  `asin(``NUMBER``)` 计算一个数的弧（反）正弦，单位为度。

1.  `atan(``NUMBER``)` 计算一个数的弧（反）正切，单位为度。

1.  `atan2(``y``,` `x``)` 计算两值的弧（反）正切；返回 x 轴和向量[*x, y*]之间的完整角度（0–360 度）。

1.  `abs(``NUMBER``)` 计算一个数的绝对值。

1.  `sign(``NUMBER``)` 返回一个单位值，用于提取值的符号。

1.  `floor(``NUMBER``)` 计算不大于该数的最大整数。

1.  `ceil(``NUMBER``)` 计算下一个更高的整数值。

1.  `round(``NUMBER``)` 计算该数的四舍五入值。

1.  `ln(``NUMBER``)` 计算一个数的自然对数。

1.  `exp(``NUMBER``)` 计算数学常数 *e*（2.718…）的幂。

1.  `log(``NUMBER``)` 计算一个数的以 10 为底的对数。

1.  `pow(``NUMBER``,` `NUMBER``)` 计算一个基数的幂。

1.  `sqrt(``NUMBER``)` 计算一个数的平方根。

``` *   `rands(``min``,` `max``,` `count``,` `seed``)` Generates a vector of random numbers; optionally includes the seed for generating repeatable values.*   `min(``VECTOR` `|` `a``,` `b``,` `c``)` Calculates the minimum value in a vector or list of parameters.*   `max(``VECTOR` `|` `a, b, c``)` Calculates the maximum value in a vector or list of parameters.*   `norm(``VECTOR``)` Returns the Euclidean norm of a vector.*   `cross(``VECTOR, VECTOR``)` `Calculates the cross-product of two vectors in 3D space.` ```

````` ```` ## Other Functions    1.  `len(``VECTOR``|``STRING``)` Calculates the length of a vector or string parameter. 2.  `echo(``STRING``)` Prints a value to the console window for debugging purposes. 3.  `concat(``VECTOR,VECTOR,` `...)` Returns a new vector that’s the result of appending the elements of the supplied vectors. 4.  `lookup(...)` Looks up a value in a table and linearly interpolates whether there’s no exact match. 5.  `str(...)` Converts all parameters to strings and concatenates. 6.  `chr(``NUMBER` `|` `VECTOR` `|` `STRING``)` Converts ASCII or Unicode values to a string. 7.  `ord(``CHARACTER``)` Converts a character into an ASCII or Unicode value. 8.  `search(...)` Finds all occurrences of a value or list of values in a vector, string, or more complex list-of-list construct. 9.  `version()` Returns the OpenSCAD version as a vector. 10.  `version_num()` Returns the OpenSCAD version as a number. 11.  `parent_module(``INDEX``)` ``Returns the name of the module `idx` levels above the current module in the instantiation stack.`` ```*   `is_undef(``VARIABLE``)`, `is_list(``VARIABLE``)`, `is_num(``VARIABLE``)`, `is_bool(``VARIABLE``)`, `is_string(``VARIABLE``), is_function(``VARIABLE``)` Returns `true` if the argument is of the specified type.*   `assert(``expression``)` Will cause a compilation error if the expression is not true.*   `let (``variable` `=` `value``) ...` Assigns a value to a variable only in the following expression.``` ```` `````
